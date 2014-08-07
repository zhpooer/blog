title: Scala in Action-scala 入门
date: 2014-07-12 09:20:07
tags:
- scala
---

# 面向对象 #
什么是纯粹的面向对象语言
* Encapsulation/information hiding.(封装)
* Inheritance.(继承)
* Polymorphism/dynamic binding.(多态)
* All predefined types are objects.
* All operations are performed by sending messages to objects.
* All user-defined types are objects.

every value is an object, and every operation is a message send,
`1+2` 的被scala翻译为 `1.+(2)`

Along with the pure object-oriented features, Scala has made some *innovations* on
OOP space:
* 模块混入(Modular mixin composition), This feature of Scala has traits in common with
both Java interfaces and abstract classes. You can define contracts using one or
more traits and provide implementations for some or all of the methods.(trait)
* *Self-type* — A mixin doesn’t depend on any methods or fields of the class that it’s
mixed into, but sometimes it’s useful to use fields or methods of the class it’s
mixed into, and this feature of Scala is called self-type.
* *Type abstraction*, 类型抽象 — There are two principle forms of abstraction in programming
languages: parameterization and abstract members. Scala supports both forms
of abstraction uniformly for types and values.


# 函数式编程 #
Functional programming is a programming paradigm
that treats computation as the evaluation of mathematical functions and avoids state
and mutable data.(像数学方法调用, 去除状态和可变数据)

**Mutable vs. immutable data**

An object is called mutable when you can alter the contents of the object if you have
a reference to it. In the case of an immutable object, the contents of the object can’t
be altered if you have a reference to it.

It’s easy to create a mutable object; all you have to do is provide access to the muta-
ble state of the object. The disadvantage of mutable objects is keeping track of the
changes. In a multithreaded environment you need lock/synchronization techniques
to avoid concurrent access. For immutable objects, you don’t have to worry about
these situations.

A function relates every value of the domain (the input) to exactly one value of the codomain (the out-
t). (输入与结果的不变性)

Another aspect of functional program-ming
is that it doesn’t have side effects or mutability. 

Functional programming languages that
support this style of programming provide at least
some of the following features:
* Higher-order functions 
* Lexical closures 
* Pattern matching
* Single assignment 
* Lazy evaluation 
* Type inference 
* Tail call optimization 
* List comprehensions
* Mondadic effects 

scala支持 头等函数(first-class)
~~~~~~
val inc = (x:Int) => x + 1
inc(1)

List(1, 2, 4).map((x:Int) => x + 1)
~~~~~~

# 多范式编程 #

提供多范式编程的原因:
可以为程序员提供各种解决问题的方案, 以及最佳实现取解决问题.
函数式编程可以提供一些简单的组件(function)非常容易地实现一个有趣的功能.
面向对象使用继承, 类等功能简单地实现一个复杂的系统.

In the case of OOP , building blocks are objects, and in
functional programming building blocks are functions.
In Scala, functions are treated as objects.

## 函数即对象 ##

~~~~~~
List(1,2,3).map((x:Int) => x + 1)
// 会被翻译成
List(1, 2, 3).map(new Function1[Int, Int]{ def apply(x:Int): Int = x + 1})
~~~~~~

scala 提供了简单的语言胶合机制, 使添加新的语言特性变得简单方便.
你可以使用提供一些方法, 结合中缀操作符, 后缀操作符以及闭包, 传名函数.

~~~~~~
def loopTill(cond: => Boolean)(body: => Unit) :Unit = {
    if(conf) {
        body
        loopTill(cond)(body)
    }
}
var i = 10
loopTill(i>0) {
    println(i)
    i -= 1
}
~~~~~~

> 什么是闭包  
Closure is a first-class function with free variables that are bound
in the lexical environment. In the loopTill example, the free variable is i.
Even though it’s defined outside the closure, you could still use it inside. The
second parameter in the loopTill example is a closure, and in Scala that’s
represented as an object of type scala.Function0.


The biggest complaint from the dynamic language camp about
statically typed languages is that they don’t help the productivity of the programmer
and they reduce productivity by forcing programmers to write boilerplate code. And

Counting the number of lines in a file in Scala
~~~~~~
val src = scala.io.Source.fromFile(“someFile.txt”)
val count = src.getLines().map(x => 1).sum
~~~~~~

# 静态类型 #

静态类型:
Static typing is a typing system where the values and the variables
have types. A number variable can’t hold anything other than a number.
Types are determined and enforced at compile time or declaration time.

动态类型:
Dynamic typing is a typing system where values have types but the
variables don’t. It’s possible to successively put a number and a string inside
the same variable.

静态类型的类型检查, 可以在编译时防止错误的发送,
但是动态类型需要在运行时, 发现错误

静态类型的另一个好处就是可以有一个强大的 IDEs 帮助你重构

传统的静态类型语言在声明一个变量或者调用一个函数时,强制你提供过于的类型信息.
但是scala 提供了类型推断来帮助程序员像编写动态函数一样编写scala
~~~~~~
val computers = Array(
    Map("name" -> "Macbook", "color" -> "white"),
    Map("name" -> "HP Pavillion", "color" -> "black")
)
~~~~~~

*COMPILE MACROS* The Scala 2.10 release adds experimental support for
compile-time macros.11 This allows programmers to write macro defs: func-
tions that are transparently loaded by the compiler and executed during com-
pilation. This realizes the notion of compile-time metaprogramming for Scala.


| sbt 命令 |
|------|
| :help | 帮助 |
| :cp   | 进入复制文本模式 |
| :load  | 载入 scala 文件 |
| :replay or :r | 从新载入 |
| :quite or :q | 退出 |
| :type | 显示类型 |
| :import | 显示所有已经导入的包 |

~~~~~~
val first :: rest = List(1, 2, 3)
~~~~~~


If the function has side effects, the common convention is to use “()” even though it
isn’t required.
~~~~~~
def myFirstMethod = "exciting times ahead"
~~~~~~

# Command-line REST client #

REST, REpresentational State Transfer
* Application state and functionality are divided into resources.
* Every resource is uniquely addressable using a universal syntax.
* All resources share a uniform interface for transfer of state between client
and resource, consisting of well-defined operations (GET, POST, PUT, DELETE,
OPTIONS, and so on, for RESTful web services) and content types.
* A protocol that’s client/server, stateless cacheable, and layered.

