title: 传智播客day37-AJAX
date: 2014-05-23 19:32:46
tags:
- 传智播客
- AJAX
---

# 同步和异步 #

同步: 提交请求 -> 等待服务器处理 -> 处理完毕返回 -> 这期间客户端不能干任何事

异步: 请求通过事件出发 -> 服务器处理(浏览器可以做其他事) -> 处理完毕返回

# AJAX #

AJAX: Asynchronous Javascript and XML, 允许浏览器和服务器通信而无须刷新
当前页面的技术, 与服务器无关

核心是JavaScript对象: XmlHttpRequest

# 与服务器通信 #
服务端
~~~~~~
// ServletDemo
public void doGet(req, resp) {
    resp.setContentType("text/xml;charset=utf-8");
    PrintWriter out = resp.getWriter();
    
    // 返回 xml
    SAXReader reader = new SAXReader();
    Document document = reader.read(path);
    String xml = document.asXML();
    out.write(xml);
}
~~~~~~
客户端
~~~~~~
function createXmlHttpRequest(){
    var xmlHttp;
    try{
        xmlHttp = new XMLHttpRequest();
    } catch(e) {
        try {
            xmlHttp = new ActiveXObject("Msxml2.XMLHTTP");
        } catch (e){
            xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
        } catch(e){}
    }
    return xmlHttp;
}
window.onload = function(){ // 当页面被全部记载完毕后再执行
    document.getElementById("b1").onclick = function(){
        // 创建 XMLHttpRequest 对象
        var xhr = createXmlHttpRequest();
        
        // xhr的readyState 改变都会触发 onreadystatechange 事件
        // 0 (未初始化) 对象已建立，但是尚未初始化（尚未调用open方法） 
        // 1 (初始化) 对象已建立，尚未调用send方法 
        // 2 (发送数据) send方法已调用，但是当前的状态及http头未知 
        // 3 (数据传送中) 已接收部分数据，因为响应及http头不全，
        // 4 (完成) 数据接收完毕,此时可以通过通过responseBody和responseText获取完整的回应数据 
        xhr.onreadystatechange = function() {
            if(xhr.readyState == 4) {
                // 数据正确返回
                if(xhr.status==200 || xhr.status ==304){
                    // 如果返回是文本数据
                    var data = xhr.responseText;
                    // 如果返回是 xml
                    // 如果文档类型不正确, responseXML 的值将会是空的
                    var dom = xhr.responseXML; // 返回是 dom 对象
                    document.getElementById("d1").innerHTML = data;
                }
            }
        }
        // 初始化xhr对象 ,// 建立与服务器的连接
        xhr.open("GET", "/ServletDemo?username=xxx");
        
        // 发送数据
        // 如果是GET方法, 不会发送任何数据, 传递null即可
        xhr.send(null); 
        
        // 如果是Post方法, 如果没有数据作为请求的一部分发送, 也是用null
        // 设置请求消息头, 告知服务器, 发送的正文数据类型
        xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
        xhr.send("name=xxx&age=xxx");
        // 或 xhr.send({name:"itcast"});

    }
}
~~~~~~

# XMLHttpRequest 对象方法 #
| 方法           |             描述 |
|----------------|
| abort()                 |   停止当前请求                        |
| getAllResponseHeaders() | 把http请求的所有响应首部作为键/值对返回|
| getResponseHeader("headerLabel") | 返回指定首部的串值 |
| open("method", "url", "isAsync")   | 建立对服务器的调用, method可以为 GET POST |
| send(content)           | 向服务器发送请求 |
| setRequestHeader("label", "value") | 把指定首部设置为所提供的值, 在设置任何首部之前必须先调用open() |

# XMLHttpRequest 对象属性 #
| 属性        |       描述 |
|--------------------|
| onreadystatechange | 状态改变的事件触发器(回调函数), 服务器触发 |
| readyState         | 对象状态(int), 0=未初始化, 1=读取中, 2=已读取, 3=交互中, 4=完成 |
| responseText       | 服务器进程返回数据的文本版本 |
| responseXML        | 服务器返回数据兼容DOM的XML的文档对象 |
| status             | 返回状态码, 404, 200 |
| statusText         | 状态文本信息 |

# 案例校验用户名 #

~~~~~~
<form>
    <input type="text" id="username" name="username" onblur="checkUsername()"/>
    <span id="checkResult"></span>
    <input type="text" name="email"/>
    <input type="submit"/>
</form>
<script type="text/javascript">
function checkUsername(){
     var nameInputElm = document.getElementById("username");
     var xmlHttpReq = createXmlHttpRequest()
     xmlHttpReq.onreadystatechange = function(){
         if(xmlHttpReq.readyState == 4) {
             if(xmlHttpReq.status == 200  || xhr.status == 304) {
                 var result = xmlHttpReq.responseText;
                 document.getElementById("checkResult").innerHTML = result;
             }
         }
     };
     xmlHttpReq.open("POST", "/checkUsername");
     xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
     xmlHttpReq.send({username: nameInputElm.value})
}
</script>
~~~~~~

~~~~~~
// checkUsernameServlet
List names = Array.asList(new String[]{"abcd", "efg", "qwe"});
public void doPost(request, response) {
    response.setContentType("text/html;charset=utf-8");
    request.setCharactorEncoding("utf-8");
    String username = request.getParameter("username");
    
    if(names.contains(username)) {
        response.getWriter().print("用户名已经存在");
    } else {
        response.getWriter().print("用户名不存在存在");
    }
}
~~~~~~

# JSON #

JavaScript Object Notation, 比 xml 更轻巧, 是 JavaScript 的原生格式
~~~~~~
public void doGet(req, resp) {
    resp.setContentType("text/json;charset=utf-8");
    PrintWriter out = resp.getWriter();
    String str = "{name:'山东'}";
    out.print(str);
}
~~~~~~

~~~~~~
var xhr = createXmlHttpRequest();
xhr.onreadystatechange = function() {
    if(xhr.status==200 || xhr.status ) {
       // 返回值是 json 的字符串形式
       var data = xhr.responseText;
       // 把普通的 json 文本转换成 json 数据
       var json = eval("("+ data + ")");
    }
}
~~~~~~

## JSONlib 使用 ##

~~~~~~
// 省, 邮政编码
Province p = new Provice("山东省", 250000);
JSONObject jsonObj = JSONObject.fromObject(p);
jsonObj.toString();   // {"name":"山东省", "zipcode":"250000"}

// 输出数组
Province p1 = new Provice("山东省", 250000);
Province p2 = new Provice("浙江", 320000);
List ps = new ArrayList<Province>();
ps.add(p1);
ps.add(p2);
JSONArray jsonArr = JSONArray.fromObject(p2);
jsonArr.toString() // [{name:**, zipcode:**},{name:**, zipcode:**}]

// 过滤输出
JsonConfig cfg = new JsonConfig();  
cfg.setExcludes(new String[]{"zipcode"}); // 不包含的字段列表
JSONArray jsonArr = JSONArray.fromObject(p2, cfg);
jsonArr.toString() // [{name:**},{name:**}]

~~~~~~

## FlexJson ##
flexjson 是一个轻量级的java类库, 序列化 Json

* 序列化对象
~~~~~~
Product p = new Product(1, "冰箱", 200);
p.setOrders(orders);
JSONSerializer jsonSerializer = new JSONSerializer();
// jsonStr:  {class:"zhpooer.Product" , id: 1, name: "冰箱", price: "200"}
// 不会序列化集合
String jsonStr = jsonSerializer.serialize(p);
// 序列化文档
String jsonStr =jsonSerializer.include("orders").serialize(p);
// 排除序列化属性
String jsonStr =jsonSerializer.exclude("name").serialize(p);
// 使用注解排除属性 @JSON(include=false)
~~~~~~

# 服务器返回xml #
Xml格式, 不依赖任何语言,, 跨平台第三方通用数据格式

## xstream ##
实现 xml 和 java 之间相互转换

XMl解析方式: DOM, SAX, Stax(pull 解析采用Stax)

导入jar包: `xstream,jar` `xpp3.jar`
~~~~~~
// 将对象转换为 xml
User user = new User();
user.setId(1);
user.setName("");
user.setGender("男");
XStream xStream = new XStream();
// 给User取别名
xStream.alias("user", User.class);
xStream.alias("users", List.class);
String xml = xStream.toXML();

// 解析XML
XStream xStream = new XStream();
xStream.alias("user", User.class);
xStream.alias("users", List.class);
xStream.fromXML(new FileInputStream("user.xml"));
~~~~~~

## 使用XML 注解 ##
~~~~~~
// 起别名
@XStreamAlias("User")
public class User{
    @XStreamAsAttribute
    private int id;
    // 不使用
    @XStreamOmitField
    private String gender;
}
XStream xStream = new XStream();
// 是注解生效
xStream.autodetectAnnotations(true);
~~~~~~


# Tip #
~~~~~~
<!-- 不处理链接 -->
<a href="javascript:void(0)">商品数据</a>
~~~~~~
