title: Akka in Action-Integration
date: 2014-08-06 14:45:54
tags:
- akka
- scala
---

# Message endpoints #

The implementation of an interface between two systems isn't always
easy, because the interface contains two areas: the transport layer and the data
which is sent over this transport layer.

For example, we are creating an order system for use in a book
stockroom, that processes orders from all kinds of customers. These customers can
order the books by visiting the store. The bookstore already uses an application to
sell and order books.  So the new system needs to exchange data with this existing
application.

Because you probably can't change the external
application, you have to create a component that can send and/or receive messages
from the existing application. This component is called an endpoint. Endpoints are
part of your system and are the glue between the external system and the rest of
your system.

There are a lot of different transport protocols to potentially
support: REST/HTTP, TCP, MQueues, or simple files. 

## Normalizer ##

We have seen that our order system receives the orders from the bookshop
application, but it is possible that our system also receives orders from a web shop,
or by customers sending email.

We can use the Normalizer pattern to make these
different sources all feed into a single interface on the application side. The pattern
translates the different external messages to a common, canonical message.

We create three different endpoints to consume the different messages, but
translate them into the same message, which is sent to the rest of the system.

![akka order system](/img/akka_order_system.png)

Translating the different messages into a common message is called the
Normalizer pattern.

 Let us assume that
there is another bookshop that is connecting to this system using the same
messages but using MQueue to send those message.

![akka normalizer partern](/img/akka_normalizer.png)


## Canonical Data Model ##
 But when the connectivity requirements between the systems
increases we need more and more endpoints.

To solve this problem, we can use the Canonical Data Model. This pattern
connects multiple applications using interface(s) that are independent of any
specific system.

When the Bookshop application
wants to send a message to the order system, the message is first translated to the
canonical format and then it is sent using the common transport layer.

The Normalizer
pattern is used to connect several similar clients to another system. But when the
number of integrated systems increases, we need the Canonical Data Model, which
looks like the Normalizer Pattern, because it also uses normalized messages.

The difference is that the Canonical Data Model provides an additional level of
indirection between the application's individual data formats and those used by the
remote systems. While the Normalizer is only within one application.

# Camel Framework #

Camel is an apache framework whose goal is to make integration easier and more
accessible. It makes it possible to implement the standard enterprise integration
patterns in a few lines of code. This is achieved by addressing three areas:
1. Concrete implementations of the widely used Enterprise Integration Patterns
2. Connectivity to a great variety of transports and APIs
3. Easy to use Domain Specific Languages (DSLs) to wire EIPs and transports together

The Camel module works internally with Camel classes. Important Camel
classes are the camel context and the ProducerTemplate. The CamelContext
represents a single Camel routing rule base, and the ProducerTemplate is needed
when producing messages.

## Implement a consumer endpoint ##

The example we are going to implement is an Order System receiving messages
from a bookshop. 
Let's say the received messages are XML files in a
directory. The transport layer is in this case the file system. The endpoint of the
order system needs to track new files and when there is a new file it has to parse
the XML content and create a message the system can process.

~~~~~~
// Consumer Endpoint

import akka.camel.{CamelMessage, Consumer}

class OrderConsumerXml(uri:String, next:ActorRef) extends Consumer{
  // override the endpointUri
  def endpointUri = uri

  // receive the Camel message
  def receive = {
    case msg:CamelMessage => {
      val content = msg.bodyAs[String]
      val xml = XML.loadString(content)
      val order = xml\\"order"
      val customer = (order \\ "customerId").text
      val productId = (order \\ "productId").text
      val number = (order \\ "number").text.toInt
      next ! new Order(customer, productId, number)
    }
  }
}

// This Uri starts with the Camel component.
// http://camel.apache.org/components.html
val camelUri = "file:messages"
~~~~~~

When a new message is received, it comes to the Actor through its usual
method, as a CamelMessage. A CamelMessage contains a body, which is the
actual message received, and a map of headers. The content of these headers
depends on the protocol used. 

~~~~~~
val probe = TestProbe()
val camelUri = "file:messages"
val consumer = system.actorOf(Props(new OrderConsumerXml(camelUri, probe.ref)))
~~~~~~

Because we use the Camel Consumer trait, a lot of components are started and
we have to wait for these components before we can proceed with our test. To be
able detect that Camel's startup has finished, we need to use the CamelExtension.

~~~~~~
val camelExtention = CamelExtension(system)
val activated =
  camelExtention.activationFutureFor(consumer)(timeout = 10 seconds, executor = system.dispatcher)
Await.result(activated, 5 seconds)


val msg = new Order("me", "Akka in Action", 10)
val xml =
  <order>
    <customerId>{ msg.customerId }</customerId>
    <productId>{ msg.productId }</productId>
    <number>{ msg.number }</number>
  </order>
val msgFile = new File(dir, "msg1.xml")
FileUtils.write(msgFile, xml.toString())
probe.expectMsg(msg)
system.stop(consumer)
~~~~~~

## CHANGING THE TRANSPORT LAYER OF OUR CONSUMER ##

~~~~~~
val probe = TestProbe()
val camelUri = "mina:tcp://localhost:8888?textline=true&sync=false"
val consumer = system.actorOf(Props(new OrderConsumerXml(camelUri, probe.ref)))

val activated =
  CamelExtension(system).activationFutureFor(consumer)(timeout = 10 seconds, executor = system.dispatcher)
  
Await.result(activated, 5 seconds)
val msg = new Order("me", "Akka in Action", 10)
val xml = <order>
            <customerId>{ msg.customerId }</customerId>
            <productId>{ msg.productId }</productId>
            <number>{ msg.number }</number>
          </order>
val xmlStr = xml.toString().replace("n", "")

val sock = new Socket("localhost", 8888)
val ouputWriter = new PrintWriter(sock.getOutputStream, true)
ouputWriter.println(xmlStr)
ouputWriter.flush()
probe.expectMsg(msg)
ouputWriter.close()
system.stop(consumer)
~~~~~~

* `textline=true`, This indicates that we are expecting plain text over this connection and that each message
is ended with a newline
* `sync=false`, This indicates that we don't create a response


~~~~~~
def receive = {
  case msg: CamelMessage => {
    try {
      val content = msg.bodyAs[String]
      val xml = XML.loadString(content)
      val order = xml \ "order"
      val customer = (order \ "customerId").text
      val productId = (order \ "productId").text
      val number = (order \ "number").text.toInt
      next ! new Order(customer, productId, number)
      sender ! "<confirm>OK</confirm>"
    } catch {
      // 如果是同步的通信, 发生错误, actor重启, 会失去发送者的信息
      case ex: Exception =>
        sender ! "<confirm>%s</confirm>".format(ex.getMessage)
    }
  }
}
~~~~~~

## USING THE CAMEL CONTEXT ##

For example when we want to use the ActiveMQ component. To be able to use
this we need to add the component to the Camel context and define the MQ broker.
This requires the camel context.

~~~~~~
// Component name should be used in the Uri
val camelContext = CamelExtension(system).context
camelContext.addComponent("activemq",
  ActiveMQComponent.activeMQComponent(
    "vm:(broker:(tcp://localhost:8899)?persistent=false)"))

val camelUri = "activemq:queue:xmlTest"
val consumer = system.actorOf(
    Props(new OrderConsumerXml(camelUri, probe.ref)))
val activated = CamelExtension(system).activationFutureFor(
    consumer)(timeout = 10 seconds, executor = system.dispatcher)
...
sendMQMessage(xml.toString())
probe.expectMsg(msg)
system.stop(consumer)

// Because a broker is started, we also need to stop them when we are ready. 
// This can be done using the BrokerRegistry of ActiveMQ
val brokers = BrokerRegistry.getInstance().getBrokers
brokers.foreach { case (name, broker) => broker.stop() }
~~~~~~

## Implement a producer endpoint ##

~~~~~~
import akka.camel.Producer

class SimpleProducer(uri: Strint) extends Producer {
  def endpointUri = uri
}

implicit val ExecutionContext = system.dispatcher
val probe = TestProbe()
val camelUri =
  "mina:tcp://localhost:8888?textline=true&sync=false"
val consumer = system.actorOf(
  Props(new OrderConsumerXml(camelUri, probe.ref)))
  
val producer = system.actorOf(
  Props(new SimpleProducer(camelUri)))
val activatedCons = CamelExtension(system).activationFutureFor(
  consumer)(timeout = 10 seconds, executor = system.dispatcher)
val activatedProd = CamelExtension(system).activationFutureFor(
  producer)(timeout = 10 seconds, executor = system.dispatcher)
  
val camel = Future.sequence(List(activatedCons, activatedProd))
Await.result(camel, 5 seconds)
~~~~~~

Here we can do the translation of our message to the expected XML

~~~~~~
class OrderProducerXml(uri: String) extends Producer {
  def endpointUri = uri
  override def oneway: Boolean = false
  override protected def transformOutgoingMessage(message: Any): Any = {
    message match {
      case msg: Order => {
        val xml = <order>
          <customerId>{ msg.customerId }</customerId>
          <productId>{ msg.productId }</productId>
          <number>{ msg.number }</number>
        </order>
        
        xml.toString().replace("n", "")
      }
      case other => message
    }
  }
  
  // 反向序列化
  override def transformResponse(message: Any): Any = {
    message match {
      case msg: CamelMessage => {
        try {
          val content = msg.bodyAs[String]
          val xml = XML.loadString(content)
          (xml \ "confirm").text
        } catch {
          case ex: Exception =>
            "TransformException: %s".format(ex.getMessage)
        }
      }
      case other => message
    }
  }
}
~~~~~~

There is a method called routeResponse.
This method is responsible for sending the received
response to the original sender. 

# Example of implementing a REST interface #

REST is a standard protocol to expose intuitive interfaces
to systems. We are still creating an endpoint for our system.

Spray is an open-source toolkit for REST/HTTP and low-level network IO on
top of Scala and Akka.

We start by defining the
messages for both interfaces. The Order system will support two functions. The
first function is to add a new order and the second function is to get the status of an
order. The REST interface we are going to implement supports a POST and a GET.

~~~~~~
class ProcssOrders extends Actor {
  val orderList = new mutable.HashMap[Long, TrackingOrder]
  val lastOrderId = 0L

  def receive = {
    case order:Order => {
      lastOrderId += 1
      val newOrder = new TrackingOrder(lastOrderId, "received", order)
      orderList += lastOrdered -> newOrder
      sender ! newOrder
    }
    case order:OrderId => {
      orderList.get(order.id) match {
        case Some(intOrder) =>
          sender ! intOrder.copy(status="process")
        case None => sender ! NoSuchOrder(order.id)
      }
    }
    case "reset" => {
      lastOrderId = 0
      orderList.clear()
    }
  }
}
~~~~~~

##  Implementing a Rest endpoint with Spray ##

Spray also has it own test kit and is able to test your code without
building a complete application.

When you need REST/HTTP support, Spray is a great way to connect your Akka
applications to other Systems.

~~~~~~
import spray.routing.HttpService

trait OrderService extends HttpService {
  val myRoute = path("orderTest") {
    get {
      parameters('id.as[Long]).as(OrderId) { orderId =>
        complete {
          val askFuture = orderSystem ? orderId
          askFuture.map {
            case result:TrackingOrder => {
              <statusResponse>
                <id>{result.id}</id>
                <status>{result.status}</status>
              </statusResponse>
            }
            case result:NoSuchOrder => {
              <statusResponse>
                <id>{result.id}</id>
                <status>ID is unknown</status>
              </statusResponse>
            }
          }
        }
      }
    } ~
    post {
      //add order
      entity(as[String]) { body =>
        val order = XMLConverter.createOrder(body.toString)
        complete {
          val askFuture = orderSystem ? order
          askFuture.map {
            case result: TrackingOrder => {
              <confirm>
                <id>{ result.id }</id>
                <status>{ result.status }</status>
              </confirm>.toString()
            }
            case result: Any => {
              <confirm>
                <status>
                  Response is unknown{ result.toString() }
                </status>
              </confirm>.toString()
            }
          }
        }
      }
    }
  }
}
class OrderServiceActor (val orderSystem:ActorRef) extends Actor with OrderService {
  // actorRefFactory used by Spray framework
  def actorRefFactory = context
  
  // use Spray Route
  def receive = runRoute(myRoute)
}
~~~~~~


~~~~~~
class OrderHttpServer(host: String, portNr: Int, orderSystem: ActorRef)
    extends SprayCanHttpServerApp {
  //create and start our service actor
  val service = system.actorOf(Props(
    new OrderServiceActor(orderSystem)), "my-service")
  //create a new HttpServer using our handler tell it where to bind to
  val httpServer = newHttpServer(service)
  httpServer ! Bind(interface = host, port = portNr)
  
  def stop() {
    system.stop(httpServer)
    system.shutdown()
  }
}
~~~~~~

测试代码
~~~~~~
val orderSystem = system.actorOf(Props[OrderSystem])
val orderHttp = new OrderHttpServer("localhost", 8181, orderSystem)

orderSystem ! "reset"
val url = "http://localhost:8181/orderTest"
val msg = new Order("me", "Akka in Action", 10)
val xml =
  <order>
    <customerId>{ msg.customerId }</customerId>
    <productId>{ msg.productId }</productId>
    <number>{ msg.number }</number>
  </order>

val urlConnection = new URL(url)
val conn = urlConnection.openConnection()
conn.setDoOutput(true)
conn.setRequestProperty("Content-type", "text/xml; charset=UTF-8")
val writer = new OutputStreamWriter(conn.getOutputStream)
writer.write(xml.toString())
writer.flush()

//check result
val reader = new BufferedReader(new InputStreamReader((conn.getInputStream)))
val response = new StringBuffer()
var line = reader.readLine()
while (line != null) {
  response.append(line)
  line = reader.readLine()
}
writer.close()
reader.close()

conn.getHeaderField(null) must be("HTTP/1.1 200 OK")
val responseXml = XML.loadString(response.toString)
val confirm = responseXml \ "confirm"

(confirm \ "id").text must be("1")
(confirm \ "status").text must be("received")

val url2 = "http://localhost:8181/orderTest?id=1"


val urlConnection2 = new URL(url2)
val conn2 = urlConnection2.openConnection()
//Get response
val reader2 = new BufferedReader(new InputStreamReader((conn2.getInputStream)))
val response2 = new StringBuffer()
line = reader2.readLine()
while (line != null) {
  response2.append(line)
  line = reader2.readLine()
}
reader2.close()

// check response
conn2.getHeaderField(null) must be("HTTP/1.1 200 OK")
val responseXml2 = XML.loadString(response2.toString)
val status = responseXml2 \ "statusResponse"

(status \ "id").text must be("1")
(status \ "status").text must be("processing")
~~~~~~
