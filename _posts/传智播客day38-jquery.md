title: 传智播客day38-JQuery
date: 2014-05-24 11:17:29
tags:
- 传智播客
- JavaScript
---

当前流行的 JavaScript 库有:
jQuery, MooTools, Prototype, Dojo, YUI, EXT_JS, DWR(用java代码写)

# JQuery 初识#

轻量级js库, 压缩后只有 21K

~~~~~~
var inputObj = document.getElementById("uname");
log(inputObj.value)
// 等同于, 用 jQuery
log(#("#username").val());

// 文档全部加载完毕后执行, 相当于 window.onload = function(){}
// 方式一
$(document).ready(function(){ /*todo*/ });
// 方式二
$().ready(function(){});
~~~~~~

# JQuery 对象 #

* jQuery 对象就是通过jQuery包装DOM对象后产生的对象
* jQuery 对象是 jQuery 独有的. 如果一个对象是 jQuery 对象,
那么它就可以使用 jQuery 里的方法: `$(“#test”).html()`
* 虽然jQuery对象是包装DOM对象后产生的，但是jQuery无法使用DOM对象的任何方法，
同理DOM对象也不能使用jQuery里的方法.乱使用会报错
* 约定：如果获取的是 jQuery 对象, 那么要在变量前面加上 `$`
~~~~~~
var $variable = jQuery 对象
var variable = DOM 对象
~~~~~~

## DOM对象和JQuery对象互转 ##
~~~~~~
// DOM 转换为 JQuery 对象
var inputObj = document.getElementById("username");
var $obj = $(inputObj);
log($obj.val())

// JQuery -> DOM对象

var $inputObj = $("#username");
// 方式一: JQuery 是一个数组对象, 可以通过[index], 得到 DOM 对象
var inputObj = $inputObj[0];
// 方式二:
var domObj = $inputObj.get(0);
~~~~~~

# JQuery 选择器 #

对事件处理, 遍历 DOM 和 Ajax 操作都依赖于选择器

优点:
* 简洁的写法
~~~~~~
$("#id") == document.getElementById("id")
$("tagname") == document.getElementsByTagName("tagname")
~~~~~~
* 完善的事件处理机制

## 基本选择器 ##

基本选择器是 jQuery 中最常用的选择器, 也是最简单的选择器,
它通过元素 id, class 和标签名来查找 DOM 元素
(在网页中 id 只能使用一次, class 允许重复使用).

|  选择器   | 用法    |  返回值   |  说明   | 
|------------------------|
|   #id     |  $(”#myDiv”); |   单个元素的组成的集合 | 这个就是直接选择html中的id=”myDiv”|
| Element   |   $(”div”)  |  集合元素 | 例如 div, input, a等等.|
|  class    |  $(”.myClass”) |  集合元素 | 直接选择html代码中class=”myClass”的元素或元素组(因为在同一html页面中class是可以存在多个同样值的)|
| *         |   $(”*”)   |   集合元素 | 匹配所有元素,多用于结合上下文来搜索 |
| selector1, selector2, selectorN |$(”div,span,p.myClass”)    |  集合元素 |  将每一个选择器匹配到的元素合并后一起返回 |


~~~~~~
$("#b1").click(function(){
    // 改变id为one的元素的背景色
    $("#one").css("background-color", "red");
    $("span, #one").css("background-color", "red");
})
~~~~~~

## 层次选择器 ##
如果想通过 DOM 元素之间的层次关系来获取特定元素,
例如后代元素, 子元素, 相邻元素, 兄弟元素等, 则需要使用层次选择器.

|  选择器    |   用法   |   说明 |
|--------------|
|ancestor descendant | $(”form input”) | 在给定的祖先元素下匹配所有*后代元素* |
| parent > child |  $(”form > input”)  | 在给定的父元素下匹配所有*子元素* |
| prev + next | $(”label + input”) | 匹配所有紧接在 prev 元素后的 next 元素 (单个)|
| prev ~ siblings  | $(”form ~ input”)  |  匹配 prev *元素之后*的所有 siblings 元素 |

~~~~~~
// id 为 two的元素的所有 <div> 兄弟元素
$("#two").siblings(".div").css("background-color", "red");
~~~~~~

## 过滤选择器 ##
过滤选择器主要是通过特定的过滤规则来筛选出所需的 DOM 元素,
该选择器都以 “:” 开头

按照不同的过滤规则, 过滤选择器可以分为基本过滤, 内容过滤,
可见性过滤, 属性过滤, 子元素过滤和表单对象属性过滤选择器.

|  选择器    |   用法   |   说明 |
|--------------|
| :first | $(”tr:first”) | 匹配找到的第一个元素 |
| :last  |  $(”tr:last”) | 匹配找到的最后一个元素.与 :first 相对应 |
| :not(selector) | $(”input:not(:checked)”) | 意思是没有被选中的input(当input的type=”checkbox”)|
| :even  | $(”tr:even”) | 匹配所有索引值为偶数的元素，从 0 开始计数 |
| :odd  | $(”tr:odd”)  | 匹配所有索引值为奇数的元素,和:even对应 |
| :eq(index) | $(”tr:eq(0)”)| 匹配一个给定索引值的元素.eq(0) 括号里面不是元素排列数.|
| :gt(index) | $(”tr:gt(0)”) | 匹配所有大于给定索引值的元素 |
| :lt(index) | $(”tr:lt(2)”) | 匹配所有小于给定索引值的元素 |
| :header(固定写法) |  $(”:header”).css(”background”, “#EEE”) | 匹配如 h1, h2, h3之类的标题元素.这个是专门用来获取h1,h2这样的标题元素.|
| :animated(固定写法)|  | 说明: 匹配所有正在执行动画效果的元素 |

~~~~~~
$("div:first").css("background-color", "red"); // == $("div").first()
$("div:not(.one)").css("background-color", "red");
~~~~~~

## 内容过滤选择器 ##

|  选择器    |   用法   |   说明 |
|-------------------|
| :contains(text) | `$(”div:contains(’John’)”)` | 匹配包含给定文本的元素 |
| :empty          | `$(”td:empty”)`| 匹配所有不包含子元素或者文本的空元素 |
| :has(selector)  | `$(”div:has(p)”).addClass(”test”)` | 匹配含有选择器所匹配的元素的元素, 给所有包含p元素的div标签加上class=”test”.|
| :parent         | `$(”td:parent”)` | 匹配含有子元素或者文本的元素| 

~~~~~~
// 不含有 文本 'di' 的元素
$('div:not(:contains('di'))')
$('div:not(:empty)') // $('div:parent')
~~~~~~


## 可见度过滤选择器 ##

|  选择器    |   用法   |   说明 |
|--------------|
| :hidden    | $(”tr:hidden”)   | 匹配所有的不可见元素(也包括不可见的孩子们)，意思是css中display:none和input type=”hidden”的都会被匹配到.|
| :visible   |  $(”tr:visible”) | 匹配所有的可见元素 |

~~~~~~
/*
index: 当前遍历的索引
domElm: 当前遍历的DOM对象
*/
$("div:hidden").each( function(index, domElm){
    $(this).show();  //相当于 domElm
});
~~~~~~


## 练习 ##
~~~~~~
// 给网页中所有 <p> 元素添加 onclick 事件, 打印文本 html()/text()
$("p").each( function(){
    $(this).click( function() {
        $(this).html();
        $(this).text();
    });
});
// 使一个特定表格隔行变色
$("table:eq(0) tr:even").css("background-color", "red");
$("table:eq(0) tr:odd").css("background-color", "yellow");
~~~~~~

## 属性过滤选择器 ##
|  选择器    |   用法   |   说明 |
|--------------------|
| [attribute]         |  $(”div[id]“)                   | 选取了所有带”id”属性的div标签    |
| [attribute=value]   |  $(”input[name='newsletter']“)  | 匹配给定的属性是某个特定值的元素    |
| [attribute!=value]  |  $(”input[name!='newsletter']“) | 匹配所有不含有指定的属性，或者属性不等于特定值的元素 |
| [attribute^=value]  |  $(”input[name^=‘news’]“)   | 匹配给定的属性是以某些值开始的元素 |
| [attribute$=value]  |  $(”input[name$=‘letter’]“) | 匹配给定的属性是以某些值结尾的元素 |
| [attribute* =value] |  $(”input[name*=‘man’]“)    | 匹配给定的属性是以包含某些值的元素  |
| [attributeFilter1][attributeFilter2][attributeFilterN] | $(”input[id][name$=‘man’]“) | 复合属性选择器,需要同时满足多个条件时使用 | 

## 子元素过滤选择器 ##
|  选择器    |   用法   |   说明 |
|--------------|
| :nth-child(index/even/odd/equation)      | $(”ul li:nth-child(2)”)   | 匹配其父元素下的第N个子或奇偶元素, 从1开始 |
| :first-child  |  $(”ul li:first-child”)  | 匹配第一个子元素.’:first’ 只匹配一个元素,而此选择符将为每个父元素匹配一个子元素 |
| :last-child   |  $(”ul li:last-child”)   | 匹配最后一个子元素.’:last’ 只匹配一个元素,而此选择符将为每个父元素匹配一个子元素       |
| :only-child   |  $(”ul li:only-child”)   | 如果某个元素是父元素中唯一的子元素,那将会被匹配.如果父元素中含有其他元素,那将不会被匹配 |

nth-child() 选择器详解如下：
* :nth-child(even/odd): 能选取每个父元素下的索引值为偶(奇)数的元素
* :nth-child(2): 能选取每个父元素下的索引值为 2 的元素
* :nth-child(3n): 能选取每个父元素下的索引值是 3 的倍数 的元素
* :nth-child(3n + 1): 能选取每个父元素下的索引值是 3n + 1的元素

~~~~~~
// 每个class为one的div父元素下的第一个子元素
$("div[class='one'] :first-child")
~~~~~~
## 表单对象属性过滤选择器 ##
|  选择器    |   用法   |   说明 |
|--------------|
| :enabled   |  $(”input:enabled”)     | 匹配所有可用元素 |
| :disabled  |  $(”input:disabled”)    | 匹配所有不可用元素 |
| :checked   |  $(”input:checked”)     | 匹配所有选中的被选中元素(复选框、单选框等，不包括select中的option) |
| :selected  |  $(”select option:selected”)   | 匹配所有选中的option元素 |

如果一个`<input disabled="true"/>`, 提交的时候, 不会提交,
想让他提交 `<input readonly="readonly"/>`

## 表单选择器 ##

|  选择器    |   用法   |   说明 |
|-------------------|
| :input  |  $(”:input”)  | 匹配所有 input, textarea, select 和 button 元素  |
| :text  |  $(”:text”)    | 匹配所有的单行文本框  |
| :password  |  $(”:password”)  | 匹配所有密码框 |
| :radio  |  $(”:radio”)        | 匹配所有单选按钮 |
| :checkbox  |  $(”:checkbox”)  | 匹配所有复选框 |
| :submit  |  $(”:submit”)      | 匹配所有提交按钮 |
| :image  |  $(”:image”)        | 匹配所有图像域 |
| :reset  |  $(”:reset”)        | 匹配所有重置按钮 |
| :button  |  $(”:button”)      | 匹配所有按钮.这个包括直接写的元素button |
| :file  |  $(”:file”)          | 匹配所有文件域.|
| :hidden  |  $(”input:hidden”) | 匹配所有不可见元素, 或者type为hidden的元素.这个选择器就不仅限于表单了, 除了匹配input中的hidden外,那些style为hidden的也会被匹配 |
