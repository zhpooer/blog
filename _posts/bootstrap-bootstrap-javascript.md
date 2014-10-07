title: Bootstrap-Bootstrap javascript
date: 2014-08-26 09:41:17
tags:
- bootstrap
---

# Overview #

As mentioned above, Bootstrap has a Data API where you can write data attributes into
the HTML of the page. If you need to turn off the Data API, you can unbind the attributes
by adding the following line of JavaScript:
`$('body').off('.data-api')`

If you need to disable a single plugin, you can do it programmatically using
the namespace of the plugin along with the data-api namespace:
`$('body').off('.alert.data-api')`


# Modal #

A modal is a child window that is layered over its parent window.

~~~~~~
<div class="modal hide fade">
  <div class="modal-header">
    <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
      &times;</button>
    <h3>Modal header</h3>
  </div>
  <div class="modal-body">
    <p>One fine body...</p>
  </div>
  <div class="modal-footer">
    <a href="#" class="btn">Close</a>
    <a href="#" class="btn btn-primary">Save changes</a>
  </div>
</div>
~~~~~~

~~~~~~
<!-- Button to trigger modal -->
<a href="#myModal" role="button"
   class="btn" data-toggle="modal">Launch demo modal</a>
<!-- Modal -->
<div id="myModal" class="modal hide fade" tabindex="-1" role="dialog"
     aria-labelledby="myModalLabel" aria-hidden="true">
  <div class="modal-header">
    <button type="button" class="close" data-dismiss="modal"
      aria-hidden="true">×</button>
    <h3 id="myModalLabel">Modal header</h3>
  </div>
  <div class="modal-body">
    <p>One fine body...</p>
  </div>
  <div class="modal-footer">
    <button class="btn" data-dismiss="modal" aria-hidden="true">Close</button>
    <button class="btn btn-primary">Save changes</button>
  </div>
</div>
~~~~~~

To start with, set `data-toggle="modal"` on the link or button
that you want to use to invoke the modal and then set the `data-target="#foo"` to the
ID of the modal that you’d like to use.

To call a modal with id="myModal", use a single line of JavaScript:
`$('#myModal').modal(options)`

To use the data attributes, prepend `data-` to the option name (e.g., `data-backdrop=""`)

| Name | Type | Default |  Description |
|-----|
| backdrop |  Boolean | true | Set to false if you don’t want the modal to be closed when the user clicks outside of the modal.|
| keyboard |  Boolean | true | Closes the modal when escape key is pressed; set to false to disable.|
| show |  Boolean | true | Shows the modal when initialized.|
| remote |  path | false | Using the jQuery .load method, inject content into the modal body. If an href with a valid URL is added, it will load that content. |

~~~~~~
$('#myModal').modal({
  keyboard: false
})

$('#myModal').modal('toggle')

$('#myModal').modal('show')

$('#myModal').modal('hide')
~~~~~~

## Event ##

| Event | Description |
|-----|
| show | Fired after the show method is called. |
| shown |Fired when the modal has been made visible to the user. |
| hide | Fired when the hide instance method has been called. |
| hidden |  Fired when the modal has finished being hidden from the user. |

~~~~~~
$('#myModal').on('hidden', function () {
  alert('Hey girl, I heard you like modals...');
})
~~~~~~

# Dropdown ##

To use a dropdown, add `data-toggle="dropdown"` to a link or button to
toggle the dropdown.

~~~~~~
<li class="dropdown">
  <a href="#" id="drop" role="button" class="dropdown-toggle"
    data-toggle="dropdown">Word <b class="caret"></b></a>
  <ul class="dropdown-menu" role="menu" aria-labelledby="drop">
    <li><a tabindex="-1" href="#">MAKE magazine</a></li>
    <li><a tabindex="-1" href="#">WordPress DevelopmentS</a></li>
    <li><a tabindex="-1" href="#">Speaking Engagements</a></li>
    <li class="divider"></li>
    <li><a tabindex="-1" href="#">Social Media</a></li>
  </ul>
</li>

~~~~~~

~~~~~~
$('.dropdown-toggle').dropdown()

$().dropdown('toggle')
~~~~~~

# Scrollspy #

The Scrollspy plugin (Figure 4-3) allows you to target sections of the page based on
scroll position.

For Scrollspy, you will need to add `data-spy="scroll"` to the `<body>` tag, along with
`data-target=".navbar"`


~~~~~~
<body data-spy="scroll" data-target=".navbar">...</body>

<div class="navbar">
  <div class="navbar-inner">
    <div class="container">
      <a class="brand" href="#">Jake's BBQ</a>
      <div class="nav-collapse">
        <ul class="nav">
          <li class="active"><a href="#">Home</a></li>
          <li><a href="#pork">Pork</a></li>
          <li><a href="#beef">Beef</a></li>
          <li><a href="#chicken">Chicken</a></li>
        </ul>
      </div><!-- /.nav-collapse -->
    </div>
  </div><!-- /navbar-inner -->
</div>
~~~~~~


# Toggleable Tabs #

~~~~~~
<ul class="nav nav-tabs">
  <li><a href="#home" data-toggle="tab">Home</a></li>
  <li><a href="#profile" data-toggle="tab">Profile</a></li>
  <li><a href="#messages" data-toggle="tab">Messages</a></li>
  <li><a href="#settings" data-toggle="tab">Settings</a></li>
</ul>
<div class="tab-content">
  <div class="tab-pane active" id="home">...</div>
  <div class="tab-pane" id="profile">...</div>
  <div class="tab-pane" id="messages">...</div>
  <div class="tab-pane" id="settings">...</div>
</div>

~~~~~~

~~~~~~
$('#myTab a[href="#profile"]').tab('show'); // Select tab by name
$('#myTab a:first').tab('show'); // Select first tab
$('#myTab a:last').tab('show'); // Select last tab
$('#myTab li:eq(2) a').tab('show'); // Select third tab (0-indexed)

$('a[data-toggle="tab"]').on('shown', function (e) {
  e.target // activated tab
  e.relatedTarget // previous tab
})

$('a[data-toggle="tab"]').on('show', function (e) {
  e.target // activated tab
  e.relatedTarget // previous tab
})
~~~~~~

# Tooltips #

~~~~~~
<a href="#" rel="tooltip" title="This is the tooltip">Tooltip Example</a>
~~~~~~

~~~~~~
// using javascript
$('#example').tooltip(options)
$('#element').tooltip('show')
$('#element').tooltip('hide')
$('#element').tooltip('toggle')
$('#element').tooltip('destroy')
~~~~~~

# Popover #

For the popover to activate, a user just needs to hover the cursor
over the element. 

~~~~~~
<a href="#" class="btn" rel="popover" title="Using Popover"
   data-content="Just add content to the data-content attribute.">Click Me!</a>
~~~~~~

~~~~~~
$('#element').popover('show')
~~~~~~

# Alert #


~~~~~~
<a class="close" data-dismiss="alert" href="#">&times;</a>
~~~~~~


~~~~~~
$(".alert").alert('close')
$('#my-alert').bind('closed', function () {
  // do something...
})
~~~~~~

# Buttons #

## Loading State ##

To add a loading state to a button, simply add `data-loading-text="Loading..."`
as an attribute to the button:
~~~~~~
<button type="button" class="btn btn-primary" data-loading-text="Loading...">
  Submit!</button>
~~~~~~

When the button is clicked, the .disabled class is added, giving the appearance that it
can no longer be clicked.

## Single Toggle ##

When clicking on a button with the `data-toggle="button"` attribute, a
class of `.active` is added:
~~~~~~
<button type="button" class="btn btn-primary" data-toggle="button">Toggle
</button>
~~~~~~

## Checkbox Buttons ##

Buttons can work like checkboxes

~~~~~~
<div class="btn-group" data-toggle="buttons-checkbox">
  <button type="button" class="btn btn-primary">Left</button>
  <button type="button" class="btn btn-primary">Middle</button>
  <button type="button" class="btn btn-primary">Right</button>
</div>
~~~~~~


## Radio Buttons ##

~~~~~~
<div class="btn-group" data-toggle="buttons-radio">
  <button type="button" class="btn btn-primary">Left</button>
  <button type="button" class="btn btn-primary">Middle</button>
  <button type="button" class="btn btn-primary">Right</button>
</div>
~~~~~~

## Usage ##

The `.button` method can be applied to any class or ID. To enable all buttons in
the `.nav-tabs` via JavaScript, add the following code:
`$('.nav-tabs').button()`

~~~~~~
$().button('toggle')
$().button('reset')
$().button('string')
~~~~~~

# Collapse #

Whether you use it to build accordion navigation or content boxes, it
allows for a lot of content options.

~~~~~~
<div class="accordion" id="accordion2">
  <div class="accordion-group">
    <div class="accordion-heading">
      <a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion2"
        href="#collapseOne">
        Collapsible Group Item #1
      </a>
    </div>
    <div id="collapseOne" class="accordion-body collapse in">
      <div class="accordion-inner">
        Anim pariatur cliche...
      </div>
    </div>
  </div>
  <div class="accordion-group">
    <div class="accordion-heading">
      <a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion2"
        href="#collapseTwo">
         Collapsible Group Item #2
      </a>
    </div>
    <div id="collapseTwo" class="accordion-body collapse">
      <div class="accordion-inner">
        Anim pariatur cliche...
      </div>
    </div>
  </div>
</div>

<!-- You can also use the data attributes to make all content collapsible:  -->
<button type="button" class="btn btn-danger" data-toggle="collapse"
  data-target="#demo">
  simple collapsible
</button>

<div id="demo" class="collapse in"> ... </div>
~~~~~~
