title: Scala in Action-applications using Akka
date: 2014-07-26 07:35:04
tags:
- scala
---

# The philosophy behind Akka #

The philosophy behind Akka is simple:
make it easier for developers to build correct,
concurrent, scalable, and fault-tolerant applications.

At the core, Akka is an event-based platform and relies on actors for message
passing and scalability. Akka puts both local and remote actors at your disposal. 

# Simple concurrency with Akka #

* Actors, An actor is an object that processes messages asynchronously and encapsulates
state.
* STM, Software transactional memory is a concurrency model analogous to database
transactions for controlling access to a shared state
* Agents, Agents provide abstraction over mutable data. They only allow you to mutate the
data through an asynchronous write action.
* Dataflow, This means that it behaves the same every
time you execute it. So if your problem deadlocks the first time,
it will always deadlock, helping you to debug the problem.

You can model an application using actors, handle muta-
ble state with STM or agents, and use dataflow concurrency to compose multiple
concurrent processes. 

## Remote actors ##

Akka remote actors allow you to deploy actors in remote machines and
send messages back and forth transparently.

The messages are automatically serialized using the Google protocol buffer.
Think of the Google protocol buffer as XML but smaller and faster, and Netty
as a non-blocking I/O (NIO) implementation, which allows Akka to efficiently use
threads for I/O operations.

~~~~~~
resolvers ++= Seq(
  "Akka Repo" at "http://akka.io/repository",
  "Typesafe Repo" at "http://repo.typesafe.com/typesafe/repo"
)
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % "2.1.0",
  "com.typesafe.akka" %% "akka-remote % "2.1.0"
)
~~~~~~

We will work with a list of URLs. The goal is to connect
to the URL and count all the words on the page. 

## Making mutable data safe with STM ##

STM is similar to database transactions, but is used for memory instead.

* Atomicity—This property states that all modifications should follow the “all or
nothing” rule. In STM, all the modification is done through an atomic transac-
tion, and if one change fails all the other changes are rolled back.
* Consistency—This property ensures that an STM transaction takes the system
from one consistent state to another.
* Isolation—This property requires that no other STM transaction sees partial
changes from other transactions.

It rolls back from exceptions and is composable.

In STM, state is defined as the value that an entity
with a specific identity has at a particular point.

A value is something that doesn’t change (it’s immutable).
And identity is a stable reference to a value at a given point in time.
The mutable part here is the identity,
which gets associated with a series of values.
And STM makes the mutation of reference from one value to another atomic.

~~~~~~
resolvers += ("Typesafe Repository" at "http://repo.typesafe.com/typesafe/releases/")
libraryDependencies ++= Seq(
  "org.scala-stm" %% "scala-stm" % "0.7",
  "org.specs2" %% "specs2" % "1.13" % "test"
)
~~~~~~

~~~~~~
// Refs are nothing but mutable references to values that you
// can share safely with multiple concurrent participants.
val ref1 = Ref(HashMap[String, Any](
  "service1" -> "10",
  "service2" -> "20",
  "service3" -> null))
val ref2 = Ref(HashMap[String, Int]())

def atomicInsert(key: String, value: Int) = atomic { implicit txn =>
  val oldMap = ref2.get
  val newMap = oldMap + ( key -> value)
  // using swap to replace the old value with the new value.
  ref2.swap(newMap)
}

// The transform method allows you to transform the value
// referenced by Ref by applying the given function
def atomicDelete(key: String): Option[Any] = atomic {
  val oldMap = ref1.get
  val value = oldMap.get(key)
  ref1.transform(_ - key)
  value
}

~~~~~~
To perform any operation on Ref you have to use the atomic
method defined in the STM package by passing an in-transaction parameter. The Scala
STM library creates the transaction object and grants the caller permission to perform
transactional reads and writes. Any refs you change in the closure will be done in an
STM transaction. 

The transaction parameter is marked as implicit so you
don’t have to pass it around.

~~~~~~
// wrap both the atomicDelete and atomicInsert functions in an atomic function

def atomicSwap(key: String) = atomic { implicit txn =>
  val value: Option[Any] = atomicDelete(key)
  atomicInsert(key, Integer.parseInt(value.get.toString))
}

~~~~~~

## Agents ##

Agents provide asynchronous changes to any individual storage location bound to it.
An agent only lets you mutate the location by applying an action.
Actions in this case are functions that asynchronously are applied to
the state of Agent and in which the
return value of the function becomes the new value of Agent .

Reading a value from Agent is synchronous and instantaneous.
The difference between Ref and Agent is that Ref
is a synchronous read and write; Agent is reactive.

Akka provides two methods: send and sendOff. The send
method uses the reactive thread pool allocated for agents,
and sendOff uses a dedicated thread, ideal for a long-running processes.

~~~~~~
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % "2.1.0",
  "com.typesafe.akka" %% "akka-agent" % "2.1.0",
  "org.specs2" %% "specs2" % "1.13" % "test"
)
~~~~~~

~~~~~~
// with a file writer that
// logs messages to the log file through send actions
import akka.agent.Agent
implicit val system = ActorSystem("agentExample")
val writer = new FileWriter("src/test/resources/log.txt")
val a = Agent(writer)
a.send { w => w.write("This is a log message"); w}
// Shut Agent down
a.close
writer.close

~~~~~~

Agent will be running until you invoke the close method.
An actor system is created for the agent because,
behind the scenes, agents are implemented using actors. If
you have to do more than logging to a file,
something that will take time, use the sendOff method:
~~~~~~
a.sendOff { someLongRunningProcess }
~~~~~~

Note that at any time, only one send action is invoked.
Even if actions are sent from multiple concurrent processes,
the actions will be executed in sequential order.

This is important because if you
have a side-effect action, like logging to a file,
you don’t want to do that with STM.
Why? Because if STM transactions fail,
they retry automatically, meaning your
sideeffecting operation is executed multiple times.

Agent is associated with data, and you send behavior to Agent
from outside, in the form of a function. In the case of actors,
the behavior is defined
inside the actor, and you send data in the form of a message.

## Dataflow ##
Dataflow concurrency is a deterministic concurrency model.
If you run it and it works,
it will always work without deadlock.
Alternatively, if it deadlocks the first time, it will
always deadlock. 

The dataflow concurrency allows you to
write sequential code that performs parallel operations.
The limitation is that your code should be completely side-effect free.

Dataflow is implemented in Akka using Scala’s delimited continuations compiler
plug-in. `build.sbt`
~~~~~~
autoCompilerPlugins := true

libraryDependencies <+= scalaVersion { v => compilerPlugin(
"org.scala-lang.plugins" % "continuations" % v) }

scalacOptions += "-P:continuations:enable"

libraryDependencies += "com.typesafe.akka." %% " akka-dataflow" % "2.1.0"

~~~~~~

A dataflow variable is like a single-assignment variable.
Once the value is bound, it won’t change,
and any subsequent attempt to bind a new value will be ignored.

~~~~~~
// dataflow variable
// Here Akka Promise is used to create a dataflow variable.
// A Promise is a read handle to a value that will
// be available at some point in the future.
val messageFromFuture = Promise[String]()

// Any dataflow operation is performed in the Future.flow block:
Future.flow {
  messageFromFuture()
}

~~~~~~
The preceding call will wait in a thread unless a value is bound to messageFromFuture.
Future.flow returns a Future so you can perform other operations without blocking
the main thread of execution. Think of a Future as a data structure to retrieve the
result of some concurrent operation. To assign a value to a dataflow variable, use the
`<<` method as in the following:

~~~~~~
Future.flow {
  messsageFromFuture << "Future looks very cool"
}
~~~~~~

Once a value is bound to a dataflow variable,
all the Futures that are waiting on the
value will be unblocked and able to continue with execution. 

~~~~~~
importimportimportakka.actor._
akka.dispatch._
Future.flow

object Main extends App {
  implicit val system = ActorSystem("dataflow")
  val messageFromFuture, rawMessage, parsedMessage = Promise[String]()
  flow {
    messageFromFuture << parsedMessage()
    println("z = " + messageFromFuture())
  }
  flow { rawMessage << "olleh" }
  flow { parsedMessage << toPresentFormat(rawMessage()) }
  def toPresentFormat (s: String) = s.reverse
}
~~~~~~

# Building a real-time pricing system: Akkaoogle #

You’ll build a large web-based product search site
called Akkaoogle (see figure 12.4). It will be similar to Google’s
product search application (www.google.com/products) except that,
instead of returning all products matching your criteria,
your application will only return the cheapest deal found on
the web.

It gets the product price from two types of vendors
that are offering the product.
You can pay money to Akkaoogle and become an
internal vendor.
In this case, the product information is stored in Akkaoogle,
and you pay a small service charge.
You can also sign up as external vendor,
in which case Akkaoogle makes a RESTful web
service call to fetch the price -- but the downside is
you pay a bigger service charge. 

When the user is looking for the cheapest deal,
Akkaoogle checks with all the vendors (internal and external)
and finds the lowest price for the user.

You have to find the cheapest deal in no more than
200 to 300 milliseconds.

## The high-level architecture of Akkaoogle ##

![akkaoogle](/img/akkaoogle.png)
* Request handler—This is an actor that handles HTTP requests from the user.
You’ll use an asynchronous HTTP library called Mist, provided by Akka, to
implement this actor.
* Search cheapest product—This is the main entry point to execute a search to find
the cheapest deal. This actor will search both internal and external vendors.
* Internal load balancer—This is a load-balancing actor that sends messages to
worker actors to find the cheapest product available in the internal database.
* External load balancer—This actor invokes all the external vendor services and
finds the cheapest price among them.
* Find product price and find vendor price —The worker actors do the work of finding
the price.
* Monitor—A simple monitor actor logs the failures that happen in external ven-
dor services.
* Data loader—An actor that loads data to the database. This could be used to
load product data for internal vendors.


##  Setting up the project for Akkaoogle ##


~~~~~~
// Build.scala
object H2TaskManager {
  var process: Option[Process] = None
  lazy val H2 = config("h2") extend(Compile)
  val startH2 = TaskKey[Unit]("start", "Starts H2 database")
  val startH2Task =
  startH2 in H2 <<= (fullClasspath in Compile) map { cp =>
      startDatabase(cp.map(_.data).map(_.getAbsolutePath()).filter(_.contains(
      "h2database")))}
  def startDatabase(paths: Seq[String]) = {
    process match {
      case None =>
        val cp = paths.mkString(System.getProperty("path.seperator"))
        val command = "java -cp " + cp + " org.h2.tools.Server"
        println("Starting Database with command: " + command)
        process = Some(Process(command).run())
        println("Database started ! ")
      case Some(_) =>
        println("H2 Database already started")
    }
  }
  val stopH2 = TaskKey[Unit]("stop", "Stops H2 database")
  val stopH2Task = stopH2 in H2 :={
    process match {
      case None => println("Database already stopped")
      case Some(_) =>
        println("Stopping database...")
        process.foreach{_.destroy()}
        process = None
        println("Database stopped...")
    }
  }
}

object AkkaoogleBuild extends Build with ConfigureScalaBuild {
  import H2TaskManager._
  lazy val root = project(id = "akkaoogle", base = file("."))
    .settings(startH2Task, stopH2Task)
    .settings(
      organization := "scalainaction",
      scalaVersion := "2.10.0",
      scalacOptions ++= Seq("-unchecked", "-deprecation"),
      resolvers +=
      "Typesafe Repo" at "http://repo.typesafe.com/typesafe/repo",
      parallelExecution in Test := false
    )
    .settings(
      libraryDependencies ++= Seq(
        "com.typesafe.akka" % "akka-actor" % "2.1.0",
        "com.typesafe.akka" % "akka-remote" % "2.1.0",
        "com.typesafe.akka" % "akka-agent" % "2.1.0",
        "org.specs2" %% "specs2" % "1.13" % "test",
        "com.h2database" % "h2" % "1.2.127",
        "org.squery1" % "squery1_2.10.0-RC5" % "0.9.5-5",
        "org.eclipse.jetty" % "jetty-distribution" % "8.0.0.M2")
    )
}
~~~~~~

I test drove most of the application,
but I won’t show you test cases here.
I encourage you to go through the test cases
in this chapter’s accompanying codebase.

## Implementing the domain models ##


~~~~~~
//  create a common trait called Model that extends the KeyedEntity trait
//  This trait provides an id field that acts as a primary key

implicit val transactionFailures: Table[TransactionFailure] = AkkaoogleSchema.transactionFailures
implicit val vendors: Table[ExternalVendor] = AkkaoogleSchema.vendors
implicit val products: Table[Product] = AkkaoogleSchema.products
trait Model[A] extends KeyedEntity[Long] { this: A =>
  val id: Long = 0
  def save(implicit table: Table[A]): Either[Throwable, String] = {
    tx {
      try {
        table.insert(this)
        Right("Domain object is saved successfully")
      } catch {
        case exception => Left(exception)
      }
    }
  }

}

class Product(val description: String,
              val vendorName: String,
              val basePrice: Double,
              val plusPercent: Double) extends Model[Product] {
  def calculatePrice = basePrice + (basePrice * plusPercent / 100)
}
              
class ExternalVendor(val name: String, val url: String)
      extends Model[ExternalVendor]

//  You’ll log (to the database) every time a call to
// an external vendor service fails 
class TransactionFailure(val vendorId: String,
        val message: String,
        val timestamp: Date) extends Model[TransactionFailure]

object TransactionFailure {
  def findAll = tx {
    from(transactionFailures)(s => select(s)) map(s => s)
  }
}

object Product {
  def findByDescription(description: String): Option[Product] =
    tx {
      products.where(p => p.description like description).headOption
    }
}

object ExternalVendor {
  def findAll = tx {
    from(vendors)(s => select(s)) map(s => s)
  }
}
~~~~~~

Schema
~~~~~~
package com.akkaoogle.db
import org.squeryl._
import org.squeryl.adapters._
import org.squeryl.PrimitiveTypeMode._
import java.sql.DriverManager
import com.akkaoogle.db.models._

object AkkaoogleSchema extends Schema {
  val products = table[Product]("PRODUCTS")
  val vendors = table[ExternalVendor]("VENDORS")
  val transactionFailures = table[TransactionFailure]("TRANSACTION_LOG")
  def init = {
    import org.squeryl.SessionFactory
    Class.forName("org.h2.Driver")
    if(SessionFactory.concreteFactory.isEmpty) {
      SessionFactory.concreteFactory = Some(()=>
        Session.create(
          DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "sa", ""),
          new H2Adapter))
    }
  }
  def tx[A](a: =>A): A = {
    init
    inTransaction(a)
  }
  def createSchema() {
    tx { drop ; create }
  }
}
~~~~~~

## Implementing the core with actors ##



~~~~~~
package com.akkaoogle.calculators
// deal on the web and track the availability of the external services for quality purposes
object messages {
  // represents a request triggered by a user looking for the cheapest deal.
  case class FindPrice(productDescription: String, quantity: Int)
  // The response of the FindPrice message
  case class LowestPrice(vendorName: String,
  case class LogTimeout(actorId: String, msg: String)
  //  The FindStats and Stats messages are used for administration purposes
  case class FindStats(actorId: String)
  case class Stats(actorId: String, timeouts: Int)
}
~~~~~~
The InternalPriceCalculator actor calculates the lowest price by looking
up the product by description, shown in the following listing.
~~~~~~
package com.akkaoogle.calculators
import messages._
import com.akkaoogle.db.models._
import akka.actor._
class InternalPriceCalculator extends Actor {
  def receive = {
    case FindPrice(productDescription, quantity) =>
      val price = calculatePrice(productDescription, quantity)
      sender ! price
  }
  def calculatePrice( productDescription: String, qty: Int): Option[LowestPrice] = {
    Product.findByDescription(productDescription) map { product =>
      Some(
        LowestPrice(product.vendorName,
                    product.description,
                    product.calculatePrice * qty) )
    } getOrElse Option.empty[LowestPrice]
  }
}
//  you can have many external vendors for your application, you can’t make
// the remote service calls sequentially
// set a timeout for each service so you can
// respond to the user within a reasonable time
class ExternalVendorProxyActor(val v: ExternalVendor) extends Actor {
  def receive = {
    var result: Option[LowestPrice] = Option.empty[LowestPrice]
    val f = Future({
      val params = "?pd=" + fp.productDescription + "&q=" + fp.quantity
      val price = Source.fromURL(v.url + params).mkString.toDouble
      Some(LowestPrice(v.name, fp.productDescription, price * fp.quantity))
    }) recover { case t => Option.empty[LowestPrice] }
    f pipeTo sender
  }
}

// You need the actor in the following listing to broadcast
// the FindPrice message to each proxy actor

// The ExternalPriceCalculator actor is created with references to ExternalVendorProxyActor.
// The FindPrice message is broadcast to all the proxy actors using the ? method
class ExternalPriceCalculator(val proxies: Iterable[ActorRef])
      extends Actor {
  def receive = {
    case FindPrice(productId, quantity) =>
      val futures = proxies map { proxy =>
        val fp = FindPrice(productId, quantity)
        
        (proxy ? fp).mapTo[Option[LowestPrice]] recover {
          case e: AskTimeoutException =>
            AkkaoogleActorServer.lookup("monitor") ! LogTimeout(proxy.path.name, "Timeout for " + fp)
            Option.empty[LowestPrice]
          }
      }
      val lowestPrice: Future[Option[LowestPrice]] =
        findLowestPrice(futures)
      val totalPrice: Future[Option[LowestPrice]] = lowestPrice.map {
        l => l.map(p => p.copy(price = p.price + (p.price * .02)))
      }
      totalPrice pipeTo sender
  }
}  

def findLowestPrice(futures: Iterable[Future[Option[LowestPrice]]]):
    Future[Option[LowestPrice]] = {
  val f: Future[Option[LowestPrice]] = Future.fold(futures)(Option.empty[LowestPrice]) {
    (lowestPrice: Option[LowestPrice], currentPrice: Option[LowestPrice]) => {
      currentPrice match {
        case Some(first) if (lowsetPrice.isEmpty) => Some(first)
        case Some(c) if (c.price < lowestPrice.get.price) => Some(c)
        case _ => lowestPrice
      }
    }
  }
  f
}

// find the lowest price from both internal and external vendors and return the result.
class CheapestDealFinder extends Actor {
  def receive = {
    case req: FindPrice =>
      val internalPrice =
        (AkkaoogleActorServer.lookup("internal-load-balancer") ? req).mapTo[Option[LowestPrice]]
      val externalPrice =
        (AkkaoogleActorServer.lookup("external-load-balancer") ?
          req).mapTo[Option[LowestPrice]] recover {
            case e: AskTimeoutException => Option.empty[LowestPrice]
        }
      val lowestPrice: Future[Option[LowestPrice]] = findLowestPrice(internalPrice :: externalPrice :: Nil)
      lowestPrice pipeTo sender
  }
}

~~~~~~

## Increase scalability with remote actors, dispatchers, and routers ##

Akka comes with special kinds of actors called routers, which can
effectively route messages between multiple instances of actors. The router actor acts
as a gateway to a collection of actors. You send a message to the router actor, and the
router actor forwards the message to one of the actors, based on some routing policy.

~~~~~~
//  the SmallestMailboxRouter router routes messages based on the mailbox.
// creates 10 instances of CheapestDealFinder actors and
// creates a SmallestMailboxRouter to route messages to them
val cheapestDealFinderLoadBalancer = system.actorOf(
  Props[CheapestDealFinder].withRouter(SmallestMailboxRouter(nrOfInstances = 10)),
  name = "cheapest-deal-finder-balancer")
~~~~~~

~~~~~~
val internalPriceCalculators: List[ActorRef] = createInternalPriceCalculators(10)

// RoundRobinRouter routes messages to actors in round-robin fashion. 
val internalLoadBalancer = system.actorOf(1
    Props[InternalPriceCalculator]
  .withRouter(RoundRobinRouter (routees = internalPriceCalculators)),
  name = "internal-load-balancer")
  
val proxies = createExternalProxyActors(ExternalVendor.findAll)

val externalPriceCalculators: List[ActorRef] = createExternalPriceCalculators(10, proxies)

val externalLoadBalancer = system.actorOf(
    Props [ExternalPriceCalculator]
  .withRouter(RoundRobinRouter(routees = externalPriceCalculators)),
  name="external-load-balancer")

~~~~~~
Instead of allowing the router to create the actor instances,
the instances are passed as a parameter(they are called routees).

### IMPROVE PERFORMANCE WITH DISPATCHERS ###
Every actor system has a default dispatcher that’s used if nothing is configured.
In Akka, message dispatchers are the engine behind the actors that makes Actor run.

Think of a dispatcher as a service with a thread pool that knows how to execute actors
when a message is received.

But if you notice some contention on a single dispatcher, you can start creating
dedicated dispatchers for a group of actors. 

Remember, all the actors are created from the same actor system.
You can easily configure the default dispatcher by adding more threads to it

* Dispatcher—The default dispatcher used by the actor system. It’s an event-based
dispatcher that binds actors to a thread pool. It creates one mailbox per actor.
* PinnedDispatcher—Dedicates one thread per actor. It’s like creating thread-based actors.
* BalancingDispatcher—This event-driven dispatcher redistributes work from busy
actors to idle actors. All the actors of the same type share one mailbox.
* CallingThreadDispatcher—It runs the actor on the calling thread.
It doesn’t create any new thread. Great for unit testing purposes.

Using dispatchers in Akka is a simple two-step process: first,
specify them in the configuration file, then set up the actor with the dispatcher. 

~~~~~~
akkaoogle {
  dispatchers {
    external-price-calculator-actor-dispatcher {
      # Dispatcher is the name of the event-based dispatcher
      type = Dispatcher
      # What kind of ExecutionService to use
      executor = "fork-join-executor"
      # Configuration for the fork-join pool
      fork-join-executor {
        # Min number of threads to cap factor-based parallelism number to
        parallelism-min = 2
        # Parallelism (threads) ... ceil(available processors * factor)
        parallelism-factor = 2.0
        # Max number of threads to cap factor-based parallelism number t
        parallelism-max = 100
      }
      # Throughput defines the maximum number of messages to be
      # processed per actor before the thread jumps to the next actor.
      # Set to 1 for as fair as possible.
      throughput = 100
    }
    internal-price-calculator-actor-dispatcher {
      # Dispatcher is the name of the event-based dispatcher
      type = BalancingDispatcher
      # What kind of ExecutionService to use
      executor = "thread-pool-executor"
      thread-pool-executor {
        # Minnumber of threads to cap factor-based core number to
        core-pool-size-min = 5
      }
    }

  }
}
~~~~~~

To use these dispatchers you will use the withDispatcher method of Props, as in
the following:

~~~~~~
private def createInternalPriceCalculators(initialLoad: Int)
     (implicit system: ActorSystem) = {
  (for (i <- 0 until initialLoad) yield
    system.actorOf(Props[InternalPriceCalculator]
      .withDispatcher("dispatchers.internal-price-calculator-actor-dispatcher"),
    name=("internal-price-calculator" + i))).toList
}

private def createExternalPriceCalculators(initialLoad: Int,
     proxies: List[ActorRef])(implicit system: ActorSystem) = {
  (for (i <- 0 until initialLoad) yield system.actorOf(
    Props(new ExternalPriceCalculator(proxies))
      .withDispatcher("dispatchers.external-price-calculator-actor-dispatcher"),
    name = ("external-price-calculator" + i))).toList
}
~~~~~~

## Handling shared resources with Agent #

The monitor actor needs to log any transaction failure with external vendors. You
can always extend its functionality for internal use, but for now it needs to handle the
following two message types:

~~~~~~
case class LogTimeout(actorId: String, msg: String)
case class FindStats(actorId: String)
~~~~~~

To build the monitoring piece for the Akkaoogle application, you have to rely on a
shared mutable state, and this section shows you how to put Agent to use.

The monitor actor needs to log any transaction failure with external vendors.

~~~~~~
case class LogTimeout(actorId: String, msg: String)
case class FindStats(actorId: String)
~~~~~~

On receiving a LogTimeout message, it needs to save the transaction
failure information to the database and also keep track of
the number of times a particular service failed.

The side effect that’s saving information to the database
can’t be done safely within an STM transaction, because an STM transaction
could retry the operations in a transaction multiple times
if there’s any read/write inconsistency.
If you use Agent, it can participate in the STM transaction and
get executed only when the STM transaction completes successfully.

~~~~~~
package com.akkaoogle.infrastructure
import akka.agent.Agent
import akka.actor.Actor
import com.akkaoogle.calculators.messages.{Stats, FindStats, LogTimeout}
import java.util.Date
import com.akkaoogle.db.models._
class MonitorActor extends Actor {
  import context._
  
  val errorLogger = Agent(Map.empty[String, Int])
  // ideally you may want to save the existing error count in some persistence
  // storage so you can fetch the errors for later use.
  def preRestart = errorLogger send { old => Map.empty[String, Int] }
  def receive = {
    case LogTimeout(actorId, msg) =>
      logTimeout(actorId, msg)
    case FindStats(actorId) =>
      val timeouts = errorLogger().getOrElse(actorId, 0)
      sender ! Stats(actorId, timeouts = timeouts)
    }

  private def logTimeout(actorId: String, msg: String): Unit = {
    errorLogger send { errorLog =>
      val current = errorLog.getOrElse(actorId, 0)
      val newErrorLog = errorLog + (actorId -> (current + 1))
      val l = new TransactionFailure(actorId, msg, new Date(System.currentTimeMillis))
      l.save
      newErrorLog
    }
  }
}
~~~~~~
## Setting up Play2-mini ##

Play2-mini is a lightweight REST framework on top of the Play2 framework. It maps an
HTTP request to a function that takes an HTTP request and returns a response.

~~~~~~
trait ConfigureScalaBuild {
  lazy val typesafe = "Typesafe Repository" at "http://repo.typesafe.com/typesafe/releases/"
  lazy val typesafeSnapshot = "Typesafe Snapshots Repository" at "http://repo.typesafe.com/typesafe/snapshots/"
  val netty = Some("play.core.server.NettyServer")
  def scalaMiniProject(org: String, name: String, buildVersion: String,
        baseFile: java.io.File = file(".")) =
    Project(id = name, base = baseFile, settings = Project.defaultSettings).settings(
      version := buildVersion,
      organization := org,
      resolvers += typesafe,
      resolvers += typesafeSnapshot,
      libraryDependencies += "com.typesafe" %% "play-mini" % "2.1=RC2",
      mainClass in (Compile, run) := netty,
      ivyXML := <dependencies> <exclude org="org.springframework"/></dependencies>
    )
}

importimportsbt._
Keys._
object AkkaoogleBuild extends Build with ConfigureScalaBuild {
  import H2TaskManager._
  lazy val root =
    scalaMiniProject("com.akkaoogle","akkaoogle","1.0")
      .settings(startH2Task, stopH2Task)
      .settings(
        organization := "scalainaction",
        scalaVersion := "2.10.0",
        scalacOptions ++= Seq("-unchecked", "-deprecation"),
        resolvers += "Typesafe Repo" at "http://repo.typesafe.com/typesafe/repo",
        parallelExecution in Test := false
      ).settings(
      libraryDependencies ++= Seq(
        "com.typesafe.akka" %% "akka-actor" % "2.1.0",
        "com.typesafe.akka" %% "akka-remote" % "2.1.0",
        "com.typesafe.akka" %% "akka-agent" % "2.1.0",
        "com.h2database" % "h2" % "1.2.127",
        "org.squeryl" % "squery1_2.10-RC5" % "0.9.5-5",
        "org.specs2" %% "specs2" % "1.13" % "test",
        "org.eclipse.jetty" % "jetty-distribution" % "8.0.0.M2" % "test")
      )
}
~~~~~~

### Running with Play2-mini ###
Play2-mini–based application needs to implement `com.typesafe.play.mini.Setup`
~~~~~~
package com.typesafe.play.mini
class Setup(a: Application) extends GlobalSettings {
  ...
}
// Think of Application as a controller of the MVC model that handles all the requests.
package com.typesafe.play.mini
trait Application {
  // you have to implement is the routes method
  def route: PartialFunction[RequestHeader, Handler]
}
~~~~~~

~~~~~~
import com.akkaoogle.infrastructure._
import org.h2.tools.Server
import com.akkaoogle.db.AkkaoogleSchema._

//  com.akkaoogle.http.App will handle all the HTTP requests for the Akkaoogle application. 
object Global extends com.typesafe.play.mini.Setup(com.akkaoogle.http.App) {
  println("initializing the Akkaoogle schema")
  //  initialize the various parts
  createSchema()
  AkkaoogleActorServer.run()
}

package com.akkaoogle.http
import com.typesafe.play.mini._
import play.api.mvc._
import play.api.mvc.Results._
import com.akkaoogle.infrastructure._
import akka.pattern.{ ask, pipe, AskTimeoutException }
import com.akkaoogle.calculators.messages._
import play.api.libs.concurrent._
import scala.collection.JavaConverters._

object App extends Application {
  def route = {
    case GET(Path("/")) => Action { request =>
      Ok(views.index()).as("text/html")
    }
    case GET(Path("/akkaoogle/search")) & QueryString(qs) =>
      Action { request =>
        val desc = QueryString(qs, "productDescription").get.asScala
        val f =
          (AkkaoogleActorServer.lookup("cheapest-deal-finder-balancer") ?
             FindPrice(desc.head, 1)).mapTo[Option[LowestPrice]]
        val result = f.map({
          case Some(lowestPrice) =>
            Ok(lowestPrice.toString).as("text/html")
          case _ =>
            Ok("No price found").as("text/html")
            Return
        })
        AsyncResult(result.asPromise)
      }
  }
}
~~~~~~

~~~~~~
package com.akkaoogle.infrastructure

import com.akkaoogle.calculators._
import akka.actor._
import com.akkaoogle.db.models._
import akka.actor.{ActorRef, Actor}
import akka.routing._
import com.typesafe.config.ConfigFactory

object AkkaoogleActorServer {
  var system: Option[ActorSystem] = None
  def run(): Unit = {
    println("starting the remote server...")
    system = Some(ActorSystem("akkaoogle", ConfigFactory.load.getConfig("akkaoogle")))
    system.foreach(s => register(s))
  }
  
  private def register(implicit system: ActorSystem) {
    val monitor = system.actorOf(Props[MonitorActor], name = "monitor")
    
    val cheapestDealFinderLoadBalancer = system.actorOf(
      Props[CheapestDealFinder].withRouter(SmallestMailboxRouter(nrOfInstances = 10)),
      name = "cheapest-deal-finder-balancer")
      
    val internalPriceCalculators: List[ActorRef] = createInternalPriceCalculators(10)
    val internalLoadBalancer = system.actorOf(
      Props[InternalPriceCalculator].withRouter(RoundRobinRouter(routees = internalPriceCalculators)),
      name = "internal-load-balancer")
      
    val proxies = createExternalProxyActors(ExternalVendor.findAll)
    val externalPriceCalculators: List[ActorRef] = createExternalPriceCalculators(10, proxies)
    val externalLoadBalancer = system.actorOf(
      Props [ExternalPriceCalculator].withRouter(RoundRobinRouter(routees = externalPriceCalculators)),
      name="external-load-balancer")
  }
  
  def lookup(name: String): ActorRef = {
    system map { s =>
      val path = s / name
      s.actorFor(path)
    } getOrElse(throw new RuntimeException("No actor found"))
  }
  
  def stop(){
    system.foreach(_.shutdown())
  }
  
  private def createExternalProxyActors(vendors: Iterable[ExternalVendor])(implicit system: ActorSystem) = {
    val proxies = for(v <- vendors) yield {
      println("Creating vendor proxies for " + v.name)
      val ref = system.actorOf(Props(new ExternalVendorProxyActor(v))
          .withDispatcher("dispatchers.proxy-actor-dispatcher"),name=v.name)
      ref
    }
    proxies.toList
  }

  private def createInternalPriceCalculators(initialLoad: Int)(implicit system: ActorSystem) = {
    (for (i <- 0 until initialLoad) yield
    system.actorOf(
      Props [InternalPriceCalculator].withDispatcher("dispatchers.internal-price-calculator-actor-dispatcher"),
      name=("internal-price-calculator" + i))).toList
  }
  
  private def createExternalPriceCalculators(
    initialLoad: Int, proxies: List[ActorRef])(
    implicit system: ActorSystem) = {
      (for (i <- 0 until initialLoad)
        yield
      system.actorOf(
        Props(new ExternalPriceCalculator(proxies)).withDispatcher("dispatchers.external-price-calculator-actor-dispatcher"),
        name = ("external-price-calculator" + i))).toList
  }
}

~~~~~~
