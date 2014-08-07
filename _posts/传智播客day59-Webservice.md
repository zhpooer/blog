title: 传智播客day59-Webservice
date: 2014-07-18 09:08:54
tags:
- 传智播客
- webservice
---

# Socket #

模拟天气查询 API

服务端
~~~~~~
// 启动端口监听, 端口号建议在1万以后
ServerSocket serverSocket = new ServerSocket(10001);

while(){
    Socket socket = serverSocket.accept();
    // 获取输入流
    DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());
    
    // 获取输出流
    DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());
    
    // 接收客户端请求的数据
    String cityName = dataInputStream.readUTF();
    
    dataOutputStream.write("天气晴朗");
    
    dataOutputStream.close();
    dataInputStream.close();
    // socket不用释放, 客户端socket关闭, 服务端自动关闭
}
~~~~~~

客户端
~~~~~~
Socket socket = new Socket("localhost", 10001);

// 获取输入流
DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());

// 获取输出流
DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());

dataOutputStream.writeUTF("上海");

String result = dataInputStream.readUTF();
dataOutputStream.close();
dataInputStream.close();
socket.close();
~~~~~~

# Webservice #

JAX-WS 的全称为 Java API for XML-Based Webservices

Webservice底层基于socket通信，采用soap协议进行通信，
webservice不需专门针对数据流的发送和接收进行处理，
是针对web开发远程调用的一种技术。

~~~~~~
// 服务端

// 编写SEI(Service Endpoint Interface)
// SEI在webservice中称为port
// 在java中称为接口，接口类型叫portType
public interface WeatherInterface {
    //天气查询
    public String queryWeather(String cityName);
}

@WebService
public class WeatherInterfaceImpl implements WeatherInterface {
    @Override
    public String queryWeather(String cityName) {
        System.out.println("from client.."+cityName);
        String result = "晴朗";
        System.out.println("to client..."+result);
        return result;
    }
    // 访问 http://192.168.1.100:1234/weather?wsdl 可以查询 webservice 说明
    public static void main(String[] args) {
        //发送webservice服务
        Endpoint.publish("http://192.168.1.100:1234/weather",
            new WeatherInterfaceImpl());
    }
}
~~~~~~

分析服务说明, 网络描述语言, 基于 xml
* 从下往上

## 客户端 ##

~~~~~~
// 根据ws描述文件, 自动生成 java 客户端代码
// wsimport -s . http://192.168.1.100:1234/weather?wsdl

public static void main(){
    // 调用webservice服务
    // 创建服务视图service
    WeatherServerService weatherServerService = new WeatherServerService();

    // 服务视图bingding获取porttype
    WeatherServer weatherServer = weatherServerService.getWeatherServerPort();
    // 通过 protType 调用 werbService方法
    String result = weatherServer.queryWeather("郑州");
}
~~~~~~

# 什么是 webservice #

* Web service 即web服务，它是一种跨编程语言和跨操作系统平台的远程调用技术即跨平台远程调用技术。
* 采用标准SOAP(Simple Object Access Protocol) 协议传输，soap属w3c标准。
* 基于http传输xml，即soap=http+xml
* 采用wsdl作为描述语言即webservice使用说明书，wsdl属w3c标准。
* xml和XSD(XML Schema Datatypes)是webservice的跨平台的基础，
XML主要的优点在于它既与平台无关，又与厂商无关。
XML是由万维网协会(W3C)创建，W3C制定的XSD定义了一套标准的数据类型，
数据类型用xml进行描述。

## 三要素 ##

### soap ###

SOAP即简单对象访问协议(Simple Object Access Protocal)
是一种简单的基于 XML 的协议，
它使应用程序通过 HTTP 来交换信息，简单理解为soap=http+xml。

Soap协议版本主要使用soap1.1、soap1.2

SOAP可以运行在任何其他应用协议上。例如，SMTP、tr069等。

### wsdl ###

WSDL 是基于 XML 的用于描述Web Service及其函数、参数和返回值。
通俗理解Wsdl是webservice的使用说明书。

### UDDI ###

UUDI 是一种目录服务，通过它，
企业可注册并搜索 Web services。
企业将自己提供的Web Service注册在UDDI，
也可以使用别的企业在UDDI注册的web service服务，从而达到资源共享。


## 应用场景 ##

* 应用程序集成

    分布式程序之间进行集成使用webservice直接调用服务层方法，
    不仅缩短了开发周期，还减少了代码复杂度，并能够增强应用程序的可维护性。
* 软件重用        

    将一个软件的功能以webservice方式暴露出来，达到软件重用。
    例如上边分析的天气预报，
    将天气查询功能以webservice接口方式暴露出来非常容易集成在其它系统中；
    再比如一个第三方物流系统将快递查询、快递登记暴露出来，
    从而集成在电子商务系统中。
* 跨防火墙通信

    因为webservice和网页程序都是运行在web容器且用相同的端口和协议。

不使用 webservice
* 单个程序间通信, 当应用之间需要通信且无需将接口暴露给第三方系统时
完全没有必要使用webservice技术，
这时企业自定义一种简单的接口协议即可，简单高效
* 同构程序间通信, 同构程序是指采用相同的编程语言的程序之间通信，
比如java远程调用RMi技术就可以非常高效的实现远程调用，使用简单方便，
必需保证两边应用都是java编写才可用使

优点：
1. 采用xml支持跨平台远程调用。
2. 基于http的soap协议，可跨越防火墙。
3. 支持面向对象开发。
4. 有利于软件和数据重用，实现松耦合。

缺点：
1. 由于soap是基于xml传输，本身使用xml传输会传输一些无关的东西从而效率不高，随着soap协议的完善，
soap协议增加了许多内容，这样就导致了使用soap协议去完成简单的数据传输的效率更加不高。
2. webservice作为web跨平台访问的标准技术，很多公司都限定要求使用webservice，
其实对于简单的接口如果直接用http传输自定义数据内容比webservice开发更快捷，
例如第三方支付公司的支持接口。

# wsdl #

WSDL 指网络服务描述语言(Web Services Description Language)。

WSDL是一种使用 XML 编写的文档。这种文档可描述某个 Web service。它可规定服务的位置，以及此服务提供的操作（或方法）。

WSDL 是一种 XML 文档

WSDL 用于描述网络服务

WSDL 也可用于定位网络服务

## wsdl 说明书结构 ##
* `<service>` 整个webservice的服务视图，它包括了所有的服务端点
* `<binding>` 为每个端口定义消息格式和协议细节
* `<portType>` 描述 web service可被执行的操作，以及相关的消息，
通过binding指向portType
* `<message>` 定义一个操作（方法）的数据参数(可有多个参数)
* `<types>` 定义 web service 使用的全部数据类型

# 使用javax.xml.ws.Service 进行客户端编程 #

~~~~~~
// 创建 url, 指定webservice地址
// 参数是wsdl的地址
Url url = new Url("http://localhost:1234/weather?wsdl");

// 创建Qname，指定命名空间和视图名称
// 第一个参数是 webservice的namespace
// 第二个参数是 webService的服务视图
QName qName = new QName("http://server.jaxws.ws.itcast.cn/", "WeatherServerService");

// 创建服务视图 service
Servic service = Service.create(url, qName);

// 通过服务视图 获取 PortType
// 从服务视图中得到服务端点即服务接口
WeatherWebServiceSoap soap = service.getPort(WeatherWebServiceSoap.class);
// 通过服务端点调用服务方法
String result = soap.getWeatherByCityName("郑州");

~~~~~~

# SOAP #

SOAP 是一种网络通信协议

SOAP即SimpleObjectAccessProtocol简易对象访问协议

SOAP 用于跨平台应用程序之间的通信

SOAP 被设计用来通过因特网(http)进行通信

SOAP ＝ HTTP+XML，其实就是通过HTTP发xml数据

SOAP 很简单并可扩展支持面向对象

SOAP 允许您跨越防火墙

SOAP 将被作为 W3C 标准来发展

## 使用工具监视http协议和Soap协议 ##

使用firefox监视http的协议格式

     http： 请求方式：get或post
      get：
     请求： ContentType =text/html;charset=utf-8
     响应 ： ContentType = text/html;charset=utf-8
      post：
     请求： ContentType = application/x-www-form-urlencoded
     响应 ： ContentType = text/html;charset=utf-8
     soap协议无法用浏览器监视，因为webservice没有通过浏览器
    
使用Myeclipse  的WebService Explorer监视soap协议体

使用TCP/IP Monitor监视soap协议体和协议头


## 组成 ##

soap1.1 和 soap1.2 区别
~~~~~~
soap1.1:
Post
Content-type:text/xml;charset='utf-8'

soap1.2:
Post
Content-type:application/soap+xml;
~~~~~~

请求和响应都是如下格式
~~~~~~
<?xml version="1.0"?>
<!-- 此元素将整个 XML 文档标识为一条 SOAP 消息 -->
<soap:Envelope
      xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
      soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
    <!-- 可选的 Header 元素，包含头部信息 -->
    <soap:Header>
    </soap:Header>
    <!-- 包含所有的调用和响应信息 -->
    <soap:Body>
        <!-- 供有关在处理此消息所发生错误的信息 -->
        <soap:Fault>
        </soap:Fault> 
    </soap:Body>
</soap:Envelope>
~~~~~~

## 注解 ##

~~~~~~
@Webservice(targetNamespace="http://webservice.itcast.cn/"
    serviceName="WeatherService",
    portName="WeatherServicePort")
public class WeatherServer implements WeatherInterface{
    @WebMethod(operationName="queryWeatherByCityName")
    public @WebResult(name="weatherResult") String queryWeather(@WebParam(name="cityName") String city){}

    // 不暴露接口
    @WebMethod(exclude=true)
    public String notpublic(){}
    
}
~~~~~~


# 使用复杂类型 #

~~~~~~
public interface WeatherInterface {
    // 返回未来三天的天气
    public List<WeatherModel> queryWeatherAll();
}

public class WeatherModel {
     private String cityName;
     private Date date;
     private Integer temperator_max;
     private Interger temperator_min;
}

@Override
public List<WeatherModel> queryWeatherAll(){
     WeatherModel m1 = new WeatherModel();
     WeatherModel m2 = new WeatherModel();
     WeatherModel m3 = new WeatherModel();
     return new ArrayList({m1, m2, m3});
}
~~~~~~

# 发布为web工程 #


* 下载`jaxws-ri-2.2.8`的扩展包
* 创建web工程
* 将扩展包中的jar拷贝至web工程下
* 编写服务端代码，编写方法与之前我们学习的 jax-ws 方法一致
~~~~~~
@WebService
@BindingType(value="http://www.w3.org/2003/05/soap/bindings/HTTP/") // 用于生成 soap1.2
public class WeatherService implements IWeatherService {
}
~~~~~~
* 根据类生成 wsdl 文件
~~~~~~
-- -cp classpath 目录
-- -r 输出wsdl路径
-- -wsdl:Xsoap1.2 
wsgen -wsdl:Xsoap1.1 -cp ./WEB-INF/classes -r ../wsdl server.WeatherServer
~~~~~~
* 在 `WEB-INFO` 下创建 `sun-jaxws.xml`
~~~~~~
<endpoints xmlns='http://java.sun.com/xml/ns/jax-ws/ri/runtime'
           version='2.0'>
    <endpoint name='WeatherServer'
            implementation='cn.itcast.ws.server.WeatherServer'
            wsdl='WEB-INF/wsdl/WeatherServerService.wsdl'
            binding="http://www.w3.org/2003/05/soap/bindings/HTTP/" 
            url-pattern='/weather'/>
    <!-- 可以在这里添加多个 endpoint  -->
</endpoints>

<!-- soap1.1 -->
<endpoints xmlns='http://java.sun.com/xml/ns/jax-ws/ri/runtime'
    version='2.0'>
    <endpoint name='ServerJws' implementation='cn.itcast.weather.server.ServerJws'
        wsdl='WEB-INF/wsdl/WeatherServerService.wsdl'
        url-pattern='/weather' />
</endpoints>
~~~~~~
* 配置web.xml
~~~~~~
<listener>
    <listener-class>
    com.sun.xml.ws.transport.http.servlet.WSServletContextListener
    </listener-class>
</listener>  
<servlet>
    <servlet-name>weather</servlet-name>
    <servlet-class>
    com.sun.xml.ws.transport.http.servlet.WSServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<!-- /weather必须和sun-jaxws.xml中的url-pattern="/weather"相同 -->
<servlet-mapping>
    <servlet-name>weather</servlet-name>
    <url-pattern>/weather</url-pattern>
</servlet-mapping>
~~~~~~
* 访问tomcat下的web工程即可(http://ip:端口/工程目录/weather)

注意, 如果要发布 soap1.2协议, 必须使用方法 `wsdl:Xsoap1.2`生成 wsdl,
另外, 服务类添加 `@BindingType`
