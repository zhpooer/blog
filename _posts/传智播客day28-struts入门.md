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
            <!-- same as below -->
            <!-- <result name="index"> -->
            <!--     <param name="location">index.jsp </param> -->
            <!-- </result> -->
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
<package name="childOfHello" extends="helloworld" namespace="/" abstract="false"> </package>
~~~~~~
* abstract：可选值为true|false。说明他是一个抽象包。抽象包中没有action元素的。(默认为false)
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

## 配置文件解析2 ##
### package标签 ###
必须直接或间接地继承自struts-default的包.

作用: 方便管理我们的动作(struts-default是核心配置文件)

属性：
* abstract：可选值为true|false。说明他是一个抽象包。抽象包中没有action元素的。(默认为false)
* name：包名。不能重复。方便管理动作的。
* namespace：名称空间
* extends：继承什么

### action标签 ###
* name: 必须的, 动作名称
~~~~~~
<package name="p2" extends="struts-default">
    <!-- 只要找不到的action的name，找act4。默认动作名称 -->
    <default-action-ref name="act4"></default-action-ref>
</package> 
~~~~~~
* class：可选的。默认值是com.opensymphony.xwork2.ActionSupport
~~~~~~
<package name="p2" extends="struts-default">
    <!-- 只要找不到的action的class，找com.opensymphony.xwork2.ActionSupport。默认class -->
    <default-class-ref name="com.opensymphony.xwork2.ActionSupport"></default-class-ref>
</package> 
~~~~~~
* method: 可选. 默认值是`public String execute(){return "success"}`

### result标签 ###
type：默认值dispatcher。转发，目标JSP

name：默认值是success。
~~~~~~
<package name="default" namespace="/test" extends="struts-default">
    <action name="hello" class="com.itheima.action.HelloAction" method="execute">
        <result name="female">/female.jsp</result>
        <result name="male">/male.jsp</result>
    </action>
</package>
~~~~~~
访问包中带有名称空间的动作时：
* `http://localhost:8080/day22_01_strutsHello/test/hello.action`
* `http://localhost:8080/day22_01_strutsHello/test/aaa/bbb/hello.action`

动作有搜索顺寻：
1. 从/test/aaa/bbb找，不存在
2. 从/test/aaa找，不存在
3. 从/test，找到了
4. 一旦找到就不向上找了

## struts2 结果类型 ##
1. 结果类型其实就是一个实现com.opensymphony.xwork2.Result的类，用来输出你想要的结果
2. 在struts-default.xml文件中已经提供了内置的几个结果类型

### chain ###
转发到另一个动作

`<result-type name="chain" class="com.opensymphony.xwork2.ActionChainResult"/>`

如果转发的动作在一个名称空间中
~~~~~~
<action name="testChain1" class="com.itheima.action.CaptchaAction" method="download">
    <result name="success" type="chain">testChain2</result>
</action>
<action name="testChain2" class="com.itheima.action.CaptchaAction" method="download">
    <result name="success" type="dispatcher">/2.jsp<result>
</action> 
~~~~~~
如果转发的动作不在一个名称空间中
~~~~~~
<package>
    <action name="testChain1" class="com.itheima.action.CaptchaAction" method="download">
        <result name="success" type="chain">
        <!-- 如果需要转发的动作不在一个名称空间内，则需要进行参数的设置（原理看chain源码） -->
        <!-- 源码中有setNamespace和setActionName方法，去掉set，第一个字母改小写 -->
            <param name="namespace">/result1</param>
            <param name="actionName">testChain2</param>
        </result>
    </action>
</package>
<package name="p2" namespace="/result1" extends="base">
    <action name="testChain2" class="com.itheima.action.CaptchaAction" method="download">
        <result name="success" type="dispatcher">/2.jsp<result>
    </action>
</package>
~~~~~~

### dispatcher ###

`<result-type name="dispatcher" class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>(默认的)`

请求转发（地址栏不会变）。struts配置
~~~~~~
<action name="testChain2" class="com.itheima.action.CaptchaAction" method="download">
    <result name="success" type="dispatcher">/2.jsp<result>
    <!-- 这两个设置效果相同 -->
    <!--
        <result name="success" type="dispatcher">
            <param name="location">/2.jsp<result>
        </result>
    -->
</action>
~~~~~~

### redirectAction ###
`<result-type name="redirectAction" class="org.apache.struts2.dispatcher.ServletActionRedirectResult"/>`

请求重定向到另一个动作
~~~~~~
<action name="testRedirect1" class="com.itheima.action.CaptchaAction" method="download">
    <result name="success" type="redirectAction">testRedirect2</result>
</action> 
~~~~~~

### redirect ###
`<result-type name="redirect" class="org.apache.struts2.dispatcher.ServletRedirectResult"/>`

请求重定向（地址栏会变）.
~~~~~~
<action name="testRedirect" class="com.itheima.action.CaptchaAction" method="download">
    <result name="success" type="redirect">/2.jsp<result>
</action> 
~~~~~~

### stream ###
`<result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>`

结果类型为流，例如用于文件下载(具体原理需要看源码，而源码的核心就是execute方法）
struts.xml的配置(属性参数对应的都是类中的set方法）

~~~~~~
<action name="testStream" class="com.itheima.action.CaptchaAction" method="download">
    <!-- 不需要转向或重定向的页面，因为直接下载就可以了 -->
    <result name="success" type="stream"><!-- 在文档中拷贝以下参数 -->
    <!-- 为了能让所有类型的文件都能下载，在Tomcat/conf/web.xml里面搜索bin，找到以下mapping参数 -->
       <param name="contentType">application/octet-stream</param>
    <!-- 查看Stream对应的类源代码,对应里面的字符串变量，根据这个字符串找输入流-->
       <param name="inputName">imageStream</param>
    <!-- 消息头的设置，指定下载，且指定下载文件名称 -->
       <param name="contentDisposition">attachment;filename="1.jpg"</param>
    <!-- 设置缓存的大小 -->
       <param name="bufferSize">1024</param>
    </result>
</action>
~~~~~~
实现文件下载
~~~~~~
public class CaptchaAction {
    //设置一个流，并生成set，get方法。
    private InputStream imageStream;
    public InputStream getImageStream() {
        return imageStream;
    }
    public void setImageStream(InputStream imageStream) {
        this.imageStream = imageStream;
    }
    public String download() throws FileNotFoundException{
        //获得文件的真实路径
        String realPath = ServletActionContext.getServletContext().getRealPath("/WEB-INF/111.jpg");
        //获得文件输入流
        imageStream = new FileInputStream(realPath);
        return "success";
    }
    public String method1(){
        try{
            //int i=1/0;//人为制造异常，可以让catch转向全局结果集，error.jsp

return "success";
        }catch(Exception e){
            return "error";
        }
    }
}
~~~~~~
### plain ###
`<result-type name="plainText" class="org.apache.struts2.dispatcher.PlainTextResult" />`

显示指定页面的源代码（不好用，只有java语句才能显示源代码）

~~~~~~
<action name="testPlanText" class="com.itheima.action.CaptchaAction" method="showPlanText">
    <result name="success" type="plainText">/1.jsp<result>
</action>
~~~~~~

### 其他结果类型 ###

~~~~~~
<result-type name="httpheader" class="org.apache.struts2.dispatcher.HttpHeaderResult"/>
<result-type name="freemarker" class="org.apache.struts2.views.freemarker.FreemarkerResult"/>显示模板。。。需要实验。
<result-type name="velocity" class="org.apache.struts2.dispatcher.VelocityResult"/>显示模板。。。需要实验。
<result-type name="xslt" class="org.apache.struts2.views.xslt.XSLTResult"/>(显示样式？）
~~~~~~

如果提供的结果类型不够用，就需要自定义了(注意需要实现Result接口)

自定义完的结果类型，需要先声明，才能使用：（随机验证码图片结果类型，实际应用的案例）
~~~~~~
<package name="base" extends="struts-default">
    <!-- 配置局部结果视图 -->
    <result-types>
        <result-type name="captchaResults" class="com.itheima.action.CaptchaResults"></result-type>
    </result-types>
    <!-- 配置全局结果视图, 只能配置在package里, 但是可以通过继承来使用 -->
    <global-results>
        <result name="error">/error.jsp</result>
    </global-results>
</package>
<!-- 需继承base,因为自定义的局部结果视图配置在base里面,且base继承了核心配置文件 -->
<package name="p1" namespace="/results" extends="base">
    <action name="captcha" class="com.itheima.action.CaptchaAction" method="genImage">
        <result type="captchaResults" name="success">
            <!-- 调用结果处理类的setter方法，注入参数的值-->
            <param name="width">600</param>
            <param name="height">400</param>
        </result>
    </action>
</package>
~~~~~~

### 全局结果类型 ###
当很多提交请求跳转到相同的页面，这个时候，
这个页面就可以成为全局的页面。在struts2中提供了全局页面的配置方法。
~~~~~~
<!-- 这个配置必须写在action配置的上面。dtd约束的规定。 -->
<!-- 配置全局结果视图, 只能配置在package里, 但是可以通过继承来复用 -->
<global-results>
    <result name="success"> success.jsp </result>
</global-results>
~~~~~~

# struts中存在一些内置常量 #
在struts2-core-*.jar的org.apache.struts2的default.properties文件中存在一些内置常量

~~~~~~
<!-- request.setCharacterEncoding(), 针对post请求参数编码有效 -->
<constant name="struts.i18n.encoding" value="UTF-8"></constant>
<!-- 配置需要struts框架处理的uri的扩展名 -->
<constant name="struts.action.extension" value="do,,action"></constant>
<!-- 开发模式：打印更多的异常信息。配置文件会自动加载 -->
<!-- devMode模式是开发模式，开启它则默认开启了struts.i18n.reload、struts.configuration.xml.reload -->
<constant name="struts.devMode" value="true"></constant>
<!-- 静态资源是不是设皇城, 开发阶段, 修改true -->
<constant name="struts.server.static.browserCache" value="true"></constant>
<!-- 配置不支持动态方法调用 -->
<constant name="struts.enable.DynamicMethodInvocation" value="false"></constant>
<!-- 让struts重新加载配置文件，但不会导致web应用重新启动。 -->
<constant name="struts.configuration.xml.reload" value="false"></constant>
<!-- 指定每次请求到达，重新加载资源文件 -->
<constant name="struts.i18n.reload" value="true"/>
<!-- 工厂类, 和spring 整合用 -->
<constant name="struts.objectFactory" value="spring"/>
<!-- 表达式直接访问static静态方法的开关 -->
<constant name="struts.ognl.allowStaticMethodAccess" value="true"></constant>
<!-- 配置全局国际化消息资源包,value写资源包的基名，多个资源包之间用逗号，分隔-->
<constant name="struts.custom.i18n.resources" value="com.itheima.resources.msg"></constant>
<!-- 更改strutsUI标签的显示样式模板，参考struts2-core-*.jar中的template -->
<constant name="struts.ui.theme" value="xhtml"></constant>
<!-- 动作名字里面默认是不允许出现/的,以下常量设置可以出现/ -->
<constant name="struts.enable.SlashesInActionNames" value="true"></constant>
<!-- 动作名字里面默认是不允许出现/的,如果有名称空间,除了以上常量,还需要打开这个开关 -->
<constant name="struts.mapper.alwaysSelectFullNamespace" value="true"></constant> 
~~~~~~

常量可以在下面多个文件中进行定义，struts2加载常量的搜索顺序如下，后面的设置可以覆盖前面的设置：
* default.properties文件
* struts-default.xml
* struts-plugin.xml
* struts.xml
* struts.properties（为了与webwork向后兼容而提供）
* web.xml

包含配置(<include>):在struts.xml文件这，使用<include>属性来包含其他配置文件，需要放在<struts>下,<package>外
~~~~~~
<include file="struts-mobile.xml"></include>
~~~~~~
# struts Action 初始化 #

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
    public String execute() { }
}
~~~~~~

~~~~~~
<!-- 方式三 -->
<action name="anyName"> <!-- 没有写 class, 默认执行 com.opensymphony.xwork2.ActionSupport -->
    <result> index.jsp </result>
</action>
~~~~~~
## Action 依赖注入 ##

~~~~~~
public Action extends ActionSupport(){
   @BeanProperty private String message;
}
~~~~~~
~~~~~~
<action>
    <param name="message"> auto insert into </param>
    <result type="redirect">/7.jsp?msg=${message} </result>
</action>
<!-- 7.jsp: ${param.msg} -->
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

