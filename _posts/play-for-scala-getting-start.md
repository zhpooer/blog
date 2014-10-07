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

## Add an HTML page template ##

`app/views/hello.scala.html`

~~~~~~
@(name:String)
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello</title>
  </head>
  <body>
    <h1>Hello <em>@name</em></h1>
  </body>
</html>

~~~~~~

~~~~~~
def hello(name: String) = Action {
  Ok(views.html.hello(name))
}
~~~~~~

`sbt console`
~~~~~~
views.html.hello.render("Play!")
~~~~~~

# Your first Play application #

We’ll start with a simple list of products, each of which has a name
and a description.
This is a prototype, with a small number of products, so there isn’t
any functionality for filtering, sorting, or paging the list.

    Paperclips Large
      Large Plain Pack of 1000
    Zebra Paperclips
      Zebra Length 28mm Assorted 150 Pack

To make the product list page work, we’ll need a combination of the following:
* A view template—A template that generates HTML
* A controller action—A Scala function that renders the view
* Route configuration—Configuration to map the URL to the action
* The model—Scala code that defines the product structure, and some test data

## Getting started ##

~~~~~~
play new products
rm products/public/images/favicon.png
rm products/public/javascripts/jquery-1.7.1.min.js
~~~~~~
copying `docs/assets/css/bootstrap.css` to our
application’s `public/stylesheets` directory.
Also copy `glyphicons-halflings-white.png` and `glyphicons-halflings.png` to
`public/img`

`main.css`
~~~~~~
body { color:black; }
body, p, label { font-size:15px; }
.label { font-size:13px; line-height:16px; }
.alert-info { border-color:transparent; background-color:#3A87AD;
  color:white; font-weight:bold; }
div.screenshot { width: 800px; margin:20px; background-color:#D0E7EF; }
.navbar-fixed-top .navbar-inner { padding-left:20px; }
.navbar .nav > li > a { color:#bbb; }
.screenshot > .container { width: 760px; padding: 20px; }
.navbar-fixed-top, .navbar-fixed-bottom { position:relative; }
h1 { font-size:125%; }
table { border-collapse: collapse; width:100%; }
th, td { text-align:left; padding: 0.3em 0;
  border-bottom: 1px solid white; }
tr.odd td { }
form { float:left; margin-right: 1em; }
legend { border: none; }
fieldset > div { margin: 12px 0; }
.help-block { display: inline; vertical-align: middle; }
.error .help-block { display: none; }
.error .help-inline { padding-left: 9px; color: #B94A48; }
footer { clear: both; text-align: right; }
dl.products { margin-top: 0; }
dt { clear: right; }
.barcode { float:right; margin-bottom: 10px; border: 4px solid white; }
~~~~~~

`conf/application.conf`
~~~~~~
#  Be sure to use a different secret for your production
#environment and never check that into your source code repository.
application.secret="Wd5HkNoRKdJP[kZJ@OV;HGa^<4tDvgSfqn2PJeJnx4l0s77NTl"
# application.langs="en,es,fr,nl"
application.langs="en"
~~~~~~
If you later want to copy entries from the default `application.conf` file,
you can find it in `$PLAY_HOME/framework/skeletons/scalaskel/conf/.`

We’ll define in a messages file for each language:
* `conf/messages—Default` messages for all languages, for messages not localized
for a particular language
* `conf/messages.es—Spanish` (which is called Español in Spanish)
* `conf/messages.fr—French` (Français in French)
* `conf/messages.nl—Dutch` (Nederlands in Dutch)

~~~~~~
# conf/messages
application.name = Product catalog
# conf/messages.es
application.name = Catálogo de productos
~~~~~~

## Adding the model ##
We need to include three things in the example application’s model,
* A model class—The definition of the product and its attributes
* A data access object (DAO)—Code that provides access to product data
* Test data—A set of product objects

`app/models/Product.scala`
~~~~~~
package models
case class Product(ean: Long, name: String, description: String)

object Product {
  var products = Set(Product(5010255079763L, "Paperclips Large","Large Plain Pack of 1000"),
    Product(5018206244666L, "Giant Paperclips","Giant Plain 51mm 100 pack"),
    Product(5018306332812L, "Paperclip Giant Plain","Giant Plain Pack of 10000"),
    Product(5018306312913L, "No Tear Paper Clip","No Tear Extra Large Pack of 1000"),
    Product(5018206244611L, "Zebra Paperclips","Zebra Length 28mm Assorted 150 Pack")
  )
  def findAll = products.toList.sortBy(_.ean)
}
~~~~~~

## Product list page ##

`app/views/products/list.scala.html`
~~~~~~
<!-- The implicit Lang parameter is used for the localized message -->
<!-- lookup performed by the Messages object. -->
@(products: List[Product])(implicit lang: Lang)
@main(Messages("application.name")) {
  <dl class="products">
  @for(product <- products) {
    <dt>@product.name</dt>
    <dd>@product.description</dd>
  }
  </dl>
}
~~~~~~

### Layout template ##
`app/views/main.scala.html`

~~~~~~
@(title: String)(content: Html)(implicit lang: Lang)

<!DOCTYPE html>
<html>
  <head>
    <title>@title</title>
    <link rel="stylesheet" type="text/css" media="screen"
      href='@routes.Assets.at("stylesheets/bootstrap.css")'>
    <link rel="stylesheet" media="screen"
      href="@routes.Assets.at("stylesheets/main.css")">
  </head>
  <body>
    <div class="screenshot">
      <div class="navbar navbar-fixed-top">
        <div class="navbar-inner">
          <div class="container">
            <a class="brand" href="@routes.Application.index()">
              @Messages("application.name")
            </a>
          </div>
        </div>
      </div>
      <div class="container">@content</div>
    </div>
  </body>
</html>
~~~~~~

### Controller action method ##

`app/controllers/Products.scala`
~~~~~~
package controllers

import play.api.mvc.{Action, Controller}
import models.Product

object Products extends Controller {
  def list = Action { implicit request =>
    val products = Product.findAll
    Ok(views.html.products.list(products))
  }
}

~~~~~~

### Adding a routes configuration ##

`conf/routes`

~~~~~~
GET / controllers.Application.index

GET /products controllers.Products.list

GET /assets/*file controllers.Assets.at(path="/public", file)
~~~~~~

### Replacing the welcome page with a redirect ##

`app/controllers/Application.scala`
~~~~~~
package controllers
import play.api.mvc.{Action, Controller}

object Application extends Controller {
  def index = Action {
    Redirect(routes.Products.list())
  }
}
~~~~~~

Delete `app/views/index.scala.html` template.

### Checking the language localizations ##

`app/views/debug.scala.html`

~~~~~~
@()(implicit lang: Lang)

@import play.api.Play.current

<footer>
  lang = @lang.code,
  user = @current.configuration.getString("environment.user"),
  date = @(new java.util.Date().format("yyyy-MM-dd HH:mm"))
</footer>
~~~~~~

`conf/application.conf`
~~~~~~
# USER is an environment variable
environment.user=${USER}
~~~~~~

`app/views/main.scala.html`
~~~~~~
<div class="container">
@content
@debug()
</div>
~~~~~~

## Details page ##

The page’s URL, for example `/products/5010255079763`, includes the EAN code,
which is also used to generate a bar code image
* A new finder method—To fetch one specific product
* A view template—To show this details page
* An HTTP routing configuration—For a URL with a parameter
* A bar code image—To display on the page

### Model finder method ###
`app/models/Product.scala`
~~~~~~
object Product {
  def findByEan(ean: Long) = products.find(_.ean == ean)
}
~~~~~~

### Details page template ###
`app/views/products/details.scala.html`
~~~~~~
@(product: Product)(implicit lang: Lang)
@main(Messages("products.details", product.name)) {
  <h2>
    <!-- 生成横条码  -->
    @tags.barcode(product.ean)
    @Messages("products.details", product.name)
  </h2>

  <dl class="dl-horizontal">
    <dt>@Messages("ean"):</dt>
    <dd>@product.ean</dd>
    <dt>@Messages("name"):</dt>
    <dd>@product.name</dd>
    <dt>@Messages("description"):</dt>
    <dd>@product.description</dd>
  </dl>

}
~~~~~~
`app/views/tags/barcode.scala.html`
~~~~~~
@(ean: Long)
<img class="barcode" alt="@ean" src="@routes.Barcodes.barcode(ean)">
~~~~~~

### Additional message localizations ###

`conf/messages`
~~~~~~
ean = EAN
name = Name
description = Description

products.details = Product: {0}
~~~~~~
`conf/messages.es`
~~~~~~
ean = EAN
name = Nombre
description = Descripción

products.details = Producto: {0}
~~~~~~

### Adding a parameter to a controller action ###

`app/controllers/Products.scala`
~~~~~~
def show(ean: Long) = Action { implicit request =>
  Product.findByEan(ean).map { product =>
    Ok(views.html.products.details(product))
  }.getOrElse(NotFound)
}
 
~~~~~~

### Adding a parameter to a route ###

~~~~~~
GET /products/:ean controllers.Products.show(ean: Long)
~~~~~~

### Generating a bar code image ###

`project/Build.scala`
~~~~~~
val appDependencies = Seq(
  "net.sf.barcode4j" % "barcode4j" % "2.0"
)
~~~~~~
`app/controllers/Barcodes.scala`
~~~~~~
package controllers

import play.api.mvc.{Action, Controller}

object Barcodes extends Controller {
  val ImageResolution = 144
  // Action that returns PNG response
  def barcode(ean: Long) = Action {
    import java.lang.IllegalArgumentException
    val MimeType = "image/png"
    try {
      val imageData = ean13BarCode(ean, MimeType)
      Ok(imageData).as(MimeType)
    } catch {
      case e: IllegalArgumentException =>
        BadRequest("Couldn’t generate bar code. Error: " + e.getMessage)
    }
  }
  def ean13BarCode(ean: Long, mimeType: String): Array[Byte] = {
    import java.io.ByteArrayOutputStream
    import java.awt.image.BufferedImage
    import org.krysalis.barcode4j.output.bitmap.BitmapCanvasProvider
    import org.krysalis.barcode4j.impl.upcean.EAN13Bean
    val output: ByteArrayOutputStream = new ByteArrayOutputStream
    val canvas: BitmapCanvasProvider =
      new BitmapCanvasProvider(output, mimeType, ImageResolution,
        BufferedImage.TYPE_BYTE_BINARY, false, 0)
    val barcode = new EAN13Bean()
    barcode.generateBarcode(canvas, String valueOf ean)
    canvas.finish
    output.toByteArray
  }
}
~~~~~~
`conf/routes`
~~~~~~
GET /barcode/:ean controllers.Barcodes.barcode(ean: Long)
~~~~~~

## Adding a new product ##

### Additional message localizations ###
`conf/messages`
~~~~~~
products.form = Product details
products.new = (new)
products.new.command = New

products.new.submit = Add
products.new.success = Successfully added product {0}.

validation.errors = Please correct the errors in the form.
validation.ean.duplicate = A product with this EAN code already exists
~~~~~~

### Form object ###

`app/controllers/Products.scala`

~~~~~~
import play.api.data.Form
import play.api.data.Forms.{mapping, longNumber, nonEmptyText}
import play.api.i18n.Messages

private val productForm: Form[Product] = Form(
  mapping(
    "ean" -> longNumber.verifying(
      "validation.ean.duplicate", Product.findByEan(_).isEmpty),
    "name" -> nonEmptyText,
    "description" -> nonEmptyTest
  )(Product.apply)(Product.unapply)
)
~~~~~~

### Form template ###

`app/views/main.scala.html`

~~~~~~
<div class="container">
  <a class="brand" href="@routes.Application.index()">
    @Messages("application.name")
  </a>
  <ul class="nav">
    <li class="divider-vertical"></li>
    <li class="active">
      <a href="@routes.Products.list()">
        @Messages("products.list.navigation")
      </a>
    </li>
    <li class="active">
      <a href="@routes.Products.newProduct()">
        <i class="icon-plus icon-white"></i>
        @Messages("products.new.command")
      </a>
    </li>
    <li class="divider-vertical"></li>
  </ul>
</div>

<div class="container">
  @if(flash.get("success").isDefined){
    <div class="alert alert-success">
      @flash.get("success")
    </div>
  }
  @if(flash.get("error").isDefined){
    <div class="alert alert-error">
      @flash.get("error")
    </div>
  }
  @content
  @debug()
</div>
~~~~~~
`app/views/products/list.scala.html`

~~~~~~
<p>
  <a href="@controllers.routes.Products.newProduct()" class="btn">
  <i class="icon-plus"></i> @Messages("products.new.command")</a>
</p>
~~~~~~

`app/views/products/editProduct.scala.html`
~~~~~~
@(productForm: Form[Product])(implicit flash: Flash, lang: Lang)

<!-- Twitter Bootstrap helpers -->
@import helper._
@import helper.twitterBootstrap._

@main(Messages("products.form")) {
  <h2>@Messages("products.form")</h2>

  @helper.form(action = routes.Products.save()) {
    <fieldset>
      <legend>
        @Messages("products.details", Messages("products.new"))
      </legend>
      @helper.inputText(productForm("ean"))
      @helper.inputText(productForm("name"))
      @helper.textarea(productForm("description"))
    </fieldset>
    <p><input type="submit" class="btn primary" value='@Messages("products.new.submit")'></p>
  }
}
~~~~~~

### Saving the new product ###

`app/models/Product.scala`
~~~~~~
object Product {
  def add(product: Product) {
    products = products + product
  }
}
~~~~~~

### Validating the user input ###

`app/controllers/Products.scala`
~~~~~~
import play.api.mvc.Flash

def save = Action {implicit request =>
  val newProductForm = productForm.bindFromRequest()
  newProductForm.fold(
    hasErrors = { form =>
      Redirect(route.Products.newProduct()).
        flashing(Flash(form.data) +
          ("error" -> Messages("validation.errors")))
    },
    success = { newProduct =>
      Product.add(newProduct)
      Redirect(routes.Products.show(newProduct.ean)).
        flashing("success" -> message)
    }
  )
}

def newProduct = Action { implicit request =>
  val form = if (flash.get("error").isDefined)
    productForm.bind(flash.data)
  else
    productForm
    
  Ok(views.html.products.editProduct(form))
}
~~~~~~

### Adding the routes for saving products ###

~~~~~~
POST /products controllers.Products.save

GET /products/new controllers.Products.newProduct
~~~~~~
