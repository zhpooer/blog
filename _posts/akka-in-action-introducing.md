title: Akka in Action-Introduce
date: 2014-08-03 11:15:32
tags:
- Akka
- scala
---



# 启动案例 #
Akka is based on the Actor programming model.

~~~~~~
git clone https://github.com/RayRoestenburg/akka-in-action.git
sbt assembly
java -jar target/scala-2.10/goticks-server.jar
~~~~~~

# 模板 #

~~~~~~
name := "goticks"

version := "0.1-SNAPSHOT"

organization := "com.goticks"

scalaVersion := "2.10.0"

resolvers ++=
  Seq("repo" at "http://repo.typesafe.com/typesafe/releases/",
    "Spray Repository" at "http://repo.spray.io",
    "Spray Nightlies" at "http://nightlies.spray.io/")

libraryDependencies ++= {
  val akkaVersion = "2.1.2"
  val sprayVersion = "1.1-20130123"
  Seq(
    "com.typesafe.akka" %% "akka-actor" % akkaVersion,
    "io.spray" % "spray-can" % sprayVersion,
    "io.spray" % "spray-routing" % sprayVersion,
    "io.spray" %% "spray-json" % "1.2.3",
    "com.typesafe.akka" %% "akka-slf4j" % akkaVersion,
    "ch.qos.logback" % "logback-classic" % "1.0.10",
    "com.typesafe.akka" %% "akka-testkit" % akkaVersion % "test",
    "org.scalatest" %% "scalatest" % "1.9.1" % "test"
  )
}

~~~~~~

# GoTicks.com #
Our ticket selling service which will allow customers to buy tickets to all sorts of
events, concerts, sports games and the like.

Once all the tickets are sold for an event the server should respond with a 404 (Not
Found) HTTP status code.

REST API

| Description | Http Method | URL | Body | Response exammple |
|--------------|
| create an event with a number of tickets | PUT | /events | {event:rhcp, nrOfTickets:250} | Http 200 OK |
| Get an overview of all events and the number of tickets available. | GET | /events |  | [ { event : "RHCP", nrOfTickets : 249}, { event : "Radiohead", nrOfTickets : 130}, ] |
| Buy a ticket | GET | /ticket/:eventName |  | { event: "RHCP", nr: 1 } or HTTP 404 | 

启动服务器
~~~~~~
sbt run
~~~~~~

# Structure of the App #

![Structure of the App](/img/akka_buy_tickets.png)

~~~~~~
package com.goticks
import spray.can.server.SprayCanHttpServerApp
import akka.actor.Props
import com.typesafe.config.ConfigFactory
object Main extends App with SprayCanHttpServerApp {
  val config = ConfigFactory.load()
  val host = config.getString("http.host")
  val port = config.getInt("http.port")
  val api = system.actorOf(
    Props(new RestInterface()),
    "httpInterface"
  )
  newHttpServer(api) ! Bind(interface = host, port = port)
}
~~~~~~

REST Interface Message Classes
~~~~~~
// Message to create an event
case class Event(event:String, nrOfTickets:Int)
// Message for requesting the state of all events
case object GetEvents
// Response message that contains current status of all events
case class Events(events:List[Event])
// Signal event to indicate an event was created
case object EventCreated
// Request for a ticket for a particular event
case class TicketRequest(event:String)
// Signal event that the event is sold out
case object SoldOut
// New tickets for an Event, created by BoxOffice
case class Tickets(tickets:List[Ticket])
// Message to buy a ticket from the TicketSeller
case object BuyTicket
// The numbered ticket to an event
case class Ticket(event:String, nr:Int)
~~~~~~
Akka is going to get
these parts to go together with immutable messages, so the Actors have to be
designed to get all the information they need, and produce all that is needed if they
enlist any collaborators. 

## The Actor that handles the sale: TicketSeller ##
The TicketSeller is created by the BoxOffice and just simply keeps a list of tickets.

~~~~~~
package com.goticks
import akka.actor.{PoisonPill, Actor}
class TicketSeller extends Actor {
  import TicketProtocol._
  var tickets = Vector[Ticket]()
  def receive = {
    case GetEvents => sender ! tickets.size
    case Tickets(newTickets) =>
      tickets = tickets ++ newTickets
    case BuyTicket =>
      if (tickets.isEmpty) {
        sender ! SoldOut
        // cleans up the actor 
        self ! PoisonPill
      }
      tickets.headOption.foreach { ticket =>
        tickets = tickets.tail
        sender ! ticket
      }

  }
}

~~~~~~

## BoxOffice ##
The BoxOffice needs to create TicketSeller children for every event and delegates
the selling to them.
~~~~~~
// create Event
case Event(name, nrOfTickets) =>
  // If TicketSellers have not been created already
  if(context.child(name).isEmpty) {
    // create the actor
    val ticketSeller = context.actorOf(Props[TicketSeller], name)
    val tickets = Tickets((1 to nrOfTickets).map{
      nr=> Ticket(name, nr)).toList
    }
    ticketSeller ! tickets
  }
  sender ! EventCreated
// buy ticket
case TicketRequest(name) =>
  context.child(name) match {
    case Some(ticketSeller) => ticketSeller.forward(BuyTicket)
    case None => sender ! SoldOut
  }

~~~~~~

~~~~~~
import akka.pattern.ask
val capturedSender = sender

def askEvent(ticketSeller:ActorRef): Future[Event] = {
  val futureInt = ticketSeller.ask(GetEvents).mapTo[Int]
  futureInt.map { nrOfTickets =>
    Event(ticketSeller.actorRef.path.name, nrOfTickets)
  }
}

val futures = context.children.map { ticketSeller =>
  askEvent(ticketSeller)
}

// sends an Events message back to the sender once all responses
// have been handled.
Future.sequence(futures).map { events =>
  capturedSender ! Events(events.toList)
}
~~~~~~

## REST Interface ##

~~~~~~
// Creation of the BoxOffice Actor
val BoxOffice = context.actorOf(Props[BoxOffice])

// stays around for the lifetime of the HTTP request
def createResponder(requestContext:RequestContext) = {
  context.actorOf(Props(new Responder(requestContext, BoxOffice)))
}

//  a snippet of the DSL that is used to handle HTTP requests:
path("ticket") {
  get {
      entity(as[TicketRequest]) { ticketRequest => requestContext =>
        val responder = createResponder(requestContext)
        BoxOffice.ask(ticketRequest).pipeTo(responder)
    }
  }
}

class Responder(requestContext:RequestContext, BoxOffice:ActorRef)
    extends Actor with ActorLogging {
  import TicketProtocol._
  import spray.httpx.SprayJsonSupport._
  
  def receive = {
    case ticket:Ticket =>
      requestContext.complete(StatusCodes.OK, ticket)
      self ! PoisonPill
    case EventCreated =>
      requestContext.complete(StatusCodes.OK)
      self ! PoisonPill
    case SoldOut =>
      requestContext.complete(StatusCodes.NotFound)
      self ! PoisonPill
    case Events(events) =>
      requestContext.complete(StatusCodes.OK, events)
      self ! PoisonPill
  }
}
~~~~~~
