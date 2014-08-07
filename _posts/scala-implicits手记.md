title: Scala Implicits手记
date: 2014-06-28 09:57:30
tags:
- scala
---

[详细说明](http://debasishg.blogspot.com/2010/06/scala-implicits-type-classes-here-i.html)

传统的代理模式
~~~~~~
case class Address(no: Int, street: String, city: String, 
  state: String, zip: String)

trait LabelMaker[T] {
  def toLabel(value: T): String
}


// adapter class
case class AddressLabelMaker extends LabelMaker[Address] {
  def toLabel(address: Address) = {
    import address._
    "%d %s, %s, %s - %s".format(no, street, city, state, zip)
  }
}

// the adapter provides the interface of the LabelMaker on an Address
AddressLabelMaker().toLabel(Address(100, "Monroe Street", "Denver", "CO", "80231"))
~~~~~~

在scala中可以这样
~~~~~~
object LabelMaker {
  implicit object AddressLabelMaker extends LabelMaker[Address] {
    def toLabel(address: Address): String = {
      import address._
      "%d %s, %s, %s - %s".format(no, street, city, state, zip)
    }
  }
}

def printLabel[T](t: T)(implicit lm: LabelMaker[T]) = lm.toLabel(t)
// 简化成这样, 可以理解 隐式捕获 和 T 相关的LabelMaker, 语法糖
def printLabel[T: LabelMaker](t: T) = implicitly[LabelMaker[T]].toLabel(t)

~~~~~~
