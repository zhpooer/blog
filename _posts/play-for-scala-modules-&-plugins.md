title: Play for Scala-Modules & Plugins
date: 2014-08-18 20:56:34
tags:
- scala
- play
---

# Modules #

Currently available modules for Play 2 provide anything from alternate template
engines to NoSQL database layers.
This section will explain how to use a common module and, later on, how
to build a module yourself.

## Using Modules ##

Play modules are, like any other library, a collection of files in a JAR. This means that
you add a module to your project the same way you add any other library: you add it to
appDependencies in `project/Build.scala` .

www.playframework.com/documentation/2.1.x/Modules

Let’s get started: make a copy of the sample project in chapter 2, and add the
dependency and resolver.

~~~~~~
import sbt._
import Keys._
import PlayProject._

object ApplicationBuild extends Build {
  val appName = "product-details"
  val appVersion = "1.0-SNAPSHOT"
  val appDependencies = Seq(
    "net.sf.barcode4j" % "barcode4j" % "2.0",
    "securesocial" %% "securesocial" % "2.1.0"
  )

  val main = PlayProject(appName, appVersion,
    appDependencies, mainLang = SCALA ).settings(
      resolvers +=
        Resolver.url(
          "SecureSocial Repository", url("http://repo.scala-sbt.org/scalasbt/sbt-plugin-releases/")
        )(Resolver.ivyStylePatterns)
    )
}
~~~~~~

According to the documentation, SecureSocial provides a replacement
for Action called SecuredAction. This method acts the same way as Action,
except that it first checks whether the user is
logged in and redirects to a login page if necessary.

~~~~~~
def list = SecuredAction { implicit request =>
  val products = Product.findAll
  Ok(views.html.products.list(products))
}

class SimpleUserService(val app: Application) extends UserService
    with Plugin {
    
  var users: Map[UserId, SocialUser] = Map()
  // stores login tokens
  var tokens: Map[String, Token] = Map()

  def find(id: UserId): Option[SocialUser] = {
    users.get(id)
  }

  def findByEmailAndProvider(email: String, providerId: String) = {
    users.values.find { user =>
      user.id.providerId == providerId &&
      user.email == Some(email)
    }
  }

  def save(user: Identity): Identity = {
    val socialUser: SocialUser = SocialUser(user)
    users = users + (user.id -> socialUser)
    socialUser
  }
  // savetoken
  def save(token: Token) {
    tokens = tokens + (token.uuid -> token)
  }

  def findToken(token: String) = {
   // Looks up a token
    tokens.get(token)
  }

  def deleteToken(uuid: String) {
    tokens = tokens - uuid
  }

  def deleteExpiredTokens() {
    tokens = tokens.filter{ !_._2.isExpired }
  }
}

~~~~~~
`conf/securesocial.conf`, If you prefer to keep it in
`conf/application.conf`, that’s fine too.
`conf/application.conf` should contains `include "securesocial.conf"`.
~~~~~~
userpass {
  withUserNameSupport=false
  sendWelcomeEmail=false
  enableGravatarSupport=false
  tokenDuration=60
  tokenDeleteInterval=5
  minimumPasswordLength=8
  enableTokenJob=true
  hasher=bcrypt
}
securesocial {
  onLoginGoTo=/
  onLogoutGoTo=/login
  ssl=false
  sessionTimeOut=60
  assetsController=controllers.ReverseMyCustomAssetsController
}
~~~~~~

~~~~~~
GET /login securesocial.controllers.LoginPage.login
GET /logout securesocial.controllers.LoginPage.logout
~~~~~~

## Creating modules ##

Creating a Play module is as easy as making a Play application. In fact, that’s how you
start with a new module—you create a new Play application as the starting point.

~~~~~~
play new ean
rm app/public/*
rm app/views/*
rm conf/application.conf
~~~~~~

`app/controller/Barcodes.scala`
~~~~~~
package com.github.playforscala.barcodes
import play.api.mvc.{Action, Controller}
import org.krysalis.barcode4j.output.bitmap.BitmapCanvasProvider
import org.krysalis.barcode4j.impl.upcean.EAN13Bean
import util.{Failure, Success, Try}

object Barcodes extends Controller {
  val ImageResolution = 144
  
  def barcode(ean: Long) = Action {
    val MimeType = "image/png"
    Try(ean13BarCode(ean, MimeType)) match {
      case Success(imageData) => Ok(imageData).as(MimeType)
      case Failure(e) =>
        BadRequest("Couldn’t generate bar code. Error: " +e.getMessage)
    }
  }

  def ean13BarCode(ean: Long, mimeType: String): Array[Byte] = {
    import java.io.ByteArrayOutputStream
    import java.awt.image.BufferedImage
    val output = new ByteArrayOutputStream
    val canvas =
      new BitmapCanvasProvider(output, mimeType, ImageResolution,
        BufferedImage.TYPE_BYTE_BINARY, false, 0)

    val barCode = new EAN13Bean
    barCode.generateBarcode(canvas, String valueOf ean)
    canvas.finish()
    output.toByteArray
  }
}
~~~~~~

We’ll explain that in the “Testing your module” section. The
route will therefore look like this:

~~~~~~
GET /:ean com.github.playforscala.barcodes.Barcodes.barcode(ean: Long)
~~~~~~

### Publish ###

~~~~~~

val main = play.Project(appName, appVersion, appDependencies).settings(
  publishTo := Some("My Maven repository" at "http://maven.example.com/releases"),
  credentials += Credentials(Path.userHome / ".repo-credentials")
)
~~~~~~

### TESTING YOUR MODULE ###

`play new module-test`

`project/Build.scala`:
~~~~~~
...
val appDependencies = Seq(
  "playforscala" %% "ean-module" % "1.0-SNAPSHOT"
)
...
~~~~~~

`app/views/index.scala.html`
~~~~~~
@(message: String)
@main("Welcome to Play 2.0") {
  @tags.barcode(1234567890128l)
}
~~~~~~

# Plugins #

Play provides a `play.api.Plugin trait`, specifically
for modules to initialize themselves.

Note that Plugin is only really useful for modules, because a `Global` object in
a Play application can do anything a `Plugin` can do.

The Plugin trait has three methods: onStart, onStop, and enabled. The first two are
called on application startup and shutdown, respectively, but only if the plugin is
enabled.

You can “enable” your plugin in your module’s play.plugins file and
provide the user with a more convenient way to really enable the plugin,
in application.conf, for instance.

Let’s say we want to cache our generated bar
codes, and for some reason we don’t want to use Play’s built-in cache.
* Concurrent calls should be handled concurrently
* Multiple calls for the same bar code should cause no more than one cache miss

In order to satisfy those requirements, we’ll use an actor.
It would only be able to render one bar code at a time if we did that.
The easiest solution is to have the future’s onComplete send the
rendered image to the client.

`app/com/github/playforscala/barcodes/BarcodeCache.scala`

~~~~~~
package com.github.playforscala.barcodes

import akka.actor.Actor
import concurrent._
import org.krysalis.barcode4j.output.bitmap.BitmapCanvasProvider
import org.krysalis.barcode4j.impl.upcean.EAN13Bean
import scala.util.Try
import play.api.libs.concurrent.Execution.Implicits._

case class RenderImage(ean: Long)
case class RenderResult(image: Try[Array[Byte]])

class BarcodeCache extends Actor {
  var imageCache = Map[Long, Future[Array[Byte]]]()
  def receive = {
    case RenderImage(ean) => {
      val futureImage = imageCache.get(ean) match {
        case Some(futureImage) => futureImage
        case None =>
          val futureImage = future { ean13BarCode(ean, "image/png") }
          imageCache += (ean -> futureImage)
          futureImage
      }
      val client = sender
      futureImage.onComplete {
        client ! RenderResult(_)
      }
    }
  }

  def ean13BarCode(ean: Long, mimeType: String): Array[Byte] = {
    import java.io.ByteArrayOutputStream
    import java.awt.imageBufferedImage
    val output = new ByteArrayOutputStream
    val canvas = new BitmapCanvasProvider(output, mimeType,
      Barcodes.imageResolution, BufferedImage.TYPE_BYTE_BINARY,
      false, 0)
    val barCode = new EAN13Bean
    barCode.generateBarcode(canvas, String valueOf ean)
    canvas.finish()

    output.toByteArray
  }
}
~~~~~~

`app/com/github/playforscala/barcodes/Barcodes.scala`
~~~~~~
package com.github.playforscala.barcodes

import akka.actor.ActorRef
import akka.pattern.ask
import util.Try
import scala.concurrent.Future
import play.api.libs.concurrent.Execution.Implicits._
import scala.concurrent.duration._
import akka.util.Timeout

object Barcodes {
  var barcodeCache: ActorRef = _
  val mimeType = "image/png"
  val imageResolution = 144

  def renderImage(ean: Long): Future[Try[Array[Byte]]] = {
    implicit val timeout = Timeout(20.seconds)
    barcodeCache ? RenderImage(ean) map {
      case RenderResult(result) => result
    }
  }
}

~~~~~~
`app/com/github/playforscala/barcodes/`

~~~~~~
package com.github.playforscala.barcodes
import play.api.mvc.{Action, Controller}
import util.{Failure, Success}
import play.api.libs.concurrent.Execution.Implicits._

object BarcodesController extends Controller {
  def barcode(ean: Long) = Action {
    Async {
      Barcodes.renderImage(ean) map {
        case Success(image) => Ok(image).as(Barcodes.mimeType)
        case Failure(e) =>
          BadRequest("Couldn’t generate bar code. Error: " + e.getMessage)
      }
    }
  }
}
~~~~~~

`.../playforscala/barcodes/BarcodesPlugin.scala`
~~~~~~
package com.github.playforscala.barcodes

import play.api.{Application, Logger, Plugin}
import play.api.libs.concurrent.Akka
import play.api.Play.current
import akka.actor.Props

class BarcodesPlugin(val app: Application) extends Plugin {
  override def onStart(){
    Logger.info("initializing cace")
    Barcodes.barcodeCache = Akka.system.actorOf(Props[BarcodeCache])
  }
  
  override def onStop() {
    Logger.info("stopping application")
  }
  // if this method returns false, none of the others are ever called
  override def enable = true;
}

~~~~~~
There’s one more thing to do to make the plugin work.
It must be configured in a file called `conf/play.plugins`

The priority determines the order in which the plugins are initialized,
with lower numbers being first.
~~~~~~
1000:com.github.playforscala.barcodes.BarcodesPlugin
~~~~~~

If your application is going to get hit with a lot of requests for different
bar codes simultaneously, you’re going to fill up the default thread pool—which might
slow things down in the rest of the application. You might want to use a separate
thread pool for your bar code Future objects. If your application runs on multiple
servers for performance reasons, you might want to use Akka’s distributed features to
run one instance of the BarcodeCache actor that all application instances will talk to.

# Deploying to production #

As a better alternative, you can use play start. This will start Play in production
mode. In this mode, a new JVM is forked for your application, and
it’s running separately from the play command.

The play process will terminate but leave your application running.
Your application’s process ID is written to a file RUNNING_PID.

You can stop this application with play stop. This will send the SIGTERM signal to
your application’s process. 

Play provides the `stage` and `dist` tasks. When running play
stage, Play compiles your application to a JAR file, and—together with all the
dependency JARs—puts it in the `target/staged` directory.
It also creates a start script in `target/start`.

After running `play dist`, you get a directory dist that contains a zip file
with your application. You might need to make the start script
executable first with `chmod +x` start.


## Working with multiple configurations ##

Don’t use the same credentials for your production database

`conf/application.conf`
~~~~~~
mail.override.enabled = true
mail.override.address = "info@example.org"

include "development.conf"
~~~~~~

The first two lines of this configuration override email recipient addresses, making the
application send all notifications to one address, `info@example.org`

The last line includes settings from another configuration file in the same
directory called development.conf. This allows each developer to create their
own `conf/development.conf` and override the default test configuration.

Be sure to add this file to .gitignore or your source control system’s equivalent.

A nice thing about the configuration library is that the configuration doesn’t break
if the `development.conf` file doesn’t exist; the library just silently ignores it.

For production, then, we can use a separate /etc/paperclips/production.conf
configuration file:
~~~~~~
include classpath("application.conf")
email.override.enabled=false
~~~~~~
To use the production configuration instead of the default configuration,
specify the file as a system property when starting the application:
~~~~~~
play "start -Dconfig.file=/etc/paperclips/production.conf"
~~~~~~

## Creating native packages for a package manager ##

The sbt plugin sbt-native-packager helps you create these deb and rpm
packages as well as Homebrew packages that can be used on Mac OS X,
and MSI packages for Windows.

The play2-native-packager plugin builds deb packages for Debian or Ubuntu,
and the play2-ubuntu-package plugin builds lightweight deb packages
designed specifically for recent versions of Ubuntu.

## Setting up a front-end proxy ##

It also gives you the ability to do upgrades without downtime. If you have a front-end
proxy doing load balancing between two application instances, you can take one
instance down, upgrade it, and bring it back up, all without downtime.

HAProxy is a powerful and reliable proxy that has a plethora of advanced options,
but is still easy to get started with.

Suppose we want to set up HAProxy to listen on port 80, and redirect traffic to two
instances of our Play application. We’ll also use WebSockets in this application,
so we must make sure that these connections are properly proxied as well.

~~~~~~
global
  daemon
  maxconn 256
  
defaults
  mode http
  timeout connect 5s
  timeout client 50s
  timeout server 50s
  option forwardfor
  option http-server-close
  
frontend http-in
  bind *:80
  default_backend playapp
  
backend playapp
  server s1 127.0.0.1:9000 maxconn 32 check
  server s2 127.0.0.1:9001 maxconn 32 check
~~~~~~

If you set the pidfile.path to `/dev/null`, no PID file will be created.

## Using SSL ##

Play can automatically generate a key store for you with a self-signed certificate,
which is useful in development mode. All you need to start experimenting with SSL is
to set the `https.port` system property:
~~~~~~
play -Dhttps.port=9001 run
~~~~~~

The generated key store is saved in `conf/generated.keystore`, and Play will reuse
it if you restart your application so you don’t get the certificate warning again and again.


Once you have a key store file with your key and certificates, you need to point Play
to it. Set `https.keyStore` to point to your key store and `https.keyStorePassword` to
your password:
~~~~~~
play -Dhttps.port=9001 -Dhttps.keyStore=mykeystore.jks
  -Dhttp.keyStorePassword=mypassword run
~~~~~~

Even though Play supports SSL, the recommended way to use SSL with Play in produc-
tion is to let the front end—like HAProxy or Apache—handle it.

## Deploying to an application server ##

e Play doesn’t use the Servlet API, which makes it
impossible to run on an application server that expects web applications to use it.
Luckily, there’s a plugin for Play 2, the `play2-war-plugin`, that can package your
application as a WAR. It provides a layer between the Servlet API and your Play application.

Some of the more advanced features of Play, like WebSockets, don’t work with all
Servlet API versions.


