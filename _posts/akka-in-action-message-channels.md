title: Akka in Action-Message Channels & Finite State Machines
date: 2014-08-05 11:11:47
tags:
- akka
- scala
---

# Chinnel types #

Other names which are often used for these kind of channels are EventQueue or
EventBus. Akka has an EventStream which implements a publish-subscribe
channel. But when this implementation isn't sufficient, then Akka has a collection
of traits which helps to implement an custom publish subscribe channel.

Next we describe two special channels. The first is the Dead Letter channel,
which contain message that couldn't be delivered.
This channel can help when debugging, why some messages
aren't processed or to monitor where there are problems.


## Point to Point ##

The point-to-point channel sends the message to one receiver.

![point 2 point img](/img/akka_point2point.png)

The round-robin
Router in section 7.3.1 is an example of the channel having multiple receivers. The
processing of the messages can be done concurrently by different Receivers, but
only one Receiver consumes any one message.

Because in Akka the ActorRef is the implementation of a point-to-point channel.
Because all the messages send will be delivered to one Actor.

## Publish subscribe ##

The channel can also deliver the same message to multiple receivers.

To solve this problem we can use
the Publish-subscribe channel. The channel is able to send the same message to
multiple receivers, without the sender knows which receiver.

When a receiver is interested in a message of the publisher, it subscribes itself
to the channel.

The most easiest when needed a publish-subscribe channel,
is to use the EventStream.The EventStream can be seen as a
manager of multiple Publish-Subscribe channels.

~~~~~~
// subscirbe to the EventStream to receive Order messages
system.eventStream.subscribe(giftModule, classOf[Order])
// unsubscribe
system.eventStream.unsubscribe(giftModule, classOf[Order])

// 取消订阅所有消息
system.eventStream.unsubscribe(giftModule.ref)
// 发布消息
system.eventStream.publish(msg)
~~~~~~

## CUSTOM EVENTBUS ##

Let assume that we only want to send a gift when someone ordered more than one
book.

 An EventBus is generalized so
that it can be used for all implementations of a publish-subscribe channel. In the
generalized form there are three entities.
* Event, This is the type of all events published on that bus. In the Akka EventStream all uses
AnyRef as event and therefor supports all type of messages
* Subscriber, This is the type of subscribers allowed to register on that event bus. In the Akka
EventStream the subscribers are ActorRef's
* Classifier, This defines the classifier to be used in selecting subscribers
for dispatching events. 

EventBus Interface
~~~~~~
package akka.event
trait EventBus {
  type Event
  type Classifier
  type Subscriber
  /**
  * Attempts to register the subscriber to the specified Classifier
  * @return true if successful and false if not (because it was
  * already subscribed to that Classifier, or otherwise)
  */
  def subscribe(subscriber: Subscriber, to: Classifier): Boolean
  /**
  * Attempts to deregister the subscriber from the specified Classifier
  * @return true if successful and false if not (because it wasn't
  * subscribed to that Classifier, or otherwise)
  */
  def unsubscribe(subscriber: Subscriber, from: Classifier): Boolean
  /**
  * Attempts to deregister the subscriber from all Classifiers it may
  * be subscribed to
  */
  def unsubscribe(subscriber: Subscriber): Unit
  /**
  * Publishes the specified Event to this bus
  */
  def publish(event: Event): Unit
}
~~~~~~

~~~~~~
class OrderMessageBus extends EventBus {
  type Event = Order
  // chosen to classify the Order messages on the criteria "is Multiple
  // Book Order" and use a Boolean as classifier 
  type Classifier = Boolean

}
~~~~~~

Akka has three composable traits which can help in keeping
track of the subscribers.
* LookupClassification,  It maintain a set of subscribers for each
possible classifier and extract a classifier from each event.
* SubchannelClassification, This trait is used when classifiers form a hierarchy
and it is desired that subscription can be possible not only at the leaf nodes,
but also to the higher nodes.
* ScanningClassification,  it can be used when classifiers have an overlap. This
means that one Event can be part of more classifiers, for example if we give more gifts
when ordering more books.

API of LookupClassification
* classify(event: Event): Classifier
This is used for extracting the classifier from the incoming events.
* compareSubscribers(a: Subscriber, b: Subscriber): Int
This method must define a order over the subscribers, to be able to compare them just as
the java.lang.Comparable.compare method.
* publish(event: Event, subscriber: Subscriber),
This method will be invoked for each event for all subscribers which registered
themselves for the events classifier.
* mapSize: Int, This returns the expected number of the different classifiers.
This is used for the initial size of an internal data structure.


~~~~~~
import akka.event.ActorEventBus
import akka.event.{ LookupClassification, EventBus }

class OrderMessageBus extends EventBus with LookupClassification
    with ActorEventBus {  //  defines that the subscriber is an ActorRef.
  type Event = Order
  type Classifier = Boolean
  def mapSize = 2
  
  protected def classify(event: StateEventBus#Event) = {
    event.number > 1
  }
  
  // publish method by sending the event to the subscriber
  protected def publish(event: OrderMessageBus#Event,
      subscriber: OrderMessageBus#Subscriber) {
    subscriber ! event
  }

}


// Test for event bus
val bus = new OrderMessageBus
val singleBooks = TestProbe()
bus.subscribe(singleBooks.ref, false)

val multiBooks = TestProbe()
bus.subscribe(multiBooks.ref, true)

val msg = new Order("me", "Akka in Action", 1)
bus.publish(msg)

singleBooks.expectMsg(msg)
multiBooks.expectNoMsg(3 seconds)

val msg2 = new Order("me", "Akka in Action", 3)
bus.publish(msg2)
singleBooks.expectNoMsg(3 seconds)
multiBooks.expectMsg(msg2)
~~~~~~

# Specialize Channel #

DeadLetter channel,  Only failed message are put on this channel. Listening on this
channel can help to find problems in your system.

Guaranteed deliver channel, his channel guaranties
all messages which are send are also delivered

## Dead letter ##
By monitoring this channel you know which messages aren't processed and can
take corrective actions.

 To get these dead letter messages you
only need to subscribe your actor to the EventStream with the DeadLetter class as
the Classifier.

~~~~~~
val deadLetterMonitor: ActorRef

system.eventStream.subscribe(
  deadLetterMonitor,
  classOf[DeadLetter])

// 测试代码
val deadLetterMonitor = TestProbe()
system.eventStream.subscribe(
    deadLetterMonitor.ref,
    classOf[DeadLetter])
    
val actor = system.actorOf(Props[EchoActor], "echo")
actor ! PoisonPill
val msg = new Order("me", "Akka in Action", 1)

actor ! msg

//  wrapped also into a DeadLetter object
val dead = deadLetterMonitor.expectMsgType[DeadLetter]
dead.message must be(msg)
dead.sender must be(testActor)
dead.recipient must be(actor)
~~~~~~

# Guaranteed delivery #
The guaranteed delivery channel is point-to-point channel with the guaranty that
the message is always delivered to the receiver.

This means that the channel must have
all kind of mechanism and checks to be able to guaranty the delivery, for example
the message has to be saved on disk in case the process crashes.

The general rule of message delivery is that messages are delivered
at-most-once. This means that Akka promise that messages are delivered once or
fails to deliver, Which means that the message is lost.

Sending local messages will not likely fails, because it is like a normal method
call. This fails only when there are catastrophic VM errors, like
StackOverflowError, OutOfMemoryError or a memory access violation.
So the guaranties when sending a message to a local actor, are pretty good
and reliable.

The problem of losing the messages is when using remote actors. When using
remote actors, it is a lot more likely for a message delivery failure to occur.

The Egress is an Actor which is started by the
ReliableProxy and both Actors implements the checks and resend functionality to
be able to keep track of which of the messages are delivered to the remote receiver.

 One restriction of using the ReliableProxy is that the tunnel is only one-way
and for one receiver. This means that when the receiver replies to the sender the
tunnel is NOT used.

~~~~~~
import akka.contrib.pattern.ReliableProxy
val echo = system.actorFor(node(server) / "user" / "echo")
// In the example we create a proxy using the echo reference.
// When failing to send a message it is retried after 500 milliseconds.
val proxy = system.actorOf(Props(new ReliableProxy(echo, 500.millis)), "proxy")
~~~~~~
We create a Multi-node test with two nodes, the client and server
node. On the server Node we create a EchoActor as receiver and on the client node
we run our actual test.
~~~~~~
import akka.remote.testkit.MultiNodeSpecCallbacks
import akka.remote.testkit.MultiNodeConfig
import akka.remote.testkit.MultiNodeSpec

trait STMultiNodeSpec
    extends MultiNodeSpecCallbacks
    with WordSpec
    with MustMatchers
    with BeforeAndAfterAll {
  override def beforeAll() = multiNodeSpecBeforeAll()
  override def afterAll() = multiNodeSpecAfterAll()
}

object ReliableProxySampleConfig extends MultiNodeConfig {
  val client = role("Client")
  val server = role("Server")
  testTransport(on = true)
}

class ReliableProxySampleSpecMultiJvmNode1 extends ReliableProxySample
class ReliableProxySampleSpecMultiJvmNode2 extends ReliableProxySample

~~~~~~
TODO P257

# Using a Finite State Machine #
Finite-state machine (FSM), also called a state machine, is a common,
language-independent modeling technique.

The simplest example of a Finite State Machine is a device whose operation
proceeds through several states, transitioning from one to the next as certain events
occur.

The simplest example of a Finite State Machine is a device whose operation
proceeds through several states, transitioning from one to the next as certain events
occur.

![fsm](/img/akka_FSM.png)

## Creating an FSM Model ##

The inventory Service gets requests for
specific books and sends a reply. When the book is in inventory, the order system
gets a reply that a book has been reserved. But it is possible that there aren't any
books left and that the inventory will have to ask the publisher for more books,
before it can service the order.

![fms](/img/akka_FMS.png)



~~~~~~

// State, The super type of all state names
sealed trait State
case object WaitForRequests extends State
case object ProcessRequest extends State
case object WaitForPublisher extends State
case object SoldOut extends State
case object ProcessSoldOut extends State

// StateData, The type of the state data which are tracked by the FSM.
case class StateData(nrBooksInStore:Int,pendingRequests:Seq[BookRequest])


import akka.actor.{Actor, FSM}
class Inventory() extends Actor with FSM[State, StateData] {
  // define the initial state and the initial StateData.
  startWith(WaitForRequests, new StateData(0,Seq()))

  // Declare the transitions for state WaitForRequests
  when(WaitForRequests) {
    // Declare the possible Event when a BookRequest messages occur
    case Event(request:BookRequest, data:StateData) => 
      val newStateData = data.copy(pendingRequests = data.pendingRequests :+ request)
      
      if (newStateData.nrBooksInStore > 0) {
        goto(ProcessRequest) using newStateData
      } else {
        goto(WaitForPublisher) using newStateData
      }
    case Event(PendingRequests, data:StateData) => 
      if (data.pendingRequests.isEmpty) {
        stay
      } else if(data.nrBooksInStore > 0) {
        goto(ProcessRequest)
      } else {
        goto(WaitForPublisher)
      }
  }

  when(WaitForPublisher) {
    case Event(supply:BookSupply, data:StateData) => {
      goto(ProcessRequest) using data.copy(nrBooksInStore = supply.nrBooks)
    }
    case Event(BookSupplySoldOut, _) => {
      goto(ProcessSoldOut)
    }
  }

  when(ProcessRequest) {
    case Event(Done, data:StateData) => {
      goto(WaitForRequests) using data.copy(
          nrBooksInStore = data.nrBooksInStore - 1,
          pendingRequests = data.pendingRequests.tail)
    }
  }

  when(SoldOut) {
    case Event(request:BookRequest, data:StateData) => {
      goto(ProcessSoldOut) using new StateData(0,Seq(request))
    }
  }

  when(ProcessSoldOut) {
    case Event(Done, data:StateData) => {
      goto(SoldOut) using new StateData(0,Seq())
    }
  }

  whenUnhandled {
    // common code for all states
    case Event(request:BookRequest, data:StateData) => {
      // Only update the stateData
      stay using data.copy(pendingRequests = data.pendingRequests :+ request)
    }
    // Log when the event isn't handled
    case Event(e, s) => {
      log.warning("received unhandled request {} in state {}/{}", e, stateName, s)
      stay
    }
  }

  // entry Action of the WaitForRequests state
  onTransition {
    case _ -> WaitForRequests => {
      if (!nextStateData.pendingRequests.isEmpty) {
        // go to next state
        self ! PendingRequests
      }
    }
    
    case _ -> WaitForPublisher => {
      publisher ! PublisherRequest
    }
    
    case _ -> ProcessRequest => {
      val request = nextStateData.pendingRequests.head
      reserveId += 1
      request.target ! new BookReply(request.context, Right(reserveId))
      self ! Done
    }
    
    case _ -> ProcessSoldOut => {
      nextStateData.pendingRequests.foreach(request => {
        request.target ! new BookReply(request.context, Left("SoldOut"))
      })
      self ! Done
    }
  }

}

~~~~~~

## TESTING THE FSM ##


~~~~~~
class Publisher(totalNrBooks: Int, nrBooksPerRequest: Int)
    extends Actor {
  var nrLeft = totalNrBooks
  def receive = {
    case PublisherRequest => {
      if (nrLeft == 0)
      sender ! BookSupplySoldOut
      else {
        val supply = min(nrBooksPerRequest, nrLeft)
        nrLeft -= supply
        sender ! new BookSupply(supply)
      }
    }
  }
}

val publisher = system.actorOf(Props(new Publisher(2,2)))
val inventory = system.actorOf(Props(new Inventory(publisher)))
val stateProbe = TestProbe()

// 订阅状态变化信息
inventory ! new SubscribeTransitionCallBack(stateProbe.ref)
stateProbe.expectMsg(new CurrentState(inventory, WaitForRequests))

inventory ! new BookRequest("context1", replyProbe.ref)
stateProbe.expectMsg(new Transition(inventory, WaitForRequests, WaitForPublisher))
stateProbe.expectMsg(new Transition(inventory, WaitForPublisher, ProcessRequest))
stateProbe.expectMsg(new Transition(inventory, ProcessRequest, WaitForRequests))
replyProbe.expectMsg(new BookReply("context1", Right(1)))

~~~~~~

## Timers within FSM ##

When it is in the state 'WaitingForPublisher,'
we don't wait forever for the publisher to reply.

~~~~~~
when(WaitForPublisher, stateTimeout = 5 seconds) {
  case Event(supply:BookSupply, data:StateData) => {
    goto(ProcessRequest) using data.copy(nrBooksInStore = supply.nrBooks)
  }
  case Event(BookSupplySoldOut, _) => {
    goto(ProcessSoldOut)
  }
  // Define the timeout transition
  case Event(StateTimeout,_) => goto(WaitForRequests)
}

// 当测试时
// stateProbe.expectMsg(6 seconds,new Transition(inventory, WaitForPublisher, WaitForRequests))

~~~~~~

The timer can also be set by specifying the next state using the
method forMax.
~~~~~~
goto(WaitForPublisher) using (newData) forMax (5 seconds)
~~~~~~

## Termination of FSM ##
The FSM has an
specific handler for these cases: onTermination. This handler is also a partial
function and takes a StopEvent as an argument.

There are three possible reasons this can be received.
* Normal. This is received when there is a normal termination.
* Shutdown. This is received when the FSM is stopped due to a shutdown.
* Failure(cause: Any), This reason is received when the termination was caused by a failure

~~~~~~
onTermination {
  case StopEvent(FSM.Normal, state, data)
  case StopEvent(FSM.Shutdown, state, data)
  case StopEvent(FSM.Failure(cause), state, data)
}
~~~~~~

# Implement Shared state using agents #
Akka accomplishes this by sending
actions to the agent for each operation, where the messaging infrastructure will
preclude a race condition.

For our example, we need to share the number
of copies sold for each book, so we will create an Agent that contains this value.

~~~~~~
case class BookStatics(val nameBook: String, nrSold: Int)
// a BookStatics instance is created which is put into a map using the title as the key
case class StateBookStatics(val sequence: Long, books: Map[String, BookStatics])

~~~~~~

The state object contained by the agent must be immutable.

~~~~~~
import scala.concurrent.ExecutionContext.Implicits.global
import akka.agent.Agent
val stateAgent = new Agent(new StateBookStatics(0,Map()))

val currentBookStatics = stateAgent.get // 使用 stateAgent() 效果一样

// 如果 agent 的值是依赖前一个状态的呢?
val newState = StateBookStatics(1, Map(book -> bookStat ))
stateAgent send newState

// 可以这样
val book = "Akka in Action"
val nrSold = 1
stateAgent send( oldState => {
  val bookStat = oldState.books.get(book) match {
    case Some(bookState) =>
      bookState.copy(nrSold = bookState.nrSold + nrSold)
    case None => new BookStatics(book, nrSold)
  }
  oldState.copy(oldState.sequence+1, oldState.books + (book -> bookStat ))
})
~~~~~~

## Waiting for the state update ##

In some cases, we need to update shared state and use the new state. For example,
we need to know which book is selling the most, and when a book becomes
popular, we want to notify the authors.

~~~~~~
implicit val timeout = Timeout(1000)

// It works exactly as the send method only it returns a Future,
// which can be used to wait for the new state.
val future = stateAgent alter( oldState => {
  val bookStat = oldState.books.get(book) match {
    case Some(bookState) =>
      bookState.copy(nrSold = bookState.nrSold + nrSold)
    case None =>
      new BookStatics(book, nrSold)
  }
  oldState.copy(oldState.sequence+1,oldState.books + (book -> bookStat ))
})

val newState = Await.result(future, 1 second)
~~~~~~

 It is possible that there are multiple changes at
nearly the same time and we want the final state or another thread needs the final
state and only knows that the process before it may have updated the state. 
~~~~~~
val future = stateAgent.future
val newState = Await.result(future, 1 second)
~~~~~~

~~~~~~
import scala.concurrent.ExecutionContext.Implicits.global
val agent1 = Agent(3)
// When using this notation, agent2 is a newly created Agent that contains the
// value 4 and agent1 is just the same as before (it still contains the value 3).
val agent2 = agent1 map (_ + 1)
~~~~~~
