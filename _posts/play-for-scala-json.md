title: Play for Scala-JSON
date: 2014-08-18 10:39:36
tags:
- play
- scala
---

# Creating the single-page Play application #

In this section, we’ll add dynamic data from the server to our web page: a table of
products that shows each product’s EAN code
~~~~~~
package models
case class Product(ean: Long, name: String, description: String)
object Product {
  var products = Set(
    Product(5010255079763L, "Paperclips Large",
      "Large Plain Pack of 1000"),
    Product(5018206244666L, "Giant Paperclips",
      "Giant Plain 51mm 100 pack"),
    Product(5018306332812L, "Paperclip Giant Plain",
      "Giant Plain Pack of 10000"),
    Product(5018306312913L, "No Tear Paper Clip",
      "No Tear Extra Large Pack of 1000"),
    Product(5018206244611L, "Zebra Paperclips",
      "Zebra Length 28mm Assorted 150 Pack")
  )

  def findAll = this.products.toList.sortBy(_.ean)
  def findByEan(ean: Long) = this.products.find(_.ean == ean)
  def save(product: Product) = {
    findByEan(product.ean).map( oldProduct =>
      this.products = this.products - oldProduct + product
    ).getOrElse(
      throw new IllegalArgumentException("Product not found")
    )
  }
}
~~~~~~
Page template

~~~~~~
<!DOCTYPE html>
<html>
  <head>
    <title>Products</title>
    <link rel='stylesheet' type='text/css'
          href='@routes.Assets.at("stylesheets/bootstrap.css")'>
    <link rel='stylesheet' type='text/css'
          href="@routes.Assets.at("stylesheets/main.css")">
    <script src="@routes.Assets.at("javascripts/jquery-1.9.0.min.js")"
            type="text/javascript"></script>
    <script src='@routes.Assets.at("javascripts/products.js")'
            type='text/javascript'></script>
  </head>
  <body>
    <div class="screenshot">
      <div class="navbar navbar-fixed-top">
        <div class="navbar-inner">
          <div class="container">
            <a class="brand" href="@routes.Application.index()">
            </a>
            <ul class="nav"></ul>
          </div>
        </div>
      </div>
      <div class="container">
      </div>
    </div>
  </body>
</html>
~~~~~~

## Constructing JSON data value objects ##

~~~~~~
package controllers
import play.api.mvc.{Action, Controller}
import models.Product
import play.api.libs.json.Json

object Products extends Controller {
  def list = Action {
    val productCodes = Product.findAll.map(_.ean)
    //  automatically add a Content-Type: application/json HTTP response header
    Ok(Json.toJoson(productCodes))
  }
}
~~~~~~

`conf/routes`
~~~~~~
GET / controllers.Application.index
GET /products controllers.Products.list
GET /assets/*file controllers.Assets.at(path="/public", file)
~~~~~~

可以通过`curl --include http://localhost:9000/products`, 测试

### WORKING WITH THE JSON OBJECTS ###

The `play.api.libs.json.JsValue` type represents any kind of JSON value.

Play’s JSON library is located in `play.api.libs.json`, and it contains case classes
for each of JSON’s types:
* JsString,  takes a String as a parameter 
* JsNumber, takes a BigDecimal
* JsBoolean,  can be constructed from a sequence of key/value tuples: `Seq[(String, JsValue)]`.
* JsObject
* JsArray, takes a Seq[JsValue]
* JsNull

~~~~~~
import play.api.libs.json._
val category = JsString("paperclips")
val quantity = JsNumber(42)

val product = Json.obj(
  "name" -> JsString("Blue Paper clips"),
  "ean" -> JsString("12345432123"),
  "description" -> JsString("Big box of paper clips"),
  "pieces" -> JsNumber(500),
  "manufacturer" -> Json.obj(
    "name" -> JsString("Paperclipfactory Inc."),
    "contact_details" -> Json.obj(
      "email" -> JsString("contact@paperclipfactory.example.com"),
      "fax" -> JsNull,
      "phone" -> JsString("+12345654321")
    )
  ),
  "tags" -> Json.arr(
    JsString("paperclip"),
    JsString("coated")
  ),
  "active" -> JsBoolean(true)
)
~~~~~~

### GENERATING STRINGS FROM JSON VALUES ###

When you return JSON from a controller action,
you pass the JsValue to the result directly.

~~~~~~
val productJsonString = Json.stringify(product)
~~~~~~

### FETCHING JSON DATA FROM THE CLIENT ###

~~~~~~
<div class="container">
  <table data-list="@routes.Products.list">
  </table>
</div>
~~~~~~

`app/assets/javascripts/products.coffee`
~~~~~~
jQuery ($) ->
  $table = $('.container table')
  productListUrl = $table.data('list')

  $.get productListUrl, (products) ->
    $.each products, (index, eanCode) ->
      row = $('<tr/>').append $('<td/>').text(eanCode)
      $table.append row
~~~~~~

## Converting model objects to JSON objects ##

~~~~~~
GET /products/:ean controllers.Products.details(ean: Long)
~~~~~~

~~~~~~
def details(ean: Long) = Action {
  Product.findByEan(ean).map { product =>
    Ok(Json.toJson(product))
  }.getOrElse(NotFound)
}
~~~~~~

### JSON FORMATTERS ###

The type signature of the toJson method looks like this:
~~~~~~
def toJson[T](object: T)(implicit writes: Writes[T]): JsValue

// writes implementations
implicit object StringWrites extends Writes[String] {
  def writes(o: String) = JsString(o)
}
~~~~~~

Play also provides implicit conversions from `Writes[T]` to `Writes[List[T]]`,
`Writes[Set[T]]`, and `Writes[Map[String, T]]`.

~~~~~~
case class Product(ean: Long, name: String, description: String)

import play.api.libs.json._
implicit object ProductWrites extends Writes[Product] {
  def writes(p: Product) = Json.obj(
    "ean" -> Json.toJson(p.ean),
    "name" -> Json.toJson(p.name),
    "description" -> Json.toJson(p.description)
  )
}

// 更简介的方法
import play.api.libs.json._
import play.api.libs.functional.syntax._
implicit val productWrites: Writes[Product] = (
  (JsPath \ "ean").write[Long] and
  (JsPath \ "name").write[String] and
  (JsPath \ "description").write[String]
)(unlift(Product.unapply))

// we can use a helper function that defines a default case
// class formatter at runtime, which gives
// the same output as the previous example:
import play.api.libs.json._
import play.api.libs.functional.syntax._
implicit val productWrites = Json.writes[Product]
~~~~~~

`Writes[Product]` for the administrative interface

~~~~~~
import play.api.libs.json._
import play.api.libs.functional.syntax._
val adminProductWrites: Writes[Product] = (
  (JsPath \ "ean").write[Long] and
  (JsPath \ "name").write[String] and
  (JsPath \ "description").write[String] and
  (JsPath \ "price").write[BigDecimal]
)(unlift(Product.unapply))

val json = Json.toJson(product)(AdminProductWrites)
~~~~~~

### USING A CUSTOM FORMATTER ###

~~~~~~
<table data-list="@routes.Products.list"
       data-details="@routes.Products.details(0)">
</table>
~~~~~~

~~~~~~
jQuery ($) ->
  $table = $('.container table')
  productListUrl = $table.data('list')

  loadProductTable = ->
    $.get productListUrl, (product) ->
      $.each products, (index, eanCode) ->
        row = $('<tr/>').append $('<td/>').text(eanCode)
        $table.append row
        loadProductDetail row

  productDetailUrl = (eanCode) ->
    $talbe.data('details').replace '0', eanCode

  loadProductDetails = (tableRow) ->
    eanCode = tableRow.text()

    $.get productDetailUrl (eanCode), (product) ->
      tableRow.append $('<td/>').text(product.name)
      tableRow.append $('<td/>').text(product.description)
      
  loadProductTable()
~~~~~~

# Sending JSON data to the server #
We’re going to cheat by using the HTML5 contenteditable attribute to
make the table cells directly editable.


## Editing and sending client data ##

~~~~~~
saveRow = ($row) ->
  [ean, name, description] = $row.children().map -> $(this).text()
  product =
    ean: parseInt(ean)
    name: name
    description: description
  jqxhr = $.ajax
    type: "PUT"
    url: productDetailsUrl(ean)
    contentType: "application/json"
    data: JSON.stringify product

  jqxhr.done (response) ->
    $label = $('<span/>').addClass('label label-success')
    $row.children().last().append $label.text(response)
    $label.delay(3000).fadeout()
  jqxhr.fail (data) ->
    $label = $('<span/>').addClass('label label-important')
    message = data.responseText || data.statusText
    $row.children().last().append $label.text(message)
    
$table.on 'focusout', 'tr', () ->
  saveRow $(this)
~~~~~~


## Consuming JSON ##

~~~~~~
PUT /products/:ean controllers.Products.save(ean: Long)
~~~~~~

~~~~~~
def save(ean: Long) = Action(parse.json) { request =>
  val productJson = request.body
  val product = productJson.as[Product]

  try {
    Product.save(product)
    Ok("Saved")
  } catch {
    case e: IllegalArgumentException =>
      BadRequest("Product not found")
  }
}

import play.api.libs.functional.syntax._
implicit val productReads: Reads[Product] = (
  (JsPath \ "ean").read[Long] and
  (JsPath \ "name").read[String] and
  (JsPath \ "description").read[String]
)(Product.apply _)

def save(product: Product) = {
  findByEan(product.ean).map( oldProduct =>
    this.products = this.products - oldProduct + product
  ).getOrElse(
    throw new IllegalArgumentException("Product not found")
  )
}
~~~~~~

## Consuming JSON in more detail ##

Consuming JSON is a two-step process. The first step is going from a JSON string to JsValue objects.
~~~~~~
import play.api.libs.json._
val jsValue: JsValue = Json.parse("""{ "name" : "Johnny" }""")
~~~~~~

Often, you don’t even need to manually perform this step.
~~~~~~
def postProduct() = Action { request =>
  val jsValueOption = request.body.asJson
  jsValueOption.map { json =>
    // Do something with the JSON
  }.getOrElse {
    // Not a JSON body
  }
}
~~~~~~

If you’re only willing to accept JSON for an action, which is common,
you can use the `parse.json` body parser:
~~~~~~
// it’ll return an HTTP status of 400 Bad Request if the content type is wrong.
def postProduct2() = Action(parse.json) { request =>
  val jsValue = request.body
  // Do something with the JSON
}
~~~~~~
Sometimes you have to deal with misbehaving clients that send JSON without
proper Content-Type headers. In that case, you can use the `parse.tolerantJson`
body parser, which doesn’t check the header, but just tries to parse the body as JSON.

JsValue has the `as[T]` and `asOpt[T]` methods,

~~~~~~
val jsValue = JsString("Johnny")
val name = jsValue.as[String]

val age = jsValue.as[Int] // Throws play.api.libs.json.JsResultException
val age: Option[Int] = jsValue.asOpt[Int] // == None
val name: Option[String] = jsValue.asOpt[String] // == Some("Johnny")

val age = jsValue.validate[Int] // == JsError
val name = jsValue.validate[String] // == JsSuccess(Johnny,)
~~~~~~

Of course, often you’ll be dealing with more complex JSON structures. There are
three methods for traversing a `JsValue` tree:
* `\`—Selects an element in a `JsObject`, returning a `JsValue`
* `\\`—Selects an element in the entire tree, returning a `Seq[JsValue]`
* `apply`—Selects an element in a `JsArray` , returning a `JsValue`
~~~~~~
import Json._
val json: JsValue = toJson(Map(
  "name" -> toJson("Johnny"),
  "age" -> toJson(42),
  "tags" -> toJson(List("constructor", "builder")),
  "company" -> toJson(Map(
    "name" -> toJson("Constructors Inc.")))))

val name = (json \ "name").as[Strnig]
val age = (json \ "age").asOpt[Int]
val companyName = (json \ "company" \ "name").as[String]
val firstTag = (json \ "tags")(0).as[String]
val allNames = (json \\ "name").map(_as[String])

(json \ "name") match {
  case JsString(name) => println(name)
  case JsUndefined(error) => println(error)
  case _ => println("Invalid type!")
}

~~~~~~


## Reusable consumers ##
The `Reads[T]` trait has a single method, `reads(json: JsValue): JsResult[T]`,
which deserializes JSON into a `JsSuccess` that wraps an object of type T or a `JsError`
that gives you access to JSON parsing errors, following the pattern of Scala’s
`Either[Error, T]`.

~~~~~~
jsValue.as[String]
jsValue.as[String](play.api.libs.json.Reads.StringReads)
~~~~~~

~~~~~~
case class PricedProduct(
  name: String,
  description: Option[String],
  purchasePrice: BigDecimal,
  sellingPrice: BigDecimal)

val productJsonString = """{
  "name": "Sample name",
  "description": "Sample description",
  "purchase_price" : 20,
  "selling_price": 35
}"""

import play.api.libs.json._
import play.api.libs.functional.syntax._
implicit val productReads: Reads[PricedProduct] = (
  (JsPath \ "name").read[String] and
  (JsPath \ "description").readNullable[String] and
  (JsPath \ "purchase_price").read[BigDecimal] and
  (JsPath \ "selling_price").read[BigDecimal]
)(PricedProduct.apply _)

val productJsValue = Json.parse(productJsonString)
val product = productJsValue.as[PricedProduct]
~~~~~~

## Combining JSON formatters and consumers ##

The trait `Format[T]` extends both `Reads[T]` and `Writes[T]`

We can define a `Format[PricedProduct]` implementation,
using JsPath’s format and `formatNullable` methods, as shown in the following listing.

~~~~~~
import play.api.libs.json._
import play.api.libs.functional.syntax._
implicit val productFormat = (
  (JsPath \ "name").format[String] and
  (JsPath \ "description").formatNullable[String] and
  (JsPath \ "purchase_price").format[BigDecimal] and
  (JsPath \ "selling_price").format[BigDecimal]
)(PricedProduct.apply, unlift(PricedProduct.unapply))

// 也可以这样
implicit val productFormat = Format(productReads, productWrites)

import play.api.libs.json._
implicit val productReads = Json.reads[PricedProduct]
implicit val productWrites = Json.writes[PricedProduct]
~~~~~~

# Validating JSON #

~~~~~~
{
  "name": "Blue Paper clips",
  "ean": "12345432123",
  "description": "Big box of paper clips",
  "pieces": 500,
  "manufacturer": {
    "name": "Paperclipfactory Inc.",
    "contact_details": {
      "email": "contact@paperclipfactory.example.com",
      "fax": null,
      "phone": "+12345654321"
    }
  },
  "tags": [
    "paperclip",
    "coated"
  ],
  "active": true
}
~~~~~~

## Mapping the JSON structure to a model ##

~~~~~~
case class Contact(email: Option[String], fax: Option[String],
    phone: Option[String])
case class Company(name: String, contactDetails: Contact)
case class Product(ean: Long, name: String,
    description: Option[String], pieces: Option[Int],
    manufacturer: Company, tags: List[String], active: Boolean)

implicit val companyReads: Reads[Company] = (
  (JsPath \ "name").read[String] and
  (JsPath \ "contact_details").read(
  (
    (JsPath \ "email").readNullable[String] and
    (JsPath \ "fax").readNullable[String] and
    (JsPath \ "phone").readNullable[String]
  )(Contact.apply _))
)(Company.apply _)

implicit val productReads: Reads[Product] = (
  (JsPath \ "ean").read[Long] and
  (JsPath \ "name").read[String] and
  (JsPath \ "description").readNullable[String] and
  (JsPath \ "pieces").readNullable[Int] and
  (JsPath \ "manufacturer").read[Company] and
  (JsPath \ "tags").read[List[String]] and
  (JsPath \ "active").read[Boolean]
)(Product.apply _)

~~~~~~

## Handling “empty” values ##

The JSON API uses a nullable[T] to handle the JsNull case, such as our example’s
description: Option[String] property. 

~~~~~~
def nullable[T](implicit rds: Reads[T]): Reads[Option[T]] = Reads(js =>
  js match {
    case JsNull => JsSuccess(None)
    case js => rds.reads(js).map(Some(_))
  }
)
~~~~~~

## Adding validation rules and validating input ##

~~~~~~
implicit val companyReads: Reads[Company] = (
  (JsPath \ "name").read[String] and
  (JsPath \ "contact_details").read(
  (
    (JsPath \ "email").readNullable[String](email) and
    (JsPath \ "fax").readNullable[String](minLength[String](10)) and
    (JsPath \ "phone").readNullable[String](minLength[String](10))
  )(Contact.apply _))
)(Company.apply _)

implicit val productReads: Reads[Product] = (
  (JsPath \ "ean").read[Long] and
  (JsPath \ "name").read[String](minLength[String](5)) and
  (JsPath \ "description").readNullable[String] and
  (JsPath \ "pieces").readNullable[Int] and
  (JsPath \ "manufacturer").read[Company] and
  (JsPath \ "tags").read[List[String]] and
  (JsPath \ "active").read[Boolean]
)(Product.apply _)

def save = Action(parse.json) { implicit request =>
  val json = request.body
  json.validate[Product].fold(
    valid = { product =>
      Product.save(product)
      Ok("Save")
    },
    invalid = {
      error => BadRequest(JsError.toFlatJson(errors))
    }
  )
}
~~~~~~

## Returning JSON validation errors ##

This fails validation with the following JSON result, which is generated by the
`JsError.toFlatJson` helper.
~~~~~~
{
  "obj.manufacturer.contact_details.email" : [
    { "msg" : "validate.error.email", "args" : [] }
  ],
  "obj.name" : [
    {"msg" : "validate.error.minlength", "args" : [5] }
  ],
  "obj.tags" : [
    {"msg" : "validate.error.missing-path", "args" : [] }
  ]
}

// You may requires errors in a particular simplified JSON format
[
  {
    "path" : "/manufacturer/contact_details/email",
    "errors": ["validate.error.email"]
  },
  { "path" : "/name", "errors" : ["validate.error.minlength"] },
  { "path" : "/tags", "errors" : ["validate.error.missing-path"] }
]

// format a path as a string
implicit val JsPathWrites =
  Writes[JsPath](p => JsString(p.toString))

// format an error as a string
implicit val ValicateionErrorWrites =
  Writes[ValidationError](e => JsString(e.message))

implicit val jsonValidateErrorWrites = (
  (JsPath \ "path").write[JsPath] and
  (JsPath \ "errors").write[Seq[ValidationError]]
  tupled
)
~~~~~~


One such library is Jerkson; it’s possible to use Jerkson directly, or
you can use any other JSON library that you like.

# Authenticating JSON web service requests #

Authentication means identifying the “user” who’s sending the request,
by requiring and checking valid credentials, usually username and password.

In a conventional web application, authentication is usually implemented by using
an HTML login form to submit credentials to a server application, which then
maintains a session state that future requests from the same user are
associated with. In our JSON web service architecture, there are no HTML forms,
so we use different methods to associate authentication credentials with requests.

## Adding authentication to action methods ##
The simplest approach is to perform authentication for every HTTP request, before
returning the usual response or an HTTP error that indicates that the client isn’t
authorized to access the requested resource.

### COMPOSING ACTIONS TO ADD BEHAVIOR ###

~~~~~~
def index = AuthenticatedAction { request =>
  Ok("Authenticated response...")
}

def AuthenticatedAction(f: Request[AnyContent] => Result):
    Action[AnyContent] = {
  Action { request =>
    if (authenticate(request)) {
      f(request)
    } else {
      Unauthorized
    }
  }
}
~~~~~~

### EXTRACTING CREDENTIALS FROM THE REQUEST  ###
~~~~~~
// Helper function to extract credentials from a request query string
def readQueryString(request: Request[_]):
    Option[Either[Result, (String, String)]] = {
  request.queryString.get("user").map{ user =>
    request.queryStirng.get("password").map { password =>
      Right((user.head, password.head))
    }.getOrElse {
      Left(BadRequest("Password not specified"))
    }
  }
}

// Updated action helper that extracts credentials before authentication
def AuthenticatedAction(f: Request[AnyContent] => Result):
    Action[AnyContent] = {
  Action { request =>
    val maybeCredentials = readQueryString(request)
    maybeCredentials.map { resultOrCredentials =>
      resultOrCredentials match {
        case Left(errorResult) => errorResult
        case Right(credentials) => {
          val (user, password) = credentials
          if (authenticate(user, password)) {
            f(request)
          } else {
            Unauthorized("Invalid user name or password")
          }
        }
      }
    }.getOrElse {
      Unauthorized("No user name and password provided")
    }
  }    
}
~~~~~~

## Using basic authentication ##

A more standard way to send authentication credentials with an HTTP request is to use
HTTP basic authentication, which sends credentials in an HTTP header.

A server requests basic authentication by sending an HTTP 401 Unauthorized
response with an additional `WWW-Authenticate` header. The header has a value like
`Basic realm="Product catalog"`. This specifies the required authentication type
and names the protected resource.

~~~~~~
def readBasicAuthentication(headers: Headers):
    Option[Either[Result, (String, String)]] = {
  headers.get(Http.HeaderNames.AUTHORIZATION).map { header =>
    val BasicHeader = "Basic (.*)".r
    header match {
      case BasicHeader(base64) => {
        try {
          import org.apache.commons.codec.binary.base64
          val decodedBytes = base64.decodeBase64(base64.getBytes)
          val credentials = new String(decodedBytes).split(":", 2)
          credentials match {
            case Array(username, password) =>
              Right(username -> password)
            case _ => Left("Invalid basic authentication")
          }
        }
      }
      case _ => Left(BadRequest("Bad Authorization header"))
    }
  }
}
val maybeCredentials = readQueryString(request) orElse
  readBasicAuthentication(request.headers)
// curl --include --user peter:secret http://localhost:9000/
~~~~~~

## Other authentication methods ##

Web services often use one of two alternatives:
* Token-based authentication—Providing a signed API key that clients can send with
requests, either in a custom HTTP header or query string parameter
* Session-based authentication—Using one method to authenticate, and then
providing a session identifier that clients can send, either in an HTTP cookie or an
HTTP header

