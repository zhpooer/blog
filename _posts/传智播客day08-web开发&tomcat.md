title: 传智播客day08-web开发&tomcat
date: 2014-04-09 08:55:30
tags:
- 传智播客
- java
- web开发
- tomcat
- http协议
---
# Tips #
~~~~~~
ByteArrayOutputStream baos = new ByteArrayOutputStream();//带缓冲的输出流
GZIPOutputStream gout = new GZIPOutputStream(baos);
gout.write(b);
baos.toByteArray();
~~~~~~

# web开发 #
web资源: 静态web资源, 动态web资源

静态资源:  
1. *.html
2. *.css
3. *.js
2. 图片

动态资源:  
1. servlet
2. *.jsp

## 常见服务器 ##
WebLogic: Oracle公司产品, 支持JavaEE规范, 收费

WebShpereAS (Application Server): IBM, 支持JavaEE规范

JbossAS: redhat公司产品, 支持JavaEE规范

Tomcat: 轻量级, *只实现了 Servlet/JSP 规范*

~~~~~~
ServerSocket server = new ServerSocket(8888);
Socket client = server.accept();
OutputStream out = client.getOutputStream();
InputStream in = new FileInputStream("");

int len = -1;
byte b[] = new byte[1024];
while((len = in.read(b)) != -1) {
    out.write(b, 0, len);
}
out.close();
server.close();
in.close();
~~~~~~

## JavaEE 规范 ##
是很多技术的总称, 有很多接口或抽象类组成, 如:  
Servlet/JSP, JDBC, JNDI, JTA, JPA, JMS ...

我们按照规范的要求来开发web应用, 然后部署到服务器上运行

# Tomcat服务器 #
|Tomcat | servlet | j2ee版本 | jdk版本 |
|--------------------------------|
| 8.0   | 3.2     | *    |1.7  |
| 7.0   | 3.0     | 6.0  |1.6  |
| 6.0   | 2.5     | 5.0  |1.5  |

启动成功后,访问 http://localhost:8080

修改端口: server.xml

webapps目录存放开发的app应用

## web应用的目录结构 ##
*WEB-INF*: 放在web应用的根目录, 用户无法直接访问

classes: 在WEB-INFO下, 放class字节码文件

lib: 在WEB-INFO下, 放应用需要的jar包

web.xml: 在WEB-INFO下, 应用的配置文件

### 存放类的地方 ###
1. 本应用的classes目录
2. 本应用的lib中的jar包
3. Tomcat的lib目录

按照123依次搜索

### 发布 ###
1. 直接把应用目录放到 webapps 下
2. 打包成war包, 放到webapps下 `jar -cvf`

## tomcat的组成结构 ##

### server.xml ###
~~~~~~
<Server>
    <Service>
        <Connector>
        </Connector>
        <Engine>
            <Host>
            </Host>
        </Engine>
    </Service>
</Server>
~~~~~~
用户的访问都是通过Tomcat连接器连接过来的,

tomcat应用管理这很多主机(Host)

每个主机管理着很多应用(Context)

### 配置虚拟目录(Context) ###
把任意目录配成Tomcat管理的应用

方式一: 弊端，需要重启
~~~~~~
<Host>
    <!-- path: 用户访问的应用目录了, 一般以/ 开头 -->
    <!-- doBase: 应用的真是存放目录 -->
    <!-- localhost/shit/1.html -->
    <Context path="/shit" docBase="E://">
    </Context>
</Host>
~~~~~~

方式二: 推荐，优点不用重启
1. 新建SH.xml
~~~~~~
<Context docBase="fullpath">
</Context>
~~~~~~
2. 放在 Tomcat_home/conf/localhost/SH.xml
3. 访问 http://localhost/SH/*.html

** 实际开发中直接把文件拷贝到webapp目录 **

### 配置虚拟主机主机 ###
虚拟主机：不同域名下的访问目录
* 在Engine添加以下内容
~~~~~~
<Host name="www.google.com" appBase="dir"
      unpackWARS="true" autoDeploy="true">
    <Context> </Context>
</Host>
~~~~~~


### 配置默认应用, 默认主页, 默认端口 ###
`http://localhost:8080/home/index.html` => `http://localhost`
1. 修改端口

    server.xml 8080 -> 80
    
2. 修改默认应用

      1. 修改应用名称为ROOT
      
      2. touch tomcat/conf/Catalina/localhost/ROOT.xml
~~~~~~
<?xml version="1.0"?>
<Context docBase="Myapp"/>
~~~~~~
3. 修改应用默认主页
修改应用下的web.xml
~~~~~~
<welcome-file-list>
    <welcome-file>1.html</welcome-file>
</welcome-file-list>
~~~~~~

# http协议 #
1. 作用: 描述客户端和服务端的数据的传递的协议
2. 全称: 超文本传输协议.
3. 版本: 1.0, 1.1

    1.0: 连接 请求 响应 关闭, 无状态
    
    1.1: 连接 (请求 响应)*n 关闭

## html协议的组成 ##

### 请求部分 ###
* 请求消息行,位于请求部分第一行, 必须小于1KB, 如 `GET /  HTTP/1.1 `

    默认的请求方式是GET,常用的为GET POST HEAD OPTIONS
    
    URL: 协议+ip:端口+资源地址
    
    URI: 资源地址

* 请求消息头, 从第二行开始至第一个空行结束

    作用: 向服务器端传递一些附加信息
    
    形式: Header-Name: headervalue1, headerValue2

    Accept: 告知服务器, 浏览器能接受的MIME类型, 在`conf/web.xml`里面查看

    Accept-Charset: 告知服务器, 客户端能接受的字符集

    Accept-Encoding: 告知服务器, 客户端能接受的压缩编码

    Accept-Language: 告知服务器, 客户端能支持的语言

    Referer: 告知服务器, 当前页面是由哪一个页面转过来的.
    若用户是直接访问,就没有.

    Content-Type: 告知服务器,客户端提交的正文的MIME类型.默认是
    application/x-formdata-urlencode. 可以通过表单的form的enctype属性指定.
    可选值multipart/form-data(文件上传)

    If-Modified-Since: 告知服务器, 本地所缓存的资源的最后的修改时间

    User-Agent: 浏览器类型

    Content-Length: 请求正文的长度

    Connection: 需要持久连接

    Cookie: 向服务器发送cookie数据

    Date: 发送请求的时间
    
* 请求体, 从第一个空行到结束

### 响应部分 ###
* 响应码, 如 `HTTP/1.1 200 OK`

    状态码,说明了本次请求的结果状态
    
|   状态码 |  说明      |
|-------------------------|
|    2xx| 成功接受请求     |
|    200| 成功             |
|    304| 未修改           |
|    302| 307: 临时重定向  |
|    404| 访问的资源不存在 |
|    500| 服务器内部错误   |

* 响应消息头,从第二行开始至第一个空行结束

    Location: 返回一个地址, 和302一起使用,实现请求重定向

    Server: 显示服务器类型

    Content-Encoding: 告知客户端,服务端的压缩格式

    Content-Length: 告知客户端,响应正文的内容长度

    Content-Type: 告知客户端,响应正文的MIME类型
    `Content-Type: text/html;charset=utf-8`

    Last-Modified: 文件的最后修改时间, 节省带宽

    Refresh: 浏览器定时刷新时间

    Content-Disposition: 告知客户端, 以下载方式打开
    `Content-Disposition:attachment:filename=1.jpg`

    Expire: -1  
    Cache-Control:no-cache  (1.1)  
    Progma:no-cache (1.0)  

* 响应正文, 从第一个空行到结束  
