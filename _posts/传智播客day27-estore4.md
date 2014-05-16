title: 传智播客day27-estore4
date: 2014-05-11 09:01:06
tags:
- 传智播客
---

# 订单的删除 #

在订单未付款的状态下, 用户可以去取消订单
注意在 OrderService 要加入事务控制, 参考 [orm简介](/2014/04/26/传智播客day20-orm简介/#-)
~~~~~~
// OrderCancelServlet
public void doGet(req, resp) {
    // 获取要取消的订单
    String orderid = req.getParameter("orderid");
    orderService.cancelOrder(orderid);
    resp.sendRedirect("/orderSearch");
}

// OrderService
public class OrderService {
    public void cancelOrder(String orderid){
        orderDao.deleteOrderItem(orderid);
        orderDao.deleteOrder(orderid);
    }
}

// OrderDao 
public class OrderDao {
    public void deleteOrder(String orderid) {
        String sql = "delete from orderitem where order_id=?";
        queryRunner.update(conn, sql, orderid);
    }
    public void deleteOrderItem(String orderid) {
        String sql = "delete from orders where id=?";
        queryRunner.update(conn, sql, orderid);
    }
}
~~~~~~

# 定时清理未支付订单(任务调度) #

监听器 Listener

定时器
1. `java.util.Timer` 结合 `java.util.TimerTask`
2. `java.util.concurrent.ScheduledExcutorService`
3. `Quantz` **框架**

功能需求: 用户生成订单后,  有效支付时间24小时, 如果24小时不支付,
系统自动清理订单

## 启动定时任务 ##
`ServletContextListener` 中启动定时任务

`Timer` API
* `Timer.schedule(TimerTask task, Date firstTime, long period)`

* `Timer.schedule(TimerTask task, long delay, long period)`


~~~~~~
// cn.itcast.estore.web.Listener
// 清理24小时未付款的订单
public class OrderCleanListener implements ServletContextListener{
    Timer timer = null;
    public void init(){
        Timer timer = new timer();
        timer.schedule(new TimerTask(){
            public void run(){
                orderService.cleanUnPayOrders();
            }
        }, 0, 1000l*60*30); // 每隔30分钟执行一次, 加上 L 防止越界
    }
    public void destroy(){
        if(timer!=null) timer.cancel();
    }
}

// OrderService
public void OrderService {
    public void cleanUnPayOrders() {
        //查询所有订单
        List<Order> orders = orderDao.findAllOrders();
        for(Order order : orders) {
            if(order.getPayState == 0 && System.currentTimeMillis()-order.getCreatetime().getTime() >= 1000*60*60) {
                cancelOrder(order.getId());
            }
        }
    }
}
~~~~~~

# 系统权限管理 #

1. URL 级别权限控制
2. 方法级别权限控制 (一次请求会执行多个方法, 注解+动态代理+反射)

引入配置文件进行权限控制
* 系统用户 admin, normal
* 存在三种页面: 未登录可以访问, user 可以访问, admin 可以访问
* 配置两个文件
  * `admin.conf` 需要管理员才能访问
~~~~~~
/active
/addCart
/delcartProduct
/orderAdd
/orderCancel
/pay
/updateCarrtProductNum
/cart.jsp
/order_add.jsp
~~~~~~
  * `user.conf` 用户才能访问
~~~~~~
/orderCancel
/order.jsp
/product_add.jsp
~~~~~~
* 通过过滤器来进行权限控制
~~~~~~
public class PrivilegeFilter implements Filter {
    private List<String> adminPaths = new ArrayList<String>;
    private List<String> userPath = new ArrayList<String>;
    public void init(FilterConfig filterConfig) {
        String adminFile = filterConfig.getServletContext().getRealPath("/WEB-INF/classes/admin.conf");
        String userFile = getClass().getResource("/user.conf").getFile();
        BufferedReader  r1 = new BufferedReader(FileReader(adminFile));
        BufferedReader  r2 = new BufferedReader(FileReader(userFile));
        while((tmp=r1.readLine())!=null) {
            adminPaths.add(tmp);
        }
        r1.close();
        while((tmp=r2.readLine())!=null) {
            userPaths.add(tmp);
        }
        user.close();
    }
    public void doFilter(req, resp, chain) {
        // 字段访问资源路径是不是在admin.con或者user.con配置,
        String uri = req.getRequestURI();
        String reqPath = uri.subString(req.getContextPath().length);
        // 如果 存在于 admin 或者 user 需要登陆, 获取当前登陆 角色,
        // 判断对应访问是否包含当前资源访问路径
        if(adminPaths.contains(reqPath) || userPaths.contains(reqPath)){
            User user = req.getSession().getAttribute("user");
            if(user==null){
                throw new RuntimeException("权限不足, 请先登陆");
            } else {
                if(user.getRole().equals("admin")) {
                    if(adminPaths.contains(reqPath)){
                        chain.doFilter(req, resp);
                    } else {
                        throw new RuntimeException("权限不足");
                    }
                } else if(user.getRole().equals("user")) {
                    if(userPath.contains(reqPath)){
                        chain.doFilter(req, resp);
                    } else {
                        throw new RuntimeException("权限不足");
                    }
                }
            }
        } else {
            // 如果不存在, 不需要登陆, 直接放行
            chain.doFiler(req, resp);            
        }
    }
    public void destroy(){
    }
}
~~~~~~

# 异常页面的编写 #

Servlet : 配置web.xml 指定的异常处理页面

JSP: `<%@page errorPage="..."%>`, 解决 500

~~~~~~
<!-- web.xml 配置异常处理 -->
<error-page>
    <error-code> 500 </error-code>
    <location>/500.jsp </location>
</error-page>
<error-page>
    <error-code> 404 </error-code>
    <location>/404.jsp </location>
</error-page>
~~~~~~

* 404.jsp, 自动跳转到系统主页面
  1. `response.setHeader("refresh", "时间(秒);url=/");`
  2. `<meta>` 标签
~~~~~~
<meta http-equiv="refresh" content="200;url=www.google.com">
~~~~~~
* 500页面, 在错误页面, 获取 异常 信息
~~~~~~
<%page isErrorPage="true"%>

<%=exception.getClass().getName()%>
<%=exception.getMessage()%>
~~~~~~

# 日志记录 #

在软件开发时 或者 项目后期运维时, 用户系统信息记录到指定的文件中
* 用户调试
* 根据日志信息修复系统Bug

## 日志技术选型 ##
* JDK Logging, `java.util.logging`
* Log4j 日志技术, Apache后来运维, 第三方日志框架, `log4j1.x`和`log2.x`
  * 主流: log4j1.2.x
* commons-logging(Apache), JCL, 主要是为了统一不同的日志实现框架
  * commons-logging + log4j
  * commons-logging + jdk loggin
  * 开发人员只需要学习 commongs-logging 接口, 就集成不同日志实现技术(最新的struts2, spring 使用 commons-logging)
* slf4j (Simple Logging Facade for Java), 作用类似 commons-logging, 起到同一日志接口实现
  * Hibernate 使用 slf4j 日志接口

## 日志实现技术 log4j ##

区分: 日志技术和 System.out
   * System.out将信息打印到控制台, 有些服务器, 将控制台所有信息记录日志文件
   * 日志技术, 向日志文件记录日志, 分等级记录, 通过控制等级, 控制不同级别日志输出

## 编写Log4j ##
~~~~~~
@Test
public void test不写日志文件() {
    // 1. 需要创建日志记录器
    Logger logger = Logger.getLogger(this.getClass());
    // 2. 指定记录器的输出源(日志输出到哪里去)
    BasicConfigurator.configure(); //将日志输出到控制台
    // 3. 记录日志(提供6个日志级别)
    // 在输出日志时, 只会输出比指定日志级别更高日志级别
    logger.setLevel(Level.ERROR);
    // 默认日志级别是Debug
    // fatal error warn
    // info debug trace
    logger.debug("调试信息");
}

~~~~~~

## 编写Log4j配置文件 ##
* xml 格式: log4j.xml
* properties 格式: log4j.properties(最常见)
  * 在src下新建 log4j.properties 配置文件
  * 配置记录器 logger (采用那个输出源, 疏忽日志级别)
  * 配置输出源 Appender (输出到哪里)
    * org.apache.log4j.ConsoleAppender(控制台) 
    * org.apache.log4j.FileAppender(文件) 
    * org.apache.log4j.DailyRollingFileAppender (每天产生一个日志文件) 
    * org.apache.log4j.RollingFileAppender (文件到达指定大小时产生一个新文件) 
    * org.apache.log4j.WriterAppender (将日志信息以流格式发送到任何地方) 
  * 配置布局 layouts (输出格式)
    * org.apache.log4j.HTMLLayout （以HTML表格形式布局）
    * org.apache.log4j.PatternLayout （可以灵活地指定布局模式）
      * %m 输出代码中指定的消息
      * %p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL
      * %r 输出自应用启动到输出该log信息耗费的毫秒数
      * %c 输出所属的类目，通常就是所在类的全名
      * %t 输出产生该日志事件的线程名
      * %n 输出一个回车换行符，Windows平台为“\r\n”，Unix平台为“\n”
      * %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
      * %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)基本应用
    * org.apache.log4j.SimpleLayout （包含日志信息的级别和信息字符串）
    * org.apache.log4j.TTCCLayout （包含日志产生的时间、线程、类别等信息）
    
~~~~~~
log4j.rootLogger(默认记录级别) = DEBUG, A1 # 输出级别 + 输出源...
log4j.logger.包名(包记录级别) = 输出级别 , 输出源1 , 输出源2

# appender
log4j.appender.A1 = org.apache.log4j.ConsoleAppender

# layout
# log4j.appender.输出源名称.layout = 布局实现类
log4j.appender.A1.layout = org.apache.log4j.PatternLayout # 使用
# 自定义输出格式
log4j.appender.A1.layout.ConversionPattern = %-4r [%t] %-5p %c %x - %m%n

# 输出到文件
log4j.appender.FILE = org.apache.log4j.FileAppender
log4j.appender.FILE.File = /var/log

~~~~~~

# 在项目代码中配置 Log4j #

1. 定义记录器
~~~~~~
private static final Logger LOG = Logger.getLogger(OrderService.class);
~~~~~~
2. 在程序中使用六个级别方法, 记录日志  
   常用级别: ERROR, WARN, INFO, DEBUG
~~~~~~
LOG.error(e.getMessage(), e);
~~~~~~

# 项目TODO #
* 分页控制, 提取标签
* 配置多个tomcat运行, 集群处理(Apache + tomcat)
* 缓存优化, 静态缓存
* Tomcat配置 gzip 压缩
~~~~~~
<Connector compress="on" compressableMimeType="text/html, text/css">
</Connector>
~~~~~~
