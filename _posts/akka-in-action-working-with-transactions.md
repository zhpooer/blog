title: Akka in Action-Working with Transactions
date: 2014-08-06 09:47:35
tags:
- akka
- scala
---

# Software Transactional Memory #

We have an event and we have a number of seats that multiple
threads want to lay claim to. Our shared data is a list of seats

~~~~~~
case class Seat(seatNumber:Int)
val availableSeats: Seq[Seat])

// When we want to get a seat from the list we need to
// get the first available seat and update the list.
val head = availableSeats.head
availableSeats = availableSeats.tail
~~~~~~

When we can't or want to use immutable messages
and just want to protect the shared data from becoming inconsistent.

## Protecting Shared data ##
The most common solution to protecting shared data is that when a thread wants to
access the shared data, we block all other threads from accessing the shared
structure.

~~~~~~
val reservedSeat = availableSeats.synchronized {
  head = availableSeats.head
  availableSeats = availableSeats.tail
  head
}
~~~~~~

A problem with this is that when a thread only wants to read all available seats
it still has to lock the list too.

All this locking decreases the performance of the system.

This is called "pessimistic locking”.

Clearly, since there is 'pessimistic locking,' there must also be 'optimistic
locking.'

~~~~~~
import concurrent.stm.Ref
val availableSeats = Ref(Seq[Seat]())

//  update our availableSeats we can write
availableSeats() = availableSeats().tail

// When we want to protect the seat list, we get the following 
import concurrent.stm._
val availableSeats = Ref(seats)
val reservedSeat = atomic {implicit txn => {
  val head = availableSeats().head
  availableSeats() = availableSeats().tail
  head
}}

~~~~~~
The critical section
will be executed only once when using synchronized, but using the STM atomic,
the critical section can be executed more than once. This is because at the end of
the block's execution, a check is done to see if there was a collision. 

## Using the STM transactions ##

But when we
want to do a simple read of shared data we need to create an atomic block and this
means writing a lot of code just for a single, simple read. When using only one
reference, you could also use the View of a reference. The Ref.View enables you
to execute one action on one Reference.

~~~~~~
availableSeats.single.get

val mySeat = atomic {implicit txn => {
    val head = availableSeats().head
    availableSeats() = availableSeats().tail
    head
  }}
}

val myseat = availableSeats.single.getAndTransform(_.tail).head
~~~~~~
Using the Ref.View method makes the code a little bit more compact and also
makes the critical section smaller, which decreases the chance of a collision,
improving the total performance of the system.

~~~~~~
// 硬重试
// When the availableSeat list is empty, we call the retry,
// which triggers to execute the alternative atomic block.
val availableSeats = Ref(Seq[Seat]())
val mySeat = atomic { implicit txn => {
  val allSeats = availableSeats()
  if (allSeats.isEmpty)
    retry
  val reservedSeat = allSeats.head
  availableSeats() = allSeats.tail
  Some(reservedSeat)
}}.orAtomic {implicit txn => {
  //else give up and do nothing
  // return a None to indicate we were unable to get a seat.
  None
}}
mySeat must be (None)

~~~~~~

# Agents within transactions #

When an Agent is used within a transaction, it isn't necessary to wrap it with an
STM reference to be able to use its data.

Our competing thread will update the agent every 50 ms and our test thread
tries to read the Agent's state twice within a transaction. When the Agent's state has
changed in the meantime, the transaction has to be retried.

~~~~~~
// Competing thread updating the agent
val seats = (for (i <- 0 until 15) yield Seat(i))
val availableSeats = Agent(seats)
val future = Future {
  for (i <- 0 until 10) {
    availableSeats send (_.tail)
  }
  Thread.sleep(50)
}

// 获取线程
var nrRuns = 0
val firstSeat = atomic { implicit txn => {
  nrRuns += 1
  val currentList = availableSeats.get
  Thread.sleep(100)
  availableSeats.get.head
}}
Await.ready(future, 1 second)
nrRuns must be > (1)
firstSeat.seatNumber must be (10)
~~~~~~
In this example we see that the critical section is executed more than once,
because the value of the agent has changed during the transaction. 

## Updating Agents within a transaction ##

This means that
when we send an action, the action is held until the transaction is committed and
when if the transaction is rolled back, the action sent to the Agent is also rolled
back.

~~~~~~
val numberUpdates = Agent(0)
val count = Ref(5)
Future {
  for (i <- 0 until 10) {
    atomic { implicit txn => {
      count() = count() +1
    }}
    Thread.sleep(50)
  }
}
var nrRuns = 0
val myNumber = atomic { implicit txn => {
  nrRuns += 1
  numberUpdates send (_ + 1)
  val value = count()
  Thread.sleep(100)
  count()
}}

nrRuns must be > (1)
myNumber must be (15)
Await.ready(numberUpdates.future(), 1 second)
// The agent is only one time updated
numberUpdates.get() must be (1)
~~~~~~

 We can't use Agents and transactions to
solve the problem of transferring money. The problem is that the two actors are
completely unrelated.
~~~~~~
def transfer(from: Agent[Int], to: Agent[Int], amount: Int): Boolean = {
  atomic { txn => 
    if (from.get < amount) false
    else {
      from send (_ - amount)
      to send (_ + amount)
      true
    }
  }
}
~~~~~~

# Actors within transactions #

~~~~~~
import akka.transactor.Coordinated
import scala.concurrent.duration._
import akka.util.Timeout


case class Withdraw(amount:Int)
case class Deposit(amount:Int)
object GetBalance

class InsufficientFunds(msg:String) extends Exception(msg)

class Account() extends Actor {
  val balance = Ref(0)
  
  def receive = {
    case coordinated @ Coordinated(Withdraw(amount)) {
      coordinated atomic { implicit t
        val currentBalance = balance()
        if ( currentBalance < amount) {
          throw new InsufficientFunds( "Balance is too low: "+ currentBalance)
        }
        balance() = currentBalance - amount
      }
    }
    case coordinated @ Coordinated(Deposit(amount)) {
      coordinated atomic { implicit t
        balance() = balance() + amount
      }
    }
    case GetBalance => sender ! balance.single.get
  }
  
  override def preRestart(reason: Throwable, message: Option[Any]) {
    self ! Coordinated(Deposit(balance.single.get))(Timeout(5 seconds))
    super.preRestart(reason, message)
  }
}

// 测试代码
implicit val timeout = Timeout(5 seconds)
val transaction = Coordinated()
transaction atomic { implicit t =>
  account1 ! transaction(Deposit(amount = 100))
}

val probe = TestProbe()
probe.send(account1, GetBalance)
probe.expectMsg(100)

// 更简洁的代码
val account1 = system.actorOf(Props[Account])
implicit val timeout = new Timeout(1 second)

account1 ! Coordinated(Deposit(amount = 100))

val probe = TestProbe()
probe.send(account1, GetBalance)
probe.expectMsg(100)
~~~~~~

~~~~~~
// 转账的代码
def receive = {
  case TransferTransaction(amount, from, to) => {
    val transaction = Coordinated()
    transaction atomic { implicit t
      from ! transaction(Withdraw(amount))
      to ! transaction(Deposit(amount))
    }
    sender ! "done"
  }
}
~~~~~~

## Creating transactors ##

Transactors are actors that are capable of dealing with messages that comprise
Coordinated transactions.

All the functional code will been seen again in the example, only the
Coordinated part is removed, because the transactor will hide it from our code.

~~~~~~
import akka.transactor.Transactor
class AccountTransactor() extends Transactor {
  val balance = Ref(0)
  def atomically = implicit txn => {
    case Withdraw(amount) => {
      val currentBalance = balance()
      if ( currentBalance < amount) {
        throw new InsufficientFunds("Balance is too low: "+ currentBalance)
      }
      balance() = currentBalance - amount
      
    case Deposit(amount) => {
      balance() = balance() + amount
    }
  }
  override def preRestart(reason: Throwable, message:Option[Any]) {
    // 为了保存数据
    self ! Deposit(balance.single.get)
    super.preRestart(reason, message)
  }
  // All messages which are implemented in the normally function,
  // will not be passed to the atomically function. 
  override def normally = {
    case GetBalance => sender ! balance.single.get
  }
}

// 测试代码
val account1 = system.actorOf(Props[AccountTransactor])
val account2 = system.actorOf(Props[AccountTransactor])
val transaction = Coordinated()
transaction atomic { implicit t
  account1 ! transaction(Withdraw(amount = 50))
  account2 ! transaction(Deposit(amount = 50))
}

~~~~~~

When we deposit some
money in an account it doesn't need to be done in a Coordinated transaction. We
already saw that we can send a coordinated message without joining the
transaction, but when using a transactor, we can also send just the message.

~~~~~~
// These two lines of code are equivalent when using a transactor. 
account1 ! Coordinated(Deposit(amount = 100))
account1 ! Deposit(amount = 100)
~~~~~~

The AccountTransactor only
acts within a transaction, but it doesn't include other actors within its transaction.
When the transfer Actor starts a coordinated transaction we need to include both
accounts in the transaction.

~~~~~~
override def coordinate = {
  case TransferTransaction(amount, from, to) =>
    sendTo(from -> Withdraw(amount),
    to -> Deposit(amount))
}
// When you want to send the received message to the other actors, you can also use the include method:
// sends the received Message to the three actors. 
override def coordinate = {
  case msg:Message => include(actor1, actor2, actor3)
}
~~~~~~

For these kind of actions, a
transactor has two methods which can be overridden, the before and after method.
These methods are called just before and after the atomically method and are also
partial functions. For our example, we don't need the before method, but using the
after to be able to send the "done" message when the transaction has successfully
ended.

~~~~~~
override def after = {
  case TransferTransaction(amount, from, to) => sender ! "done"
}
~~~~~~
