title: 传智播客day28-struts入门
date: 2014-05-12 09:14:18
tags:
- 传智播客
- struts
---
# Servlet 缺点 #
1. 写一个 servelt 需要在 web.xml 中配置8行, 如果一个系统中servlet很多, 会导致
web.xml中文件中的内容多多
2. 在项目中很多人编辑一个 web.xml 文件会出现文件冲突
3. 在一个 servlet 中方法的入口只有一个, 如果在一个 servlet 中写很多方法, 这些方法
应该传递参数, 根据每次请求参数不一致来判断执行哪个方法
4. servlet 中的方法都有两个参数 request,response,这两个参数具有严重的容器依赖性,
所以在servlet不能单独测试
5. 如果在表单中的元素很多, 在servlet 中要想获取表单的数据,
那么在servlet的方法必须调用大量的 `request.getParameter()`
6. 在一个servlet属性中声明一个数据, 会存在线程安全问题

优点: 因为是最底层的mvc, 所以效率比较高

很多项目中对servlet 进行了重构, 重构目标是:
1. 更有利于团队协作开发
2. 把servlet的缺点一次进行修改

# 重构 Servlet #
在 web.xml 文件中只写一个servlet, 中控servlet

model 层为action

在中控 servlet 中利用java 的反射机制动态调用该action

## 重构效果 ##
只写一个servlet
 
只需要在 web.xml 配置一个 servlet

## 步骤 ##
* 创建一个servlet

    需求: 访问 `http://localhost:8080/userAction.action`, 调用 `UserAction.execute()`
~~~~~~
<listener>
    <listener-class>ActionListener</listener-class>
</listener>
<servlet-mapping>
    <servlet-name>...</servlet-name>
    <url-parttern>*.actions</url-parttern>
</servlet-mapping>
~~~~~~
* 写一个监听器

    写一个监听器, 在该监听器中写一个Map, map中的key存放url的action中的部分: userAction, 
    value 存放对应类的字符串形式, 把该map放入到application域中
~~~~~~
public class ActionListener implements ServletContextListener {
    public void contextDestroyed(ServletContextEvent e) {
        e.getSevletContext().setAttribute("actions", null);
    }
    public void contextInitialized(ServletContextEvent e) {
       Map map = new HashMap<String, String>();
       map.put("userAction", "cn.itcast.Action.UserAction");
       e.getSevletContext().setAttribute("actions", map);
    }
}
~~~~~~    
* 中控 servlet 执行

    1. 把url中的urserAction做一个解析
    2. 提取 application 域中的map, 根据 userAction key, 找到value
    3. 执行 `UserAction.execute(request, response)`

~~~~~~
public class ActionServlet extends HttpServlet {
    public void doGet(req, resp){
        String fullActionName = req.getRequestURI().subString(req.getContextPath().length); // userAction.action
        String actionName = fullActionName.replace("^(.*)\\.action", "$1");
        Map actionMap = getServletContext().getAttribute("actions");
        String actionClassName = actionMap.get(actoinName);
        String clazz = Class.forName(actionClassName);
        Method m = clazz.getMethod("execute", HttpServletRequest.class, HttpServletResponse.class);
        String result = method.invoke(clazz.newInstance(), req, resp); // 返回要转发地址
        req.getRequestDispatcher(result).forward(req, resp);
    }
    public void doPost(req, resp) { doGet(req, resp); }
}
public class UserAction {
    public string execute(HttpServletRequest res, HttpServletResponse resp) {
        return "index.jsp";
    }
}
~~~~~~

# Servlet 进化史 #
1. 03-05, mvc框架是 servlet
2. apache 的 struts1,实现松耦合 , 没由解决容器依赖性问题
3. webwork, 让action没有任何容器依赖性, 把文件上传, 检验工作和保存用户的工作松耦合
4. struts 和 webwork整合成struts2

# Struts HelloWorld #
导入包:
* freemarker, 模板
* ognl, 表达式, 为了显示存储在数据, 功能类似于el表达式
* struts2-core, struts核心包
* xworks-core, web-work核心包


1. 编写Web.xml
~~~~~~
<filter>
    <filter-name>struts2 </filter-name>
    <filter-class>
        org.apache.struts2.dispatcher.ng.filter.strutsPrepareAndExecuteFilter
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>struts2 </filter-name>
    <url-pattern>/* </url-pattern>
</filter-mapping>
~~~~~~

3. 编写Action
~~~~~~
public class HelloWorldAction {
    public String execute(){
        println("hello")
        return "index";
    }
}
~~~~~~

4. 写配置文件, 在src下面写一个 struts.xml
~~~~~~
<struts>
    <!-- package 功能是用来管理 action 的, 一般情况下package是针对模块划分的 -->
    <!-- name 为 package 的名称, 是唯一的 -->
    <!-- extends 实际上是把 package 中 name为 *struts-default* 包中所有的功能继承下来 -->
    <!-- namespace 设置访问的相对路径, 和 配置 action 转向到jsp页面时的查找路径-->
    <package name="helloworld" namespace="/" extends="struts-default">
        <action name="helloworldAction" class="cn.itcast.action.HelloWorldAction">
            <result name="index"> index.jsp </result>
        </action>
    </package>
</struts>
~~~~~~

## struts2 好处 ##
web.xml 中只有一个过滤器, 不用繁琐的配置

action就是一个简单的javabean, 与servlet容器没有任何依赖

多出了一个struts.xml, 配置Action的行为

# 配置文件解析 #

在struts过滤器初始化时, 加载了几个配置文件
`struts-default.xml(在struts2核心包的根目录下)` `struts.xml(供程序员使用)` `struts-plugins.xml`

struts 先加载 `struts-default.xml`, 后加载 `struts.xml`, 如果出现相同元素, 后加载覆盖先加载

`package` 分模块管理action,
* 属性 `name`, 包名字, 值唯一
* 属性 `extends`, 用法 `extends="struts-default"`,
把 package 中 name为 *struts-default* 包中所有的功能继承下来
~~~~~~
<!-- 继承了 helloworld -->
<package name="childOfHello" extends="helloworld"> </package>
~~~~~~
* 属性`namespace`, 与url相关,  如果 `namespace="/base"`,
则要访问 `${contextPath/base/helloWorldaction.action}`, 但是 `${contextPath/base/a/helloWorldAction.action}`也能访问
  * 查找规则, 先在 `/base/a` 下找, 然后在 `/base` 下找
  * 如果`Action.execute()`返回了`"index"`, 则struts 会找根据配置文件在`/base/`文件夹下查找`index.jsp`
* 标签 `<result name="name" type=""></result>`, 结果集
  1. `Action.execute` 返回一个字符串, 返回的字符串要和struts的配置文件中的`result`标签的`name`属性值匹配
  2. `name`属性, 可以省略, 默认为 *"success"*
  3. `type`属性, 结果集类型, 可省略, 默认为值为 "dispatcher", 在`struts-default.xml` 定义 `<result-type name="dispatcher"/>`
* `include` 标签, 保证可以存在很多个 struts 配置文件
~~~~~~
<struts> <include file="included-struts.xml"/> </struts>
~~~~~~

# struts Action 配置 #

~~~~~~
// 方式一
public class MyAction implements com.opensymphony.xwork2.Action {
    public String execute(){
        return SUCCESS; // Action 中定义的常量, 匹配配置文件 struts.xml 中的 action.name
    }
}
// 方式二
// 封装了一些常用功能, 如国际化 表单验证 等功能
public class MyAction extends com.opensymphony.xwork2.ActionSupport {
    public String execute() {
    }
}

~~~~~~
~~~~~~
<!-- 方式三 -->
<action name="anyName"> <!-- 没有写 class, 默认执行 com.opensymphony.xwork2.ActionSupport -->
    <result> indelx.jsp </result>
</action>
~~~~~~

## 通配符映射 ##

在配置文件的 Action 标签中, 可以配置 action 被执行行为
~~~~~~
<!-- 方式一 -->
<!-- 访问 ${contextPath}/m1/userAction.action 时,默认会调用 UserAction 的 saveUser 方法 -->
<package name="method" namespace="/m1">
    <action name="userAction" method="saveUser" class="**.UserAction">
        <result>index.jsp</result>
    </action>
</package>

<!-- 方式二 -->
<!-- 访问 ${contextPath}/m2/userAction!deleteUser.action 时,
     会调用 UserAction 的 deleteUser 方法 -->
<package name="method" namespace="/m2">
    <action name="userAction" class="**.UserAction">
        <result>index.jsp</result>
    </action>
</package>

<!-- 方式三 -->
<!-- 访问 ${contextPath}/m3/a_add.action 或 ${contextPath}/m3/**_add.action 时,
     都会调用 UserAction 的 saveUser 方法 -->
<package name="method" namespace="/m3">
    <action name="*_add" method="saveUser" class="**.UserAction">
        <result>index.jsp</result>
    </action>
</package>

<!-- 方式四 -->
<!-- 访问 ${contextPath}/m4/saveUser_add.action 时,
     会调用 UserAction 的 saveUser 方法 -->
<package name="method" namespace="/m4">
    <action name="*_add" method="{1}" class="**.UserAction">
        <result>index.jsp</result>
    </action>
</package>

<!-- 方式五 -->
<!-- 访问 ${contextPath}/m5/UserAction_pattern.action 时,
     会调用 UserAction 的 pattern 方法 -->
<!-- 访问 ${contextPath}/m5/PersonAction_pattern.action 时,
     会调用 PersonAction 的 pattern 方法 -->
<package name="method" namespace="/m5">
    <action name="*_pattern" method="pattern" class="cn.itcast.{1}">
        <result>index.jsp</result>
    </action>
</package>

<!-- 方式六 -->
<!-- 访问 ${contextPath}/m6/UserAction_saveUser.action 时,
     会调用 UserAction 的 saveUser 方法, 并返回 saveUser.jsp -->
<package name="method" namespace="/m6">
    <action name="UserAction_*" method="{1}" class="cn.itcast.UserAction">
        <result>{1}.jsp</result>
    </action>
</package>
<!-- 变体 -->
<package name="method" namespace="/m6">
    <action name="*_*" method="{2}" class="cn.itcast.{1}">
        <result>{2}.jsp</result>
    </action>
</package>
~~~~~~

**通配的程度越高, 匹配的范围越大, 越容易出问题**
