title: Functional Programming in Scala-函数式编程入门
date: 2014-07-28 09:14:32
tags:
- scala
---

# What is Functional Programming #


In other words, functions that have no side effects.

Performing any of the following actions directly would involve a side effect:
* Reassigning a variable
* Modifying a data structure in place
* Setting a field on an object
* Throwing an exception or halting with an error
* Printing to the console or reading user input
* Reading from or writing to a file
* Drawing on the screen

We find ways to structure code so that effects
occur but are not observable (For example, we can mutate data that is declared
locally in the body of some function if we ensure that it cannot be referenced
outside that function.)

A function with input type A and output type B (written in Scala as a single type: `A => B`) 

什么是引用透明
~~~~~~
scala> val x = new StringBuilder("Hello")
x: java.lang.StringBuilder = Hello

scala> val y = x.append(", World")
y: java.lang.StringBuilder = Hello, World

scala> val r1 = y.toString
r1: java.lang.String = Hello, World

scala> val r2 = y.toString
r2: java.lang.String = Hello, World

// 有副作用的例子
scala> val x = new StringBuilder("Hello")
x: java.lang.StringBuilder = Hello

scala> val r1 = x.append(", World").toString
r1: java.lang.String = Hello, World

scala> val r2 = x.append(", World").toString
r2: java.lang.String = Hello, World, World
~~~~~~

多态函数
~~~~~~
def binarySearch[A](as: Array[A], key: A, gt: (A,A) => Boolean): Int = {
  @annotation.tailrec
  def go(low: Int, mid: Int, high: Int): Int = {
    if (low > high) -mid - 1
    else {
      val mid2 = (low + high) / 2
      val a = as(mid2)
      val greater = gt(a, key)
      if (!greater && !gt(key,a)) mid2
      else if (greater) go(low, mid2, mid2-1)
      else go(mid2 + 1, mid2, high)
    }
  }
  go(0, 0, as.length - 1)
}
~~~~~~

Lift: We can simply lift them using the map function.
~~~~~~
def lift[A,B](f: A => B): Option[A] => Option[B] = _ map f
~~~~~~

~~~~~~
import java.util.regex._

def pattern(s: String): Option[Pattern] =
  try {
    Some(Pattern.compile(s))
  } catch {
    case e: PatternSyntaxException => None
  }
}

def mkMatcher(pat: String): Option[String => Boolean] =
  pattern(pat) map (p => (s: String) => p.matcher(s).matches)

def mkMatcher_1(pat: String): Option[String => Boolean] =
  for {
    p <- pattern(pat)
  } yield ((s: String) => p.matcher(s).matches)

def doesMatch(pat: String, s: String): Option[Boolean] =
  for {
    p <- mkMatcher_1(pat)
  } yield p(s)

def bothMatch(pat: String, pat2: String, s: String): Option[Boolean] =
  for {
    f <- mkMatcher(pat)
    g <- mkMatcher(pat2)
  } yield f(s) && g(s)

// 翻译成这样
// mkMatcher(pat) flatMap (f =>
//   mkMatcher(pat2) map (g =>
//     f(s) && g(s)))

~~~~~~

Either
~~~~~~
def mean(xs: IndexedSeq[Double]): Either[String, Double] =
  if (xs.isEmpty)
    Left("mean of empty list!")
  else
    Right(xs.sum / xs.length)

// result in Left("invalid name")
for {
  age <- Right(42)
  name <- Left("invalid name")
  salary <- Right(1000000.0)
} yield employee(name, age, salary)

~~~~~~

# strictness and laziness #

Each transformation will produce a temporary list that only ever gets used as input
to the next transformation and is then immediately discarded.

~~~~~~
List(1,2,3,4) map (_ + 10) filter (_ % 2 == 0) map (_ * 3)
~~~~~~

If the evaluation of an expression runs forever or throws an error
instead of returning a definite value, we say that the expression does
not terminate, or that it evaluates to bottom. A function f is strict if
the expression f(x) evaluates to bottom for all x that evaluate to
bottom.

To say a function is non-strict just means that the function may
choose not to evaluate one or more of its arguments.


the Boolean functions `&&` and `||` are non-strict.

The if function would be non-strict, since it
will not evaluate all of its arguments. To be more precise,
we would say that the if function is strict in its condition parameter, 

~~~~~~
scala> def pair2(i: => Int) = { lazy val j = i; (j, j) }
scala> pair2 { println("hi"); 1 + 41 }
~~~~~~
