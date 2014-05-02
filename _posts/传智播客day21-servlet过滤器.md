title: 传智播客day21-servlet过滤器
date: 2014-04-28 09:02:59
tags:
- 传智播客
---

# 过滤器概述 #
Filter, 应用中的保安. 利用过滤器来实现对请求和响应的拦截.

# 编写过滤器的步骤 #
1. 编一个类, 实现 javax.servlet.Filter
~~~~~~
public class FilterDemo1 implements javax.servlet.Filter {
    // 由容器调用, 每次响应和调用都会经过该方法
    public void doFilter(ServletRequest req, ServletResponse, FilterChain chain){
          // 执行前
          chain.doFilter(req, res); // 放行,让下一个资源执行
          // 执行后
    }
    // 由容器调用, 完成过滤器的初始化
    public void init(FilterConfig config){}
    public void destroy(){}
}
res.getWriter().write("Hello");
~~~~~~
2. 配置web.xml,指定哪些资源需要拦截
~~~~~~
<filter>
<!-- 定义一个过滤器, 并定制名称 -->
    <filter-name>FilterDemo1 </filter-name>
    <filter-class>...</filter-class>
</filter>
<!-- 多个过滤器的拦截的顺序, 由filter-mapping元素出现的顺序决定 -->
<filter-mapping>
    <filter-name>FilterDemo1</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
~~~~~~

# 过滤器的执行过程 #
![过滤器执行过程](/img/filter_process.png)

## 生命周期 ##
1. **应用在加载时**会被初始化和初始化
2. 针对用户的每次资源访问, 容器都会调用`doFilter`方法
3. 应用被卸载或服务器停止时, 会执行`destroy`

## 配置过滤器初始化参数 ##
~~~~~~
public init(FilterConfig f){
    String value = f.getInitParameter("encoding");
    Enumeration en = f.getInitParameterNames();
}
~~~~~~
~~~~~~
<filter>
<init-param>
    <param-name></param-name>
    <param-value></param-value>
</init-param>
</filter>
~~~~~~

# 过滤器的简单案例 #
## 统一字符编码的过滤器 ##
~~~~~~
// setCharactorFilter
public void doFilter(ServletRequest req, ServletResponse, FilterChain chain){
    String encoding = "UTF-8";
    String value = filterConfig.getInitParameter("encoding");
    if(value!=null){
        encoding = "utf-8";
    }
    req.setCharactorEncoding(encoding); // 只针对POST请求, 对get请求不起作用
    res.setContentType("text/html;charset=" + encoding);
    chain.doFilter(req, res);
}
~~~~~~
## 动态资源缓存过滤器(NoCache) ##
~~~~~~
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain){
    HttpServletRequest req;
    HttpServletResponse res;
    try {
        req = (HttpServletRequest) request;
        res = (HttpServletRequest) response;
    } catch {
        throw new RuntimeException("is not a http request");
    }
    res.setDateHeader("", -1);
    res.setHeader("Cache-Control", "no-Cache");
    res.setHeader("Pragma", "no-Cache");
    chain.doFilter(req, res);
}
~~~~~~
## 静态资源缓存过滤器 ##
~~~~~~
<init-param>
    <param-name>css </param-name>
    <!-- 单位为小时 -->
    <param-value>1 </param-value> 
</init-param>
~~~~~~

~~~~~~
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain){
    HttpServletRequest req;
    HttpServletResponse res;
    try {
        req = (HttpServletRequest) request;
        res = (HttpServletRequest) response;
    } catch {
        throw new RuntimeException("is not a http reqest");
    }
    long time = 0; //缓存的时间的偏移量

    String uri = res.getRequestURI(); // /servlet/index.html

    String extendName = uri.subString(uri.lastIndex(".") + 1);
    if(html!=null && "html".equals(extendName)){
        String value = filterConfig.getInitParameter("css");
        time = Long.parse(value*60*60*1000);
    }
    res.setDateHeader("Expires", System.currentTimeMillis()+time);
    chain.doFilter(req, res);
}
~~~~~~

## 自动登陆过滤器 ##
~~~~~~
// 用户账号和密码存到Cookie中, 保存形式是 'user_password'
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain){
    HttpServletRequest req;
    HttpServletResponse res;
    try {
        req = (HttpServletRequest) request;
        res = (HttpServletRequest) response;
    } catch {
        throw new RuntimeException("is not a http reqest");
    }

    // 判断用户是否登陆
    HttpSession session = req.getSession();
    User user = (User)session.getAttribute("user");
    if(user==null){
        Cookie[] cs = req.getCookies();
        // 找到用户名和密码
        for(c <- cs if "loginInfo".equals(c.getName()){
            /// 再次比对用户名和密码
            String username = c.getValue().split("\\_")[0];
            String password = c.getValue().split("\\_")[1];
            if(password==password){
                User u = new User();
                u.setUsername(username);
                u.setPassword(password);
                req.getSession().setAttribute("user", u);
            }
            break;
        }
    }
    chain.doFilter(req, res);
}
~~~~~~
### 用户密码加密 ###
~~~~~~
public static String md5(String s){
    MessageDigest md = MessageDigest.getInstance("md5");
    byte b[] = md.digest(s.getBytes());
    Base64Encoder en = new Base64Encoder();
    return en.encode(b)
}
//  cookie中不能存中文 , 可以用base64编码
new Base64Encoder().encode(s.getBytes());

~~~~~~

# 过滤器的高级配置 #
~~~~~~
<filter-mapping>
    <!-- 设置用户请求时, 什么时候执行过滤 -->
    <!-- 默认就是request -->
    <dispatcher> REQUEST </dispatcher> <!-- 请求阶段 -->
    <dispatcher> FORWARD </dispatcher> <!-- 转发阶段 -->
    <dispatcher> INCLUDE </dispatcher> <!-- 动态包含 -->
    <dispatcher> ERROR </dispatcher> <!-- 出异常时 -->
</filter-mapping>
~~~~~~
注:
* page指令, errorPage="*.jsp", 属于转发方式
* web.xml 配置的全局错误提示页面, 是的状态是 ERROR

# 过滤器的高级案例 #

## 全站中文乱码解决过滤器 ##
请求: Post Get参数乱码; 响应输出乱码解决过滤器
~~~~~~
public void doFilter(){
    HttpServletRequest req;
    HttpServletResponse res;
    try {
        req = (HttpServletRequest) request;
        res = (HttpServletRequest) response;
    } catch {
        throw new RuntimeException("is not a http reqest");
    }
    // 看服务端有没有配置编码参数
    String encoding = "utf-8";
    String value = filtercofnig.getInitParameter("encoding");
    if(value!=null){
        encoding = value;
    }
    // 解决post方式的乱码
    req.setCharactorEncoding(encoding);
    res.setCharactorEncoding(encoding);
    res.setContentType("text/html;charset=" + encoding);
    // 解决get方式的乱码
    MyHttpServletRequest mrequest = new MyHttpServletRequest(request);
    chain.doFilter(mrequest, res);
}
class MyHttpServletRequest extends HttpServletRequestWrapper {
     public MyHttpServletRequest(HttpServletRequest req){
         super(req);
     }
     // 只处理get请求编码
     @Override public String getParameter(String name) {
         String value = super.getParameter(name);
         if(value==null){
             return null;
         }

         String method = super.getMethod();
         if("get".equalsIgnoreCase(method)) {
             value = new String(value.getBytes("iso-8859-1"), super.getCharactorEncoding());
         }
         return value;
     }
}
~~~~~~
## 脏话过滤器 ##
~~~~~~
//必须把字符编码过滤器放在前面
public void doFilter(){
    HttpServletRequest req;
    HttpServletResponse res;
    try {
        req = (HttpServletRequest) request;
        res = (HttpServletRequest) response;
    } catch {
        throw new RuntimeException("is not a http reqest");
    }
    DirtyWordsHttpServletRequest dwrequest = new DirtyWordsHttpServletRequest(req);
    chain.doFilter(dwrequest, res);
}
class DirtyWordsHttpServletRequest extends HttpServletRequestWrapper{
    private String dwords[] = {"xx", "xxx"};
    public String getParameter(String name){
        String value = super.getParameter(name);
        if(value==null) return null;
        for(String s : dwords){
            value = value.replace(s, "*")
        }
        return value;
    }
}
~~~~~~

## HTML标记过滤器 ##
`<`换成`&lt;`, 可以防止代码注入
~~~~~~
class HtmlHttpServletRequest extends HttpServletRequestWrapper{
    private String dwords[] = {"xx", "xxx"};
    public String getParameter(String name){
        String value = super.getParameter(name);
        if(value==null) return null;
        value = filter(value);
        return value;
    }
}
~~~~~~


## 全站 GZIP 压缩过滤器 ##
![gzip压缩原理](/img/gzip_filter.png)
~~~~~~
public void doFilter(){
    HttpServletRequest req;
    HttpServletResponse res;
    try {
        req = (HttpServletRequest) request;
        res = (HttpServletRequest) response;
    } catch {
        throw new RuntimeException("is not a http reqest");
    }
    GZIPHttpResponse ghp = new GZIPHttpResponse(res);
    chain.doFilter(req, ghp);
    
    ByteArrayOutputStream bo = new ByteArrayOutputStream();
    GZIPOutputStream gout = new GZIPOutputStream();
    gout.write(ghp.getBytes());
    gout.close();
    Byte[] ba = bo.toByteArray();
    res.setHeader("Content-Encoding", "gzip");
    res.setContentLength(o.length); // 不设置该值时, 浏览器访问Html静态资源会慢
    res.getOutputStream().write(ba);


}
public class GZIPHttpResponse extends HttpServletResponseWrapper{
    private ByteArrayOutputStream baos = new ByteArrayOutputStream();
    private PrintWriter pw = null;
    // 字符流
    @Override public ServletOutPutStream getOutputStream(){
        return new GZIPOutputStream(baos);
    }

    // 字节流
    @Override public PrintWriter getWriter(){
        //设置编码, 解决字符编码问题, 所以一定要设置在 字符编码过滤器中, 设置 res.setCharactorEncoding();
        pw = new PrintWriter(new OutputStreamWriter(baos, super.getCharactorEncoding()));
        return pw;
    }

    public Byte[] getBytes(){
        if(pw!=null) pw.flush();
        baos.flush();
        baos.getBytes();
    }
}

class GZIPServletOupStream extends ServletOutPutStream {
    private ByteArrayOutputStream baos;
    public GZIPServletOupStream( ByteArrayOutputStream baos ){
        this.baos = baos;
    }
    public void write(int b) {
        baos.write(b);
    }
}
~~~~~~

显示数据用的静态资源需要压缩, 还有`JSP`
~~~~~~
<filter-mapping>
    <filter-name> GZIPFilter </filter-name>
    <url-pattern> *.jsp </url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name> GZIPFilter </filter-name>
    <url-pattern> *.css </url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name> GZIPFilter </filter-name>
    <url-pattern> *.js </url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name> GZIPFilter </filter-name>
    <url-pattern> *.html </url-pattern>
</filter-mapping>
<!-- .... -->
~~~~~~
