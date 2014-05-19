title: 传智播客day12-session
date: 2014-04-15 09:03:15
tags:
- HttpSession
- 传智播客
---

# Session #
默认情况下, 一个浏览器独占一个session对象
* HttpSession也是一个域对象
* session对象由服务器创建
* 每一个session都有一个唯一的ID, 存到Cookie

~~~~~~
void setAttribute(String key, Object value);
Object getAttribute(String key);
void removeAttribute(String key);
String getId();
~~~~~~

~~~~~~
// 放数据
HttpSession session = req.getSession();
session.setAttribute("p", "ppp");
// 取数据
(String) session.getAttribute("p");

req.getSession(true); // 如果没有session, 就创建
req.getSession(false); // 没有sessioin, 不创建, 返回null
~~~~~~

# session访问延时 #
~~~~~~
Cookie c = new Cookie("JSESSIONID", session.getId());
c.setPath(req.getContextPath());
c.setMaxAge();
res.addCookie(c);
~~~~~~

# 案例 #

## 简单的购物车 ##
*ShowAllBooksServlet*
~~~~~~
// 显示所有商品, 提供购买链接
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
out.write("<h1>本站有以下好书</h1>");
Map<String, Book> books = BookDB.findAllBooks();
for(Map.Entry<String, Book> me:bookes.entrySet()) {
   out.write(me.getValue().getName() + "<a href='/BuyServlet?bookId=" + me.getKey() + "'>购买</a><br/>" );
}
out.write("<a href='/ShowCartServlet'>查看购物车</a>")
~~~~~~
*BuyServlet*
~~~~~~

String bookId  = req.getParameter("bookId");
Book book = BookDb.findBookById(bookId);
HttpSession sessioin = request.getSession();
// 生成购物车
List<Book> cart = (List<Book>) session.getAttribute("cart");
if(cart = null) {
    cart = new ArrayList<Book>();
    session.setAttribute("cart", cart);
}
//
cart.add(book);
out.write(book.getName + "商品已经放入购物车");
out.write("<a href='/ShowAllBooksServlet'>继续购物</a>");
~~~~~~

*ShowCartServlet*
~~~~~~
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();
HttpSession session = req.getSession(false); // 只是查询
if(session == null){
    out.write("赶紧购物");
    return;
}

List<Book> books = (List<Book>)session.getAttribute("cart");
if(books=null || books.size()==0){
    out.write("您还没有购物");
} else {
    out.write("您购买了如下购物出<br/>");
    for(Book book:books){
        out.write(book.getName + "<br/>");
    }
}

out.write("<a href='/ShowAllBooksServlet'>继续购物</a>");
out.write("<a href='/ClearCartServlet'>清空</a>");
~~~~~~
*ClearCartServlet*
~~~~~~
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();

List<Book> books = (List<Book>)session.getAttribute("cart");
// 方式一: 干掉session对象
session.invalidate(); // 让服务器端的HttpSession立即消失, 不推荐

// 方式二；值干掉购物车
session.removeAttribute("cart");
res.setHeader("Refresh","2;URL=/ShowAllBooksServlet");
out.write("清空成功, 2秒后转到主页");
~~~~~~

## 用户登陆 ##

~~~~~~
public class User{
    private String username;
    private String password;
    private String nickname;
}
public class UserDB{
    private static List<User> users = new ArrayList<User>;
    static {
        user.add(new User("u1", "123", "u11"));
        user.add(new User("u2", "123", "u22"));
        user.add(new User("u3", "123", "u33"));
    }
    public User find(String username, String password){
         for(User u:users){
              if(u.getName().equals(username)
                  && u.getPassword().equals(password))
              return u;
         }
         return null;
    }
}
<body>
    <form action="LoginServlet" metho="post">
        用户名: <input name="username"/> <br/>
        密码: <input name="password" type='password''/> <br/>
        <input type=submit'/> <br/>
    </form>
</body>
~~~~~~
*IndexServlet*
~~~~~~
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();

HttpSession  session = req.getSession();
User user = (User) session.getAttribute("user");
if(user==null) {
    out.write("< href='Login.html'>登陆</a>")
} else {
    out.write("欢迎你" + user.getNickName());
    out.write("<a href='LogoutServlet'>登出</a>");
}
~~~~~~
*LoginServlet*
~~~~~~
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();

User user = new User();
BeanUtils.populate(user, req.getParameterMap());

User dbUser = UserDB.find(user.getUsername(), user.getPassword());
if(dbUser==null){
    res.setHeader("Refresh", "2;URL=Login.html");
    out.write("两秒后转到登陆页面");
    return;
}
req.getSession().setAttribute("user", dbUser);
res.setHeader("Refresh", "2;URL=/");
out.write("登陆成功, 两秒后到主页");
~~~~~~
*LogoutServlet*
~~~~~~
res.setContentType("text/html;charset=UTF-8");
PrintWriter out = res.getWriter();

session.removeAttribute("user");
res.setHeader("Refresh","2;URL=/");
out.write("清空成功, 2秒后转到主页");
~~~~~~

## 防止表单的重复提交 ##
解决方案一: 客户端解决, 利用javascript
~~~~~~
<input id="btn" byte="button" onclick="toSubmit()"/>
<script type='text/javascript'>
    function toSubmit(){
        document.forms[0].submit();
        document.getElementById("btn").disabled = true;
    }

</script>
~~~~~~
解决方案二: 令牌机制

服务器一生成一个唯一的ID, 向HttpSession中放一个,向表单隐藏域中放一个.

服务器二从中拿到ID,比对, 如果一样删掉HttpSession中的一个,
如果不一样认为是重复提交.
~~~~~~
// 方式一
String token = System.nanoTime() + ""; // 纳秒
// 方式二
String token = UUID.randomUUID().toString();
// 方式三,数据指纹
System.nanoTime() + new Random().nextLong()
// sha | md5
java.security.MessageDigest md = MessageDigest.getInstance("md5");
byte b[] = md.digest(s.getBytes());
// b 不一定由对应的字符, 用Base64编码: 3字节---->> 四字节
sun.misc.BASE64Encoder base = new BASE64Encoder();
String token = base.encode(b);
~~~~~~

## 客户端禁用Cookie ##
解决方案一: 提示: 为了更好的浏览网站, 请不要禁用你的cookie

解决方案二: URL重写

> http://localhost/Servlet;JSESSIONID=11

> HttpServletResponse.encodeUrl(String url); // 重写

> 但是要对所有URL重写

~~~~~~
req.getSession(); // 一定要先运行, 生成一个session
// 如果客户端没有禁用, 就不需要重写了.
res.encodeURL(url); // http://localhost/Servlet;JSESSIONID=11
~~~~~~

# session 生命周期 #

## session的销毁 ##
1. 调用 session.invalidate(), 立刻销毁
2. 超时, 默认是30分钟; 在 web.xml 下配置

~~~~~~
<session-config>
    <session-timeout>30</session-timeout>
    <!-- 单位是分钟 -->
</session-config>
~~~~~~

## session的持久化状态 ##
是由服务器管理, 称为钝化或搁置, 出现情况:
* HttpSession 长时间没有了
* 内存用的太多
* 服务器重启

放在HttpSession中的数据所属的类, 必须实现 java.io.Serializable 接口

