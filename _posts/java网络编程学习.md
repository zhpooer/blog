title: java网络编程学习
date: 2014-04-14 17:00:05
tags:
- java
- 网络编程
---

# java.net.InetAddress #

`Inet6Address` ipv6协议
~~~~~~
InetAddress ip = InetAddress.getLocalHost();
InetAddress ip = InetAddress.getByName();
InetAddress[] ip = InetAddress.getAllByName(); // 所有的地址

ip.getHostAddress(); // 192.168.1.11
ip.getHostName(); // www.google.com
~~~~~~

# Socket #

* Socket 就是为网络服务提供的一种机制.
* 通信两端都有Socket
* 网络通信其实就是Socket间的通信
* 数据在两个Socket间通过IO传输

## java.net.DatagramSocket ##
DatagramPacket: 用于发送或者接受的数据包
~~~~~~
DatagramSocket ds = new DatagramSocket();
String s = "xxxxxxxxxxx";
byte[] buf = str.getBytes();
// 发送, 可以用192.1.1.255发广播
DatagramPacket dp = new DatagramPacket(buf, buf.length,InetAddress.getByName("192.168.1.100"), 10000);
ds.send(dp);
ds.close();

// 接收
DatagramSocket ds = new DatagramSocket(10000); // 要明确接受端口
DatagramPacket dp = new DatagramPacket(buf, buf.length);
ds.receive(dp); // 阻塞式的
String ip = dp.getAddress.getHostAddress();
int port = dp.getPort();
new String(dp.getData(), 0, dp.getLength());
~~~~~~

## Socket 和 ServerSocket ##

~~~~~~
// 客户端
Socket s = new Socket("192.168.1.100", 10000);

OutputStream out = s.getOutputStream(); // s.getInputStream();
out.write("xxxx".getBytes());
s.shutdownOutput(); // 告诉服务端, 这边数据发送完毕,让服务端停止读取
s.close();

// 服务端
ServerSocket ss = new ServerSocket(100000);
Socket s == ss.accept(); // 获取客户端对象, 阻塞

s.getInputStream();
in.read(buf);
new String(buf, 0, len);

s.close();
ss.close();
~~~~~~

### 多线程服务端 ###
~~~~~~
while(true){
    Socket s = ss.accept();
    Thread thread = new Thread(new Handler(s));
    thread.start();
}
~~~~~~

# URL&URLConnection #
URL: 同一资源定位符, http协议

URI: 统一资源标识, mailto等 包括URL

~~~~~~
String str_url = "http://www.google.com/index.html?name.lisi";
URL url = new URL(str_url);
url.getProtocol(); // http
url.getHost();  // 192.168.1.100
url.getFile();  // /index.html?name.lisi
url.getPath(); //  /index.html
url.getQuery(); // name.lisi

url.openStream(); // url.openConnection().openStream();

URLConnection uc = url.openConnection();
uc.getHeaderField("Content-Type"); //text/html
~~~~~~
