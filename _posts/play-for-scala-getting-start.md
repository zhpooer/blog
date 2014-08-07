title: Play for Scala-Getting Start
date: 2014-08-07 16:35:31
tags:
- scala
- play
---

# What Play is #

Play makes you more productive. Play is also a web framework whose HTTP interface is
simple, convenient, flexible, and powerful. Most importantly, Play improves on the
most popular non-Java web development languages and frameworks—PHP and Ruby
on Rails—by introducing the advantages of the Java Virtual Machine (JVM).


# Hello Play #
~~~~~~
play new hello
play run
~~~~~~

Files in a new Play application

~~~~~~
.gitignore
app/controllers/Application.scala
app/views/index.scala.html
app/views/main.scala.html
conf/application.conf
conf/routes
project/build.properties
project/Build.scala
project/plugins.sbt
public/images/favicon.png
public/javascripts/jquery-1.7.1.min.js
public/stylesheets/main.css
test/ApplicationSpec.scala
test/IntegrationSpec.scala
~~~~~~

`app/controllers/Application.scala`
~~~~~~
def index = Action {
  Ok("Hello world")
}
~~~~~~

`conf/routes`
~~~~~~
GET / controllers.Application.index()
~~~~~~

也可以这样
~~~~~~
def hello(name: String) = Action {
  Ok("Hello " + name)
}

// 访问 http://localhost:9000/hello?n=Play!
// GET /hello controllers.Application.hello(n: String)
~~~~~~
