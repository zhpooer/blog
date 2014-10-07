title: Bootstrap-CSS
date: 2014-08-21 20:57:40
tags:
- bootstrap
---

# Typography #

Starting with typography, Bootstrap uses Helvetica Neue, Helvetica, Arial,
and sans-serif in its default font stack.

All body copy has the font-size set at 14 pixels, with the line-height set at 20 pixels.
The `<p>` tag has a margin-bottom of 10 pixels, or half the line-height.

## Headings ##

All six standard heading levels have been styled in Bootstrap, with the
`<h1>` at 36 pixels tall, and the `<h6>` down to 12 pixels.

In addition, to add an inline subheading to any of the headings, simply
add `<small>` around any of the elements and you will get smaller text in a lighter color.

## Lead Body Copy ##

To add some emphasis to a paragraph, add `class="lead"`.This will
give you larger font size, lighter weight, and a taller line height.
This is generally used for the first few paragraphs in a section,
but it can really be used anywhere:

~~~~~~
<p class="lead">Bacon ipsum dolor sit amet tri-tip pork loin ball tip frankfurter
swine boudin meatloaf shoulder short ribs cow drumstick beef jowl.
Meatball chicken sausage tail, kielbasa strip steak turducken venison prosciutto.
Chuck filet mignon tri-tip ribeye, flank brisket leberkas. Swine
turducken turkey shank, hamburger beef ribs bresaola pastrami venison rump.</p>
~~~~~~

## Emphasis ##

When `<small>` is applied to body text, the font shrinks to 85% of
its original size.

## Bold ##

To add emphasis to text, simply wrap it in a `<strong>` tag. This will
add font-weight:bold; to the selected text.

## Italics ##

For italics, wrap your content in the `<em>` tag. 

## Emphasis Classes ##

Along with `<strong>` and `<em>`, Bootstrap offers a few other classes
that can be used to provide emphasis.
These could be applied to paragraphs or spans:
~~~~~~
<p class="muted">This content is muted</p>
<p class="text-warning">This content carries a warning class</p>
<p class="text-error">This content carries an error class</p>
<p class="text-info">This content carries an info class</p>
<p class="text-success">This content carries a success class</p>
<p>This content has <em>emphasis</em>, and can be <strong>bold</strong></p>
~~~~~~

## Abbreviations ##

~~~~~~
<abbr title="Real Simple Syndication">RSS</abbr>
~~~~~~

## Addresses  ##

~~~~~~
<address>
  <strong>Jake Spurlock</strong><br>
  <a href="mailto:#">flast@oreilly.com</a>
</address>
~~~~~~

## Blockquotes ##

To add blocks of quoted text to your document—or for any quotation that you want to
set apart from the main text flow—add the `<blockquote>` tag around the text.

~~~~~~
<blockquote>
  <p>That this is needed, desperately needed, is indicated by the
  incredible uptake of Bootstrap. I use it in all the server software
  I'm working on. And it shows through in the templating language I'm
  developing, so everyone who uses it will find it's "just there" and
  works, any time you want to do a Bootstrap technique. Nothing to do,
  no libraries to include. It's as if it were part of the hardware.
  Same approach that Apple took with the Mac OS in 1984.</p>
  <small>Developer of RSS, <cite title="Source Title">Dave Winer</cite>
  </small>
</blockquote>
~~~~~~

If you want a `<blockquote>` with content that is right aligned,
add `.pull-right` to the tag.

## Lists ##
~~~~~~
<h3>Favorite Outdoor Activities</h3>
<ul>
  <li>Backpacking in Yosemite</li>
  <li>Hiking in Arches
    <ul>
      <li>Delicate Arch</li>
      <li>Park Avenue</li>
    </ul>
  </li>
  <li>Biking the Flintstones Trail</li>
</ul>

<h3>Self-Referential Task List</h3>
<ol>
  <li>Turn off the internet.</li>
  <li>Write the book.</li>
  <li>... Profit?</li>
</ol>

<h3>Common Electronics Parts</h3>
<dl>
  <dt>LED</dt>
  <dd>A light-emitting diode (LED) is a semiconductor light source.</dd>
  <dt>Servo</dt>
  <dd>Servos are small, cheap, mass-produced actuators used for radio
  control and small robotics.</dd>
</dl>
~~~~~~

To change the `<dl>` to a horizontal layout, with the `<dt>` on the left side and the `<dd>`
on the right, simply add `class="dl-horizontal"` to the opening tag.

# Code #

Generally, if you are going to be displaying code
inline, you should use the `<code>` tag.
But if the code needs to be displayed as a
standalone block element or if it has multiple lines,
then you should use the `<pre>` tag:


~~~~~~
<p>Instead of always using divs, in HTML5, you can use new elements like
<code>&lt;section&gt;</code>, <code>&lt;header&gt;</code>, and
<code>&lt;footer&gt;</code>. The html should look something like this:</p>
<pre>
&lt;article&gt;
&lt;h1&gt;Article Heading&lt;/h1&gt;
&lt;/article&gt;
</pre>
~~~~~~



# Tables #
~~~~~~
<table class="table">
  <caption>...</caption>
  <thead>
    <tr>
      <th>...</th>
      <th>...</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>...</td>
      <td>...</td>
    </tr>
  </tbody>
</table>
~~~~~~

## Optional Table Classes ##

Along with the base table markup and the `.table` class, there are a few additional classes
that you can use to style the markup. These four classes are:
* `.table-striped`, you will get stripes on rows within the `<tbody>`
* `.table-bordered`, you will get borders surrounding every element
and rounded corners around the entire table
* `.table-hover`,  a light gray background will be added to
rows while the cursor hovers over them.
* `.table-condensed`, row padding is cut in
half to condense the table. This is useful if you want denser information.

## Table Row Classes ##

| Class  |  Description |  Background  color |
|------|
| .success | Indicates a successful or positive action.| Green |
| .error | Indicates a dangerous or potentially negative action. | Red |
| .warning | Indicates a warning that might need attention. | Yellow| 
| .info | Used as an alternative to the default styles. | Blue | 

# Forms #

~~~~~~
<form>
  <fieldset>
    <legend>Legend</legend>
    <label for="name">Label name</label>
    <input type="text" id="name" placeholder="Type something...">
    <span class="help-block">Example block-level help text here.</span>
    <label class="checkbox" for="checkbox">
      <input type="checkbox" id="checkbox"> Check me out
    </label>
    <button type="submit" class="btn">Submit</button>
  </fieldset>
</form>

~~~~~~

## Optional Form Layouts ##

### Search form ###

Add `.form-search` to the `<form>` tag, and then add `.search-query`
to the `<input>` for an input box with rounded corners
and an inline submit button

~~~~~~
<form class="form-search">
  <input type="text" class="input-medium search-query">
  <button type="submit" class="btn">Search</button>
</form>
~~~~~~

### Inline form ###

To have the label and the input
on the same line, use this inline form code:

~~~~~~
<form class="form-inline">
  <input type="text" class="input-small" placeholder="Email">
  <input type="password" class="input-small" placeholder="Password">
  <label class="checkbox">
    <input type="checkbox"> Remember me
  </label>
  <button type="submit" class="btn">Sign in</button>
</form>
~~~~~~

### Horizontal form ###

To create a form that uses the horizontal layout, do the following:
* Add a class of `.form-horizontal` to the parent `<form>` element.
* Wrap labels and controls in a `<div>` with class `.control-group`.
* Add a class of `.control-label` to the labels.
* Wrap any associated controls in a `<div>` with class `.controls` for proper alignment.

~~~~~~
<form class="form-horizontal">
  <div class="control-group">
    <label class="control-label" for="inputEmail">Email</label>
    <div class="controls">
      <input type="text" id="inputEmail" placeholder="Email">
    </div>
  </div>
  <div class="control-group">
    <label class="control-label" for="inputPassword">Password</label>
    <div class="controls">
      <input type="password" id="inputPassword" placeholder="Password">
    </div>
  </div>
  <div class="control-group">
    <div class="controls">
      <label class="checkbox">
        <input type="checkbox"> Remember me
      </label>
      <button type="submit" class="btn">Sign in</button>
    </div>
  </div>
</form>
~~~~~~

## Supported Form Controls ##

Bootstrap natively supports the most common form controls. Chief among them are
input, textarea, checkbox, radio, and select.

If you want multiple checkboxes to appear on the same line together, add the .inline
class to a series of checkboxes or radio buttons

~~~~~~
<label for="option1" class="checkbox inline">
  <input id="option1" type="checkbox" id="inlineCheckbox1" value="option1"> 1
</label>
<label for="option2" class="checkbox inline">
  <input id="option2" type="checkbox" id="inlineCheckbox2" value="option2"> 2
</label>

<select multiple="multiple">
  <option>1</option>
  <option>2</option>
  <option>3</option>
  <option>4</option>
  <option>5</option>
</select>

~~~~~~

## Extended Form Controls ##

In addition to the basic form controls listed in the previous section, Bootstrap offers a
few other form components to complement the standard HTML form elements.


### Prepended and appended inputs ###

~~~~~~
<div class="input-prepend">
  <span class="add-on">@</span>
  <input class="span2" id="prependedInput" type="text" placeholder="Username">
</div>
<div class="input-append">
  <input class="span2" id="appendedInput" type="text">
  <span class="add-on">.00</span>
</div>

<div class="input-prepend input-append">
  <span class="add-on">$</span>
  <input class="span2" id="appendedPrependedInput" type="text">
  <span class="add-on">.00</span>
</div>

<div class="input-append">
  <input class="span2" id="appendedInputButtons" type="text">
  <button class="btn" type="button">Search</button>
  <button class="btn" type="button">Options</button>
</div>

<form class="form-search">
  <div class="input-append">
    <input type="text" class="span2 search-query">
    <button type="submit" class="btn">Search</button>
  </div>
  <div class="input-prepend">
    <button type="submit" class="btn">Search</button>
    <input type="text" class="span2 search-query">
  </div>
</form>
~~~~~~

## Form Control Sizing ##

If you want the input to act as a block-level element, you can
add `.input-block-level` and it will be the full
width of the container element

~~~~~~
<input class="input-mini" type="text" placeholder=".input-mini">
<input class="input-small" type="text" placeholder=".input-small">
<input class="input-medium" type="text" placeholder=".input-medium">
<input class="input-large" type="text" placeholder=".input-large">
<input class="input-xlarge" type="text" placeholder=".input-xlarge">
<input class="input-xxlarge" type="text" placeholder=".input-xxlarge">
~~~~~~

### Grid sizing ###

You can use any .span from .span1 to .span12 for form control sizing.

~~~~~~
<input class="span1" type="text" placeholder=".span1">
<input class="span2" type="text" placeholder=".span2">
<input class="span3" type="text" placeholder=".span3">
<select class="span1">
...
</select>
<select class="span2">
...
</select>
<select class="span3">
...
</select>
~~~~~~

If you want to use multiple inputs on a line, simply use the `.controls-row` modifier
class to apply the proper spacing. It floats the inputs to collapse the
white space; sets the correct margins; and, like the `.row` class, clears the float:

~~~~~~
<div class="controls">
  <input class="span5" type="text" placeholder=".span5">
</div>
<div class="controls controls-row">
  <input class="span4" type="text" placeholder=".span4">
  <input class="span1" type="text" placeholder=".span1">
</div>  
~~~~~~

### Uneditable text ###

If you want to present a form control without allowing the user to edit the input, simply
add the class `.uneditable-input`

~~~~~~
<span class="input-xlarge uneditable-input">Some value here</span>
~~~~~~

### Form actions ###

When you place the form actions at the bottom of a `.horizontal-form`, the inputs will
correctly line up with the floated form controls

~~~~~~
<div class="form-actions">
  <button type="submit" class="btn btn-primary">Save changes</button>
  <button type="button" class="btn">Cancel</button>
</div>
~~~~~~

### Help text ###
Bootstrap form controls can have either block or inline text that flows with the inputs

~~~~~~
<input type="text"><span class="help-inline">Inline help text</span>

<input type="text"><span class="help-block">A longer block of help text that
breaks onto a new line and may extend beyond one line.</span>
~~~~~~

## Form Control States ##

In addition to the `:focus` state, Bootstrap offers styling
for disabled inputs and classes for form validation.

### Validation states ###

~~~~~~
<div class="control-group warning">
  <label class="control-label" for="inputWarning">Input with warning</label>
  <div class="controls">
    <input type="text" id="inputWarning">
    <span class="help-inline">Something may have gone wrong</span>
  </div>
</div>

<div class="control-group error">
  <label class="control-label" for="inputError">Input with error</label>
  <div class="controls">
    <input type="text" id="inputError">
    <span class="help-inline">Please correct the error</span>
  </div>
</div>

<div class="control-group success">
  <label class="control-label" for="inputSuccess">Input with success</label>
  <div class="controls">
    <input type="text" id="inputSuccess">
    <span class="help-inline">Woohoo!</span>
  </div>
</div>
~~~~~~

## Buttons ##

| Class |  Description  |
|---|
| btn |  Standard gray button with gradient|
| btn btn-primary | Provides extra visual weight and identifies the primary action in a set of buttons (blue)|
| btn btn-info |  Used as an alternative to the default styles (light blue)|
| btn-success|  Indicates a successful or positive action (green)|
| btn btn-warning | Indicates caution should be taken with this action (orange)|
| btn btn-danger|  Indicates a dangerous or potentially negative action (red)| btn btn-inverse Alternate dark-gray button, not tied to a semantic action or use|
| btn btn-link|  De-emphasizes a button by making it look like a link while maintaining button behavior |


### Button Sizes ###

If you need larger or smaller buttons, simply add `.btn-large`, `.btn-small`, or
`.btn-mini`  to links or buttons

~~~~~~
<p>
  <button class="btn btn-mini btn-primary" type="button">Mini button</button>
  <button class="btn btn-mini" type="button">Mini button</button>
</p>
~~~~~~

If you want to create buttons that display like a block-level element, simply
add the `.btn-block` class. These buttons will display at 100% width:
~~~~~~
<button class="btn btn-large btn-block btn-primary" type="button">Block-
level button</button>
<button class="btn btn-large btn-block" type="button">Block-level button</button>
~~~~~~

### Disabled Button Styling ###

~~~~~~
<a href="#" class="btn btn-large btn-primary disabled">Primary link</a>
<a href="#" class="btn btn-large disabled">Link</a>

<button type="button" class="btn btn-large btn-primary disabled"
        disabled="disabled">Primary button</button>
~~~~~~

# Images #

Images have three classes that can be used to apply some simple
styles: `.img-rounded` adds `border-radius:6px` to give the image rounded
corners, `.img-circle` makes the entire image round by adding `border-radius:500px`, and
`.img-polaroid` adds a bit of padding and a gray border:

~~~~~~
<img src="..." class="img-rounded">
<img src="..." class="img-circle">
<img src="..." class="img-polaroid">
~~~~~~


# Icons #

Bootstrap bundles 140 icons into one sprite that can be used with buttons, links, navi‐
gation, and form fields. The icons are provided by GLYPHICONS

~~~~~~
<div class="btn-toolbar">
  <div class="btn-group">
    <a class="btn" href="#"><iclass="icon-align-left"></i></a>
    <a class="btn" href="#"><i class="icon-align-center"></i></a>
    <a class="btn" href="#"><i class="icon-align-right"></i></a>
    <a class="btn" href="#"><i class="icon-align-justify"></i></a>
  </div>
</div>
~~~~~~

## Navigation ##

~~~~~~
<ul class="nav nav-list">
  <li class="active"><a href="#"><i class="icon-home icon-white"></i>Home</a></li>
  <li><a href="#"><i class="icon-book"></i> Library</a></li>
  <li><a href="#"><i class="icon-pencil"></i> Applications</a></li>
  <li><a href="#"><i class="i"></i> Misc</a></li>
</ul>
~~~~~~
