title: 传智播客day12-jsp
date: 2014-04-15 14:40:33
tags:
- jsp
- 传智播客
---

# JSP #
全称Java Server Pages, 是一种用于开发动态web资源的技术

JSP=html中写java代码

Servlet: 只能写java代码, 写html不是很方便

# JSP 的执行原理 #

jsp 翻译成 .Java 再 编译成 .class, 然后Tomcat加载
~~~~~~
// JSP中可以取到的变量
PageContext pageContext = null;
HttpSession session = null;
ServletContext application = null;
ServletConfig config = null;
JspWriter out = null;
Object page = this;
JspWriter _jspx_out = null;
PageContext _jspx_page_context = null;
~~~~~~

# JSP语法 #

## 模板元素 ##

JSP中的HTML元素, 定义了JSP的外观和框架

## 脚本表达式 ##
输出变量的值到页面上
~~~~~~
<%=new Date().toLocaleString()%>
//translate to out.println( new Date().toLocaleString() );
~~~~~~

## 脚本片段 ##
书写Java逻辑, 原封不动的java代码
~~~~~~
<%
   语句一;
   语句二;
%>
~~~~~~
~~~~~~
<table border="1">
    <tr>
        <th> </th>
        <th> </th>
    </tr>
    <%
    for(int i=1;i<=10;i++){
    %>
    <tr>
        <td><%=i%></td>
        <td><%="aaa" + i%></td>
    </tr>
    <%
    }
    %>
</table>
~~~~~~

## 声明 ##
定义类成员(静态变量, 静态方法, 实例变量, 实例方法)
~~~~~~
<%!
   // 实例变量
   int i = 100;
   public void method(){
       println("hello world");
   }
%>
~~~~~~

## 注释 ##
~~~~~~
<%--
   out.write("helloworld")
--%>
~~~~~~

`<%-- --%>`: 被注释的代码, 根本不会被翻译到Servlet中

`<!--  -->`: 浏览器注释, 代码还是会被执行

## 指令 ##
* 作用 给服务器用, 指示服务器应该如何对待JSP页面
* 基本语法: `<%@指令名称 指令属性=值 指令属性1=值1%>`

### page ###
~~~~~~
<%@page
  language="java"
  import="java.util.Data, java.util.Random"
%>
~~~~~~

|    属性        |   默认值       |               作用|
|----------------------------------------------------|
| language       |  java         | 设置编写也语言     |
|  import        |               | 导入包, 用逗号分隔 |
| session        |  true         | 是否生成 HttpSession 对象 |
|  buffer        |  8kb          | 字符输出流的缓存, none| 
|autoFlush       |  true         | 自动刷新缓存        |
|isThreadPage    |  false        | 是否线程安全        |
|errorPage       |               | 页面出错时, 转到的页面, 如果以`/`开头,为绝对路径       |
|isErrorPage     |  false        | 是否产生Exception对象, 错误后可以调用 exceptioin.getStackTrace() |
|contentType     |               | 改字符流编码,通知客户端显示字符集  |
|pageEncoding    | iso8859-1     | 指示服务器读取JSP时所用的编码, 同时有 contentType 的功能|
|isELIgnored     | false         | 是否忽略EL表达式 |


### taglib ###
引入外部的标签用的
~~~~~~
<%@ taglib uri="" prefix="c"%>
<c:if test=""></c:if>
~~~~~~

### include ###
页面包含
~~~~~~
<!-- 首选, 效率高, 静态包含, 两个 jsp 翻译成一个 jsp -->
<%@ include
    file="/**.jsp"
%>
<!-- 动态包含, 两个 jsp, 合并显示-->
<jsp:include page="/**.jsp"/>
~~~~~~

## JSP 常用内置标签(动作元素) ##
|标签        |  作用     |   详细说明            |
|----------------------------------------------|
|jsp:include |  动态包含   |    page:指向被包含页面|
|jsp:forward |   转发      |    page: 指向转发路径|
|jsp:param   | 传递请求参数 |   name:键 value:值             |

~~~~~~
// 源页面
<jsp:forward page="">
    <jsp:param value='key' name='username'/>
</jsp:forward>

// 目的页面
request.getParameter("username")
~~~~~~

# Error 的全局配置 #
~~~~~~
<!-- error 全局配置 -->
<error-page>
    <!-- 异常类型匹配-->
    <exception-type>java.lang.Exception </exception-type>
    <location> /error.jsp</location>
</error-page>
<error-page>
    <!-- 响应码匹配 -->
    <error-code>404</error-code>
    <location> /error.jsp</location>
</error-page>
~~~~~~
