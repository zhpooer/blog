title: Play for Scala-Define Http interface
date: 2014-08-09 16:20:51
tags:
- play
- scala
---

# Benefits of good URL design #

If you don’t think changing the URL matters, then this is probably a good time to read
Cool URIs Don’t Change, which Tim Berners-Lee wrote in
1998 (http://www.w3.org/Provider/Style/URI.html),
adding to his 1992 WWW style guide, which is an important
part of the documentation for the web itself.

ervlet API URL
mapping is too limited to handle even our first three example URLs, because it only
lets you match URLs exactly, by prefix or by file extension. What’s missing is a notion
of path parameters that match variable segments of the URL, using URL templates:

~~~~~~
/product/{ean}/edit
/product/(\d+)/edit
~~~~~~


Here are several benefits of good URL design:
* A consistent public API —The URL scheme makes your application easier to
understand by providing an alternative machine-readable interface.
* The URLs don’t change—Avoiding implementation-specifics makes the URLs sta-
ble, so they don’t change when the technology does.
* Short URLs—Short URLs are more usable; they’re easier to type or paste into
other media, such as email or instant messages.

# Controllers—the interface between HTTP and Scala #

Controllers are the application components that handle HTTP requests for application
resources identified by URLs.

In Play, you use controller classes to make your application respond to HTTP
requests for URLs, such as the product catalog URLs:

~~~~~~
/products
/product/5010255079763
/product/5010255079763/edit
~~~~~~

## Controller classes and action methods ##

We’ll start by defining a Products controller class, which will contain four action
methods for handling different kinds of requests: list, details , edit, and update

A controller is a Scala object that’s a subclass of `play.api.mvc.Controller` ,
which provides various helpers for generating actions. 

~~~~~~
package controllers

import play.api.mvc.{Action, Controller}
object Products extends Controller {
  def list(pageNumber: Int) = Action {
    NotImplemented
  }
  def detail(ean: Long) = Action {
    NotImplemented
  }
  def edit(ean: Long) = Action {
    NotImplemented
  }
  def update(ean: Long) = Action {
    NotImplemented
  }
}
~~~~~~

**GROUP CONTROLLERS BY MODEL ENTITY**

Create one controller for each of the
key entities in your application’s high-level data model. For example, the four
key entities—Product, Order, Warehouse, and User—might correspond to a
data model with more than a dozen entities. In this case, it’d probably be a
good idea to have four controller classes: Products, Orders, Warehouses, and
Users. 

In Play, each controller is a Scala object that defines one or more actions. Play uses an
object instead of a class because the controller doesn’t have any state; the controller is
used to group some actions.

**DON’T DEFINE A var IN A CONTROLLER OBJECT**

Each action is a Scala function that takes an HTTP request and returns an HTTP result.
`Request[A] => Result`

The controller layer is therefore the mapping between stateless HTTP requests and
responses and the object-oriented model. In MVC terms, controllers process events
(HTTP requests in this case), which can result in updates to the model. Controllers are
also responsible for rendering views. 

## HTTP and the controller layer’s Scala API ##

Play models controllers, actions, requests, and responses as Scala traits in the
`play.api.mvc` package—the Scala API for the controller layer.

MVC API traits and classes correspond to HTTP concepts and act as
wrappers for the corresponding HTTP data:
* `play.api.mvc.Cookie` —An HTTP cookie: a small amount of data stored on the client and sent with subsequent requests
* `play.api.mvc.Request` —An HTTP request: HTTP method, URL, headers, body, and cookies
* `play.api.mvc.RequestHeader` —Request metadata: a name-value pair
* `play.api.mvc.Response` —An HTTP response, with headers and a body; wraps a Play Result
* `play.api.mvc.ResponseHeader`  —Response metadata: a name-value pair


Play controllers use the following concepts in addition
to HTTP concepts:

* `play.api.mvc.Action` — A function that processes a client Request
and returns a Result
* `play.api.mvc.Call` — An HTTP request: the combination of an HTTP method
and a URL
* `play.api.mvc.Content` — An HTTP response body with a particular content type
* `play.api.mvc.Controller` — A generator for Action functions
* `play.api.mvc.Flash` — A short-lived HTTP data scope used to set data for
the next request
* `play.api.mvc.Result` — The result of calling an Action to process a Request,
used to generate an HTTP response
* `play.api.mvc.Session` — A set of string keys and values,
stored in an HTTP cookie

## Action composition ##

~~~~~~
def list = Action {
  // Check authentication.
  // Check for a cached result.
  // Process request...
  // Update cache.
}

// 可以这样写
def list =
  Authenticated {
    Cached {
      Action {
        // Process request...
      }
    }
  }
~~~~~~

This example uses Action to create an action function that’s passed as a parameter to
Cached, which returns a new action function. This, in turn, is passed as a parameter to
Authenticated, which decorates the action function again.

# Routing HTTP requests to controller actions #

Once you have controllers that contain actions, you need a way to map different
request URLs to different action methods.

In Play, mapping the combination of an HTTP method and a URL to an action
method is called routing. 

## Router configuration ##

The great thing about this approach is that your web application’s URLs—its public
HTTP interface—are all specified in one place, which makes it easier for you to
maintain a consistent URL design.


The routes file structure is line-based: each line is either a blank line, a comment line,
or a route definition. A route definition has three parts on one line, separated by
whitespace.

`conf/routes`
~~~~~~
GET / controllers.Products.home()

GET /products  controllers.Products.list()

GET /products controllers.Products.list(page: Int ?= 1) ## Option[Int]

GET /products controllers.Products.list(page: Int = 1) ## Int

GET /product/:ean controllers.Products.details(ean: Long)

GET /product/:ean/edit controllers.Products.edit(ean: Long)
~~~~~~

The benefit of this format is that you can see your whole URL design in one place,
which makes it more straightforward to manage than if the URLs were specified in
many different files.

## Matching URL path parameters that contain forward slashes ##

    /photo/5010255079763.jpg
    /photo/customer-submissions/5010255079763/42.jpg
    /photo/customer-submissions/5010255079763/43.jpg

~~~~~~
## 这样上面的配置不会匹配 斜杠(slash)
GET /photo/:file controllers.Media.photo(file: String)

## 必须这样
GET /photo/*file controllers.Media.photo(file: String)
~~~~~~

## Constraining URL path parameters with regular expressions ##

为如下的url做匹配

    /product/5010255079763
    /product/paper-clips-large-plain-1000-pack

正则表达式要写在`<>`里面
~~~~~~
GET /product/$ean<\d{13}> controllers.Products.details(ean: Long)

GET /product/:alias controllers.Products.alias(alias: String)
~~~~~~

## Binding HTTP data to Scala objects ##

Play, along with other modern web frameworks such as Spring MVC, improves on
treating HTTP request parameters as strings by performing type conversion before it
attempts to call your action method. 

Here’s what happens when Play’s router handles the request `PUT /product/5010255079763`
1. The router matches the request against configured routes and selects the route:
`PUT /product/:ean controllers.Products.update(ean: Long)`
2. The router binds the ean parameter using one of the type-specific binders—in
this case, the Long binder converts 5010255079763 to a Scala Long object
3. The router invokes the selected route’s Products.update action, passing
`5010255079763L` as a parameter.

If you send an HTTP request for `/product/x`,
the binding will fail because x isn’t a number, and Play
will return an HTTP response with the 400 (Bad Request)
status code and an error page

# Generating HTTP calls for actions with reverse routing #

In addition to mapping incoming URL requests to controller actions,
a Play application can do the opposite: map a particular action
method invocation to the corresponding URL.

## Hardcoded URLs ##

The interesting part is what happens next, after the product is deleted.
Let’s suppose that after deleting the product,
we want to show the updated product list. We could
render the product list page directly, but this exposes us to the double-submit
problem: if the user “reloads” the page in a web browser,
this could result in a second call
to the delete action, which will fail because the specified product no longer exists.

The standard solution to the double-submit problem is the redirect-after-POST pattern:
after performing an operation that updates the application’s persistent state, the
web application sends an HTTP response that consists of an HTTP redirect.

~~~~~~
def delete(ean: Long) = Action {
  Product.delete(ean)
  Redirect("/proudcts")
}
~~~~~~

This looks like it will do the job, but it doesn’t smell too nice because
we’ve hardcoded the URL in a string.

解决办法如下

## Reverse routing ##

You can do reverse routing by writing Scala code.
~~~~~~
def delete(ean: Long) = Action {
  Product.delete(ean)
  Redirect(routes.Products.list())
}
~~~~~~

Keeping these two points in mind:
* Routing is when URLs are routed to actions—left to right in the routes file
* Reverse routing is when call definitions are “reversed” into URL s—right to left

~~~~~~
scala> val call = controllers.routes.Products.list()
scala> val (method, url) = (call.method, call.url)
~~~~~~

# Generating a response #

An HTTP response consists of an HTTP status code, optionally followed by response
headers and a response body. Play gives you total control over all three, which lets you
craft any kind of HTTP response you like, but it also gives you a convenient API for
handling common cases.

## Debugging HTTP responses ##
To use cURL, use the `--request` option to specify the HTTP method and
`--include` to include HTTP response headers in the output
~~~~~~
curl --request GET --include http://localhost:9000/products
~~~~~~

## Response body ##

The response body will consist of this representation, in some particular format.

 * Plain text—Such as an error message, or a lightweight web service response
 * HTML —A web page, including a representation of the resource as well as
 application user-interface elements, such as navigation controls
 * JSON —A popular alternative to XML that’s better suited to Ajax applications
 * XML —Data accessed via a web service
 * Binary data—Typically nontext media such as a bitmap image or audio

~~~~~~
// plain text
def version = Action {
  Ok("Version 2.0")
}

// html
def index = Action {
  Ok(views.html.index())
}

// json
def json = Action {
  import play.api.libs.json.Json
  val success = Map("status" -> "success")
  val json = Json.toJson(success)
  Ok(json)
}

// scala.xml.NodeSeq
def xml = Action {
  Ok(<status>success</status>)
}

~~~~~~

### BINARY DATA ###

In Play, returning a binary result to the
web browser is the same as serving other formats: as with XML and JSON, pass the
binary data to a result type. 

条形码生成
~~~~~~
val appDependencies = Seq(
  "net.sf.barcode4j" % "barcode4j" % "2.0"
)
~~~~~~

~~~~~~
def ean13Barcode(ean: Long, mimeType: String): Array[Byte] = {
  import java.io.ByteArrayOutputStream
  import java.awt.image.BufferedImage
  import org.krysalis.barcode4j.output.bitmap.BitmapCanvasProvider
  import org.krysalis.barcode4j.impl.upcean.EAN13Bean
  
  val BarcodeResolution = 72
  val output: ByteArrayOutputStream = new ByteArrayOutputStream
  val canvas: BitmapCanvasProvider =
    new BitmapCanvasProvider(output, mimeType,
          BarcodeResolution, BufferedImage.TYPE_BYTE_BINARY, false, 0)
  val barcode = new EAN13Bean()
  
  barcode.generateBarcode(canvas, String valueOf ean)
  canvas.finish
  output.toByteArray
}

// GET /barcode/:ean controllers.Products.barcode(ean: Long)
def barcode(ean: Long) = Action {
  import java.lang.IllegalArgumentException
  val MimeType = "image/png"
  try {
    val imageData: Array[Byte] = ean13Barcode(ean, MimeType)
    Ok(imageData).as(MimeType)
  } catch {
    case e: IllegalArgumentException =>
      BadRequest("Could not generate bar code. Error: " + e.getMessage)
  }
}

~~~~~~

## HTTP status codes ##

The simplest possible response that you might want to generate consists of only an
HTTP status line that describes the result of processing the request. 

~~~~~~
def list = Action { request =>
  NotImplemented
}

// 等同于
def list = Action {
  new Status(501)
}

~~~~~~
*NotImplemented* is one of many HTTP status codes that are defined in the
`play.api.mvc.Controller` class via the `play.api.mvc.Results` trait.

## Response headers ##

In addition to a status, a response may also include response headers:
metadata that instructs HTTP clients how to handle the response

    HTTP/1.1 501 Not Implemented
    Content-Length: 0


~~~~~~
Status(FOUND).withHeaders(LOCATION -> url)

val url = routes.Products.details(product.ean).url
Created.withHeaders(LOCATION -> url)
~~~~~~

### SETTING THE CONTENT TYPE ###

Every HTTP response that has a response body also has a Content-Type header,
whose value is the MIME type that describes the response body format.

Play automatically sets
the content type for supported types, such as text/html when rendering an HTML
template, or text/plain when you output a string response.

~~~~~~
val json = """{ "status": "success" }"""
Ok(json).withHeaders(CONTENT_TYPE -> "application/json")

// 也可以这样
Ok("""{ "status": "success" }""").as("application/json")

//  JSON is defined in the play.api.http.ContentTypes trait
//  which Controller extends.
Ok("""{ "status": "success" }""").as(JSON)
~~~~~~

Play sets the content type automatically for some more types:
Play selects `text/xml` for
`scala.xml.NodeSeq` values, and `application/json`
for `play.api.libs.json.JsValue` values.

### SESSION DATA ###

~~~~~~
Ok(results).withSession(
  request.session + ("search.previous" -> query)
)

Ok(results).withSession(
  request.session - "search.previous"
)
~~~~~~

The session is implemented as an HTTP session cookie, which means that
its total size is limited to a few kilobytes.

**DON’T CACHE DATA IN THE SESSION COOKIE** Don’t try to use session data as a
cache to improve performance by avoiding fetching data from server-side
persistent storage. Apart from the fact that session data is limited to the 4 KB of data
that fits in a cookie, this will increase the size of subsequent HTTP requests,
which will include the cookie data, and may make performance worse overall.


The canonical use case for session cookies is to identify the currently authenticated
user.  You can load user-specific data from a persistent data model instead.

The session Play cookie is signed using the application secret key as a salt to pre-
vent tampering.

### FLASH DATA ###

Displaying a message when handling the next request, after a redirect, is such a
common use case that Play provides a special session scope called flash scope.

Flash scope works the same way as the session, except that any data
that you store is only available when processing the next HTTP request,
after which it’s automatically deleted. 

~~~~~~
// 设置
Redirect(routes.Products.flash()).flashing(
  "info" -> "Product deleted!"
)

// 获取
val message = request.flash("info")
~~~~~~

### SETTING COOKIES ###

Cookies store small amounts of data in an HTTP client, such as a web browser on a
specific computer.

If you do need to use cookies, you can use the Play API to
create cookies and add them to the response, and to read them from the request.

**AVOID USING COOKIES**

## Serving static content ##

Not everything in a web application is dynamic content: a typical web application also
includes static files, such as images, JavaScript files, and CSS stylesheets. Play serves
these static files over HTTP the same way it serves dynamic responses: by routing an
HTTP request to a controller action.

### USING THE DEFAULT CONFIGURATION ###

Put files and folders inside your application’s `public/` folder and access
them using the URL path `/assets`, followed by the path relative to public.

`public/images/favicon.png` 访问 `http://localhost:9000/assets/images/favicon.png`

~~~~~~
<link href="/assets/images/favicon.png" rel="shortcut icon" type="image/png"/>

~~~~~~

`conf/routes` 的默认配置
~~~~~~
GET /assets/*file controllers.Assets.at(path="/public", file)
GET /images/*file controllers.Assets.at(path="/public/images", file)
GET /styles/*file controllers.Assets.at(path="/public/styles", file)
~~~~~~

### USING AN ASSET’S REVERSE ROUTE ###

~~~~~~
<link href="@routes.Assets.at("images/favicon.png")"
      rel="shortcut icon" type="image/png">

<link href="@routes.Assets.at("/public/images", "favicon.png")"
      rel="shortcut icon" type="image/png">
~~~~~~

### CACHING AND ETAGS ###

In addition to reverse routing, another benefit of using the assets controller
is its builtin caching support, using an HTTP Entity Tag (ET ag).
This allows a web client to make conditional HTTP requests for a resource
so that the server can tell the client it can
use a cached copy instead of returning a resource that hasn’t changed.

Once it has an ET ag value, an HTTP client can make a conditional request, which
means “only give me this resource if it hasn’t been modified since I got the version
with this ET ag.”

    If-None-Match: 978b71a4b1fef4051091b31e22b75321c7ff0541

When this header is included in the request, and the favicon.png file hasn’t been
modified (it has the same ETag value), then Play’s assets controller will return the fol-
lowing response, which means “you can use your cached copy”:

    HTTP/1.1 304 Not Modified
    Content-Length: 0

### COMPRESSING ASSETS WITH GZIP ###

HTTP compression is a feature of modern web servers and web clients that helps
address page sizes by sending compressed versions of resources over HTTP .

The way this works is that the web browser indicates that it can handle a com-
pressed response by sending an HTTP request header such as `Accept-Encoding:gzip`
that specifies supported compression methods. 

In Play, HTTP compression is transparently built into the assets controller, which can
automatically serve a compressed version of a static file, if it’s available, and if gzip is
supported by the HTTP client. This happens when all of the following are true:

* Play is running in prod mode (production mode is explained in chapter 9); HTTP
compression isn’t expected to be used during development.
* Play receives a request that’s routed to the assets controller.
* The HTTP request includes an Accept-Encoding: gzip header.
* The request maps to a static file, and a file with the same name but with an
additional `.gz` suffix is found.

所以必须要先运行
~~~~~~
gzip --best < ui.js > ui.js.gz
~~~~~~
