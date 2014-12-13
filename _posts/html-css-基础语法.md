title: Html CSS 基础语法
date: 2014-11-21 14:12:34
tags:
---

~~~~~~
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>无标题文档</title>
    <style type="text/css">
    </style>
    <link type="text/css" rel="stylesheet" href=""/>
    <link rel="shortcut icon" href=""/>
  </head>
  <body>
     <form>
       <input id="male" type="radio"/>
       <!-- 控制焦点范围 -->
       <label for="male">男</label>
        会进一步类级别,.
       <!-- 只在html5中有用 -->
       <label>
           <input/>
       </label>
     </form>
  </body>
</html>
~~~~~~

~~~~~~
h1 {
  display:block;
  /*
    display:none;
    display: inline-block;
    visible: false; 占位
  */
  background:red!Important; /*最高优先级*/
  border: 1px #000 solid;
  border: 1px #000 dashed;
  /*坐标(0,0) 对住中位线*/
  background; url() no-repeat center top;
  background; url() no-repeat right 0;
  background; url() no-repeat 0 bottom;
  <!--  repeat-x repeat-y repeat-->

}
~~~~~~

行高, 从文字中心基线出发, 向上到向下延伸一定的距离

测量方法, 从一行文字的最大有效像素到下一行文字的最大有效像素

~~~~~~
p {
  line-height: 2.2em;
  /* line-height: 盒子高度; 垂直居中 */
  font-weight: bold; // bold is 700
  font-weight: normal;
  font-style: itatic;
  letter-spacing: 10px; // 字符间距
  word-spacing: 10px;   // 单词间距
  text-indent: 2em; // 首行缩进
  text-decoration: none; // 无下划线
  text-decoration: underline;
  text-decoration: line-through;
  text-decoration: overline; //顶部
}

a:link {}
a:visited{}
a:hover{}
a:active{}
~~~~~~

# 盒子模型 #

~~~~~~
.box {
  padding: 30px;
  marging: 20px;
  border: 1px solid white;
  padding: 10px 20px 40px 80px; // 上右下左
  padding: 10px 40px 80px; // 上 左右(40) 下
  padding: 10px 40px; // 上下 左右
}
~~~~~~


# 网页三步准备工作 #

`hr` 标签有兼容性问题, 不建议使用(圆角or直角)

~~~~~~
/* 清空标签默认样式 */
body, h1, p, input, div, span, a, img, ul, li, ol, dl, dt, dd, h2, h3, h4, h5, h6{
  padding: 0px;
  margin: 0px;
  list-style: none; /** 兼容性问题 **/
  border: 0px;
}
/** 置body的全局样式(文字三属性) **/
body {
  color: #393939;
  font-size: 12px;
  font-family: "Verdana", "Microsoft YaHei", "Simsun";
  font-family:  "Microsoft YaHei", "SimHei";
  /* 大小 行高 字体*/
  font: 100px/1.5 "宋体", "黑体";
}
a {
  color: #393939;
  text-decoration: none;
}
a:hover {
  text-decoration: underline;
}
~~~~~~

块级标签, 水平居中
~~~~~~
.main {
  width: 330;// 必须要有宽度
  margin: 100px, auto;
}
~~~~~~

> 装饰性的图片用背景  
> 可以用 line-heihgt 文本上下 margin

嵌套排列的两个盒子也有塌陷问题, 给子盒子添加 `margin-top`,
会将父盒子一起往下挪,解决办法:
1. 给父盒子添加 `border` 属性, 能够完整的划分出盒子的边缘
2. 最好的解决办法: 给父盒子添加 `overflow: hidden`

行内标签, 浏览器当做文字处理;
如果想要改变行内标签的垂直方向的位置, 通过`margin` `padding` 是不能生效的.
只能通过 `line-height` 改变垂直方向的位置

# 浮动 #

浮动是第一种脱离标准流的方式, 半脱离, 只要是浮动的的标签浏览器当做不存在
`float: left | right`

所有浮动之后的标签显示模式变成了行内块

`clear:left` 清除左侧浮动的影响;
`clear:right` 清除右侧浮动的影响; `clear:both`



# Overflow #

控制父容器内容溢出, 显示问题

`overflow: hidden`; `overflow:auto;` 自适应, 垂直滚动条

打断字母, 强制字母换行西阿汉按`word-break: break-all`

# 左右分行 #

~~~~~~
<style type="text/css">
.main {
  margin: 0 auto;
  width: 1000px;
  overflow:hidden; // 强制检测浮动流
}

.left {
  float: left;
  width: 300px;
}

.right{
  float: right;
  width: 650px;
}
</style>

<div class="main">
  <div class="left">
  </div>
  <div class="right">
  </div>
</div>

~~~~~~

# 定位 #

`position: relative;` 偏移原先位置, 配合 `top`, `left`, `right`, `bottom` 使用

`position: absolute;` 完全脱离, 参照物是浏览器; 如果绝对定位的盒子有最近定位的父容器,
那么就以这个父容器为参照物.(子绝对, 父相对)

改变z轴堆叠顺序, `z-index: 998`

# 兼容 #

ie6双倍边距问题 如果外边距的方向和浮动方向相同, 那么ie6浏览器肯定会出现双倍边距问题;
如果外边距方向和浮动方向不同, 可能会出现双倍边距问题. 解决办法`_display: inline;`, 加下划线, 只针对ie6启用.

ie浏览器, 图片链接的边框线问题. `img{border: 0;}`

ie6, img底部留白, 显示将回车当做空格
1. `img{display:block;}`
2. 将父级设置溢出隐藏 `div{overflow:hidden;}` 

# 滑动门 #

内部所有标签都左浮动, 解决非矩形盒子背景为题

~~~~~~
.left {
float: left;
width: 5px; height: 40px;
background: url(img/left.jpg);
}
.center {
float: left;
height: 40px;
line-height: 40px;
background: url(img/center.jpg);
}
.right {
float: left;
width: 5px; height: 40px;
background: url(img/right.jpg);
}
~~~~~~

~~~~~~
<li>
    <span class="left"></span>
    <div class="center">
    </div>
    <span class="right"></span>
</li>
~~~~~~

# css 精灵 #

图片整合技术, css sprite, css 雪碧

针对的图片形式是*背景图像*


# 滤镜 #

半透明效果, 包括上面的文字也会透明, 所以要hack一下
~~~~~~
filter: alpha(opacity=60);/*只有ie内核*/
opacity: 0.6;
~~~~~~


# 建站流程 #

* 定义站点, 明确建站目的
* 激励网站结构图, 绘制流程图
* 首页的设计和制作
* 其他页面的设计和制作
* 测试
* 发布和维护

网站基本页面类型
* 首页 index.html
* 列表页 list.html 网站主导航点击进去的页面就是列表页
* 详情页(文章页), detail.html, 除了主导航以外的页面都是详情页

# 大 banner #

~~~~~~
banner_wrapper {
width: 100%; overflow: hidden; position: relative;
}
banner {
left: -50%; margin-left: -XXpx; position: absolute;
}
~~~~~~

# 楔形 #

制作小三角
~~~~~~
.box {
width: 0px; height: 0px;  border-left: 10px solid #000;
border-right: 10px solid #fff;
border-bottom: 10px solid #fff;
border-top: 10px solid #fff;
overflow: hidden; /**hack ie6 **/
}
~~~~~~


# IE6 hack #

* 选择器 hack
  * 针对ie6 `*html.header{width: 100px;}`
  * 针对ie7 `*+html.header{width: 100px;}`
* 属性 hack
  * ie6: `_color: red`
  * ie7及其以下: `*color: red`
* 后缀 hack
  * ie6-10: `color:red\9`
  * ie8-10: `color:red\0`
  * ie9, 10: `color:red\9\0`
* 浏览器hack
~~~~~~
<!--[if IE]> 只能被ie识别 ;<![endif]-->
<!--[if IE 6]> 只能被ie6识别 ;<![endif]-->
<!--[if gte IE 6]> <![endif]-->
<!--[if gt IE 6]>  ;<![endif]-->
<!-- lte lt -->

<!-- 判断不是ie -->
<!--[if ! ie]><!-->要判断的内容<!--<![endif]-->
~~~~~~
 
行内块间距问题
1. 去掉换行
2. 加上注释
~~~~~~
<span></span><!--
--><span></span>
~~~~~~
3. `margin-left: -8px`
4. `word-spacing: -8px`
5. 父盒子 `font-size: 0`

## ie6 bug ##

ie6注释引起的bug(多余字符)
1. 去掉注释
2. 加上空格
3. 加`position:relative`

ie6 li出现空白间隙, 因为li里面浮动太复杂, 导致有间隙;
解决方法 `vertical-lign: middle;`

绝对定位, 父盒子奇数长宽, 出现间距; 解决办法, 尽量奇数

# Tip #

`text-indent: 9999em` 控制隐藏文字, 优化 logo

表单标签和表单标签对齐, 或表单标签和普通标签对齐,
那么用浮动 `float`, 浮动可以实现完全没有间距的左对齐和顶对齐

ie6 float 浮动会自动展开, 除非给他加确定的宽度

ie6元素高度给设置成19px以下, 设置不了, `overflow: hidden`

`white-space: pre` 不合并空格; `nowrap` 强制在同一行显示所有文本

`text-overflow: ellipsis`, 如果文本溢出, 显示省略号

~~~~~~
<div title="tooltip"></div>
~~~~~~

