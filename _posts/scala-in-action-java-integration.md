title: Scala in Action-Java integration
date: 2014-07-25 20:51:27
tags:
- scala
---

# Using Java classes in Scala #
~~~~~~
import org.joda.time.DateTime;
import org.joda.time.Days;
import java.util.Date;

public class DateCalculator {
    public int daysBetween(Date start, Date end) {
        Days d = Days.daysBetween(new DateTime(start.getTime()),
        new DateTime(end.getTime()));
        return d.getDays();
    }
}
~~~~~~

~~~~~~
import chap11.java._
import java.util.Date
class PaymentCalculator(val payPerDay: Int = 100) extends DateCalculator {
    def calculatePayment(start: Date, end: Date) = {
        daysBetween(start, end) * payPerDay
    }
}
~~~~~~

# Working with Java checked exceptions #
he Scala compiler won’t force you. In cases where you think you should
catch the exception, don’t rethrow the exception from Scala. It’s a bad practice.
A better way is to create an instance of the Either or Option type. 

~~~~~~
def write(content: String): Either[Exception, Boolean] = {
  val w = new Writer
  try {
    w.writeToFile(content)
    Right(true)
  } catch {
    case e: java.io.IOException => Left(e)
  }
}
~~~~~~

# Working with Java generics using existential types #
`Vector<?>` could be represented as `Vector[T] forSome { type T }`
in Scala. Reading from left to right, this type
expression represents a vector of T for some type of T .
This type T is unknown and could be anything.
But T is fixed to some type for this vector.

~~~~~~
import java.util.{ Vector => JVector }
def printLanguages[C <: JVector[T] forSome { type T}](langs: C):Unit = {
  for(i <- 0 until langs.size) println(langs.get(i))
}
~~~~~~

There’s placeholder syntax for existential type `JVector[_]`. It means the same
thing as `JVector[T] forSome { type T }`. The preceding printLanguages method
could also be written as follows:
~~~~~~
def printLanguages[C <: JVector[_]](langs: C):Unit = {
  for(i <- 0 until langs.size) println(langs.get(i))
}
~~~~~~

## Working with Java collections ##
The Scala library ships with two utility classes that do
exactly that for you:
~~~~~~
scala.collection.JavaConversions
scala.collection.JavaConverters
~~~~~~
`JavaConversions` provides a series of implicit conversions that convert
between a Java collection and the closest corresponding Scala collection, and vice
versa. `JavaConverters` uses a “Pimp my Library” pattern to add the asScala
method to Java collection and asJava method to Scala collection types.

My recommendation would be to use JavaConverters because it makes the conversion
explicit.
~~~~~~
scala> import java.util.{ArrayList => JList }
import java.util.{ArrayList => JList}
scala> val jList = new JList[Int]()
jList: java.util.ArrayList[Int] = []
scala> jList.add(1)
res1: Boolean = true
scala> jList.add(2)
res2: Boolean = true
scala> import scala.collection.JavaConverters._
import scala.collection.JavaConverters._
scala> jList.asScala foreach println
~~~~~~

# Building web applications in Scala using Java frameworks #
~~~~~~
mvn archetype:generate -DgroupId=scala.in.action -DartifactId=top.artists
-DarchetypeArtifactId=maven-archetype-webapp
~~~~~~
