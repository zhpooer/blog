title: 传智播客Day02-html&css
date: 2014-03-31 08:48:34
tags:
- 传智播客
- html
- css
- javascript
---

html剩余
=========

## div span p ##

div: 块标签，主要用来封装数据，配合css使用

span: 行级标签，只占据一行中被封装数据的大小，主要用来封装数据

p:段落

css
=========
`CSS: 层叠样式表，用来美化和修饰页面，不区分大小写`

## css封装 ##

1.通过sytle标签属性，给其指定样式。`属性名:属性;属性名:属性;`
~~~~~
<div style="color:red;font-size:24px;background-color: #9966cc;"></div>
~~~~~
2.把样式代码封装到style标签当中
~~~~~
<head>
    <style text="text/css">
        div{color:red;font-size:24px;background-color: #9966cc;}
        span{color:red;font-size:24px;background-color: #9966cc;}
    </style>
</head>
~~~~~
3.把样式代码提取到一个单独的文件
~~~~~css:
/* url.css */
div{color:red;font-size:24px;background-color: #9966cc;}
span{color:red;font-size:24px;background-color: #9966cc;}
~~~~~
~~~~~css:
<style type="text/css">
@import url(url.css)
</style>
<!-- rel：引入的外部文件和当前页面的关系 -->
<link rel="stylesheet" type="text/css" href="url.css"/>
~~~~~
~~~~~css:
/* total.css */
@CHARSET "UTF-8";
@import url(1.css)
@import url(2.css)
~~~~~

## 选择器 ##
选择器： 用来确定被（操作）的标签的一种语法
1. 标签名称选择器,
`div{color:red}` or `span`
2. 类选择器,
`.class-name{color:red}`
`<div class="class-name"/>`
3. id选择器，
`#div-id{color:red}`
`<div id="div-id"/>`

### 选择器优先级 ###
`style > id > class > name`

### 组合选择器 ###
共用多个样式代码
`selector1, selector2 ... {color:red}`

### 关联选择器 ###
逐渐缩小范围，确定被操作元素
`parent child {color:red}`

### 伪类选择器 ###
要有顺序： lvha
~~~~~~css:
/* :link, visited,:hover, :active */
selector:active { color: red }
~~~~~~

## css定位模型 ##
1. 基本流模型
2. 浮动流模型定位：脱离基本流，按照指定方向浮动，
遮挡时文字环绕在周围
~~~~~~css:
/* 浮动流 */
div { float: left; margin: 8px;}
~~~~~~

## css盒模型 ##
* margin: 外边距
```
/* 上右下左*/
margin: 10px 0px 10px 0px
```
* padding: 内边距
* border: 边框
> border-width, border-color, border-style
>
> total_width or height = padding + magin + border + width

## 位置模型 ##

按照我们指定的位置显示相关的标签
> position: static | absolute | fixed | relative

relative: 没有脱离基本流，相对于自身原来位置,*相对于第一个包含的相对定位或绝对定位*

static: 默认值，按基本流排列

absolute: 脱离基本流，*相对于第一个包含的相对定位或绝对定位*

fixed:　相对于窗口？

z-index: 决定谁在最顶端

> display: none | block `/* 显示隐藏标签 */`

~~~~~~
position: absolute; top:20px; left:20px; z-index:100;
~~~~~~

## DEMO ##

~~~~~~html:
<body>
  <div>
    <img src="xiaobao.png"/>
    Here’s an example that shows an application area for which Scala is particularly well
suited. Consider the task of implementing an electronic auction service. We use
an Erlang-style actor process model to implement the participants of the auction.
Actors are objects to which messages are sent. Every actor has a “mailbox” of its in-
coming messages which is represented as a queue. It can work sequentially through
the messages in its mailbox, or search for messages matching some pattern.
For every traded item there is an auctioneer actor that publishes information about
the traded item, that accepts offers from clients and that communicates with the
seller and winning bidder to close the transaction. We present an overview of a
simple implementation here.
  </div>
  <div id="div1"></div>
  <div id="div4">
      <div id="div2"></div>
  </div>

  <div id="div3"></div>
</body>
~~~~~~
~~~~~~css:
div{
    width:600px;
    border: solid 9px red;
    margin: 11px;
    padding: 15px;
}
img {
    float: left;
    margin: 8px;
}
#div1 {
    width: 40px;
    height: 40px;
    position: static;
    top: 50px;
    right: 20px;
    background-color: yellow;
}
#div2 {
    width: 40px;
    height: 40px;
    top: 0px;
    right: 5px;
    position: relative;
    background-color: green;
}
#div3 {
    width: 40px;
    height: 40px;
    top: 50px;
    right: 20px;
    position: static;
    background-color: blue;
}
#div4 {
    top: 50px;
    left: 20px;
    position: absolute;
    background-color: purple;
}
~~~~~~


# javascript #

**javascript是运行在浏览器中的一门语言, 可以动态操作页面上的元素**

## 嵌入浏览器 ##
1. 把代码放到script当中
~~~~~~html:
<script type=“text/javascript">
/* code */
</script>
~~~~~~
2. 从外部引入
~~~~~~javascript:
<script type=“text/javascript" src="*.js"/> // 外部文件的编码格式，和html的编码格式要一致
~~~~~~
3. 写在标签里
~~~~~~html:
<a href="javascript: alert('hello world')"/>
~~~~~~

## javascript核心语法 ##

### 定义变量 ###
变量的类型由给定的值来确定  `var name = "string"`

没赋值，为 undefined. `var name; // name为undefined`

```
demo = "else" //不声明亦可, 有什么副作用?
alert(demo)

result=3*"a" // NaN
result=3*"2" //  6 , 自动类型转换
0 null NaN　undefined　 // 都表示假，其他表示真

```

> 基本数据类型：string, boolean(true|false), number, null, undefined  
> 应用数据类型: object, function
