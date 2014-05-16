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
回到我们之前关于查找第一个出现单词的案例, 你也许想要在屏幕输出这个单词的位置:
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

## Functional Composition and For-Comprehensions ##
## Projections ##
## Extending Futures ##
# Blocking #
# Exceptions #
# Promises #
# Utilities #
