title: learning scalaz 进阶
date: 2014-07-29 17:17:46
tags:
- scala
- scalaz
---

# Tagged type #

~~~~~~
type Tagged[U] = { type Tag = U }
type @@[T, U] = T with Tagged[U]
~~~~~~

Suppose we want a way to express mass using kilogram,
because kg is the international standard of unit.
Normally we would pass in Double and call it a day,
but we can’t distinguish that from other Double values.
Can we use case class for this?
~~~~~~
case class KiloGram(value: Double)
// we have to call x.value every time we need to extract the value out of it.

// Tagged type to the rescue.
sealed trait KiloGram
def KiloGram[A](a: A): A @@ KiloGram = Tag[A, KiloGram](a)
val mass = KiloGram(20.0)

2 * mass // res2: Double = 40.0
~~~~~~

A `@@` KiloGram is an infix notation of `scalaz.@@[A, KiloGram]`

~~~~~~
scala> sealed trait JoulePerKiloGram
defined trait JoulePerKiloGram

scala> def JoulePerKiloGram[A](a: A): A @@ JoulePerKiloGram = Tag[A, JoulePerKiloGram](a)
JoulePerKiloGram: [A](a: A)scalaz.@@[A,JoulePerKiloGram]

scala> def energyR(m: Double @@ KiloGram): Double @@ JoulePerKiloGram =
     |   JoulePerKiloGram(299792458.0 * 299792458.0 * m)
energyR: (m: scalaz.@@[Double,KiloGram])scalaz.@@[Double,JoulePerKiloGram]

scala> energyR(mass)
res4: scalaz.@@[Double,JoulePerKiloGram] = 1.79751035747363533E18

scala> energyR(10.0)
<console>:18: error: type mismatch;
 found   : Double(10.0)
 required: scalaz.@@[Double,KiloGram]
    (which expands to)  Double with AnyRef{type Tag = KiloGram}
              energyR(10.0)
~~~~~~

# Monoids #

Monoid defined in scalaz
~~~~~~
trait Monoid[A] extends Semigroup[A] { self =>
  ////
  /** The identity element for `append`. */
  def zero: A
  ...
}

trait Semigroup[A]  { self =>
  def append(a1: A, a2: => A): A
  ...
}

// operators
trait SemigroupOps[A] extends Ops[A] {
  final def |+|(other: => A): A = A.append(self, other)
  final def mappend(other: => A): A = A.append(self, other)
  final def ⊹(other: => A): A = A.append(self, other)
}

scala> List(1, 2, 3) |+| List(4, 5, 6)
res26: List[Int] = List(1, 2, 3, 4, 5, 6)

scala> "one" |+| "two"
res27: String = onetwo

scala> Monoid[List[Int]].zero
res15: List[Int] = List()

scala> Monoid[String].zero
res16: String = ""
~~~~~~

# Tags.Multiplication #

This is where Scalaz 7 uses tagged type.
There are 8 tags for Monoids and 1 named Zip for Applicative.

~~~~~~
// 乘法表示, 也可以有加法
scala> Tags.Multiplication(10) |+| Monoid[Int @@ Tags.Multiplication].zero
res21: scalaz.@@[Int,scalaz.Tags.Multiplication] = 10

scala> 10 |+| Monoid[Int].zero
res22: Int = 10
~~~~~~

## Tags.Disjunction and Tags.Conjunction ##

或 非
~~~~~~
scala> Tags.Disjunction(true) |+| Tags.Disjunction(false)
res28: scalaz.@@[Boolean,scalaz.Tags.Disjunction] = true

scala> Monoid[Boolean @@ Tags.Disjunction].zero |+| Tags.Disjunction(true)
res29: scalaz.@@[Boolean,scalaz.Tags.Disjunction] = true

scala> Monoid[Boolean @@ Tags.Disjunction].zero |+| Monoid[Boolean @@ Tags.Disjunction].zero
res30: scalaz.@@[Boolean,scalaz.Tags.Disjunction] = false

scala> Monoid[Boolean @@ Tags.Conjunction].zero |+| Tags.Conjunction(true)
res31: scalaz.@@[Boolean,scalaz.Tags.Conjunction] = true

scala> Monoid[Boolean @@ Tags.Conjunction].zero |+| Tags.Conjunction(false)
res32: scalaz.@@[Boolean,scalaz.Tags.Conjunction] = false
~~~~~~

## Ordering as Monoid ##

~~~~~~
scala> Ordering.LT |+| Ordering.GT
<console>:14: error: value |+| is not a member of object scalaz.Ordering.LT
              Ordering.LT |+| Ordering.GT
                          ^

scala> (Ordering.LT: Ordering) |+| (Ordering.GT: Ordering)
res42: scalaz.Ordering = LT

scala> (Ordering.GT: Ordering) |+| (Ordering.LT: Ordering)
res43: scalaz.Ordering = GT

scala> Monoid[Ordering].zero |+| (Ordering.LT: Ordering)
res44: scalaz.Ordering = LT

scala> Monoid[Ordering].zero |+| (Ordering.GT: Ordering)
res45: scalaz.Ordering = GT

// 定义长度比较
scala> def lengthCompare(lhs: String, rhs: String): Ordering =
         (lhs.length ?|? rhs.length) |+| (lhs ?|? rhs)
lengthCompare: (lhs: String, rhs: String)scalaz.Ordering

scala> lengthCompare("zen", "ants")
res46: scalaz.Ordering = LT

scala> lengthCompare("zen", "ant")
res47: scalaz.Ordering = GT
~~~~~~

# Functor Laws #

~~~~~~
trait FunctorLaw {
  /** The identity function, lifted, is a no-op. */
  def identity[A](fa: F[A])(implicit FA: Equal[F[A]]): Boolean = FA.equal(map(fa)(x => x), fa)

  /**
   * A series of maps may be freely rewritten as a single map on a
   * composed function.
   */
  def associative[A, B, C](fa: F[A], f1: A => B, f2: B => C)(implicit FC: Equal[F[C]]): Boolean =
    FC.equal(map(map(fa)(f1))(f2), map(fa)(f2 compose f1))
}
~~~~~~

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

initialCommands in console in Test := "import scalaz._, Scalaz._, scalacheck.ScalazProperties._, scalacheck.ScalazArbitrary._,scalacheck.ScalaCheckBinding._"
~~~~~~

~~~~~~
$ sbt test:console
[info] Starting scala interpreter...
[info] 
import scalaz._
import Scalaz._
import scalacheck.ScalazProperties._
import scalacheck.ScalazArbitrary._
import scalacheck.ScalaCheckBinding._
Welcome to Scala version 2.10.3 (Java HotSpot(TM) 64-Bit Server VM, Java 1.6.0_45).
Type in expressions to have them evaluated.
Type :help for more information.

scala> functor.laws[List].check
~~~~~~

这些规则可能会被破坏

# Applicative Laws #

~~~~~~
trait ApplicativeLaw extends FunctorLaw {
  def identityAp[A](fa: F[A])(implicit FA: Equal[F[A]]): Boolean =
    FA.equal(ap(fa)(point((a: A) => a)), fa)
    
  def composition[A, B, C](fbc: F[B => C], fab: F[A => B], fa: F[A])(implicit FC: Equal[F[C]]) =
    FC.equal(ap(ap(fa)(fab))(fbc), ap(fa)(ap(fab)(ap(fbc)(point((bc: B => C) => (ab: A => B) => bc compose ab)))))
    
  def homomorphism[A, B](ab: A => B, a: A)(implicit FB: Equal[F[B]]): Boolean =
    FB.equal(ap(point(a))(point(ab)), point(ab(a)))
    
  def interchange[A, B](f: F[A => B], a: A)(implicit FB: Equal[F[B]]): Boolean =
    FB.equal(ap(point(a))(f), ap(f)(point((f: A => B) => f(a))))
}
~~~~~~

# Semigroup Law #
~~~~~~
trait SemigroupLaw {
  def associative(f1: F, f2: F, f3: F)(implicit F: Equal[F]): Boolean =
    F.equal(append(f1, append(f2, f3)), append(append(f1, f2), f3))
}
~~~~~~

# Monoid Laws #

~~~~~~
trait MonoidLaw extends SemigroupLaw {
  def leftIdentity(a: F)(implicit F: Equal[F]) = F.equal(a, append(zero, a))
  def rightIdentity(a: F)(implicit F: Equal[F]) = F.equal(a, append(a, zero))
}
~~~~~~

# Option as Monoid #
~~~~~~
implicit def optionMonoid[A: Semigroup]: Monoid[Option[A]] = new Monoid[Option[A]] {
  def append(f1: Option[A], f2: => Option[A]) = (f1, f2) match {
    case (Some(a1), Some(a2)) => Some(Semigroup[A].append(a1, a2))
    case (Some(a1), None)     => f1
    case (None, Some(a2))     => f2
    case (None, None)         => None
  }
  def zero: Option[A] = None
}

scala> (none: Option[String]) |+| "andy".some
res23: Option[String] = Some(andy)

scala> (Ordering.LT: Ordering).some |+| none
res25: Option[scalaz.Ordering] = Some(LT)
~~~~~~

if we don’t know if the contents are monoids,
we can’t use mappend between them, so what are we to do? Well,
one thing we can do is to just discard the second
value and keep the first one. For this, the First a type exists.
~~~~~~
scala> Tags.First('a'.some) |+| Tags.First('b'.some)
res26: scalaz.@@[Option[Char],scalaz.Tags.First] = Some(a)

scala> Tags.First(none: Option[Char]) |+| Tags.First('b'.some)
res27: scalaz.@@[Option[Char],scalaz.Tags.First] = Some(b)

scala> Tags.First('a'.some) |+| Tags.First(none: Option[Char])
res28: scalaz.@@[Option[Char],scalaz.Tags.First] = Some(a)

// 同理可得
scala> Tags.First('a'.some) |+| Tags.First('b'.some)
res26: scalaz.@@[Option[Char],scalaz.Tags.First] = Some(a)

scala> Tags.First(none: Option[Char]) |+| Tags.First('b'.some)
res27: scalaz.@@[Option[Char],scalaz.Tags.First] = Some(b)

scala> Tags.First('a'.some) |+| Tags.First(none: Option[Char])
res28: scalaz.@@[Option[Char],scalaz.Tags.First] = Some(a)
~~~~~~

# Foldable #

~~~~~~
trait Foldable[F[_]] { self =>
  /** Map each element of the structure to a [[scalaz.Monoid]], and combine the results. */
  def foldMap[A,B](fa: F[A])(f: A => B)(implicit F: Monoid[B]): B

  /**Right-associative fold of a structure. */
  def foldRight[A, B](fa: F[A], z: => B)(f: (A, => B) => B): B

  ...
}
~~~~~~

~~~~~~
/** Wraps a value `self` and provides methods related to `Foldable` */
trait FoldableOps[F[_],A] extends Ops[F[A]] {
  implicit def F: Foldable[F]
  ////
  final def foldMap[B: Monoid](f: A => B = (a: A) => a): B = F.foldMap(self)(f)
  final def foldRight[B](z: => B)(f: (A, => B) => B): B = F.foldRight(self, z)(f)
  final def foldLeft[B](z: B)(f: (B, A) => B): B = F.foldLeft(self, z)(f)
  final def foldRightM[G[_], B](z: => B)(f: (A, => B) => G[B])(implicit M: Monad[G]): G[B] = F.foldRightM(self, z)(f)
  final def foldLeftM[G[_], B](z: B)(f: (B, A) => G[B])(implicit M: Monad[G]): G[B] = F.foldLeftM(self, z)(f)
  final def foldr[B](z: => B)(f: A => (=> B) => B): B = F.foldr(self, z)(f)
  final def foldl[B](z: B)(f: B => A => B): B = F.foldl(self, z)(f)
  final def foldrM[G[_], B](z: => B)(f: A => ( => B) => G[B])(implicit M: Monad[G]): G[B] = F.foldrM(self, z)(f)
  final def foldlM[G[_], B](z: B)(f: B => A => G[B])(implicit M: Monad[G]): G[B] = F.foldlM(self, z)(f)
  final def foldr1(f: (A, => A) => A): Option[A] = F.foldr1(self)(f)
  final def foldl1(f: (A, A) => A): Option[A] = F.foldl1(self)(f)
  final def sumr(implicit A: Monoid[A]): A = F.foldRight(self, A.zero)(A.append)
  final def suml(implicit A: Monoid[A]): A = F.foldLeft(self, A.zero)(A.append(_, _))
  final def toList: List[A] = F.toList(self)
  final def toIndexedSeq: IndexedSeq[A] = F.toIndexedSeq(self)
  final def toSet: Set[A] = F.toSet(self)
  final def toStream: Stream[A] = F.toStream(self)
  final def all(p: A => Boolean): Boolean = F.all(self)(p)
  final def ∀(p: A => Boolean): Boolean = F.all(self)(p)
  final def allM[G[_]: Monad](p: A => G[Boolean]): G[Boolean] = F.allM(self)(p)
  final def anyM[G[_]: Monad](p: A => G[Boolean]): G[Boolean] = F.anyM(self)(p)
  final def any(p: A => Boolean): Boolean = F.any(self)(p)
  final def ∃(p: A => Boolean): Boolean = F.any(self)(p)
  final def count: Int = F.count(self)
  final def maximum(implicit A: Order[A]): Option[A] = F.maximum(self)
  final def minimum(implicit A: Order[A]): Option[A] = F.minimum(self)
  final def longDigits(implicit d: A <:< Digit): Long = F.longDigits(self)
  final def empty: Boolean = F.empty(self)
  final def element(a: A)(implicit A: Equal[A]): Boolean = F.element(self, a)
  final def splitWith(p: A => Boolean): List[List[A]] = F.splitWith(self)(p)
  final def selectSplit(p: A => Boolean): List[List[A]] = F.selectSplit(self)(p)
  final def collapse[X[_]](implicit A: ApplicativePlus[X]): X[A] = F.collapse(self)
  final def concatenate(implicit A: Monoid[A]): A = F.fold(self)
  final def traverse_[M[_]:Applicative](f: A => M[Unit]): M[Unit] = F.traverse_(self)(f)

  ////
}
~~~~~~

用法如下
~~~~~~
scala> List(1, 2, 3).foldRight (1) {_ * _}
res49: Int = 6

scala> 9.some.foldLeft(2) {_ + _}
res50: Int = 11

// use monoid
scala> List(1, 2, 3) foldMap {identity}
res53: Int = 6

scala> List(true, false, true, true) foldMap {Tags.Disjunction}
res56: scalaz.@@[Boolean,scalaz.Tags.Disjunction] = true
~~~~~~

# Monads #

Monads are a natural extension applicative functors, and they provide
a solution to the following problem: If we have a value with context, m a,
how do we apply it to a function that takes a normal a and returns a value with a contex

~~~~~~
trait Monad[F[_]] extends Applicative[F] with Bind[F] { self =>
  ////
}
~~~~~~

## Bind ##

~~~~~~
trait Bind[F[_]] extends Apply[F] { self =>
  /** Equivalent to `join(map(fa)(f))`. */
  def bind[A, B](fa: F[A])(f: A => F[B]): F[B]
}

/** Wraps a value `self` and provides methods related to `Bind` */
trait BindOps[F[_],A] extends Ops[F[A]] {
  implicit def F: Bind[F]
  ////
  import Liskov.<~<

  def flatMap[B](f: A => F[B]) = F.bind(self)(f)
  def >>=[B](f: A => F[B]) = F.bind(self)(f)
  def ∗[B](f: A => F[B]) = F.bind(self)(f)
  def join[B](implicit ev: A <~< F[B]): F[B] = F.bind(self)(ev(_))
  def μ[B](implicit ev: A <~< F[B]): F[B] = F.bind(self)(ev(_))
  def >>[B](b: F[B]): F[B] = F.bind(self)(_ => b)
  def ifM[B](ifTrue: => F[B], ifFalse: => F[B])(implicit ev: A <~< Boolean): F[B] = {
    val value: F[Boolean] = Liskov.co[F, A, Boolean](ev)(self)
    F.ifM(value, ifTrue, ifFalse)
  }
  ////
}
~~~~~~

~~~~~~
scala> 3.some flatMap { x => (x + 1).some }
res2: Option[Int] = Some(4)

scala> (none: Option[Int]) flatMap { x => (x + 1).some }
res3: Option[Int] = None
~~~~~~

## Monad ##

~~~~~~
scala> Monad[Option].point("WHAT")
res5: Option[String] = Some(WHAT)

scala> 9.some flatMap { x => Monad[Option].point(x * 10) }
res6: Option[Int] = Some(90)

scala> (none: Option[Int]) flatMap { x => Monad[Option].point(x * 10) }
res7: Option[Int] = None
~~~~~~

### 小鸟的案例 ##

小鸟在平衡杆两边停留

~~~~~~
type Birds = Int

case class Pole(left: Birds, right: Birds){
  def landLeft(n: Birds): Pole = copy(left = left + n)
  def landRight(n: Birds): Pole = copy(right = right + n) 
}

// 可以这样使用
Pole(0, 0).landLeft(1).landRight(1).landLeft(2)
~~~~~~

version 2

~~~~~~
case class Pole(left: Birds, right: Birds) {
  def landLeft(n: Birds): Option[Pole] = 
    if (math.abs((left + n) - right) < 4) copy(left = left + n).some
    else none
  def landRight(n: Birds): Option[Pole] =
    if (math.abs(left - (right + n)) < 4) copy(right = right + n).some
    else none
}

// 可以这样使用
Pole(0, 0).landRight(1) flatMap {_.landLeft(2)}

Monad[Option].point(Pole(0, 0)) >>= {_.landRight(2)} >>= {_.landLeft(2)} >>= {_.landRight(2)}


// >> 符号的使用
scala> (none: Option[Int]) >> 3.some
res25: Option[Int] = None

scala> 3.some >> 4.some
res26: Option[Int] = Some(4)

scala> 3.some >> (none: Option[Int])
res27: Option[Int] = None
~~~~~~

~~~~~~
// 这里会报错是因为 = 号的优先级是最低的
scala> Monad[Option].point(Pole(0, 0)) >>= {_.landLeft(1)} >> (none: Option[Pole]) >>= {_.landRight(1)}
<console>:26: error: missing parameter type for expanded function ((x$1) => x$1.landLeft(1))
              Monad[Option].point(Pole(0, 0)) >>= {_.landLeft(1)} >> (none: Option[Pole]) >>= {_.landRight(1)}

// 要这么写
Monad[Option].point(Pole(0, 0)).>>=({_.landLeft(1)}).>>(none: Option[Pole]).>>=({_.landRight(1)})
// 或者这么写
(Monad[Option].point(Pole(0, 0)) >>= {_.landLeft(1)}) >> (none: Option[Pole]) >>= {_.landRight(1)}
~~~~~~

## for syntax ##

~~~~~~
scala> 3.some >>= { x => (none: Option[String]) >>= { y => (x.shows + y).some } }
res17: Option[String] = None

scala> (none: Option[Int]) >>= { x => "!".some >>= { y => (x.shows + y).some } }
res16: Option[String] = None

scala> 3.some >>= { x => "!".some >>= { y => (none: Option[String]) } }
res18: Option[String] = Nonescala> 3.some >>= { x => (none: Option[String]) >>= { y => (x.shows + y).some } }
res17: Option[String] = None

scala> (none: Option[Int]) >>= { x => "!".some >>= { y => (x.shows + y).some } }
res16: Option[String] = None

scala> 3.some >>= { x => "!".some >>= { y => (none: Option[String]) } }
res18: Option[String] = None
~~~~~~

~~~~~~
scala> for {
         x <- 3.some
         y <- "!".some
       } yield (x.shows + y)
res19: Option[String] = Some(3!)

scala> def routine: Option[Pole] =
         for {
           start <- Monad[Option].point(Pole(0, 0))
           first <- start.landLeft(2)
           second <- first.landRight(2)
           third <- second.landLeft(1)
         } yield third
routine: Option[Pole]

scala> routine
res20: Option[Pole] = Some(Pole(3,2))

scala> def routine: Option[Pole] =
         for {
           start <- Monad[Option].point(Pole(0, 0))
           first <- start.landLeft(2)
           _ <- (none: Option[Pole])
           second <- first.landRight(2)
           third <- second.landLeft(1)
         } yield third
routine: Option[Pole]

scala> routine
res23: Option[Pole] = None

scala> def wopwop: Option[Char] =
         for {
           (x :: xs) <- "".toList.some
         } yield x
wopwop: Option[Char]

scala> wopwop
res28: Option[Char] = None
~~~~~~


## List Monad ##


~~~~~~
scala> ^(List(1, 2, 3), List(10, 100, 100)) {_ * _}
res29: List[Int] = List(10, 100, 100, 20, 200, 200, 30, 300, 300)

scala> List(3, 4, 5) >>= {x => List(x, -x)}
res30: List[Int] = List(3, -3, 4, -4, 5, -5)

scala> for {
         n <- List(1, 2)
         ch <- List('a', 'b')
       } yield (n, ch)
res33: List[(Int, Char)] = List((1,a), (1,b), (2,a), (2,b))
~~~~~~

## MonadPlus and the guard function ##

The MonadPlus type class is for monads that can also act as monoids.
~~~~~~
scala> for {
         x <- 1 |-> 50 if x.shows contains '7'
       } yield x
res40: List[Int] = List(7, 17, 27, 37, 47)
~~~~~~

~~~~~~
trait MonadPlus[F[_]] extends Monad[F] with ApplicativePlus[F] { self =>
  ...
}

trait ApplicativePlus[F[_]] extends Applicative[F] with PlusEmpty[F] { self =>
  ...
}

trait PlusEmpty[F[_]] extends Plus[F] { self =>
  ////
  def empty[A]: F[A]
}

trait Plus[F[_]]  { self =>
  def plus[A](a: F[A], b: => F[A]): F[A]
}
~~~~~~

~~~~~~
scala> List(1, 2, 3) <+> List(4, 5, 6)
res43: List[Int] = List(1, 2, 3, 4, 5, 6)

scala> (1 |-> 50) filter { x => x.shows contains '7' }
res46: List[Int] = List(7, 17, 27, 37, 47)
~~~~~~

## A knight's quest ##
Say you have a chess board and only one knight piece on it.
We want to find out if the knight can reach a certain position
in three moves.

移动三步能到达的位置

~~~~~~
case class KnightPos(c: Int, r: Int) {
  // 移动一步能到达的位置
  def move: List[KnightPos] =
    for {
      KnightPos(c2, r2) <- List(KnightPos(c + 2, r - 1), KnightPos(c + 2, r + 1),
        KnightPos(c - 2, r - 1), KnightPos(c - 2, r + 1),
        KnightPos(c + 1, r - 2), KnightPos(c + 1, r + 2),
        KnightPos(c - 1, r - 2), KnightPos(c - 1, r + 2)) if (
        ((1 |-> 8) contains c2) && ((1 |-> 8) contains r2))
    } yield KnightPos(c2, r2)

  // 移动三步
  def in3: List[KnightPos] =
    for {
      first <- move
      second <- first.move
      third <- second.move
    } yield third
    
  // 移动三步是否能到达这个位置
  def canReachIn3(end: KnightPos): Boolean = in3 contains end
}
~~~~~~


## Writer ##

Whereas the Maybe monad is for values with an added context of 
failure, and the list monad is for nondeterministic values, Writer
monad is for values that have another value attached that acts as a
sort of log value.

~~~~~~
def isBigGang(x: Int): (Boolean, String) =
  (x > 9, "Compared gang size to 9.")

implicit class PairOps[A](pair: (A, String)) {
  def applyLog[B](f: A => (B, String)): (B, String) = {
    val (x, log) = pair
    val (y, newlog) = f(x)
    (y, log ++ newlog)
  }
}

(3, "Smallish gang.") applyLog isBigGang  // (false,Smallish gang.Compared gang size to 9.)

~~~~~~

version 2

~~~~~~
implicit class PairOps[A, B: Monoid](pair: (A, B)) {
  def applyLog[C](f: A => (C, B)): (C, B) = {
    val (x, log) = pair
    val (y, newlog) = f(x)
    (y, log |+| newlog)
  }
}

// (false,Smallish gang.Compared gang size to 9.)
(3, "Smallish gang.") applyLog isBigGang
~~~~~~


### Writer define ###

~~~~~~
type Writer[+W, +A] = WriterT[Id, W, A]

sealed trait WriterT[F[+_], +W, +A] { self =>
  val run: F[(W, A)]

  def written(implicit F: Functor[F]): F[W] =
    F.map(run)(_._1)
  def value(implicit F: Functor[F]): F[A] =
    F.map(run)(_._2)
}

trait WriterV[A] extends Ops[A] {
  def set[W](w: W): Writer[W, A] = WriterT.writer(w -> self)

  def tell: Writer[A, Unit] = WriterT.tell(self)
}
~~~~~~

~~~~~~
scala> 3.set("something")
res57: scalaz.Writer[String,Int] = scalaz.WriterTFunctions$$anon$26@159663c3

scala> "something".tell
res58: scalaz.Writer[String,Unit] = scalaz.WriterTFunctions$$anon$26@374de9cf

scala> MonadWriter[Writer, String]
res62: scalaz.MonadWriter[scalaz.Writer,String] = scalaz.WriterTInstances$$anon$1@6b8501fa

scala> MonadWriter[Writer, String].point(3).run
res64: (String, Int) = ("",3)
~~~~~~

### Using for syntax with Writer ###

~~~~~~
scala> def logNumber(x: Int): Writer[List[String], Int] =
         x.set(List("Got number: " + x.shows))
logNumber: (x: Int)scalaz.Writer[List[String],Int]

scala> def multWithLog: Writer[List[String], Int] = for {
         a <- logNumber(3)
         b <- logNumber(5)
       } yield a * b
multWithLog: scalaz.Writer[List[String],Int]

scala> multWithLog.run
res67: (List[String], Int) = (List(Got number: 3, Got number: 5),15)


def gcd(a: Int, b: Int): Writer[List[String], Int] =
  if (b == 0)
    for {
      _ <- List("Finished with " + a.shows).tell
    } yield a
  else
    List(a.shows + " mod " + b.shows + " = " + (a % b).shows).tell >>= { _ =>
      gcd(b, a % b)
    }

gcd(8, 3).run
~~~~~~

[What stands out for immutable collection is Vector since
it has effective constant for all operations.](http://docs.scala-lang.org/overviews/collections/performance-characteristics.html)

~~~~~~
def gcd(a: Int, b: Int): Writer[Vector[String], Int] =
  if (b == 0) for {
      _ <- Vector("Finished with " + a.shows).tell
    } yield a
  else for {
      result <- gcd(b, a % b)
      _ <- Vector(a.shows + " mod " + b.shows + " = " + (a % b).shows).tell
    } yield result
~~~~~~

性能比较

~~~~~~
import std.vector._

def vectorFinalCountDown(x: Int): Writer[Vector[String], Unit] = {
  import annotation.tailrec
  @tailrec def doFinalCountDown(x: Int, w: Writer[Vector[String], Unit]): Writer[Vector[String], Unit] = x match {
    case 0 => w >>= { _ => Vector("0").tell }
    case x => doFinalCountDown(x - 1, w >>= { _ =>
      Vector(x.shows).tell
    })
  }
  val t0 = System.currentTimeMillis
  val r = doFinalCountDown(x, Vector[String]().tell)
  val t1 = System.currentTimeMillis
  r >>= { _ => Vector((t1 - t0).shows + " msec").tell }
}

def listFinalCountDown(x: Int): Writer[List[String], Unit] = {
  import annotation.tailrec
  @tailrec def doFinalCountDown(x: Int, w: Writer[List[String], Unit]): Writer[List[String], Unit] = x match {
    case 0 => w >>= { _ => List("0").tell }
    case x => doFinalCountDown(x - 1, w >>= { _ =>
      List(x.shows).tell
    })
  }
  val t0 = System.currentTimeMillis
  val r = doFinalCountDown(x, List[String]().tell)
  val t1 = System.currentTimeMillis
  r >>= { _ => List((t1 - t0).shows + " msec").tell }
}
~~~~~~

## Reader ##

~~~~~~
// function as a functor
scala> val f = (_: Int) * 5
f: Int => Int = <function1>

scala> val g = (_: Int) + 3
g: Int => Int = <function1>

scala> (g map f)(8)
res22: Int = 55

// function as a applicative
scala> val f = ({(_: Int) * 2} |@| {(_: Int) + 10}) {_ + _}
f: Int => Int = <function1>

scala> f(3)
res35: Int = 19

// function as a monad
scala> val addStuff: Int => Int = for {
         a <- (_: Int) * 2
         b <- (_: Int) + 10
       } yield a + b
addStuff: Int => Int = <function1>

scala> addStuff(3)
res39: Int = 19
~~~~~~
