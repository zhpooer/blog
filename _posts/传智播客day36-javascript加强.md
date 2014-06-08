title: 传智播客day36-javascript加强
date: 2014-05-22 14:24:14
tags:
- 传智播客
- javascript
---
# window对象常用方法 #

`window`中的对象在浏览器中可以随便使用, 如 `document`
~~~~~~
//标准输入框, 请输入您的数据
String prompt(String message, String default); 
alert(String message);
var sure = confirm("are you sure");
open("");  //打开一个窗口
close(); // 关闭窗口
~~~~~~

# Form 表单对象 #
访问表单的方式
* document.forms[n]
* document.表单名字

表单对象常用属性, 可以在 js 中获取或改变
* action
* method
* name

~~~~~~
document.forms[0].action; // 获取表单的 action 值
document.formname.method; // 获取表单的 method 值
~~~~~~

# 定义函数 #
~~~~~~
// 方式一, 构造函数方式
function printFormAciton() {
    document.forms[0].action
}
// 方式二, 函数构造函数
var printFormMethod = new Function("p1", "p2", "return p1+p2");
// 方式三, 匿名函数
var printForm = funciton(any) { alert(any) };

~~~~~~

# DOM 编程 #

DOM: Document Object Model, 文档对象模型
* DOM 是针对xml(html)的基于树的API
* DOM树: 节点(node)的层次
* DOM 把一个文档表示为一个家谱树(父 子 兄弟)
* DOM 定义了 Node 接口以及许多种节点类型来表示XML节点的多个方面

## 节点 ##
![domapi](/img/dom_op.png)

节点, 一切都是节点
* 整个HTML文档是一个文档节点
* 每个HTML标签都是一个元素节点
* 标签中的文字是文本节点
* 标签的属性是属性节点

### 查找操作 ###
~~~~~~
//搀着节点元素
var inputObj = document.getElementById("tid");
// 获取标签中的值
inputObj.value;


// 通过名字查找元素 getElementsByName()
var inputObjs = document.getElementsByName("tname");
// 该数据的长度
inputObjs.length;
// 遍历元素
for (var i=0;i<inputObjs.length;i++){
    inputObjs[i].value
}
// 为每个文本框增加change事件
for (var i=o;i<inputObjs.length;i++) {
    inputObjs[i].onchange = function(){
        alert(this.value);
    }
}

// 查找元素节点
// 获取所有的input元素, 返回值是数组
var inputObjs = document.getElementsByTagName("input");
inputObjs.length; // 元素长度
// 输出 type="text" 中value属性的值不包含按钮(button)
for(var i=0;i<inputObjs.length;i++) {
    if(inputObjs[i].type!="button") {
         inputObjs[i].value;
    }
}
//输出所有下拉选 id=edu 内容
var eduSelectObj = document.getElementById("edu");
var optionObjs = eduSelectObj.getElementsByTagName("option");
for(var i=0;i<optionObjs.lenght;i++) {
    alert(optionObjs[i].value);
}
// 输出选中的值
log(eduSelectObj.value); // 方式一
方式二
for(var i=0;i<optionObjs.lenght;i++) {
    if(optionObjs[i].selected==true) log(optionObjs[i].value);
}

// 查看是否存在子节点
// 文本节点, 和属性节点不可能包含子节点,
// 如果 hasChildNodes() 方法返回 false,
// 则 childNodes() firstChild(), lastChild() 将是空数组和空字符串
log(document.getElementById("edu").hasChildNodes());

// nodeName, 一个字符串, 内容是给定节点的名字 node.nodeName
// 元素节点, 返回这个元素的名称
// 属性节点, 返回属性名称
// 文本节点, 返回 内容为 #text 的字符串
document.getElementById("").nodeName;
// 返回整数, 1表示: 元素节点, 2表示: 属性节点, 3表示: 文本节点
document.getElementById("").nodeType;
// 给定节点当前值, 元素节点 返回 null
document.getElementById("").nodeValue;

// 属性节点获取
var pObj = document.getElementById("");
pObj.getAttributeNode("name");
~~~~~~

### 更新操作 ###
~~~~~~
// 替换操作
var bjElement = document.getElementById("1");
bjElement.onclick = function () {
    var fkElement = document.getElementById("2");
    bjElement.parentNode.replaceChild(fkElement);
}
// 设置节点
bjElement.setAttribute("name", "xxxxx");

// 创建节点
var btElement = document.getElementById("bt1");
var tjElement = document.createElement("li");
tjElement.setAttribute("id", "tj");
tjElement.setAttribute("value", "tianjing");
var textNode = document.createTextNode("天津");
tjElement.appendChild(textNode);
var cityElement = document.getElementById("city");
cityElement.appendChild(tjElement);
// 在bjElement 之前插入
cityElement.insertBefore(bjElement, tjElement);

// 删除节点
var tjElm = cityElement.removeChild(tjElement);

// innerHTML, 所有浏览器都支持, 但不是DOM标准
 var divElm = document.getElementById("div1");
divElm.innerHTML = "<h1>今天</h1>";

// table 的 appendChild
var table = document.getElementById("tablename");
table.appendChild(child); // 在ie中可以, firefox不可以
table.tbody.appendChild(child); // 在firefox中可以, 但是 ie 不可以
// 解决方法
// table 中是有tbody, 不管什么情况, 都要创建一个tbody并加入到table中把tr接到 tbody上
var tbodyElm = document.createElement("tbody");
tbodyElm.appendChild(trElm);
table.appendChild(tbodyElm);

//操作节点数组
var optionChildren = selectElm.getElementsByTagName("option");
// 等同于 selectElm.selectedIndex
for(var i=0;i<optionChildren.length;i++) {
    // 错误, 不能这么写, 因为数组是可变的
    selectElm.removeChild(optionChildren[i]); 
}

~~~~~~


### 案例: 列表联动 ###
xml数据: 
~~~~~~
<china>
    <provice name="吉林省">
        <city>1 </city>
        <city>2 </city>
        <city>3 </city>
    </provice>
    <provice name="辽宁省">
        <city> </city>
    </provice>
</china>
~~~~~~
Html结构: 
~~~~~~
<select id="provice" name="provice">
    <option value="">请选择</option>
    <option value="吉林省">吉林省</option>
</select>
<select id="city" name="city">
    <option value="">请选择</option>
</select>
~~~~~~
javascript:
~~~~~~
fuunction parseXML(){
    try //Internet Explorer {
        xmlDoc=new ActiveXObject("Microsoft.XMLDOM");
    } catch(e) {
        try //Firefox, Mozilla, Opera, etc. {
            xmlDoc=document.implementation.createDocument("","",null);
        } catch(e) {
            alert(e.message);
            return;
        }
    }
    xmlDoc.async=false;
    xmlDoc.load();
    return xmoDoc;
}
document.getElementById("province").onchange = function(){
    // 要先清理 city 选择框
    document.getElementById("city").innerHTML = "";
    var xmlDoc = parseXML();
    var selectedProvice = this.value;

    var xmlProvice;

    var xmlProviceElm = xmlDoc.getElementByTagName("provice");
    for(var i=0;i< xmlProviceElm.length;i++) {
        if( selectedProvice == xmlProviceElm[i].getAttribute("name")) {
            xmlProvice = xmlProviceElm[i];
        }
    }
    if(xmlProviceElm != null) {
        var xmlCities = xmlProvice.getElemntByTagName("city");
        for(var i=0;i < xmlCities.length; i++) {
            var optionElm = document.createElement("option");
            var textNode = optionElm.setAttribute("value", xmlCities[i].firstChild.nodeValue);
            optionElm.appendChild(textNode);
            document.getElementById("city").appendChild(optionElm);
        }
    }
}
~~~~~~

### 弹出窗口 ###
~~~~~~
function openWindow(){
    // 方式一: 
    /* window.showModalDialog(sUrl[, vArguments],[,sFeature])
    * 只针对ie 有用
    * sUrl: 要打开窗口的页面, 可以使用绝对路径, 相对路径
    * vArguments: 窗口传递的参数
    * sFeature: 其他窗口参数, 可以查看文档
    */
    // 将window窗口对象传过去
    window.shwoModalDialog("a2.html", window, {help:no;status:no;});
    
    // 方式二:
    /*
    * newwindow = window.open(sUrl [, sName][, sName][, sFeature][,breplace]);
    * sUrl: 要打开的窗口的页面
    * sName: 窗口的位置: _blank 新窗口, _self 当前窗口
    * sFeature: 窗口样式, 可以查看文档
    */
    window.open("a2.html", "_blank", "toolbar=no, status=no")
}

function viewData(pid, pname) {
    // 和方式一交互
    var parentWindow = window.dialogArguments; // 得到传过来的 window
    // 和方式二交互
    var parentWindow = window.openner();
    
    // 可以对 parentWindow 进行操作, 或函数调用
    window.close();
}
~~~~~~


# js中的逻辑判断 #
在运算逻辑中, 0, "", false, null, undefined, NaN 均为 false


# 使用小结 #
* 若应用程序不需要与其他应用程序共享数据的时候,
使用 HTML 片段来返回数据时最简单的
* 如果数据需要重用, JSON 文件是个不错的选择,
其在性能和文件大小方面有优势
* 当远程应用程序未知时, XML 文档是首选,
因为 XML 是 web 服务领域的 “世界语”
