title: Play for Scala-Input with forms
date: 2014-08-12 16:22:42
tags:
- scala
- play
---

# Forms—the concept #

Play provides the so-called forms API. The term form isn’t just about HTML forms in
a Play application; it’s a more general concept. The forms API helps you to validate data,
manage validation errors, and map this data to richer data structures.

~~~~~~
case class User(
  username : String,
  realname : Option[String],
  email    : String )

val userForm = Form(
  mapping(
    "username" -> nonEmptyText(8),
    "realname" -> optional(text),
    "email" -> email)(User.apply)(User.unapply))

def createUser() = Action { implicit request =>
  userForm.bindFromRequest.fold(
    formWithErrors => BadRequest,
    user => Ok("User OK!"))
}
~~~~~~

# Forms basics #

Play’s forms are powerful, but they’re built on a few simple ideas.

## Mappings ##

A Mapping is an object that can construct something from the data in an HTTP
request. This process is called binding.

So a Mapping[User] can construct a User instance, and
a Mapping[Int] can create an Int.

 If you submit an HTML form with an input tag
`<input type="text" name="age">`, a `Mapping[Int]` can convert that age value,
which is submitted as a string, into a Scala Int.

The data from the HTTP request is transformed into a `Map[String, String]`, and
this is what the Mapping operates on.

But a Mapping can not only construct an object from a map of data;
it can also do the reverse operation of deconstructing an object
into a map of data.

 A mapping is an object of type `Mapping[T]` that can take a
`Map[String, String]`, and use it to construct an object of type T ,

For example, `Forms.number` is a mapping of type `Mapping[Int]`,
whereas `Forms.text` is a mapping of type
`Mapping[String]`. There’s also `Forms.email`,
which is also of type `Mapping[String]`,
but it also contains a constraint that the
string must look like an email address.

## Creating a form ##
~~~~~~
val data = Map(
  "name" -> "Box of paper clips",
  "ean" -> "1234567890123",
  "pieces" -> "300"
)

// The type of mapping is
// play.api.data.Mapping[(String, String, Int)]
// indicates the type of objects that this mapping can construct.
val mapping = Forms.tuple(
  "name" -> Forms.text,
  "ean" -> Forms.text,
  "pieces" -> Forms.number)

// This form is of type Form[(String, String, Int)].
// Form has a single type parameter, and it has the same meaning.
// But a form not only wraps a Mapping, it can also contain data.
val productForm = Form(mapping)
~~~~~~

You can use the following Play-provided
basic mappings to start composing more complex mappings:

* `boolean: Mapping[Boolean]`
* `checked(msg: String): Mapping[Boolean]`
* `date: Mapping[Date]`
* `email: Mapping[String]`
* `ignored[A](value: A): Mapping[A]`
* `longNumber: Mapping[Long]`
* `nonEmptyText: Mapping[String]`
* `number: Mapping[Int]`
* `sqlDate: Mapping[java.sql.Date]`

## Processing data with a form ##

The process of putting your data in the form is called binding,
and we use the bind method to do it:
~~~~~~
//  it returns a new Form—a copy of the original form populated with the data.
val processedForm = productForm.bind(data)

if(!processedForm.hasErrors) {
  val productTuple = processedForm.get // Do something with the product
} else {
  val errors = processedForm.getErrors // Do something with the errors
}
~~~~~~
`Form.fold` takes two parameters, where the first is a function that accepts the
“failure” result, and the second accepts the “success” result as the single parameter.

~~~~~~
val processedForm = productForm.bind(data)
processedForm.fold (
  formWithErrors => BadRequest,
  productTuple => {
    Ok(views.html.product.show(product))
  }
)
~~~~~~

### Either ###
~~~~~~
def getProduct(): Either[String, Product] = {
  if(validation.hasError) {
    Left(validation.error)
  } else {
    Right(Product())
  }
}
def showProduct() = Action {
  getProduct().fold(
    failureReason => InternalServerError(failureReason),
    product => Ok(views.html.product.show(product))
  )
}
~~~~~~

## Object mapping ##

To do so, we’ll have to provide the mapping with a function to construct the value.

~~~~~~
case class Product(
  name: String,
  ean: String,
  pieces: Int)

import play.api.data.Forms._

// This makes the type of this mapping Mapping[Product].
val productMapping = mapping(
  "name" -> text,
  "ean" -> text,
  "pieces" -> number)(Product.apply)(Product.unapply)

// Using our Mapping[Product], we can now easily create a Form[Product]:
val productForm = Form(productMapping)

productForm.bind(data).fold(
  formWithErrors => ...,
  product => 
)
~~~~~~

## Mapping HTTP request data ##

~~~~~~
def processForm() = Action { request =>
  productForm.bindFromRequest()(request).fold(
  ...
  )
}

// 加入 imlicit
def processForm() = Action { implicit request =>
  productForm.bindFromRequest().fold(
    ...
  )
}

~~~~~~

Browsers submit HTTP bodies with either an `application/x-www-form-urlencoded`
or a `multipart/form-data` content type, depending on the form, and it’s
also common to send JSON over the wire. The bindFromRequest method uses the
Content-Type header to determine a suitable decoder for the body.

# Creating and processing HTML forms #

Play also provides helpers that generate forms and take the
tedium out of showing validation and error messages in the appropriate places.

## Writing HTML forms manually ##

model class:
~~~~~~
case class Product(
  ean: Long,
  name: String,
  description: String,
  pieces: Int,
  active: Boolean)

val productForm = Form(mapping(
  "ean" -> longNumber,
  "name" -> nonEmptyText,
  "description" -> text,
  "pieces" -> number,
  "active" -> boolean)(Product.apply)(Product.unapply))

def create() = Action { implicit request =>
  productForm.bindFromRequest.fold(
    formWithErrors => BadRequest("Oh noes, invalid submission!"),
    value => Ok("created: " + value)
  )
}
~~~~~~

~~~~~~
@()
@main("Product Form") {
  <form action="@routes.Products.create()" method="post">
    <div>
      <label for="name">Product name</label>
      <input type="text" name="name" id="name">
    </div>
    <div>
      <label for="description">Description</label>
      <textarea id="description" name="description"></textarea>
    </div>
    <div>
      <label for="ean">EAN Code</label>
      <input type="text" name="ean" id="ean">
    </div>
    <div>
      <label for="pieces">Pieces</label>
      <input type="text" name="pieces" id="pieces">
    </div>
    <div>
      <label for="active">Active</label>
      <input type="checkbox" name="active" value="true">
    </div>
    <div class="buttons">
      <button type="submit">Create Product</button>
    </div>
  </form>
}
~~~~~~

## Generating HTML forms ##

Play provides helpers, template snippets that can render a form field for you, including
extra information like an indication when the value is required and an error message
if the field has an invalid value. The helpers are in the `views.template` package.

~~~~~~
@(productForm: Form[Product])

@main("Product Form") {
  @helper.form(action = routes.GeneratedForm.create) {
    @helper.inputText(productForm("name"))
    @helper.textarea(productForm("description"))
    @helper.inputText(productForm("ean"))
    @helper.inputText(productForm("pieces"))
    @helper.checkbox(productForm("active"))
    <div class="form-actions">
      <button type="submit">Create Product</button>
    </div>
  }
}
~~~~~~

~~~~~~
def createForm() = Action {
  Ok(views.html.products.form(productForm))
}
~~~~~~

## Input helpers ##

Play ships predefined helpers for the most common input types:
* inputDate—Generates an input tag with type date .
* inputPassword—Generates an input tag with type password.
* inputFile—Generates an input tag with type file .
* inputText—Generates an input tag with type text .
* select—Generates a select tag.
* inputRadioGroup—Generates a set of input tags with type radio.
* checkbox—Generates an input tag with type checkbox.
* textarea—Generates a textarea element.
* input—Creates a custom input.

~~~~~~
@*  notation '_class creates a Scala Symbol named _class *@
@helper.inputText(productForm("name"), '_class -> "important", 'size -> 40)
~~~~~~

These are the extra symbols with underscores that you can use:
* `_label`—Use to set a custom label
* `_id`—Use to set the id attribute of the dl element
* `_class`—Use to set the class attribute of the dl element
* `_help`—Use to show custom help text
* `_showConstraints`—Set to false to hide the constraints on this field
* `_error`—Set to a `Some[FormError]` instance to show a custom error
* `_showErrors`—Set to false to hide the errors on this field

## Customizing generated HTML ##

Play allows you to customize the generated HTML in two ways. First, you
can customize which input element is generated, in case you need some special input
type. Second, you can customize the HTML elements around that input element.

Suppose we want to create an input with type datetime.

~~~~~~
@* the first is the Field that we want to create the input for *@
@helper.input(myForm("mydatetime")) { (id, name, value, args) =>
  @* a type (String, String, Option[String], Map[Symbol,Any]) => Html *@
  <input type="datetime" name="@name" id="@id" value="@value" @toHtmlArgs(args)>
}
~~~~~~

We use the toHtmlArgs method from the `play.api.templates.PlayMagic`
object to construct additional attributes from the args map.

They have an additional parameter list that takes an implicit `FieldConstructor` and a `Lang`.

`FieldConstructor` is a trait with a single apply method that takes a
`FieldElements` object and returns Html. Play provides a `defaultFieldConstructor`
that generates the HTML we saw earlier, but you can implement
your own `FieldConstructor` if you want different HTML.

~~~~~~
@(elements: views.html.helper.FieldElements)

@import play.api.i18n._
@import views.html.helper._

<div class="control-group @elements.args.get('_class)
  @if(elements.hasErrors) {error}"
  id="@elements.args.get('_id).getOrElse(elements.id + "_field")" >
    <label class="control-label" for="@elements.id">
      @elements.label(elements.lang)
    </label>
    <div class="controls">
      @elements.input
      <span class="help-inline">
        @if(elements.errors(elements.lang).nonEmpty) {
          @elements.errors(elements.lang).mkString(", ")
        } else {
          @elements.infos(elements.lang).mkString(", ")
        }
      </span>
    </div>
</div>

~~~~~~

~~~~~~
package views.html.helper

package object bootstrap {
implicit val fieldConstructor = new FieldConstructor {
  def apply(elements: FieldElements) =
    bootstrap.bootstrapFieldConstructor(elements)
  }
}
~~~~~~

~~~~~~
@(productForm: Form[Product])
@import views.html.helper.bootstrap._
@main("Product Form") {
  @helper.form(action = routes.GeneratedForm.create) {
    @helper.inputText(productForm("name"))
    @helper.textarea(productForm("description"))
    @helper.inputText(productForm("ean"))
    @helper.inputText(productForm("pieces"))
    @helper.checkbox(productForm("active"))
    <div class="form-actions">
      <button type="submit">Create Product</button>
    </div>
  }
}
~~~~~~

# Validation and advanced mappings #

Additionally, we’ll see how we can create our own mappings,
for when we want to bind things that don’t have a predefined mapping.

## Basic validation ##

Mappings contain a collection of constraints, and when a value is bound, it’s checked
against each of the constraints.

A `Mapping[T]` has the method `verifying(constraints: Constraint[T]*)`, which
copies the mapping and adds the constraints. Play provides a small number of
constraints on the `play.api.data.validation.Constraints` object:

* `min(maxValue: Int): Constraint[Int]`—A minimum value for an Int mapping
* `max(maxValue: Int): Constraint[Int]`—A maximum value for an Int mapping
* `minLength(length: Int): Constraint[String]`—A minimum length for a String mapping
* `maxLength(length: Int): Constraint[String]`—A maximum length for a String mapping
* `nonEmpty`: Constraint[String]—Requires a not-empty string
* `pattern(regex: Regex, name: String, error: String): Constraint[String]`—
A constraint that uses a regular expression to validate a String



These are also the constraints that Play uses when you utilize one of the mappings
with built-in validations, like `nonEmptyText`.

~~~~~~
"name" -> text.verifying(Constraints.nonEmpty)
~~~~~~

## Custom validation ##

In our product form, we’d like to check whether a product with the same EAN code
already exists in our database. 

~~~~~~
def eanExists(ean: Long) = Product.findByEan(ean).isEmpty

// We can then use verifying to add it to our mapping
"ean" -> longNumber.verifying(eanExists(_))

// add the validation massage
"ean" -> longNumber.verifying("This product already exists.", Product.findByEan(_).isEmpty)
~~~~~~

## Validating multiple fields ##

In our product form, we might want to allow people to add new products to the
database without a description, but not to make it active if there’s no description.

~~~~~~
val productForm = Form(mapping(
  "ean" -> longNumber.verifying("This product already exists!",
    Product.findByEan(_).isEmpty),
  "name" -> nonEmptyText,
  "description" -> text,
  "pieces" -> number,
  "active" -> boolean)(Product.apply)(Product.unapply).verifying(
    "Product can not be active if the description is empty",
    product =>
      !product.active || product.description.nonEmpty))
~~~~~~

If this top-level mapping causes an error, it’s called the global error,
which you can retrieve with the globalError method on Form.

~~~~~~
@productForm.globalError.map { error =>
  <span class="error">@error.message</span>
}
~~~~~~

## Optional mapings ##

~~~~~~
case class Person(name: String, age: Option[Int])

val personMapping = mapping(
  "name" -> nonEmptyText,
  "age"  -> optional(number)
)(Person.apply)(Person.unapply)
~~~~~~

## Repeated mappings ##

~~~~~~
<input type="text" name="tags[0]">
<input type="text" name="tags[1]">
<input type="text" name="tags[2]">
~~~~~~

~~~~~~
@helper.repeat(form("tags"), min = 3) { tagField =>
  @helper.inputText(tagField, '_label -> "Tag")
}
~~~~~~

~~~~~~
"tags" -> list(text)
~~~~~~

## Nested mappings ##

~~~~~~
val contactsForm = Form(tuple(
  "main_contact_name" -> text,
  "main_contact_email" -> email,
  "technical_contact_name" -> text,
  "technical_contact_email" -> email,
  "administrative_contact_name" -> text,
  "administrative_contact_email" -> email))

// same as
val contactMapping = tuple(
  "name" -> text,
  "email" -> email)
  
val contactsForm = Form(tuple(
  "main_contact" -> contactMapping,
  "technical_contact" -> contactMapping,
  "administrative_contact" -> contactMapping))
~~~~~~

~~~~~~
@helper.inputText(form("main_contact.name"))
@helper.inputText(form("main_contact.email"))
~~~~~~

Also like this

~~~~~~
val appointmentMapping = tuple(
  "location" -> text,
  "start" -> tuple(  // Field name start.date
    "date" -> date,
    "time" -> text),
  "attendees" -> list(mapping(
    "name" -> text,
    "email" -> email)(Person.apply)(Person.unapply))) // attendees[0].name>
~~~~~~

## Custom mappings ##

For example, we might have a date picker in our HTML form that we want to bind
to a Joda Time LocalDate, which is basically a date without time zone information.

We can create a `Mapping[LocalDate]` by transforming a Mapping[String] as
follows:
~~~~~~
val localDateMapping = text.transform(
  (dateString: String) =>
    LocalDate.parse(dateString),
  (localDate: LocalDate) =>
    localDate.toString)
~~~~~~

The transform method uses these to transform a `Mapping[String]` into a
`Mapping[LocalDate]`.

The transform method is therefore best used for transformations that are
guaranteed to work. When that’s not the case, you can use the second,
more powerful method of creating your own Mapping,
which is also how Play’s built-in mappings are created.

~~~~~~
implicit val localDateFormatter = new Formatter[LocalDate] {
  def bind(key:String, data: Map[String, String]) =
    data.get(key) map { value =>
      Try {
        Right(LocalDate.parse(value))
      } getOrElse Left(Seq(FormError(key, "error.date", Nil)))
    } getOrElse Left(Seq(FormError(key, "error.required", Nil)))
  def unbind(key: String, id: LocalDate) = Map(key -> id.toString)
  
  override val format = Some(("date.format", Nil))
}

//  we can easily construct a Mapping[LocalDate] using the Forms.of method
val localDateMapping = Forms.of(localDateFormatter)
~~~~~~

`conf/messages`
~~~~~~
date.format=Date (YYYY-MM-DD)
error.date=Date formatted as YYYY-MM-DD expected
~~~~~~

Because the parameter of the of method is implicit, and we’ve declared our `localDateFormatter`
as implicit as well, we can leave it off, but we do have to specify the
type parameter then. Additionally, if we have `Forms._` imported, we can write this:

~~~~~~
val localDateMapping = of[LocalDate]

// The single method is identical to the tuple method,
// except it’s the one you need to use if you have only a single field.
val localDateForm = Form(single(
  "introductionDate" -> localDateMapping
))
~~~~~~

~~~~~~
@helper.inputText(productForm("introductionDate"), '_label -> "Introduction Date")
~~~~~~

## Dealing with file uploads ##

~~~~~~
<form action="@routes.FileUpload.upload" method="post"
      enctype="multipart/form-data">
  <input type="file" name="image">
  <input type="submit">
</form>

~~~~~~

~~~~~~
//  request.body is of type MultipartFormData[TemporaryFile]
def upload() = Action(parse.multipartFormData) { request =>
  request.body.file("image").map { file =>
    // FilePart[TemporaryFile], which has a ref property
    // This TemporaryFile deletes its underlying file when it’s garbage collected
    file.ref.moveTo(new File("/tmp/image"))
    Ok("Retrieved file %s" format file.filename)
  }.getOrElse(BadRequest("File missing!"))
}

~~~~~~

Even though you don’t use forms for processing files, you can still use them for
generating inputs and reporting validation errors.

~~~~~~
def upload() = Action (parse.MultipartFormData) { implicit request =>
  val form = Form(tuple(
    "description" -> text,
    // ignores the form data but delivers its parameter as the value
    "image" -> ignored(request.body.file("image")).
       verifying("file missing", _.isDefined) // If not defined, no file was uploaded
  ))
  form.bindFromRequest.fold(
    formWithErrors => {
      Ok(views.html.fileupload.uploadform(formWithErrors))
    },
    value => Ok
  )
}
~~~~~~

~~~~~~
// actually is 
// Form[(String,Option[play.api.mvc.MultipartFormData.FilePart[play.api.libs.Files.TemporaryFile]])]
@(form: Form[_])
@helper.form(action = routes.FileUpload.upload, 'enctype -> "multipart/form-data") {
  @helper.inputText(form("description"))  
  @helper.inputFile(form("image"))
}

~~~~~~

~~~~~~
def showUploadForm() = Action {
  val dummyForm = Form(ignored("dummy"))
  Ok(views.html.fileupload.uploadform(dummyForm))
}
~~~~~~
