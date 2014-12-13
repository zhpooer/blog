title: BOM 基本语法
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

var b = comfirm("any");

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

# Tips #

~~~~~~
var win = window.open('about:blank');
// 清空页面并写入
win.document.write();
win.close(); // 关闭窗口

// 可视区居中, 可以用 fixed 但是 ie6 不支持
window.onresize = window.onload = window.onscroll = function (){
  var oDiv = document.getElementById('div');
  var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
  var t = (document.documentElement.clientHeight - oDiv.offsetHeight)/2;
  oDiv.style.top = scrollTop + t + 'px';
  }


~~~~~~

~~~~~~
window.onload=function () {
  var oBtn=document.getElementById('btn1');
  var bSys=true;
  var timer=null;

  //如何检测用户拖动了滚动条
  window.onscroll=function () {
    if(!bSys) {
      clearInterval(timer);
    }
    bSys=false;
  };

  oBtn.onclick=function () {
    timer=setInterval(function (){
      var scrollTop=document.documentElement.scrollTop||document.body.scrollTop;
      var iSpeed=Math.floor(-scrollTop/8);

      if(scrollTop==0) {
        clearInterval(timer);
      }
      bSys=true;
      document.documentElement.scrollTop=document.body.scrollTop=scrollTop+iSpeed;
    }, 30);
  };
};
~~~~~~
