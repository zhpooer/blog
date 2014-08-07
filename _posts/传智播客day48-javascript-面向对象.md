title: 传智播客day48-Javascript 面向对象
date: 2014-06-17 09:00:32
tags:
- 传智播客
---

# 简介 #

Javascript 核心组成部分
* ECMAScript, 标准, 语法
* BOM, 浏览器对象模型, `window` 对象
  * `document`
  * `alert()`, `eval()`
* DOM, 文档对象模型, 利用DOM解析XML, HTML
  * `document`对象
    * `documentElement`属性, 指向根节点
  * `element`对象
    * 获取属性, `getAttribute()`
  * `node`对象

# 面向对象 #

含义
* 万物皆对象
* new 对象()
* 继承, 实现, 多态

# 函数 #
`function 函数名(){}`

定义函数
~~~~~~
// 普通方式
function fn1(){ };
// 构造函数
var fn2 = new Function();
// 直接方式
var fn3 = function(){ };
~~~~~~

## Arguments对象 ##
javascript 不存在函数重载,
如果定义多个同名函数, 只有最后定义的是起作用的

~~~~~~
function fn(){
   // 获取当前函数的实参的个数, 可以模拟函数重载的效果
   arguments.length;
}
~~~~~~

## 全局变量和局部变量 ##
~~~~~~
function fn1(){
   // 相当于 window.b = "b", 不建议, 不规范
   b = "b";
}

// 在全局变量和局部变量同名时, 函数中只能访问局部变量
function fn2(){
    // 局部变量 a 被定义出来, 但还没有初始化
    alert(a);    // 输出是 undefined
    var a = "b";
}
~~~~~~

## 匿名函数 ##
~~~~~~
// 函数可以当做参数传递, 简称回调函数
// 匿名回调函数, 匿名函数可以当做参数传递给一个函数
var one = function(){return 1;}
function fn(a,b){return a()+b();}
fn(one, function(){return 2;});  // 3

// 一次性调用函数, 自调函数
(
  function (str){
    alert(str);
  }
)("hello"); // 调用函数

function fn(){
   var a = "a";
   // 内部函数, 私有函数, 保证私有性
   function n(){ alert(a); }
   return n;
}
fn()();
~~~~~~
# 闭包 #
闭包, 函数可以函数之外定义的变量
* 降低耦合度
* 实现作用域的跨域访问

局限性
* 函数初始化的位置在局部(函数中)
* 函数和函数之间存在耦合度

~~~~~~

funtion fn(){
    var f = []
    for(var i=0;i<3;i++) {
        f[i] = function(){
            return i;
        }
    }
    return f;
}
var f = fn();
f[0]();   // 3
f[1]();   // 3
f[2]();   // 3
~~~~~~

# 对象 #

~~~~~~
// 定义对象
var obj1 = new Object();

var obj2 = {};

function obj3(){}

var hero = {
    name: "zhangwuji",
    syaMe : function(){
        alert("i am zhangwuji");
    }
}

function Hero(){
    this.name = "zhang"
    this.sayMe = function(){
        alert("i am zhangwuji");
    }
}

$("#ok").click(function(){
    this.value;
})
~~~~~~

## 操纵对象和方法 ##
~~~~~~
var hero = {
    name : "zhangwuji",
    sayMe: function(){
        alert("i am zhangwuji");
    }
}

// 调用普通对象的属性和方法
hero.name
hero.sayMe()
hero['sayMe']();

//修改
hero.name = "zhouzhiruo";
hero.sayMe = function (){
   alert("我是周芷若");
}
// 增加
hero.value = "zhouzhiruo";

// 删除
delete hero.name;
delete hero.sayMe;

// 调用函数对象(构造器)的属性和方法
function Hero(){
    this.name = "zhangwuji";
    this.sayMe = function(){
        alert("i am zhangwuji");
    }
}
var hero = new Hero();
~~~~~~

## 内部类型 ##
~~~~~~
// javascript 是弱类型
var str = "abcdefg";
var i = 1;
var arr = [1,2,3,4,5];
~~~~~~

## javascript内建对象 ##

数据封装类
* Object对象, 是所有对象的父类
~~~~~~
// 定义一个对象
var obj1 = new Object();
var obj2 = {};

var a = {}; // 对象
var b = []; // 空数组
var c = //; // 正则表达式
~~~~~~
* Array对象
~~~~~~
var arr1 = [];
var arr2 = new Array();
// length 获取数组长度

// 常用方法
// pop(), push() 插入和删除
// reverse() 颠倒数组中的元素
var str = "abcde";
var arr = str.split(""); // a, b, c, d, e
~~~~~~
* String对象
~~~~~~
var str1 = "aaa";
var str2 = new String("aaa");

alert(typeof str1);  // String
alert(typeof str2);  // Object

// == 和 != 值相等
// === 和 !== 全相等(值和类型)

"abcde".substr(3, 2);  // de
~~~~~~
* Number对象


工具类
* Date对象
~~~~~~
var d = new Date();
d.getDate();
d.getDay();  // 星期
~~~~~~
* Math对象
* Regex对象, `/^abc$/.test()`
* Functions对象, 全局对象
~~~~~~
isNaN();
eval();
decodeURI();
~~~~~~
* Events对象

错误类
* Error对象
* Thrown对象

# 原型 #
函数本身就是对象, 即函数对象.
对象一定具有属性和方法, 原型就是函数对象的一个特殊属性,
只要函数对象存在, 就一定由原型对象

~~~~~~
function Hero(){
    this.name = "zhangwuji";
    this.sayMe = function(){
        alert("i am zhangwuju");
    }
}
// 通过原型为函数对象 Hero 增加属性和方法
var hero = new Hero();
Hero.prototype.value = "zhouzhiruo";
Hero.prototype.sayHello = function(){
    alert("hello");
}
log(hero.value); // zhouzhiruo

// 以上会造成代码原型混乱
// 以下更简单, 但是有顺序需求,
// 必须在 new Hero 之前定义
Hero.prototype = {
    value: "zhouzhiruo",
    sayVal: function(){
        alert("hero");
    }
}
~~~~~~

当函数对象自身的属性或方法与原型的属性或方法同名
* 调用的属性和方法是函数对象自身的属性和方法
* 原型上的属性和方法都是真实存在的
* 函数自身上的方法可以重写原型上的方法和属性

意义
* 利用原型为函数对象增加属性和方法
* 利用自身的属性和方法重写原型的属性和方法
* 利用原型为内建类型提供属性和方法

通过增加内建对象的属性和方法, 做到自定义方法最大化
~~~~~~
Array.prototype.inArray = function(color) {
    for(var i=0, len=this.length;i<len;i++){
        if(this[i]===color){
            return true;
        }
    }
    return false;
}
~~~~~~

# 继承 #

继承的关系
* 子类的实例可以共享父类的方法
* 子类可以重写, 重载及扩展父类的方法
* 子类和父类都是子类实例的类型

~~~~~~
// 继承方式一, 缺陷, 继承依赖于 new A
function A(){
    this.a = "a";
}
function B(){
    this.b = "b";
}

var a = new A();
B.prototype = a;

var b = new B();
log(b.a); // 输出 a

// 继承方式二:
function A(){}
A.prototype = {a:"a"};
function B(){
   this.b = "b";
}
B.prototype = A.prototype;
~~~~~~

## 多函数对象继承 ##

~~~~~~
// 方式一:
function A(){}
A.prototype = {
    a: "a"
}
function B(){}
B.prototype = A.prototype;
B.prototype.name = "a";
~~~~~~

## 普通对象的继承 ##
普通对象之间是没有原型
~~~~~~
// 浅复制, 对象内容一致
function exendsCopy(obj){
    var b = {};
    for(var p in obj) {
        b[i] = p[i];
    }
    b.uber = obj;
    return b;
}
var a = {name: "a"}
var b = extendCopy(a);
~~~~~~

深复制, 内存地址一样(银行系统)

# jQeury插件 #
分类:
* 封装对象方法的插件 `$(exp).each()`
  * `jQeury.fn.extend(obj)`
* 封装全局方法的插件 `$.each()`
  * `jQuery.extend(obj)`
* 选择器插件
  * 扩展jQeury的选择器内容(xPath 插件)

~~~~~~
$("#input").test()

jQuery.fn.extends({
    test: function(){
        // this是jqeury对象
        this.val();
    }
})
~~~~~~

# 2048游戏 #
~~~~~~
<header>
    <h1>2048</h1>
    <a href="javascript:newGame();" id="newgamebutton">new game</a>
    <p>score<span id="score">0</span></p>
</header>
<div id="grid-container">
    <div class="grid-cell" id="grid-cell-0-1"></div>
    <div class="grid-cell" id="grid-cell-0-2"></div>
    <div class="grid-cell" id="grid-cell-0-3"></div>
    <div class="grid-cell" id="grid-cell-0-4"></div>
    
    <div class="grid-cell" id="grid-cell-1-0"></div>
    <div class="grid-cell" id="grid-cell-1-1"></div>
    <div class="grid-cell" id="grid-cell-1-2"></div>
    <div class="grid-cell" id="grid-cell-1-3"></div>

    <div class="grid-cell" id="grid-cell-2-0"></div>
    <div class="grid-cell" id="grid-cell-2-1"></div>
    <div class="grid-cell" id="grid-cell-2-2"></div>
    <div class="grid-cell" id="grid-cell-2-3"></div>

    <div class="grid-cell" id="grid-cell-3-0"></div>
    <div class="grid-cell" id="grid-cell-3-1"></div>
    <div class="grid-cell" id="grid-cell-3-2"></div>
    <div class="grid-cell" id="grid-cell-3-3"></div>
</div>
~~~~~~

~~~~~~
var board = new Array();
$(function(){
    newgame();
})
function newgame(){
    // 初始化棋盘格子
    init();
    // 随机生成数字
    gernerateOneNumber();
    gernerateOneNumber();

    $(document).keydown(function(event){
        case 37: // left
            // 是否可以向左移动
            if(moveLeft()){
            }
            break;
    })
}

function init(){
    for(var i=0;i<4;i++){
        board[i] = [];
        for(var j=0;j<4;j++){
            board[i][j] = 0;
            var gridCell = $("grid-cell-" + i + "-" + j );
            // 通过js设置grid的位置
            gridCell.css("top", getPosTop(i, j));
            gridCell.css("left", getPosLeft(i, j));
        }
    }
}

function updateBoardView(){
    // 随机生成位置
    for(var i=0;i<4;i++){
        for(var j=0;j<4;j++){
            
            $("#grid-container").append("<div class='number-cell' id="number-cell-i-j"></div>")
            var numberCell = $("#number-cell-i-j");
            if(board[i][j] == 0){
                numberCell.css("width", "0px");
                numberCell.css("height", "0px");
                numberCell.css("top", getPosTop(i, j)+50);
                numberCell.css("left", getPosLeft(i, j)+50);
            }
        }
    }
    // 设置数字值的字体样式
    $(".number-cell").css("line-height", "100px");
    $(".number-cell").css("font-size", "60px");
    // 初始化数字格子
    updateBoardView();
}

function generateOneNumber(){
    var randx = parseInt(Math.floor(Math.random()*4));
    var randy = parseInt(Math.floor(Math.random()*4));
    while(true){
        if(board[randx][randy] == 0){ break }
        randx = parseInt(Math.floor(Math.random()*4));
        randy = parseInt(Math.floor(Math.random()*4));
    }
    var randNumber = Math.random()<0.5?2:4;
    board[randx][randy] = randNumber;
    showNumberWithAnimation(randx, randy, randNumber);
}

function getPosTop(i, j){
    return 20 + i*120;
}

function getPosLeft(i, j){
    return 20 + j*120;
}

~~~~~~

<!-- 自然, 说话, 关怀 ,惊喜 -->
# 就业指南 #

学习

找工作
* 准备
* 投简历(时间8:30到9:00点), 2-4页简历
  * 基本信息
        姓名, 年龄, 毕业院校, 专业, 工作经验,
        目前所在地, 联系方式(手机号码, email一起), 不贴照片(建议)
  * 工作经历, 某某年至某某年, 就职于某某公司, 负责某某工作
  * 项目经验
        某某系统(项目名称), 项目周期(某某到某某, N个月),
        所在平台(JavaEE, Oracle, Linux)及架构(三大框架)
        项目描述,
          该项目或者系统是某某公司或者企业提出需求,
          完成了哪些功能, 实现了哪些需求, 满足了哪些要求, 运用了哪些技术.

          我在项目中, 担任哪些角色, 负责哪些工作, 完成哪些功能,
          实现哪些需求, 满足哪些需求, 运用哪些技术,
          遇到哪些问题是如何解决的, 从中学到了哪些内容
  * 专业技能(若不及三页, 宋体小四, 行间距1.5倍), 自我评价
        学习能力, 具备良好的团队合作意识, 吃苦耐劳, 能适应长期加班
  * 简历网站: 拉钩网, 猎聘网, 内推网, 一天200份左右, 直接发hr邮件
  * 简历邮件主题
        主题: 应聘职位, 姓名, 手机号码
        正文: 简历
        附件: 简历
* 笔试
  * 带纸和笔, 最好组团
  * 收集笔试题(>2000), 过滤掉会的, 收集不会和会但是不熟
  * 准备自我最好的技术, 做适当提升
  * 留联系方式
* 面试
  * 笔试三天之后, 直接问面试是什么时间?
  * 日志记录, 记录地点日期
  * 之前先了解公司
  * 注意着装, 头发, 上衣带领, 衣服有袖, 不穿短裤以及运动裤, 色调浅色, 商务休闲鞋
  * 带纸和笔, 手机静音
  * 坐姿, 自然
  * 规避技术题
    1. 研究笔试
    2. 简历上针对性罗列技术
    3. 心里暗示(不停地重复哪些技术比较好), 主导面试, 自我介绍,
    准备开场白(工作经历, 性格和爱好, 60秒搞定), 成为聊天的主线(平常培养)
    4. 碰到不会的问题, 直接回答我不会
    5. 非技术类问题, 1. 永远不要说实话, 2. 都是演员, 谁认真都输了, 3. 注意对方是什么样的人(话多话少, 技术关注, 寻找共同话题)
            极限测试, 中间没有间歇, 停顿一会儿
            压力测试, 不停打击情绪和自信心
            问缺点, 一工作时间不强, 懒散但是工作上不是, 双向性格
            为什么选择编程, 感兴趣, 专业培训, 提高能力, 不能一辈子只做技术或开发
* 谈工资, 在真实的自我评估上, 工作经验加成, 相关工作经验加成,
  如果自我评估5000, 期望值8000, 第一次说期望值, 若太高, 对方给出价格和期望值平均,
  若还是太高, 再来一边, 最后行就行, 不行就拉倒, 当对方纠结时,
  说服它, 抬高对方, 抬高对方公司(还差钱不), 抬高自己(肯定可以给你带来更多价值)
  
入职
* 合同分一年合同(试用期不能超过一个月), 三年合同(试用期不能超过三个月),
五年合同(试用期不能超过三个月). 如果是半年试用期, 问一下为什么超过半年
* 签完合同, 可以随时离开, 一个月之前提前通知
* 如果在合同履行过程中, 无辜辞退, 最多可以有18个月工资
* 注意个人所得税, 保险(五险一金), 税前8000, 税后6000;税前12000,税后9000;税前15000,税后12000;
饭补一个月四五百;笔记本补助, 一个月50;出差补助(100或50), 技术住宿(200-300), 吃饭补助(50-100);

入职黑色三个月
* 适应阶段, 遇到问题
  * 上网搜索答案
  * 问同学
  * 回来找老师
  * 问公司同事
* 找都技术牛人, 想尽各种办法处好关系, 以备后用, **投其所好**
* 出差带小礼品
* 出差回来单独请客吃饭(如领导, 牛人, 行政或人力), 不聊工作, 从电影, 风土人情扯到工作
* 努力工作时, 要注意周围. 有成绩, 要告诉别人. (如出差工作, 日报, 周报, 大家辛苦工作时的照片)

## 指南 ##

先谈薪资, 再谈入职
