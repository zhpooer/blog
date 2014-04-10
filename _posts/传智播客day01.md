title: 传智播客Day01
date: 2014-03-30 14:58:05
tags:
- 课堂笔记
- html
---

html
=========

网页 = html + css + javascript

html:封装数据

css:美化数据

javascript: 操作数据

## html语言规范 ##
html 是标记语言，是由标签组成

```html:
<html>
  <!-- 注释 -->
  <head>
    <title>这是标题</title>
  </head>
  <body></body>
</html>
```

### html注释 ###

只有<!-- -->一种方式，本省就支持多行注释

### font标签 ###
*如何用eclipse建立静态网页*

#### font 更改字体大小和颜色 ####

```html:
<font size="7">我</font>今天很<font color="red">开心</font>
```

size:改变字体大小

color：改变字体颜色

已经逐渐退出历史舞台，被css取代

### h标签 ###

> h1 h2 h3 h4 h5 h6
`<h1><h2></h2></h1>`
*会如何显示*

### 转义字符 ###

`1<bf>2 <12> ab`
<bf>不会显示，<12>会显示

 空格: `&nbsp;`

### 标签种类 ###

1. 有开始和结束标签，开始和结束标签成对出现。作用封装数据。
2. 只有一个标签，没有结束标记。如`</br>`,单一功能标签

### 列表相关标签 ###

```html:
<dl>
    <dt></dt>
    <dd></dd>
    <dd></dd>
</dl>
```

dl:
    define list

dt:
    define title

dd:
    define data

```html:
<ol>
    <li> <li/>
    <li> <li/>
    <li> <li/>
</ol>
<ul>
    <li> <li/>
    <li> <li/>
    <li> <li/>
</ul>
```

`ol type="a/i/1/I/A"`

ol: ordered list

ul: unorded list

li: list item

### 图像标签 ###
`<img src="图片地址" alt="谢谢，无法正常显示"/>`
功能性标签，width、height不推荐

#### 绝对路径 ####

#### 相对路径 ####

### 表格标签table ###

表格是由行组成的，行是由列组成

~~~~~~html:
<table border="1" width="400px" bordercolor="red"
       cellpadding="3" cellspacing="5">
    <caption>标题</caption>
    <tr>
        <th align="center" colspan="2"> title1 加粗居中 </th>
        <th> title2 </th>
    </tr>
    <tr>
        <td> data1 </td>
        <td> data2 </td>
        <td rowspan="2"></td>
    </tr>
    <tr>
        <td> datax1 </td>
        <td> datax2 </td>
    </tr>
</table>
~~~~~~

th: 加粗居中

rowspan: 跨行

calspan: 跨列

table width: 可设为百分比

### 超链接 ###

`<a href="http://www.163.com">网易</a>`

`<a href="javascript:void(0)">网易</a>`

~~`<a href="www.163.com">网易</a>`~~

#### 协议 ####
* mailto
* http
* thunder
* ftp
* file

#### 锚点 ####
`<a name="top">定义锚点</a>`

`<a href="#top">定位锚点</a>`

### frameset ###

左右结构
~~~~
<html>
    <head> </head>
    <frameset cols="200px," noresize="noresize">
        <frame src="relative-path" >
            <a href="" target="right_frame"/>
        </frameset>
        <frame name="righ_frame" src="relative-path"/>
    </frameset>
    <body></body>
</html>
~~~~

上下，左右结构

~~~~
<frameset rows="20%," >
    <frame> </frame>
    <frameset cols="20%,*">
        <frame> </frame>
        <frame> </frame>
    </frameset>
</frameset>
~~~~

### iframe ###

iframe: 嵌入一个窗口,用法如frameset
~~~~html:
<iframe src="*" name="*" display="none"/>
~~~~

### form标签 ###

~~~~
<form action="" method="post/get">
    <!-- 文本框  -->
    <input type="text" name="user"/>
    <!-- 密码框 -->
    <input type="password" name="psy"/>
    <input type="password" name="repsy"/>
    <!-- 单选框，name要同一组 -->
    <input type="radio" name="sex"/>男
    <input type="radio" name="sex"/>女
    <!-- 多选框 -->
    <input type="checkbox" checked value="1"  name="intere"/>
    <input type="checkbox" value="2" name="intere"/>
    <input type="checkbox" name="intere"/>
    <!-- 下拉 -->
    <select name="">
        <option value="none" > 选择省份 </option>
        <option value="b" selected> 北京 </option>
        <option> 上海 </option>
        <option> 广州 </option>
    </select>
    <!-- 多行文本框 -->
    <textarea rows="3" cols="30">
    </textarea>
    <!-- 上传 -->
    <input type="file"/>
    <!-- 隐藏框 -->
    <input type="hidden"/>
    <!-- 提交 -->
    <input type="submit" value="提交"/>
    <!-- 重置 -->
    <input type="reset" value="重置"/>
</form>
~~~~
`没有name属性，不会提交`

要有value和name标签
