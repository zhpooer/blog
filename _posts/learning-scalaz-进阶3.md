title: learning scalaz 进阶3
date: 2014-07-30 14:43:47
tags:
- scala
- scalaz
---

# Origami programming #

The dual of folding is unfolding.
The Haskell standard List library deﬁnes
the function unfoldr for generating lists.

~~~~~~
Prelude Data.List> unfoldr (\b -> if b == 0 then Nothing else Just (b, b-1)) 10
[10,9,8,7,6,5,4,3,2,1]
~~~~~~

## DList ##

There’s a data structure called DList that supports `DList.unfoldr`.
DList, or difference list, is a data structure that supports
constant-time appending.

~~~~~~
scala> DList.unfoldr(10, { (x: Int) => if (x == 0) none else (x, x - 1).some })
res50: scalaz.DList[Int] = scalaz.DListFunctions$$anon$3@70627153

scala> res50.toList
res51: List[Int] = List(10, 9, 8, 7, 6, 5, 4, 3, 2, 1)
~~~~~~

## Folds for Streams ##

In Scalaz unfold defined in StreamFunctions is introduced by `import Scalaz._`:

~~~~~~
scala> unfold(10) { (x) => if (x == 0) none else (x, x - 1).some }
res36: Stream[Int] = Stream(10, ?)

scala> res36.toList
res37: List[Int] = List(10, 9, 8, 7, 6, 5, 4, 3, 2, 1)
~~~~~~

## The Essence of the Iterator Pattern ##
In 2006 the same author wrote [The Essence of the Iterator Pattern](http://www.cs.ox.ac.uk/jeremy.gibbons/publications/iterator.pdf).
This paper discusses applicative style by breaking down the GoF Iterator
pattern into two aspects: mapping and accumulating.


## Monoidal applicatives ##
Scalaz implements Monoid[m].applicative to turn any monoids into
an applicative.

~~~~~~
scala> Monoid[Int].applicative.ap2(1, 1)(0)
res99: Int = 2

scala> Monoid[List[Int]].applicative.ap2(List(1), List(1))(Nil)
res100: List[Int] = List(1, 1)
~~~~~~

## Combining applicative functors ##

Like monads, applicative functors are closed under products;
so two independent idiomatic effects can generally be fused into one,
their product.

In Scalaz, product is implemented under Applicative typeclass

~~~~~~
scala> Applicative[List].product[Option]
res0: scalaz.Applicative[[α](List[α], Option[α])] = scalaz.Applicative$$anon$2@211b3c6a

scala> Applicative[List].product[Option].point(1)
res1: (List[Int], Option[Int]) = (List(1),Some(1))

// The product seems to be implemented as a Tuple2. 
scala> ((List(1), 1.some) |@| (List(1), 1.some)) {_ |+| _}
res2: (List[Int], Option[Int]) = (List(1, 1),Some(2))

scala> ((List(1), 1.success[String]) |@| (List(1), "boom".failure[Int])) {_ |+| _}
res6: (List[Int], scalaz.Validation[String,Int]) = (List(1, 1),Failure(boom))
~~~~~~

two sequentially-dependent idiomatic effects
can generally be fused into one, their composition.

~~~~~~
trait Applicative[F[_]] extends Apply[F] with Pointed[F] { self =>
...
  /**The composition of Applicatives `F` and `G`, `[x]F[G[x]]`, is an Applicative */
  def compose[G[_]](implicit G0: Applicative[G]): Applicative[({type λ[α] = F[G[α]]})#λ] = new CompositionApplicative[F, G] {
    implicit def F = self
    implicit def G = G0
  }
...
}

scala> Applicative[List].compose[Option]
res7: scalaz.Applicative[[α]List[Option[α]]] = scalaz.Applicative$$anon$1@461800f1

scala> Applicative[List].compose[Option].point(1)
res8: List[Option[Int]] = List(Some(1))
~~~~~~

## Idiomatic traversal ##

The corresponding typeclass in Scalaz 7 is called Traverse:
~~~~~~
trait Traverse[F[_]] extends Functor[F] with Foldable[F] { self =>
  def traverseImpl[G[_]:Applicative,A,B](fa: F[A])(f: A => G[B]): G[F[B]]
}

trait TraverseOps[F[_],A] extends Ops[F[A]] {
  final def traverse[G[_], B](f: A => G[B])(implicit G: Applicative[G]): G[F[B]] =
    G.traverse(self)(f)
  ...
}
~~~~~~

~~~~~~
scala> List(1, 2, 3) traverse { x => (x > 0) option (x + 1) }
res14: Option[List[Int]] = Some(List(2, 3, 4))

scala> List(1, 2, 0) traverse { x => (x > 0) option (x + 1) }
res15: Option[List[Int]] = None
~~~~~~

## sequence ##

There’s a useful method that Traverse introduces called sequence.
~~~~~~
/** Traverse with the identity function */
final def sequence[G[_], B](implicit ev: A === G[B], G: Applicative[G]): G[F[B]] = {
  val fgb: F[G[B]] = ev.subst[F](self)
  F.sequence(fgb)
}
~~~~~~

~~~~~~
scala> List(1.some, 2.some).sequence
res156: Option[List[Int]] = Some(List(1, 2))

scala> List(1.some, 2.some, none).sequence
res157: Option[List[Int]] = None
~~~~~~


# import guide #

In Scala, imports are used for two purposes:
1. To include names of values and types into the scope.
2. To include implicits into the scope.


## import scalaz._ ##

1. First, the names. Typeclasses like `Equal[A]` and `Functor[F[_]]`
are implemented as trait,
and are defined under scalaz package.
2. also the names, but type aliases. scalaz’s package object
declares most of the major type aliases like `@@[T, Tag]` and `Reader[E, A]`,
which is treated as a specialization of ReaderT transformer.
3. idInstance is defined as typeclass instance of `Id[A]` for `Traverse[F[_]]`, `Monad[F[_]]`

## import Scalaz._ ##

~~~~~~
package scalaz

object Scalaz
  extends StateFunctions        // Functions related to the state monad
  with syntax.ToTypeClassOps    // syntax associated with type classes
  with syntax.ToDataOps         // syntax associated with Scalaz data structures
  with std.AllInstances         // Type class instances for the standard library types
  with std.AllFunctions         // Functions related to standard library types
  with syntax.std.ToAllStdOps   // syntax associated with standard library types
  with IdInstances              // Identity type and instances
~~~~~~

### StateFunctions ###

Remember, import brings in names and implicits. First, the names.
StateFunctions defines several functions:
~~~~~~
package scalaz

trait StateFunctions {
  def constantState[S, A](a: A, s: => S): State[S, A] = ...
  def state[S, A](a: A): State[S, A] = ...
  def init[S]: State[S, S] = ...
  def get[S]: State[S, S] = ...
  def gets[S, T](f: S => T): State[S, T] = ...
  def put[S](s: S): State[S, Unit] = ...
  def modify[S](f: S => S): State[S, Unit] = ...
  def delta[A](a: A)(implicit A: Group[A]): State[A, A] = ...
}
~~~~~~

~~~~~~
for {
  xs <- get[List[Int]]
  _ <- put(xs.tail)
} yield xs.head
~~~~~~

### std.AllFunctions ###
~~~~~~
package scalaz
package std

trait AllFunctions
  extends ListFunctions
  with OptionFunctions
  with StreamFunctions
  with math.OrderingFunctions
  with StringFunctions

object AllFunctions extends AllFunctions
~~~~~~

For example, ListFunctions bring in intersperse function that puts
a given element in ever other position:
~~~~~~
intersperse(List(1, 2, 3), 7)
~~~~~~

### IdInstances ###

defines the type alias `Id[A]` as follows:

~~~~~~
type Id[+X] = X
~~~~~~

### std.AllInstances ###

the fact that list is a monad and that
monad introduces `>>=` operator are two different things.

one of the most interesting design of scalaz 7 is that it
rigorously separates the two concepts into “instance” and “syntax.”

`std.allinstances` is a mixin of typeclass instances for built-in (std) data structures:

~~~~~~
package scalaz.std

trait allinstances
  extends anyvalinstances with functioninstances with listinstances with mapinstances
  with optioninstances with setinstances with stringinstances with streaminstances with tupleinstances
  with eitherinstances with partialfunctioninstances with typeconstraintinstances
  with scalaz.std.math.bigdecimalinstances with scalaz.std.math.bigints
  with scalaz.std.math.orderinginstances
  with scalaz.std.util.parsing.combinator.parsers
  with scalaz.std.java.util.mapinstances
  with scalaz.std.java.math.bigintegerinstances
  with scalaz.std.java.util.concurrent.callableinstances
  with nodeseqinstances
  // intentionally omitted: iterableinstances

object allinstances extends allinstances
~~~~~~

### syntax.totypeclassops ###

~~~~~~
package scalaz
package syntax

trait totypeclassops
  extends tosemigroupops with tomonoidops with togroupops with toequalops with tolengthops with toshowops
  with toorderops with toenumops with tometricspaceops with toplusemptyops with toeachops with toindexops
  with tofunctorops with topointedops with tocontravariantops with tocopointedops with toapplyops
  with toapplicativeops with tobindops with tomonadops with tocojoinops with tocomonadops
  with tobifoldableops with tocozipops
  with toplusops with toapplicativeplusops with tomonadplusops with totraverseops with tobifunctorops
  with tobitraverseops with toarridops with tocomposeops with tocategoryops
  with toarrowops with tofoldableops with tochoiceops with tosplitops with tozipops with tounzipops with tomonadwriterops with tolistenablemonadwriterops
~~~~~~

### syntax.todataops ###

~~~~~~
trait todataops extends toidops with totreeops with towriterops with tovalidationops with toreducerops with tokleisliops

///
package scalaz.syntax

trait idops[a] extends ops[a] {
  final def ??(d: => a)(implicit ev: null <:< a): a = ...
  final def |>[b](f: a => b): b = ...
  final def squared: (a, a) = ...
  def left[b]: (a \/ b) = ...
  def right[b]: (b \/ a) = ...
  final def wrapnel: nonemptylist[a] = ...
  def matchorzero[b: monoid](pf: partialfunction[a, b]): b = ...
  final def dowhile(f: a => a, p: a => boolean): a = ...
  final def whiledo(f: a => a, p: a => boolean): a = ...
  def visit[f[_] : pointed](p: partialfunction[a, f[a]]): f[a] = ...
}

trait toidops {
  implicit def toidops[a](a: a): idops[a] = new idops[a] {
    def self: a = a
  }
}

///
package scalaz
package syntax

trait treeops[a] extends ops[a] {
  def node(subforest: tree[a]*): tree[a] = ...
  def leaf: tree[a] = ...
}

trait totreeops {
  implicit def totreeops[a](a: a) = new treeops[a]{ def self = a }
}
~~~~~~

the same goes for `writerops[a]`, `validationops[a]`,
`reducerops[a]`, and `kleisliidops[a]`:

~~~~~~
scala> 1.node(2.leaf)
res7: scalaz.tree[int] = <tree>

scala> 1.set("log1")
res8: scalaz.writer[string,int] = scalaz.writertfunctions$$anon$26@2375d245

scala> "log2".tell
res9: scalaz.writer[string,unit] = scalaz.writertfunctions$$anon$26@699289fb

scala> 1.success[string]
res11: scalaz.validation[string,int] = success(1)

scala> "boom".failurenel[int]
res12: scalaz.validationnel[string,int] = failure(nonemptylist(boom))
~~~~~~

### syntax.std.toallstdops ###

introduces methods and operators to scala’s standard types.

~~~~~~
package scalaz
package syntax
package std

trait toallstdops
  extends tobooleanops with tooptionops with tooptionidops with tolistops with tostreamops
  with tofunction2ops with tofunction1ops with tostringops with totupleops with tomapops with toeitherops
~~~~~~

~~~~~~
scala> false /\ true
res14: boolean = false

scala> false \/ true
res15: boolean = true

scala> true option "foo"
res16: option[string] = some(foo)

scala> (1 > 10)? "foo" | "bar"
res17: string = bar

scala> (1 > 10)?? {list("foo")}
res18: list[string] = list()


scala> 1.some? "foo" | "bar"
res28: string = foo

scala> 1.some | 2
res30: int = 1


scala> list(1, 2) filterm {_ => list(true, false)}
res37: list[list[int]] = list(list(1, 2), list(1), list(2), list())
~~~~~~

# hacking on a project #

~~~~~~
git clone -b scalaz-seven git://github.com/scalaz/scalaz.git scalaz-seven
git branch topic/vectorinstance
git co topic/vectorinstance
~~~~~~

What’s actually going on is not just the combination of applicative
`functors (m ⊠ n)`, but the combination of applicative functions:

~~~~~~
(⊗)::(Functor m,Functor n) ⇒ (a → m b) → (a → n b) → (a → (m ⊠ n) b)
(f ⊗ g) x = Prod (f x) (g x)
~~~~~~

Int is a Monoid, and any Monoid can be treated as an applicative functor,
which is called monoidal applicatives.
The problem is that when we make that into a function, it’s not distinguishable
from `Int => Int`, but we need `Int => [α]Int`


TODO

# Arrow #

An arrow is the term used in category theory as an abstract
notion of thing that behaves like a function.

In Scalaz, these are `Function1[A, B]`, `PartialFunction[A, B]`,
`Kleisli[F[_], A, B], and CoKleisli[F[_], A, B]`. Arrow
abstracts them all similar to the way other typeclasses abtracts containers.

~~~~~~
// Looks like Arrow[=>:[_, _]] extends Category[=>:].
trait Arrow[=>:[_, _]] extends Category[=>:] { self =>
  def id[A]: A =>: A
  def arr[A, B](f: A => B): A =>: B
  def first[A, B, C](f: (A =>: B)): ((A, C) =>: (B, C))
}
~~~~~~

## Category and Compose ##
~~~~~~
trait Category[=>:[_, _]] extends Compose[=>:] { self =>
  /** The left and right identity over `compose`. */
  def id[A]: A =>: A
}

trait Compose[=>:[_, _]]  { self =>
  def compose[A, B, C](f: B =>: C, g: A =>: B): (A =>: C)
}
~~~~~~

compose function composes two arrows into one. Using compose, Compose
introduces the following operators:

~~~~~~
trait ComposeOps[F[_, _],A, B] extends Ops[F[A, B]] {
  final def <<<[C](x: F[C, A]): F[C, B] = F.compose(self, x)
  final def >>>[C](x: F[B, C]): F[A, C] = F.compose(x, self)
}
~~~~~~

The meaning of `>>>` and `<<<` depends on the arrow, but for functions,
it’s the same as andThen and compose:

~~~~~~
scala> val f = (_:Int) + 1
f: Int => Int = <function1>

scala> val g = (_:Int) * 100
g: Int => Int = <function1>

scala> (f >>> g)(2)
res0: Int = 300

scala> (f <<< g)(2)
res1: Int = 201
~~~~~~

## Arrow, again ##

`=>:[A, B]` as `A =>: B`

~~~~~~
trait ArrowOps[F[_, _],A, B] extends Ops[F[A, B]] {
  final def ***[C, D](k: F[C, D]): F[(A, C), (B, D)] = F.splitA(self, k)
  final def &&&[C](k: F[A, C]): F[A, (B, C)] = F.combine(self, k)
  ...
}
~~~~~~

~~~~~~
scala> val f = (_:Int) + 1
f: Int => Int = <function1>

scala> val g = (_:Int) * 100
g: Int => Int = <function1>

// combines two arrows into a new arrow by running
// the two arrows on a pair of values
scala> (f *** g)(1, 2)
res3: (Int, Int) = (2,200)

// combines two arrows into a new arrow by running the
// two arrows on the same value
scala> (f &&& g)(2)
res4: (Int, Int) = (3,200)
~~~~~~

# Unapply #

One thing that I’ve been fighting the Scala compiler over
is the lack of type inference support across the different kinded types
like `F[M[_, _]]` and `F[M[_]]`, and `M[_] and F[M[_]]`.

~~~~~~
scala> Applicative[Function1[Int, Int]]
<console>:14: error: Int => Int takes no type parameters, expected: one
              Applicative[Function1[Int, Int]]
                          ^
// an instance of Applicative[M[_]] is (* -> *) -> *                          
scala> Applicative[({type l[A]=Function1[Int, A]})#l]
res14: scalaz.Applicative[[A]Int => A] = scalaz.std.FunctionInstances$$anon$2@56ae78ac
~~~~~~

One of the way Scalaz helps you out is to provide
meta-instances of typeclass instance called Unapply.

~~~~~~
trait Unapply[TC[_[_]], MA] {
  /** The type constructor */
  type M[_]
  /** The type that `M` was applied to */
  type A
  /** The instance of the type class */
  def TC: TC[M]
  /** Evidence that MA =:= M[A] */
  def apply(ma: MA): M[A]
}
~~~~~~

TODO

## parallel composition ##

TODO

# Momo #

Pure functions don’t imply they are computationally cheap.

Given you have some space in RAM, we could trade some of the
expensive calculations for space by caching the result.
This is called memoization.

~~~~~~
sealed trait Memo[@specialized(Int) K, @specialized(Int, Long, Double) V] {
  def apply(z: K => V): K => V
}
~~~~~~
We pass in a potentially expensive function as an input
and you get back a function that behaves the same but may cache the result.

Memo object there are some default implementations of Memo like
`Memo.mutableHashMapMemo[K, V]`, `Memo.weakHashMapMemo[K, V]`, and `Memo.arrayMemo[V]`.

缓存每次运算的结果
~~~~~~
scala> val slowFib: Int => Int = {
         case 0 => 0
         case 1 => 1
         case n => slowFib(n - 2) + slowFib(n - 1)
       }
slowFib: Int => Int = <function1>

scala> slowFib(45)
res2: Int = 1134903170

scala> val memoizedFib: Int => Int = Memo.mutableHashMapMemo {
         case 0 => 0
         case 1 => 1
         case n => memoizedFib(n - 2) + memoizedFib(n - 1)
       }
memoizedFib: Int => Int = <function1>

scala> memoizedFib(45)
res14: Int = 1134903170
~~~~~~

# functional programming #
An expression e is referentially transparent if every occurrence e can
be replaced with its value without affecting the observable result of the program.

# Effect system #

## ST ##

~~~~~~
sealed trait ST[S, A] {
  private[effect] def apply(s: World[S]): (World[S], A)
}
~~~~~~


This looks similar to State monad, but the difference I think is that
the state is mutated in-place, and in return is not observable from outside.


## STRef ##

STRef is a mutable variable that’s used only within
the context of ST monad. It’s created using ST.

~~~~~~
sealed trait STRef[S, A] {
  protected var value: A

  /**Reads the value pointed at by this reference. */
  def read: ST[S, A] = returnST(value)
  /**Modifies the value at this reference with the given function. */
  def mod[B](f: A => A): ST[S, STRef[S, A]] = ...
  /**Associates this reference with the given value. */
  def write(a: => A): ST[S, STRef[S, A]] = ...
  /**Synonym for write*/
  def |=(a: => A): ST[S, STRef[S, A]] = ...
  /**Swap the value at this reference with the value at another. */
  def swap(that: STRef[S, A]): ST[S, Unit] = ...
}

sealed trait STArray[S, A] {
  val size: Int
  val z: A
  private val value: Array[A] = Array.fill(size)(z)
  /**Reads the value at the given index. */
  def read(i: Int): ST[S, A] = returnST(value(i))
  /**Writes the given value to the array, at the given offset. */
  def write(i: Int, a: A): ST[S, STArray[S, A]] = ...
  /**Turns a mutable array into an immutable one which is safe to return. */
  def freeze: ST[S, ImmutableArray[A]] = ...
  /**Fill this array from the given association list. */
  def fill[B](f: (A, B) => A, xs: Traversable[(Int, B)]): ST[S, Unit] = ...
  /**Combine the given value with the value at the given index, using the given function. */
  def update[B](f: (A, B) => A, i: Int, v: B) = ...
}
~~~~~~

~~~~~~
scala> import effect._
import effect._

scala> import ST.{newVar, runST, newArr, returnST}
import ST.{newVar, runST, newArr, returnST}

scala> def e1[S] = for {
         x <- newVar[S](0)
         r <- x mod {_ + 1}
       } yield x
e1: [S]=> scalaz.effect.ST[S,scalaz.effect.STRef[S,Int]]

scala> def e2[S]: ST[S, Int] = for {
         x <- e1[S]
         r <- x.read
       } yield r 
e2: [S]=> scalaz.effect.ST[S,Int]

scala> type ForallST[A] = Forall[({type λ[S] = ST[S, A]})#λ]
defined type alias ForallST

scala> runST(new ForallST[Int] { def apply[S] = e2[S] })
res5: Int = 1
~~~~~~

~~~~~~
"STArray" in {
  def e1[S] = for {
    arr <- newArr[S, Boolean](3, true)
    _ <- arr.write(0, false)
    r <- arr.freeze
  } yield r
  runST(new ForallST[ImmutableArray[Boolean]] { def apply[S] = e1[S] }).toList must be_===(
    List(false, true, true))
}
~~~~~~

~~~~~~
scala> def mapM[A, S, B](xs: List[A])(f: A => ST[S, B]): ST[S, List[B]] =
         Monad[({type λ[α] = ST[S, α]})#λ].sequence(xs map f)
mapM: [A, S, B](xs: List[A])(f: A => scalaz.effect.ST[S,B])scalaz.effect.ST[S,List[B]]

scala> def sieve[S](n: Int) = for {
         arr <- newArr[S, Boolean](n + 1, true)
         _ <- arr.write(0, false)
         _ <- arr.write(1, false)
         val nsq = (math.sqrt(n.toDouble).toInt + 1)
         _ <- mapM (1 |-> nsq) { i =>
           for {
             x <- arr.read(i)
             _ <-
               if (x) mapM (i * i |--> (i, n)) { j => arr.write(j, false) }
               else returnST[S, List[Boolean]] {Nil}
           } yield ()
         }
         r <- arr.freeze
       } yield r
sieve: [S](n: Int)scalaz.effect.ST[S,scalaz.ImmutableArray[Boolean]]

scala> type ForallST[A] = Forall[({type λ[S] = ST[S, A]})#λ]
defined type alias ForallST

scala> def prime(n: Int) =
         runST(new ForallST[ImmutableArray[Boolean]] { def apply[S] = sieve[S](n) }).toArray.
         zipWithIndex collect { case (true, x) => x }
prime: (n: Int)Array[Int]

scala> prime(1000)
~~~~~~

# IO Monad #

~~~~~~
sealed trait IO[+A] {
  private[effect] def apply(rw: World[RealWorld]): Trampoline[(World[RealWorld], A)]
}
~~~~~~

~~~~~~
scala> import scalaz._, Scalaz._, effect._, IO._
scala> val action1 = for {
         _ <- putStrLn("Hello, world!")
       } yield ()
action1: scalaz.effect.IO[Unit] = scalaz.effect.IOFunctions$$anon$4@149f6f65

scala> action1.unsafePerformIO
Hello, world!
~~~~~~

IO actions
~~~~~~
  /** Reads a character from standard input. */
  def getChar: IO[Char] = ...
  /** Writes a character to standard output. */
  def putChar(c: Char): IO[Unit] = ...
  /** Writes a string to standard output. */
  def putStr(s: String): IO[Unit] = ...
  /** Writes a string to standard output, followed by a newline.*/
  def putStrLn(s: String): IO[Unit] = ...
  /** Reads a line of standard input. */
  def readLn: IO[String] = ...
  /** Write the given value to standard output. */
  def putOut[A](a: A): IO[Unit] = ...
  // Mutable variables in the IO monad
  def newIORef[A](a: => A): IO[IORef[A]] = ...
  /**Throw the given error in the IO monad. */
  def throwIO[A](e: Throwable): IO[A] = ...
  /** An IO action that does nothing. */
  val ioUnit: IO[Unit] = ...
}
~~~~~~

~~~~~~
scala> val action2 = IO {
         val source = scala.io.Source.fromFile("./README.md")
         source.getLines.toStream
       }
action2: scalaz.effect.IO[scala.collection.immutable.Stream[String]] = scalaz.effect.IOFunctions$$anon$4@bab4387

scala> action2.unsafePerformIO.toList
res57: List[String] = List(# Scalaz, "", Scalaz is a Scala library for functional programming., "", It provides purely functional data structures to complement those from the Scala standard library., ...
~~~~~~

~~~~~~
def program: IO[Unit] = for {
  line <- readLn
  _    <- putStrLn(line)
} yield ()

scala> (program |+| program).unsafePerformIO
123
123
~~~~~~

## Enumeration-Based I/O with Iteratees ##

~~~~~~
sealed trait Input[E] {
  def fold[Z](empty: => Z, el: (=> E) => Z, eof: => Z): Z
  def apply[Z](empty: => Z, el: (=> E) => Z, eof: => Z) =
    fold(empty, el, eof)
}

sealed trait IterateeT[E, F[_], A] {
  def value: F[StepT[E, F, A]]
}
type Iteratee[E, A] = IterateeT[E, Id, A]

object Iteratee
  extends IterateeFunctions
  with IterateeTFunctions
  with EnumeratorTFunctions
  with EnumeratorPFunctions
  with EnumerateeTFunctions
  with StepTFunctions
  with InputFunctions {

  def apply[E, A](s: Step[E, A]): Iteratee[E, A] = iteratee(s)
}

type >@>[E, A] = Iteratee[E, A]
~~~~~~
Let’s try implementing the counter example from EBIOI.
For that we switch to iteratee project using sbt:
~~~~~~
sealed trait IterateeT[E, F[_], A] {
  def value: F[StepT[E, F, A]]
}
type Iteratee[E, A] = IterateeT[E, Id, A]

object Iteratee
  extends IterateeFunctions
  with IterateeTFunctions
  with EnumeratorTFunctions
  with EnumeratorPFunctions
  with EnumerateeTFunctions
  with StepTFunctions
  with InputFunctions {

  def apply[E, A](s: Step[E, A]): Iteratee[E, A] = iteratee(s)
}

type >@>[E, A] = Iteratee[E, A]
~~~~~~
