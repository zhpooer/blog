title: Akka in Action-Testing Actors & Fault tolerance
date: 2014-08-03 15:20:54
tags:
- scala
- akka
---

# difficult in testing Actors #


* Timing - Sending messages is asynchronous, so it is difficult to know when
to assert expected values in the unit test.
* Asynchronicity - Actors are meant to be run in parallel on several threads.
* Statelessness - An actor hides its internal state and does not allow access to
this state.
* Collaboration/Integraton - If you want to do an integration test of a couple
of actors, you would need to eavesdrop in between the actors to assert that
the messages have the expected values.


# akka-testkit #

Akka provides the akka-testkit module.The testkit module makes a
couple of different types of tests possible:
* Single threaded unit testing 
* Multi-threaded unit testing,  The testkit module provides the TestKit
and TestProbe classes, which make it possible to receive replies from
actors, inspect messages and set timing bounds for particular messages to
arrive.
* Multiple JVM testing, comes in handy when you want to test remote actor systems.

The TestKit has the TestActorRef extending the LocalActorRef class
and sets the dispatcher to a CallingThreadDispatcher that is built for
testing only.

# Preparing to Test #
The TestKit exposes a system value, which can be accessed
in the test to create actors and everything else you would like to do with the system.
~~~~~~
import org.scalatest.{ Suite, BeforeAndAfterAll }
import akka.testkit.TestKit

// Stop the system after all tests are done
trait StopSystemAfterAll extends BeforeAndAfterAll {
  this: TestKit with Suite =>
  override protected def afterAll() {
    super.afterAll()
    system.shutdown()
  }
}
~~~~~~

# One-way messages #

There are a three variations that we will look at:
* SilentActor - An actor's behavior is not directly observable from the
outside, it might be an intermediate step that the actor takes to create some
internal state.
* SendingActor - An actor sends a message to another actor (or possibly
many actors) after it is done processing the received message.
* SideEffectingActor - An actor receives a message and interacts with a
normal object in some kind of way.

## SilentActor Examples ##

~~~~~~
object SilentActorProtocol {
  case class SilentMessage(data: String)
  case class GetState(receiver: ActorRef)
}
class SilentActor extends Actor {
  import SilentActorProtocol._
  var internalState = Vector[String]()
  def receive = {
    case SilentMessage(data) =>
      internalState = internalState :+ data
    case GetState(receiver) => receiver ! internalState
  }
  def state = internalState
}

class SilentActor01Test extends TestKit(ActorSystem("testsystem"))
    with WordSpec         // WordSpec provides an easy to read DSL for testing in the BDD style
    with MustMatchers     // MustMatchers provides easy to read assertions
    with StopSystemAfterAll {
  // MustMatchers provides easy to read assertions
  "A Silent Actor" must {
    "change state when it receives a message, single threaded" in {
      import SilentActorProtocol._
      val silentActor = TestActorRef[SilentActor]
      silentActor ! SilentMessage("whisper")
      // Get the underlying actor and assert the state
      silentActor.underlyingActor.state must (contain("whisper"))
    }
    
    "change state when it receives a message, multi-threaded" in {
      import SilentActorProtocol._
      val silentActor = system.actorOf(Props[SilentActor], "s3")
      silentActor ! SilentMessage("whisper1")
      silentActor ! SilentMessage("whisper2")
      silentActor ! GetState(testActor)
      // Used to check what message(s) have been sent to the testActor
      expectMsg(Vector("whisper1", "whisper2"))
      
      //Write the test, first fail
      fail("not implemented yet")
    }
  }
}
~~~~~~

## SendingActor Example ##
Returning to our ticketing example from Chapter 1, we need to test the fact that
when we buy a Ticket to an Event , the count of available tickets is properly
decremented. 

~~~~~~
"A Sending Actor" must {
  "send a message to an actor when it has finished" in {
    import Kiosk01Protocol._
    val props = Props(new Kiosk01(testActor))
    val sendingActor = system.actorOf(props, "kiosk1")
    val tickets = Vector(Ticket(1), Ticket(2), Ticket(3))
    val game = Game("Lakers vs Bulls", tickets)
    sendingActor ! game
    // the testActor should receive one ticket less
    // expect Message Partial function
    expectMsgPF() {
      case Game(_, tickets) =>
        tickets.size must be(game.tickets.size - 1)
    }
  }
}

object Kiosk01Protocol {
  case class Ticket(seat: Int)
  case class Game(name: String, tickets: Seq[Ticket])
}

class Kiosk01(nextKiosk: ActorRef) extends Actor {
  import Kiosk01Protocol._
  def receive = {
  case game @ Game(_, tickets) =>
    nextKiosk ! game.copy(tickets = tickets.tail)
  }
}

~~~~~~

Let's write a test for the FilteringActor. 
The FilteringActor that we are going to build should filter out
duplicate events.

~~~~~~
"filter out particular messages" in {
  import FilteringActorProtocol._
  val props = Props(new FilteringActor(testActor, 5))
  val filter = system.actorOf(props, "filter-1")
  
  filter ! Event(1)
  filter ! Event(2)
  filter ! Event(1)
  filter ! Event(3)
  filter ! Event(1)
  filter ! Event(4)
  filter ! Event(5)
  filter ! Event(5)
  //  Event(6) does not match the pattern in the case statement
  filter ! Event(6)
  
  // The receiveWhile method returns the collected items as they are returned in
  // the partial function as a list, which is not allowed to have any duplicates. 
  val eventIds = receiveWhile() {
    case Event(id) if id <= 5 => id
  }
  
  eventIds must be(List(1, 2, 3, 4, 5))
  // Assert that the duplicates are not in the result
  expectMsg(Event(6))
}



object FilteringActorProtocol {
  case class Event(id: Long)
}

// The oldest message that was received is discarded when a max
// bufferSize is reached to prevent the lastMessages list from growing too
// large and possibly causing us to run out of space.

class FilteringActor(nextActor: ActorRef,
      bufferSize: Int) extends Actor {
    import FilteringActorProtocol._
    var lastMessages = Vector[Event]()
    def receive = {
      case msg: Event =>
        if (!lastMessages.contains(msg)) {
          lastMessages = lastMessages :+ msg
          nextActor ! msg
          if (lastMessages.size > bufferSize) {
            // discard the oldest
            lastMessages = lastMessages.tail
          }
        }
    }
}

"filter out particular messages using expectNoMsg" in {
  import FilteringActorProtocol._
  val props = Props(new FilteringActor(testActor, 5))
  val filter = system.actorOf(props, "filter-2")
  filter ! Event(1)
  filter ! Event(2)
  expectMsg(Event(1))
  expectMsg(Event(2))
  filter ! Event(1)
  expectNoMsg
  filter ! Event(3)
  expectMsg(Event(3))
  filter ! Event(1)
  expectNoMsg
  filter ! Event(4)
  filter ! Event(5)
  filter ! Event(5)
  expectMsg(Event(4))
  expectMsg(Event(5))
  expectNoMsg()
}
~~~~~~

The TestProbe class is very much like the TestKit, only you
can use this class without having to extend from it.
Simply create a TestProbe with `TestProbe()` and start using it.

## SideEffectingActor Example ##
It does just one thing: its Greeter receives
a message and outputs it to the console.

The SideEffectingActor allows us to test scenarios such as these:
where the effect of the action is not directly accessible.

~~~~~~
import Greeter01Test._
// Create a system with a configuration that attaches a test event listener
class Greeter01Test extends TestKit(testSystem)
    with WordSpec
    with MustMatchers
    with StopSystemAfterAll {
  "The Greeter" must {
    "say Hello World! when a Greeting("World") is sent to it" in {
      val dispatcherId = CallingThreadDispatcher.Id
      // Single threaded environment
      val props = Props[Greeter].withDispatcher(dispatcherId)
      val greeter = system.actorOf(props)
      // Intercept the log messages that were logged
      EventFilter.info(message = "Hello World!", occurrences = 1).intercept {
        // The filter is applied when the intercept code block is executed,
        // which is when we send the message.
        greeter ! Greeting("World")
      }
    }
  }
}

object Greeter01Test {
  val testSystem = {
    // parse a configuration file from a String, in this case we only override the event handlers list.
    val config = ConfigFactory.parseString("""akka.event-handlers = ["akka.testkit.TestEventListener"]""")
    ActorSystem("testsystem", config)
  }
}

// 注册一个监听器
class Greeter02(listener: Option[ActorRef] = None)
    extends Actor with ActorLogging {
  def receive = {
    case Greeting(who) =>
      val message = "Hello " + who + "!"
      log.info(message)
      listener.foreach(_ ! message)
  }
}
~~~~~~

~~~~~~
class Greeter02Test extends TestKit(ActorSystem("testsystem"))
    with WordSpec
    with MustMatchers
    with StopSystemAfterAll {
    
  "The Greeter" must {
    "say Hello World! when a Greeting("World") is sent to it" in {
      val props = Props(new Greeter02(Some(testActor)))
      val greeter = system.actorOf(props, "greeter02-1")
      greeter ! Greeting("World")
      expectMsg("Hello World!")
    }
  "say something else and see what happens" in {
      val props = Props(new Greeter02(Some(testActor)))
      val greeter = system.actorOf(props, "greeter02-2")
      // 获取没有处理的 信息
      system.eventStream.subscribe(testActor, classOf[UnhandledMessage])
      greeter ! "World"
      expectMsg(UnhandledMessage("World", system.deadLetters, greeter))
    }
  }
}
~~~~~~

# Two-way messages #
Two-way messages are quite easy to test in a black box fashion, a request
should result in a response, which you can simply assert. In the following test we
will test an EchoActor , an actor that echoes any request back in a response.

~~~~~~
"Reply with the same message it receives without ask" in {
  val echo = system.actorOf(Props[EchoActor], "echo2")
  // Call tell with the testActor as the sender
  echo.tell("some message", testActor)
  expectMsg("some message")
}

class EchoActor extends Actor {
  def receive = {
    case msg =>
      sender ! msg
  }
}
~~~~~~

# What is fault tolerance #

A fault should be contained to a part of the system and not escalate to a total
crash.

Isolating a faulty component means that some structure needs to exist to
isolate it from; the system will need a defined structure in which active parts
can be isolated.

A backup component should be able to take over when a component fails.

If a faulty component can be isolated we can also replace it in the structure.
The other parts of the system should be able to communicate with the
replaced component just as they did before with the failed component.

If a component gets into an incorrect state, we need to have the ability to get
it back to a defined initial state. 

A faulty component needs to be isolated and if it cannot recover it should be
terminated and removed from the system or re-initialized with a correct
starting state.

When a component fails we would like all calls to the component to be
suspended until the component is fixed or replaced so that when it is, the new
component can continue the work without dropping a beat.

It would be great if the fault recovery code could be separated from the
normal processing code. 

##  Plain old objects and exceptions ##

* Recreating objects and their dependencies and replacing these in the application structure
is not available as a first-class feature.
* Objects communicate with each other directly so it is hard to isolate them.
* The fault recovery code and the functional code are tangled up with each other.

## Let it crash ##
Instead of using one flow to handle both normal code and recovery code Akka
provides two separate flows; one for normal logic and one for fault recovery logic.

The normal flow consists of actors that handle normal messages, the recovery flow
consists of actors that monitor the actors in the normal flow. Actors that monitor
other actors are called supervisors.

The actor code only contains normal processing logic and no error handling or fault
recovery logic, so its effectively not part of the recovery process, which keeps
things much clearer. The mailbox for a crashed actor is suspended until the
supervisor in the recovery flow has decided what to do with the exception.

Akka has chosen to enforce parental
supervision , meaning that any actor that creates actors automatically becomes the
supervisor of those actors.

The supervisor has 4 options when deciding what to do with the actor:
* Restart; the actor must be recreated from its Props. after it is restarted (or rebooted if you
will) the actor will continue to process messages. Since the rest of the application uses an
ActorRef to communicate with the actor the new actor instance will automatically get the
next messages.
* Resume; the same actor instance should continue to process messages, the crash is
ignored.
* Stop; the actor must be terminated. It will no longer take part in processing messages.
* Escalate; the supervisor does not know what to do with it and escalates the problem to its
parent, which is also a supervisor.
![Normal and recovery flow in the logs processing application](/img/akka_recover.png)


Akka chooses not to provide the failing message to the mailbox again after
a restart, but there is a way to do this yourself if you are absolutely sure that the
message did not cause the error, which we will discuss later.

# Actor life-cycle #
During the life cycle of an actor there are three types of events:

1. The actor is created and started, for simplicity we
will refer to this as the Start event.
2. The actor is restarted on the Restart event.
3. The actor is stopped by the Stop event.


## Start event ##

An actor is created and automatically started with the actorOf method. Top level
actors are created with the actorOf method on the ActorSystem . A parent actor
creates a child actor using the actorOf on its ActorContext .

The preStart hook is called just before the actor is started. To use this trigger we
have to override the preStart method.

## Stop event ##

An actor can be stopped using the stop method on the
ActorSystem and ActorContext objects, or by sending a PoisonPill
message to an actor.

The postStop hook is called just before the actor is terminated.

A stopped actor is disconnected from its ActorRef. After the actor is
stopped, the ActorRef is redirected to the deadLetters ActorRef of the actor
system, which is a special ActorRef that receives all messages that are sent to dead
actors.

## Restart event ##

When a restart occurs the preRestart method of the crashed actor instance
is called.

~~~~~~
override def preRestart(reason: Throwable, message: Option[Any]) {
  println("preRestart")
  super.preRestart(reason, message)
}
~~~~~~
The default implementation of the
preRestart method stops all the child actors of the actor and then calls the
postStop hook. If you forget to call super.preRestart this default
behavior will not occur.

Remember that actors are (re)created from a Props
object. The Props object eventually calls the constructor of the actor.

A stopped actor is disconnected from its
ActorRef and redirected to the deadLetters ActorRef as described by the stop
event.

The preRestart method takes two arguments: the reason for the restart and
optionally the message that was being processed when the actor crashed. 

The super.postRestart can be omitted if you are certain that you
don't want the preStart to be called when restarting, in most cases though this
is not going to be the case.

## Putting the Life cycle Pieces Together ##


![akka_lifecycle](/img/akka_lifecycle.png)

~~~~~~
class LifeCycleHooks extends Actor with ActorLogging{
  System.out.println("Constructor")
  override def preStart() {println("preStart")}
  override def postStop() {println("postStop")}
  override def preRestart(reason: Throwable, message: Option[Any]) {
    println("preRestart")
    super.preRestart (reason, message)
  }
  override def postRestart(reason: Throwable) {
    println("postRestart")
    super.postRestart(reason)
  }
  def receive = {
    case "restart" => throw new IllegalStateException("force restart")
    case msg: AnyRef => println("Receive")
  }
}

~~~~~~

# Monitoring the lifecycle #

The supervision hierarchy is fixed for the lifetime of a child. Once the child is
created by the parent it will fall under the supervision of that parent as long as it
lives, there is no such thing as adoption in Akka.

~~~~~~
object LogProcessingApp extends App {
  val sources = Vector("file:///source1/", "file:///source2/")
  val system = ActorSystem("logprocessing")
  // create the props and dependencies
  val con = new DbCon("http://mydatabase")
  val writerProps = Props(new DbWriter(con))
  val dbSuperProps = Props(new DbSupervisor(writerProps))
  val logProcSuperProps = Props(new LogProcSupervisor(dbSuperProps))
  val topLevelProps = Props(new FileWatchingSupervisor(
    sources,logProcSuperProps))
  system.actorOf(topLevelProps)
}

~~~~~~

Props objects are passed as recipes to the actors so that
they can create their children without knowing the details of
the dependencies of the child actors.

## Predefined strategies ##

 There are two predefined strategies available in the
SupervisorStrategy object; the defaultStrategy and the
stoppingStrategy .

~~~~~~
final val defaultStrategy: SupervisorStrategy = {
  def defaultDecider: Decider = { //
    case _: ActorInitializationException => Stop //
    case _: ActorKilledException => Stop
    case _: Exception => Restart
  }
  OneForOneStrategy()(defaultDecider) //
}

~~~~~~

In some cases you might want to only stop the child actor that failed.
In other cases you might want to stop all child actors if one of them fails, maybe because they all
depend on a particular resource.

 The OneForOneStrategy determines that child actors will not share the
same fate, only the crashed child will be decided upon by the Decider. The other
option is to use an AllForOneStrategy which uses the same decision for all
child actors even if only one crashed.

~~~~~~
final val stoppingStrategy: SupervisorStrategy = {
  def stoppingDecider: Decider = {
    case _: Exception => Stop //
  }
  OneForOneStrategy()(stoppingDecider)
}

~~~~~~

## Custom Strategies ##

First we'll look at the exceptions that can occur in the log processing
application.

~~~~~~
@SerialVersionUID(1L)
// An unrecoverable Error that occurs when the disk for the source has crashed
class DiskError(msg: String) extends Error(msg) with Serializable

@SerialVersionUID(1L)
// An Exception that occurs when the log file is corrupt and cannot be processed.
class CorruptedFileException(msg: String, val file: File)
  extends Exception(msg) with Serializable
  
@SerialVersionUID(1L)
// An Exception that occurs when the database connection is broken.
class DbBrokenConnectionException(msg: Striing)
  extends Exception(msg) with Serializable


object LogProcessingProtocol {
  // represents a new log file
  case class LogFile(file: File)
  // A line in the log file parsed by the LogProcessor Actor
  case class Line(time: Long, message: String, messageType: String)
}

class DbWriter(connection: DbCon) extends Actor {
  import LogProcessingProtocol._
  def receive = {
    case Line(time, message, messageType) =>
      connection.write(Map('time -> time,
        'message -> message,
        'messageType -> messageType))
  }
}

/// Send the parsed lines to the dbSupervisor which in turn will forward the message
// to the dbWriter.
class DbSupervisor(writerProps: Props) extends Actor {
  override def supervisorStrategy = OneForOneStrategy() {
    case _: DbBrokenConnectionException => Restart
  }
  val writer = context.actorOf(writerProps)
  def receive = {
    case m => writer forward (m)
  }
}

class LogProcessor(dbSupervisor: ActorRef) extends Actor with LogParsing {
  import LogProcessingProtocol._
  def receive = {
    case LogFile(file) =>
      val lines = parse(file)
      lines.foreach(dbSupervisor ! _)
  }
}


class LogProcSupervisor(dbSupervisorProps: Props)
    extends Actor {
  override def supervisorStrategy = OneForOneStrategy() {
    case _: CorruptedFileException => Resume
  }
  val dbSupervisor = context.actorOf(dbSupervisorProps)
  val logProcProps = Props(new LogProcessor(dbSupervisor))
  val logProcessor = context.actorOf(logProcProps)
  def receive = {
    case m => logProcessor forward (m)
  }
}

class FileWatcher(sourceUri: String, logProcSupervisor: ActorRef)
    extends Actor with FileWatchingAbilities {
  // Registers on a source uri in the file watching API.
  register(sourceUri)
  import FileWatcherProtocol._
  import LogProcessingProtocol._
  def receive = {
    case NewFile(file, _) =>
      logProcSupervisor ! LogFile(file)
    case SourceAbandoned(uri) if uri == sourceUri =>
      self ! PoisonPill
  }
}

class FileWatchingSupervisor(sources: Vector[String], logProcSuperProps: Props)
    extends Actor {
  var fileWatchers: Vector[ActorRef] = sources.map { source =>
    val logProcSupervisor = context.actorOf(logProcSuperProps)
    //  Watch the file watchers for termination.
    val fileWatcher = context.actorOf(Props(
          new FileWatcher(source, logProcSupervisor)))
    context.watch(fileWatcher)
    fileWatcher
  }
  override def supervisorStrategy = AllForOneStrategy() {
    case _: DiskError => Stop
  }
  def receive = {
    case Terminated(fileWatcher) =>
      fileWatchers = fileWatchers.filterNot(w => w == fileWatcher)
      if (fileWatchers.isEmpty) self ! PoisonPill
  }
}


class DbImpatientSupervisor(writerProps: Props) extends Actor {
  // Escalate the issue if the problem has not been resolved within 60 seconds or it has
  // failed to be solved within 5 restarts.
  override def supervisorStrategy = OneForOneStrategy(
      maxNrOfRetries = 5,
      withinTimeRange = 60 seconds) {
    case _: DbBrokenConnectionException => Restart
  }
  val writer = context.actorOf(writerProps)
  
  def receive = {
    case m => writer forward (m)
  }
}

~~~~~~

# Futures #

TODO
