title: SASS 基础语法
date: 2014-12-03 08:56:40
tags:
- SASS
---

# Base semantic #

~~~~~~
/* 下划线 和 中线可以一起用 */
$link-color: blue;
a { color: $link_color }

/* 嵌套 */
# content {
  article {
    h1 { color: #333}
    p{margin-bottom: 1.4em }
  }
  aside { background-color: #eee }
}

/* 父选择器 */
article a {
  color: blue;
  &:hover { color: red }
}

#content aside {
  color: red;
  body.ie & { color: green}
}

nav, aside {
  a { color: blue }
}
~~~~~~

# combinators #

~~~~~~
/*子选择*/
article > section { border: 1px solid #ccc }

/* 后继选择, header后面跟着的p */
header + p { font-size: 1.1em }

/* 兄弟选择器 article 后面所有的 article */
article ~ article { border-top: 1px dashed #ccc }

article {
  ~ article { border-top: 1px dashed #ccc }
  > section { background: #eee }
  dl > {
    dt { color: #333 }
    dd { color: #555 }
  }
  nav + & { margin-top: 0 }
}

/* 简化 border-style border-width */
nav {
  border: {
  style: solid;
  width: 1px;
  color: #ccc;
  }
}

nav {
  border: 1px solid #ccc {
    left: 1px;
    right: 0px;
  }
}

~~~~~~


# Importing Sass files #

Sass has an `@import` rule as well, but Sass does its
importing when it’s compiling to CSS.
~~~~~~
@import "colors"; /* import include the colors.scss */
~~~~~~

The convention for Sass partials is to begin the filenames with _. This tells Sass that
it shouldn’t generate an individual CSS file for the partial, and should only use it for
imports.

~~~~~~
/* `themes/_night-sky.scss` */
@import "themes/night-sky";

/**
It means, if this variable is already declared,
leave it alone, but otherwise use this value.
**/
$fancybox-width: 400px !default;
.fancybox {
  width: $fancybox-width;
}

/* If a user sets $fancybox-width before @importing your Sass partial, then your declara-
tion of 400px is ignored because of the !default flag. If the user hasn’t set the value
of $fancybox-width it’ll default to 400px.*/

~~~~~~

# Nested imports #

~~~~~~
/* _blue-theme.scss */
aside {
  background: blue;
  color: white;
}

.blue-theme {@import "blue-theme"}

/** 翻译成
.blue-theme {
  aside {
    background: blue;
    color: #fff;
  }
}
**/
~~~~~~

This means you can’t directly import a plain CSS file without having Sass think you
want a plain CSS @import as well. 
* The imported filename ends with .css.
* The imported filename is a URL
(such as "http://sass-lang.com/stylesheets/application.css").
This allows Sass files to use services like Google’s Font API.
* The imported filename is a CSS url() value.

# Comment #
~~~~~~
body {
color: #333; // This won't appear in the CSS
padding: 0; /* This will appear in the CSS */
}

body {
color /* This won't appear in the CSS */: #333;
padding: 1em; /* Nor will this */ 0;
}

~~~~~~

# Mixins #
~~~~~~
@mixin rounded-corners {
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}

.notice {
  background-color: green;
  border: 2px solid #00aa00;
  @include rounded-corners;
}

@mixin no-bullets {
  list-style: none;
  li{
    list-style-image: none;
    list-style-type: none;
    margin-left;
  }
}

ul.plain {
  color: #444;
  @include no-bullets;
}

@mixin link-colors($normal, $hover, $visited){
  color: $normal;
  &:hover {color: $hover;}
&:visited {color: $visited;}
  }

a {
  @include link-colors(blue, red, green);
}

a{
  @include link-colors(
    $normal: blue,
    $visited: green;
    $hover: red
  )
  }

/* have default value */
@mixin link-colors (
  $normal,
  $hover: $normal,
  $visited: $normal,
) {
  color: $normal;
  &:hover { color: $hover;}
  &:visited {color: $visited;}
}
~~~~~~


# Selector inheritance #

~~~~~~
.error {
  border: 1px red;
  background-color: #fdd;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}

~~~~~~

So if `.seriousError` `@extended` `.important.error` ,
it would inherit styles for `.important.error` and `h1.important.error` ,
but not for `.important` or `.error` .
In this case, you’d probably want `.seriousError` to `@extend`
`.important` and `.error` separately.
