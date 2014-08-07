title: Scala in Action-type system
date: 2014-07-19 16:46:35
tags:
- scala
---

# 入门 #
It provides the following features:
* *Error detection*—Think of the compiler as a suite of test cases that can detect
common type and other program errors.
* *Abstractions*—This is the focus of this chapter. You’ll learn how the type system
provides abstractions to build components.
* *Documentation*—The signature of a function or method tells you a lot about
what it’s doing.
* *Efficiency*—The type system helps the compiler generate optimized binary code.

scala 提供的抽象工具

| Technique | Description |
|---------------------|
| Modular mixin |   a mechanism for composing traits for designing reusable components without the problems of multiple inheritance.| 
| Abstract type | Scala lets you declare abstract type members to class, trait, and subclasses that can provide concrete types. | 
| Self type     | A minin use fields or methods of the class it’s mixed into |

# Abstract type members #

Abstract types are those whose identity is
unknown at the time of declaration.
Unlike concrete types, the type of an abstract type
member is specified during the concrete implementation
of the enclosing class.
~~~~~~
trait Calculator { type S }

class SomeCalculator extends Calculator { type S = String }
~~~~~~

The benefit of abstract type members is they can hide the
internal information of a component. 

You’re going to build a price
calculator that can take a product ID and return the price of the product. There can
be many ways to calculate the price, and each way could use a different type of data
source to retrieve the price.

~~~~~~
trait Calculator {
  def initialize: DbConnection
  def close(s: DbConnection): Unit
  def calculate(productId: String): Double = {
    val s = initialize
    val price = calculate(s, productId)
    close(s)
    price
  }
  def calculate(s: DbConnection, productId: String): Double
~~~~~~
The problem with the current implementation is that it’s hard-wired to a DAO,
and a calculator that uses a different kind of data source
won’t be able to use the calculator.
~~~~~~
// 模板方法
trait Calculator {
  type S
  def initialize: S
  def close(s: S): Unit
  def calculate(productId: String): Double = {
    val s = initialize
    val price = calculate(s, productId)
    close(s)
    price
  }
  def calculate(s: S, productId: String): Double
}

~~~~~~

# Self type members #

The self type annotation allows you to access members of a mixin trait or class, and
the Scala compiler ensures that all the dependencies are correctly wired before you’re
allowed to instantiate the class. 

~~~~~~
trait B {
  def b:Unit = ...
}
//  trait A is defining a dependency to trait B
trait A {  self: B =>
  def a:Unit = b
}
~~~~~~

Because the required services are annotated with self type this , you can still access
those services, and the Scala compiler will ensure that the final class gets mixed with a
trait or a class that implements RequiredServices. 
~~~~~~
trait Connection {
  def query(q: String): String
}
trait Logger {
  def log(l: String): Unit
}
trait RequiredServices {
  def makeDatabaseConnection: Connection
  def logger: Logger
}

trait ProductFinder { this: RequiredServices =>
  def findProduct(productId: String) = {
    val c = makeDatabaseConnection
    c.query("find the lowest price")
    logger.log("querying database...")
  }
}
// 有点像spring的简化
trait TestServices extends RequiredServices {
  def makeDatabaseConnection =
    new Connection { def query(q: String) = "test" }
  def logger = new Logger { def log(l: String) = println(l) }
}

object FinderSystem extends ProductFinder with TestServices
~~~~~~

# Building a scalable component #
To build a generic product ordering system. It will be reusable in that a user can
order any kind of product.

* An order component that represents the order placed by the customer.
* An inventory component that stores the products. You need to check the
inventory to make sure you have the product before you place the order.
* A shipping component that knows how to ship an order to customer.

~~~~~~
trait OrderingSystem {
  type O <: Order
  type I <: Inventory
  type S <: Shipping
  trait Order { def placeOrder (i: I): Unit }
  trait Inventory { def itemExists(order: O): Boolean }
  trait Shipping {def scheduleShipping(order: O): Long }
~~~~~~

You still need to implement the steps for placing the order. Here they are:
* Check whether that item exists in inventory.
* Place the order against the inventory. (Inventory will reduce the count by the
amount of product in the inventory.)
* Schedule the order for shipping.
* If the item doesn’t exist in inventory, return without placing the order and
possibly notify Inventory to replenish the product.

~~~~~~
// declared in OrderingSystem
// Self type annotation for two mixin traits
trait Ordering { this: I with S =>
  def placeOrder(o: O): Option[Long] = {
    if(itemExists(o)) {
      o.placeOrder (this)
      Some(scheduleShipping(o))
    }
    else None
  }
}
~~~~~~

to use it
~~~~~~
object BookOrderingSystem extends OrderingSystem {
  type O = BookOrder
  type I = AmazonBookStore
  type S = UPS
  class BookOrder extends Order {
    def placeOrder(i: AmazonBookStore): Unit = ...
  }
  trait AmazonBookStore extends Inventory {
    def itemExists(o: BookOrder) = ...
  }
  trait UPS extends Shipping {
    def scheduleShipping(order: BookOrder): Long = ...
  }
  object BookOrdering extends Ordering with AmazonBookStore with UPS
}

import BookOrderingSystem._
BookOrdering.placeOrder(new BookOrder)
~~~~~~

# Building an extensible component #

## THE EXPRESSION PROBLEM AND THE EXTENSIBILITY CHALLENGE ##

The goal is to define a data type and operations on that data type in which one can
add new data types and operations without recompiling existing code, but while
retaining static type safety.

Any implementation of the expression problem should satisfy the following
requirements:
* Extensibility in both dimensions. You should be able to add new types
and operations that work on all the types.
(I look into this in detail in this section.)
* Strong static type-safety. Type casting and reflection are out of the question.
* No modification of the existing code and no duplication.
* Separate compilation.

## practice example ##

You have a payroll system that
processes salaries for full-time employees in the United States and Canada:

~~~~~~
case class Employee(name: String, id: Long)

trait Payroll {
  def processEmployees(employees: Vector[Employee]): Either[String, Throwable]
}

class USPayroll extends Payroll {
  def processEmployees(employees: Vector[Employee]) = ...
}

class CanadaPayroll extends Payroll {
  def processEmployees(employees: Vector[Employee]) = ...
}

// With current changes in the business, you also have to process salaries of full-time
// employees in Japan. 
class JapanPayroll extends Payroll {
  def processEmployees(employees: Vector[Employee]) = ...
}

~~~~~~

This is one type of extension the expression problem talks about.
The solution is type-safe, and you can add JapanPayroll as an extension
and plug it in to an existing payroll system with a separate compilation.

What happens when you try to add a new operation?

~~~~~~
// In this case the business has decided to hire contractors,
// and you also have to process their monthly pay.
case class Employee(name: String, id: Long)
case class Contractor(name: String)
//  The new Payroll interface should look like the following:
trait Payroll extends super.Payroll {
  def processEmployees(
      employees: Vector[Employee]): Either[String, Throwable]
  def processContractors(
      contractors: Vector[Contractor]): Either[String, Throwable]
}
~~~~~~

The problem is you can’t go back and modify the trait because that will force you to
rebuild everything—which you can’t do because of the constraint put on you by the
expression problem.

Using the Visitor pattern to solve this problem.

~~~~~~
case class USPayroll {
  def accept(v: PayrollVisitor) = v.visit(this)
}
case class CanadaPayroll {
  def accept(v: PayrollVisitor) = v.visit(this)
}
trait PayrollVisitor {
  def visit(payroll: USPayroll): Either[String, Throwable]
  def visit(payroll: CanadaPayroll): Either[String, Throwable]
}
class EmployeePayrollVisitor extends PayrollVisitor {
  def visit(payroll: USPayroll): Either[String, Throwable] = ...
  def visit(payroll: CanadaPayroll): Either[String, Throwable] = ...
}

// you can easily create a new class called ContractorPayrollVisitor
class ContractorPayrollVisitor extends PayrollVisitor {
  def visit(payroll: USPayroll): Either[String, Throwable] = ...
  def visit(payroll: CanadaPayroll): Either[String, Throwable] = ...
}
// Using the Visitor pattern, it’s easy to add new operations,
// but what about type

~~~~~~

If you try to add a new type called JapanPayroll, you have a problem.
You have to go back and change all the visitors
to accept a JapanPayroll type.

## SOLVING THE EXPRESSION PROBLEM ##

~~~~~~
trait PayrollSystem {
  case class Employee(name: String, id: Long)
  type P <: Payroll
  trait Payroll {
    def processEmployees(
      employees: Vector[Employee]): Either[String, Throwable]
  }
  def processPayroll(p: P): Either[String, Throwable]
}

trait USPayrollSystem extends PayrollSystem {
  class USPayroll extends Payroll {
    def processEmployees(employees: Vector[Employee]) =
      Left("US payroll")
  }
}

trait CanadaPayrollSystem extends PayrollSystem {
  class CanadaPayroll extends Payroll {
    def processEmployees(employees: Vector[Employee]) =
      Left("Canada payroll")
  }
}

object USPayrollInstance extends USPayrollSystem {
  type P = USPayroll
  def processPayroll(p: USPayroll) = {
    val employees: Vector[Employee] = ...
    val result = p.processEmployees(employees)
    ...
  }
}

// add new Payroll Type
trait JapanPayrollSystem extends PayrollSystem {
  class JapanPayroll extends Payroll {
    def processEmployees(employees: Vector[Employee]) = ...
  }
}

// Now add a new method to the Payroll trait
// without recompiling everything
trait ContractorPayrollSystem extends PayrollSystem {
  type P <: Payroll
  case class Contractor(name: String)
  trait Payroll extends super.Payroll {
    def processContractors(
        contractors: Vector[Contractor]): Either[String, Throwable]
  }
}

trait USContractorPayrollSystem extends USPayrollSystem with
    ContractorPayrollSystem {
  class USPayroll extends super.USPayroll with Payroll {
    def processContractors(contractors: Vector[Contractor]) =
      Left("US contract payroll")
  }
}

// use it
object RunNewPayroll {
  object USNewPayrollInstance extends USContractorPayrollSystem {
    type P = USPayroll
    def processPayroll(p: USPayroll) = {
      p.processEmployees(Vector(Employee("a", 1)))
      p.processContractors(Vector(Contractor("b")))
      Left("payroll processed successfully")
    }
  }
  def main(args: Array[String]): Unit = run
  
  def run = {
    val usPayroll = new USPayroll
    USNewPayrollInstance.processPayroll(usPayroll)
  }
}

~~~~~~

# Types of types #

## Structural types ##

Structural typing in Scala is the way to describe types by their structure, not by their
names, as with other typing.

~~~~~~
def close(closable: { def close: Unit }) = {
  closable.close
}

// used like this
type Closable = { def close: Unit }
def close(closable: { def close: Unit }) = {
  closable.close
}

def amountPaidAsSalary2(workers: Vector[{def salary: BigDecimal }]) = {
}

~~~~~~

## Higher-kinded types ##
Higher-kinded types are types that know how to create a
new type from the type argument.

The `scala.collections.immutable.List[+A]` is
an example of a higher-kinded type. It takes a type
parameter and creates a new concrete type.
import `scala.language.higherKinds` to enable it

## Phantom types ##
Phantom types are types that don’t provide any constructors to create values. You only
need these types during compile time to enforce constraints. 
~~~~~~
case class Order(itemId: Option[Item], address: Option[String])

def addItem(item: String, o: Order) =
  Order (Some(item), o.shippingAddress)
def addShipping(address: String, o: Order) =
  Order (o.itemId, Some(address))
def placeOrder (o: Order) = { ... }

~~~~~~

The problem with this approach is that the methods could get called out of order. 

~~~~~~
sealed trait ItemProvided
sealed trait NoItem
sealed trait AddressProvided
sealed trait NoAddress

case class Order[A, B, C](itemId: Option[String], shippingAddress: Option[String])

def emptyOrder = Order[InCompleteOrder, NoItem, NoAddress](None, None)

def addItem[A, B](item: String, o: Order[A, NoItem, B]) =
  o.copy[A, ItemProvided, B](itemId = Some(item))

def addShipping[A, B](address: String, o: Order[A, B, NoAddress]) =
  o.copy[A, B, AddressProvided](shippingAddress = Some(address))

def placeOrder (o: Order[InCompleteOrder, ItemProvided, AddressProvided]) ={
  ...
  o.copy[OrderCompleted, ItemProvided, AddressProvided]()
}

~~~~~~

# Ad hoc polymorphism with type classes #
Ad hoc polymorphism is a kind of polymorphism in which
polymorphic functions can be applied to arguments of different types.
Ad hoc polymorphism lets you add features to a type any time you want.

~~~~~~
trait XmlConverter[A] {
  def toXml(a: A): String
}

case class Movie(name: String, year: Int, rating: Double)

object Converters {
  implicit object MovieConverter extends XmlConverter[Movie] {
    def toXml(a: Movie) = <movie>
            <name>{a.name}</name>
            <year>{a.year}</year>
            <rating>{a.rating}</rating>
        </movie>.toString
  }
}

object Main {
  import Converters._
  def toXml[A](a: A)(implicit converter: XmlConverter[A]) =
      converter.toXml(a)
  def main(args: Array[String]) = {
    val p = Movie("Inception", 2010, 10)
    toXml(p)
  }
}

// 可以写成这样, scala2.8 之后的语法糖
def toXml[A: XmlConverter](a: A) =
   implicitly[XmlConverter[A]].toXml(a)

~~~~~~

## Solving the expression problem using type classes ##

~~~~~~
import scala.langage.higherkinds

object PayrollSystemWithTypeclass {
  case class Employee(name: String, id: Long)
  trait PayrollProcessor[C[_], A] {
    def processPayroll(payees: Seq[A]): Either[String, Throwable]
  }
  case class USPayroll[A](payees: Seq[A])
       (implicit processor: PayrollProcessor[USPayroll, A]) {
    def processPayroll = processor.processPayroll(payees)
  }
  case class CanadaPayroll[A](payees: Seq[A])
      (implicit processor: PayrollProcessor[CanadaPayroll, A]){
    def processPayroll = processor.processPayroll(payees)
  }
}

object PayrollProcessors {
  import PayrollSystemWithTypeclass._
  implicit object USPayrollProcessor extends PayrollProcessor[USPayroll, Employee] {
    def processPayroll(payees: Seq[Employee]) = Left("us employees are processed")
  }
  implicit object CanadaPayrollProcessor extends PayrollProcessor[CanadaPayroll, Employee] {
    def processPayroll(payees: Seq[Employee]) =
        Left("canada employees are processed")
  }
}
// 扩展他时

object PayrollSystemWithTypeclassExtension {
  import PayrollSystemWithTypeclass._
  case class JapanPayroll[A](payees: Vector[A])
        (implicit processor: PayrollProcessor[JapanPayroll, A]) {
    def processPayroll = processor.processPayroll(payees)
  }
  case class Contractor(name: String)
}

object PayrollProcessorsExtension {
  import PayrollSystemWithTypeclassExtension._
  import PayrollSystemWithTypeclass._
  implicit object JapanPayrollProcessor extends
        PayrollProcessor[JapanPayroll, Employee] {
    def processPayroll(payees: Seq[Employee]) =
         Left("japan employees are processed")
  }
}

implicit object USContractorPayrollProcessor
      extends PayrollProcessor[USPayroll, Contractor] {
  def processPayroll(payees: Seq[Contractor]) =
        Left("us contractors are processed")
}
// TODO

object RunNewPayroll {
  import PayrollSystemWithTypeclass._
  import PayrollProcessors._
  import PayrollSystemWithTypeclassExtension._
  import PayrollProcessorsExtension._
  def main(args: Array[String]): Unit = run
  def run = {
    val r1 = JapanPayroll(Vector(Employee("a", 1))).processPayroll
    val r2 = JapanPayroll(Vector(Contractor("a"))).processPayroll
  }
}

~~~~~~
