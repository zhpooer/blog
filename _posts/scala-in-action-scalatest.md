title: Scala in Action-ScalaTest
date: 2014-07-24 10:11:32
tags:
- scala
---

There are two kinds of automated tests: ones you write (the most common)
and ones you generate for your code.

If you’re a Java developer and have used JUnit before, using it to test your Scala
code is easy. Specs is a testing tool written in Scala for Scala and
provides more expressiveness in your tests.

Dependency injection is a design pattern used by developers to make their code
more testable.As a hybrid language, Scala provides a number of
abstraction techniques you can use to implement dependency injection.

Automated tests are tests that are recorded or prewritten and can be run by a machine
without any manual intervention.

In the agile software development process, teams don’t analyze and design the
application up front; they build it using an evolutionary design.

* What does evolving design have to do with automated testing?
* Why is evolving the design better than designing the application up front?

There are varied types of automated tests: specification-based, unit, integration,
functional, and regression, to name a few.

# Automated test generation using ScalaCheck #

In ScalaCheck, a property is a testable unit. To create a
new property in ScalaCheck, you have to make a statement that describes the behavior
you want to test.

~~~~~~
name := "ScalaCheckExample"

version := "1.0"

organization := "Scala in Action"

scalaVersion := "2.10.0"

resolvers ++= Seq(
  "Sonatype Snapshots" at "http://oss.sonatype.org/content/repositories/snapshots",
  "Sonatype Releases" at "http://oss.sonatype.org/content/repositories/releases"
)

libraryDependencies ++= Seq (
  "org.scalacheck" %% "scalacheck" % "1.10.0" % "test"
)

// append options passed to the Scala compiler
scalacOptions ++= Seq("-deprecation", "-unchecked", "-feature")
~~~~~~

~~~~~~
// The job of ScalaCheck would be to falsify this statement
// by generating random test data. 
val anyString = "some string value"
anyString.reverse.reverse == anyString

import org.scalacheck.Prop
Prop.forAll((a: String) => a.reverse.reverse == a)

package checks
import org.scalacheck._
// The org.scalacheck.Properties represents a collection of ScalaCheck
// properties, and SBT has built-in support for running Properties:
object StringSpecification extends Properties("String") {
  property("reverse of reverse gives you same string back") =
    Prop.forAll((a: String) => a.reverse.reverse == a)
}

~~~~~~

The ScalaCheck generators are responsible for generating test data, and the
org.scalacheck.Gen class represents them.

One in particular is quite important: the arbitrary generator. This is a special
generator that generates arbitrary values for any supported type.

## Working with ScalaCheck ##

1. Either will have value on either Left or Right, but not both at any point
in time.
2. fold on the Left should produce the value contained by Left.
3. fold on the Right should produce the value contained by Right.
4. swap returns the Left value to the Right and vice versa.
5. getOrElse on Left returns the value from Left or the given argument if this is
Right.
6. forAll on Right returns true if Left or returns the result of the application of
the given function to the Right value.

~~~~~~
import Gen._
import Arbitrary.arbitrary
//  creates a new instance of the Int type generator and
// maps it to create values for Left
val leftValueGenerator = arbitrary[Int].map(Left(_))
val rightValueGenerator = arbitrary[Int].map(Right(_))

// randomly generate instances of Left or Right.
// methods like oneOf or frequency , called combinators.
// They allow you to combine multiple generators.
implicit val eitherGenerator =
    oneOf(leftValueGenerator, rightValueGenerator)
    
// The generator you’ve defined here only generates Int values,
// but if you wanted to play with different types of values, you’d also
// define the generator like this:
implicit def arbitraryEither[X, Y](implicit xa: Arbitrary[X],
      ya: Arbitrary[Y]): Arbitrary[Either[X, Y]] =
  Arbitrary[Either[X, Y]]( oneOf(arbitrary[X].map(Left(_)), arbitrary[Y].map(Right(_))) )

// If you wanted to use leftValueGenerator 75% of the time compared
// to the rightValueGenerator, you could use Gen.frequency like this:
implicit val eitherGenerator =
    frequency((3, leftValueGenerator), (1, rightValueGenerator))

// 测试代码
property("isLeft or isRight not both") =
    Prop.forAll((e: Either[Int, Int]) => e.isLeft != e.isRight)

//  on the Left should produce the value contained by Left
property("left value") =
    Prop.forAll{(n: Int) => Left(n).fold(x => x, b => error("fail")) == n }

// fold on the Right should produce the value contained by Right
property("Right value") =
    Prop.forAll{(n: Int) => Right(n).fold(b => error("fail"), x => x) == n }

//def fold[X](fa: A => X, fb: B => X) = this match {
//  case Left(a) => fa(a)
//  case Right(b) => fb(b)
//}

// “swap returns the Left value to Right and vice versa”
property("swap values") = Prop.forAll{(e: Either[Int, Int]) => e match {
    case Left(a) => e.swap.right.get == a
    case Right(b) => e.swap.left.get == b
  }
}

property("getOrElse") =
  Prop.forAll{ (e: Either[Int, Int], or: Int) =>
    e.left.getOrElse(or) == (e match {
      case Left(a) => a
      case Right(_) => or
    })
  }

property("forall") = Prop.forAll {(e: Either[Int, Int]) =>
  e.right.forall(_ % 2 == 0) == (e.isLeft || e.right.get % 2 == 0)
}

~~~~~~

~~~~~~
// setting for the minimum successful (-s) tests from 100 to 500 by passing test arguments
> test-only -- -s 500
~~~~~~

# TDD #

Acceptance criteria:

    A 100 product code should use cost plus the percent amount.
    Example: 150 (cost) + 20% = $180
    All products whose ID starts with B should use an external price source to get
    the price.


* Where should you implement the pricing logic?
* Should you create a trait or start with a simple function?
* What parameters will the function take?
* How should you test the output?
* Should it hit the database or filesystem to pull up the cost?

The most common theme of TDD is to pick the simplest
solution that could possibly work. In this case the simplest solution would be to create
a function that takes a product code, looks it up in a Map, and returns the price using
the formula specified in the acceptance criterion.

Do the simplest thing that could possibly work, and
then incrementally design and build your application.

Once your test is running, you have the opportunity to refactor or clean up.
Refactoring (www.refactoring.com) is a technique you can use to improve the design of
existing code without changing the behavior.

Most popular: JUnit and Specs. JUnit is more
popular among Java developers and can be easily used to test Scala code.

## CI ##

* Jenkins CI
* Jenkins SBT plugins
* Code coverage

You can also generate a `.POM` file (Maven build file) from
your SBT project using the make-pom action.

## Using JUnit ##

~~~~~~
libraryDependencies += "junit" % "junit" % "4.10" % "test"

libraryDependencies += "com.novocode" % "junit-interface" % "0.8" % "test"
~~~~~~

# DI #
Dependency injection (DI) is a design pattern that separates
behavior from dependency resolution.

This example is about calculating the price of a product based on various pricing
rules. Typically any pricing system will have hundreds of rules, but to keep
things simple I will only talk about two:

* The cost-plus rule determines the price by adding a percentage of the cost.
* Getting the price from an external pricing source.

~~~~~~
sealed class CalculatePriceService {
  val costPlusCalculator = new CostPlusCalculator()
  val externalPriceSourceCalculator = new ExternalPriceSourceCalculator()
  val calculators = Map(
    "costPlus" -> calculate(costPlusCalculator) _ ,
    "externalPriceSource" -> calculate(externalPriceSourceCalculator) _)
    
  def calculate(priceType: String, productId: String): Double = {
    calculators(priceType)(productId)
  }
  
  private[this] def calculate(c: Calculator)(productId: String):Double =
    c.calculate(productId)
}
trait Calculator {
  def calculate(productId: String): Double
}

class CostPlusCalculator extends Calculator {
  def calculate(productId: String) = {}
}

class ExternalPriceSourceCalculator extends Calculator {
  def calculate(productId: String) = {}
}
~~~~~~

Dependency injection is a specific form of inversion of control where the
concern being inverted is the process of obtaining the needed dependencie

There are some potential problems with this,
in particular when your software is evolving.

Using DI, you can easily solve this problem. If the dependent calculators could be
passed in (injected) to the CalculatePriceService, then the service could be easily
configured with various implementations of calculators.
~~~~~~
// 通过构造器注入
sealed class CalculatePriceService(
    val costPlusCalculator: Calculator,
    val externalPriceSourceCalculator: Calculator) {
  TODO
}
~~~~~~

## Techniques to implement DI ##

A measure of a good unit test is that it should be free of side effects,
the same as writing a pure function in functional programming.

If you follow TDD as a driver for your design, you don’t have to worry too much about
the coupling problem—your tests will force you to come up with a decoupled design.
You’ll notice that your functions, classes, and methods follow a DI pattern.

Techniques to implement dependency injection

1. Cake pattern, Handles dependency using trait mixins and abstract members.
2. Structural typing, Uses structural typing to manage dependencies.
The Scala structural typing feature provides duck typinga in a type-safe manner.
3. Implicit parameters, Manages dependencies using implicit parameters so that
as a caller you don’t have to pass them.
In this case, dependencies could be easily controlled using scope.
4. Functional programming style, Uses function currying to control dependencies.
5. Using a DI framework



### Cake pattern ###
A cake pattern 13 is a technique to build multiple layers of indirection in your applica-
tion to help with managing dependencies. 

* Abstract members
* Self type
* Mixin Composition

~~~~~~
// The idea behind this Calculator trait is to have a component namespace that has all
// the calculators in your application.
trait Calculators {
  val costPlusCalculator: CostPlusCalculator
  val externalPriceSourceCalculator: ExternalPriceSourceCalculator
  trait Calculator {
    def calculate(productId: String): Double
  }
  class CostPlusCalculator extends Calculator {
    def calculate(productId: String) = {
      ...
    }
  }
  class ExternalPriceSourceCalculator extends Calculator {
    def calculate(productId: String) = {
      ...
    }
  }
}

// self type
// The benefit is that now you can reference both
// costPlusCalculator and externalPriceSourceCalculator freely.
trait CalculatePriceServiceComponent {this: Calculators =>
  class CalculatePriceService {
    val calculators = Map(
      "costPlus" -> calculate(costPlusCalculator) _
      "externalPriceSource" -> calculate(externalPriceSourceCalculator) _)
    
    def calculate(priceType: String, productId: String): Double = {
      calculators(priceType)(productId)
    }
    private[this] def calculate(c: Calculator)(productId: String):Double =
      c.calculate(productId)
  }
}
// Remember from the tests, you don’t want to use the calculators; instead you want to
// use a fake or TestDouble version of the calculators.

// For production mode you could create a pricing system by compos-
// ing all the real versions of these components, as in the following:
object PricingSystem extends CalculatePriceServiceComponent with Calculators {
  val costPlusCalculator = new CostPlusCalculator
  val externalPriceSourceCalculator = new ExternalPriceSourceCalculator
}

// or testing the pricing could be created using the fake implementation
trait TestPricingSystem extends CalculatePriceServiceComponent with Calculators {
  class StubCostPlusCalculator extends CostPlusCalculator {
    override def calculate(productId: String) = 0.0
  }
  
  class StubExternalPriceSourceCalculator extends ExternalPriceSourceCalculator {
    override def calculate(productId: String) = 0.0
  }
  val costPlusCalculator = new StubCostPlusCalculator
  val externalPriceSourceCalculator = new StubExternalPriceSourceCalculator
}
~~~~~~

~~~~~~
package scala.book.cakepatterntest {
  import junit.framework.Assert._
  import org.junit.Test
  import cakepattern._
  class CalculatePriceServiceTest extends TestPricingSystem {
    @Test
    def shouldUseCostPlusCalculatorWhenPriceTypeIsCostPlus() {
      val calculatePriceService = new CalculatePriceService
      val price = calculatePriceService.calculate("costPlus","some product")
      assertEquals(5.0D, price)
    }
  }
}
~~~~~~
This is a common technique used by Scala developers to manage dependencies. In
smaller projects, it’s reasonable to have the wiring of dependencies implemented like
the PricingSystem and the TestPricingSystem, but for large projects it may become
difficult to manage them. For large projects it makes more sense to use a DI
framework that allows you to completely separate object creation
and injection from business logic.

### Structural typing ### 

~~~~~~
type Calculators = {
  val costPlusCalculator: Calculator
  val externalPriceSourceCalculator: Calculator
}

class CalculatePriceService(c: Calculators) {
  val calculators = Map(
    "costPlus" -> calculate(c.costPlusCalculator) _ ,
    "externalPriceSource" -> calculate(c.externalPriceSourceCalculator) _)
  def calculate(priceType: String, productId: String): Double = {
    calculators(priceType)(productId)
  }
  private[this] def calculate(c: Calculator)(productId: String):Double =
    c.calculate(productId)
}

~~~~~~
The advantage of structural typing in Scala is that it’s immutable and type-safe.
The Scala compiler will ensure that the constructor
parameter of CalculatePriceService implements both the
abstract vals costPlusCalculator and externalPriceSourceCalculator .

~~~~~~
object ProductionConfig {
  val costPlusCalculator = new CostPlusCalculator
  val externalPriceSourceCalculator = new ExternalPriceSourceCalculator
  val priceService = new CalculatePriceService(this)
}

object TestConfig {
  val costPlusCalculator = new CostPlusCalculator {
    override def calculate(productId: String) = 0.0
  }
  
  val externalPriceSourceCalculator = new ExternalPriceSourceCalculator {
    override def calculate(productId: String) = 0.0
  }
  val priceService = new CalculatePriceService(this)
}
~~~~~~
You have the flexibility to pick the appropriate configuration.
Internally, structural typing is implemented using
reflection, so it’s slower compared to other approaches. Sometimes that’s acceptable,
but be aware of it when using structural typing.

### implicit parameters ###

Implicit parameters provide a way to allow parameters to be found. Using this tech-
nique you can have the Scala compiler inject appropriate dependencies into your
code. 

~~~~~~
class CalculatePriceService(
  implicit val costPlusCalculator: CostPlusCalculator,
  implicit val externalPriceSourceCalculator: ExternalPriceSourceCalculator
)

object ProductionServices {
  implicit val costPlusCalculator = new CostPlusCalculator
  implicit val externalPriceSourceCalculator = new ExternalPriceSourceCalculator
}

object ProductionConfig {
  import ProductionServices._
  val priceService = new CalculatePriceService
}


object TestServices {
  implicit val costPlusCalculator = new CostPlusCalculator {
    override def calculate(productId: String) = 0.0
  }
  implicit val externalPriceSourceCalculator = new ExternalPriceSourceCalculator {
    override def calculate(productId: String) = 0.0
  }
}
object TestConfig {
  import TestServices._
  val priceService = new CalculatePriceService
}
~~~~~~
Using implicit to handle dependencies can easily get out of hand as your application grows
in size, unless they’re grouped together like the preceding configuration objects.

### Dependency injection in functional style ###

If you consider a function as a component, then its dependencies are its parameters.
If you create function currying, you can also
hide the dependencies as you did with other patterns.

~~~~~~
trait Calculators {
  //  type Calculator is an alias of function that takes product ID and returns the price
  type Calculator = String => Double
  protected val findCalculator: String => Calculator
  protected val calculate: (Calculator, String) => Double =
    (calculator, productId) => calculator(productId)
}

//  it created a function that takes Calculator and
// returns a function that calculates the price for a productid 
object TestCalculators extends Calculators {
  val costPlusCalculator: String => Double = productId => 0.0
  val externalPriceSource: String => Double = productId => 0.0
  override protected val findCalculator = Map(
    "costPlus" -> costPlusCalculator,
    "externalPriceSource" -> externalPriceSource
  )
  def priceCalculator(priceType: String): String => Double = {
    val f: Calculator => String => Double = calculate.curried
    f(findCalculator(priceType))
  }
}

// The benefit of doing this now is you have a function that knows
// how to calculate price but hides the Calculator from the users.
~~~~~~

The priceCalculator method returns a function that takes the productId and
returns the price of the product that encapsulates the dependencies used to compute
the price.

### Using a dependency injection framework: Spring ###
DI frameworks provide the following additional services
that aren’t available in these abstraction techniques above:

* They create a clean separation between object initialization and creation from
the business logic. This way, your wiring between components becomes
transparent from the code.
* These frameworks help you to work with various other frameworks.
A DI framework will help to inject your Scala objects as dependencies.
*  Most of the DI frameworks, like Spring (www.springsource.org) and Guice
provide aspect-oriented programming ( AOP) support to handle cross-cutting
behaviors like transaction and logging out of the box.

In the Spring world, all the dependencies are called beans, because all the objects
follow the JavaBean convention. According to this convention a class
should provide a default constructor, and class properties
should be accessible using get, set, and is methods.

~~~~~~
package scala.book
import scala.reflect._
sealed class CalculatePriceService {
  @BeanProperty var costPlusCalculator: Calculator = _1
  @BeanProperty var externalPriceSourceCalculator: Calculator = _
}
@RunWith(classOf[SpringJUnit4ClassRunner])
@ContextConfiguration(locations =
    Array("classpath:/application-context.xml"))
class CalculatePriceServiceTest {
  @Resource
  var calculatePriceService: CalculatePriceService = _
}

~~~~~~
In large projects it’s recommended to have a test
version of a configuration file where you can configure
all your beans with fake implementations of their dependencies.

# Behavior-driven development using Specs2 #
Behavior-driven development (BDD ) is about implementing an application by describ-
ing the behavior from the point of view of stakeholders.

BDD is doing TDD the right way. The first
thing to notice is that the definition of BDD doesn’t talk about testing at all.
And BDD puts more emphasis on solving business problems.
In fact, it recommends looking at the application from the stakeholder’s perspective.

* Delivering value quickly—Because you’re focused on viewing the application
from the stakeholder’s point of view, you understand and deliver value quickly.
* Focus on behavior—This is the most important improvement because at the end
of the day, behaviors that you implement are the ones your stakeholders want.

Think of a specification as a list of examples.

## Getting started with Specs2 ##

~~~~~~
scalaVersion := "2.10.0"

libraryDependencies += "org.specs2" %% "specs2" % "1.13" % "test"
~~~~~~

~~~~~~

trait TestPricingSystem extends CalculatePriceServiceComponent with Calculators {
  class StubCostPlusCalculator extends CostPlusCalculator {
    override def calculate(productId: String) = 5.0D
  }
  class StubExternalPriceSourceCalculator extends ExternalPriceSourceCalculator {
    override def calculate(productId: String) = 10.0D
  }
  val costPlusCalculator = new StubCostPlusCalculator
  val externalPriceSourceCalculator = new StubExternalPriceSourceCalculator
}

package scala.book

import org.specs2.mutable._

class CalculatePriceServiceSpecification extends Specification {
  "Calculate price service" should {
    "calculate price for cost plus price type" in {
      val service = new CalculatePriceService
      val price: Double = service.calculate("costPlus", "some product")
      price must beEqualTo(5.0D)
    }
    "calculate price for external price source type" in {
      val service = new CalculatePriceService
      val price: Double = service.calculate("externalPriceSource","some product")
      price must be_==(10.0D)
    }
  }

}
// The must method is again added by Specs using implicit conversions to
// almost all the types to make the specification more readable.

// nest 
"calculate price for cost plus price type" in {
  val service = new CalculatePriceService
  val price: Double = service.calculate("costPlus", "some product")
  price must beEqualTo(5.0D)
  "for empty product id return 0.0" in {
    val service = new CalculatePriceService
    service.calculate("costPlus", "") must beEqualTo(0.0D)
  }
}

~~~~~~

Another interesting way to declare specifications in Specs is to use data tables.
Data tables allow you to execute your example with a set of test data.

~~~~~~
"cost plus price is calculated using 'cost + 20% of cost + given service charge' rule" in {
  "cost" | "service charge" | "price" |>
  100.0 !  4    ! 124  |
  200.0 ! 4     ! 244  |
  0.0   ! 2     ! 2    | {
  (cost, serviceCharge, expected) =>
    applyCostPlusBusinessRule(cost, serviceCharge) must be_==(expected)
}

~~~~~~

# Testing asynchronous messaging systems #

Let’s see Awaitility work in a simple example.
Imagine that you have an order-placing service that saves orders to the
database asynchronously, and you place an order by sending a PlaceOrder message.
Here’s the dummy ordering service implemented as an actor:

~~~~~~
package example.actors
case class PlaceOrder(productId: String, quantity: Int, customerId: String)
class OrderingService extends Actor {
  def act = {
    react {
      case PlaceOrder(productId, quantity, customer) =>
    }
  }
}
~~~~~~

~~~~~~
import org.specs2.mutable._
import example.actors._
import com.jayway.awaitility.scala._
import com.jayway.awaitility.Awaitility._
class OrderServiceSpecification extends Specification with AwaitilitySupport {
  "Ordering system" should {
    "place order asynchronously" in {
      val s = new OrderingService().start
      s ! PlaceOrder("product id", 1, "some customer id")
      // waits until the order is saved into the database.
      // The default timeout for Awaitility is 10 seconds
      await until {orderSavedInDatabase("some customer id") }
      1 must_== 1
    }
    // Inside the orderSavedInDatabase, you could go to the data source and
    // check whether the order is saved for a given customer ID
    def orderSavedInDatabase(customerId: String) = ...
  }

}

~~~~~~
