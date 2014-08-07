title: scala培训monad
date: 2014-06-22 10:14:17
tags:
- scala
---

# monad #

* magic
* category
* theory
* design pattern
* api

函数式争论, 可赋值和不可赋值(运输的隐喻, 货车(可以赋值)和石油管道(Monad))

## scala中的Monad ##
可扩展, 可重用, 可测试

* Option, 实现 `map` 和 `flatmap`
~~~~~~
def getPrice(): Option[Int];
def getQuantities():Option[Int];

def amount():Option[Int] =
    getPrice().flatMap(price => getQuantities().map(price * _));

def amount():Option[Int] = {
    for{
        price <- getPrice()
        quantities <- getQuantities()
    } yield price * quantities;
}
~~~~~~
* Try
~~~~~~
// java, 命令式, 不可重用
try{
   val conn = DriverManager.getConnection(url, user, password);
} catch {
   case e:* => 
}

def getDriver():Try[String];

for(driver <- getDriver()) yield driver
~~~~~~
* Future
* all collections
~~~~~~
// 八皇后问题
def queen(n:Int) = {
    def placeQueens(k: Int): List[List[(Int, Int)]] =
        if(k==0) List(List())
        else for {
            
        }
}
~~~~~~


## Monad是一个设计模式 ##

* 不确定性技术
* 异常处理
* 并发
* 解析, **scalaz**, `ValidationNEL`
* 持续计算
* 输入, 输出
~~~~~~scala:

def greet {
    println("请输入")
    val key = readLine
    println("您输入的是" + key)
}

// Monad
cass class IO[A](run: () => A) {
   def map[B](f: A=> B): IO[B]= IO(() => f(run()))
   def flatMap[B](f: A => IO[B]) :IO[B] => f(run())
}

// 将调用包装成函数
def io[A](a: => A): IO[A] = IO(() => a)

def putLine(a: String): IO[Unit] = io(println(s))
def getLine: IO[String] = io(readLine)

// 函数的调用和传递, 返回一个函数
def greet: IO[Unit] = for {
    _ <- putLine("")
    name <- getLine
    result <- putLine("")
} yield ()

// 运行函数
greet.run();
~~~~~~

* 可变性计算

**sap**

# Reactive with Akka #

可用:
* 出错了还能用
* 高负荷, 高压力下

可扩展, 可恢复, 可回应

actor:
* 具有ID
* 具有行为
* 交流方式是异步(synchronous)
* 一个actor是单线程

时间驱动(Event Driven)
* Event是头等函数
* 消息会回复
* 消息会被存到一个 Queue
* 消息可以分发

~~~~~~
class Greeter extends Actor {
    var greeting = ""
    def receive = {
       case Greet => sender ! Greeting(greeting) // 
    }
}

val system = ActorSystem("helloakka")

system.actorOf()
~~~~~~

