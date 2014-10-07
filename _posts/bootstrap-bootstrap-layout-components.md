title: Bootstrap-Bootstrap Layout Components
date: 2014-08-23 19:55:00
tags:
- bootstrap
---

Bootstrap provides a toolkit of flexible components that can be used
in designing application interfaces, web features, and more. All of the
plugins are available in one separate JavaScript file, or you
can use the Bootstrap customizer to pick and choose which plugins you want.

# Dropdown Menus #
Dropdown menus are toggleable, contextual menus for displaying links in a list format.
The dropdowns can be used on a variety of different elements, navs, buttons, and more.

You can have a single dropdown or extend the dropdown into another submenu.

~~~~~~
<ul class="dropdown-menu" role="menu" aria-labelledby="dropdownMenu">
  <li><a tabindex="-1" href="#">Action</a></li>
  <li><a tabindex="-1" href="#">Another action</a></li>
  <li><a tabindex="-1" href="#">Something else here</a></li>
  <li class="divider"></li>
  <li><a tabindex="-1" href="#">Separated link</a></li>
</ul>

~~~~~~

## Right-align ##

Add `.pull-right` to a `.dropdown-menu` to right-align the dropdown menu to
the parent object:

~~~~~~
<ul class="dropdown-menu pull-right" role="menu" aria-labelledby="dLabel">
...
</ul>
~~~~~~

## Submenu ##

~~~~~~
<ul class="dropdown-menu" role="menu" aria-labelledby="dLabel">
  ...
  <li class="dropdown-submenu">
    <a tabindex="-1" href="#">More options</a>
    <ul class="dropdown-menu">
    ...
    </ul>
  </li>
</ul>
~~~~~~

# Button Groups #

Button groups allow multiple buttons to be stacked together.
~~~~~~
<div class="btn-group">
  <button class="btn">1</button>
  <button class="btn">2</button>
  <button class="btn">3</button>
</div>

<div class="btn-toolbar">
  <div class="btn-group">
    <a class="btn" href="#"><i class="icon-align-left"></i></a>
    <a class="btn" href="#"><i class="icon-align-center"></i></a>
    <a class="btn" href="#"><i class="icon-align-right"></i></a>
    <a class="btn" href="#"><i class="icon-align-justify"></i></a>
  </div>
  <div class="btn-group">
    <a class="btn" href="#"><i class="icon-italic"></i></a>
    <a class="btn" href="#"><i class="icon-bold"></i></a>
    <a class="btn" href="#"><i class="icon-font"></i></a>
    <a class="btn" href="#"><i class="icon-text-height"></i></a>
    <a class="btn" href="#"><i class="icon-text-width"></i></a>
  </div>
  <div class="btn-group">
    <a class="btn" href="#"><i class="icon-indent-left"></i></a>
    <a class="btn" href="#"><i class="icon-indent-right"></i></a>
  </div>
</div>

<div class="btn-group btn-group-vertical">
...
</div>
~~~~~~

# Buttons with Dropdowns #

To add a dropdown to a button, simply wrap the button and dropdown
menu in a `.btn-group`. You can also use `<span class="caret"></span>`
to act as an indicator that the button is a dropdown:
~~~~~~
<div class="btn-group">
  <button class="btn btn-danger">Danger</button>
  <button class="btn btn-danger dropdown-toggle" data-toggle="dropdown">
    <span class="caret"></span>
  </button>
  <ul class="dropdown-menu">
    <li><a href="#">Action</a></li>
    <li><a href="#">Another action</a></li>
    <li><a href="#">Something else here</a></li>
    <li class="divider"></li>
    <li><a href="#">Separated link</a></li>
  </ul>
</div>

~~~~~~

## Split Button Dropdowns ##

~~~~~~
<div class="btn-group">
  <button class="btn">Action</button>
  <button class="btn dropdown-toggle" data-toggle="dropdown">
    <span class="caret"></span>
  </button>
  <ul class="dropdown-menu">
    <!-- dropdown menu links -->
  </ul>
</div>
~~~~~~

## Dropup Menus ##

Menus can also be built to drop up rather than down . To make this
change, simply add `.dropup` to the `.btn-group` container.
To have the button pull up from the righthand side, add `.pull-right` to the `.dropdown-menu`

~~~~~~
<div class="btn-group dropup">
  <button class="btn">Dropup</button>
  <button class="btn dropdown-toggle" data-toggle="dropdown">
    <span class="caret"></span>
  </button>
  <ul class="dropdown-menu">
    <!-- dropdown menu links -->
  </ul>
</div>
~~~~~~

# Navigation Elements #

Bootstrap provides a few different options for styling navigation elements. All of them
share the same markup and base class, `.nav`.

## Tabular Navigation ##

~~~~~~
<ul class="nav nav-tabs">
  <li class="active">
    <a href="#">Home</a>
  </li>
  <li><a href="#">Profile</a></li>
  <li><a href="#">Messages</a></li>
</ul>
~~~~~~

## Basic Pills Navigation ##

~~~~~~
<ul class="nav nav-pills">
  <li class="active">
    <a href="#">Home</a>
  </li>
  <li><a href="#">Profile</a></li>
  <li><a href="#">Messages</a></li>
</ul>
~~~~~~

## Disabled class ##
~~~~~~
<ul class="nav nav-pills">
  ...
  <li class="disabled"><a href="#">Home</a></li>
  ...
</ul>
~~~~~~

## Stackable Navigation ##
To make them appear vertically stacked, just add the `.nav-stacked` class.

~~~~~~
<ul class="nav nav-tabs nav-stacked">
  ...
</ul>
~~~~~~

## Dropdowns ##

Navigation menus share a similar syntax with dropdown menus.

~~~~~~
<ul class="nav nav-tabs">
  <li class="dropdown">
    <a class="dropdown-toggle" data-toggle="dropdown" href="#">
      Dropdown
      <b class="caret"></b>
    </a>
    <ul class="dropdown-menu">
      <li><a href="#">Action</a></li>
      <li><a href="#">Another action</a></li>
      <li><a href="#">Something else here</a></li>
      <li class="divider"></li>
      <li><a href="#">Separated link</a></li>
    </ul>
  </li>
</ul>
~~~~~~

## Navigation Lists ##

~~~~~~
<ul class="nav nav-list">

  <li class="nav-header">List Header</li>
  <li class="active"><a href="/">Home</a></li>
  <li><a href="#">Library</a></li>
  <li><a href="#">Applications</a></li>
  
  <li class="nav-header">Another List Header</li>
  <li><a href="#">Profile</a></li>
  <li><a href="#">Settings</a></li>
  
  <li class="divider"></li>
  <li><a href="#">Help</a></li>
</ul>
~~~~~~

### Horizontal divider ###
To create a divider, much like an `<hr />`, use an empty `<li>` with a class of `.divider`:

~~~~~~
<ul class="nav-menu">
  ...
  <li class="divider"></li>
  ....
</ul>
~~~~~~



## Tabbable Navigation ##

You can also add interaction by opening different windows of content.
To make navigation tabs, create
a `.tab-pane` with a unique ID for every tab, and then wrap them
in `.tab-content`:

~~~~~~
<div class="tabbable">
  <ul class="nav nav-tabs">
    <li class="active"><a href="#tab1" data-toggle="tab">Meats</a></li>
    <li><a href="#tab2" data-toggle="tab">More Meat</a></li>
  </ul>
  <div class="tab-content">
    <div class="tab-pane active" id="tab1">
      <p>Bacon ipsum dolor sit amet jerky flank...</p>
    </div>
    <div class="tab-pane" id="tab2">
      <p>Beef ribs, turducken ham hock...</p>
    </div>
  </div>
</div>
~~~~~~

Tabs on the left get the `.tabs-left` class. `<div class="tabbable tabs-left">`

Tabs on the right get the `.tabs-right` class.

~~~~~~
<div class="tabbable tabs-right">
  <ul class="nav nav-tabs">
    <li class="active"><a href="#tab1" data-toggle="tab">Section A</a></li>
    <li><a href="#tab2" data-toggle="tab">Section B</a></li>
    <li><a href="#tab3" data-toggle="tab">Section C</a></li>
  </ul>
  <div class="tab-content">
    <div class="tab-pane active" id="tab1">
      <p>I'm in section A.</p>
    </div>
    <div class="tab-pane" id="tab2">
      <p>I'm in section B.</p>
    </div>
    <div class="tab-pane" id="tab3">
      <p>I'm in section C.</p>
    </div>
  </div>
</div>
~~~~~~

# Navbar #
The navbar includes styling for site names and basic navigation.
It can later be extended by adding form-specific controls and specialized dropdowns.

~~~~~~
<div class="navbar">
  <div class="navbar-inner">
    <a class="brand" href="#">Title</a>
    <ul class="nav">
      <li class="active"><a href="#">Home</a></li>
      <li><a href="#">Link</a></li>
      <li><a href="#">Link</a></li>
    </ul>
  </div>
</div>

<a class="brand" href="#">Project name</a>
~~~~~~

Note the `.brand` class in the code. This will give the text a lighter font-weight and
slightly larger size.

If you want to add a divider to your links, you can do that by adding an empty
list item with a class of `.divider-vertical`:

~~~~~~
<ul class="nav">
  <li class="active"><a href="#">Home</a></li>
  <li><a href="#">First Link</a></li>
  <li><a href="#">Second Link</a></li>
  <li class="divider-vertical"></li>
  <li><a href="#">Third Link</a></li>
</ul>
~~~~~~

## Forms ##
Of note, `.pull-left` and `.pull-right`
helper classes may help move the form into the proper position:

~~~~~~
<form class="navbar-form pull-left">
  <input type="text" class="span2" id="fname">
  <button type="submit" class="btn">
</form>

<form class="navbar-search" accept-charset="utf-8">
  <input type="text" class="search-query" placeholder="Search">
</form>
~~~~~~

## Navbar Menu Variations ##

The Bootstrap navbar can be dynamic in its positioning. By default, it is a block-level
element that takes its positioning based on its placement in the HTML. 

If you want the navbar fixed to the top, add `.navbar-fixed-top` to the `.navbar` class.



~~~~~~
<!-- add at least 40 pixels of padding to the body tag: -->
<div class="navbar navbar-fixed-top">
  <div class="navbar-inner">
    <a class="brand" href="#">Title</a>
    <ul class="nav">
      <li class="active"><a href="#">Home</a></li>
      <li><a href="#">Link</a></li>
      <li><a href="#">Link</a></li>
    </ul>
  </div>
</div>

<div class="navbar navbar-fixed-bottom">
  <div class="navbar-inner">
    <a class="brand" href="#">Title</a>
    <ul class="nav">
      <li class="active"><a href="#">Home</a></li>
      <li><a href="#">Link</a></li>
      <li><a href="#">Link</a></li>
    </ul>
  </div>
</div>
~~~~~~

### Static top navbar ###

~~~~~~
<div class="navbar navbar-static-top">
  <div class="navbar-inner">
    <a class="brand" href="#">Title</a>
    <ul class="nav">
      <li class="active"><a href="#">Home</a></li>
      <li><a href="#">Link</a></li>
      <li><a href="#">Link</a></li>
    </ul>
  </div>
</div>
~~~~~~


### Responsive navbar ###

Add the responsive features, the content that you want to be collapsed
needs to be wrapped in a `<div>` with `.nav-collapse` `.collapse` as a class.

~~~~~~
<div class="header">
  <div class="navbar-inner">
    <div class="container">
      <a class="btn btn-navbar" data-toggle="collapse"
         data-target=".nav-collapse">
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </a>
<!-- Leave the brand out if you want it to be shown when other elements
are collapsed... -->
      <a href="#" class="brand">Project Name</a>
<!-- Everything that you want collapsed, should be added to the collapse
div. -->
      <div class="nav-collapse collapse">
<!-- .nav, .navbar-search etc... -->
      </div>
    </div>
  </div>
</div>

~~~~~~

### Inverted navbar ###

To create an inverted navbar with a black background and white text as shown in
, simply add `.navbar-inverse` to the `.navbar` class:
~~~~~~
<div class="navbar navbar-inverse">
  ...
</div>
~~~~~~

# Breadcrumbs #

Breadcrumbs are a great way to show hierarchy-based information for a site.
There is a also a helper class of `.divider` that mutes the colors and
makes the text a little smaller. 

~~~~~~
<ul class="breadcrumb">
  <li><a href="#">Home</a> <span class="divider">/</span></li>
  <li><a href="#">2012</a> <span class="divider">/</span></li>
  <li><a href="#">December</a> <span class="divider">/</span></li>
  <li><a href="#">5</a></li>
</ul>

<ul class="breadcrumb">
  <li><a href="#">Home</a> <span class="divider">&rarr;</span></li>
  <li><a href="#">Dinner Menu</a> <span class="divider">&rarr;</span></li>
  <li><a href="#">Specials</a> <span class="divider">&rarr;</span></li>
  <li><a href="#">Steaks</a></li>
</ul>

<ul class="breadcrumb">
  <li><a href="#">Home</a> <span class="divider">&raquo;</span></li>
  <li><a href="#">Electronics</a> <span class="divider">&raquo;</span></li>
  <li><a href="#">Raspberry Pi</a></li>
</ul>

~~~~~~

# Pagination #

Bootstrap handles pagination like a lot of other interface elements, an unordered list,
with wrapper a `<div>` that has a specific class that identifies the element.

~~~~~~
<div class="pagination">
  <ul>
    <li><a href="#">&laquo;</a></li>
    <li><a href="#">1</a></li>
    <li><a href="#">2</a></li>
    <li><a href="#">3</a></li>
    <li><a href="#">4</a></li>
    <li><a href="#">5</a></li>
    <li><a href="#">&raquo;</a></li>
  </ul>
</div>

<div class="pagination pagination-centered">
  <ul>
    <li class="disabled"><a href="#">«</a></li>
    <li class="active"><a href="#">1</a></li>
    <li><a href="#">2</a></li>
    <li><a href="#">3</a></li>
    <li><a href="#">4</a></li>
    <li><a href="#">5</a></li>
    <li><a href="#">»</a></li>
  </ul>
</div>
~~~~~~

You can add `.pagination-centered` to the parent `<div>`.
This will center the contents of the
`<div>`. If you want the items right-aligned in the `<div>`, add `.pagination-right`.

For sizing, in addition to the normal size, there are three other sizes that can be applied by
adding a class to the wrapper `<div>`: `.pagination-large`, `.pagination-small`,
and `.pagination-mini`

## Pager ##

~~~~~~
<ul class="pager">
  <li class="previous">
    <a href="#">&larr; Older</a>
  </li>
  <li class="next">
    <a href="#">Newer &rarr;</a>
  </li>
</ul>

~~~~~~

# Label #

~~~~~~
<span class="label">Default</span>
<span class="label label-success">Success</span>
<span class="label label-warning">Warning</span>
<span class="label label-important">Important</span>
<span class="label label-info">Info</span>
<span class="label label-inverse">Inverse</span>
~~~~~~

## Badges ##

Badges are similar to labels

~~~~~~
<span class="badge">1</span>
<span class="badge badge-success">2</span>
<span class="badge badge-warning">4</span>
<span class="badge badge-important">6</span>
<span class="badge badge-info">8</span>
<span class="badge badge-inverse">10</span>
~~~~~~

# Typographic Elements #

In addition to buttons, labels, forms, tables, and tabs, Bootstrap has a few more elements
for basic page layout.

## Hero Unit ##

The hero unit is a large content area that increases the size of headings and adds a lot
of margin for landing page content 

~~~~~~
<div class="hero-unit">
  <h1>Hello, World!</h1>
  <p>This is a simple hero unit, a simple jumbotron-style component for calling
     extra attention to featured content or information.</p>
  <p><a class="btn btn-primary btn-large">Learn more</a></p>
</div>

~~~~~~

## Page Header ##

The page header is a nice little feature to add appropriate spacing
around the headings on a page.

~~~~~~
<div class="page-header">
  <h1>Example page header <small>Subtext for header</small></h1>
</div>
~~~~~~

## Thumbnails ##

To create a thumbnail, add an `<a>` tag with the class of `.thumbnail` around an
image. This adds four pixels of padding and a gray border

~~~~~~
<a href="#" class="thumbnail">
  <img alt="Kittens!" style="" src="http://placekitten.com/300/250">
</a>

<ul class="thumbnails">
  <li class="span4">
    <div class="thumbnail">
      <img data-src="holder.js/300x200" alt="300x200" style="">
      <div class="caption">
        <h3>Meats</h3>
        <p>Bacon ipsum dolor sit amet sirloin pancetta shoulder tongue doner,
        shank sausage.</p>
        <p><a href="#" class="btn btn-primary">Eat now!</a> <a href="#"
        class="btn">Later...</a></p>
      </div>
    </div>
  </li>
  <li class="span4">
  ...
  </li>
</ul>
~~~~~~

# Alerts #

~~~~~~
<div class="alert">
  <a href="#" class="close" data-dismiss="alert">&times;</a>
  <strong>Warning!</strong> Not to be alarmist, but you have now been alerted.
</div>
~~~~~~

To close the alert, you can use a button that contains the `data-dismiss="alert"` attribute.

They are added by using either `.alert-error`, `.alert-success`, or `.alert-info`.


# Progress Bars #

~~~~~~
<div class="progress">
  <div class="bar" style="width: 60%;"></div>
</div>

<div class="progress progress-striped">
  <div class="bar" style="width: 20%;"></div>
</div>

<div class="progress progress-striped active">
  <div class="bar" style="width: 40%;"></div>
</div>

<div class="progress">
  <div class="bar bar-success" style="width: 35%;"></div>
  <div class="bar bar-warning" style="width: 20%;"></div>
  <div class="bar bar-danger" style="width: 10%;"></div>
</div>
~~~~~~

In addition to the blue progress bar, there are options for green, yellow, and red using
the `.bar-success`, `.bar-warning`, and `.bar-danger` classes. 

# Media Object #

When you look at social sites like Facebook, Twitter, and others, and strip away some
of the formatting from timelines, you will see the media object

~~~~~~
<div class="media">
  <a class="pull-left" href="#">
    <img class="media-object" data-src="holder.js/64x64">
  </a>
  <div class="media-body">
    <h4 class="media-heading">Media heading</h4>
    <p>...</p>
    <!-- Nested media object -->
    <div class="media">
      ...
    </div>
  </div>
</div>

<ul class="media-list">
  <li class="media">
    <a class="pull-left" href="#">
      <img class="media-object" data-src="holder.js/64x64">
    </a>
    <div class="media-body">
      <h4 class="media-heading">Media heading</h4>
      <p>...</p>
      ...
      <!-- Nested media object -->
      <div class="media">
      ...
      </div>
    </div>
  </li>
</ul>
~~~~~~

# Miscellaneous #

Some of these components are layout-based, and a few are production-based helper
classes. 

## Wells ##

A well is a container `<div>` that causes the content to appear sunken on the page.

~~~~~~
<div class="well">
  ...
</div>
~~~~~~

There are two additional classes that can be used in conjunction with `.well`:
`.well-large` and `.well-small`. 

## Pull left ##

~~~~~~
<div class="pull-left">
  ...
</div>
.pull-left {
  float: left;
}

<div class="pull-right">
  ...
</div>
.pull-right {
  float: right;
}

~~~~~~

## Clearfix ##

To clear the float of any element, use the `.clearfix` class.

When you have two elements
of different sizes that are floated alongside each other, it is necessary to
force the following elements in the the code below or to clear the preceding content.


