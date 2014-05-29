title: 传智播客day39-JQuery Dom
date: 2014-05-25 16:26:53
tags:
- 传智播客
- jquery
---


# 内部插入节点 #
* append(content) :向每个匹配的元素的内部的结尾处追加内容
* appendTo(content) :将每个匹配的元素追加到指定的元素中的内部结尾处
* prepend(content):向每个匹配的元素的内部的开始处插入内容
* prependTo(content) :将每个匹配的元素插入到指定的元素内部的开始处

~~~~~~
$("#city").append($("#fk"))
~~~~~~

# 外部插入节点 #
* after(content) :在每个匹配的元素之后插入内容 
* before(content):在每个匹配的元素之前插入内容 
* insertAfter(content):把所有匹配的元素插入到另一个、指定的元素元素集合的后面 
* insertBefore(content) :把所有匹配的元素插入到另一个、指定的元素元素集合的前面

以上方法不但能将新创建的 DOM 元素插入到文档中, 也能对原有的 DOM 元素进行移动.

# 查找节点 #
* 查找属性节点: 通过 jQuery 选择器完成.
* 查找属性节点: 查找到所需要的元素之后, 可以调用 jQuery 对象的 attr() 方法来获取它的各种属性值

~~~~~~
$("img").attr("name"); // 获取 name 属性
~~~~~~

# 创建元素 #
使用 jQuery 的工厂函数 `$(html);`
会根据传入的 html 标记字符串创建一个 DOM 对象,
并把这个 DOM 对象包装成一个 jQuery 对象返回.

~~~~~~
$().ready( function(){
    var $sh = $("<li></li>");
    // 方式一:
    $sh.attr("id", "sh");
    $sh.attr("name", "shanghai");
    // 方式二:
    $sh.attr({id:'sh', name:'shanghai'});
    
    $sh.text("上海");
    $sh.appendTo($("#city"))
});
~~~~~~

# 删除节点 #

* `remove()`: 从 DOM 中删除所有匹配的元素,
传入的参数用于根据 jQuery 表达式来筛选元素.
当某个节点用 remove() 方法删除后,
该节点所包含的所有后代节点将被同时删除.
这个方法的返回值是一个指向已被删除的节点的引用.
* `empty()`: 清空节点 – 清空元素中的所有后代节点(不包含属性节点).

~~~~~~
$("#bj").remove();
$("#city").empty();

$("<td></td>").append($("<a></a>").text("删除"));

$("a").click(function(){
    $(this).parent().parent().empty();
})
~~~~~~

# 复制节点 #
* clone(): 克隆匹配的 DOM 元素, 返回值为克隆后的副本. 但此时复制的新节点不具有任何行为.
* clone(true): 复制元素的同时也复制元素中的的事件 

# 替换节点 #
* replaceWith(): 将所有匹配的元素都替换为指定的 HTML 或 DOM 元素
* replaceAll(): 颠倒了的 replaceWith() 方法.

注意: 若在替换之前, 已经在元素上绑定了事件, 替换后原先绑定的事件会与原先的元素一起消失
~~~~~~
$("p").replaceWith("<button></button>");
$("<button></button>").replaceAll("p");
~~~~~~

# 属性操作 #
* attr(): 获取属性和设置属性
  * 当为该方法传递一个参数时, 即为某元素的获取指定属性
  * 当为该方法传递两个参数时, 即为某元素设置指定属性的值
jQuery 中有很多方法都是一个函数实现获取和设置.
如: attr(), html(), text(), val(), height(), width(), css() 等.

removeAttr(): 删除指定元素的指定属性

~~~~~~
<select id="single">
  <option>Single</option>
  <option>Single2</option>
</select>
<select id="multiple" multiple="multiple">
  <option selected="selected">Multiple</option>
  <option>Multiple2</option>
  <option selected="selected">Multiple3</option>
</select><br/>
<input type="checkbox" value="check1"/> check1
<input type="checkbox" value="check2"/> check2
<input type="radio" value="radio1"/> radio1
<input type="radio" value="radio2"/> radio2
~~~~~~
~~~~~~
$("#single").val("Single2");
$("#multiple").val(["Multiple2", "Multiple3"]);
$("input").val(["check2", "radio1"]);

$(“div”).html(“<p>奥运接受了</p>");
$(“div”).html();
~~~~~~

# 样式操作 #
* 获取 class 和设置 class : class 是元素的一个属性,
所以获取 class 和设置 class 都可以使用 attr() 方法来完成.
* 追加样式: addClass() 
* 移除样式: removeClass(), 从匹配的元素中删除全部或指定的 class
* 切换样式: toggleClass(), 控制样式上的重复切换.如果类名存在则删除它,
如果类名不存在则添加它.
* 判断是否含有某个样式: hasClass(), 判断元素中是否含有某个 class,
如果有, 则返回 true; 否则返回 false

~~~~~~
$.trim("  text ")

$("h1").each(function(){
    $(this).click(function(){
        $(this).slideToggle("nomal");
    });
});
~~~~~~

# CSS DOM #

* 获取和设置元素的样式属性: css()
* 获取和设置元素透明度: opacity 属性, IE6,IE7不支持此属性
* 获取和设置元素高度, 宽度: height(), width().
在设置值时, 若只传递数字, 则默认单位是 px.
如需要使用其他单位则需传递一个字符串, 例如 $(“p:first”).height(“2em”);
* 获取元素在当前视窗中的相对位移: offset().
其返回对象包含了两个属性: top, left. 该方法只对可见元素有效

em是相对长度单位。相对于当前对象内文本的字体尺寸 


# Jquery 验证框架 #
~~~~~~
<script type="text/javascript" src="jquery.js"/>
<script type="text/javascript" src="jquery-validate.js"/>

~~~~~~
