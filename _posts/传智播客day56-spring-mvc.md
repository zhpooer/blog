title: 传智播客day56-Spring mvc
date: 2014-07-13 09:05:32
tags:
- 传智播客
- Spring MVC
---

# SpringMVC 原理 #

前端控制器(DispatcherServlet) 通过 映射处理器(HandlerMaping)
找到 处理器(Handler) 和拦截器(HandlerInterceptor),
它们被封装到 HandlerExecutionChain.

前端控制器通过适配器(HandlerAdapter)调用 处理器(Handler),
处理器返回模型和视图(ModelAndView).

前端控制器把通过视图解析器(ViewResolver)解析出视图(View),
再把视图渲染, 返回给用户

# DEMO #

~~~~~~
<servlet>
    <servlet-name>springmvc </servlet-name>
    <servlet-class> org.springframework.web.servlet.DispatcherServlet </servlet-class>
    <load-on-startup> 1 </load-on-startup>
    <!-- 如果不配置也会默认找 springmvc-servlet.xml -->
    <init-param>
        <param-name>
        contextConfigLocation
        </param-name>
        <param-value>
        classpath:springmvc-servlet.xml
        </param-value>
    </init-param>
</servlet>
<!-- post提交解决乱码过滤器 -->
<filter>
    <filter-name> CharactorEncoding</filter-name>
    <filter-class> CharactorEncodingFilter </filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name> CharactorEncoding </filter-name>
    <url-pattern> /* </url-pattern>
</filter-mapping>

<servlet-mapping>
    <servlet-name> springmvc </servlet-name>
    <url-pattern> *.action </url-pattern>
</servlet-mapping>
~~~~~~
`springmvc-servlet.xml`

~~~~~~
<!-- 映射器 -->
<!-- 将bean的名字当做请求的url -->
<bean class="web.servlet.handler.BeanNameUrlHandlerMapping"></bean>

<!-- 简单url映射器,和 beannamehandler 可以共存-->
<!-- <bean class="SimpleUrlhandlerMapping"> -->
<!--     <property name="mappings"> -->
<!--         <props> -->
<!--             <prop key="/hello.action">hello_controller</prop> -->
<!--         </props> -->
<!--     </property> -->
<!-- </bean> -->

<!-- 适配器 -->
<!-- 所有实现了 mvc.Controller 都可以作为后端控制器 -->
<bean class="SimpleControllerHandlerAdaptor">
    <!-- 配置前缀和后缀  -->
    <!-- 在设置后, 在设置视图时 /WEB-INFO/jsp/hello.jsp => hello -->
    <property name="prefix" value="/WEB-INF/jsp/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>

<!-- 需要实现 HttpRequestHandler 接口  -->
<!-- <bean class="HttpRequestHandlerAdaptor"></bean> -->

<!-- jsp 视图解析器 -->
<bean class="InternalResourceViewResolver"> </bean>

<!-- 控制器, 可以访问 hello.action -->
<bean id="hello_controller" name="/hello.action" class="HelloWorldController">
</bean>
~~~~~~

~~~~~~
public class HelloWorldController implements Controller {
    public ModelAndView handleRequest(reqeust, response) {
        ModelAndView modelAndView = new ModelAndView();
        // 设置模型数据
        modelAndView.addObject("message", "hello world");
        // 设置 jsp 地址
        modelAndView.setViewName("hello");
        return modelAndView;
    }
}
// hello.jsp: ${message}
~~~~~~



# 其他控制器 #
~~~~~~
<!-- 参数控制器 实现简单的视图转发, 转发到 success.jsp -->
<bean name="/success.action" class="ParameterizableViewController">
    <property name="viewName" value="success"></property>
</bean>


~~~~~~

~~~~~~
// <!-- 命令控制器 -->, 自动完成 pojo 的注入
public class MyCommandController extends AbstractCommandController {
    public MyCommandController() {
        this.setCommandClass(Student.class);
    }
    @Override protected ModelAndView handler(request, response, object, errors) {
        Student studuent = ojbect;
        println(student);
        return null;
    }
    
    // 注册字符串转日期
    @Override protected void initBnder(request, binder) {
        // true 为, 允许为空
        binder.registerCustomerEditor(Date.class, new CustomerDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
    }
}

// 表单控制器
// 配置:
// property name=formView value=userform, 配置表单视图
// property name=sucess value=success, 配置成功页面
public class MyFormContorller extends SimplerFormController{
    public MyformController(){
        this.setCommandClass(Student.class);
        this.setCommandName("student");
    }
    // 通过get 请求进入此方法
    @Override protected showForm(){}
    // 通过post 请求进入此方法
    @Override public void doSubmitAciton(Object command){}
}
~~~~~~

参数控制器, 命令控制器, 表单控制器缺点:
传递参数格式固定为命令对象, 不灵活, controller中的方法固定,
不能任意定义控制方法


# 使用注解 #

~~~~~~
<context:component-scan base-package="action"></context:component-scan>
<!-- 注解Mapping -->
<bean class="RequestMappingHandlerMapping"></bean>
<!-- 注解适配器 -->
<bean class="RequestMappingHandlerAdaptor"></bean>
<!-- 视图解析器 -->
<bean class="InternalResourceViewResolver">...</bean>
~~~~~~

~~~~~~
@Controller
public class HelloWorldController {
    // 控制器的方法
    @RequestMapping(value="/hello") // 配置访问的链接
    public String helloworld(HttpServletRequest request, Student student, Model model){
        model.addAttribute("message", "HelloWorld"); // 通过model将参数传回页面
        // view: ${student.name}
        model.addAttribute("student", new Student("张三"));
        return "hello"; // 返回的视图, 经过视图解析器解析成 hello.jsp
    }
}
~~~~~~

## URI ##

~~~~~~
@Controller
@ResourceeMapping("/user") // 设置模块路径
public class UserAction {
    @RequestMapping("/userlist")
    public String userlist(Model model) throws Exception {
        return "user/userlist";
    }
    
    @RequestMapping("/useradd")
    // 使用model, modelMap, map 都可以将数据传回页面, 都是一样di``
    public String useradd(Request req, Response res,
         Session session, Model model,
         Map map, ModelMap modelMap,
         
    ) {
        return "user/useradd";
    }
    
    @RequestMapping("/useraddsubmit", method=RequestMethod.POST)
    // 如果对象以属性点的方式来提交数据, 如 student.**, 那么可以这样做 User{student}
    public String useraddsubmit(Model model, User student) {
        return "redirect:/user/userlist.action"; // 重定向
    }

    // 通过模板设置 userid, 以及设定方法
    @RequestMapping(value="/useredit/{userid}", method={RequestMethod.GET, RequestMethod.POST})
    // 也可以通过正则表达式来限定
    @RequestMapping("/useredit/{userid:\\d+}")
    public String useredit(@PathVariable String usesrid, Model model) {
    }
    // 批量删除
    @RequestMapping("/userdeletelist")
    // 提交checkbox
    public String deletelist(String[] deleteid){
    
        return "success";
    }

    // 批量添加
    // User{studnets:List}
    // name=studnets[0...].name ...
    // 提交list给服务器
    // 同理也可以将Map提交给服务器 name=scores['name']
    @RequestMapping("/useraddlist")
    public String useraddlist(Model model, User user,
      @RequestPram(defaultValue="2", required=true, value="group" ) String groupid){
        return "user/useraddlist";
    }
    
    // 注册类型转换器
    @InitBinder
    protected void initBnder(request, binder) {
        // true 为, 允许为空
        binder.registerCustomerEditor(Date.class, new CustomerDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
    }
}
~~~~~~

# 实现json数据交互 #

`@RequestBody`, 注解用户读取http请求的内容(字符串), 通过springMVC
提供的HttpMessageConverter 接口将读到的内容转换为 json, xml
等格式的数据并绑定到controller方法的参数上

`@ResponseBody`, 该注解用于将Controller的方法返回的对象,
通过HttpMessageConverter接口转换为指定格式的数据如：json,xml等,
通过Response响应给客户端

Springmvc默认用 `MappingJacksonHttpMessageConverter` 对json数据进行转换，
需要加入jackson的包，`jackson-core-asl.jar`, `jackson-mapper-asl.jar`

~~~~~~
<!-- 注解适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
	    <list>
		    <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"></bean>
		</list>
	</property>
</bean>

~~~~~~

~~~~~~
@Controller
public class Body {
    @RequstMapping("/")
    // 请求参数是 json, 响应也是json
    public @ResponseBody Student requestjson(@RequestBody Student student) {
        return student;
    }
}
~~~~~~

# 简化配置 #
~~~~~~
<!-- 可以用如下配置简化其他注解配置 -->
<!--  如注解映射器, 注解适配器-->
<mvc:annotation-driven />
~~~~~~

# 拦截器 #

~~~~~~
public class HandlerInterceptor1 implements HandlerInterceptor {
    /**
    * controller执行前调用此方法
    * 返回true表示继续执行，返回false中止执行
    * 这里可以加入登录校验、权限拦截等
    */
    @Override
    public boolean preHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler) throws Exception {
        // false 表示被拦截, 不执行 controller 方法
        // true 表示方向
        return false;
    }
    
    /**
    * controller执行后但未返回视图前调用此方法
    * 这里可在返回用户前对模型数据进行加工处理，
    * 比如这里加入公用信息以便页面显示
    */
    @Override
    public void postHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler,
        ModelAndView modelAndView) throws Exception {
        
    }
    /**
    * controller执行后且视图返回后调用此方法
    * 这里可得到执行controller时的异常信息
    * 这里可记录操作日志，资源清理等
    */
	@Override
	public void afterCompletion(HttpServletRequest request,
        HttpServletResponse response, Object handler, Exception ex) {
	}

}

~~~~~~


~~~~~~
<!-- 对某种映射器配置拦截器 -->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="interceptors">
        <list>
            <ref bean="handlerInterceptor1" />
            <ref bean="handlerInterceptor2" />
        </list>
    </property>
</bean>
<bean id="handlerInterceptor1" class="springmvc.intercapter.HandlerInterceptor1" />
<bean id="handlerInterceptor2" class="springmvc.intercapter.HandlerInterceptor2" />

<!-- 针对所有mapping配置全局拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**">
        </mvc:mapping>
        <bean class="HandlerIntercetpor1"></bean>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/**">
        </mvc:mapping>
        <bean class="HandlerIntercetpor2"></bean>
    </mvc:interceptor>
</mvc:interceptors>
~~~~~~

## 用户认证DEMO ##
~~~~~~
public class LoginInterceptor implements HandlerInterceptor{
    @Override
    public boolean preHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler) throws Exception {
        //如果是登录页面则放行
        if(request.getRequestURI().indexOf("login.action")>=0){
            return true;
        }
        HttpSession session = request.getSession();
        //如果用户已登录也放行
        if(session.getAttribute("user")!=null){
            return true;
        }
        //用户没有登录挑战到登录页面
        request.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(request, response);
        
        return false;
    }
}
~~~~~~
