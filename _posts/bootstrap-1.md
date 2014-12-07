title: Bootstrap
date: 2014-12-06 16:08:18
tags:
---

There are many websites that offer Bootstrap themes. 
* http://bootswatch.com (free)
* http://startbootstrap.com (free)
* http://jobpixels.com (free)
* http://bootstrappage.com (free and paid)
* http://wrapbootstrap.com (paid)
* http://themes.walkingpixels.com (paid)
* http://themeforest.net (paid)

~~~~~~
<!DOCTYPE html>
<html>
  <head>
    <title>A simple blog </title>
    <meta name="viewport" content="width=device-width, initial-
                                   scale=1.0"/>
    <link href="css/bootstrap.min.css" rel="stylesheet"/>
    <link href="css/custom.css" rel="stylesheet"/>
  </head>
  <body>
    <nav class="navbar navbar-default" role="navigation">
      <div class="container">
        <div class="navbar-header">
          <button type="button"
                  class="navbar-toggle"
                  data-toggle="collapse"
                  data-target="#bs-example-navbar-collapse-1">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand"
             href="#">Blog</a>
        </div>
        <div class="collapse navbar-collapse"
             id="bs-example-navbar-collapse-1">
          <ul class="nav navbar-nav">
            <li class="active"><a href="#">Home</a></li>
            <li><a href="#">Archive</a></li>
            <li><a href="#">About</a></li>
            <li><a href="#">Contact</a></li>
          </ul>
        </div>
      </div>
    </nav>
    <div class="container">
      <div class="content">
        <div class="jumbotron">
          <div class="container">
            <h1>A simple blog</h1>
          </div>
        </div>
        <article>
          <header>
            <h2>Extending Bootstrap</h2>
            <p><time pubdate="pubdate">1/12/2012 3:36 PM</time>
            &middot; <a href="#">Blogger</a></p>
          </header>
          <p>Recently I stumbled on a book on extending Twitter
          Bootstrap and it really...</p>
          <p class="read-more"><a href="#">Read more &raquo;</a></p>
          <footer>
            <ul class="list-inline">
              <li><a href="#" class="label label-primary">Bootstrap</a></li>
              <li><a href="#" class="label label-primary">CSS</a></li>
              <li><a href="#" class="label label-primary">LESS</a></li>
              <li><a href="#" class="label label-primary">JavaScript</a></li>
              <li><a href="#" class="label label-primary">Grunt</a></li>
            </ul>
          </footer>
        </article>
      </div>
    </div>
    <script src="https://code.jquery.com/jquery.js"></script>
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.1.1/js/bootstrap.min.js"></script>
  </body>
</html>
~~~~~~

~~~~~~
<!-- `@import "bootstrap/bootstrap";` -->
<!-- recess less/main.less --compile > css/main.css -->
<!-- Remove it -->
<link href="css/bootstrap.min.css" rel="stylesheet"/>
<link href="css/custom.css" rel="stylesheet"/>
<!-- Add it -->
<link href="main.css" rel="stylesheet"/>
~~~~~~


# customizing variables #

`custom-variables.less`
~~~~~~
@brand-color: #bada55;
@navbar-default-bg: darken(@brand-primary, 10%);
@navbar-default-brand-hover-color: #fff;
@navbar-default-link-color: #fff;
@navbar-default-link-active-color: lighten(@brand-primary, 25%);
@navbar-default-link-active-bg: darken(@brand-primary, 20%);
@jumbotron-bg: lighten(@brand-primary, 30%);

~~~~~~

`main.less`
~~~~~~
@import "custom-theme";
@import "custom-variables";
~~~~~~

`custom-theme.less`
~~~~~~
.content {.make-md-column(9);}
article {margin-bottom: 40px;}
.sidebar {.make-md-column(3);}
.sidebar-avatar {
  display: block;
  margin-bottom: 20px;
  max-width: 100%;
}
.sidebar-bio {color: @gray;}
~~~~~~

~~~~~~
<div class="row">
  <div class="content">
    <div class="jumbotron">
      <div class="container">
        <h1>A simple blog</h1>
      </div>
    </div>
    <article>
      <header>
        <h2>Extending Bootstrap</h2>
        <p><time pubdate="pubdate">1/12/2012 3:36 PM</time>
        &middot; <a href="#">Blogger</a></p>
      </header>
      <p>Recently I stumbled on a book on extending Twitter
      Bootstrap and it really ...</p>
      <p class="read-more"><a href="#">Read more
      &raquo;</a></p>
      <footer>
        <ul class="list-inline">
          <li><a href="#" class="label label-
                                 primary">Bootstrap</a></li>
          <li><a href="#" class="label label-
                                 primary">CSS</a></li>
          <li><a href="#" class="label label-
                                 primary">LESS</a></li>
          <li><a href="#" class="label label-
                                 primary">JavaScript</a></li>
          <li><a href="#" class="label label-
                                 primary">Grunt</a></li>
        </ul>
      </footer>
    </article>
  </div>
  <aside class="sidebar">
    <img class="sidebar-avatar" src="http://
                                     lorempixel.com/400/400/cats" alt="Avatar"/>
    <p class="sidebar-bio">Christoffer is a web developer
    that Lorem ipsum dolor sit amet, consectetur
    adipisicing elit. Asperiores, maxime, neque?
    Assumenda at commodi et eum illum, incidunt ipsa
    laborum molestias, necessitatibus numquam quod
    ratione sint vero. Amet, facilis iusto. </p>
  </aside>
</div>
~~~~~~

# Grunt #

~~~~~~
sudo npm install –g grunt-cli

npm init
npm install grunt
npm install grunt-contrib-less
npm install grunt-contrib-watch
~~~~~~

~~~~~~
module.exports = function (grunt) {
  // Grunt configuration
  grunt.initConfig({
    less: {
      app: {
        files: {"less/main.less": "css/main.css"}
      }
    },
    watch: {
      styles: {
        files: ["less/**/*.less"],
        tasks: ["less:app"],
        options: {spawn: false}
      }
    }
  });
  // Load plugins
  grunt.loadNpmTasks("grunt-contrib-less");
  grunt.loadNpmTasks("grunt-contrib-watch");

};
~~~~~~

`grunt less:app` `grunt watch:styles`

`livereload: true`, 浏览器自动加载更改


# Cusomizing the grid #

~~~~~~
@grid-columns: 24;

.sidebar {
  .make-md-column(6);
}
.content {
  .make-md-column(18);
}

@grid-gutter-width: 50px;

@screen-xs-min: 500px;
@screen-sm-min: 790px;
@screen-md-min: 1020px;
@screen-lg-min: 1240px;
~~~~~~

# 设置 #

~~~~~~
@grid-columns:              12;
@grid-gutter-width:         30px;
@grid-float-breakpoint:     768px;

.wrapper {
  .make-row();
}
.content-main {
  .make-lg-column(8);
}
.content-secondary {
  .make-lg-column(3);
  .make-lg-column-offset(1);
}
~~~~~~

~~~~~~
<h1>h1. Bootstrap heading <small>Secondary text</small></h1>
<h2>h2. Bootstrap heading <small>Secondary text</small></h2>
<h3>h3. Bootstrap heading <small>Secondary text</small></h3>
<h4>h4. Bootstrap heading <small>Secondary text</small></h4>
<h5>h5. Bootstrap heading <small>Secondary text</small></h5>
<h6>h6. Bootstrap heading <small>Secondary text</small></h6>
~~~~~~

通过添加 `.lead` 类可以让段落突出显示。


`@font-size-base` 和 `@line-height-base` 决定排版尺寸


~~~~~~
<p class="text-left">Left aligned text.</p>
<p class="text-center">Center aligned text.</p>
<p class="text-right">Right aligned text.</p>
<p class="text-justify">Justified text.</p>
<p class="text-nowrap">No wrap text.</p>

<p class="text-lowercase">Lowercased text.</p>
<p class="text-uppercase">Uppercased text.</p>
<p class="text-capitalize">Capitalized text.</p>

<!-- 略缩语句 -->
<abbr title="attribute">attr</abbr>

<!-- 无样式列表 -->
<ul class="list-unstyled">
  <li>...</li>
</ul>
<!-- 内联样式 -->
<ul class="list-inline">
  <li>...</li>
</ul>
<!-- 横排排列 -->
<dl class="dl-horizontal">
  <dt>...</dt>
  <dd>...</dd>
</dl>
~~~~~~
