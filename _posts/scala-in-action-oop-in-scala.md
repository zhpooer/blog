title: Scala in Action-OOP in Scala
date: 2014-07-12 21:56:24
tags:
- scala
---

# 一些特性 #

## `_root_`的使用 ##
~~~~~~
package monads { class IOMonad }
package io {
  package monads {
    class Console { val m = new _root_.monads.IOMonad }
  }
}

import java.util.Date
import java.sql.{Date => SqlDate}
// 也可以这样
import java.sql.{Date => _ }
~~~~~~

## bean方法 ##

~~~~~~
// 如果在构造函数中没有定义 var 或者 val, 将会是似有的
class Person(var firstName:String, var lastName:String,
        private var _age:Int) {
  def age = _age
  def age_=(newAge: Int) = _age = newAge
}

~~~~~~

## object 的工厂方法 ##
~~~~~~
abstract class Role { def canAccess(page: String): Boolean }
class Root extends Role {
  override def canAccess(page:String) = true
}
class SuperAnalyst extends Role {
  override def canAccess(page:String) = page != "Admin"
}
class Analyst extends Role {
  override def canAccess(page:String) = false
}
object Role {
  def apply(roleName:String) = roleName match {
    case "root" => new Root
    case "superAnalyst" => new SuperAnalyst
    case "analyst" => new Analyst
  }
}

~~~~~~

## package object ##
~~~~~~
// 一般放在 package.scala文件下
// minimumAge and verifyAge will be available to
// all members of the package bar
package object bar {
  val minimumAge = 18
  def verifyAge = {}
}

// 用法如下
package bar
class BarTender {
  def serveDrinks = { verifyAge; ... }
}
~~~~~~

## trait ##

In OOP languages, a mixin is a class that provides certain functionality
that could be used by other classes.

Another difference between traits and abstract classes in Scala is that an
abstract class can have constructor parameters, but traits can’t take any param-
eters. Both can take type parameters.

The difference between def and val is that val gets evaluated when an
object is created, but def is evaluated every time a method is called.

## case class ##

when you prefix a class with case, the following things will hap-
pen automatically:
* Scala prefixes all the parameters with val, and that will make them public value.
But remember that you still never access the value directly; you always access
through accessors.
* Both equals and hashCode are implemented for you based on the given
parameters.
* The compiler implements the toString method that returns the class name
and its parameters.
* Every case class has a method named copy that allows you to easily create a mod-
ified copy of the class’s instance. You’ll learn about this later in this chapter.
* A companion object is created with the appropriate apply method, which takes
the same arguments as declared in the class.
* The compiler adds a method called unapply, which allows the class name to be
used as an extractor for pattern matching (more on this later).
*  A default implementation is provided for serialization:
~~~~~~
scala> val me = Person("Nilanjan", "Raychaudhuri")
me: Person = Person(Nilanjan,Raychaudhuri)

scala> val myself = Person("Nilanjan", "Raychaudhuri")
myself: Person = Person(Nilanjan,Raychaudhuri)

scala> me.equals(myself)
res1: Boolean = true

scala> me.hashCode
res2: Int = 1688656232

scala> myself.hashCode
res4: Int = 1688656232
~~~~~~

## copy constructor ##
~~~~~~
scala> val skipOption = Skip(10, NoOption)
  skipOption: Skip = Skip(10,NoOption())
  
scala> val skipWithLimit = skipOption.copy(anotherOption = Limit(10, NoOption))
  skipWithLimit: Skip = Skip(10,Limit(10,NoOption))

// 内部结构如下
case class Skip(number: Int, anotherOption: QueryOption)
    extends QueryOption {
  def copy(number: Int = number,
  anotherOption: QueryOption = anotherOption) = {
    Skip(number, anotherOption)
  }
}

~~~~~~

## abstract override ##

~~~~~~
trait DogMood {
  def greet
}

trait AngryMood extends DogMood {
  // 父方法是抽象方法, 错误
  override def greet = {
    println("bark")
    super.greet
  }
}
// 但是可以这样
//  abstract override, which means it should be mixed in
// with some class that has the concrete definition of the greet method.
trait AngryMood extends DogMood {
 abstract override def greet = {
   println("bark")
   super.greet
 }
}

~~~~~~

## sealed ##
classes marked final in Scala
can’t be overridden by subclasses. But classes marked sealed can be overridden as
long as the subclasses belong to the same source file.


# Value Class #
Scala allows user-defined value classes (which could be case
classes as well) that extend AnyVal. Value classes are a new mechanism to avoid run-time allocation of the objects.
To create a value class you need to abide by some
important rules, including:
* The class must have exactly one val parameter (vars are not allowed).
* The parameter type may not be a value class.
* The class can not have any auxiliary constructors.
* The class can only have def members, no vals or vars.
* The class cannot extend any traits, only universal traits.

Value classes allow you to add extension
methods to a type without the runtime overhead of creating instances.

~~~~~~
class Wrapper(val name: String) extends AnyVal {
  def up() = name.toUpperCase
}

val w = new Wrapper("hey")
w.up()

// 他被编译成这样
object Wrapper {
  def up$extension(_name: String) = _name.toUpperCase
}

~~~~~~

A value class can only extend a universal trait, one that extends Any (normally traits by
default extend AnyRef). Universal traits can only have def members and no initializa-
tion code:
~~~~~~
trait Printable extends Any {
  def p() = println(this)
}
case class Wrapper(val name: String) extends AnyVal with Printable {
  def up() = name.toUpperCase
}
...
val w = Wrapper("Hey")
w.p()

~~~~~~

Even though now you can invoke the p method on a Wrapper instance at runtime an
instance will also be created because the implementation of the p method prints the
type. There are limitations when allocation is necessary; if you assign a value class to
an array, the optimization will fail.

# implicit #

~~~~~~
implicit def double2Int(d: Double): Int = d.toInt

val someInt: Int = 2.3

-- rewritten by the compiler:
val someInt: Int = double2Int(2.3)

-- we can avoid the runtime cost by turning our implicit classes
-- into value classes
implicit class RangeMaker(val left: Int) extends AnyVal {
  def -->(right: Int): Range = left to right
}
~~~~~~

# DEMO: Mongo #

## 1 ##

As a developer, I want an easier way to connect to my MongoDB server and access
document databases.

As a developer, I want to query and manage documents.

~~~~~~
package com.scalainaction.mongo

class MongoClient(val host:String, val port:Int) {
  require(host != null, "You have to provide a host name")
  private val underlying = new Mongo(host, port)
  def this() = this("127.0.0.1", 27017)
  def version = underlying.getVersion
  def dropDB(name:String) = underlying.dropDatabase(name)
  def createDB(name:String) = DB(underlying.getDB(name))
  def db(name:String) = DB(underlying.getDB(name))
}

package com.scalainaction.mongo
import com.mongodb.{DB => MongoDB}
// 私有构造器, 除了伴生对象能够访问
class DB private(val underlying: MongoDB) {
  def collectionNames = for(name <- new
    JSetWrapper(underlying.getCollectionNames)) yield name

}
object DB {
  def apply(underlying: MongDB) = new DB(underlying)
}

~~~~~~

case class 的 unapply
~~~~~~
object Person {
def apply(firstName:String, lastName:String) = {
  new Person(firstName, lastName)
}
def unapply(p:Person): Option[(String, String)] =
  Some((p.firstName, p.lastName))
}
~~~~~~

## 2 ##

The second user story you need to implement in your driver is an ability to create,
delete, and find documents in a MongoDB database.

~~~~~~

package com.scalainaction.mongo
import com.mongodb.{DBCollection => MongoDBColleciton}
import com.mongodb.DBObject

class DBCollection(override val underlying: MongoDBCollection) extends ReadOnly

trait ReadOnly {
  val underlying:MongoDBCollection
  def name = underlying getName
  def fullName = underlying getFullName
  def find(doc: DBObject) = underlying find doc
  def findOne(doc: DBObject) = underlying findOne doc
  def findOne = underlying findOne
  def getCount(doc: DBObject) = underlying getCount doc
}

trait Administrable extends ReadOnly {
  def drop: Unit = underlying drop
  def dropIndexes: Unit = underlying dropIndexes
}

trait Updatable extends ReadOnly {
  def -=(doc: DBObject): Unit = underlying remove doc
  def +=(doc: DBObject): Unit = underlying save doc
}

trait Memoizer extends ReadOnly {
  val history = scala.collection.mutable.Map[Int, DBObject]()
  override def findOne = {
    history.getOrElseUpdate(-1, { super.findOne })
  }
  override def findOne(doc: DBObject) = {
    history.getOrElseUpdate(doc.hashCode, { super.findOne(doc) })
  }
}


import com.mongodb.{DB => MongoDB}
import scala.collection.convert.Wrappers._
class DB private(val underlying: MongoDB) {
  private def collection(name: String) = underlying.getCollection(name)
  def readOnlyCollection(name: String) =
      new DBCollection(collection(name)) with Memoizer
  def administrableCollection(name: String) =
      new DBCollection(collection(name)) with Administrable with Memoizer
  def updatableCollection(name: String) =
      new DBCollection(collection(name)) with Updatable with Memoizer
  def collectionNames = for(name <- new
    JSetWrapper(underlying.getCollectionNames)) yield name
}
object DB {
  def apply(underlying: MongoDB) = new DB(underlying)
}
~~~~~~

## 3 ##
~~~~~~
sealed trait QueryOption
case object NoOption extends QueryOption
case class Sort(sorting: DBObject, anotherOption: QueryOption)
    extends QueryOption
case class Skip(number: Int, anotherOption: QueryOption)
    extends QueryOption
case class Limit(limit: Int, anotherOption: QueryOption)
    extends QueryOption

case class Query(q: DBObject, option: QueryOption = NoOption) {
  def sort(sorting: DBObject) = Query(q, Sort(sorting, option))
  def skip(skip: Int) = Query(q, Skip(skip, option))
  def limit(limit: Int) = Query(q, Limit(limit, option))
}

trait ReadOnly {
  val underlying: MongoDBCollection
  def name = underlying getName
  def fullName = underlying getFullName
  def find(query: Query): DBCursor = {
    def applyOptions(cursor:DBCursor, option: QueryOption): DBCursor = {
      option match {
        case Skip(skip, next) => applyOptions(cursor.skip(skip), next)
        case Sort(sorting, next)=> applyOptions(cursor.sort(sorting), next)
        case Limit(limit, next) => applyOptions(cursor.limit(limit), next)
        case NoOption => cursor
      }
    }
  applyOptions(find(query.q), query.option)
}
  def find(doc: DBObject): DBCursor = underlying find doc
  def findOne(doc: DBObject) = underlying findOne doc
  def findOne = underlying findOne
  def getCount(doc: DBObject) = underlying getCount doc
}

~~~~~~

