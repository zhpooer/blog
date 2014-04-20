title: 传智播客day13-JSP&JavaBean
date: 2014-04-16 09:18:59
tags:
- java
- JSP
- 传智播客
- JavaBean
---

# JSP中的九大隐式对象 #
就是JSP对应的Servlet那些变量
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
| 内置对象名称 | 对应的类型 | 备注 |
|----------------------------------------------|
|  request    | javax.servlet.http.HttpServletRequest    |                        |
|  response   | javax.servlet.http.HttpServletResponse   |                        |
|  session    | javax.servlet.http.HttpSession           | jsp指令 session="true" |
|  application| javax.servlet.ServletContext             |                        |
|  config     | javax.servlet.ServletConfig              |                        |
|  page       | java.lang.Object                         | 代表当前servlet对象本身 |
|  exception  | java.lang.Throwable                      | jsp指令 isErrorPage="true"|
|  out        | javax.servlet.jsp.JspWriter              | 和 reponse.getWriter()一样 |
|  pageContext| javax.servlet.jsp.PageContext |

## 隐式对象 out的使用 ##
~~~~~~
<%
    out.write("123"); // JspWriter, 缓存8kb
    response.getWriter.write("456");
%>
<!--结果是 456123-->
~~~~~~

## pageContext 内置对象 ##
作用:
* 是一个域对象, 存的数据只能是当前页面来访问.
还能操作其他三个域对象(ServletRequest, HttpSession, ServletContext)的数据

    可以操作本身的域对象和其他的域对象
    1. setAttribute(String key, Object value, int scope)
    2. getAttribute(String key, int scope)
    3. removeAttribute(int scope)
    4. scope: `PageContext.PAGE_SCOPE`
      , `PageContext.REQUEST_SCOPE`
      , `PageContext.SESSION_SCOPE`
      , `PageContext.APPLICATION_SCOPE`
    5. findAttribute(String key), **依次从页面、请求、会话、应用范围找对应的值**
    
* 获得其他8个隐式对象

    1. `pageContext.getRequest();`
    2. `pageContext.getSession();`
    3. ...

* 提供了转发和包含的简单方法

    1. `pageContext.forward("");`
    2. `pageContext.include("");`
    
# 四大域对象 #
* PageContext: 页面范围的数据, 用的很少
* ServletRequest: 请求范围的数据, 用的很多. 显示一次数据后就没有用了
* HttpSession: 会话范围内的数据, 用的很多, 每次请求和响应都需要的共享数据
* ServletContext: 应用范围内的数据, 用的不多, 是所有的客户端都共享的信息. 注意同步!!!

数据能不能取到, 关键是不是从一个地方取的数据

# JavaBean 概念 #

* 什么是JavaBean(VO, Value Object; DO: Data Object; POJO: 简单Java对象)
    遵循一定的命名规则:
    1. 必须由默认的构造方法
    2. 类的声明为public类型
    3. 字段都是私有的 `private boolean married`;
    4. 提供公有的getter或setter方法(属性)
    5. 一般实现 java.io.Serializable 接口
* 应用场景

   封装数据, 便于传递数据

# JavaWeb开发模型 #
* 模型一: JavaBean + JSP,只能开发很简单的应用,不适合企业级应用

* 模型二: MVC, Model+View+Controller

    * Model: JavaBean 数据
    * View: JSP
    * Controller: Servlet

    MVC只是三层架构模式中的前段展示层


## 三层架构模型 ##
![三层架构模型](/img/3layerstructor.png)

## 在JSP中使用JavaBean ##
使用动作元素(内置标签)
* jsp:useBean: 用于从指定范围根据名称查找Javabean,
如果没有找到, 创建该 JavaBean 的实例, 然后放到指定的范围.
~~~~~~
<jsp:useBean id="p" class="cn.itcast.Person" 
             scope="page|request|session|application"></jsp:useBean>
<% // 等价代码
   cn.itcast.Person p = null;
   p = (Person) pageContext.getAttribute("p", pageContext.PAGE_SCOPE);
   if(p==null){
       p = new Person();
       pageContext.setAttribute("p", p, pageContext.PAGE_SCOPE);
   }
%>
~~~~~~

* jsp:setProperty: 给指定名称的对象设置值
~~~~~~
<jsp:setProperty property="name" name="p" value="xxx"/>
<%// 等价代码
p.setName("xxx");
%>
~~~~~~

* jsp:getProperty: 如果一个属性的值是null, 界面就会显示null
~~~~~~
<jsp:getProperty property="name" name="p"/>
<%=p.getName()%> <!-- 等价代码 -->
~~~~~~

### 用请求参数设置 JavaBean 中的值 ###
在模型一中用到
~~~~~~
<form action="Bean.jsp" method="post">
    <input type="text" name="name"/>
    <input type="radio" name="gender" value="female"/> 先生
    <input type="radio" name="gender" value="male"/> 小姐
    <input type="submit"/>
</form>

<!-- Bean.jsp -->
<jsp:setProperty property="name" name="p" param="name"/>
<jsp:setProperty property="name" name="p" param="gender"/>
<!-- 使用通配符, 简化上面调用, 前提: 表单的字段名和JavaBean的字段名一致 -->
<jsp:setProperty property="*" name="p"/>
~~~~~~
