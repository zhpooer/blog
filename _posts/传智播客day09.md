title: 传智播客day09
date: 2014-04-10 08:33:53
tags:
- servlet
---

# Servlet #
servlet是运行在服务器中的动态资源, 能接收用户的请求,发出响应.

## 创建 Servlet 步骤##

~~~~~~
package cn.itcast;
import javax.servlet.*;
import java.io.*;

public class HelloServlet extends GenericServlet {
    @Override
    public void service(ServletRequest req, ServletRespons res)
                        throws IOExcetpion, ServletException {
    }
}
~~~~~~

~~~~~~
set classpath=%classpath;c:\*.jar ; 在windows上
export CLASSPATH=$CLASSPATH:*.jar ; 在linux上
javac -d . HelloServlet.java
~~~~~~

~~~~~~
<web-app>
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>cn.itcast.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>HelloServlet</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file> </welcome-file>
    </welcome-file-list>
</web-app>
~~~~~~

## servlet的生命周期 ##
~~~~~~
public class HelloServlet extends GenericServlet {
   //只实例化一次
    public HelloServlet(){
         System.out.println("调用了一次");
    }

    //用户第一次访问
    @Override
    public void init(ServletConfig conf) throws ServletException {
         System.out.println("只调用了一次");
    }
    
    @Override
    public void destroy(){
         System.out.println("只调用了一次");
    }

    @Override
    public void service(ServletRequest req, ServletRespons res)
                        throws IOExcetpion, ServletException {
         System.out.println("每次访问都调用");
    }
}
~~~~~~

## 编写servlet方法 ##
1. 编写类直接实现javax.servlet.Servlet接口
2. 编写类直接继承javax.servlet.GernericServlet 接口
3. 编写类直接继承javax.servlet.http.HttpServlet 接口

    原因: 服务端编程都是基于HTTP协议的

    javax.servlet.\*, javax.servlet.http.\*, 一个具体包是实现了http协议
~~~~~~
/*
* 建议不要重写service方法
* service(HttpServletRequest req, HttpServletResponse resp)
*/
public class MyServlet extends HttpServlet {
     @Override public void doGet(HttpServletRequest res, HttpServletResponse req){}
     @Override public void doPost(HttpServletRequest res, HttpServletResponse req){}
     @Override public void doPut(HttpServletRequest res, HttpServletResponse req){}
}
~~~~~~

## servlet 的一些细节##
1. 一个servlet可以被映射到多个地址
    
2. servlet的映射可以使用通配符\*
    方式一: 以\*开头,以某些扩展名结尾 `*.do`
    
    方式二: 以/开头, 以\*结尾 `/test/*`
    
    方式三: 匹配所有地址 `/*`

    缺省: `/`

3. 如果用户的访问路径,在web.xml中由多个匹配情况下,按照以下原则优先级

    1. 绝对匹配
    2. 以斜线开头的
    3. 以\*开头的匹配路径

4. 用户的所有访问都经过servlet

    在tomcat/conf/web.xml下的配置中有一个 `/`,设置默认访问servlet

### 配置应用启动时就初始化的顺序  ###
~~~~~~
<servlet>
    <servlet-name> </servlet-name>
    <servlet-class> </servlet-class>
    <load-on-startup>2</load-on-startup>
</servlet>
~~~~~~

## servlet线程安全 ##

servlet在内存中只有一份和生命周期有关的

在servlet里面尽量不要使用实例变量, 使用局部变量
~~~~~~
// 解决方法一, 不靠谱
public synchronized void goGet() 
~~~~~~
## servlet 核心继承图 ##
![sevlet核心继承图](/img/servlet_api.png)
## ServletConfig 详解 ##
ServletConfig 是由服务器产生的
### 获取 ServletConfig ###
~~~~~~
public class MyServlet extends HttpServlet {
     @Override
     public void doGet(HttpServletRequest res, HttpServletResponse req){
         this.getServletConfig();
     }
}
~~~~~~
### 配置 ServletConfig ###
~~~~~~
<servlet>
    <servlet-name> </servlet-name>
    <servlet-class> </servlet-class>
    <init-param>
        <param-value>value</param-value>
        <param-name>name </param-name>
    </init-param>
    <init-param>
        <param-value> </param-value>
        <param-name> </param-name>
    </init-param>
</servlet>
~~~~~~
~~~~~~
ServletConfig cfg = getServletConfig();
String value = cfg.getInitParameter("name"); //没有,返回Null

Enumeration e = cfg.getInitParameterNames();
while(e.hasMoreElements()) {
    String paramName = (String) e.nextElement();
}
~~~~~~

## ServletContext ##
1. Servlet代表整个JavaWeb应用, 每个应用都会有一个唯一的 ServletContext
实例.
    
2. 生命周期: 在应用被服务器加载时由容器完成创建, 和应用一同存在
3. 获取Context的实例
~~~~~~
ServletConfig cfg = getServletConfig();
ServletContext sc = cfg.getServletContext()
// or 
ServletContext sc = getServletContext()
~~~~~~

### ServletContext应用 ###
多个Servlet可以通过ServletContext通讯,进行数据共享
~~~~~~
<web-app>
    <context-param>
        <param-name> </param-name>
        <param-value> </param-value>
    </context-param>
    <context-param>
        <param-name> </param-name>
        <param-value> </param-value>
    </context-param>
</web-app>

~~~~~~

~~~~~~
ServletContext sc = getServletContext();
// 共享变量
sc.setAttribute("p", "p1");
Object obj = sc.getAttribute("p");

sc.removeAttribute("p");
Enumeration em = sc.getAttributeNames();

//获取context-param
sc.getInitParameter("p");
Enumeration em = sc.getInitParameterNames();

//servlet转发,只是服务器上的转发,客户端不知道
RequestDispatcher rd = sc.getRequestDispatcher("/servlet/ServletDemo");
rd.forward(req, res);

~~~~~~

域对象的概念: 域表示存活范围, 表示的是应用范围, 与生命周期有关

#### ServletContext 实现下载 ####
~~~~~~
//得到文件的输入流(利用ServletContext获取文件的真实存在路径)
String realPath = getServletContext().getRealPath("/.jpg"); // 必须一斜线开头
InputStream in = new FileInputStream(realPath);
//告知客户端以下载的方式打开
res.setHeader("Content-Disposition", "attachment:filename=1.jpg");
//利用response的字节流输出内
out = response.getOutputStream();
int len = -1;
byte b[] = new byte[1024];
while((len=in.read(b))!=-1){
    out.write(b, 0, len);
}
//关流
in.close();
out.close();
~~~~~~

#### 读取Servlet下的配置文件 ####
~~~~~~
//读取各种配置文件和方式
String path = getServletContext.getRealPath("/cfg.properties");
// String path = getServletContext.getRealPath("/WEB-INF/classes/");
InputStream inStream = new FileInputStream(path);
Properties props = new Properties();
props.load()

/** 用Resourcebundle**/
ResourceBundle rb = ResourceBundle.getBundle("cfg"); // 专门读取类路径下的文件.properties
ResourceBundle rb = ResourceBundle.getBundle("cn.itcast.sc.cfg");

/** 利用ClassLoader, 不适合加载太大的文件 **/
ClassLoader cl = MyServlet.class.getClassLoader();

cl.getResourceAsStream("cn/itcast/sc/cfg.properties");
// or URL path = cl.getResource("cn/itcast/sc/cfg.properties"); URL编码出现问题
~~~~~~

