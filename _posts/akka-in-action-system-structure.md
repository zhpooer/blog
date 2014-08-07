title: Akka in Action-System Structure & Routing Messages
date: 2014-08-04 18:47:29
tags:
- akka
- scala
---

One of the immediate implications of Actor based programming is how do we
model code that requires collaborators to work together if each unit of work is done
in parallel?

Akka allows us to use these design approaches while still making use of its
inherent concurrency.
* integration tools and platforms
* messaging systems
* WSO2, and SOA and Web-service based solutions

# Pipes and Filters #

The concept of piping refers to the ability for one process or thread to pump its
results to another processor for additional processing.

In many systems a single event will trigger a sequence of tasks.

It receives the photo and before the event is sent to central
processing, a number of checks are done. When no license plate is found in the
photo, the system is unable to process the message any further and therefore, it will
be discarded.  In this example we also discard the message when the speed is below
the maximum speed. Which means that only messages that contain the license
plate of a speeding vehicle end up getting to the central processor.

Each Filter consists of three parts, the inbound pipe where the message is
received, the processor of the message, and finally the outbound pipe where the
result of the processing is published.

An important restriction is that each filter must accept and
send the same messages, because the outbound pipe of a filter can be the inbound
pipe of any other filter in the pattern. 

A Pipe with Two Filters Example
~~~~~~
case class Photo(license: String, speed: Int)

class SpeedFilter(minSpeed: Int, pipe: ActorRef) extends Actor {

  // Filter all Photos which have a speed lower than the minimal speed
  def receive = {
    case msg: Photo =>
      if (msg.speed > minSpeed) pipe ! msg
  }
}

// Filter all Photos which have an empty license
class LicenseFilter(pipe: ActorRef) extends Actor {
  def receive = {
    case msg: Photo =>
      if (!msg.license.isEmpty) pipe ! msg
  }
}


// Pipe and filter test

val endProbe = TestProbe()
val speedFilterRef = system.actorOf( Props(new SpeedFilter(50, endProbe.ref)))

val licenseFilterRef = system.actorOf(Props(new LicenseFilter(speedFilterRef)))
val msg = new Photo("123xyz", 60)

licenseFilterRef ! msg
endProbe.expectMsg(msg)
licenseFilterRef ! new Photo("", 60)

endProbe.expectNoMsg(1 second)
licenseFilterRef ! new Photo("123xyz", 49)
endProbe.expectNoMsg(1 second)
~~~~~~

# Scatter-Gather Pattern #
The first case is when the
tasks are functionally the same, but only one is passed through to the gather
component as the chosen result. The second scenario is when work is divided for
parallel processing and each processor submits its results which are then combined
into a result set by the aggregator. 

## Competing Tasks ##

A client buys a product, let's say a book at a
web shop, but the shop doesn't have the requested book in stock, so it has to buy
the book from a supplier. But the shop is doing business with three different
suppliers and wants to pay the lowest price.


~~~~~~
case class PhotoMessage(id: String,
    photo: String,
    creationTime: Option[Date] = None,
    speed: Option[Int] = None)
    
// task  handler
class GetSpeed(pipe:ActorRef) extends Actor {
  def receive = {
    case msg:PhotoMessage => {
      pipe ! msp.copy(speed = ImageProcessing.getSpeed(msg.photo))
    }
  }
}
class GetTime(pipe: ActorRef) extends Actor {
  def receive = {
    case msg: PhotoMessage => {
      pipe ! msg.copy(creationTime = ImageProcessing.getTime(msg.photo))
    }
  }
}

// distribute received message to all processing tasks
class RecipientList(recipientList:Seq[ActorRef]) extends Actor {
  def receive = {
    case msg: AnyRef => recipientList.foreach( _ ! msg)
  }
}

case class TimeoutMessage(msg:PhotoMessage)

// 聚合, 等待两个actor消息, 当一个消息回来时, 接着等待另一个, 加上了定时功能
class Aggregator(timeout:Duration, pipe:ActorRef) extends Actor {
  val messages = new ListBuffer[PhotoMessage]
  implicit val ec = context.system.dispatcher
  // Send all the received messages to our own mailbox

  override def preRestart(reason: Throwable, message: Option[Any]) {
    super.preRestart(reason, message)
    messages.foreach(self ! _)
    messages.clear()
  }
  
  // The first thing when receiving a message,
  // is to check if it is the first message or the second. 
  def receive = {
    case rcvMsg: PhotoMessage => {
      messages.find(_.id == rcvMsg.id) match {
        case Some(alreadyRcvMsg) => {
        
          val newCombinedMsg = new PhotoMessage(
            rcvMsg.id,
            rcvMsg.photo,
            rcvMsg.creationTime.orElse(alreadyRcvMsg.creationTime),
            rcvMsg.speed.orElse(alreadyRcvMsg.speed) )
            
          pipe ! newCombinedMsg
          //cleanup message
          messages -= alreadyRcvMsg
        }
        case None =>{
          messages += rcvMsg
          // 如果在规定的时间内没有收到信息
          context.system.scheduler.scheduleOnce(
            timeout,
            self,
            new TimeoutMessage(rcvMsg))
        }
    }
    case TimeoutMessage(rcvMsg) => {
      messages.find(_.id == rcvMsg.id) match {
        case Some(alreadyRcvMsg) => {
          pipe ! alreadyRcvMsg
          messages -= alreadyRcvMsg
        }
        case None => //message is already processed
    }
    case ex: Exception => throw ex
  }
}

~~~~~~
Test Aggregator missing a message
~~~~~~
val endProbe = TestProbe()
val actorRef = system.actorOf(Props(new Aggregator(1 second, endProbe.ref)))
val photoStr = ImageProcessing.createPhotoString(new Date(), 60)
val msg1 = PhotoMessage("id1", photoStr, Some(new Date()), None)
actorRef ! msg1
actorRef ! new IllegalStateException("restart")
val msg2 = PhotoMessage("id1",photoStr, None, Some(60))
actorRef ! msg2

val combinedMsg = PhotoMessage("id1", photoStr, msg1.creationTime, msg2.speed)
endProbe.expectMsg(combinedMsg)

~~~~~~

# Routing Messages #
Akka’s routing feature lets you alter a message’s path from
sender to receiver.

Akka provides a set of standard routers that implement routing patterns that
frequently show up in those everyday problems.

## RoundRobinRouter ##

The RoundRobinRouter sends messages to the Actors for which it fronts in
a round-robin fashion.

## SmallestMailboxRouter ##

When a message comes into the Router, it decides to route the message to
the composed Actor whose Mailbox is the smallest.

The SmallestMailboxRouter is a pretty good choice when it comes to
balancing load among your composed Actors. 

## Configuring a Router ##
The configurations are specified in the akka.actor.deployment block. Be-
low is a simple configuration of a RoundRobinRouter, which would appear
in your application.conf file:
~~~~~~
akka {
  actor {
    deployment {
      /DatabaseConnectionRouter {
        router = "round-robin"
        nr-of-instances = 20
      }
    }
  }
}
~~~~~~

~~~~~~
import akka.routing.FromConfig
class DBConnection extends Actor { ...
}
val dbRouter = system.actorOf(Props[DBConnection].withRouter(FromConfig(),
                              "DatabaseConnectionRouter"), "DBRouter")
~~~~~~

The dbRouter will be an instance of a RoundRobinRouter that will rep-
resent the database connection, and it will also be the parent of 20 instances
of the DBConnection Actor.

## Routers and Children ##

Routers route to routees. You can create those routees dynamically by the
Router, or you can assign them to it from an already created set. 

These two different methods of assigning routees have an impact
on the relationship and supervision of those routees.

## Letting the Router Create the Routees ##

Advantages
* The Router handles the Supervision.
* It works well with configuration.

Disadvantages
* You’ll have a difficult time constructing anything but a single type of
Actor.
* You can’t name them the way you might like.

## Passing the Router Pre-Created Actors ##

* You have to have parents for them.
* It’s not as flexible from a configuration perspective. 

## The Router and Its Children’s Life Cycles ##

When the Router creates the routees, they are its children, which means that
the Router must manage their life cycles. The Router will assign a
supervisorStrategy that always escalates the decision to its parent.

~~~~~~
val dbRouter = system.actorOf(Props.empty.withRouter(RoundRobinRouter(
    nrOfInstances = 5,
    supervisorStrategy = OneForOneStrategy {
    // define your Decider here
    }), "DBRouter")

// you can assign that easily enough:
val dbRouter = system.actorOf(Props.empty.withRouter(RoundRobinRouter(
    nrOfInstances = 5,
    supervisorStrategy = SupervisorStrategy.defaultStrategy
    ), "DBRouter")
~~~~~~

## Routers on a Plane ##
A BroadcastRouter would make the perfect component for allowing the passengers
to receive important information, such as “Fasten Seat Belts,”.

~~~~~~
zzz.akka.avionics {
  passengers = [
    [ "Kelly Franqui", "01", "A" ],
    [ "Tyrone Dotts", "02", "B" ],
    [ "Malinda Class", "03", "C" ],
    [ "Kenya Jolicoeur", "04", "A" ],
    [ "Christian Piche", "10", "B" ],
    [ "Neva Delapena", "11", "C" ],
    [ "Alana Berrier", "12", "A" ],
    [ "Malinda Heister", "13", "B" ],
    [ "Carlene Heiney", "14", "C" ],
    [ "Erik Dannenberg", "15", "A" ],
    [ "Jamie Karlin", "20", "B" ],
    [ "Julianne Schroth", "21", "C" ],
    [ "Elinor Boris", "22", "A" ],
    [ "Louisa Mikels", "30", "B" ],
    [ "Jessie Pillar", "31", "C" ],
    [ "Darcy Goudreau", "32", "A" ],
    [ "Harriett Isenhour", "33", "B" ],
    [ "Odessa Maury", "34", "C" ],
    [ "Malinda Hiett", "40", "A" ],
    [ "Darcy Syed", "41", "B" ],
    [ "Julio Dismukes", "42", "C" ],
    [ "Jessie Altschuler", "43", "A" ],
    [ "Tyrone Ericsson", "44", "B" ],
    [ "Mallory Dedrick", "50", "C" ],
    [ "Javier Broder", "51", "A" ],
    [ "Alejandra Fritzler", "52", "B" ],
    [ "Rae Mcaleer", "53", "C" ]
  ]
}

~~~~~~

~~~~~~
object Passenger {
  // These are notifications that tell the Passenger
  // to fasten or unfasten their seat belts
  case object FastenSeatbelts
  case object UnfastenSeatbelts
  // Regular expression to extract Name-Row-Seat tuple
  val SeatAssignment = """([\w\s_]+)-(\d+)-([A-Z])""".r
}

// The DrinkRequestProbability trait defines some
// thresholds that we can modify in tests to
// speed things up.
trait DrinkRequestProbability {
  // Limits the decision on whether the passenger
  // actually asks for a drink
  val askThreshold = 0.9f
  
  // The minimum time between drink requests
  val requestMin = 20.minutes
  
  // Some portion of this (0 to 100
  // to requestMin
  val requestUpper = 30.minutes
  // Gives us a 'random' time within the previous
  // two bounds
  def randomishTime(): Duration = {
    requestMin + scala.util.Random.nextInt(requestUpper.toMillis.toInt).millis
  }
}

// The idea behind the PassengerProvider is old news at this point.
// We can use it in other classes to give us the ability to slide
// in different Actor types to ease testing.
trait PassengerProvider {
  def newPassenger(callButton: ActorRef): Actor =
    new Passenger(callButton) with DrinkRequestProbability
}
~~~~~~

~~~~~~
class Passenger(callButton: ActorRef) extends Actor with ActorLogging {
  this: DrinkRequestProbability =>
  import Passenger._
  import FlightAttendant.{GetDrink, Drink}
  import scala.collection.JavaConverters._
  
  // We'll be adding some randomness to our Passenger,
  // and this shortcut will make things a little more
  // readable.
  val r = scala.util.Random
  
  // It's about time that someone actually asked for a
  // drink since our Flight Attendants have been coded
  // to serve them up
  case object CallForDrink
  
  // The name of the Passenger can't have spaces in it,
  // since that's not a valid character in the URI
  // spec. We know the name will have underscores in
  // place of spaces, and we'll convert those back
  // here.
  val SeatAssignment(myname, _, _) =
    self.path.name.replaceAllLiterally("_", " ")
    
  // We'll be pulling some drink names from the
  // configuration file as well
  val drinks = context.system.settings.config.getStringList("zzz.akka.avionics.drinks").asScala.toIndexedSeq
  
  // A shortcut for the scheduler to make things look
  // nicer later
  val scheduler = context.system.scheduler
  
  // We've just sat down, so it's time to get a drink
  override def preStart() {
    self ! CallForDrink
  }

  // This method will decide whether or not we actually
  // want to get a drink using some randomness to
  // decide
  def maybeSendDrinkRequest(): Unit = {
    if (r.nextFloat() > askThreshold) {
      val drinkname = drinks(r.nextInt(drinks.length))
      callButton ! GetDrink(drinkname)
    }
    scheduler.scheduleOnce(randomishTime(), self, CallForDrink)
  }

  // Standard message handler
  def receive = {
    case CallForDrink =>
      maybeSendDrinkRequest()
    case Drink(drinkname) =>
      log.info("{} received a {} - Yum", myname, drinkname)
    case FastenSeatbelts =>
      log.info("{} fastening seatbelt", myname)
    case UnfastenSeatbelts =>
      log.info("{} UNfastening seatbelt", myname)
  }
}

~~~~~~


Testing and the Event Stream

~~~~~~
trait TestDrinkRequestProbability extends DrinkRequestProbability {
  override val askThreshold = 0f
  override val requestMin = 0.milliseconds
  override val requestUpper = 2.milliseconds
}

class PassengersSpec extends TestKit(ActorSystem()) with ImplicitSender {
  import akka.event.Logging.Info
  import akka.testkit.TestProbe
  var seatNumber = 9
  
  def newPassenger(): ActorRef = {
    seatNumber += 1
    system.actorOf(Props(new Passenger(testActor) with TestDrinkRequestProbability), s"Pat_Metheny-$seatNumber-B")
  }
  
  "Passengers" should {
    "fasten seatbelts when asked" in {
      val a = newPassenger()
      val p = TestProbe()
      // This says that we want the TestProbe’s ActorRef (p.ref) to be the handle
      // to the subscribed Actor, and that we want it to receive events that match the
      // class akka.event.Logger.Info. 
      system.eventStream.subscribe(p.ref, classOf[Info])
      
      a ! FastenSeatbelts
      p.expectMsgPF() {
        case Info(_, _, m) =>
          m.toString must include ("fastening seatbelt")
      }
    }
  }
}


~~~~~~
### The Passenger Router ###

~~~~~~
object PassengerSupervisor {
  // Allows someone to request the BroadcastRouter
  case object GetPassengerBroadcaster
  // Returns the BroadcastRouter to the requestor
  case class PassengerBroadcaster(broadcaster: ActorRef)
  // Factory method for easy construction
  def apply(callButton: ActorRef) = new PassengerSupervisor(callButton)
        with PassengerProvider
}

class PassengerSupervisor(callButton: ActorRef) extends Actor {
  this: PassengerProvider =>
  import PassengerSupervisor._
  // We'll resume our immediate children instead of restarting them
  // on an Exception
  override val supervisorStrategy = OneForOneStrategy() {
    case _: ActorKilledException => Escalate
    case _: ActorInitializationException => Escalate
    case _ => Resume
  }
  
  // Internal messages we use to communicate between this Actor
  // and its subordinate IsolatedStopSupervisor
  case class GetChildren(forSomeone: ActorRef)

  case class Children(children: Iterable[ActorRef], childrenFor: ActorRef)
  
  // We use preStart() to create our IsolatedStopSupervisor
  override def preStart() {
    context.actorOf(Props(new Actor {
      val config = context.system.settings.config
      
      override val supervisorStrategy = OneForOneStrategy() {
        case _: ActorKilledException => Escalate
        case _: ActorInitializationException => Escalate
        case _ => Stop
      }
      override def preStart() {
        import scala.collection.JavaConverters._
        import com.typesafe.config.ConfigList
        // Get our passenger names from the configuration
        val passengers = config.getList("zzz.akka.avionics.passengers")
        
        // Iterate through them to create the passenger children
        passengers.asScala.foreach { nameWithSeat =>
          val id = nameWithSeat.asInstanceOf[ConfigList].unwrapped().asScala.mkString("-").replaceAllLiterally(" ", "_")
          // Convert spaces to underscores to comply with URI standard
          context.actorOf(Props(newPassenger(callButton)), id)
        }
      }
    
      override def receive = {
        case GetChildren(forSomeone: ActorRef) =>
          sender ! Children(context.children, forSomeone)
      }
    }), "PassengersSupervisor")
  }

  def receive = noRouter

  // TODO: This noRouter method could be made simpler by using a Future.
  // We'll have to refactor this later.
  def noRouter: Receive = {
    case GetPassengerBroadcaster =>
      context.actorFor("PassengersSupervisor") ! GetChildren(sender)
    case Children(passengers, destinedFor) =>
      val router = context.actorOf(Props().withRouter(
        BroadcastRouter(passengers.toSeq)), "Passengers")
        
      destinedFor ! PassengerBroadcaster(router)
      context.become(withRouter(router))
  }
  
  def withRouter(router: ActorRef): Receive = {
    case GetPassengerBroadcaster =>
      sender ! PassengerBroadcaster(router)
  }

}
~~~~~~

### Using the Passenger Router ###

~~~~~~
package zzz.akka.avionics

import akka.actor.{ActorSystem, Actor, ActorRef, Props}
import akka.testkit.{TestKit, ImplicitSender}
import scala.concurrent.util.duration._
import com.typesafe.config.ConfigFactory
import org.scalatest.{WordSpec, BeforeAndAfterAll}
import org.scalatest.matchers.MustMatchers


// ActorSystem so we have a known quantity we can test with
object PassengerSupervisorSpec {
  val config = ConfigFactory.parseString("""
    zzz.akka.avionics.passengers = [
      [ "Kelly Franqui", "23", "A" ],
      [ "Tyrone Dotts", "23", "B" ],
      [ "Malinda Class", "23", "C" ],
      [ "Kenya Jolicoeur", "24", "A" ],
      [ "Christian Piche", "24", "B" ]
    ]
  """)
}

// We don't want to work with "real" passengers.
 This mock
// passenger will be much easier to verify things with
trait TestPassengerProvider extends PassengerProvider {
  override def newPassenger(callButton: ActorRef): Actor =
    new Actor {
      def receive = {
        case m => callButton ! m
      }
    }
}

// The Test class injects the configuration into the
// ActorSystem
class PassengerSupervisorSpec extends
    TestKit(ActorSystem("PassengerSupervisorSpec", PassengerSupervisorSpec.config))
        with ImplicitSender
        with WordSpec
        with BeforeAndAfterAll
        with MustMatchers {
  import PassengerSupervisor._
  // Clean up the system when all the tests are done
  override def afterAll() {
    system.shutdown()
  }

  "PassengerSupervisor" should {
    "work" in {
      // Get our SUT
      val a = system.actorOf(Props(new PassengerSupervisor(testActor) with TestPassengerProvider))
      
      // Grab the BroadcastRouter
      a ! GetPassengerBroadcaster
      val broadcaster = expectMsgPF() {
        case PassengerBroadcaster(b) =>
          // Exercise the BroadcastRouter
          b ! "Hithere"
          
          // All 5 passengers should say "Hithere"
          expectMsg("Hithere")
          expectMsg("Hithere")
          expectMsg("Hithere")
          expectMsg("Hithere")
          expectMsg("Hithere")
          
          // And then nothing else!
          expectNoMsg(100.milliseconds)
          // Return the BroadcastRouter
          b
      }
      
      // Ensure that the cache works
      a ! GetPassengerBroadcaster
      expectMsg(PassengerBroadcaster(`broadcaster`))
    }
  }
}
~~~~~~

TODO


