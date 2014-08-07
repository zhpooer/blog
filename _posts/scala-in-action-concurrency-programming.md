title: Scala in Action-Concurrency programming
date: 2014-07-23 20:17:00
tags:
- scala
---

Think of an actor as an object that processes a message (your request) and
encapsulates state (state is not shared with other actors).
The ability to perform an action in response to an incoming message
is what makes an object an actor. The actor model encourages no shared state architecture. 

Think of Future as a proxy object that you can create for a result that will be avail-
able at some later time. You can use Promise to complete a Future by providing the
result. 

# What is concurrent programming? #
Concurrency is when more than one task can start and complete in overlapping time
periods.

In parallel programming, you can literally run multiple tasks
at the same time, and it’s possible with multicore processors.
A concurrent program sometimes becomes a parallel program when it’s running in a multicore environment.

But it’s hard to write a correct and bug-free concurrent program.
* Only a handful of programmers know how to write a correct,
concurrent application or program. The correctness of the program is important.
* Debugging multithreaded programs is difficult. The same program that causes
deadlock in production might not have any locking issues when debugging
locally. Sometimes threading issues show up after years of running in production.
* Threading encourages shared state concurrency, and it’s hard to make
programs run in parallel because of locks, semaphores, and dependencies between
threads.

# New trends in concurrency #

* STM, Software transactional memory
* Dataflow concurrency, The principle behind the dataflow concurrency is to share variables across multiple
tasks or threads.
* Message passing concurrency, In this concurrency
model, components communicate by sending messages. Messages can be sent
both synchronously and asynchronously, but asynchronously sending messages to
other components is more common. 

# Implementing message-passing concurrency with actors #
In this concurrency model, actors communicate with each other through sending and
receiving messages. An actor processes incoming messages and executes the actions
associated with it. Typically, these messages are immutable because you shouldn’t
share state between them for reasons discussed previously.

There are two main communication abstractions in actor: send and receive.
~~~~~~
// sending msg to actor a
a ! msg

// receive operation
receive {
  case pattern1 =>
  ...
  case pattern =>
}
~~~~~~

~~~~~~
import akka.actor.Actor
case class Name(name: String)
class GreetingsActor extends Actor {
  def receive = {
    case Name(n) => println("Hello " + n)
  }
}
~~~~~~
Before sending any messages to the GreetingsActor actor, the actor needs to be
instantiated by creating an ActorSystem. Think of an ActorSystem as the manager of
one or more actors.
~~~~~~
// libraryDependencies += "com.typesafe.akka" %% "akka-actor" % "2.1.0"
import akka.actor.Props
import akka.actor.ActorSystem

val system = ActorSystem("greetings")

val a = system.actorOf(Props[GreetingsActor], name = "greetings-actor")
a ! Name("Nilanjan")
// shuts down the infrastructure and all its actors.
Thread.sleep(50)
system.shutdown()
~~~~~~

## ActorSystem ##

An actor system is a hierarchical group of actors that share a common configuration.
It’s also the entry point for creating and looking up actors.

Similarly actors form a hierarchy where parent actors
spawn child actors to delegate work until
it is small enough to be handled by an individual actor.

An ActorSystem is a heavyweight structure that will allocate 1. . .N
threads, so create one per logical subsystem of your application. For example,
you can have one actor system to handle the backend database, another to
handle all the web service calls, and so forth. Actors are very cheap. A given
actor consumes only 300 bytes so you can easily create millions of them.

At the top of the hierarchy is the guardian actor, created automatically with each actor
system. All other actors created by the given actor system become the child of the
guardian actor. In the actor system, each actor has one supervisor (the parent actor)
that automatically takes care of the fault handling. So if an actor crashes, its parent
will automatically restart that actor (more about this later).

The simplest way to create an actor is to create an ActorSystem and use its actorOf
method:

~~~~~~
val system = ActorSystem(name = "word-count")
val m: ActorRef = system.actorOf(Props[SomeActor],
      name = "someActor")
~~~~~~

Note here that when you create an actor in Akka, you never get the direct refer-
ence to the actor. Instead you get back a handle to the actor called ActorRef.
The foremost purpose of ActorRef is to send messages to the actor it rep-
resents.

An actor path uniquely identifies an actor in the actor system.
Because actors are created in a hierarchical structure, they form a similar
structure to a filesystem. 
As a path in a filesystem points to an individual resource,
an actor path identifies an actor reference in an actor system.

Uses the system / method to retrieve the actor
reference of the WordCountWorker actor:
~~~~~~
class WordCountWorker extends Actor { ... }
...
val system = ActorSystem(name = "word-count")
system.actorOf(Props[WordCountWorker], name = "wordCountWorker")
...
val path: ActorPath = system / "WordCountWorker"
val actorRef: ActorRef = system.actorFor(path)
actorRef ! “some message”
~~~~~~
If the actorFor fails to find an actor
pointed to by the path, it returns a reference to the dead-letter mailbox of the actor
system. It’s a synthetic actor where all the undelivered messages are delivered.

The parent actor first stops all the child actors and sends all unprocessed messages to the dead-letter mailbox before ter-
minating itself. 

### How do Scala actors work? ###
Every actor system comes with a default MessageDispatcher component.
Its responsibility is to send a message to the actor’s mailbox
and execute the actor by invoking the receive block.

Every MessageDispatcher is backed by a thread pool, which is easily
configured using the configuration file.

To send a message to an actor mail-box the ActorRef
* first sends the message to the MessageDispatcher associated with the actor.
* The MessageDispatcher immediately queues the message in the mailbox of the
actor.
* The control is immediately returned to the sender of the message. 

Handling a message is a bit more involved
1. When an actor receives a message in its mailbox, MessageDispatcher schedules
the actor for execution. Sending and handling messages happens in two differ-
ent threads. If a free thread is available in the thread pool that thread is
selected for execution of the actor. If all the threads are busy, the actor will be
executed when threads becomes available.
2. The available thread reads the messages from the mailbox.
3. The receive method of the actor is invoked by passing one message at a time.

# Divide and conquer using actors #

In the following example, the challenge is to count the number of words in each file
in a given directory and sort them in ascending order. One way of doing it would be to
loop through all the files in a given directory in a single thread, count the words in
each file, and sort them all together. But that’s sequential. To make it concurrent, we
will implement the divide-and-conquer (also called a fork-join) pattern with actors.

Actor API
~~~~~~
// If a given message doesn’t match any pattern inside the receive method then the
// unhandled method is called with the akka.actor.UnhandledMessage message. 
def unhandled(message: Any): Unit

// This field holds the actor reference of this actor. 
val self: ActorRef

// This is the ActorRef of the actor that sent the last received message. 
final def sender: ActorRef

// This provides the contextual information for the actor, the current message, and the
// factory methods to create child actors. 
val context: ActorContext

// This supervisor strategy defines what will happen when a failure is detected in an
// actor. You can override to define your own supervisor strategy.
def supervisorStrategy: SupervisorStrategy

def preStart()

// his method is called on the current instance of the actor. This is a great place to
// clean up. The default implementation is to stop all the child actors and then invoke
// the postStop method.
def preRestart()

// This method is called after the current actor instance is stopped.
def postStop()

// Then the postRestart is invoked on the fresh instance.
// The default implementation is to invoke the preStart method
def postRestart()
~~~~~~

you’ll create two actor classes: one that
will scan the directory for all the files and accumulate results, called WordCountMaster,
and another one called WordCountWorker to count words in each file. 

~~~~~~
// The docRoot will specify the location of the files and
// numActors will create the number of worker actors. 
case class StartCounting(docRoot: String, numActors: Int)

// Message Class
case class FileToCount(fileName:String)
case class WordCount(fileName:String, count: Int)

class WordCountWorker extends Actor {
  def countWords(fileName:String) = {
    val dataFile = new File(fileName)
    Source.fromFile(dataFile).getLines.foldRight(0)(_.split(" ").size + _)
  }
  def receive = {
    case FileToCount(fileName:String) =>
      val count = countWords(fileName)
      sender ! WordCount(fileName, count)
  }
  override def postStop(): Unit = {
    println(s"Worker actor is stopped: ${self}")
  }
}

class WordCountMaster extends Actor {
  var fileNames: Seq[String] = Nil
  var sortedCount : Seq[(String, Int)] = Nil
  def receive = {
    case StartCounting(docRoot, numActors) =>
      val workers = createWorkers(numActors)
      fileNames = scanFiles(docRoot)
      beginSorting(fileNames, workers)
      
    case WordCount(fileName, count) =>
      sortedCount = sortedCount :+ (fileName, count)
      sortedCount = sortedCount.sortWith(_._2 < _._2)
      if(sortedCount.size == fileNames.size) {
        println("final result " + sortedCount)
        finishSorting()
      }
  }
  private def createWorkers(numActors: Int) = {
    for (i <- 0 until numActors) yield
    context.actorOf(Props[WordCountWorker], name = s"worker-${i}")
  }
  private def scanFiles(docRoot: String) =
    new File(docRoot).list.map(docRoot + _)
    
  private[this] def beginSorting(fileNames: Seq[String], workers: Seq[ActorRef]) {
    fileNames.zipWithIndex.foreach( e => {
      workers(e._2 % workers.size) ! FileToCount(e._1)
    })
  }
  
  private[this] def finishSorting() {
    context.system.shutdown()
  }
}

def main(args: Array[String]) {
  val system = ActorSystem("word-count-system")
  val m = system.actorOf(Props[WordCountMaster], name="master")
  m ! StartCounting("src/main/resources/", 2)
}

~~~~~~


## ActorDSL ##

~~~~~~
import akka.actor.ActorDSL._

val testActor = actor(new Act {
  become {
    case "ping" => sender ! "pong"
  }
})

// Behind the scene Act extends the Actor trait and become adds the behavior of the receive block.
// Using this DSL syntax you no longer have to create a class.

object ActorDSLExample extends App {
  import akka.actor.ActorDSL._
  import akka.actor.ActorSystem
  implicit val system = ActorSystem("actor-dsl")
  val testActor = actor(new Act {
    become {
      case "ping" => sender ! "pong"
    }
  })
  actor(new Act {
    whenStarting { testActor ! "ping"}
    become {
      case x =>
        println(x)
        context.system.shutdown()
    }
  })
}
~~~~~~
# Fault tolerance made easy with a supervisor #
Think of this supervisor as an actor that links to supervised actors and restarts
them when one dies.

That way, if a node (machine) is down,
you can restart an actor in a different box. Always
remember to delegate the work so that if a crash occurs,
another supervisor can recover.

Akka comes with two restarting strategies: One-for-One and All-for-One.

In the One-for-One strategy, if one actor dies, it’s recreated.

If you have multiple actors that participate in one workflow, restarting a single actor
might not work. In that case, use the All-for-One restart strategy,
in which all actors supervised by a supervisor are restarted
when one of the actors dies.

When no supervisor strategy is defined, it uses the default strategy (OneForOne ),
which restarts the failing child actor in case of Exception.



~~~~~~
import akka.actor.SupervisorStrategy._
class WordCountWorker extends Actor {
  //if no pattern matches, the fault is escalated to the parent. 
  override val supervisorStrategy = OneForOneStrategy(maxNrOfRetries = 3,
        withinTimeRange = 5 seconds) {
    case _: Exception => Restart
  }
}

class WordCountMaster extends Actor {
  override val supervisorStrategy = AllForOneStrategy() {
    case _: Exception =>
      println("Restarting...")
      Restart
  }
}

~~~~~~

# Future and Promise #

A Future is an object that can hold a value that may become available, as its name sug-
gests, at a later time.
~~~~~~
import ExecutionContext.Implicits.global

def someFuture[T]: Future[T] = Future {
  someComputation()
}
~~~~~~

Since the Future is executed asynchronously we need to specify the
scala.concurrent.ExecutionContext. ExecutionContext is an abstraction over a
thread pool that is responsible for executing all the tasks submitted to it.

~~~~~~
someFuture.onComplete {
  case Success(result) => println(result)
  case Failure(t) => t.printStackTrace
}

val promise: Promise[String] = Promise[String]()
val future = promise.future
...
val anotherFuture = Future {
  promise.success("Done")
  doSomethingElse()
}

future.onSuccess { case msg => startTheNextStep() }
~~~~~~

A common use case of Future is to perform some computation concurrently without
needing the extra utility of an actor. The most compelling feature of the Scala Future
library is it allows us to compose concurrent operations, which is hard to achieve with
actors.

word count use future
~~~~~~
import scala.concurrent._
import ExecutionContext.Implicits.global
import scala.util.{Success, Failure}
import java.io.File
import scala.io.Source

object main {
  def main(args: Array[String]) {
    val promiseOfFinalResult = Promise[Seq[(String, Int)]]()
    val path = "src/main/resources/"
    val futureWithResult: Future[Seq[(String, Int)]] = for {
      files <- scanFiles(path)
      result <- processFiles(files)
    } yield {
      result
    }
    
    futureWithResult.onSuccess {
      case r => promiseOfFinalResult.success(r)
    }
    promiseOfFinalResult.future.onComplete {
      case Success(result) => println(result)
      case Failure(t) => t.printStackTrace
    }
  }
  
  private def processFiles(fileNames: Seq[String]): Future[Seq[(String,Int)]] = {
    val futures: Seq[Future[(String, Int)]] = fileNames.map(name =>
        processFile(name))
    val singleFuture: Future[Seq[(String, Int)]] =
    Future.sequence(futures)
    singleFuture.map(r => r.sortWith(_._2 < _._2))
  }
  private def processFile(fileName: String): Future[(String, Int)] =
  Future {
    val dataFile = new File(fileName)
    val wordCount = Source.fromFile(dataFile).getLines.foldRight(0)(_.split("
    ").size + _)
    (fileName, wordCount)
  } recover {
    case e: java.io.IOException =>
    println("Something went wrong " + e)
    (fileName, 0)
  }
  
  private def scanFiles(docRoot: String):Future[Seq[String]] = Future { new
    File(docRoot).list.map(docRoot + _) }
}
~~~~~~

## Mixing Future with actors ##

* Send a message to an actor and receive a response from it. So far we have only
used fire-and-forget using the ! method. But getting a response is also a very
common use case (a.k.a ask pattern).
* Reply to sender when some concurrent task ( Future) completes (a.k.a pipe
pattern).

~~~~~~
import akka.pattern.{ask, pipe}
implicit val timeout = Timeout(5 seconds)
class GreetingsActor extends Actor {
  val messageActor = context.actorOf(Props[GreetingsChildActor])
  def receive = {
    case name =>
      val f: Future[String] = (messageActor ask name).mapTo[String]
      f pipeTo sender
  }
}
class GreetingsChildActor extends Actor {
  def receive = { ...
  }
}
~~~~~~
In this case we are using the ask method (you can use ? as well) of
the ActorRef to send and receive a response. Since messages are processed
asynchronously the ask method returns a Future.
The mapTo message allows us to transform
the message from Future[Any] to Future[String]. 

The challenge is we don’t know
when the message will be ready so that we can send the reply to the sender. The
pipeTo pattern solves that problem by hooking up with the Future so that when the
future completes it can take the response inside the future and send it to the sender.
