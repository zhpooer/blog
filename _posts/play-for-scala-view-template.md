title: Play for Scala-View Template
date: 2014-08-12 11:36:59
tags:
- scala
- play
---

# The why of a template engine #

Templates allow you to reuse pieces of your HTML when you need them, such as a
header and a footer section that are the same or similar on every page. 

Another reason to use templates is because they help you to separate business logic
from presentation logic; separating these two concerns has several advantages.

# Type safety of a template engine #

Play Scala templates are HTML files containing snippets of Scala code,
which are compiled into plain Scala code before your application is started. 

As an example, we’ll build a catalog application. The main page will be a list of all
the articles in the catalog, and every article on this page will have a hyperlink to a
details page for that article, where more information about that article is shown. 

## A type-safe template engine ##

~~~~~~
@(articles: Seq[models.Article])
<hi>Articles</hi>
<ul>
@for(article <- articles) {
  <li>
    @article.name -
    <a href="@controllers.routes.Aritcles.show(article.id)">
      details
    </a>
  </li>
}
</ul>
~~~~~~

~~~~~~
object Articles extends Controller {
  def index = Action {
    val articles = Article.findAll()
    Ok(views.html.articles.index(articles))
  }
  def show(id:Long) = Action {
    Articles.findById(id) match {
      case None => NotFound
      case Some(article) => Ok(views.html.articles.show(article))
    }
  }
}
~~~~~~

Play’s type-safe template engine will help you build a more robust application. Both
your IDE and Play itself will warn you when a refactoring causes type errors, even
before you render the template.

# Template basics and common structures #
In Scala templates, the `@` character marks the start of a Scala expression.

## 特殊字符 @ ##
~~~~~~
Hello @name!
Your age is @user.age.
Next year, your age will be @(user.age + 1)

@* 这样是错误的 *@
Next year, your age will be @user.age + 1

Next year, your age will be
@{val ageNextYear = user.age + 1; ageNextYear}

@* 输出 @ 符号 *@
username@@example.com

@* 这是注释 *@
~~~~~~

## 表达式 ##


Play also imports various things into the scope of your templates. The following are
automatically imported by Play:

* `models._`
* `controllers._`
* `play.api.i18n._`
* `play.api.mvc._`, makes MVC components available.
* `play.api.data._`, contains tools for dealing with forms and validation. 
* `views.%format%._`,  is replaced by the template format that you’re using.

## Displaying collections ##

~~~~~~
<ul>
@articles.map { article =>
  <li>@article.name</li>
}
</ul>

@* The template compiler automatically adds the yield keyword *@
<ul>
@for(article <- articles) {
  <li>@article.name</li>
}
</ul>

@for(article <- articles) {
  Article name: @article.name
}

~~~~~~


If you’re interested in the details of the template engine,
you can take a look at the file `ScalaTemplateCompiler.scala` in the Play
framework source. 

### ADDING THE INDEX OF THE ELEMENT ###

~~~~~~
<ul>
@for((article, index) <- articles.zipWithIndex) {
  <li>Best seller number @(index + 1): @article.name</li>
}
</ul>

<ul>
@for((article, index) <- articles.zipWithIndex) {
  <li class="@if(index == 0){first}
    @if(index == articles.length - 1){last}">
  Best seller number @(index + 1): @article.name</li>
}
</ul>

~~~~~~

## Security and escaping ##

An application developer must always keep security in mind, and when dealing with
templates, avoiding cross-site scripting vulnerabilities is especially relevant. 

### CROSS-SITE SCRIPTING VULNERABILITIES ###

HTML injection could lead to minor annoyances, like broken markup and invalid
HTML documents, but much more serious problems arise when a malicious user
inserts scripts in your web page. These scripts could, for example, steal other visitors’
cookies when they use your application, and send these cookies to a server under the
attacker’s control.

### ESCAPING ###

Everything that you write literally in a template is considered HTML by Play, and is
output unescaped. This HTML is always written by the template author, so it’s consid-
ered safe. 

But all Scala expressions are escaped.

~~~~~~
@* embedCode 在这里被当成 没有特殊字符的文本, 不会被转义 *@
@article.embeddedVideo.map { embedCode =>
  <h3>Product video</h3>
  @Html(embedCode)
}

~~~~~~

## Using plain Scala ##
As shown earlier, you can use plain Scala if you
create a block with `@()` or `@{}`.

By default, the output is escaped. If you want to prevent this, wrap the result in an Html.

Any scala.xml.NodeSeq is also rendered unescaped, so you
can use the following code:
~~~~~~
@{
  <b>hello</b>
}
~~~~~~

只调用 `article.countReview()` 一次, 值映射到 total
~~~~~~
@defining(article.countReview()) { total =>
  <h3>This article has been reviewed @total times</h3>
  <p>@(article.countPositiveReviews()) out of these
  @total reviews were positive!</p>
}
~~~~~~

# Structuring pages: template composition #

~~~~~~
def catalog() = Action {
  val products = ProductDAO.list
  Ok(views.html.shop.catalog(products))
}
~~~~~~

`views/navigation.scala.html`
~~~~~~
@()
<div id="navigation">
  <ul>
    <li><a href="@routes.Application.home">Home</a></li>
    <li><a href="@routes.Shop.catalog">Catalog</a></li>
    <li><a href="@routes.Application.contact">Contact</a></li>
  </ul>
</div>
~~~~~~

`views/catalog.scala.html`

~~~~~~
<!-- other code -->
<!-- 导入 navigation 模板 -->
  @navigation()
<!-- other code -->
~~~~~~

## Layouts ##
`app/views/main.scala.html`, with a single parameter named content of type `Html`
~~~~~~
@(content: Html)
<!DOCTYPE html>
<html>
  <head>
    <title>paperclips.example.com</title>
    <link href="@routes.Assets.at("stylesheets/main.css")"
          rel="stylesheet">
  </head>
  <body>
    <div id="header">
      <h1>Products</h1>
    </div>
    @navigation
    <div id="content">
      @content
    </div>
    <footer>
      <p>Copyright paperclips.example.com</p>
    </footer>
  </body>
</html>
~~~~~~

可以这样使用
~~~~~~
@(products: Seq[Product])
@main {
  <h2>Products</h2>
  <ul class="products">
  @for(product <- products) {
    <li>
      <h3>@product.name</h3>
      <p class="description">@product.description</p>
    </li>
  }
  </ul>
}
~~~~~~

可以这样定义
~~~~~~
@* @(title="paperclips.example.com")(content: Html) 或这样*@
@(title: String)(content: Html)
<html>
  <head>
  <title>@title</title>
  <!-- any -->
~~~~~~
这样使用
~~~~~~
@main("Products") {
  // content here
}
~~~~~~

## Tags ##

`views/products/tags/productlist.scala.html`
~~~~~~
@(products: Seq[Product])
<ul class="products">
@for(product <- products) {
  <li>
    <h3>@product.name</h3>
    <p class="description">@product.description</p>
  </li>
}
</ul>
~~~~~~

`catalog.scala.html`
~~~~~~
@(products: Seq[Product])
@main {
  <h2>Products</h2>
  @views.html.products.tags.productlist(products)
}
~~~~~~

# Reducing repetition with implicit parameters #

Let’s continue with our web shop example. This time we’ll assume that we want to
maintain a shopping cart on the website, and in the top-right corner of every page we
want to show the number of items the visitor has in their shopping cart.

`main.scala.html`
~~~~~~
@(cart: Cart)(content: Html)
<html>
    <head></head>
    <body></body>
</html>
~~~~~~

Now that the main template needs a Cart parameter, we’ll have to pass one to it,
which means adapting our catalog template. But this template doesn’t have a refer-
ence to a Cart object, so it’ll need to take one as a parameter as well:

`cart.scala.html`
~~~~~~
@(products: Seq[Product], cart: Cart)
@main(cart) {
  <h2>Catalog</h2>
  @views.html.products.tags.productlist(products)
}
~~~~~~

We’ll also have to pass a Cart from the action:
~~~~~~
def catalog() = Action { request =>
  val products = ProductDAO.list
  Ok(views.html.shop.catalog(products, cart(request)))
}

def cart(request: Request) = {
  // Get cart from session
}
~~~~~~

可以这样:

We can use an implicit parameter to change the method signature of our catalog
template as follows:
~~~~~~
@(products: Seq[Product])(implicit cart: Cart)
~~~~~~

~~~~~~
def catalog() = Action { implicit request =>
  val products = ProductDAO.list
  Ok(views.html.shop.catalog(products))
}

implicit def cart(implicit request: RequestHeader) = {
  // Get cart from session
}
~~~~~~

We can make our newly created cart method reusable, by moving it into a trait:
~~~~~~
trait WithCart {
  implicit def cart(implicit request: RequestHeader) = {
    // Get cart from session
  }
}
~~~~~~

If you have an implicit Request in
scope in your controller, you also have an implicit RequestHeader, Session,
Flash, and Lang in scope, because the Controller trait defines
implicit conversions for these types.

# Using LESS and CoffeeScript: the asset pipeline #

LESS is a stylesheet language that’s
transformed to CSS by a LESS interpreter or compiler,
whereas CoffeeScript is a scripting language that’s transformed into
JavaScript by a CoffeeScript compiler.

## LESS ##

~~~~~~
.header {
  background-color: #0b5c20;
}
.footer {
  background-color:#0b5c20;
}
.footer a {
  font-weight: bold;
}
~~~~~~
LESS:
~~~~~~
@green: #0b5c20;
.header {
  background-color:@green;
}

.footer {
  background-color:@green;
  a {
    font-weight:bold;
  }
}
~~~~~~

## CoffeeScript ##

~~~~~~
math =
  root: Math.sqrt
  square: square
  cube: (x) -> x * square x
~~~~~~

## The asset pipeline ##

Play supports automatic build-time CoffeeScript and LESS compilation, and it
shows compilation errors in the familiar Play error page.

For example, if you place a CoffeeScript file in
`app/assets/javascripts/application.coffee`,
you can reference it from a template like this:
~~~~~~
<script src="@routes.Assets.at("javascripts/application.js")"></script>
~~~~~~

You can also use an automatically generated minified version of
your JavaScript file by changing `application.js` to `application.min.js` .


# Internationalization #

Internationalization is a refactoring to remove locale-specific code from
your application. Localization is making a locale-specific version of an application. In
an internationalized web application, this means having one or more selectable
locale-specific versions.

`application.conf`
~~~~~~
application.langs="en,en-US,nl"
~~~~~~

`conf/messages`
~~~~~~
welcome = Welcome!
users.login = Log in
shop.thanks = Thank you for your order
~~~~~~

Play provides an implicit conversion from a Request to a Lang, which is more useful.
~~~~~~
Messages("users.login")(Lang("en"))
~~~~~~

~~~~~~
@(implicit request: Request)
<h1>@Messages("welcome")</h1>
~~~~~~

~~~~~~
validation.required={0} is required
~~~~~~
You can substitute these by specifying more parameters to the call to Messages:
~~~~~~
Messages("validation.required", "email")
~~~~~~

~~~~~~
shop.basketcount=Your cart {0,choice,0#is empty|1#has one item
  |1< has {0} items}.
~~~~~~

~~~~~~
<p>@Messages("shop.basketcount", 0)</p>
<p>@Messages("shop.basketcount", 1)</p>
<p>@Messages("shop.basketcount", 10)</p>
<!-- 输出 -->
<!-- Your cart is empty. -->
<!-- Your cart has one item. -->
<!-- Your cart has 10 items. -->
~~~~~~

