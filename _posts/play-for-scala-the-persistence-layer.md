title: Play for Scala-the persistence layer
date: 2014-08-11 08:00:25
tags:
- scala
- play
---

# What are Anorm and Squeryl? #


In order to talk to the database, you’ll have to create SQL at some point.
A modern object-relation mapper (ORM) like Hibernate or the Java Persistence API (JPA)
provides its own query language (HQL and JPQL, respectively), which is then translated
into the target database’s SQL dialect.


Anorm and Squeryl are at opposite ends of the SQL-generation/translation spectrum.

Squeryl generates SQL by providing a Scala domain-specific language ( DSL)
that’s similar to actual SQL. Anorm doesn’t generate any SQL, and instead
relies on the developer to write SQL. 

* Anorm allows you to write any SQL that you can come up with, even using
proprietary extensions of the particular database that you’re using.
* Squeryl’s DSL allows the compiler to check that your queries are correct, which
meshes well with Play’s emphasis on type safety.

## Configuring your database ##

Play comes with support for an H2 in-memory database out of the box, but there’s no
database configured by default. 


`conf/application.conf`, 要配置其他的数据库, 可以注释掉
~~~~~~
db.default.driver=org.h2.Driver
db.default.url="jdbc:h2:mem:play"
~~~~~~

An in-memory database is fine for development and testing but doesn’t cut it
for most production environments.

`Build.scala`
~~~~~~
val appDependencies = Seq(
  jdbc,
  anorm,
  "postgresql" % "postgresql" % "9.1-901.jdbc4"
)
~~~~~~
`application.conf`
~~~~~~
db.default.user=user
db.default.password=qwerty
db.default.url="jdbc:postgresql://localhost:5432/paperclips"
db.default.driver=org.postgresql.Driver
~~~~~~

## Creating the schema ##

Anorm can’t create your schema for you because it doesn’t know anything about your model.
Squeryl can create your schema for you, but it’s unable to update it. This means
you’ll have to write the SQL commands to create (and later update) your schema yourself.

Play does offer some help in the form of evolutions. To use evolutions, you write an
SQL script for each revision of your database; Play will then automatically detect that a
database needs to be upgraded and will do so after asking for your permission.

Evolutions scripts should be placed in the `conf/evolutions/default` directory
and be named `1.sql` for the first revision, `2.sql` for the second, and so on.

~~~~~~
# --- !Ups
CREATE TABLE products (
  id long,
  ean long,
  name varchar,
  description varchar);
  
CREATE TABLE warehouses (
  id long,
  name varchar);

CREATE TABLE stock_items (
  id long,
  product_id long,
  warehouse_id long,
  quantity long);

# --- !Downs
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS warehouses;
DROP TABLE IF EXISTS stock_items;
~~~~~~

Just click the red button labeled “Apply this script now!”

# Using Anorm #

Anorm lets you write SQL queries and provides an API to parse result sets. 

When you retrieve data with Anorm, there are three ways to
process the results: the Stream API, pattern matching, and parser combinators. 

## Defining your model ##

Therefore, your model is simply a bunch of classes that represent the entities
that you want to use in your application and store in the database.

~~~~~~
case class Product(
  id: Long,
  ean: Long,
  name: String,
  description: String)
case class Warehouse(id: Long, name: String)

case class StockItem(
  id: Long,
  productId: Long,
  warehouseId: Long,
  quantity: Long)
~~~~~~

## Using Anorm’s stream API ##

~~~~~~
import anorm.SQL
import anorm.SqlQuery

// The apply method has an implicit parameter
// block that takes a java.sql.Connection
// which Play provides in the form of DB.withConnection.
val sql: SqlQuery = SQL("select * from products order by name asc")

import play.api.Play.current
import play.api.db.DB

def getAll:List[Product] = DB.withConnection{
    implicity connection =>
    sql().map(row =>
        Product(row[Long]("id"), row[Long]("ean"),
                row[String]("name"), row[String]("description"))
    ).toList
}
~~~~~~

The `row` variable in the function body passed to map is an `SqlRow` ,
which has an apply method that retrieves the requested field by name.

## Pattern matching results ##

~~~~~~
def getAllWithPatterns: List[Product] = DB.withConnection {
  implicit connection =>
  import anorm.Row
  sql().collect {
    case Row(Some(id: Long), Some(ean: Long),
             Some(name: String), Some(description: String)) =>
      Product(id, ean, name, description)
  }.toList
}
~~~~~~

We’ve said before that the query’s apply method returns a standard Scala `Stream`.

Streams are simply lists that haven’t computed—or in this case retrieved—their
contents yet. This is why we had to convert
them to Lists with toList to actually retrieve the contents.

## Parsing results ##

You can also parse results with parser combinators,1 a functional programming
technique for building parsers by combining other parsers,

### BUILDING A SINGLE-RECORD PARSER ###

We’ll need to retrieve (and therefore parse) our entities many times, so it’s a good
idea to build parsers for each of our entities.

~~~~~~
import anorm.RowParser
val productParser: RowParser[Product] = {
  import anorm.~
  import anorm.SqlParser._
  long("id") ~
  long("ean") ~
  str("name") ~
  str("description") map {
    case id ~ ean ~ name ~ description =>
      Product(id, ean, name, description)
  }
}

import anorm.ResultSetParser
// Anorm needs a ResultSetParser 
val productsParser: ResultSetParser[List[Product]] = {
  productParser *
}

// 查询结果
def getAllWithParser: List[Product] = DB.withConnection {
  implicit connection =>
  sql.as(productsParser)
}
~~~~~~

## BUILDING A MULTIRECORD PARSER ##

To fetch stock item data, we’ll use SQL to query the `products` and
`stock_items` database tables.

~~~~~~
val stockItemParser: RowParser[StockItem] = {
  import anorm.SqlParser._
  import anorm.~
  long("id") ~ long("product_id") ~
  long("warehouse_id") ~ long("quantity") map {
    case id ~ productId ~ warehouseId ~ quantity =>
      StockItem(id, productId, warehouseId, quantity)
  }
}

def productStockItemParser: RowParser[(Product, StockItem)] = {
  import anorm.SqlParser._
  import anorm.~
  // flatten (in map (flatten) ) turns the given ~[Product, StockItem] into a standard tuple.
  productParser ~ StockItem.stockItemParser map (flatten)
}

def getAllProductsWithStockItems: Map[Product, List[StockItem]] = {
  DB.withConnection { implicit connection =>
    val sql = SQL("select p.*, s.* " +
                  "from products p " +
                  "inner join stock_items s on (p.id = s.product_id)")
    val results: List[(Product, StockItem)] =
    sql.as(productStockItemParser *)
    //  Map[Product, StockItem] =>
    //  Map[Product, List[(Product, StockItem)]] => 
    //  Map[Product, List[StockItem]]
    results.groupBy { _._1 }.mapValues { _.map { _._2 } }
  }
}
~~~~~~

## Inserting, updating, and deleting data ##

~~~~~~
def insert(product: Product): Boolean =
  DB.withConnection { implicit connection =>
    val addedRows =
      SQL("""insert
             into products
             values ({id}, {ean}, {name}, {description})""").on(
               "id" -> product.id,
               "ean" -> product.ean,
               "name" -> product.name,
               "description" -> product.description
      ).executeUpdate()
  addedRows == 1
}

def update(product: Product):Boolean =
  DB.withConnection { implicit connection =>
    val updateRows =
      SQL("""update products
             set name = {name},
             ean = {ean},
             description = {description},
             where id = {id}
      """).on (
        "id" -> product.id,
        "name" -> product.name,
        "ean" -> product.ean,
        "description" -> product.description
      ).executeUpdate()
    updatedRows == 1
  }

def delete(product: Product): Boolean =
  DB.withConnection { implicit connection =>
    val updatedRows = SQL("delete from products where id = {id}").
    on("id" -> product.id).executeUpdate()
    updatedRows == 0
  }
~~~~~~


# Using Squeryl #

This means that Squeryl is an ORM that gives you a feature that other
ORMs don’t: a type-safe query language.

You can write queries in a language that the Scala compiler understands,
and you’ll find out whether there are errors in your queries at compile time.

Contrast this with other ORMs (or Anorm—Anorm is not an ORM) that rely
on the database to tell you that there are errors in your query,
and don’t complain until the queries are actually run.

## Plugging Squeryl in ##

`project/Build.scala`
~~~~~~
val appDependencies = Seq(
  jdbc,
  "org.squeryl" %% "squeryl" % "0.9.5-6"
)
~~~~~~

~~~~~~
import org.squeryl.adapters.H2Adapter
import org.squeryl.{Session, SessionFactory}
import play.api.db.DB
import play.api.{Application, GlobalSettings}

object Global extends GlobalSettings {
  override def onStart(app: Application) {
    SessionFactory.concreteFactory = Some(() =>
      Session.create(DB.getConnection()(app), new H2Adapter) )
  }
}
// DB.getConnection is intended to be used in an environment where an
// Application is available as an
// implicit, and you can call it without the second parameter list.
implicit val implicitApp = app
DB.getConnection()

~~~~~~

## Defining your model ##

~~~~~~
import org.squeryl.KeyedEntity

case class Product(
  id: Long,
  ean: Long,
  name: String,
  description: String) extends KeyedEntity[Long]
  
case class Warehouse(
  id: Long,
  name: String) extends KeyedEntity[Long]

case class StockItem(
  id: Long,
  product: Long,
  location: Long,
  quantity: Long) extends KeyedEntity[Long]
~~~~~~

The only thing that’s different from vanilla case classes here is that
we’re extending `KeyedEntity`. This tells Squeryl that it can use the
id for updates and deletes.

### IMMUTABILITY AND THREADS ###

When an object is immutable, you can only change it by making a copy.
This ensures that other threads that have a
reference to the same object won’t be affected by the changes.

### DEFINING THE SCHEMA ###

`org.squeryl.Schema` contains some utility methods and will allow
us to group our entity classes in such a way that Squeryl can make sense of them.

We’ve defined three classes to contain records, and we’ve
told Squeryl which tables we want it to create and how to map them to our model.

~~~~~~
import org.squeryl.Schema
import org.squeryl.PrimitiveTypeMode._

object Database extends Schema {
  // The table method returns a table for the
  // class specified as the type parameter
  val productsTable: Table[Product] =
    table[Product]("products")
  val stockItemsTable: Table[StockItem] =
    table[StockItem]("stock_items")
  val warehousesTable: Table[Warehouse] =
    table[Warehouse]("warehouses")
    
  on(productsTable) { p => declare {
    p.id is(autoIncremented)
  }}
  
  on(stockItemsTable) { s => declare {
    s.id is(autoIncremented)
  }}
  
  on(warehousesTable) { w => declare {
    w.id is(autoIncremented)
  }}
}
~~~~~~
Squeryl does define a create method that creates the schema when
called from the Database object.

## Extracting data—queries ##

~~~~~~
import org.squeryl.PrimitiveTypeMode._
import org.squeryl.Table
import org.squeryl.Query
import collection.Iterable
object Product {
  import Database.{productsTable, stockItemsTable}
  // product: Query result row name
  def allQ: Query[Product] = from(productsTable) {
    product => select(product) orderBy(product.name desc)
  }
}

// it also extends Iterable
def findAll: Iterable[Product] = inTransaction {
  allQ.toList
}
~~~~~~

~~~~~~
def productsInWarehouse(warehouse: Warehouse) = {
  join(productsTable, stockItemsTable)((product, stockItem) =>
    where(stockItem.location === warehouse.id).
    select(product).
    on(stockItem.product === product.id)
  )
}
def productsInWarehouseByName(name: String, warehouse: Warehouse): Query[Product]= {
  from(productsInWarehouse(warehouse)){ product =>
    where(product.name like name).select(product)
  }
}
~~~~~~

## Saving records ##

~~~~~~
def insert(product: Product): Product = inTransaction {
  productsTable.insert(product)
}

def update(product: Product) {
  inTransaction { productsTable.update(product) }
}
~~~~~~

如果不这样, 那么插入时会产生 原先的 id 会变
~~~~~~
def insert(product: Product): Product = inTransaction {
  val defensiveCopy = product.copy
  productsTable.insert(defensiveCopy)
}

val myImmutableObject = Product(0, 5010255079763l,
  "plastic coated blue",
  "standard paperclip, coated with blue plastic")
Database.productsTable.insert(myImmutableObject)
// id 会变成 xxx
println(myImmutableObject)
~~~~~~

## Handling transactions ##

In order to ensure your database’s data integrity, you’ll want to use transactions.
Databases that provide transactions guarantee that all write operations
in the same transaction will either succeed together or fail together.

Squeryl provides two methods for working with transactions: `transaction` and
`inTransaction`.

The difference is that transaction always makes its own transaction and
inTransaction only makes a transaction (and eventually commits) if it’s not already
in a transaction.

This means that because our DAO methods wrap everything in an
inTransaction, they themselves can be wrapped in a transaction and succeed or fail
together and never separately.

~~~~~~
import models.{ Database, Product, StockItem }
import org.squeryl.PrimitiveTypeMode.transaction
import Database.{productsTable, stockItemsTable}

def addNewProductGood(product: Product, stockItem: StockItem) {
  transaction {
    productsTable.insert(product)
    stockItemsTable.insert(stockItem)
  }
}

// 不推荐
def addNewProductBad(product: Product, stockItem: StockItem) {
  productsTable.insert(product)
  stockItemsTable.insert(stockItem)
}
~~~~~~

## Entity relations ##

~~~~~~
import org.squeryl.PrimitiveTypeMode._
import org.squeryl.dsl.{OneToMany, ManyToOne}
import org.squeryl.{Query, Schema, KeyedEntity, Table}

object Database extends Schema {
  val productsTable = table[Product]("products")
  val warehousesTable = table[Warehouse]("warehouses")
  val stockItemsTable = table[StockItem]("stockItems")

  val productToStockItems =
    oneToManyRelation(productsTable, stockItemsTable).
      via((p,s) => p.id === s.productId)

  val warehouseToStockItems =
    oneToManyRelation(warehousesTable, stockItemsTable).
      via((w,s) => w.id === s.warehouseId)
}


case class Product(
  id: Long,
  ean: Long,
  name: String,
  description: String) extends KeyedEntity[Long] {
  lazy val stockItems: OneToMany[StockItem] =
    Database.productToStockItems.left(this)
}

case class Warehouse(
  id: Long,
  name: String) extends KeyedEntity[Long] {
  lazy val stockItems: OneToMany[StockItem] =
    Database.warehouseToStockItems.left(this)
}

case class StockItem(
  id: Long,
  productId: Long,
  warehouseId: Long,
  quantity: Long) extends KeyedEntity[Long] {
  lazy val product: ManyToOne[Product] =
    Database.productToStockItems.right(this)
  lazy val warehouse: ManyToOne[Warehouse] =
    Database.warehouseToStockItems.right(this)
}
~~~~~~

~~~~~~
def getStockItems(product: Product) =
  inTransaction {
    product.stockItems.toList
  }

def getLargeStockQ(product: Product, quantity: Long) =
  from(product.stockItems) ( s =>
    where(s.quantity gt quantity)
      select(s)
  )

~~~~~~

Obviously, you need to be able to add stock items to products and warehouses. You
could set the foreign keys in each stock item by hand, which is simple enough, but
Squeryl offers some help here.


~~~~~~
// 建立外键联系
product.stockItems.assign(stockItem)
warehouse.stockItems.assign(stockItem)
transaction { Database.stockItemsTable.insert(stockItem) }

// The difference between assign and associate is
// that associate also saves the stock item
transaction {
  product.stockItems.associate(stockItem)
  warehouse.stockItems.associate(stockItem)
}
~~~~~~

### STATEFUL RELATIONS ###

Instead of providing queries, Squeryl’s stateful relations
provide collections of related entities that you can access directly.

You only need to change the call to
left to leftStateful and similarly right to rightStateful:
~~~~~~
lazy val stockItems =
  Database.productToStockItems.leftStateful(this)
~~~~~~

You’ll have problems instantiating entities out-
side of a transaction.

`StatefulOneToMany` has an associate method that does the same thing as its non-
stateful counterpart, but it doesn’t have an assign method.

Apart from that, there’s a refresh method that refreshes the list from the database. 

# Caching data #

Any database worth its salt will cache results for queries it encounters often. But
you’re still dealing with the overhead of talking to the database, and there are usually
more queries hitting the database, which may push these results out of the cache or
invalidate them eagerly. 

An application cache can be more useful than a database cache, because it knows
what it’s doing with the data and can make informed decisions about when to invali-
date what.

~~~~~~
def insert(product: Product) {
  val insertedProduct = Product.insert(product)
  Cache.set("product-" + product.id, product)
}

def show(productId: Long) {
  Cache.getAs[Product]("product-" + productId) match {
    case Some(product) => Ok(product)
    case None => Ok(Product.findById(productId))
  }
}
~~~~~~
