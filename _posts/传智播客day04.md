title: 传智播客day04
date: 2014-04-03 08:55:12
tags:
- 课堂笔记
- javascript
- DOM
---
# javascript #

## prototype ##

prototype 是一个类的属性，是类的一个属性,
是类的一个实例，也就是说当一个类被声明后，js自动创建，
并且作为类的prototype属性而存在,称作类的*原型*

~~~~~
function Person(){
}
log(Person.prototype) // Person(){}
var p = new Person();
log(p.prototype); // undefined
log(console.log(p.__proto__==Person.prototype); // true

Person.prototype.name = "123"; // readonly
log(p.name); // 123

p.name = 124;
log(p.name); // 124

~~~~~

在扩展共用属性，用prototype；在应用个性化的属性时，用构造函数

## DOM ##
dom: 文档对象模型，是一套规范

dom 定义了哪些对象，这些对象有哪些方法和属性

~~~~~~
window.onload = function (){
    document.getElementsByTagName("div") // 获得所有的div
    document.getElementsByName("div")[0]  //获得集合，name值不唯一如'<input type="radio/>"'
    var div1 = document.getElementById() // 获得的引用，代表对象
    log(div1.innerHTML)
}
~~~~~~

~~~~~~
<div></div> // 代表标签
~~~~~~

*标签和标签之间也都是文本*
/* 拷贝老师 */！！！！！

~~~~~
<!-- 如何获得div3 -->
<table>
    <tr> <td id="td1">
            <div>div3</div>
    </td> </tr>
</table>
~~~~~
~~~~~
var td1 = document.getElementById("td1")
td1.getElementsByTagName("div")[0] // 所有容器对象，都有get*方法
~~~~~
> `document.forms` // 获得页面上所有的form

### 对象的属性和方法 ###

nodeName: 节点名称
>    元素节点，标签名

>    属性节点，属性名

>    文本节点，#text

nodeValue: 节点值
>    元素节点，null

>    属性节点，属性值

>    文本节点，文本内容

nodeType: 节点类型
>    元素节点，1

>    属性节点，2

>    文本节点，3

### DEMO ###
~~~~~
<ul>
    <li id="bj">
      北京
      <p>海淀</p>
      奥运
    </li>
</ul>

~~~~~
~~~~~
var li = document.getElementById("bj");
console.debug(li.childNodes);
for (var i=0;i<li.childNodes.length;i++){
  var child = li.childNodes[i];
  if (child.nodeType == 3)
    console.log(child.nodeValue);
  else
    console.log(child.innerHTML);
}
~~~~~
### DEMO2 ###
打印明天你好
~~~~~
<h1 id="h1">明天休息</h1>
~~~~~

~~~~~
var h1 = document.getElementById("h1");
console.log(h1.innerHTML);
console.log(h1.firstChild.nodeValue);
console.log(h1.lastChild.nodeValue);
console.log(h1.childNodes[0].nodeValue);
~~~~~

### DEMO3 ###
打印option中的内容
~~~~~~
<select id="select">
    <option> aaa </option>
    <option>bbb </option>
    <option>ccc </option>
    <option>dddd </option>
</select>
~~~~~~

~~~~~~
var options = document.getElementById("select").childNodes;

for(var i=0;i<options.length;i++){
  if(options[i].nodeType == 1)
    console.log(options[i].innerHTML);
}
~~~~~~

### 创建节点 ###
~~~~~~
var tr = document.createElement("tr");
tr.innerHTML = "<td>content</td>"
~~~~~~
~~~~~~
<a href='javascript:void(0)' onclick='onClick(this)'/>
~~~~~~
`所有的input元素都通过value来获得其值`

### string 转换 ###
~~~~~
String.prototype.trim = function (){ return ""}
"aa  ".trim(); // 字符串虽然是基本数据类型，但是会自动转换String对象

~~~~~

~~~~~
<form onsubmit=“return false;”></form> // not work
~~~~~
## BOM ##
bom 浏览器对象模型

1 window, 顶级元素，每一个窗口都有一个window对象

javascript所有定义的全局变量和函数都是window对象的

2 frame

3 location
> window.location.href = "http://url"

4 history

