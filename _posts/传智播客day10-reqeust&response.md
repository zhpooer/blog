title: 传智播客day10-reqeust&response
date: 2014-04-12 09:26:31
tags:
- servlet
- ServletRequest
- ServletResponse
- 传智播客
---

# HttpServletResponse #
* 给客户处发出信息
* 由容器创建
* response.getOutputStream 和 response.getWriter 如果没有手动关 系统会自动关

~~~~~~
setStatus(int stauts); // 输出响应码
addHeader();
setHeader(); // 输出响应消息头
~~~~~~

## 输出响应正文: 中文 ##
Response的字节流和字符流不能在一个Servlet中同时使用

### 字节流输出 ###
~~~~~~
String data = "上海创智播客";
ServletOutputStream out = res.getOutputStream();
byte b[] = data.getBytes(); // 本地默认, Win:GBK, Linux:UTF-8
// byte b[] = data.getBytes("utf-8");
// 方式一
res.setHeader("Content-Type", "text/html;chaset=UTF-8");

// 方式二
out.write("<meta http-equiv='Content-Type' content='text/html;charset=UTF-8'>"); // meta 没有闭符号

// 方式三
res.setContentType("text/html;charset=UTF-8");

out.write(b);

~~~~~~

### 字符流输出 ###
~~~~~~
String s = "中文";
// 方式一: 推荐
res.setCharacterEncoding("UTF-8"); // 通知程序
res.setContentType("text/html;charset=UTF-8"); // 通知客户端

// 方式二: 只用这一句, 两个作用, 通知程序和客户端 
res.setContentType("text/html;charset=UTF-8");

PrintWriter out = res.getWriter();
out.write(s); //默认编码是 iso-8859-1
~~~~~~

## 中文文件的下载 ##

~~~~~~
String path = getServletContext().getRealPath("/WEB-INF/classes/中文.jpg");
String filename = path.substring(path.lastIndexOf("//") + 1);
// 重点!!!
res.setHeader("Content-Disposition", "attachment:filename=" + URLEncoder.encode(filename, "UTF-8")+".jpg");
OutputStream out = res.getOutputStream();
InputStream in = new FileInputStream(path);
byte b[] = new byte[1024];
int len = -1;
while((len=in.read(b))!=-1){
    out.write(b, 0, len);
}
in.close();
out.close();
~~~~~~

## 画验证码图片 ##
~~~~~~
int width = 120;
int height = 25;
BufferedImage img = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
// 画边框和背景
Graphics g = img.getGraphics();
g.setColor(Color.BLUE);
g.drawRect(0, 0, width, height);
g.setColor(Color.YELLOW);
g.fillRect(1, 1, width-2, height-2);

// 画干扰线
g.setColor(Color.GRAY);
Random r = new Random();
for( int i=0;i< 15; i++)
	g.drawLine(r.nextInt(width), r.nextInt(height), r.nextInt(width), r.nextInt(height));
    
// 画验证码
g.setColor(Color.RED);
g.setFont(new Font( "宋体", Font.BOLD|Font.ITALIC , 18));
int x = 18;
for (int i=0; i<4; i++){
	g.drawString(r.nextInt(10) + "", x, 20);
	x += 22;
}

ImageIO.write(img, "jpg", res.getOutputStream());
~~~~~~

~~~~~~
// 让图片重新刷新
function (){
   var imgObj = document.getElementById("img");
   imgObj.src = "/ResponseDemo?" + new Date().getTime();
}
~~~~~~

## 定时刷新 ##
~~~~~~
res.setIntHeader("Refresh", 2); // 两秒刷新一次
res.setHeader("Refresh", "2;URL=/url"); // 两秒刷新到其他网页
~~~~~~

## 控制资源的缓存时间 ##
~~~~~~
res.setContentType("text/html;charset=UTF-8");
// 过期的时间 + 单位是毫秒
res.setDateHeader("Expires", System.currentTimeMillis() + 1*1000*60*60);//
res.getWriter().write("我爱北京天安门");

~~~~~~

# HttpServletRequest #
* 获取客户端带给服务器的信息
* 由容器创建
## 常用方法 ##
~~~~~~
req.getRequestURL();   // 协议+主机+资源地址
req.getRequestURI();   // 资源地址
req.getQueryString();    
req.getRemoteAddr();  // 来访者的IP
req.getRemotePort();  //来访者使用的端口
req.getMethod();     //请求方式
req.getProtocol();  //客户端使用的http协议版本
~~~~~~

## 获取请求消息头 ##

~~~~~~
		res.setContentType("text/html;charset=UTF-8");
		PrintWriter out = res.getWriter();

		// 获取单一的消息头
		out.println( req.getHeader("Accept-Encoding") );
		// 获取重名的消息头值
		Enumeration<String> em = req.getHeaders("Accept-Encoding");
		while(em.hasMoreElements()){
			out.println(em.nextElement());
		}
		// 获取请求消息头的内容
		Enumeration<String> en = req.getHeaderNames();
		while(en.hasMoreElements()){
			out.println( req.getHeader(en.nextElement()) );
		}
~~~~~~

## 获取客户端提交过来的请求参数 ##

~~~~~~
/* 根据之指定的名称获取请求参数 */
/*
* 没有对应的表单字段返回的是null
* 根据表单的输入域的name获取用户输入值
* 用户输入的都是字符串类型
*/
req.getParameter("username");

/* 获取重名的请求 */
String ps[] = req.getParameterValues("");

/* 获取表单中的所有请求参数 */

// 方法一:
Enumeration<String> names= req.getParameterNames();
while(names.hasMoreElements()) {
    req.getParameterValues(names.nextElement());
}

// 利用内省把数据封装到JavaBean, 前提是JavaBean中的属性名和表单中的字段名一样
// 方法二
Person p = new Person();
Enumeration<String> names= req.getParameterNames();
while(names.hasMoreElements()){
    String paramName = names.nextElement();
	String[] values = req.getParameterValues(paramName);
	// 或用BeanUtils.setProperty(p, paramName, values);
	PropertyDescriptor pd = new PropertyDescriptor(paramName, Person.class);
	Method m = pd.getWriteMethod();
	if(values!=null && values.length() == 1){
	    m.invoke(pd, values);
	}else {
	    //m.invoke(pd, (Object)values); 或
	    m.invoke(pd, new Object[] (values));
	}
}
// 方法三
Map<String, String[]> map = req.getParameterMap();
for(Entry<String, String[]> me: map.entrySet()){
    BeanUtils.setProperty(p, me.getKey(),me.getValue());
}

// 方法四
BeanUtils.populate(p, req.getParameterMap());
~~~~~~

### 各种表单输入域的获取 ###

~~~~~~
<body>
    <form action="/HelloServlet" method="post">
        <input name="id" value="436" type="hidden"/>
        <table border="1">
            <tr>
                <td>姓名: </td>
                <td> <input type="text" name="name"/> </td>
            </tr>
            <tr>
                <td>密码 </td>
                <td> <input type="password" name="password"/></td>
            </tr>
            <tr>
                <td>性别 </td>
                <td>
                    <input type="radio" name="gender" value="0"/>
                    <input type="radio" name="gender" value="1"/>
                </td>
            </tr>
            <tr>
                <td>已婚 </td>
                <td>
                    <input type="checkbox" name="married"/>
                </td>
            </tr>
            <tr>
                <td>爱好 </td>
                <td>
                    <input type="checkbox" name="hobby" value="eat"/>
                    <input type="checkbox" name="hobby" value="sleep"/>
                    <input type="checkbox" name="hobby" value="java"/>
                </td>
            </tr>
            <tr>
                <td> 省份 </td>
                <td>
                    <select name="province">
                        <option value="SH">上海 </option>
                        <option value="BJ">北京 </option>
                    </select>
                </td>
            </tr>
            <tr>
                <td> 简介 </td>
                <td>
                    <textarea rows="3" cols="23" name="description"> </textarea>
                </td>
            </tr>
            <tr>
                <td colspan="2">
                    <input type="submit"/>
                    <!-- <input type="button" value="注册" onclick="regist()"/> -->
                    <!-- <input type="image" src="images/btn.png"/> -->
                    <input type="reset"/>
                </td>
            </tr>
        </table>
    </form>
    <script type="text/javascript">
        function regist(){
            var nameInput = document.getElementById("name");
            if(nameInput.value==""){
                alert("请输入正确名称");
                return false;
            }
            document.forms[0].submit();
        }
    </script>
</body>
~~~~~~

~~~~~~
public class Student {
   private int id;
   private String name;
   private String password;
   private String gender;  // 单选和多选如果没选传回服务器是null
   private boolean married;
   private String[] hobby;
   private String province;
   private String descriptioin;
}
public void doGet(res, req){
    Student s = new Student();
    BeanUtils.populate(s, request.getParameterMap());
}
~~~~~~

input type=text 这样的输入域, 用户没有输入值, 服务器得到的是空字符串

type=radio/checkbox 用户没有输入值, 服务器得到的是null


### 中文请求的转码 ###
浏览器以当前编码发送中文字符串到服务器,服务器默认会以 ISO8859-1 编码来解码


如果客户端按照get方式发送请求, 我们只能手工进行指定字符集编码
~~~~~~
// 服务器以 IOS 解码, 所以重新取二进制编码
String str = req.getParameter("username").getBytes("ISO8859-1");
// 再重新编码
String uname = new String(str, "utf-8");
~~~~~~

如果客户端按照post方式发送请求, 设置请求对象的字符集
~~~~~~
// 设置查询码表
req.setCharacterEncoding("utf-8");
req.getParameter("username");
~~~~~~

## request域对象 ##
~~~~~~
// Servlet1
request.setAttribute("p", "pp");

// Servlet2, Servlet 必须进行请求转发
request.getAttribute("p");
~~~~~~

### 请求重定向 ###
1. 地址栏会变
2. 发两次请求
3. 跨服务器
4. 会发 302, Location

~~~~~~
// 方式一
res.setStatus(302);
res.setHeader("Location", "www.sina.com");
// 方式二
res.sendRedirect("www.sina.com");
~~~~~~

### 转发 ###
1. 服务器的行为
2. 发一次请求
3. 地址栏不变
~~~~~~
req.setCharacterEncoding("utf-8");
String userName = req.getParameter("username");
req.setAttribute("", ""); // 传递参数
// 获取RequestDispatcher, 用于实现请求转发
// 将数据进行向下传递
// 方式一: 转发的路径必须以斜线开头, "/"
getServletContext().getRequestDispatcher("/servleturl").forward(req, res); // 内部转发

// 方式二:  转发的路径可以以斜线开头, 可以不以斜线开头, 表示相对路径
req.getRequestDispatcher("").forward(req, res);
~~~~~~

## 请求包含 ##
RequestDispatcher: 请求分发器
1. 实现转发, 源 -> 目标
2. 实现包含, 目标 -> 源
~~~~~~
/* 运行结果: "我喜欢你"*/
// Servlet1
response.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
out.write("我喜欢")
RequestDispatcher rd = request.getRequestDispatcher("Servlet2");
rd.include(req.res);

// Servlet2
response.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
out.write("你")
~~~~~~

## 各种URL地址的写法 ##
* 相对路径 
* 绝对路径(开发中推荐), 拥抱变化, `/`

    给服务器用的地址 斜线就代表当前应用. 给浏览器用的地址, 必须加上应用名称

重定向: responst.sendRedirect()

转发: request.getRequestDispatcher()

超连接: `<a ref=""/>`

表单: `<form action=""/>`
