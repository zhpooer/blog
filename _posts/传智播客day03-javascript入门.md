title: 传智播客day03-javascript入门
date: 2014-04-01 09:03:33
tags:
- 传智播客
- javascript
---
# TODO #
*什么是 强类型，弱类型，动态类型，静态类型*

# javascript #

## undefined 和 null ##

undefined: 本身是数据类型，唯一的值是undefined，表示未定义和指定的

1. var 声明变量，当时没赋值，是 undefined
2. js 所有函数都有返回值，如果没有明确返回什么，返回值是 undefined
3. 访问的对象不存在属性时，值是 undefined

null 本身是数据类型，唯一的值是 null, 表示一个无效的对象
> `var o = null; o = new Date()`

## 运算符种类 ##

### 算数运算符 ###
~~~~~
/*
 * +: 有两个意义，
 * 1. 表示运算负求和
 * 2. 字符串链接符
 *    如果左右两边出现一个字符串的时候，就是字符串连接符
**/
var result = 3 + "2" // 32
var result = true + 2 // 3
var result = null + 2 // 2
var result = undefined + 2 // NaN
~~~~~

### 逻辑算术符 ###
~~~~~
/*
 * && : 寻找表示假的值，若没有找到，返回最后一个元素值，找到则返回那个值
 * || : 寻找表示真的的值，若没有找到，返回最后一个元素值, 返回最后一个元素值，找到则返回那个值
 *
**/
var result = "a" && "b" // b
var result = "a" || "b" // a
var result = !!"false" // true
// ||使用
function myFun(defaultValue) {
    var result = result || 1;
}
~~~~~
### typeof 和 instanceof ###
typeof 用来判断一个值是哪种数据类型

instanceof 判断一个值是否是某个类的实例

~~~~
typeof "hello"; // String
typeof 1; // number
typeof undefined; // undefined
typeof true; boolean
typeof null; // object
~~~~
> typeof (new Date); // object

> (new Date) instanceof Date // true

## 语句 ##

### if ###
> if(num=9) // equals if(9)

> if(9==num) 规避风险

### for ###

### while ###

### DEMO 九九乘法表 ###
~~~~~~
// TODO 用table输出到页面，使格式对齐
for(var i=1; i<10; i++) {
  var line = "";
  for(j = i; j < 10; j ++){
    var result = i*j;
    line += (i + "x" + j + "=" + result + "  ");
  }
  console.log(line);
}
//表格
function render(content, tag){
    var result = "";
    result += ("<" + tag + ">");
    result += content;
    result += ("</" + tag + ">");
    return result;
}
var tableContent = "";
for(var i = 1; i<=9; i++) {
    var row = "";
    for(var j = i; j <= 9; j ++){
        var result = i*j;
        row += render( i + "x" + j + "=" + result, "td" );
    }
    tableContent += render( row, "tr" );
}
document.write( render(tableContent, "table") );
~~~~~~

## 函数 ##
~~~~~~
<script>
    function myfn(){}

    function myfn2(args){}

    myfun2() // args passed undefined
</script>
<input onclick="myfn();"/>
~~~~~~
同名函数会被覆盖

### 函数arguments参数 ###
~~~~~~
function myfn(){}
myfun("a","b") // arguments => ["a", "b"]
~~~~~~

### 函数是变量 ###
~~~~~~
function myfn(){}
// var myfn = function(){} 匿名函数
// var myfn = new Function("arg_name, arg_name2", ”alert('hello world');“);
function handler( fun ){ fun() }
handler(myfn)
~~~~~~

## 全局变量和局部变量 ##

在script标签内，在函数外定义变量都是全局变量

> `for(var i=0;i<1;i++){} log(i);` // i is 1

> `function myfn(){ a = "1"; var b = "2" }` // a是局部变量，b是全局变量

## 数组 ##

> var arr = new Array() | var arr = []

可以跨越下标赋值，被跨越部分值是undefined,同一个数组可以装载不同类型值
> arr.length = 0 //清空数组

### demo ###
~~~~~~
//求最大值
function max(){
  if (arguments.length == 0 ) return undefined;
  var maxNum = arguments[0];
  for(var i=0; i<arguments.length; i++) {
    maxNum = maxNum > arguments[i] ? maxNum : arguments[i];
  }
  return maxNum;
}

max(23, 1, 990, 29, 100, 65);
~~~~~~

~~~~~~

var info = {
  "张三": [18, "男", "本科"],
  "李四": [17, "男", "大专"],
  "王五": [16, "男", "高中"]
};

function getinfoFromName(name) {
  console.log(info[name]);
}

getinfoFromName("张三");
getinfoFromName("李四");
getinfoFromName("王五");
getinfoFromName("aa");

~~~~~~

> new Array(23) // length == 23

> new Array(2.3) // 无效

### 数组方法 ###

1. push，向末尾添加
2. concat，连接两个或更多数组
3. join，拼接成字符串，指定连接符号，默认是逗号

## 面向对象 ##
~~~~~
function Persion(){};
var p = new Persion(); // 创建对象
p.name = ""; p["age"] = 12; // 添加属性
p.eat = function(){}   //添加方法
~~~~~

### 构造函数 ###
~~~~~
function Person(name,age){
    this.name = ""
    this.age = 12;
    this.eat = function (){}
};
~~~~~
js每一个类都有一个属性constructor,表示构造函数
js中的每一个函数内部都有一个this,this表示当前对象(谁调用就是谁)

~~~~~
var name = "out"
function sleep() {
    return this.name;
}
var p = new Object();  //new 一个新的运行上下文
p.name = "in";
p.sleep = sleep;

sleep();  // out, this 表示 最外部这运行上下文(环境)
p.sleep(); // in, this 表示 p 这个对象的运行上下文(环境)
~~~~~
