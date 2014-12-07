title: BOM&DOM 基本语法
date: 2014-11-23 18:51:30
tags:
---

# BOM #

~~~~~~
var wroxWin = window.open("http://www.wrox.com",
     "wroxWindow",
     "height=400, width=400, top=10, left=10, resizable=yes")
wroxWin.resizeTo(500, 500);
wroxWin.moveTo(100, 100);
wroxWin.close();
log(wroxWin.closed);
alert(wroxWin.opener == window);

// Timeout
var timeoutId = setTimeout(function(){
  alert();
}, 1000);
clearTimeout(timeoutId);

// interval
var intervalId = setInterval(doFunc, 500);
clearInterval(intervalId);

// prompt
var result = prompt("What is your name?", "")
if(result !== null){}
~~~~~~

## location 对象 ##

~~~~~~
log(window.location == document.location); // true

location.hash; // #contens
location.host; // www.wrox.com:80
location.hostname; // www.worx.com
location.href; // http://www.wrox.com/ab/
location.pathname; // /WileyCDA
location.port;  // 80
location.protocol; // http
location.search; // ?q=javascript

// 每次修改属性, 都会重新加载
location.assign("http://www.baidu.com")
window.location = "httt://www.baidu.com"
location.href = "http://www.baidu.com"

location.replace(""); // 浏览器不会记录历史
location.reload(); // 可能从缓存中加载
location.reload(true); // 从服务器中加载
~~~~~~


## navigator ##

检测插件
~~~~~~
function hasPlugin(name) {
  name = name.toLowerCase();
  for( var i=0, i< navigator.plugin.length; i++) {
    if(navigator.plugin[i].name.toLowerCase().indexOf(name) > -1) {
      return true
    }
  }
  return false;
}

function hasIEPlugin(name) {
  try {
    new activeXObject(name);
    return true;
  } catch (ex) {
    return false;
  }
}

function hasFlash() {
  var result = hasPlugin("Flash");
  if (!result) {
    result = hasIEPlugin("ShockwaveFlash.ShockwaveFlash");
  }
  return result;
}
~~~~~~

## Screen 对象 ##

screen 对象基本上只用来表示客户端的能力,
其中包括浏览器窗口外部的显示器的信息, 如像素宽度和高度等.

~~~~~~
window.resizeTo(screen.availWidth, screen.availHeight);
~~~~~~

## history 对象 ##

~~~~~~
history.go(-1); // 后退一页
history.go(1);
history.go(2); // 前进二页

history.back();
history.forward();

if(history.length == 0 ){} // 用户打开的第一个页面

history.go("wrox.com"); // 跳转到最近的 wrox.com 的页面

~~~~~~

# DOM #

~~~~~~
var returnedNode = someNode.appendChild(newNode);
log(returnedNode == newNode); // true

returnedNode = someNode.insertBefore(newNode, null);
log(returnedNode == someNode.lastChild); // true

returnedNode = someNode.insertBefore(newNode, someNode.firstChild);
log(returnedNode == someNode.firstChild); // true

returnedNode = someNode.insertBefore(newNode, someNode.lastChild);
log(newNode == someNode.childNodes[someNode.childNodes.length - 2]);

// 替换
var returnedNode = someNode.replaceChild(newNode, someNode.firstchild);

var formerFirstChild = someNode.removeChild(someNode.firstChild);

var html = document.documentElement;
html == docuemnt.childNodes[0];
html == docuemnt.firstChild;

var body = document.body;
var doctype = document.doctype;

// 取得完整的URL
var url = document.URL;

// 取得域名
var domain = document.domain;
// 获得来源域名的URL
var referer = document.referer;

// 当前页面是 p2p.wrox.com
document.domain = "wrox.com";  // ok
document.domain = "nczonline.net"; // failed


// 查找元素

document.getElementById("myDiv");
// 返回 HTMLCollection
var imgs = document.getElementByTagName("img");
imgs.length;
imgs[0].src;
imgs.item(0).src;
imgs.namedItem("myImage");

// 获取文档中的所有元素
var allElements = document.getElementByTagName("*")

document.anchors;
document.forms;
document.images;
document.links;

// 文档写入
document.write("<strong>" + (new Date()).toString() + "</strong>");

element.tagName.toLowerCase() == "div"; // 判断是不是div

var div = document.getElementBy("myDiv");
div.tagName == div.nodeName; // true
div.id;
div.className;
div.title;
div.lang;
div.dir;
div.getAttribute("title");
div.removeAttribute("title");
div.setAttribute("title", "xx");

element.attributes.getNameItem("id").nodeValue;
element.attributes.removeNameItem("id");

var div = document.createElement("div");
div.di = "";
div.className = "box";

document.body.appendChild(div);

document.createTextNode("Hello world.");
//  合并多个文本节点
element.normalize();


val attr = document.createAttribute("align");
attr.value = "left";
element.setAttributeNode(attr);
~~~~~~

根据html5属性, 自定义特性应该加上 `data-`前缀
