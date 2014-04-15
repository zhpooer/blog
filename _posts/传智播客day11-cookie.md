title: 传智播客day11-cookie
date: 2014-04-13 09:23:18
tags:
- 传智播客
- cookie
---

# 会话概述 #

在一次会话中, 浏览器可以发出多次请求和收到多次服务器响应

服务器为客户保存操作产生的数据, 比如购物网站的购物车

解决方案: Cookie(客户端技术), Session(服务端技术)

> 服务器发送 Set-Cookie: name=any
> 客户端访问 cookie: name=any


~~~~~~
Session s = request.getSession();
~~~~~~

## Cookie技术 ##

![cookiez组成](/img/cookie-info.png)

### Cookie 中的属性 ###
name: 名称

*value: 取值(不能是中文)*

*path: 可选, 路径, 默认值写Cookie那个程序的访问路径, 不包括文件名*

*maxAge: 可选. 最大存活时间, 默认是一次会话, 存在于浏览器进程内存中, 单位是秒*

domain: 域名 可选,默认值是写Cookie的网站

comment: 备注 可选

version: 版本 可选

###  向客户端输出一个Cookie ###
> `HttpServletResponse.addCookie(Cookie c)`

客户端浏览器针对一个网站, 最多能存20个Cookie. 总共能存300个.
每个Cookie的大小不能超过4KB.(资源稀少)

### 服务端如何获取Cookie ###
> `HttpServletRequest.getCookies()`

~~~~~~
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
out.write("您最后来访的时间是: <br/>");
Cookie[] cookies = req.getCookies(); // null
for(int i=0; cookies!=null&&i<cookies.length;i++){// 一定要判断是否为空
     Cookie c = cookies[i];
     if("lastAccessTime".equals(c.getName())){
         String value = c.getValue();
         long time = Long.parseLong(value);
         Date d = new Date(time);
         DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
         String stime = df.format(d);
         out.write(stime);
         break;
     }
}

Cookie c = new Cookie("lastAccessTime", System.currentTimeMillis()+"");
c.setMaxAge( 24*60*60 ); //单位是秒
res.addCookie(c);

~~~~~~

### 把指定的cookie删掉 ###
修改Cookie的生命周期,
~~~~~~
Cookie c = new Cookie("lastAccessTime", "");
c.setMaxAge(0);
response.addCookie(c);  //覆盖掉原有的Cookie
~~~~~~

### 如何确定一个Cookie ###
域名+路径+Cookie的名称

带不带cookie给服务器, 浏览器说了算. 根据你当前访问的路径来判断.
如果当前访问路径.startWith(已存Cookie路径) 为true, 就带给成立

~~~~~~
c.setPath(req.getContextPath()); // 所有应用下的都能访问到该Cookie
~~~~~~

#### 案例一: 录入用户信息 ####
~~~~~~
// 产生一个登陆页面
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();

String username = "";
String checked = ""

Cookie cs[] = req.getCookies();
for(int i=0; cs !=null && i < cs.length; i++) {
   Cookie c = cs[i];
   if("loginInfo".equals(c.getName())){
       username = c.getValue();
       checked = "checked"
       break;
   }
}
out.write("<form action='/HelloServlet2' method='post'><br/>");
out.write("username: <input type='text' name='username' value='" +username +  "'/> <br/>");
out.write("password: <input type='password' name='password' value=''/> <br/>");
out.write("remenber: <input type='checkbox' name='remember' checked='" + checked + "' /> <br/>");
out.write("<input type=submit' name='password' value=''/> <br/>");
out.write("</form>");

~~~~~~

~~~~~~
// 负责完成登陆验证, 同时以Cookie的形式把用户名记住, 前提不能是中文
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
// 获取用户名和密码
String username = request.getParameter("username");
String password = request.getParameter("password");
// 获取用户是否选择了记住用户名
String remember = request.getParameter("remember"); // 没有选着就是null

// 判断用户名和密码是否正确: 用户名翻转之后就是密码.比如用户名abc, 那么密码是cba
if(!username.equals(new StringBuffer(password).reverse().toString())){
    out.write("错误的用户名或密码, 2秒后转向登陆页面");
    res.retHeader("Refresh", "2;URL=" + req.getContextPath() + "/HelloServlet");
    return;
}
Cookie c = new Cookie("loginInfo", username)
c.setPath(req.getContextPath());
if(remember!=null){
// 如果用户选择记住用户名, 写cookie
    c.setMaxAge(Integer.MAX_VALUE);
} else {
// 如果没有用户选择记住用户名, 删除cookie
    c.setMaxAge(0);
}
cookie.addCookie(c);
~~~~~~

#### 案例二: 购书 ####
~~~~~~
public class Book{
   private String id;
   private String name;
   private int price;
   private String author;
   private String description;
}
~~~~~~


~~~~~~
public class BookDB{
   private static Map<String, Book> books = new HasMap<String, Book>();
   static {
        books.put("1", new Book("1", "葵花宝典", 10f, "xxx", "aaa"));
        books.put("2", new Book("2", "葵花宝典1", 10f, "xxx", "aaa"));
        books.put("3", new Book("3", "葵花宝典2", 10f, "xxx", "aaa"));
   }
   public static Map<String, Book> findAllBooks(){
       return books;
   }

   public static findBookById(String id) {
       books.get(id);
   }
}
~~~~~~


~~~~~~
//1 显示所有商品
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
out.print("<h1>本站有以下好书</h1>");
Map<String, Book> books = BookDB.findAllBooks();
for (Map.Entry<String, Book> me: books.entrySet()){
    out.write(me.getValue().getName() + "<a target='_blank' href='" + req.getContextPath + "/HelloServlet2?bookId=" + me.getValue().getId() + "' >查看</a></br>");
}
//2 显示用户最近的浏览记录

out.write("您最近的浏览历史是<br/>");
Cookie cs[] = req.getCookies();
for(int i=0;cs!=null&&i<cs.length;i++){
     Cookie c = cs[i];
     if("bookHistory".equals(c.getName())){
          String value = c.getValue();
          String ids[] = value.split("\\-");
          for(String id:ids) {
              out.write(BookDB.findBookById(id).getName()+"<br/>")
          }
          break;
     }
}
~~~~~~

~~~~~~
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
// 显示商品的详细信息, 写Cookie
out.write("详细呢容: <br/>");
String bookId = req.getParameter("bookId");
Book book = BookDB.findBookById(bookId);
out.println(book.toString());

// 写cookie: 记住浏览历史记录

String bookids = makeIds(req, bookId);
Cookie c = new Cookie("bookHistory", bookids);
c.setMaxAge(Integer.MAX_VALUE);
c.setPath(req.getContextPath());
res.addCookie(c);

// 组织写到cookie中的书记的id, 多个id之间分割.最多三个
/*

一个cookie都没有      当前书:1            1
由cookie,但没有bookhistory  1       1
bookHistory=1               2           2-1
2-1                         2           1-2
*/
private String makeIds(HttpServletRequest res, String bookId){
    Cookie cs[] = res.getCookies();
    if(cs==null) return bookId;
    Cookie bookHistoryCookie = null;
    for(Cookie c:cs) {
         if("bookHistory".equals(c.getName())){
             bookHistoryCookie = c;
             break;
         }
    }
    if(bookHistoryCookie == null) return bookId;
    String value = bookHistoryCookie.getValue();
    String ids[] = value.split("\\-");
    LinkedList<String> list = new LinkedList<String>(Arrays.asList(ids));
    if(list.size()<3){
        if(list.contains(bookId))list.remove(bookId);
        list.addFirst(bookId);
    } else {
        if(list.contains(bookId)){
            list.remove(bookId);
        } else {
            list.removeLast();
        }
        list.addFirst(bookId);                
    }
    StringBuffer sb = new StringBuffer();
    for(int i=0; i< list.size();i++){
        if(i>0) ab.append("-");
        sb.append(list.get(i));
    }
    return sb.toString();
}
~~~~~~
