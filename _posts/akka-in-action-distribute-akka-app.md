title: Akka in Action-Distribute Akka App
date: 2014-08-04 09:58:24
tags:
- akka
- scala
---

We'll scale the goticks.com app out to two nodes; a frontend
and a backend server.
The REST Interface will run
on a frontend node. The BoxOffice and all TicketSellers will run on a backend
node. Both nodes have a static reference to each other's network addresses. 

![net work topologies](/img/akka_network_topologies.png)

Messages between the nodes are sent over the transport protocol and
need to be encoded and decoded into network specific protocol data units. 

~~~~~~
"com.typesafe.akka" %% "akka-multi-node-testkit" % akkaV % "test",
"com.typesafe.akka" %% "akka-testkit" % akkaV % "test",
~~~~~~

# Remote REPL action #

Akka provides two ways to get a reference to an actor on a remote node. One is to
look up the actor by its path, the other is to create the actor, get its reference and
deploy it remotely.

`sbt console`

~~~~~~
// Select the Remote ActorRef Provider to bootstrap remoting
// the configuration section for remoting
// 建立一个远程的 Actor
val conf = """
  akka {
    actor {
      provider = "akka.remote.RemoteActorRefProvider"
    }
    remote {
      enabled-transports = ["akka.remote.netty.tcp"]
      netty.tcp {
        hostname = "0.0.0.0"
        port = 2551
      }
    }
  }
"""

import com.typesafe.config._
import akka.actor._
val config = ConfigFactory.parseString(conf)
// Create the ActorSystem with the parsed Config object.
val backend = ActorSystem("backend", config)

class Simple extends Actor {
  def receive = {
    case m => println(s"received $m!")
  }
}
// Create the simple actor in the backend actor system with the name "simple"
backend.actorOf(Props[Simple], "simple")
~~~~~~

~~~~~~
// 建立一个前端的 Actor
val conf = """
  akka {
    actor {
      provider = "akka.remote.RemoteActorRefProvider"
    }
    remote {
      enabled-transports = ["akka.remote.netty.tcp"]
      netty.tcp {
        hostname = "0.0.0.0"
        port = 2552
      }
    }
  }
"""
import com.typesafe.config._
import akka.actor._
val config = ConfigFactory.parseString(conf)
val frontend= ActorSystem("frontend", config)

// Select the actor with an ActorSelection
// the guardian actor is aways called 'user'
val path = "akka.tcp://backend@0.0.0.0:2551/user/simple"
// Think of the actorSelection method as a query in the actor hierarchy
val simple = frontend.actorSelection(path)

~~~~~~

`scala> simple ! "Hello Remote World!"`

# Remote Lookup #

Instead of directly creating a BoxOffice actor in the RestInterface actor we will
look it up on the backend node.

## version 1 ##
Creator of BoxOffice

~~~~~~
trait BoxOfficeCreator { this: Actor =>
  def createBoxOffice:ActorRef = {
    context.actorOf(Props[BoxOffice], "boxOffice")
  }
}
class RestInterface extends HttpServiceActor with RestApi {
  def receive = runRoute(routes)
}

trait RestApi extends HttpService
    with ActorLogging
    with BoxOfficeCreator { actor: Actor =>
  // BoxOffice is created using the createBoxOffice method
  val boxOffice = createBoxOffice
  // rest of the code of the RestApi
}
~~~~~~

## version 2 ##

A RemoteBoxOfficeCreator trait will override the default behavior of which
the details will follow shortly.

A SingleNodeMain, FrontendMain and a
BackendMain are created to start the app in single node mode or to start a
frontend and backend separately.

load from the files singlenode.conf, frontend.conf and backend.conf respectively
~~~~~~
// 同一台机子上
//Snippet from SingleNodeMain
val system = ActorSystem("singlenode", config)
val restInterface = system.actorOf(Props[RestInterface], "restInterface")

// 前端
//Snippet from FrontendMain
val system = ActorSystem("frontend", config)
class FrontendRestInterface extends RestInterface with RemoteBoxOfficeCreator
  
val restInterface = system.actorOf(Props[FrontendRestInterface], "restInterface")

// 后端
//Snippet from BackendMain
val system = ActorSystem("backend", config)
val config = ConfigFactory.load("backend")
val system = ActorSystem("backend", config)

system.actorOf(Props[BoxOffice], "boxOffice")
~~~~~~

The RemoteBoxOfficeCreator loads these extra configuration properties:
~~~~~~
backend {
  host = "0.0.0.0"
  port = 2552
  protocol = "akka.tcp"
  system = "backend"
  actor = "user/boxOffice"
}
~~~~~~

~~~~~~
object RemoteBoxOfficeCreator {
  val config = ConfigFactory.load("frontend").getConfig("backend")
  val host = config.getString("host")
  val port = config.getInt("port")
  val protocol = config.getString("protocol")
  val systemName = config.getString("system")
  val actorName = config.getString("actor")
}

trait RemoteBoxOfficeCreator extends BoxOfficeCreator { this:Actor =>
  import RemoteBoxOfficeCreator._
  def createPath:String = {
    s"$protocol://$systemName@$host:$port/$actorName"
  }
  override def createBoxOffice = {
  val path = createPathcontext.actorOf(Props(classOf[RemoteLookup],path),
      "lookupBoxOffice")
  }
}
~~~~~~
The RemoteBoxOfficeCreator creates a separate RemoteLookup Actor to
lookup the boxOffice.

The RemoteLookup actor is a state machine that can only be in one of two
states we have defined: identify or active. It uses the become method to switch its
receive method to identify or active. The RemoteLookup tries to get a valid
ActorRef to the BoxOffice when it does not have one yet in the identify state or it
forwards all messages sent to a valid ActorRef to the BoxOffice in the active state.

If the RemoteLookup detects that the BoxOffice has been terminated it tries to get
a valid ActorRef again when it receives no messages for a while. We'll use Remote
Deathwatch for this. Sounds like something new but from the perspective of API
usage it's exactly the same thing as normal actor monitoring/watching.

~~~~~~
import scala.concurrent.duration._

class RemoteLookup(path:String) extends Actor with ActorLogging {
  // Send a ReceiveTimeout message if no message has
  // been received for 3 seconds
  context.setReceiveTimeout(3 seconds)
  sendIdentifyRequest()
  def sendIdentifyRequest(): Unit = {
    val selection = context.actorSelection(path)
    selection ! Identify(path)
  }

  // The actor is initially in identify receive state
  def receive = identify

  def identify: Receive = {
    case ActorIdentity(`path`, Some(actor)) =>
      // No longer send a ReceiveTimeout if the actor
      // gets not messges since it is now active.
      context.setReceiveTimeout(Duration.Undefined)
      log.info("switching to active state")
      // Change to active receive state
      context.become(active(actor))
      context.watch(actor)
    case ActorIdentity(`path`, None) =>
      log.error(s"Remote actor with path $path is not available.")
    case ReceiveTimeout =>
      sendIdentifyRequest()
    case msg:Any =>
      log.error(s"Ignoring message $msg, not ready yet.")
  }

  def active(actor: ActorRef): Receive = {
    case Terminated(actorRef) =>
      log.info("Actor $actorRef terminated.")
      context.become(identify)
      log.info("switching to identify state")
      context.setReceiveTimeout(3 seconds)
      sendIdentifyRequest()
    case msg:Any => actor forward msg
  }
}
~~~~~~

## Remote Deployment ##

~~~~~~
// creates and deploys the boxOffice remotely to the backend as
// well. The Props configuration object specifies a remote scope for deployment.
val uri = "akka.tcp://backend@0.0.0.0:2552"
val backendAddress = AddressFromURIString(uri)
val props = Props[BoxOffice].withDeploy( Deploy(scope = RemoteScope(backendAddress)) )
context.actorOf(props, "boxOffice")
~~~~~~

When we use configured remote deployment all
we have to do is tell the frontend actor system that when an actor is created with
the path `/restInterface/boxOffice` it should not create it locally but
remotely.

~~~~~~
actor {
  provider = "akka.remote.RemoteActorRefProvider"
  deployment {
    /restInterface/boxOffice {
      remote = "akka.tcp://backend@0.0.0.0:2552"
    }
  }
}
~~~~~~

TODO

# Configuration #

Akka uses the Typesafe Config Library, which sports a pretty state-of-the-art set of
capabilities.

~~~~~~
val config = ConfigFactory.load()
~~~~~~

Since the library supports a number of different configuration formats,
it looks for different files, in the following order:
* application.properties
* application.json
* application.conf

~~~~~~
// application.conf
hostname="localhost"
// If there is an env var, override, otherwise, leave it with the value we just assigned
hostname=${?HOST_NAME}
MyAppl {
  version = 10
  description = "My application"
  database {
    connect="jdbc:mysql://${hostname}/mydata"
    user="me"
  }
}

~~~~~~
usage
~~~~~~
val applicationVersion = config.getInt("MyAppl.version")
val databaseConnectSting = config.getString("MyAppl.database.connect")
~~~~~~

## Using Defaults ##
The configuration library contains a fall-back mechanism;
the defaults are placed into a configuration object.

It to provide the defaults we need, we have to know how to configure them.
They are configured in the file reference.conf and placed in the root of the jar file; the idea is
that every library contains its own defaults.


~~~~~~
// This way it doesn't try to load application.{conf,json,properties},
// but myapp.{conf,json,properties}
val config = ConfigFactory.load("myapp")
~~~~~~

Another option is to use system properties.
Sometimes, this is the easiest thing
because you can just create a bash script and set a property and the app will pick it
up and start using it
* config.resource specifies a resource name - not a base-name, i.e. application.conf not
application
* config.file specifies a file system path, again it should include the extension
* config.url specifies a URL

## Akka Configuration ##
default configuration
~~~~~~
val system = ActorSystem("mySystem")

// supply the configuration while creating an ActorSystem
val configuration = ConfigFactory.load("mysystem")
val systemA = ActorSystem("mysystem",configuration)

//  it can be found in the settings of the ActorSystem.
val mySystem = ActorSystem("myAppl")
// Once the ActorSystem is constructed, we can get the config
// just by referencing it using this path
val config = mySystem.settings.config
val applicationDescription = config.getString("myAppl.name")
~~~~~~

## Mutiple Systems ##

~~~~~~
// baseConfig.conf
MyAppl {
  version = 10
  description = "My application"
}

// subAppl.conf
include "baseConfig"
MyAppl {
  description = "Sub Application"
}

~~~~~~


~~~~~~
MyAppl {
  version = 10
  description = "My application"
}

subApplA {
  MyAppl {
    description = "Sub application"
  }
}
~~~~~~

~~~~~~
val configuration = ConfigFactory.load("combined")
val subApplACfg = configuration.getConfig("subApplA")
// subApplA 覆盖 原始配置
val config = subApplACfg.withFallback(configuration)
~~~~~~

# Logging #

The Akka toolkit has implemented a logging adapter to be able to support all
kinds of logging frameworks and also minimize the dependencies on other
libraries. 

~~~~~~
class MyActor extends Actor {
  val log = Logging(context.system, this)
  ...
}
~~~~~~

~~~~~~
akka {
  # Event handlers to register at boot time
  # (Logging$DefaultLogger logs to STDOUT)
  
  # This eventHandler doesn't use a log framework,
  # but logs all the received messages to standard out.
  event-handlers = ["akka.event.Logging$DefaultLogger"]
  # Options: ERROR, WARNING, INFO, DEBUG
  loglevel = "DEBUG"
}

~~~~~~

When you want to create your own eventhandler you
have to create an Actor which handles several messages. An example of such
handler is
~~~~~~
import akka.event.Logging.InitializeLogger
import akka.event.Logging.LoggerInitialized
import akka.event.Logging.Error
import akka.event.Logging.Warning
import akka.event.Logging.Info
import akka.event.Logging.Debug
class MyEventListener extends Actor{
  def receive = {
    case InitializeLogger(_) =>
      sender ! LoggerInitialized
    case Error(cause, logSource, logClass, message) =>
      println( "ERROR " + message)
    case Warning(logSource, logClass, message) =>
      println( "WARN " + message)
    case Info(logSource, logClass, message) =>
      println( "INFO " + message)
    case Debug(logSource, logClass, message) =>
      println( "DEBUG " + message)
  }
}
~~~~~~

The Akka toolkit has two
implementations of this logging eventHandler. The first is already mentioned and
that is the default logger to STDOUT. The second implementation is using SLF4J.

~~~~~~
# import akka-slf4j.jar.
akka {
  event-handlers = ["akka.event.slf4j.Slf4jEventHandler"]
  # Options: ERROR, WARNING, INFO, DEBUG
  loglevel = "DEBUG"
}
~~~~~~

For convenience you can also use the ActorLogging trait to mix-in the log member into actors.

~~~~~~
class MyActor extends Actor with ActorLogging {
  ...
}
~~~~~~

The adapter also supports the ability to use placeholders in the message.
Placeholders prevent you from having to check logging levels.

~~~~~~
log.debug("two parameters: {}, {}", "one","two")
~~~~~~

## Controlling Akka's logging ##
Akka provides a simple configuration layer that allows you to exert
some control over what it outputs to the log, and as we are both using the pubsub
attached to a single adapter, once we change these settings, we will see the results
in whatever our chosen appenders are.

~~~~~~
akka
{
  # logging must be set to DEBUG to use any of the options below
  loglevel = DEBUG
  
  # Log the complete configuration at INFO level when the actor
  # system is started. This is useful when you are uncertain of
  # what configuration is used.
  log-config-on-start = on
  debug {
    # logging of all user-level messages that are processed by
    # Actors that use akka.event.LoggingReceive enable function of
    # LoggingReceive, which is to log any received message at
    # DEBUG level
    receive = on
    
    # enable DEBUG logging of all AutoReceiveMessages
    # (Kill, PoisonPill and the like)
    autoreceive = on
    
    # enable DEBUG logging of actor lifecycle changes
    # (restarts, deaths etc)
    lifecycle = on
    
    # enable DEBUG logging of all LoggingFSMs for events,
    # transitions and timers
    fsm = on
    
    # enable DEBUG logging of subscription (subscribe/unsubscribe)
    # changes on the eventStream
    event-stream = on
  }
  remote {
    # If this is "on", Akka will log all outbound messages at
    # DEBUG level, if off then they are not logged
    log-sent-messages = on
    # If this is "on," Akka will log all inbound messages at
    # DEBUG level, if off then they are not logged
    log-received-messages = on
  }
}
~~~~~~


~~~~~~
// Now when you set the property akka.debug.receive to on, the messages
// received by our actor will be logged.
class MyActor extends Actor with ActorLogging {
  def receive = LoggingReceive {
    case ... => ...
  }
}
~~~~~~


# Deploying Stand-alone application #

To create a stand alone application we use the MicroKernel of Akka combined
with the akka-plugin to create a distribution.

~~~~~~
class HelloWorld extends Actor with ActorLogging {
  def receive = {
    case msg:String =>
      val hello = "Hello %s".format(msg)
      sender ! hello
      log.info("Sent response {}",hello)
  }
}

class HelloWorldCaller(timer:Duration, actor:ActorRef) extends Actor with ActorLogging {
  case class TimerTick(msg:String)
  // Using the Akka scheduler to send messages to yourself
  override def preStart() {
    super.preStart()
    context.system.scheduler.schedule(
      timer, // The duration before the schedule is triggered for the first time
      timer, // The duration between the scheduled triggers
      self,  // The message which is sent
      new TimerTick("everybody"))
  }
  
  def receive = {
    case msg: String => log.info("received {}",msg)
    case tick: TimerTick => actor ! tick.msg
  }
}


import akka.actor.{ Props, ActorSystem }
import akka.kernel.Bootable
import scala.concurrent.duration._
// Extends the Bootable trait to be able to be called when starting the application
class BootHello extends Bootable {
  val system = ActorSystem("hellokernel")
  def startup = {
    val actor = system.actorOf(Props[HelloWorld])
    val config = system.settings.config
    val timer = config.getInt("helloWorld.timer")
    system.actorOf(Props( new HelloWorldCaller( timer millis, actor)))
  }
  def shutdown = {
    system.shutdown()
  }
}
~~~~~~

~~~~~~
# reference.conf
helloWorld {
  timer=5000
}
~~~~~~

application.conf
~~~~~~
akka {
  event-handlers = ["akka.event.slf4j.Slf4jEventHandler"]
  # Options: ERROR, WARNING, INFO, DEBUG
  loglevel = "DEBUG"
}
~~~~~~

`project/plugins.sbt`
~~~~~~
resolvers += "Typesafe Repository" at "http://repo.akka.io/releases/"
addSbtPlugin("com.typesafe.akka" % "akka-sbt-plugin" % "2.0.1")
~~~~~~

`project/HelloKernelBuild.scala`
~~~~~~
import sbt._
import Keys._
import akka.sbt.AkkaKernelPlugin
import akka.sbt.AkkaKernelPlugin.{ Dist, outputDirectory, distJvmOptions }
object HelloKernelBuild extends Build {
  lazy val HelloKernel = Project(
    id = "hello-kernel-book",
    base = file("."),
    settings = defaultSettings ++ AkkaKernelPlugin.distSettings ++ Seq(
      libraryDependencies ++= Dependencies.helloKernel,
      distJvmOptions in Dist := "-Xms256M -Xmx1024M",
      outputDirectory in Dist := file("target/helloDist")
    )
  )
  
  lazy val buildSettings = Defaults.defaultSettings ++ Seq(
    organization := "com.manning",
    version := "0.1-SNAPSHOT",
    scalaVersion := "2.9.1",
    crossPaths := false,
    organizationName := "Mannings",
    organizationHomepage := Some(url("http://www.mannings.com"))
  )
  
  lazy val defaultSettings = buildSettings ++ Seq(
    resolvers += "Typesafe Repo" at "http://repo.typesafe.com/typesafe/releases/",
    // compile options
    scalacOptions ++= Seq("-encoding", "UTF-8","-deprecation","-unchecked"),
    javacOptions ++= Seq("-Xlint:unchecked","-Xlint:deprecation")
  )
}
// Dependencies
object Dependencies {
  import Dependency._
  val helloKernel = Seq(
    akkaActor, akkaKernel,
    akkaSlf4j, slf4jApi, slf4jLog4j,
    Test.junit, Test.scalatest, Test.akkaTestKit)
}
object Dependency {
  // Versions
  object V {
    val Scalatest = "1.6.1"
    val Slf4j = "1.6.4"
    val Akka = "2.0"
  }
  // Compile
  val commonsCodec = "commons-codec" % "commons-codec"% "1.4"
  val commonsIo = "commons-io" % "commons-io" % "2.0.1"
  val commonsNet = "commons-net" % "commons-net" % "3.1"
  val slf4jApi = "org.slf4j" % "slf4j-api" % V.Slf4j
  val slf4jLog4j = "org.slf4j" % "slf4j-log4j12"% V.Slf4j
  val akkaActor = "com.typesafe.akka" % "akka-actor" % V.Akka
  val akkaKernel = "com.typesafe.akka" % "akka-kernel" % V.Akka
  val akkaSlf4j = "com.typesafe.akka" % "akka-slf4j" % V.Akka
  val scalatest = "org.scalatest" %% "scalatest" % V.Scalatest
  
  object Test {
    val junit = "junit" % "junit" % "4.5" % "test"
    val scalatest = "org.scalatest" %% "scalatest" % V.Scalatest % "test"
    val akkaTestKit ="com.typesafe.akka" % "akka-testkit" % V.Akka % "test"
  }
}

~~~~~~

~~~~~~
sbt dist
~~~~~~
After this, SBT has created a distribution in the directory `/target/helloDist`. This
directory contains 4 subdirectories
* bin, This contains the start script. One for windows and one for Unix
* config, This directory contains the configuration files needed to run our application.
* deploy, This directory is where our jar file placed
* lib, This directory contains all the jar files our application depends upon.

## Akka with a web application ##

There are a number of options for deploying Akka in a webapp,
we are showing play-mini because it is very simple and lightweight.

~~~~~~
// Just by extending Application, we bring a lot of functionality in here.
object PlayMiniHello extends Application {
  lazy val system = ActorSystem("webhello")
  lazy val actor = system.actorOf(Props[HelloWorld])
  val writeForm = Form("name" -> text(1,10))
  
  def route = {
    case GET(Path("/test")) => Action {
      Ok("TEST @ %s\n".format(System.currentTimeMillis))
    }
    case GET(Path("/hello")) => Action { implicit request =>
      val name = try {
        // Bind our form to the implicit request and get the result
        writeForm.bindFromRequest.get
      } catch {
        case ex:Exception => {
          log.warning("no name specified")
          system.settings.config.getString("helloWorld.name")
        }
      }
      //  Instead of returning our response directly, we create an AsyncResult.
      AsyncResult {
        // Translate the AskTimeoutException into the string "Timeout"
        val resultFuture = actor ? name recover {
          case ex:AskTimeoutException => "Timeout"
          case ex:Exception => {
            log.error("recover from "+ex.getMessage)
            "Exception:" + ex.getMessage
          }
        }
        val promise = resultFuture.asPromise
        promise.map {
          case res:String => {
            Ok(res)
          }
          case ex:Exception => {
            log.error("Exception "+ex.getMessage)
            Ok(ex.getMessage)
          }
          case _ => {
            Ok("Unexpected message")
          }
        }
      }
    }
  }
}

//  This allows us to use the onStart and onStop methods
// which create and stop the Actor system.
object Global extends com.typesafe.play.mini.Setup(ch04.PlayMiniHello)

~~~~~~
reference.conf
~~~~~~
helloWorld {
  name=world
}
~~~~~~

application.conf
~~~~~~
helloWorld {
  name="world!!!"
}
akka {
  event-handlers = ["akka.event.slf4j.Slf4jEventHandler"]
  # Options: ERROR, WARNING, INFO, DEBUG
  loglevel = "DEBUG"
}
~~~~~~

Build.scala
~~~~~~
importimportimportsbt._
Keys._
PlayProject._
object Build extends Build {
  lazy val root = Project(id = "playminiHello",base = file("."), settings = Project.defaultSettings)
    .settings(
      resolvers += "Typesafe Repo" at "http://repo.typesafe.com/typesafe/releases/",
      resolvers += "Typesafe Snapshot Repo" at "http://repo.typesafe.com/typesafe/snapshots/",
      libraryDependencies ++= Dependencies.hello,
      mainClass in (Compile, run) := Some("play.core.server.NettyServer"))
}

object Dependencies {
  import Dependency._
  val hello = Seq(akkaActor, akkaSlf4j, playmini)
  // slf4jLog4j,
}
object Dependency {
  // Versions
  object V {
    val Slf4j ="1.6.4"
    val Akka  = "2.0"
  }
  
  // Compile
  val slf4jLog4j = "org.slf4j" % "slf4j-log4j12"% V.Slf4j
  val akkaActor  = "com.typesafe.akka" % "akka-actor" % V.Akka
  val playmini   = "com.typesafe" %% "play-mini" % "2.0-RC3"
  val akkaSlf4j  =  "com.typesafe.akka" % "akka-slf4j" % V.Akka
}

// sbt run
// At this moment the application is running and listening on port 9000
~~~~~~

