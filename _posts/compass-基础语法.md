title: Compass 基础语法
date: 2014-12-04 09:18:21
tags:
---


# Compass Base #
~~~~~~
@import "reset/utilities";
@include global-reset;

/* a :hover :active :visited :focus */
a { @include link-colors(#333, #00f, #f00, #555, #f00); }

a { @include link-colors(
  #333,
  $hover: #00f,
  $active: #f00,
  $visited: #555,
  $focus: #f00);
}

// a {text-decoration: none;} a:hover{underline}
a {@include hover-link}

// 隐藏超链接
p.secret a,
p.secret a:hover,
p.secret a:focus {
color: inherit;
cursor: inherit;
text-decoration: inherit
}

p.secret a { @include unstyled-link }

// 列表标签
ul.features {
  @include pretty-bullets('pretty-bullet.png',
    $padding: 10px,
    $line-height: 22px)
}

ul.no-bullet { @include no-bullets}
li.no-bullet { @include no-bullet }

// 标题: default $padding is 4px. 
ul.nav { @include horizontal-list }
// For browsers that support :first-child and :last-child, we can omit the padding
// on the outside-facing edge of those elements.

// 列表一行展示, 用 ! 分割
ul.words { @include delimited-list("! ") }


// 超出自动省略
td.dot-dot-dot {
  @include ellipsis;
  }

td { @include nowrap }

// 图片替换文本
h1.coffee { @include replace-text("coffee-header.png") }
~~~~~~


# Layout helpers #

~~~~~~
@import "compass/layout"; .

@include sticky-footer(40px, "#content", "#footer", "#sticky-footer");

// 弹出窗口 绝对定位
a.login { @include stretch(5px, 5px, 5px, 5px) }


~~~~~~

# CSS3 #

~~~~~~
@import "compass/css3";
$experimental-support-for-opera: false;
$experimental-support-for-microsoft: false;
$experimental-support-for-khtml: false;
.notice {
  @include border-radius(5px);
}

.h2 {
  @include box-shadow(#ccc 5px 5px 2px);
  text-shadow: #ddd -1px 1px 0;
  background: #999;
  padding: 1em;
}


.motion {
  @include text-shadow(
   rgba(#000,.5) -200px 0 0,
   rgba(#000,.4) -400px 0 0,
   rgba(#000,.3) -600px 0 0,
   rgba(#000,.2) -800px 0 0
  );
  font-size: 2em;
  font-style: italic;
  text-align: right;
}
  
@import "compass";
@include font-face("ChunkFiveRegular",
font-files( "Chunkfive-webfont.woff", woff,
  "Chunkfive-webfont.ttf", ttf,
  "Chunkfive-webfont.svg", svg),
  "Chunkfive-webfont.eot", normal, normal);
~~~~~~

# Support for Internet Explorer with CSS PIE #

~~~~~~
//compass install compass/pie
@import "compass/css3/pie";
.pie-element {
  // relative is the default, so passing relative
  // is redundant, but we do it here for clarity.
  @include pie-element(relative);
}

.rounded {
  @include pie;
  @include border-radius(20px);
}

.gradient {
  @include pie;
  @include background(linear-gradient(#aaa, #333));
}

~~~~~~
# Sprite #

https://github.com/Keyamoon/IcoMoon-limited

`<map>` part is a placeholder and should be replaced with
the name of the folder containing your sprite images.

The all-sprites mixin will write all the necessary CSS
for the entire sprite map, whereas the second mixin will
output CSS for a single named sprite.
~~~~~~
@include all-<map>-sprites;
@include <map>-sprite($name);
~~~~~~


~~~~~~
@import "compass/utilities/sprites";
// generate sprite from images/icons/
@import "icons/*.png";

@include all-icons-sprites;
.add-button { @extend .icons-box-add; }

// 导入单个sprite, 不使用 @include all-icons-sprites
.add-button {
  @include icons-sprite(box-add);
}

$<map>-<property>: setting;
$<map>-<sprite>-<property>: setting;

// sprite map 中的间隔
$icons-spacing: 4px;
$icons-arrow-spacing:12px;

// 会在map中重复
$icons-arrow-repeat: repeat-x;

// 左边距  4px, arrow浮动到最右边
$icons-position: 4px;
$icons-arrow-position: 100%;
// 布局方式, 默认是 vertical
$<map>-layout: vertical/horizontal/diagonal/smart;

// configuring-automatic-sprites/layout.
$icons-layout: smart;
// 清理
$<map>-clean-up: true/false;
~~~~~~

## Customizing the sprite CSS ##

~~~~~~
@import "icons/*.png";
.next {
  @include icons-sprite(arrow);
  width: icons-sprite-width(arrow);
  height: icons-sprite-height(arrow);
}

.add-button {
  @include icons-sprite(box-add);
}

// 是否自动度量元素的高度和宽度, 会给容器自动添加 width, height
$<map>-sprite-dimensions: true/false;

$disable-magic-sprite-selectors: true/false;
~~~~~~

Magic sprite selectors are enabled by default, meaning Compass will automatically
output CSS `:hover` , `:active`, and `:target` pseudo selectors for sprites
with names ending in `_hover`, `_active`, or `_target`.

You add `arrow.png` and `arrow_hover.png` to your sprite folder.


帮助函数
~~~~~~
$icons: sprite-map("icons/*.png", $arrow-spacing: 5px);
$icons: sprite-map("icons/*.png", $arrow-spacing: 5px);
sprite($map, $sprite, [$offset-x], [$offset-y])

$icons: sprite-map("icons/*.png");
.next {
  background: sprite($icons, arrow) no-repeat;
}
.add-button {
  background: sprite($icons, box-add) no-repeat;
}

$icons: sprite-map("icons/*.png");
.sprite-base { background: $icons no-repeat; }
.next {
  @extend .sprite-base;
  background-position: sprite-position($icons, arrow);
}
.add-button {
  @extend .sprite-base;
  @include sprite-background-position($icons, box-add);
}
  
// 自动添加 width height
$icons: sprite-map("icons/*.png");
.sprite-base { background: $icons no-repeat; }
.next {
  @extend .sprite-base;
  @include sprite-background-position($icons, arrow);
  @include sprite-dimensions($icons, arrow);
}

~~~~~~

# production #
~~~~~~
# 配置文件中

# Increment the deploy_version before every
# release to force cache busting.
asset_cache_buster do |http_path, real_path|
"v=1"
end

asset_cache_buster :none

// 设置相对路径
relative_assets = true
~~~~~~

We encourage you to investigate using a rapid
prototyping framework like Serve (http://get-serve.com/)
or Middleman (http://middlemanapp.com/), which include
support for Sass and Compass out of the box.

## deploy ##

`compass compile --force -e production` 

~~~~~~
if environment == :production
  output_style = :compact
  end

// 部署时, 改变路径

http_path = '/my-app'
relative_assets = false
images_dir = 'images' #locally it's the images folder
http_images_dir = 'imgs' #on the webserver it's different
~~~~~~

添加版权信息
~~~~~~
$copyright-year: unquote("2012");
$company-name: unquote("Example, Inc.");
/*!
Copyright © #{$copyright-year}, #{$company-name}
All Rights Reserved.
*/
~~~~~~

~~~~~~
compass compile my_sass_dir/application.scss
sass --compass my_sass_dir/application.scss my_css_dir/application.css

~~~~~~

~~~~~~
# STAGING=true compass compile --force -e production
if ENV['STAGING']
  relative_urls = true
  output_style = :compact
elsif environment == :production
  relative_urls = false
  output_style = :compact
else #development
  relative_urls = true
  output_style = :expanded
end
~~~~~~

~~~~~~
// compass compile --force -c staging_config.rb -e production
eval(File.read("#{File.dirname(__FILE__)}/config.rb"))
relative_urls = true
output_style = :compact

on_stylesheet_save do |filename|
  # run the gzip tool on the file
  # generates a file of the same name
  # plus a .gz at the end.
  `gzip -f #{file}`
end

~~~~~~

# 网络优化 #

PNG is a complex format that can handle a range of image types. Be sure to remove
the alpha layer unless you need transparency. We highly recommend that you install
the free tool Pngcrush and run it on all your PNG images.
http://pmt.sourceforge.net/pngcrush/

Beyond the benefits of parallelization, it’s also important to set up your assets hosts
to use a cookieless domain—a domain that doesn’t share cookies with your site. This will
result in fewer bytes being sent to your web server with each image request.

~~~~~~
asset_host do |asset|
  host_number = (asset.hash % 4) + 1
  "http://img-#{host_number}.example.com"
end
~~~~~~

~~~~~~
background-image: inline-url("logo.gif");
*background-image: image-url("logo.gif");
~~~~~~

~~~~~~
gem install css_parser
compass stats
~~~~~~

# Scripting #

~~~~~~

$grid-cells: 20;
$cell-width: 25px;
#main {
  $main-width: $grid-cells * $cell-width;
  $main-padding: 10px;
  width: $main-width;
  padding: $main-padding;
  .siderbar {width: ($main-width - $main-padding*2)/4}


$pixels-per-em: 16px/1em;
5em * $pixels-per-em // 80px

1px/2px => 1px/2px; // dont work

// that's work
$var: 1px; $var/2px => 0.5px
(1px/2px) => 0.5px
1 + (1px/2px) => 1.5px


abs($number);
ceil($number);
comparable(13in, 4cm);
floor($number);
percentage(0.4); // 40%;
round($number); 
unit($number);
unitless($number);

// 颜色函数

alpha($color);
opacity($color);
lightness($color);
red($color);
greyscale($color);
invert($color);
miix($color-1, $color-2, [$weight]);
scale($color, $lightness: 30%);

// List Funciton
nth(foo bar baz, 2); // bar
join($list1, $list2, [$separator]);
length(1 2 3);


type-of($value);  // number string color bool list 

if($condition, $if-true, $if-false)

@function grid-width($cells) {
  @return ($cell-width + $cell-padding) * $cells;
}

@mixin thing($class, $prop) {
  .thing.#{$class} {
  prop-{$prop}: val;
  }
}

@mixin bang-hack($property, $value, $ie6-value) {
  #{$property}: $value !important;
  #{$property}: $ie6-value;
}
content: "This element is #{$color}";
width: calc(10% + #{$padding});
filter: progid:DXImageTransform.Microsoft.Alpha(
  Opacity=#{$opacity * 100}
);
~~~~~~

# 控制指令 #

~~~~~~
@for $i from 1 through 5 {
  .rating-#{$i} {
    background-image; url(/images/rating-#{$i}.png);
  }
}

// count backwards from 10 to 0
@for $i from 0 through 10 {
  $i: 10 - $i;
}
// count to 20 by twos
@for $i from 0 through 10 {
  $i: $i * 2;
}

@each $section in home, about, archive, project {
  nav .#{section} {
    background-image: url(/images/nav/#{$section}.png);
  }
}


@if $alpha < 0.2 {
  background-color: black;
} @else if $alpha < 0.5 {
  background-color: gray;
} @else {
  background-color: white;
}
~~~~~~
