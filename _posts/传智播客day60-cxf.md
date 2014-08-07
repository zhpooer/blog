title: 传智播客day60-CXF
date: 2014-07-19 09:41:34
tags:
- 传智播客
---

# 入门 #

Apache CXF = Celtix + Xfire, 开源 web Service 框架, 支持多种协议,
soap1.2,1.1, XML/HTTP, RESTful, CORBA

Cxf基于SOA总线结果, 依靠 spring 完成模块的集成

# demo #

## 服务端 ##
~~~~~~
@WebService(targetNamespace="http://itcast.cn",
       serviceName="WeatherService")
// 如果要使用1.2协议, 要加上这个注解
@BindingType(javax.xml.ws.soap.SOAPBinding.SOAP12HTTP_BINDING) 
public interface IWeatherService {
    @WebMethod(operationName="queryWeatherByCityName")
    public @WebResult(name="weatherResult") String queryWeatherByCityName(@WebParam("cityname") String city){}
}

public class WeatherService implements IWeatherService {
    public String queryWeatherByCityName(String cityName){
        return "晴朗";
    }
}

publci static void main(String[] args) {
    JaxWsServerFactoryBean jaxWsServerFactoryBean = new JaxWsServerFactoryBean();
    /// webservice服务地址
    jaxWsServerFactoryBean.setAddress("http://ip:port/weather");
    // 设置 prottype
    jaxWsServerFactoryBean.sestServiceClass(WeaterServiceInterface.class);
    // 设置 serviceBean(服务运行实例)
    jaxWsServerFactoryBean.setServiceBean(new WeatherServiceImpl.class);

    // 添加输入拦截器, 在运行代码前执行
    jaxWsServerFactoryBean.getInInterceptpor().add(new LoggingInInterceptor());
    // 添加输出拦截器,  在运行代码后执行
    jaxWsServerFactoryBean.getOutInerceptpor().add(new LoggingOutInterceptor());

    // 发布服务
    jaxWsServerFactoryBean.create();
}
~~~~~~

## 客户端 ##

~~~~~~
// 产生客户端代码
// wsdl2java -d . http://ip:port/weather?wsdl
public class WeatherClient {
     public static void main(String [] args) {
         JaxWsProxyFactoryBean jaxWsProxyFactoryBean = new JaxWsProxyFactoryBean();
         // 设置的调用地址
         jaxWsProxyFactoryBean.setAddress("http://ip:port/weather");
         // 设置portType
         jaxWsProxyFactoryBean.setServiceClass(WeatherService.class);
         // 获取调用实例
         WeatherServiceInterface weatherServiceInterface = jaxWsProxyFactoryBean.create();
         // 调用
         String result = weatherServiceInterface.queryWeather("郑州");
     }
}
~~~~~~

# Cxf 与 spring 集成 #
~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xmlns:jaxrs="http://cxf.apache.org/jaxrs"
       xmlns:cxf="http://cxf.apache.org/core"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
                           http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
                           http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">
    <!-- 也可以这样定义  -->
    <!-- <jaxws:endpoint address="" implementor=""> -->
    <!-- </jaxws:endpoint> -->
    <jaxws:server id="weather" address="/weather" 
           serviceClass="cn.itcast.ws.cxf.server.WeatherInterface">
        <!-- 发布服务的类 -->
        <jaxws:serviceBean>
            <!-- 也可以这样 <ref bean="weatherServer"/> -->
            <bean class="cn.itcast.ws.cxf.server.WeatherInterfaceImpl"></bean>
        </jaxws:serviceBean>
        <jaxws:inInterceptors>
            <!-- 输入日志拦截器 -->
            <bean class="LoggingInIntercetpor"> </bean>
        </jaxws:inInterceptors>
        <jaxws:outInterceptors>
            <!-- 输出日志拦截器 -->
            <bean class="LoggingOutIntercetpor"> </bean>
        </jaxws:outInterceptors>
    </jaxws:server>
</beans>
~~~~~~

~~~~~~
<!-- web.xml -->
<servlet>
    <description>Apache CXF Endpoint</description>
    <display-name>cxf</display-name>
    <servlet-name>cxf</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>cxf</servlet-name>
    <url-pattern>/ws/*</url-pattern>
</servlet-mapping>
~~~~~~

## 客户端调用 ##

~~~~~~
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xmlns:jaxrs="http://cxf.apache.org/jaxrs"
       xmlns:cxf="http://cxf.apache.org/core"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
                           http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
                           http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">

<jaxws:client id="weatherClient" address="http://ip:port/ws/weather"
        serviceClass="WeatherServiceInterface">
</jaxws:client>
</beans>
~~~~~~

~~~~~~
public static void main (){
     ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
     WeatherServiceInterface weatherClient = applicationContext.getBean("weatherClient");
     String result = weatherClient.query("郑州");
}
~~~~~~

# 身份过滤 #

Apache WSS4J（WebService Security For Java）实现了JAVA 语言的 WS-Security，
CXF 中使用拦截器机制完成 WSS4J 功能的支持 WSS4JInInterceptor 和 WSS4JOutInterceptor

`WSS4JInInterceptor` 输入拦截器，用于服务端校验密码

`WSS4JOutInterceptor` 输出拦截器，用于客户端发送密码

## 服务端 ##
~~~~~~
<!-- 配置安全认证拦截器 -->
<jaxws:inInterceptors>
  <bean class="org.apache.cxf.interceptor.LoggingInInterceptor" />
  <bean id="wss4jInInterceptor" class="org.apache.cxf.ws.security.wss4j.WSS4JInInterceptor">
    <constructor-arg>
      <map>
         <!-- 认证方式 -->
         <entry key="action" value="UsernameToken" />
         <!-- 加密方式 PasswordDigest：md5加密，PasswordText：明文-->
         <entry key="passwordType" value="PasswordText" />
         <!-- 密码回调方法 -->
         <entry key="passwordCallbackClass" value="cn.itcast.ws.cxf.server.interceptor.ServerPasswordCallbackHandler" />
      </map>
    </constructor-arg>
  </bean>
</jaxws:inInterceptors>
~~~~~~

~~~~~~
public class ServerPasswordCallbackHandler implements CallbackHandler {
    public void handle(Callback[] callbacks) {
         // 校验用户身份
         WSPasswordCallback wsPwCallback = callbacks[0];
         String userid_fromclient = wsPwCallback.getIdentifier();
         ///  如果校验用户身份通过这是将
         if(userid_fromclient.equals(userid)) {
             // 用户身份校验通过
             wsPasswordCallback.setPassword(passwd); //填充正确密码
         } else {
             throw new IOException("用户身份不合法");
         }
    }
} 
~~~~~~

## 客户端 ##

~~~~~~
<!-- 客户端调用实例 -->
<jaxws:client id="weatherServicePort"
  address="http://localhost:8080/14webservice_cxf_spring/ws/weather?wsdl" serviceClass="webservice.itcast.cn.WeatherServicePort">
    <jaxws:outInterceptors>
        <ref bean="wss4jOutInterceptor" />
    </jaxws:outInterceptors>
</jaxws:client>

<bean id="wss4jOutInterceptor" class="org.apache.cxf.ws.security.wss4j.WSS4JOutInterceptor">
  <constructor-arg>
    <map>
      <!-- 用户认证方式与服务端一致 -->
      <entry key="action" value="UsernameToken" />
      <!-- 初始化用户令牌，不能为空, 是默认值, 可以在 callback 中重新赋值  -->
      <entry key="user" value="mrt" />
      <!-- 密码加密方式 ,PasswordDigest：md5加密，PasswordText：明文-->
      <entry key="passwordType" value="PasswordDigest" />
      <!-- 密码回调 -->
      <entry key="passwordCallbackClass"
          value="cn.itcast.ws.cxf.client.interceptor.ClientPasswordCallbackHandler" />
    </map>
  </constructor-arg>
</bean>
~~~~~~

~~~~~~
public class ClientPasswordCallbackHandler implements CallbackHandler {
    public void handle(Callback[] callbacks) {
        WSPasswordCallback wspassCallback = (WSPasswordCallback) callbacks[0];
		wspassCallback.setPassword(PASSWORD);
		wspassCallback.setIdentifier(USER);
    }
}
~~~~~~

# 便民查询网站 #

使用springmvc+CXF实现便民网站，同时对外提供webservice服务。
调用别人webservice，将自己的服务发布成webservice

![案例分析](/img/webservice_demo.png)

~~~~~~
public class WeatherClient {
    // 从公网查询天气, 将查询到的内容全部返回
    public List<String> queryWeatherByCityName(String cityname) {
        // 获取服务视图
        WeatherWebService weatherWebService = new WeatherWebService();
        // 获取 portType
        WeatherWebServiceSoap weatherWebServiceSoap =  weatherWebService.getWeatherWebServiceSoap();
        ArrayOfString arrayOfString = weatherWebServiceSoap.getWeatherbyCityName(cityname);
        List<String> resultlist = arrayOfString.getString();
        return resultlist;
    }
}

// 天气查询的结果信息
public class Weather {
    private String result; // 天气概况
    private String img; // 天气图片
    private String img2; // 天气图片
}

public interface IWeatherService {
    public List<Weather> queryWeatherByCityName(String cityname);
}

public class WeatherServiceImpl implements IWeatherService {
    @Autowired
    private WeatherClient weatherClient;
    
    public List<Weather> queryWeatherByCityName(String cityname){
        List<String> resutsList = weatherClient.queryWeatherByCityName(cityname);

        List<Weather> list = new ArrayList<Weather>();
        
        WeatherModel weatherModel = new WeatherModel();
        // ... list.weatherModel();
        return list;
    }
}

// 天气查询控制层
@Controller
public class WeatherAction {

    @Autowired
    WeatherServiceInterface weatherServiceInterface;
    
    @RequestMapping("/queryWeather")
    public String queryWeather(String cityName, Model model) {
        // 调用服务层未来查询三天的天气
        List<Weather> list = weatherServiceInterface.queryWeatherByCityName(cityName);
        model.addAttribute("weatherresult", list);
        return "queryweather"; // 返回天气查询页面
    }
}
~~~~~~


~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xmlns:jaxrs="http://cxf.apache.org/jaxrs" xmlns:cxf="http://cxf.apache.org/core"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
				            http://www.springframework.org/schema/beans/spring-beans.xsd
				            http://cxf.apache.org/jaxrs http://cxf.apache.org/schemas/jaxrs.xsd
				            http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
				            http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">
                            
<!-- 定义天气查询服务的bean -->
<bean id="weatherService" class="cn.itcast.ws.cxf.service.impl.WeatherServiceInterfaceImpl" />

<!-- 公网天气查询客户端 -->
<bean id="webWeatherClient" class="cn.itcast.ws.cxf.client.WeatherClient" />

<!-- 启动webservice服务，启动之后外边用户可以访问此webservice -->
<jaxws:server  id="weatherServer" address="/weather" serviceClass="cn.itcast.ws.cxf.service.WeatherServiceInterface">
    <jaxws:serviceBean>
        <ref bean="weatherService"/>
    </jaxws:serviceBean>
    <!-- 添加日志拦截器 -->
    <jaxws:inInterceptors>
        <!--输入日志拦截器 -->
        <bean class="org.apache.cxf.interceptor.LoggingInInterceptor"/>
    </jaxws:inInterceptors>
    <!-- 输出日志拦截器 -->
    <jaxws:outInterceptors>
        <bean class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
    </jaxws:outInterceptors>
</jaxws:server>

</beans>
~~~~~~
`springmvc-servlet.xml`
~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
        http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd 
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-3.1.xsd 
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-3.1.xsd 
        http://www.springframework.org/schema/tx 
        http://www.springframework.org/schema/tx/spring-tx-3.1.xsd ">
        
<!-- 如果使用组件扫描就不用再配置contrller的bean -->
<context:component-scan base-package="cn.itcast.ws.cxf.action" />
<!-- 使用注解驱动带替下边配置的映射器和适配置器 -->
<mvc:annotation-driven />
<!-- 注解映射器 -->	
<!-- <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" /> -->
<!-- 注解适配器 -->
<!-- <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"></bean>
        </list>
    </property>
</bean> -->

<!-- jsp视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
    <!-- 前缀 -->
    <property name="prefix" value="/WEB-INF/jsp/"></property>
    <!-- 后缀 -->
    <property name="suffix" value=".jsp"></property>
</bean>

</beans>
~~~~~~
`web.xml`
~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">
    <display-name>07193webservice_cxf_spring</display-name>
    <!-- 加载spring环境 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/classes/applicationContext.xml</param-value>
    </context-param>
    <listener>
      <listener-class> org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 配置cxf的servlet -->
    <servlet>
        <servlet-name>cxf</servlet-name> <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>cxf</servlet-name>
        <url-pattern>/ws/*</url-pattern>
    </servlet-mapping>
    <!-- springmvc的前端控制器 -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 如果这里不配置contextConfigLocation，会自动找"servlet名称+-servlet.xml"-->
        <init-param>
             <param-name>contextConfigLocation</param-name>
             <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>
    <!-- post提交解析乱码过虑器 -->
    <filter>
        <filter-name>CharacterEncoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  	    <init-param>
  		    <param-name>encoding</param-name>
  		    <param-value>utf-8</param-value>
  	    </init-param>
    </filter>
    <filter-mapping>
  	    <filter-name>CharacterEncoding</filter-name>
  	    <url-pattern>/*</url-pattern>
    </filter-mapping>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>
</web-app>
~~~~~~
