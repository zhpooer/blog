title: 传智播客day29-struts拦截器
date: 2014-05-14 11:09:51
tags:
- 传智播客
- struts
---

# Struts的工作原理及核心过滤器 #

StrutsPrepareAndExecuteFilter过滤器其实是包含2部分的
1. StrutsPrepareFilter:做准备
2. StrutsExecuteFilter：进入Struts2的核心处理。
如果是Struts2的请求就会进入该过滤器，处理完后，不放行（由结果类负责显示）。
如果是非Struts2的请求，比如默认jsp的请求，直接放行。

如果用不到其他过滤器，配置StrutsPrepareAndExecuteFilter即可;
如果用到其他过滤器，还需要使用Struts2准备好的环境，
使用`StrutsPrepareFilter`，`StrutsExecuteFilter`个过滤器，其他过滤器放在两者之间.
~~~~~~
<filter-mapping>
    <filter-name> struts-prepare </filter-name>
    <url-pattern>/* </url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name>sitemesh </filter-name>
    <url-pattern>/* </url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name> struts-execute </filter-name>
    <url-pattern>/* </url-pattern>
</filter-mapping>
~~~~~~

![struts core](/img/struts_core.png)
showcase: 各种应用的案例，在struts2-showcase里面找各种案例

# 拦截器 #

拦截器的使用 ，源自Spring AOP(面向切面编程)思想, 采用 责任链 模式
*  在责任链模式里,很多对象由每一个对象对其下家的引用而连接起来形成一条链。
*  责任链每一个节点，都可以继续调用下一个节点，也可以阻止流程继续执行

拦截器的目的: 如果一个业务逻辑方法中涉及到的逻辑相当复杂,
可以把这些业务分离开, 如 启动日志, 权限检查, 文件上传, 保存用户,
把这四方面全面分开, 实现松耦合

常用struts2 拦截器
~~~~~~
<!-- 模型驱动 -->
<interceptor-ref name="modelDriven"/>
<!-- 文件上传 -->
<interceptor-ref name="fileUpload"/>
<!-- 参数解析封装 -->
<interceptor-ref name="params">
<!-- 类型转换错误 -->
<interceptor-ref name="conversionError"/>
<!-- 请求参数校验 -->
<interceptor-ref name="validation">
<!-- 拦截跳转 input 视图 -->
<interceptor-ref name="workflow"> 
~~~~~~


意义: 把一些和业务逻辑没有关联的代码放入拦截器, 以实现业务逻辑和其他代码的松耦合

## 拦截器初探 ##
所有实际开发中，自定义拦截器 只需要 继承 AbstractInterceptor类，
提供 intercept 方法实现
~~~~~~
public class InterceptorAction extend ActionSupport {
    public String saveUser() {
        ActionContext().getContext().put("message", "userSaved");
        return "privilege"
    }
}
// 拦截器
public class PrivilegeInterceptor implements Interceptor {
    public void destroy(){}
    public void init(){}
    public String intercept(ActionInvocation action) {
         String username = ServletActionContext.getRequest().getParameter("username")
         if("admin".equals(username)) {
             return action.invoke();
         } else {
             ActionContext().getContext().put("message", "权限不足");
             return "privilege"
         }
    }
}
~~~~~~
声明拦截器
~~~~~~
<package>
    <interceptors>
        <interceptor name="privilege" class="cn..**"> </interceptor>
        <!-- 声明一个拦截器栈 -->
        <interceptor-stack name="privilegeStack">
            <!-- 引用默认拦截器栈 -->
            <interceptor-ref name="defaultStack"> </interceptor-ref>
            <!-- 引用自己的连接器 -->
            <interceptor-ref name="privilege"> </interceptor-ref>
        </interceptor-stack>
    </interceptors>
    <!-- 让struts 执行声明的拦截器栈, 和拦截器 -->
    <default-interceptor-ref name="privilegeStack"/>
</package>
~~~~~~

## 概念 ##
1. 拦截器, 实现 Interceptor 接口的一个类
2. 拦截器栈, 把很多个拦截器集中在一起
3. struts有一个默认拦截器栈, 该栈在 `struts-default.xml` 中声明, 结构为: 
~~~~~~
<package name="struts-default">
    <interceptors>
        <!-- 声明拦截器 -->
        <interceptor name="name1" class=""> </interceptor>
        <interceptor name="name2" class=""> </interceptor>
        <!-- 声明拦截器栈 -->
        <interceptor-stack name="defaultStack">
            <!-- 引用拦截器栈 -->
            <interceptor-ref name="name1"> </interceptor-ref>
            <interceptor-ref name="name2"> </interceptor-ref>
        </interceptor-stack>
    </interceptors>
    <!-- 让struts 内部执行默认的拦截器栈, 和拦截器 -->
    <default-interceptor-ref name="defaultStack"/>
</package>
~~~~~~

## 拦截器执行 ##
执行顺序: 按照拦截器的声明顺序, 从上到下执行, 执行完拦截器以后, 再执行 `Action`
~~~~~~
<interceptor-stack name="privilegeStack">
<!-- 从上到下执行 -->
    <interceptor-ref name="defaultStack"> </interceptor-ref>
    <interceptor-ref name="privilege"> </interceptor-ref>
</interceptor-stack>
~~~~~~

一个 pacakge 中可以有
1. 结果集
2. 拦截器
3. action

可以通过继承package, 来复用 `interceptor` 配置信息
~~~~~~
<!-- 声明intercept包,在内部使用公用拦截器 -->
<package name="intercept" extends="struts-default"> </package>
<!-- 复用intercept拦截-->
<package extends="intercept"> </package>
<package extends="struts-default"> </package>
~~~~~~
## 案例: 执行效率统计拦截器 ##
建立拦截器ElapsedTimeInterceprot实现Interceptor接口
~~~~~~
//动作方法及结果处理耗时统计拦截器
public class ElapsedTimeInterceprot implements Interceptor {
    public String intercept(ActionInvocation invocation) throws Exception {
        long beginTime = System.nanoTime();//纳秒：1毫秒=1000000纳秒
        String result = invocation.invoke();//放行：拦截前要做的事放在invocation.invoke()之前，拦截后放在之后
        // 可以获得Action对象
        ActionSupport as =  actioninvocation.getAction();
        as.addFieldError("some wrong");
        //结果处理完毕后执行
        long endTime = System.nanoTime();
        System.out.println(invocation.getInvocationContext().getName()+"动作执行耗时："+(endTime-beginTime)+"纳秒");
        return result;
    }
    public void destroy() {}
    public void init() {}
}
~~~~~~
进行配置文件设置
~~~~~~
<package name="p1" extends="struts-default">
    <interceptors>
        <interceptor name="elapsedTime" class="com.itheima.interceptors.ElapsedTimeInterceprot"></interceptor>
    </interceptors>
    <action name="test1" class="com.itheima.action.HelloAction1" method="test1">
        <!-- 需要获取request，所以必须先执行servletConfig拦截器 -->
        <interceptor-ref name="defaultStack"></interceptor-ref>
        <!-- 执行耗时统计拦截器 -->
        <interceptor-ref name="elapsedTime"></interceptor-ref>
        <result>/1.jsp<result>
    </action>
</package> 
~~~~~~
拦截器的扩展，定义拦截器小组
~~~~~~
<package name="p1" extends="struts-default">
    <interceptors>
        <interceptor name="elapsedTime" class="com.itheima.interceptors.ElapsedTimeInterceprot"></interceptor>
        <!-- 自己定义一个拦截器小组 -->
        <interceptor-stack name="myDefaultStack">
            <interceptor-ref name="defaultStack"></interceptor-ref>
            <interceptor-ref name="elapsedTime"></interceptor-ref>
        </interceptor-stack>
    </interceptors>
    <action name="test1" class="com.itheima.action.HelloAction1" method="test1">
        <!-- 执行自定义的拦截器小组 -->
        <interceptor-ref name="myDefaultStack"></interceptor-ref>
        <result>/1.jsp<result>
    </action>
</package>
~~~~~~
继续扩展配置设置
~~~~~~
<package name="mypackage" extends="struts-default" abstract="true">
    <interceptors>
        <interceptor name="elapsedTime" class="com.itheima.interceptors.ElapsedTimeInterceprot"></interceptor>
        <interceptor name="sessionCheck" class="com.itheima.interceptors.SessionCheckInterceptors">
            <!-- 说明test2动作方法不需要拦截, SessioncheckInterceptor 需要继承 MethodFilterIntercetpor -->
            <param name="excludeMethods">user</param>
        </interceptor>
        <interceptor-stack name="myDefaultStack">
            <interceptor-ref name="defaultStack"></interceptor-ref>
            <interceptor-ref name="elapsedTime"></interceptor-ref>
            <interceptor-ref name="sessionCheck"></interceptor-ref>
        </interceptor-stack>
    </interceptors>
    <!-- 设置该包中的所有action配置默认拦截器 ，每个包只能指定一个默认拦截器 -->
    <default-interceptor-ref name="myDefaultStack"></default-interceptor-ref>
</package>
~~~~~~

## 案例: 用户登陆拦截器 ##
是否登陆拦截器MethodFilterInterceptor(可以配置是否进行拦截excludeMethods属性,如上)

权限判断拦截器继承MethodFilterInterceptor类，这样只对某些方法起作用，而对其他方法不起作用。 (配置文件如上)
~~~~~~
//执行动作方法前检查用户是否已经登录
public class SessionCheckInterceptors extends MethodFilterInterceptor{
    protected String doIntercept(ActionInvocation invocation) throws Exception {
        String result = "login";//对应的就是一个结果
        HttpSession session = ServletActionContext.getRequest().getSession();
        User user = (User)session.getAttribute("user");
        if(user!=null)
            //如果用户有登录，则放行。
            result = invocation.invoke();
        //如果用户没有登录，则返回结果集。
        return result;
    }
}
~~~~~~
配置文件需要在结果集中增加一个:
~~~~~~
<result name="login">/login.jsp</result>
~~~~~~

# 过滤器 和 拦截器 #
使用Filter 进行权限控制, 过滤所有web请求(所有web资源访问)

使用拦截器 进行权限控制 ---- 主要拦截对Action访问(不能拦截JSP)

# Tip #
~~~~~~
this.addActionError("");  // 业务逻辑错误
this.addFieldError("");   // 验证错误
~~~~~~

