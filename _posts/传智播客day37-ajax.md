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
    resp.setContentType("text/html;charset=utf-8");
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
        // 初始化xhr对象
        xhr.open("GET", "/ServletDemo?username=xxx");
        // 建立与服务器的连接, 发送数据
        
        // 如果是GET方法, 不会发送任何数据, 传递null即可
        xhr.send(null); 
        
        // 如果是Post方法, 如果没有数据作为请求的一部分发送, 也是用null
        // 设置请求消息头, 告知服务器, 发送的正文数据类型
        xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
        xhr.send("name=xxx&age=xxx");

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
