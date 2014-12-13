title: JS-高级
date: 2014-12-12 10:32:33
tags:
---

# 高级事件 #

~~~~~~
// IE
if(obj.attachEvent){
  obj.attachEvent('onclick', function(){
    this;  // this is window
  }); 
} else {
  // FF
  oBtn.addEventListener('click', func, false); // this is oBtn
}

obj.detachEvent('onclick', fun);
obj.removeEventListener('onclick', fun);

// IE 捕获所有事件
obj.setCapture();
// IE 释放所有事件
obj.releaseCapture();

// ie chrome 鼠标滚轮 
obj.onmousewheel = func;
// FF DOM 事件
obj.addEventistener("DOMMouseScroll", func);
~~~~~~

# 字符串操作 #

~~~~~~
str.search('d');  // 返回要查找的字符串 第一次出现的位置
str.substring(1, 4); // 不包含最后一个位子

var re = /\d+/g; // m multiline
str.match(re);
re.text(str);

// 去首尾空格
var str = /^\s*|\s*$/;
// 匹配中文
var str = /\u4e00-\u9fa5/;
new RegExp('\\b' + className + '\\b', 'i');
~~~~~~

# 图片预加载 #

预判加载, 自动加载下一张图片 

延迟加载, 先加载html框架, 在加载图片
~~~~~~
var img = new Imgage();
img.src = "http://...";
~~~~~~
