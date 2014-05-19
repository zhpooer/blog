title: scala文档翻译-Future&Promise
date: 2014-05-15 09:44:38
tags:
- scala
- 翻译
---
# 拔 #
上课无聊, 刚好看scala文档, 遂翻译之, 晚上回来在网上找到已经有人做过了, 且翻译的比我好(-.-!!!),
但已经翻译了一半, 兴致也不错, 那就当练习英语好了(真吊丝啊).

不过 Scala 2.11关于 `Macro` 和 `Reflect`的内容很没有人下手, 窃喜之. 哈哈, 我的征途是星辰大海!!!

[原文连接](http://docs.scala-lang.org/overviews/core/futures.html)

[已有译文连接](https://code.csdn.net/DOC_Scala/chinese_scala_offical_document/file/Futures-and-Promises-cn.md#anchor_0)

# Introduction #

Futures 为并发提供了许多非阻塞(non-blocking)和有效率的操作. 它的概念很简单, 一个 `Future` 就是一系列的还没有发生的操作或运算(一个运算占位符).
事实上, `Future` 的中的运算会被并行地执行, 而运行结果可以被延迟得到. 这样就可以更快地 非阻塞地 异步地 并发地执行复合并行任务.

实际上, future 和 promises 的非阻塞操作是利用回调函数替换掉经典的阻塞操作.
为了简化回调操作和语法的理解, Scala 提供了提供了组合子, 如  `flatMap`, `foreach` 和 `filter`,
利用这些调用可以组合 future 进行非阻塞的操作. 如果必要的话, futures 也提供了阻塞操作(虽然这并不值得提倡).
# Futures #
一个 `Future` 是一个持有(holding)一个值的对象, 但是这个值在有些时候是取不到的. 这个值常常是一些运算的执行结果:
1. 如果这个运算还没有完成, 这个 Future 还没有完成(is not completed)
2. 如果这个运算最终得到一个值, 或在执行的过程中抛出了异常(exception), 这个 Future 执行完成(completed)

执行完成(Completion)的表现为下面给两种情况中的任意一种情况:
1. 当一个 Future 成功的得到一个值, 那么这个 Future 执行完成.
2. 当一个 Future 在运算的时候抛出到一个异常(exception), 这个 Future 因为异常而失败了.

一个 Future 只能被赋值(assigned)一次. 一旦一个 Future 对象(object)(Future内部对象?)被赋值或得到一个异常,
那么它就是不可变的(immutable), 也就是说不能被重新赋值

创建一个 future 对象(object)最简单的方法是调用 `future` 方法, 它会启动一个异步的运算,
然后返回一个指向这次运行的结果的 future. 一旦这个 future 执行完成, 就可以得到这次运算结果.

`Future[T]` 是一个表示 future 对象(objects)的类型(type).
然而, `future`是一个生成和调度异步运算的方法, 它返回一个代表*将要得到*(will)运算结果的 future 对象.

这个特性将通过一个案例展示.

假设我们要调用一个虚构(hypothetical)的 API 从一个社交网络得到一个给定用户的好友列表.
我们将会打开一个新的会话(session),然后发送请求(request)去获取一个特殊用户的的好友列表:
~~~~~~
import scala.concurrent._
import ExecutionContext.Implicits.global
val session = socialNetwork.createSessionFor("user", credentials)
val f: Future[List[Friend]] = future {
    session.getFriends()
}
~~~~~~

上边这段代码中, 我们首先导入(import)了 `scala.concurrent` 包中的内容, 使得 `Future`
类型和 `future`构造器可见.对于第二个导入,我们将在稍后解释.

接着, 我们用一个虚构的方法`createSessionFor`初始化了 session 变量, 我们将用它来给服务端发送请求.
为了得到一个用户的好友列表, 需要通过网络发送一个请求, 这需要花费很长的时间.
这些操作被封装(illustrated)在 `getFriends`方法, 它返回 `List[Friend]`.
为了更好地利用CPU资源直到请求返回, 我们不应该阻塞接下来的程序, 所以这些操作(computation)应该被异步的执行(scheduled).
`future`方法并发地执行了给定的运算代码块(computation block), 这样就可以发送请求(request)给服务端, 并且等待响应(response)

一旦服务器响应, 好友列表(list of friends)可以从 future `f`中得到.

一次不成功的操作(attempt)将会得到一个异常(exception). 在下面的例子中, `session` 没有正常初始化,
所以 `futrue`代码块在执行时, 将会抛出 `NullPointerException`. futrue `f` 将会执行失败(failed).
~~~~~~
val session = null
val f: Future[List[Friend]] = future {
  session.getFriends
}
~~~~~~

`import ExecutionContext.Implicits.global`,
这行代码导入(import)了全局默认执行环境(default global execution context).
执行环境(execution context)会执行被提交过来的任务, 你也可以认为执行环境是线程池(thread pools).
它们(execution context)对 `future` 方法非常重要, 因为它们决定了(handle)什么时候以及怎样异步的执行运算.
你也可以定义自己的执行环境, 用来使用 `future`, 但是对于上面的代码来说, 导入默认的执行空间就已经够用了.

上面的例子是基于一个虚构的社交网络 API, 它需要通过网络发送请求和等待响应.
你也可以尝试一些其他的关于异步运算的操作. 假设你有一个文本文件(text file),
你想从中找到一个单词第一次出现的位置. 等从硬盘上读取这个文件时候, 这个操作可能会阻塞,
所以可以和剩下的代码块并发地执行.
~~~~~~
val firstOccurrence: Future[Int] = future {
  val source = scala.io.Source.fromFile("myText.txt")
  source.toSeq.indexOfSlice("myKeyword")
}
~~~~~~


## Callbacks ##
我们现在知道了如何区启动以个异步的计算得到一个新的 future 值,
但是我们还不知道如何去得到这个已经就位的结果, 现在我们可以对它做一些操作了.
我们通常只对运算的结果感兴趣, 但不包括它的副作用(side-effect).

在许多 future 的实践中, 一旦 future 的使用者对其结果感兴趣, 它会阻塞
它自己的运算知道 future 的值就位--以至于用future的值执行他自己的运算.
虽然 Scala `Future` 允许这样的操作(我们接下来会讲到),
但是从性能的角度来看, 还有一个完全非阻塞的方式, 那就是注册 future 的回调方法(callback).
一旦future的值就绪, 这个回调方法会被异步的执行.
如果当注册回调函数(callback)时, future的值已经就绪, 那么回调函数将会被异步地执行, 或者
在相同的线程中同步(sequentially)地执行.

最常用的注册回调方法的方式是用 `onComplete` 方法, 它的传入一个类型为 `Try[T] => U` 的回调函数.
如果 future 执行成功, 那么回调函数会接收到一个类型为 `Success[T]` 的值,
否则会接收到一个类型为 `Failure[T]` 的值.

`Try[T]` 类似于 `Option[T]` 或者 `Either[T,S]`, 它是可以持有某种类型的单子(monad).
而现在, 它已经被设计成持有一个值(value) 或者一个异常对象(throwable object)的类型.
而一个 `Option[T]`对象, 不是代表一个值(`Some[T]`), 就是代码没有值(`None`),
`Try[T]`则表示, 当执行成功时, 它是`Success[T]`, 当执行失败时抛出错误时, 它就是 `Failure[T]`.
`Failure[T]` 不像 `None`, 它存储了许多关于为什么得不到最终值的信息.
同时你也可以认为 `Try[T]` 是 `Either[Throwable, T]`的特殊版本.

回到我们刚才的社交网络的例子, 假设我们想要拉取(fetch)自己近期的一些帖子, 并显示在屏幕上.
我们会调用 `getRecentPosts` 方法, 它会返回一个近期关于帖子的列表 `List[String]`.
~~~~~~
import scala.util.{Success, Failure}

val f: Future[List[String]] = future {
  session.getRecentPosts
}
f onComplete {
  case Success(posts) => for (post <- posts) println(post)
  case Failure(t) => println("An error has occured: " + t.getMessage)
}
~~~~~~

`onComplete` 方法通常都要处理成功(successfull)和失败(failed)两种运算结果.
如果要单单处理成功的结果, `onSuccess` 就可以出马了
~~~~~~
val f: Future[List[String]] = future {
  session.getRecentPosts
}
f onSuccess {
  case posts => for (post <- posts) println(post)
}
~~~~~~
同样可以用`onFailure`来处理运算失败的结果
~~~~~~
val f: Future[List[String]] = future {
  session.getRecentPosts
}
f onFailure {
  case t => println("An error has occured: " + t.getMessage)
}
f onSuccess {
  case posts => for (post <- posts) println(post)
}
~~~~~~
只有 future执行失败时, `onFailure` 回调才会被执行, 因此, 它包含了一个异常(exception).

如果回调函数只定义了要处理某个特殊的异常类型, 只有当这个异常出现时,
`onFailure`方法才会触发这个回调(利用偏函数(partial functions)的`isDefinedAt`) 方法.
下面例子中, 注册的`onFailure` 回调方法将永远都不会被触发:
~~~~~~
val f = future {
  2 / 0
}
f onFailure {
  case npe: NullPointerException =>
    println("I'd be amazed if this printed out.")
}
~~~~~~
回到我们之前关于查找单词第一个出现位置的案例, 你也许想要在屏幕输出这个单词的位置:
~~~~~~
val firstOccurrence: Future[Int] = future {
  val source = scala.io.Source.fromFile("myText.txt")
  source.toSeq.indexOfSlice("myKeyword")
}
firstOccurrence onSuccess {
  case idx => println("The keyword first appears at position: " + idx)
}
firstOccurrence onFailure {
  case t => println("Could not process file: " + t.getMessage)
}
~~~~~~
`onComplete`, `onSuccess`, `onFailure` 方法返回的类型是 `Unit`,
这就意味着这些方法的调用不能被链式调用(chained).
这样的设计是刻意为之, 因为链式调用也许暗示着按照一定的顺序注册回调函数
(那么就可以无序地在同一个futre中注册回调函数)

也就是说, 我们现在可以讨论回调函数*什么时候*会被调用.
因为这些回调函数需要 future 中的值, 所以直到 future 执行完成后, 它们才会被调用.
然而, 也不能保证调用它们(callback)的线程是完成 futre 的线程或者创造回调函数的线程.
反而, 当 future 执行完毕后, 在一定时间内回调函数会被一些其他线程执行.
也就是说回调函数最终会被执行.

更进一步的说, 回调函数被执行的顺序不是固定的, 甚至在多次运行的同一个应用程序中.
实际上, 回调函数会不会被一个接一个地调用, 而是会被并行(concurrently)的执行.
这就意味着在下面的例子中, totalA 的值就不确定是表示大写`a`的数量还是表示小写`a`的数量.
~~~~~~
@volatile var totalA = 0
val text = future {
  "na" * 16 + "BATMAN!!!"
}
text onSuccess {
  case txt => totalA += txt.count(_ == 'a')
}
text onSuccess {
  case txt => totalA += txt.count(_ == 'A')
}
~~~~~~
在上面的例子中, 两个回调函数可能一个一个顺序执行, 那么 `totalA` 的值就为18.
然而, 它们也可能并发的执行, 所以 `totalA` 的值不是16就是2,
只是因为 `+=` 不是原子性操作(atomic operation)(它由读和写两部分组成)

考虑到表述的完整性, 回调函数的使用的语法如下:
1. 在 future 中注册一个 `onComplete` 回调, future 执行完成后, 回调函数最终会被执行.
2. 用注册 `onComplete` 的语法, 注册一个 `onSuccess` 或 `onFailure`,
   它们只会在 future 执行成功或执行失败分别调用.
3. 在一个已经执行完成的 future 中, 注册回调函数, 这个回调函数最终还是会被调用.
4. 在 future 中注册多个回调函数的情况下, 它们的执行顺序不是固定的.
   实际上, 回调函数会被并发地执行. 然而, 特定的 `ExecutionContext` 实现可能会按明确的顺序来执行.
5. 如果一些回调函数抛出了异常, 其他回调函数会不受影响, 继续执行.
6. 在某些情况下, 有些回调函数永远不会结束(可能包含了无限循环), 其他的回调函数就可能不会被执行到.
在这种情况下, 一个潜在的阻塞回调必须使用 `blocking` 构造函数(下面有介绍)
7. 一旦执行, 回调函数将会从 future 中移除, 这样方便垃圾回收器回收. 


## Functional Composition and For-Comprehensions ##
尽管前面介绍的回调机制已经足够把 future 的结果和后继计算结合起来.
然而在有时候回调机制并不易于使用, 且会造成冗余的代码.
我们可以通过一个案例来说明. 假使我们有一个 关于货币交易系统的API.
假设这适合的点, 我们想买入美元. 我们先展示一下如何用回调来进行这个操作.
~~~~~~
val rateQuote = future {
    connection.getCurrentValue(USD)
}
rateQuote onSuccess { case quote =>
    val purchase = future {
        if (isProfitable(quote)) connection.buy(amount, quote)
        else throw new Exception("not profitable")
    }
    purchase onSuccess {
        case _ => println("Purchased " + amount + " USD")
    }
}
~~~~~~
一开始我们创建一个获取货币交易的 future `rateQuote`.
当从服务端得到数据, future 执行成功后, 计算执行操作才会进入 `onSuccess` 回调,
这时, 我们开始决定买还是不买. 因此我们创建了另一个 future `purchase`,
用来在可盈利的情况下做出购买决定, 然后向服务器发出请求.
最后, 一旦购买完成, 我们会在标准输出中打印一条通知消息.

这确实是可以行的, 但是由两点原因是这种方法并不方便.
其一, 我们不得不使用 `onSuccess`, 且在其中嵌套调用 `purchase` future.
假设, 我们要在 `purchase` 执行完成后卖出一些货币.
这时我们不得不在`onSuccess`回调中重复这个模式, 从而使代码过度嵌套, 冗长且难以理解.

其二, future `purchase` 没有在其余代码的范围内, 它只能在`onSuccess`回调内部响应.
这就意味着其他部分的程序是取不到 `purchase` future,
也不能注册其他的 `onSuccess` 回调函数, 比如说卖掉些货币.

基于这两个原因, futures 提供了组合器(combinators)使之具有了更加易用的组合形式.
`map`是最基础的组合器之一, 当给定一个 future 和一个映射函数(mapping function)来出来future的值,
映射方法会产生一个新的future, 一旦最初的 future 成功地执行, 新的future会通过该返回值完成计算.
你能够像理解容器(collections)的map一样来理解future的map.

让我们用 `map` 组合子来重写上面的一个案例
~~~~~~
val rateQuote = future {
  connection.getCurrentValue(USD)
}

val purchase = rateQuote map { quote => 
  if (isProfitable(quote)) connection.buy(amount, quote)
  else throw new Exception("not profitable")
}

purchase onSuccess {
  case _ => println("Purchased " + amount + " USD")
}
~~~~~~

通过对`rateQuote`使用 `map`, 我们减少了一次 `onSuccess`回调, 更重要的是避免了嵌套调用.
如果我们现在决定要卖掉一些其他货币, 就可以再次对 `purchase` 使用 `map`了.

但是如果 `isProfitable` 返回 `false`, 因此引起了一个异常, 那怎么办呢?
在这种情况下, `purchase` 会因为异常而失败. 更进一步地说, 如果连接服务器失败,
使得 `getCurrentValue` 抛错, 最终使 `rateQuote` 失败了呢?
在这种情况下, 我们将不能获得值去使用map, 以至于 `purchase` 自动地以和`rateQuote`相同的异常
而执行失败.

总之, 如果最初的 future 执行成功了, 那么返回的值将会和 map 函数一起执行成功.
如果 map 函数抛出了异常, 那么 future 就会带着该异常而失败.
如果最初的future以异常结束, 那么那个返回的future也会以同样的失败结束.
这种异常传导机制会适用于其他组合子(combinators).

这种设计也同样被用于for语法(for-comprehensions).
所以, futrues 同样也有 `flatMap`, `filter` 和 `foreach` 组合子.
`flatMap` 方法传入一个函数, 它把值映射到一个新的 future `g`, 一旦 `g`执行完成, 就返回一个
future.

假设我们想把一些美元换成瑞士法郎(CHF). 我们要拉取两个货币的报价,
接着根据两个报价来决定如何购买. 下面是一个在for-comprehensions中使用`flatMap`和`withFilter`的例子
~~~~~~
val usdQuote = future { connection.getCurrentValue(USD) }
val chfQuote = future { connection.getCurrentValue(CHF) }

val purchase = for {
  usd <- usdQuote
  chf <- chfQuote
  if isProfitable(usd, chf)
} yield connection.buy(amount, chf)

purchase onSuccess {
  case _ => println("Purchased " + amount + " CHF")
}
~~~~~~


## Projections ##
## Extending Futures ##
# Blocking #
# Exceptions #
# Promises #
# Utilities #
