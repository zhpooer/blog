title: 传智播客day23-框架学习基础加强
date: 2014-05-05 09:04:44
tags:
- 传智播客
---

# 回顾 #
围绕 java5 特性: 泛型技术, 反射技术


# 动态代理 #

## 代理模式和装饰者模式的区别 ##
装饰器模式: java基础IO流, javaWeb request 和 response包装  
对原有代码功能进行增强

代理模式: 对真实对象访问的**拦截作用**

~~~~~~
interface DAO { // 业务接口
    void insert();
}
class MySQLDAO implements DAO{ // 业务实现类
    public void insert(){}
}
~~~~~~

装饰模式
~~~~~~
// 装饰对象
class LogDAO implement DAO {
    // 传入被装饰对象
    DAO dao;
    // 构造方法传入
    public LogDAO(DAO dao){
        this.dao = dao;
    }
    public void insert(){
        // 代码增强
        dao.insert(); // 调用被装饰对象insert()
        // 代码增强
    }
}
// 使用
DAO dao = new LogDAO(new MySQLDAO());
~~~~~~
代理模式
~~~~~~
class DAOProxy implements DAO {
    // 被代理对象
    DAO dao;
    // 构造方法传入被代理对象
    public DAOProxy(DAO dao){
        this.dao = dao;
    }
    //可以做代码增强, 和访问拦截
    public void insert(){
        // 判断是否具有权限
        // 代码增强
        if(有权限) dao.insert(); // 调用被装饰对象insert()
        else 权限不足
        // 代码增强
    }
}
~~~~~~

## 动态代理 ##
代理模式(静态代理): 需要开发人员编写代理类

动态代理: 不需要开发人员编写代理类,
代理类在JVM中自动创建(动态构造)  
Struts2 使用静态代理  
Spring 使用动态代理

代理对象需要和被代理对象具有相同的业务方法(*同一个接口*)

## 动态代理的过程 ##

~~~~~~
interface DAO { // 业务接口
    void insert();
    void update();
}
class MySQLDAO implements DAO{ // 业务实现类
    public void insert(){
        println("insert");
    }
    public void update(){
        println("update");
    }
}
~~~~~~
在 MysqlDAO 中实现日志记录
* 方式一: 继承代码, 覆盖业务方法, 添加新功能(要求业务对象必须是手动创建)
* 方式二: 装饰者, 对已经存在的对象进行功能*扩展*
* 方式三: 代理, 对已经存在的对象进行功能*扩展*

    利用 `java.reflect.Proxy` 类, 提供 `newProxyInstance(classLoader, interfaces[], invocation)` 方法, 构造动态类
    ~~~~~~
    final DAO dao = new MySQLDAO();
    DAO proxy = Proxy.newProxyInstance(dao.getClass().getClassLoader(),
        dao.getClass().getInterfaces(), new invocationHandler {
        Object invoke(Object proxy, Method method, Object[] args){
            Object rs = method.invoke(dao, args);
            // 代码增强拦截
            log("调用了方法");
            return rs;
        }
    });
    // 代理对象内部实现
    $$Proxy implement DAO{
        public void insert(){
            handler.invoke(this, method, args);
        }
        public void update(){
            handler.invoke(this, method, args);
        }
    }
    ~~~~~~

## 动态代理案例,解决请求中文乱码 ##
index.jsp
~~~~~~
<form action="request" method="post">
    <input type="text" name="msg"/>
</form>
<form action="request" method="get">
    <input type="text" name="msg"/>
</form>
~~~~~~
~~~~~~
public class EncodeFilter extends Filter {
    public void doFilter(req, res, chain){
        req.setCharactorEncoding("utf-8");
        ServletRequest proxy = Proxy.newProxyInstance(req.getClass().getClassLoader,
            req.getClass().getInterfaces(), new InvocationHandler(){
                public Object invoke(Object proxy, Method method, Object[] args){
                   // 判断清水方式是否为get
                    if(req.getMethod().equalsIgnoreCase("get")){
                        if(method.getName().equals("getParameter")){
                            String value = method.invoke(req, args);
                            if(value=!null) {
                                value = new String(value.getBytes("ISO-8859-1"), "utf-8");
                            }
                            return value;
                        }
                    }
                    return method.invoke(req, args);
                }
            });
        chain.doFilter(proxy, res);
    }
}
if(req.getMethod().equalsIgnoreCase("get")){
    String value = super.getParameter(name);
    if(value=!null){
        value = new String(value.getBytes("ISO-8859-1"), "utf-8");
    }
    return value;
}
~~~~~~


# 注解技术(Annotation) #
注释: 给其他开发人员阅读

注解: 给程序阅读的注释, 取代配置文件

配置文件: 为了代码修改方便, 将经常变化信息, 写入配置文件

注解: 程序内部的配置信息, 出现原因: 
1. 软件复杂, 配置文件太大
2. 程序中都是接口, 没有实现类, *造成程序的可读性越来越差*
~~~~~~
List list = new ArrayList(); //耦合
// 解决耦合
List list = // 工厂提供实现类, 工厂读取配置文件获取实现类
// 造成程序的可读性越来越差
~~~~~~

## JDK常见的注解 ##
三种基本注解:
* `@Override`: 重写父类方法, 编译时进行代码检查, 
JDK5 只能用于方法覆盖, JDK6 可以用于方法实现
* `@Desprecated`: 表示程序某个方法过时
* `@SuppressWarnings`: 抑制文件警告
~~~~~~
@SuppressWarnings("all")
@SuppressWarnings("Desprecated")
~~~~~~



## 注解在企业开发中的应用 ##
完整应用步骤
1. 定义注解
2. 在目标类或者方法、变量上, 应用注解
3. 在程序运行时, 通过反射技术取解析获得注解中的信息

在企业开发中, 最常见的是框架内部已经提供注解,
已经提供解析注解的程序, 只需要将注解应用到代码中.

### 自定义注解 ###
Java 数据类型: 基本类型, 数组, class, enum, interface, @interface

~~~~~~
// 定义注解
public @interface PersonInfo {
    // 作为配置文件的替代
    // 配置信息是通过属性完成
    String name() default "小强";
    int age();
}
~~~~~~
* 注解支持类型:String, 基本数据类型, enum, Class, 其他注解类型, 以上数据类型 相应的一维数组

* 特殊注解,
  * 在应用注解中, 需要为每个属性赋值
  * 如果只有 `value` 属性, 可以省略掉 `value=`
  ~~~~~~
  @interface DBInfo{
     String value();
  }
  // 使用
  @DBInfo("Mysql")
  ~~~~~~

### JDK元数据 ###
修饰注解的注解
* `@Retention`, 修饰注解的有效范围
  * `RetentionPolicy.SOURCE`: 在.java文件中有效, 给编译器使用
  * `RetentionPolicy.CLASS`: 给类加载器用
  * `RetentionPolicy.RUNTIME`: 给程序使用
* `@Target`: 表示注解可以应用的目标
  * `ElementType.ANNOTATION_TYPE`
  * `ElementType.CONSTTRCTOR`
  * `ElementType.FIELD`
  * `ElementType.Method`
  * `ElementType.LOCAL_VARIABLE`
  * `ElementType.TYPE`: 用于类和接口
  * `ElementType.PACKAGE`
  * `ElementType.PARAMETER`
* `@Documented`: 生成的注解会被生成到文档
* `@Inherited`: 应用了这个注解的类的子类会自动继承注解

## 提取注解中的信息 ##

编写流程: 
1. 编写注解类
2. 应用注解
3. 提取注解信息, 通过 `java.lang.reflect.AnnotatedElement`接口,
所有反射接口都实现了他
  * 拿到注解修饰目标的反射对象
  * 通过 `AnnotatedElement` 接口提供的API, 操作注解
  
### 案例一 银行转账控制 ###

需求: 每次最大金额20万

~~~~~~
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface BankInfo{
    double maxmoney();
}
public class Bankz {
    // 执行转账
    @BankInfo(maxmoney=200000)
    public void transfer(String from, String to, double money) {
        // money 最大金额限制是200万元
        // 1. 传统方式, 读取properties, 最大金额
        // double maxmoney = Double.parseDouble(bundle.getBundle("bank"));
        Class<Bank> c = Bank.Class;
        Method c = c.getMethod("transfer", String.class, String.class, double.class);
        // 判断方法中是否由注解信息
        if(c.isAnnotationPresent(BankInfo.class)){
            BankInfo info = method.getAnnotaion(BankInfo.class);
            double maxmoney = info.maxmoney();
        }
        if(money <= maxmoney)
            println("正常转账, 从" + from + "账户向" + to + "账户转了" + money);
    }
}
~~~~~~


### 案例二 获得数据库连接参数 ###
~~~~~~
// 定义注解
@Retentioin(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface JdbcInfo {
    String url();
    String driverClass();
    String username();
    String password();
}
// 应用注解, 配置注解信息
public class JdbcConnection {
    @JdbcInfo(driverCalss="com.mysql.jdbc.Driver", url="jdbc:mysql:///test",password="", user="")
    public void connect(){
        // 运行时, 通过反射获取注解信息
        Class cl = JdbcConnection.class;
        Method m = c.getMethod("connect");
        if(m.isAnnotationPresent(JdbcInfo.class)) {
            JdbcInfo info = m.getAnnotaion(JdbcInfo.class);
            String url = info.url(); //
            Class.forName(driverClass);
            // 连接数据库
            Connection conn = DriverManager.getConnection(url, user, password);
        } else {
            throw new RuntimeException("连接失败");
        }
    }
}
~~~~~~

## 案例综合 权限控制 ##

基于动态代理和注解对方法级别实现细粒度权限控制

* 粗粒度: URL级别的权限控制
  * 点击页面中的每个链接, 对应服务器一个URL地址, 一个请求, 一个 Servlet
  * 在过滤器中判断当前登陆用户,是否具有访问url的权限
  * 一次请求, 一次判断
* 细粒度: 方法级别的权限控制
  * 每个功能对应一个url, 一个Sevlet, 调用多个业务层 Service, 多个持久层DAO
  * 通过注解信息, 判断用户, 控制方法的访问
  * 一次请求, 多次判断

### 编程实现 ###

#### 建立数据库 ####
~~~~~~
create table privilege(
    id int not null auto_increament,
    name varchar(255) not null,
)
insert into privilege values(1,"添加权限"),(2,"删除权限"), (3, "更改权限");
create table user(
    id int not null auto_increament,
    username varchar(255) not null,
    password varchar(255) not null,
)
create table (
    user_id int,
    privilege_id int,
    constraint user_id_fk foreign key(user_id) reference user(id),
    constraint privilege_id_fk foreign key(privilege_id) reference privilege(id),
)
~~~~~~

#### 代码框架 ####
![代码框架](/img/privilege_control.png)
##### 登陆 #####
Login.jsp
~~~~~~
<h1>登陆</h1>
${errMsg}
<form action="Login" method="post">
   用户名: <input type="text" name="username"/>
   密码: <input type="password" name="password"/>
   <input type="submit"/>
</form>
~~~~~~
~~~~~~
public class User{
    private int id;
    private String username;
    private String password;
}
~~~~~~
LoginServlet.java

使用 c3p0 和 DBUtil
~~~~~~
public void doGet(req, res){
    String username = req.getParameter("username");
    String password = req.getParameter("password");

    DataSource ds = JDBCUtils.getDataSource();
    QueryRunner qr = new QueryRunner(ds);
    User u = qr.query("select * from user where username=? and password=?", new BeanHandler<User>(User.class), username, password);
    if(user==null){
        req.setAttribute("errMsg", "密码错误");
        req.getRequestDispatcher("login.jsp").forward(req,res);
    } else {
        req.getSession().setAttribute("user", u)
        res.sendRedirect(request.getContextPath()+"main.jsp")
    }
}
~~~~~~
访问页面的每一个链接, 执行Service的各种方法
~~~~~~
<h1>主页</h1>
<h1>登陆用户: ${user.username}</h1>
<a href="/productAdd">添加商品</a>
<a href="/productDel">删除商品</a>
<a href="/productUpdate">修改商品</a>
<a href="/productQuery">查询商品</a>
~~~~~~
##### 实现业务功能 #####
~~~~~~
// producetAddServlet
public void doGet(req, res) {
    service.add();
}
// producetDelServlet
public void doGet(req, res) {
    service.del();
}
// producetUpdateServlet
public void doGet(req, res) {
    service.update();
}
// producetQueryServlet
public void doGet(req, res) {
    service.update();
}
~~~~~~
ProdectService.java
~~~~~~
public class ProductService {
    @PrivilegeInfo("添加商品")
    public void add(){
        prinln("add")
    }
    @PrivilegeInfo("删除商品)"
    public void del(){
        prinln("del")
    }
    public void update(){
        prinln("update")
    }
    public void query(){
        prinln("query")
    }
}
~~~~~~

#### 通过注解和动态代理添加权限控制 ####
访问目标方法需要权限
~~~~~~
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface privilege {
    String value();
}
~~~~~~
为了实现动态代理, 添加服务接口, 给服务类添加注解
~~~~~~
public interface IProdectService{
    @PrivilegeInfo("添加商品")
    public void add();
    @PrivilegeInfo("删除商品)"
    public void del();
}
public class ProductService implement IProdectService{
}
~~~~~~
动态代理实现权限控制
~~~~~~
// 在执行 doGet 之前
public beforeDoGet(){
    User user = req.getSession().getAttribute("user");
    service = ProducetProxyFactory.makeProxy(new ProductService, user);
    service.doSome();
}

public class ProductProxyFactory {
    // user 当前登陆用户
    // ProductService 当前业务对象
    public static IProductService makeProxy(final ProductService ps, User user) {
        Proxy.newProxyInstance(classLoader,
            ps.getClass().getClassLoader(), new InvocationHandler(){
                public  Object invoke(proxy, method, args){
                    // 判断用户是否登陆
                    if(user==null){
                        throw new RuntimeException("用户还没有登陆");
                    }
                    // 获取业务方法上的注解信息
                    if(method.isAnnotationPresent(PrivilegeInfo.class)) {
                        method.getAnnotaion(PrivilegeInfo.class);
                        String needPrivilege = privilegeInfo.value();

                        // 判断用户数是由具有权限
                        DataSource ds = JDBCUtils.getDataSource();
                        QueryRunner qr = new QueryRunner(ds);
                        String sql = "select privilege.* from privilege "
                            + "inner join user_privilege up on privilege.id=up.privilege_id where up.user=?"
                        List<Privilege> privileges = qr.query(sql, new BeanListHandler<Privilege>, user.getId);
                        for(p <- privileges) {
                            if(p.getName().equals(needPrivilege))
                                method.invoke(ps, args);
                        }
                        throw new Error("权限不足");
                    } else {
                        return method.invoke(ps, args)
                    }
                }
            });
        return proxy;
    }
}
~~~~~~

~~~~~~
public class Privilege {
    private int id;
    private String name;
}
~~~~~~
# 类加载器 #

编写.java源码 -> 编译器编译 -> .class 字节码文件 -> ClassLoader 加载.class文件

类加载器负责将.class文件 加载到内存中, 并为之生成 java.lang.Class 对象
![ClassLoader继承结构图](/img/class_loader.png)

* BootStrap 引导类加载器: 负责加载Java的核心类, 是C语言的代码, 负责加载 `JRE/lib/rt.jar`(所有常用JDK类, 都属于它)
~~~~~~
// ArrayList 位于rt.jar, 由 BootStrape 加载
ArrayList list = new ArrayList();
println(list.getClass().getClassLoader());  // 是Null, 因为c语言写的, 没有类
// 打印Bootstrap 加载类路径
URL[] urls=sun.misc.Launcher.getBootstrapClassPath().getURLs(); 
for (int i = 0; i < urls.length; i++) { 
	System.out.println(urls[i].toExternalForm()); 
}
~~~~~~
* ExtClassLoader 扩展类加载器: 加载 `jre/lib/ext/*.jar`

* AppClassLoader (系统)应用类加载器, `ClassLoader.getSystemClassLoader()` 直接获得 系统类加载器

## 细节 ##

> 面试题: 程序运行时, 加载类, 类不在工程代码中, 是否报错?

工程代码中由 AppClassLoader 加载, 还有 BootStrap 和 ExtClassLoader 加载类

> 类加载器顺序和特性?


1. 委托机制: 优先由父类加载器加载,父类加载器找不到类,
子类加载器尝试加载, 如 `NoSuchMethodError`,
可能说明类加载器已经在之前加载了一个另一个类, 而那个类没有这个方法

2. 全盘负责机制: 一个类被加载, 这个类所依赖和引用的类也会被这个类加载器加载

> 自定义类加载器, 不一定是采用委托机制

所有自定义加载器, 必须继承 ClassLoader, 重写`findClass`

> .class 文件字节码, 如何称为Class对象

1. 通过io流, 读取.class文件, 生成byte数组
2. 调用ClassLoader 提供的 `defineClass`, 生成 Class 对象

> 常见错误 `java.lang.ClassCassException`

~~~~~~
Class c = myClassLoader.findClass("java.activition.MimeType");
MimeType mimeType = (mimeType) c.newInstance();
~~~~~~
MimeType 遵循委托机制, 由父类加载器加载

c.newInstance() 由子类类加载器加载

**不同类加载器, 加载同一个类, 也会出现不同Class对象**

## Tomcat的类加载器 ##

Jndi: 通过配置, 将一个对象交给tomcat创建和管理, 在程序中目录访问原则,
获取到tomcat中的绑定的对象

Tomcat 类加载器是区别于JVM的加载器的

比传统三个类加载器, 添加 Common 类加载器(Tomcat/lib) 和 WebApp(WEB-INF/classes和lib)类加载器

违背父类委托机制, 优先加载当前工程下类和jar, 再去加载tomcat/lib公共类和jar包, 加载顺序:
* BootStrap classes of yourt JVM
* System class loader classes
* /WEB-INFO/calsses
* /WEB-INFO/lib/*.jar
* $CATALINA_HOME/lib
* $CATALINA_HOME/lib/*.jar
