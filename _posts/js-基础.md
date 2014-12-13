title: Javascript-基础
date: 2014-12-07 19:13:56
tags:
---

# 基础 #

~~~~~~
var div = document.getElementById();
div.style.display = "block";
div.className = "box";

// 基于换肤
link.href = "css2.css";

div.onmouseover = function(){
   this.style.display = "block";
}

div.innerHTML = "<font></font>"; 

var inputArr = document.getElementByTagName("input");
inputArr[0].checked = true;

// update 会在一秒后执行
setInterval(update, 1000);
update();

// 让div移动起来
div.style.left = oDiv.offsetLeft + 5 + 'px';
// 不包括外边距的实际高度
odiv.offsetWidth;

// ui 所有的li的长度
vaqr uiLength = li.offsetWidth*aLi.length;
// 复制一份
ui.innerHTML += ui.innerHTML;

// 获取计算后的样式
// IE
div.currentStyle.width;
// Firefox
getComputedStyle(div, flase).width;
~~~~~~

JS类型
* nubmer
* string
* object
* undefined
* function
* boolean

字符串转换
~~~~~~
// 从字符串中提取数字
parseInt("12px34"); // 12
parseInt("xxx");    // NaN

parseFloat("12.1");
33 + NaN;    // NaN
isNaN(NaN);
 NaN == NaN;  // false

"12" - "5";  // 7

function(){
  arguments.length();
  arguments.callee.caller;
}
~~~~~~

数组
~~~~~~
var arr = [1, 2];
arr.length = 0; // 清空数组

// 尾部操作
arr.push(3);
arr.pop();
// 头部
arr.shift();
arr.unshift(3);

arr.concat(arr2);
var str = arr.join('_');
str.split('_');

arr.splice(起始, length); // 从中间删除

arr.splice(起始, 0, 'a', 'b');// 从中间插入
arr.splice(起始, 2, 'a', 'b'); // 替换
~~~~~~

# DOM #

文档碎片, 只渲染一次, 提高速度(插入), 理论上
~~~~~~
var oFrag = document.createDocumentFragment();
oFrag.appendChild(li);
ul.appendChild(oFrag);
~~~~~~


~~~~~~
// 火狐 下空行当做一个子节点
var nodes = div.childNodes;

// 3 文本节点
// 1 元素节点
div.nodeType;
if(childNodes[i].nodeType == 1) {}

// childNodes 的兼容版, 但是包括空字符文本节点
div.children;
div.parrentNode;

// 用来定位的父元素
div.offsetParent;

// 获取第一个子节点
oFirst = oUl.firstElementChild  || oUl.firstCihld
oList = oUl.lastElementChild  || oUl.lastChild;

oNext = oUl.nextSibling  || oUl.nextElementSibling;
oPrevious = oUl.previousSibling  || oUl.previousElementSibling;

// 获取文本属性
oTxt.value = "123";
oTxt.setAttribute("value", "123");

var id = oTxt.getAttribute("value");

// 通过 class 来选择元素
var aEle = div.getElementsByTagName('*');
ale[0].className;
~~~~~~


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

ul.removeChild(btn.parentNode);
~~~~~~

根据html5属性, 自定义特性应该加上 `data-`前缀

# 事件 #

~~~~~~
document.onclick = function(ev){
  // 可视区距离
  if(ev) {
    // firefox 
    ev.clientX; ev.clientY;
  } else {
    // IE
    event.clientX; event.clientY;
  }
  // or  
  var oEent = ev || event;
  // 阻止冒泡
  oEvent.cancelBubble = true;
}

// 滚动条距离 // firefox 
document.documentElement.scrollTop;
// chrome
document.body.scrollTop;

// onpress = onkeydown + onkeyup;
document.onkeydown = function(ev){
  // shiftKey altKey
  if(en.ctrlKey){}
}

// 右键菜单
document.oncontextmenu = function(){
  if(ev.preventDeault){
   // 火狐
    ev.preventDefault();
  }
  // 阻止默认行为
  return false;
}
// 阻止提交  
form.onsubmit = function(){return false;}
// 阻止填入
txt.onkeydown = function(){return false;}

~~~~~~

# cookie #

~~~~~~
document.cookie = "user=blue;expires=" + (new Date());
// 不会覆盖
document.cookie = "pass=123";

// 获得
document.cookie.split("; ");

// 删除cookie
var expires = "前一天";
document.cookie = "user=blue;expires=" + expires;
~~~~~~

# 运动 #

1. 在开始运动时, 关闭已有定时器
2. 运动开始和停止隔开

~~~~~~

// target must be a integer
function startMove(target){
  clearInterval(obj.timer);
  // 缓存运动
  var speed = (targetOffset - div.offsetLeft)/8;
  speed = speed<0?Math.floor(speed):Math.ceil(speed);
  obj.timer = setInterval(function(){}
    // 匀速运动 // 缓存运动直接等于
    if(Math.abs(oDiv.offsetLeft-target)<speed) {
      clearInterval(timer);
      oDiv.style.left = iTarget + "px";
    } else {
       move();
    }
  , 30);
}

~~~~~~
~~~~~~
function getStyle(obj, attr){
  if(obj.currentStyle){ return obj.currentStyle[attr];}
  else { return getComputedStyle(obj, flase);}
}

function startMov(obj, attr, iTarget){
  clearInterval(obj.timer);
  obj.timer = setInterval(function(){
    var iCur;
    if(attr == 'opacity'){
      iCur = parseInt(parseFloat(getStyle(obj, attr))*100)
    } else {
      iCur = parseInt(getStyle(obj, attr));
    }
    var iSpeed = (iTarget - iCur)/8;
    iSpeed = iSpeed>0?Math,ceil(iSpeed): Math.floor(iSpeed);
    if(iCur == iTarget){ clearInterval(obj.timer);}
    else {
      if(attr == 'opacity'){
        obj.style.filter = 'alpha(opacity:' + (iCur + iSpeed)+ ')'
        obj.style.filter.opacity = ( iCur + iSpeed)/100;
      } else {
        obj.style[attr] = iCur + iSpeed + "px";
      }
    }
  }, 30);
}
~~~~~~

## 完美运动 ## 
~~~~~~
function startMove(obj, json, fn)
{
    clearInterval(obj.timer);
    obj.timer=setInterval(function (){
        var bStop=true;		//这一次运动就结束了——所有的值都到达了
        for(var attr in json)
        {
            //1.取当前的值
            var iCur=0;

            if(attr=='opacity')
            {
                iCur=parseInt(parseFloat(getStyle(obj, attr))*100);
            }
            else
            {
                iCur=parseInt(getStyle(obj, attr));
            }

            //2.算速度
            var iSpeed=(json[attr]-iCur)/8;
            iSpeed=iSpeed>0?Math.ceil(iSpeed):Math.floor(iSpeed);

            //3.检测停止
            if(iCur!=json[attr])
            {
                bStop=false;
            }

            if(attr=='opacity')
            {
                obj.style.filter='alpha(opacity:'+(iCur+iSpeed)+')';
                obj.style.opacity=(iCur+iSpeed)/100;
            }
            else
            {
                obj.style[attr]=iCur+iSpeed+'px';
            }
        }

        if(bStop)
        {
            clearInterval(obj.timer);

            if(fn)
            {
                fn();
            }
        }
    }, 30)
}
~~~~~~


## 弹性运动 ##


* 加速运动, 每次循环 `speed++`
* 减速运动, 每次循环 `speed--`

~~~~~~
function startMove(){
  obj.timer = setInterval(function(){
    iSpeed += (iTarget-height)/5;
    iSpeed *= 0.7;
    if(Math.abs(speed)<1 && Math.abs(iTarget-height)<1){
      clearInterval(obj.timer);
      obj.style.height = iTarget + 'px';
    } else {
      height += iSpeed;
      obj.style.height = height + 'px';
    } 
  }, 30);
}
~~~~~~

## 碰撞运动 ##

~~~~~~
var timer=null;

var iSpeedX=0;
var iSpeedY=0;

function startMove()
{
    clearInterval(timer);

    timer=setInterval(function (){
        var oDiv=document.getElementById('div1');

        iSpeedY+=3;

        var l=oDiv.offsetLeft+iSpeedX;
        var t=oDiv.offsetTop+iSpeedY;

        if(t>=document.documentElement.clientHeight-oDiv.offsetHeight)
        {
            iSpeedY*=-0.8;
            iSpeedX*=0.8;
            t=document.documentElement.clientHeight-oDiv.offsetHeight;
        }
        else if(t<=0)
        {
            iSpeedY*=-1;
            iSpeedX*=0.8;
            t=0;
        }

        if(l>=document.documentElement.clientWidth-oDiv.offsetWidth)
        {
            iSpeedX*=-0.8;
            l=document.documentElement.clientWidth-oDiv.offsetWidth;
        }
        else if(l<=0)
        {
            iSpeedX*=-0.8;
            l=0;
        }

        if(Math.abs(iSpeedX)<1)
        {
            iSpeedX=0;
        }

        if(Math.abs(iSpeedY)<1)
        {
            iSpeedY=0;
        }

        if(iSpeedX==0 && iSpeedY==0 && t==document.documentElement.clientHeight-oDiv.offsetHeight)
        {
            clearInterval(timer);
            alert('停止');
        }
        else
        {
            oDiv.style.left=l+'px';
            oDiv.style.top=t+'px';
        }

        document.title=iSpeedX;
    }, 30);
}
~~~~~~


