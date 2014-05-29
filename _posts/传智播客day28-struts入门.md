title: 传智播客day28-struts入门
date: 2014-05-12 09:14:18
tags:
- 传智播客
- struts
---
# 框架概述 #

三大框架: 企业主流 JavaEE 开发的一套架构 Struts + Spring + Hibernate

框架是实现了部分功能的代码(半成品), 使用框架简化企业级软件开发

学习框架, 清楚知道框架能做什么, 还有哪些工作需要自己编码实现

## Struts2 ##

Struts2 是Web层开发框架, 是优秀的MVC框架
* struts1 、webwork 、jsf 、SpringMVC 都是MVC 

MVC: 是一种思想, 是一种模式, 分为 Model模型, View视图, Controller控制器
* MVC由来是Web开发

JavaEE软件三层结构: Web层(表现层), 业务逻辑层, 数据持久层 (sun提供JavaEE开发规范)
JavaEE开发强调三层结构, Web层开发注重MVC

由传统 Struts1 和 WebWork 两个经典框架发展而来
* Struts2 内核 webwork
* Xwork提供了很多核心功能：前端拦截机（interceptor），运行时表单属性验证，类型转换，
强大的表达式语言（OGNL – the Object Graph Navigation Language），IoC（Inversion of Control反转控制）容器等

Struts2 和 Struts1 关系: 
没有关系， Struts2 全新框架，引入WebWork很多技术和思想，
Struts2 保留Struts1 类似开发流程


## 核心功能 ##
* 允许POJO（Plain Old Java Objects）对象 作为Action
* Action的execute 方法不再与Servlet API耦合，更易测试
* 支持更多视图技术（JSP、FreeMarker、Velocity）
* 基于Spring AOP思想的拦截器机制，更易扩展
* 更强大、更易用输入校验功能
* 整合Ajax支持

# strut2 快速入门 #

Web层框架都会使用前端控制器模式(JavaEE模式)
* javaWeb 编写的程序, 一次请求对应一个servlet, 此时servlet完成请求处理
* 使用框架, 所有访问通过 前端控制器, 前端控制器已经实现了部分代码功能(通用代码),
再交给不同 Action 来处理(请求分发), 一次请求, 对应一个Action

Struts2 前端控制器: `ServletPrepareAndExecuteFilter`
1. 编写请求页面
~~~~~~
<a href="${contextPath}/hello.action">访问Strut2</a>
~~~~~~
2. web.xml 配置 struts2 前端控制器
~~~~~~
<filter>
    <filter-name> struts2 </filter-name>
    <filter-class> org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter </filter-class>
</filter>
<filter-mapping>
    <filter-name> struts2 </filter-name>
    <url-pattern> /* </url-pattern>
</filter-mapping>
~~~~~~
3. 执行过滤器后, 读取struts配置文件, 将请求分发, 在根目录下创建struts.xml
~~~~~~
<package name="default" namespate="/" extends="struts-defautl">
   <!-- 将请求分发 给一个Action  -->
   <action name="hello" class="io.zhpooer.HelloAction">
       <!-- 将返回字符串与跳转页面绑定  -->
       <result name="success">
       </result>
   </action>
</package>
~~~~~~
4. HelloAction
~~~~~~
// struts2 处理请求的Action
public class HelloAction{
    // 编写 excute 方法, String 类型返回值, 无参数
    public String excute(){
        // 返回字符串来控制页面跳转
        return "success";
    }
}
~~~~~~
## 运行流程图 ##
![/img/struts_core.png]
用户请求 -> StrutsPrepareAndExecuteFilter 核心控制器 ->
 Interceptors 拦截器(实现代码功能) -> Action 的execuute -> 结果页面 Result
* 拦截器 在 struts-default.xml 定义
* 执行拦截器 是 defaultStack 中引用拦截器 

# struts2 常见配置 #

## 配置文件的加载顺序 ##

由核心控制器加载 StrutsPrepareAndExecuteFilter  (预处理，执行过滤) 
1. default.properties 该文件保存在 struts2-core-2.3.7.jar 中 org.apache.struts2包里面 (常量的默认值)
2. struts-default.xml 该文件保存在 struts2-core-2.3.7.jar(Bean、拦截器、结果类型)
3. struts-plugin.xml 该文件保存在struts-Xxx-2.3.7.jar(在插件包中存在 ，配置插件信息)
4. struts.xml 该文件是web应用默认的struts配置文件(实际开发中，通常写struts.xml)
5. struts.properties 该文件是Struts的默认配置文件  (配置常量)
6. web.xml 该文件是Web应用的配置文件 (配置常量)

后加载的文件会覆盖之前加载的文件常量内容

## Action配置 ##
### package标签 ###

必须要为`<action>`元素 配置`<package>`元素  (struts2 围绕package进行Action的相关配置)

必须直接或间接地继承自struts-default的包.

作用: 方便管理我们的动作(struts-default是核心配置文件)

属性：
* abstract：可选值为true|false。说明他是一个抽象包。抽象包中没有action元素的。(默认为false)
* name：包名。不能重复。方便管理动作的。
* namespace：名称空间
* extends：继承什么

~~~~~~
<!-- name 包名称，在struts2的配置文件文件中 包名不能重复 ，name并不是真正包名，只是为了管理Action  -->
<!-- namespace 和 <action>的name属性，决定 Action的访问路径  （以/开始 ） -->
<!-- extends 继承哪个包，通常开发中继承 struts-default 包 （struts-default包在 struts-default.xml定义 ） -->
<package name="default" namespace="/" extends="struts-default"></package>
~~~~~~

## action标签 ##

* name: 必须的, 动作名称
~~~~~~
<package name="p2" extends="struts-default">
    <!-- 只要找不到的action的name，找act4。默认动作名称 -->
    <default-action-ref name="act4"></default-action-ref>
</package>
~~~~~~
* class：可选的. 默认值是com.opensymphony.xwork2.ActionSupport
~~~~~~
<package name="p2" extends="struts-default">
    <!-- 只要找不到的action的class，找com.opensymphony.xwork2.ActionSupport。默认class -->
    <default-class-ref name="com.opensymphony.xwork2.ActionSupport"></default-class-ref>
</package>
~~~~~~
* method: 可选. 默认值是 `public String execute(){return "success"}`

## result标签 ##

type：默认值dispatcher. 转发，目标JSP

name：默认值是success.

~~~~~~
<package name="default" namespace="/test" extends="struts-default">
    <action name="hello" class="com.itheima.action.HelloAction" method="execute">
        <result name="female">/female.jsp</result>
        <result name="male">/male.jsp</result>
    </action>
</package>
~~~~~~

访问包中带有名称空间的动作时：
`http://localhost:8080/day22_01_strutsHello/test/hello.action`
`http://localhost:8080/day22_01_strutsHello/test/aaa/bbb/hello.action`

动作有搜索顺寻：
1. 从/test/aaa/bbb找，不存在
2. 从/test/aaa找，不存在
3. 从/test，找到了
4. 一旦找到就不向上找了

## 默认Action 和 Action的默认处理类 ##

默认Action ， 解决客户端访问Action不存在的问题 ，
客户端访问Action， Action找不到，默认Action 就会执行
~~~~~~
<default-action-ref name="action元素的name" />
~~~~~~

默认处理类 ，客户端访问Action，已经找到匹配`<action>`元素，
但是`<action>`元素没有class属性，执行默认处理类
~~~~~~
<!-- 在struts-default.xml 配置默认处理类 ActionSupport  -->
<default-class-ref class="完成类名" />
~~~~~~

## 常量配置 ##
在 struts2-core-*.jar 的`org.apache.struts2`的 default.properties文件中存在一些内置常量

可以在 struts.properties, struts.xml, web.xml 
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

## struts2 配置文件分离 ##
通过 `<include file="struts-part1.xml"/>` 将struts2 配置文件 拆分 

# Action 的访问 #
xwork 是一种标准的命令模式(执行`exexute()`)

1. Action可以是 POJO (PlainOldJavaObjects)简单的Java对象,
不需要继承任何父类，实现任何接口
2. 编写Action 实现Action接口
~~~~~~
// Action接口中，定义默认五种 逻辑视图名称
// 五种逻辑视图，解决Action处理数据后，跳转页面
public static final String SUCCESS = "success";  // 数据处理成功 （成功页面）
public static final String NONE = "none";  // 页面不跳转  return null; 效果一样
public static final String ERROR = "error";  // 数据处理发送错误 (错误页面)
public static final String INPUT = "input"; // 用户输入数据有误，通常用于表单数据校验 （输入页面）
public static final String LOGIN = "login"; // 主要权限认证 (登陆页面)
~~~~~~
3. 编写Action, 继承ActionSupport(推荐), 在Action中使用 表单校验、错误信息设置、读取国际化信息 三个功能

~~~~~~
// 方式二
public class MyAction implements com.opensymphony.xwork2.Action {
    public String execute(){
        return SUCCESS; // Action 中定义的常量, 匹配配置文件 struts.xml 中的 action.name
    }
}
// 方式三
public class MyAction extends com.opensymphony.xwork2.ActionSupport {
    public String execute() { }
}
~~~~~~


# Action的方法调用 #

1. 在配置 `<action>` 元素时，没有指定method属性， 默认执行 Action类中 execute方法
~~~~~~
<action name="request1" class="cn.itcast.struts2.demo3.RequestAction1" />
~~~~~~
2. 在 `<action>` 元素内部 添加 method属性，指定执行Action中哪个方法
~~~~~~
<!-- 执行 RegistAction 的regist方法 -->
<action name="regist" class="cn.itcast.struts2.demo4.RegistAction" method="regist"/> 
 <!-- 将多个请求 业务方法 写入到一个Action 类中 -->
 <action name="addBook" class="cn.itcast.struts2.demo4.BookAction" method="addBook" ></action>
 <action name="delBook" class="cn.itcast.struts2.demo4.BookAction" method="delBook" ></action>
~~~~~~
3. 使用通配符* ，简化struts.xml配置
~~~~~~
<a href="${pageContext.request.contextPath }/user/customer_add.action">添加客户</a>
<a href="${pageContext.request.contextPath }/user/customer_del.action">删除客户</a>

<!-- struts.xml -->
<!-- {1}就是第一个* 匹配内容 -->
<action name="customer_*" class="cn.itcast.struts2.demo4.CustomerAction" method="{1}"></action>

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

## 动态方法调用 ##
访问Action中指定方法，不进行配置
* 在工程中使用 动态方法调用 ，必须保证
`struts.enable.DynamicMethodInvocation = true` 常量值 为true
* 在action的访问路径 中 使用 "!方法名"

~~~~~~
<!-- 页面 -->
<a href="${pageContext.request.contextPath }/user/product!add.action">添加商品</a>
<!-- 配置 -->
<action name="product" class="cn.itcast.struts2.demo4.ProductAction"></action>
<!-- 执行 ProductAction 中的 add方法 -->
~~~~~~

# Action中使用 Servlet #
1. 使用ActionContext 对象, 解耦合方式
~~~~~~
actionContext = ActionContext.getContext();
// 获得所有请求参数Map集合
actionContext.getParameters();
// actionContext.get("company") 对request范围存取数据
actionContext.put("company", "传智播客");
// 获得session数据Map，对Session范围存取数据
actionContext.getSession();
// 获得ServletContext数据Map，对应用访问存取数据
actionContext.getApplication(); 
~~~~~~
2. ServletActionContext的静态方法可以得到Servlet相关的对象
~~~~~~
//用Servlet相关的对象request response servletContext HttpSession
HttpServletRequest request = ServletActionContext.getRequest();
HttpServletResponse response = ServletActionContext.getResponse();
ServletContext sc = ServletActionContext.getServletContext();
HttpSession session = request.getSession();
~~~~~~
3. 使用接口注入的方式，操作Servlet API(耦合)

    Action实现如下接口，struts框架则会为其注入相应的Servlet API对象：
    `ServletRequestAware`, `ServletResponseAware`, `ServletContextAware`,
    实现其他对象或者功能，参考拦截器servletConfig

# 基于注解的开发 #
注解基于约定, 根据默认规则, 实现无配置文件

## 约定实现 ##
1. 导入jar包  11个jar  +  struts2-convention-plugin-2.3.7.jar
2. 在web.xml 配置前端控制器
3. 编写页面
4. 插件中 plugin配置文件
~~~~~~
<!-- 编写Action类，必须位于 action,actions,struts,struts2 四个包中 -->
<constant name="struts.convention.package.locators" value="action,actions,struts,struts2"/>
<!-- 以Action结尾 -->
<constant name="struts.convention.action.suffix" value="Action"/>
<!-- 结果result页面存放位置 -->
<constant name="struts.convention.result.path" value="/WEB-INF/content/"/>

<!-- Action被扫描后，如何确定Action的访问路径的 ？ -->
<!-- HelloAction位于直接位于四个扫描包下，namespace是/，Action的name是hello, /hello.action -->
cn.itcast.struts2.HelloAction
<!-- BookSearchAction 不是直接位于四个扫描包下，namespace是/books, Action的name是book-search -->
<!-- 访问路径 /books/book-search.action -->
cn.itcast.actions.books.BookSearchAction 
<!-- 访问 /user/user.action -->
cn.itcast.struts.user.UserAction
<!-- 访问 /test/login.action -->
cn.itcast.estore.action.test.LoginAction  
~~~~~~
5. 根据常量配置 结果页面 位于 `/WEB-INF/content`下

    页面命名规则约定： actionName + resultCode + suffix
	例如： cn.itcast.struts.user.UserAction  
    /user/user.action 返回 SUCCESS  
    结果页面 /WEB-INF/content/user/user-success.jsp  
    找不到 /WEB-INF/content/user/user-success.html  
    找不到 /WEB-INF/content/user/user.jsp  

## 注解实现 ##
注解开发第一步 基于约定的自动扫描

约定只解决Action访问和结果页面跳转问题
* 在开发中需要为Action指定拦截器，进行更细节result配置
* 约定不够灵活，注解的功能 是和 xml配置方式 等价的

`<constant name="struts.convention.classes.reload" value="false" />` Action类文件重新自动加载

~~~~~~
@NameSpace("/user")
@ParentPackage("struts-default")
public class UserAction extends ActionSupport {
   // `@ParentPackage` 配置`<package>` 继承哪个包
   // `@Namespace`  配置包名称空间
   // 使用 `@Action` 注解配置访问路径  `@Result` 注解 配置结果页面
    @Action(value="login", results=@Result(name="success", location="/index.jsp"))
    public String execute(){
        return "success"
    }
    @Actions(value={
      @Action(value="login", results=@Result(name="success", location="/index.jsp"))
      , @Action(value="login2", results=@Result(name="success", location="/index.jsp"))
    })
    public String execute2(){
        return "success"
    }
}
~~~~~~

# 结果页面的配置 #

Action处理请求后， 返回字符串(逻辑视图名), 需要在struts.xml 提供 `<result>`元素定义结果页面

局部结果页面 和 全局结果页面
~~~~~~
<action name="result" class="cn.itcast.struts2.demo6.ResultAction">
    <!-- 局部结果  当前Action使用 -->
 	<result name="success">/demo6/result.jsp</result>
</action>
<global-results>
    <!-- 全局结果 当前包中 所有Action都可以用-->
	<result name="success">/demo6/result.jsp</result>
</global-results>
~~~~~~

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
### plainText ###
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
