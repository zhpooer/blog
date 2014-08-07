title: Scala in Action-working with scala
date: 2014-07-17 11:35:34
tags:
- scala
---

# Building web application #


## 用户故事 ##

You can move one more story from the ready phase to the
dev phase. A pair of developers looking for new work can select a card from the ready
phase and move that card to the dev phase. Once the development work is done, the
card moves to the test phase where, in this stage, a tester, business analyst, or other
members of the team will verify the work against the user story. When the story is
approved or verified, it’s moved to the deploy phase, which means it’s ready for pro-
duction deployment. This is how a card (work) flows through the system.

* As a customer, I want to create a new user story so I can add stories to the ready phase.
* As a developer, I want to move cards (stories) from one phase to another so I can signal
progress.

## sbt ##

~~~~~~
// The following expression will create a Setting[String] setting:
> set name := "Testing SBT"
[info] Reapplying settings...
[info] Set current project to Testing SBT
> set version := "1.0"
[info] Reapplying settings...
[info] Set current project to Testing SBT
> session save
~~~~~~

Settings are the way SBT stores the build definition. A build definition defines a list
of Setting[T] where Setting[T] is a transformation affecting SBT’s key value pair.
A Setting is created assigning value to SettingKey. There are three kinds of keys
in the SBT:
* SettingKey[T] is a key with a value computed only once. Examples are
name or scalaVersion.
* TaskKey[T] is a key with a value that has to be recomputed each time.
TaskKey is used to create tasks. Examples are compile and package.
* InputTask[T] is a task key which takes command-line arguments as input.

All the available keys are defined in the sbt.Keys object, and it’s automatically
imported for you in the build.sbt file.

~~~~~~
scalacOptions ++= Seq("-unchecked", "-deprecation")
~~~~~~

~~~~~~
$ mkdir -p src/{main,test}/{scala,java,resources} lib project
~~~~~~

The third option is to use giter8 (https://github.com/n8han/giter8). It’s a com-
mand-line tool to generate files and directories from templates published in Github.
This is slowly becoming a standard way of creating projects in Scala. Once giter8 is
installed, you can choose a template to generate the project structure.

![sbt project struct](/img/sbt_struct.png)

SBT project structure is recursive. The project directory is another project inside your
project that knows how to build your project.

The following is how you define dependency in SBT :
`groupID % artifactID % version`

If you use %% after groupID, SBT will add the Scala version of the proj-
ect to the artifact ID.

SBT can read the dependencies defined in the
POM file if you use the externalPom() method in your build file.

Alternatively, you can create a project definition file configured to use a local Maven
repository:
~~~~~~
resolvers += "Local Maven Repository" at
"file://"+Path.userHome+"/.m2/repository"
~~~~~~

Here’s how
the build.sbt looks after all the changes:
~~~~~~
scalaVersion := "2.10.0"

name := "Testing SBT"

version := "1.0"

scalacOptions ++= Seq("-unchecked", "-deprecation")

libraryDependencies ++= Seq(
  "org.eclipse.jetty" % "jetty-server" % "7.0.0.RC2",
  "org.scala-tools.testing" % "specs" % "1.6.2" % "test")

~~~~~~


Another common thing you can do with SBT is create custom tasks for the project. For
custom tasks, the .scala build definition file is used because the .sbt build file doesn’t
support it. To create custom tasks follow these steps:

* Create a TaskKey .
* Provide a value for the TaskKey .
* Put the task in the .scala build file under project.

~~~~~~
import sbt._
import Keys._

object ExampleBuild extends Build {
  val hello = TaskKey [Unit]("hello", "Prints 'Hello World'")
  val helloTask: Setting[Task[Unit]] = hello := {
    println("Hello World")
  }
  val project = Project (
    "example",
    file (".")).settings(helloTask)
}

~~~~~~

~~~~~~
// console-project
scala> get(name)
res2: String = Testing SBT
scala> get(scalaVersion)
res3: String = 2.10.0
scala> runTask(hello, currentState)
Hello World
res11: (sbt.State, Unit) = (sbt.State@4fae46d5,())
~~~~~~

## Seting up ##

In `project/build.properties` sets `sbt.version=0.12.0`

`build.sbt`
~~~~~~
name := "weKanban"

organization := "scalainaction"

version := "0.1"

scalaVersion := "2.10.0"

scalacOptions ++= Seq("-unchecked", "-deprecation")

~~~~~~

Remember to separate each setting expression with an empty new line so that SBT can
parse each expression .sbt file. 

`project/plugins.sbt`. This plug-in adds tasks to the SBT build to start and stop the
web server. 
~~~~~~
libraryDependencies <+= sbtVersion {v =>
  "com.github.siasia" %% "xsbt-web-plugin" % (v+"-0.2.11.1")
}
~~~~~~
The `<+=` method allows you to compute a new list element from other keys.

build.sbt:
~~~~~~
libraryDependencies ++=
 Seq(
  "org.eclipse.jetty" %
    "jetty-servlet" % "7.3.0.v20110203" % "container",
  "org.eclipse.jetty" %
    "jetty-webapp" % "7.3.0.v20110203" % "test, container",
  "org.eclipse.jetty" %
    "jetty-server" % "7.3.0.v20110203" % "container"
)
~~~~~~
Additionally, jetty-web is added into test scope.
The scope allows SBT keys to have values in more than one context.
This is useful for plug-ins because scoping allows plug-ins to
create tasks that don’t conflict with other task names. 

To include all the tasks from the plug-in to your project,
you have to import the settings from the plug-in project into
your build.sbt file as follows:
`seq(com.github.siasia.WebPlugin.webSettings :_*)`

To start the web server, run the container:`start task`,
and it will start the Jetty server at port number 8080

## introducing scalaz module ##

~~~~~~
trait Application[IN[_], OUT[_]] {
  def apply(implicit req: Request[IN]): Response[OUT]
}

object Application {
  def application[IN[_], OUT[_]](f: Request[IN] => Response[OUT])
    = new Application[IN,OUT] {
        def apply(implicit req: Request[IN]) = f(req)
      }
}

~~~~~~

`build.sbt`
~~~~~~
libraryDependencies ++= Seq(
  "org.scalaz" %% "scalaz-core" % "6.0.3",
  "org.scalaz" %% "scalaz-http" % "6.0.3",
  "org.eclipse.jetty" % "jetty-servlet" % "7.3.0.v20110203" % "container",
  "org.eclipse.jetty" % "jetty-webapp" % "7.3.0.v20110203" % "test, container",
  "org.eclipse.jetty" % "jetty-server" % "7.3.0.v20110203" % "container"
)
~~~~~~
`web.xml`
~~~~~~
<web-app>
  <servlet>
    <servlet-name>Scalaz</servlet-name>
   <!-- This servlet will create both a request
        and response of type scala.collection.Stream -->
    <servlet-class>
      scalaz.http.servlet.StreamStreamServlet
    </servlet-class>
    <init-param>
      <param-name>application</param-name>
      <param-value>
        com.kanban.application.WeKanbanApplication
       </param-value>
      </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>Scalaz</servlet-name>
    <url-pattern>/*</url-pattern>
  </servlet-mapping>
</web-app>
~~~~~~

~~~~~~
// StreamStreamServletApplication to create your application
// class because it’s enforced by the
// Scalaz servlet you’re using to handle all the HTTP request and response. 
final class WeKanbanApplication extends StreamStreamServletApplication {

  val application = new ServletApplication[Stream, Stream] {
  def application(implicit servlet: HttpServlet,
    servletRequest: HttpServletRequest,
    request: Request[Stream]) = {
      def found(x: Iterator[Byte]) : Response[Stream] = OK << x.toStream
      HttpServlet.resource(found, NotFound.xhtml)
    }
  }
}
~~~~~~

~~~~~~
<html>
  <body>
    <h1>weKanban board will come shortly</h1>
  </body>
</html>

~~~~~~

## database ##

add dependencies
~~~~~~
"com.h2database" % "h2" % "1.2.137",
"org.squeryl" % "squeryl_2.10" % "0.9.5-6"
~~~~~~

~~~~~~
class Story(val number: String, val title: String, val phase: String)

package com.kanban.models
import org.squeryl._
//  defines the schema with a table
// called “STORIES” for your Story class
object KanbanSchema extends Schema {
  val stories = table[Story]("STORIES")
}
~~~~~~

run h2base by hand
~~~~~~
java -cp ~/.ivy2/cache/com.h2database/h2/jars/h2*.jar org.h2.tools.Server
~~~~~~

or write task in `build.scala`
~~~~~~
import sbt._
import Keys._
object H2TaskManager {
  var process: Option[Process] = None
  // creates a new config name “h2” and extends the Compile config
  // The Compile config will provide the necessary
  // classpath setting you need to run the tasks.
  lazy val H2 = config("h2") extend(Compile)
  val startH2 = TaskKey[Unit]("start", "Starts H2 database")
  // <<= method in SBT helps to create a
  //  new setting that depends on other settings.
  val startH2Task = startH2 in H2 <<= (fullClasspath in Compile) map {
    cp =>
      startDatabase {
        cp.map(_.data)
        .map(_.getAbsolutePath())
        .filter(_.contains("h2database"))
      }
  }
  
  def startDatabase(paths: Seq[String]) = {
    process match {
      case None =>
        val cp = paths.mkString(System.getProperty("path.separator"))
        val command = "java -cp " + cp + " org.h2.tools.Server"
        println("Starting Database with command: " + command)
        process = Some(Process(command).run())
        println("Database started ! ")
      case Some(_) =>
        println("H2 Database already started")
    }
  }
  val stopH2 = TaskKey[Unit]("stop", "Stops H2 database")
  val stopH2Task = stopH2 in H2 :={
    process match {
      case None => println("Database already stopped")
      case Some(_) =>
        println("Stopping database...")
        process.foreach{_.destroy()}
        process = None
        println("Database stopped...")
    }
  }
}
object MainBuild extends Build {
  import H2TaskManager._
  lazy val scalazVersion = "6.0.3"
  lazy val jettyVersion = "7.3.0.v20110203"
  
  lazy val wekanban = Project( "wekanban",
      file(".")).settings(startH2Task, stopH2Task)
}
~~~~~~

Change to this
~~~~~~
libraryDependencies ++= Seq(
  "org.scalaz" %% "scalaz-core" % scalazVersion,
  "org.scalaz" %% "scalaz-http" % scalazVersion,
  "org.eclipse.jetty" % "jetty-servlet" % jettyVersion % "container",
  "org.eclipse.jetty" % "jetty-webapp" % jettyVersion % "test, container",
  "org.eclipse.jetty" % "jetty-server" % jettyVersion % "container",
  "com.h2database" % "h2" % "1.2.137",
  "org.squeryl" % "squeryl_2.10" % "0.9.5-6"
)
~~~~~~

browser `http://localhost:8082` to set h2base
    JDBC Driver class: org.h2.Driver
     Database URL: jdbc:h2:tcp://localhost/~/test
     User name: sa

~~~~~~
package com.kanban.models
import org.squeryl._
import org.squeryl.adapters._
import org.squeryl.PrimitiveTypeMode._
import java.sql.DriverManager
object KanbanSchema extends Schema {
  val stories = table[Story]("STORIES")
  def init = {
    import org.squeryl.SessionFactory
    Class.forName("org.h2.Driver")
// The Squeryl Session instance provides additional methods
// like log and methods for binding/unbinding the session to the current thread. 
    if(SessionFactory.concreteFactory.isEmpty) {
      SessionFactory.concreteFactory = Some(()=>
        Session.create(
          DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test",
          "sa", ""), new H2Adapter))
    }
  }
  def tx[A](a: =>A): A = {
    init
    inTransaction(a)
  }
}

def main(args: Array[String]) {
  println("initializing the weKanban schema")
  init
  inTransaction { drop ; create }
}

~~~~~~

加入检查, 和保存操作
~~~~~~
package com.kanban.models
import org.squeryl._
import org.squeryl.PrimitiveTypeMode._
import org.squeryl.annotations._
import KanbanSchema._
class Story(val number: String, val title: String, val phase: String){
  private[this] def validate = {
    if(number.isEmpty || title.isEmpty) {
      throw new ValidationException ("Both number and title are required")
    }
    if(!stories.where(a => a.number === number).isEmpty) {
      throw new ValidationException ("The story number is not unique")
    }
  }
  def save(): Either[Throwable, String] = {
    tx {
      try {
        validate
        stories.insert(this)
        Right("Story is created successfully")
      } catch {
        case exception: Throwable => Left(exception)
      }
    }
  }
}
object Story {
  def apply(number: String, title: String) =
  new Story(number, title, "ready")
}
class ValidationException(message: String) extends
    RuntimeException(message)
~~~~~~

## web page ##

~~~~~~
package com.kanban.views
object CreateStory {
  def apply(message: String = "") =
    <html> TODO 7.1.2
    </html>
}
~~~~~~

~~~~~~
import
import scalaz._
import Scalaz._
import scalaz.http._
import response._
import request._
import servlet._
import HttpServlet._
import Slinky._
import com.kanban.views._
import com.kanban.models._
final class WeKanbanApplication extends StreamStreamServletApplication {
  import Request._
  import Response._
  implicit val charset = UTF8

  // To read POST parameters from the request
  // use ! POST generally means a side-effect
  def param_!(name: String)(implicit request: Request[Stream]) =
    (request | name).getOrElse(List[Char]()).mkString("")

  // return parameter value as string
  def param(name: String)(implicit request: Request[Stream]) =
    (request ! name).getOrElse(List[Char]()).mkString("")

  def handle(implicit request: Request[Stream],
     servletRequest: HttpServletRequest): Option[Response[Stream]] = {
    request match {
      case MethodParts(GET, "card" :: "create" :: Nil) =>
        Some(OK(ContentType, "text/html") << strict <<
        CreateStory(param("message")))
      case MethodParts(POST, "card" :: "save" :: Nil) =>
        Some(saveStory)
      case MethodParts(GET, "kanban" :: "board" :: Nil) =>
        Some(OK(ContentType, "text/html") << transitional << KanbanBoard())
      case _ => None
    }
  }

  private def saveStory(implicit request: Request[Stream],
        servletRequest: HttpServletRequest) = {
    val title = param_!("title")
    val number = param_!("storyNumber")
    Story(number, title).save match {
      case Right(message) =>
        redirects[Stream, Stream]("/card/create", ("message", message))
      case Left(error) => OK(ContentType, "text/html") << strict <<
        CreateStory(error.toString)
    }
}

  // The Scalaz core provides a method called | for the Option class and
  // using it you can combine both handle and resource methods so that when the han-
  // dle method returns None you can invoke the resource method as a fallback to load
  // resources. 
  val application = new ServletApplication[Stream, Stream] {
    def application(implicit servlet: HttpServlet,
        servletRequest: HttpServletRequest, request: Request[Stream]) = {
      def found(x: Iterator[Byte]) : Response[Stream] = OK << x.toStream
      handle | resource(found, NotFound.xhtml)
    }

  }
}
~~~~~~

~~~~~~
// Implementing drag-and-drop for the weKanban board in the main.js file
  
function moveCard(storyNumber, phase) {
  $.post("/card/move",{storyNumber: storyNumber, phase: phase},
    function(message) {
      $('#message').html(message)
    });
  }
  function init() {
    $(function() {
      $(".story").each(function() {
        $(this).draggable();
      });
    $("#readyPhase").droppable({
      drop: function(event, ui) {
        moveCard(ui.draggable.attr("id"), "ready") }
    });
    $("#devPhase").droppable({
      drop: function(event, ui) {
      moveCard(ui.draggable.attr("id"), "dev") }
    });
    $("#testPhase").droppable({
      drop: function(event, ui) {
        moveCard(ui.draggable.attr("id"),
          "test") }
    });
    $("#deployPhase").droppable({
      drop: function(event, ui) {
        moveCard(ui.draggable.attr("id"),
          "deploy") }
    });
  });
}
~~~~~~

~~~~~~
// story model
// At this point Squeryl has only created the query—it hasn’t executed it in the database. It will execute the query the
// moment you try to access the first element in the collection.
//  by invoking the map method, so that you access these instances of Story objects outside the transaction
def findAllByPhase(phase: String) = tx {
  from(stories)(s => where(s.phase === phase) select(s)) map(s => s)
}
~~~~~~

~~~~~~
package com.kanban.views
import com.kanban.models._
// To create the view for the Kanban board
object KanbanBoard {
    private def header =
      <head>
        <meta charset="UTF-8" />
        <title>weKanban: A simple Kanban board</title>
        <script type="text/javascript" src="/js/jquery-1.4.2.js"/>
        <script type="text/javascript" src="/js/jquery.ui.core.js"/>
        <script type="text/javascript" src="/js/jquery.ui.widget.js"/>
        <script type="text/javascript" src="/js/jquery.ui.mouse.js"/>
        <script type="text/javascript" src="/js/jquery.ui.draggable.js"/>
        <script type="text/javascript" src="/js/jquery.ui.droppable.js"/>
        <script type="text/javascript" src="/js/main.js"/>
        <link type="text/css" href="/css/main.css" rel="stylesheet" />
        <script type="text/javascript">
          init()
        </script>
      </head>
    // 7.10, P216
    def apply() =
      <html>
        <head> {header}</head>
      </html>
    private def stories(phase: String) =
      for(story <- Story.findAllByPhase(phase)) yield
        <div id={story.number} class="story">
          <fieldset>
          <legend>{story.number}</legend>
            <div class="section">
              <label>{story.title}</label>
            </div>
          </fieldset>
        </div>
}

~~~~~~

## Moving cards in the Kanban board ##

~~~~~~
// in Story Class
private def phaseLimits = Map("ready" -> Some(3),
  "dev" -> Some(2), "test" -> Some(2), "deploy" -> None)

private[this] def validateLimit(phase: String) = {
  val currentSize:Long =
    from(stories)(s => where(s.phase === phase) compute(count))
  if(currentSize == phaseLimits(phase).getOrElse(-1)) {
    throw new ValidationException("You cannot exceed the limit set for
      the phase.")
  }
}

def findByNumber(number: String) =
  tx { stories.where(s => s.number === number).single }
  
def moveTo(phase: String): Either[Throwable, String] = {
  tx {
    try {
      validateLimit(phase)
      update(stories)(s =>
        where(s.number === this.number)
        set(s.phase := phase)
      )
      Right("Card " + this.number + " is moved to " + phase
           + " phase
           successfully.")
    } catch {
      case exception: Throwable => Left(exception)
    }
  }
}

~~~~~~

~~~~~~
// main.js
function moveCard(storyNumber, phase) {
  $.post("/card/move", {storyNumber: storyNumber, phase: phase},
    function(message) {
      $('#message').html(message)
    }
  );
}

~~~~~~

~~~~~~
// application
private def moveCard(implicit request: Request[Stream],
    servletRequest: HttpServletRequest) = {
  val number = param_!("storyNumber")
  val toPhase = param_!("phase")
  val story = Story.findByNumber(number)
  story.moveTo(toPhase) match {
    case Right(message) => OK(ContentType, "text/html") <<
      strict << message
    case Left(error) => OK(ContentType, "text/html") <<
      strict << error.getMessage
  }
}
// case MethodParts(POST, "card" :: "move" :: Nil) =>
//   Some(moveCard)

~~~~~~
