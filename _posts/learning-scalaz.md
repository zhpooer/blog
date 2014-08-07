title: learning scalaz
date: 2014-07-29 14:48:01
tags:
- scala
- scalaz
---
[猛击我](http://eed3si9n.com/learning-scalaz/Combined+Pages.html)


# polymorphism #

## Parametric polymorphism ##
~~~~~~
def head[A](xs: List[A]): A = xs(0)
~~~~~~
in this function head, it takes a list of A’s,
and returns an A. And it doesn’t matter what the A is:
It could be Ints, Strings, Oranages, Cars, whatever.
Any A would work, and the function is defined for every A that there can be.

## Subtype polymorphism ##

~~~~~~
def plus[A <: Plus[A]](a1: A, a2: A): A = a1.plus(a2)
~~~~~~

## Ad-hoc polymorphism ##

~~~~~~
def plus[A: Plus](a1: A, a2: A): A = implicitly[Plus[A]].plus(a1, a2)
~~~~~~

* we can provide separate function definitions for different types of A
* we can provide function definitions to types (like Int) without
access to its source code
* the function definitions can be enabled or disabled in different scopes

# Monoid #

~~~~~~
object IntMonoid {
  def mappend(a: Int, b: Int): Int = a + b
  def mzero: Int = 0
}
def sum(xs: List[Int]): Int = xs.foldLeft(IntMonoid.mzero)(IntMonoid.mappend)
~~~~~~

version 2

~~~~~~
trait Monoid[A] {
  def mappend(a1: A, a2: A): A
  def mzero: A
}
object IntMonoid extends Monoid[Int] {
  def mappend(a: Int, b: Int): Int = a + b
  def mzero: Int = 0
}
def sum[A](xs: List[A], m: Monoid[A]): A = xs.foldLeft(m.mzero)(m.mappend)
~~~~~~

version 3
~~~~~~
def sum[A](xs: List[A])(implicit m: Monoid[A]): A = xs.foldLeft(m.mzero)(m.mappend)
implicit val intMonoid = IntMonoid
// 也可以这样
def sum[A: Monoid](xs:List[A]):A = {
  val m = implicity[Monoid[A]]
  xs.foldLeft(m.mzero)(m.mappend)
}
~~~~~~

我们也可以提供其他Monoid

~~~~~~
val multiMoinoid: Monoid[Int] = new Monoid[Int] {
  def mappend(a:Int, b:Int): Int = a * b
  def mzero:Int = 1
}
sum(List(1, 2, 3, 4))(multiMonoid)
~~~~~~

# FoldLeft #

~~~~~~
object FoldLeftList {
  def foldLeft[A, B](xs: List[A], b: B, f: (B, A) => B) = xs.foldLeft(b)(f)
}
def sum[A: Monoid](xs: List[A]): A = {
  val m = implicitly[Monoid[A]]
  FoldLeftList.foldLeft(xs, m.mzero, m.mappend)
}

// 可以这样使用
sum(List(1, 2, 3, 4))(multiMonoid)
~~~~~~

版本2

~~~~~~
trait FoldLeft[F[_]] {
  def foldLeft[A, B](xs: F[A], b: B, f: (B, A) => B): B
}

object FoldLeft {
  implicit val FoldLeftList: FoldLeft[List] = new FoldLeft[List] {
    def foldLeft[A, B](xs:List[a], b: B, f: (B, A) => B) = xs.foldLeft(b)(f)
  }
}

def sum[M[_]: FoldLeft, A: Monoid](xs: M[A]): A = {
  val m = implicitly[Monoid[A]]
  val fl = implicitly[FoldLeft[M]]
  fl.foldLeft(xs, m.mzero, m.mappend)
}

// 用法如下
sum(List("a", "b", "c"))
~~~~~~

# Typeclass in Scalaz #

We would like to provide an operator.
But we don’t want to enrich just one type,
but enrich all types that has an instance for Monoid. 

~~~~~~
trait MonoidOp[A] {
  val F: Monoid[A]
  val value: A
  def |+|(a2:A) = F.mappend(value, v2)
}

implicit def toMonoidOp[A: Monoid](a: A):MonoidOp[A] = new MonoidOp[A] {
  val F = implicitly[Monoid[A]]
  val value = a
}

// 简写如下
3 |+| 4
~~~~~~

Using the same technique above, Scalaz also provides method
injections for standard library types like Option and Boolean:
~~~~~~
scala> 1.some | 2
res0: Int = 1

scala> Some(1).getOrElse(2)
res1: Int = 1

scala> (1 > 10)? 1 | 2
res3: Int = 2

scala> if (1 > 10) 1 else 2
res4: Int = 2
~~~~~~

# typeclasses #

It provides purely functional data structures to complement those from
the Scala standard library. It defines a set of foundational
type classes (e.g. Functor, Monad) and corresponding instances
for a large number of data structures.

~~~~~~
scalaVersion := "2.11.0"

val scalazVersion = "7.0.6"

libraryDependencies ++= Seq(
  "org.scalaz" %% "scalaz-core" % scalazVersion,
  "org.scalaz" %% "scalaz-effect" % scalazVersion,
  "org.scalaz" %% "scalaz-typelevel" % scalazVersion,
  "org.scalaz" %% "scalaz-scalacheck-binding" % scalazVersion % "test"
)

scalacOptions += "-feature"

initialCommands in console := "import scalaz._, Scalaz._"
~~~~~~
runt `sbt console`

## Equal ##

Instead of the standard ==, Equal enables ===, =/=, and assert_=== syntax
by declaring equal method. The main difference is that === would fail
compilation if you tried to compare Int and String.

~~~~~~
scala> 1 === 1
res0: Boolean = true

scala> 1 === "foo"
<console>:14: error: could not find implicit value for parameter F0: scalaz.Equal[Object]
              1 === "foo"

scala> 1.some =/= 2.some
res3: Boolean = true

scala> 1 assert_=== 2
java.lang.RuntimeException: 1 ≠ 2
~~~~~~

## Order ##

Ord is for types that have an ordering.
Ord covers all the standard comparing functions
such as `>, <, >= and <=`.

~~~~~~
scala> 1.0 ?|? 2.0
res10: scalaz.Ordering = LT

scala> 1.0 max 2.0
res11: Double = 2.0
~~~~~~

## Show ##

Members of Show can be presented as strings.

~~~~~~
scala> 3.show
res14: scalaz.Cord = 3

scala> 3.shows
res15: String = 3

scala> "hello".println
"hello"
~~~~~~

## Read ##

Read is sort of the opposite typeclass of Show.
The read function takes a string and returns a type
which is a member of Read.

I could not find Scalaz equivalent for this typeclass.

## Enum ##
Enum members are sequentially ordered types — they can be enumerated.
The main advantage of the Enum typeclass is that we can use its types in list ranges.
They also have defined successors and predecesors,
which you can get with the succ and pred functions.

~~~~~~
scala> 'a' to 'e'
res30: scala.collection.immutable.NumericRange.Inclusive[Char] = NumericRange(a, b, c, d, e)

scala> 'a' |-> 'e'
res31: List[Char] = List(a, b, c, d, e)

scala> 3 |=> 5
res32: scalaz.EphemeralStream[Int] = scalaz.EphemeralStreamFunctions$$anon$4@6a61c7b6

scala> 'B'.succ
res33: Char = C
~~~~~~

Instead of the standard to, Enum enables |-> that returns a List by
declaring pred and succ method on top of Order typeclass.
There are a bunch of other operations it enables like
`-+-, ---, from, fromStep, pred, predx, succ, succx, |-->, |->, |==>, and |=>. `
these are all about stepping forward or backward, and returning ranges.

## Bounded ##
Bounded members have an upper and a lower bound.
~~~~~~
scala> implicitly[Enum[Char]].min
res43: Option[Char] = Some(?)

scala> implicitly[Enum[Char]].max
res44: Option[Char] = Some( )

scala> implicitly[Enum[Double]].max
res45: Option[Double] = Some(1.7976931348623157E308)

scala> implicitly[Enum[Int]].min
res46: Option[Int] = Some(-2147483648)
~~~~~~

Enum typeclass instance returns Option[T] for max values.

## Num ##
Num is a numeric typeclass.
Its members have the property of being able to act like numbers.

I did not find Scalaz equivalent for Num, Floating, and Integral.

# Making Our Own Types and Typeclasses #

~~~~~~
// 下面这样是不行的, 因为 Equal[F] 是 nonvariant
// sealed trait TrafficLight
// case object Red extends TrafficLight
// case object Yellow extends TrafficLight
// case object Green extends TrafficLight

case class TrafficLight(name: String)
val red = TrafficLight("red")
val yellow = TrafficLight("yellow")
val green = TrafficLight("green")

implicit val TrafficLightEqual: Equal[TrafficLight] = Equal.equal(_ == _)
~~~~~~

## Yes-No typeclass ##

~~~~~~
trait CanTruthy[A] { self =>
  /** @return true, if `a` is truthy. */
  def truthys(a: A): Boolean
}

object CanTruthy {
  def apply[A](implicit ev: CanTruthy[A]): CanTruthy[A] = ev
  def truthys[A](f: A => Boolean): CanTruthy[A] = new CanTruthy[A] {
    def truthys(a: A): Boolean = f(a)
  }
}

trait CanTruthyOps[A] {
  def self: A
  implicit def F: CanTruthy[A]
  final def truthy: Boolean = F.truthys(self)
}

object ToCanIsTruthyOps {
  implicit def toCanIsTruthyOps[A](v: A)(implicit ev: CanTruthy[A]) =
    new CanTruthyOps[A] {
      def self = v
      implicit def F: CanTruthy[A] = ev
    }
}

// 可以这样使用
implicit val intCanTruthy: CanTruthy[Int] = CanTruthy.truthys({
  case 0 => false
  case _ => true
})
10.truthy  // true

// list can truthy
implicit def listCanTruthy[A]: CanTruthy[List[A]] = CanTruthy.truthys({
  case Nil => false
  case _   => true  
})
// 如果不加这一句, 那么 Nil.truthy 会报错, 因为 canTruthy 是 nonvariance 的
implicit val nilCanTruthy: CanTruthy[scala.collection.immutable.Nil.type] = CanTruthy.truthys(_ => false)

// 按需要计算
def truthyIf[A: CanTruthy, B, C](cond: A)(ifyes: => B)(ifno: => C) =
  if (cond.truthy) ifyes
  else ifno

truthyIf (true) {"YEAH!"} {"NO!"}
truthyIf (Nil) {"YEAH!"} {"NO!"}
truthyIf (2 :: 3 :: 4 :: Nil) {"YEAH!"} {"NO!"}
~~~~~~

# Functor #

And now, we’re going to take a look at the Functor typeclass,
which is basically for things that can be mapped over.

~~~~~~
trait FunctorOps[F[_],A] extends Ops[F[A]] {
  implicit def F: Functor[F]
  ////
  import Leibniz.===

  final def map[B](f: A => B): F[B] = F.map(self)(f)
  ...
}
~~~~~~

~~~~~~
scala> List(1, 2, 3) map {_ + 1}
res15: List[Int] = List(2, 3, 4)

scala> (1, 2, 3) map {_ + 1}
res28: (Int, Int, Int) = (1,2,4)

// Scalaz also defines Functor instance for Function1.
scala> ((x: Int) => x + 1) map {_ * 7}
res30: Int => Int = <function1>

scala> res30(3)
res31: Int = 28
~~~~~~

in Haskell
~~~~~~
-- fmap :: (a -> b) -> (r -> a) -> (r -> b)
ghci> fmap (*3) (+100) 1
303
ghci> (*3) . (+100) $ 1  
303 
~~~~~~

~~~~~~
// 注意 这是左结合
scala> (((_: Int) * 3) map {_ + 100}) (1)
res40: Int = 103

// scalaz defined in F[A]
final def map[B](f: A => B): F[B] = F.map(self)(f)

List(1, 2, 3) map {3*}  // res41: List[Int] = List(3, 6, 9)

// this is a Lift
// There are several neat functions under Functor typeclass
scala> Functor[List].lift {(_: Int) * 2}
res45: List[Int] => List[Int] = <function1>

scala> res45(List(3))
res47: List[Int] = List(6)


// functor 的一些其他用法
scala> List(1, 2, 3) >| "x"
res47: List[String] = List(x, x, x)

scala> List(1, 2, 3) as "x"
res48: List[String] = List(x, x, x)

scala> List(1, 2, 3).fpair
res49: List[(Int, Int)] = List((1,1), (2,2), (3,3))

scala> List(1, 2, 3).strengthL("x")
res50: List[(String, Int)] = List((x,1), (x,2), (x,3))

scala> List(1, 2, 3).strengthR("x")
res51: List[(Int, String)] = List((1,x), (2,x), (3,x))

scala> List(1, 2, 3).void
res52: List[Unit] = List((), (), ())
~~~~~~


# Applicative #
So far, when we were mapping functions over functors,
we usually mapped functions that take only one parameter.
ut what happens when we map a function like *, which takes two parameters,
over a functor?

~~~~~~
scala> List(1, 2, 3, 4) map {(_: Int) * (_:Int)}
<console>:14: error: type mismatch;
 found   : (Int, Int) => Int
 required: Int => ?
              List(1, 2, 3, 4) map {(_: Int) * (_:Int)}

scala> List(1, 2, 3, 4) map {(_: Int) * (_:Int)}.curried
res11: List[Int => Int] = List(<function1>, <function1>, <function1>, <function1>)

scala> res11 map {_(9)}
res12: List[Int] = List(9, 18, 27, 36)
~~~~~~

Meet the Applicative typeclass. It lies in the Control.
Applicative module and it defines two methods, pure and `<*>`.

Applicative 定义

~~~~~~
trait Applicative[F[_]] extends Apply[F] { self =>
  def point[A](a: => A): F[A]

  /** alias for `point` */
  def pure[A](a: => A): F[A] = point(a)
  ...
}
~~~~~~

~~~~~~
scala> 1.point[List]
res14: List[Int] = List(1)

scala> 1.point[Option]
res15: Option[Int] = Some(1)

scala> 1.point[Option] map {_ + 2}
res16: Option[Int] = Some(3)

scala> 1.point[List] map {_ + 2}
res17: List[Int] = List(3)
~~~~~~

~~~~~~
// Applicative 继承了它
trait Apply[F[_]] extends Functor[F] { self =>
  def ap[A,B](fa: => F[A])(f: => F[A => B]): F[B]
}

// 可以这样使用
scala>  9.some <*> {(_: Int) + 3}.some
res20: Option[Int] = Some(12)

scala> 1.some <* 2.some
res35: Option[Int] = Some(1)

scala> none <* 2.some
res36: Option[Nothing] = None

scala> 1.some *> 2.some
res38: Option[Int] = Some(2)

scala> none *> 2.some
res39: Option[Int] = None

cala> 9.some <*> {(_: Int) + 3}.some
res57: Option[Int] = Some(12)

scala> 3.some <*> { 9.some <*> {(_: Int) + (_: Int)}.curried.some }
res58: Option[Int] = Some(12)

// 更加简化的写法
scala> ^(3.some, 5.some) {_ + _}
res59: Option[Int] = Some(8)

scala> ^(3.some, none[Int]) {_ + _}
res60: Option[Int] = None

//还可以这样
scala> (3.some |@| 5.some) {_ + _}
res18: Option[Int] = Some(8)

// List 作为 Apply
scala> List(1, 2, 3) <*> List((_: Int) * 0, (_: Int) + 100, (x: Int) => x * x)
res61: List[Int] = List(0, 0, 0, 101, 102, 103, 1, 4, 9)

scala> List(3, 4) <*> { List(1, 2) <*> List({(_: Int) + (_: Int)}.curried, {(_: Int) * (_: Int)}.curried) }
res62: List[Int] = List(4, 5, 5, 6, 3, 4, 6, 8)

scala> (List("ha", "heh", "hmm") |@| List("?", "!", ".")) {_ + _}
res63: List[String] = List(ha?, ha!, ha., heh?, heh!, heh., hmm?, hmm!, hmm.)
~~~~~~

## 一些有趣用法 ##
~~~~~~
def sequenceA[F[_]: Applicative, A](list: List[F[A]]): F[List[A]] = list match {
  case Nil     => (Nil: List[A]).point[F]
  case x :: xs => (x |@| sequenceA(xs)) {_ :: _} 
}

scala> sequenceA(List(1.some, 2.some))
res82: Option[List[Int]] = Some(List(1, 2))

scala> sequenceA(List(3.some, none, 1.some))
res85: Option[List[Int]] = None

scala> sequenceA(List(List(1, 2, 3), List(4, 5, 6)))
res86: List[List[Int]] = List(List(1, 4), List(1, 5), List(1, 6), List(2, 4), List(2, 5), List(2, 6), List(3, 4), List(3, 5), List(3, 6))
~~~~~~

