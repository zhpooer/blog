title: 传智播客day24-Estore商城和JavaMail
date: 2014-05-06 09:07:03
tags:
- 传智播客
---

Javaweb 课程 是 Javaee 课程基础, 课程定位网站开发(PHP .NET一样)
[最终代码](https://github.com/zhpooer/itcast-estore)
# 软件开发流程 #

1. 调研需求: 售前工程师(需求工程师)配合销售, 第一时间了解客户需求
2. 需求分析: 需求工程师, 需求文档(系统描述项目有哪些功能)
3. *制作系统静态页面*  
  Java工程师, 做系统页面(Jquery easyUI 框架, ExtJS框架)
4. 架构师进行技术选型, 搭建系统架构(资深开发人员, 技术经理)
5. *数据库设计*(Power Designer)
6. *功能开发*
7. 测试


# Estore 商城需求功能分析 #

* 导入 Estore 静态页面
  * 导入项目方式, 不建议 import, 因为 myeclipse 项目, eclipse项目, maven项目,
  ant项目, 结构都不一样
    1. 新建 web 项目, estore, J2ee5.0
    2. 将demo代码赋值过来, 将源码和配置文件, 复制src, 将页面代码复制到
    3. 企业前端工程师(美工)提供静态页面(html)  
    我们要做的是, 就是将html中的静态数据, 连接数据库编程动态数据
    4. 部署到Tomcat
* 功能分析
  1. 注册(验证码, 激活邮件, md5密码加密)
  2. 登陆
  3. 添加商品(文件上传)
  4. 查看商品(条件查询, 分页)
  5. 购物车(添加商品到购物车, 查看购物车, 修改购物车中的商品数量)
  6. 订单生成
  7. 订单查看
  8. 在线付款

# 技术选型 #
前端: JSP(jstl, el) + Servlet(Filter, Listener)

后端: Apache BeanUtils(将请求数据封装到 javabean) + javamail(邮件发送) + Apache FileUpload

数据库: C3P0 + Apache DBUtils + 在线支付 + log4j日志技术

MVC 设计模式(Web项目整体架构) + DAO模式(数据层)

数据库: mysql

Jar包:
* jstl(jstl.jar, standard.jar)
* BeanUtils (beanutils.jar common-logging)
* FileUpload (fileupload.jar, io)
* Dbutils
* c3p0 (c3p0.jar)
* mysql驱动
* 日志(log4j)和javamail后期导入

# 目录开发结构 #

三层结构: 表现层(web) + 业务层 + 数据层

包名规范: 公司 + 项目组 + 项目代号 + 分层 `cn.itcast.estore.*`
* `cn.itcast.estore.web`表现层
* `cn.itcast.estore.service` 业务层
* `cn.itcast.estore.dao` 数据层
* `cn.itcast.estore.utils` 工具类
* `cn.itcast.estore.entity` 实体

# 数据库设计 #

Mysql 开发规则: 一个软件项目, 使用一个 database, 对应一个账户

## 新建数据库 ##
注意编码问题, 配置 my.ini(my.conf) 
~~~~~~
show variables like '%char%';
~~~~~~
System 系统字符集, 系统决定, 无法修改

Server 服务器字符集,修改 `[mysqld] character-set-server=utf-8`

`[mysql] default-charactor-set` 可以修改:
* Client 客户端输入字符集
* Connection 连接字符集
* Results 结果字符集

~~~~~~
create database estore;
-- 新建用户
create user angel@'localhost' identified by 'angel';
-- create user angel@'%' 任意连接都可以

grant all on estore.* to angel@'localhost';
~~~~~~

## 数据库设计 ##

Power Designer 数据库建模

传统数据库分析设计 E-R实体关系图

涉及到实体: 用户, 商品(类别), 购物车, 订单

三个实体表: 用户表, 订单表, 商品表

用户和订单关系: 在订单表中添加用户编号, 不需要单独建表

订单和商品关系: 中间表(订单号, 商品编号, ) 
![ER关系图](/img/er.png)


## 数据库建表 ##
建的表不一定和ER表中一样, 如为了代码功能增加, 增加字段(user邮箱激活码)
~~~~~~
create table user (
    id int auto_increment primary key,
    email varchar(255) not null unique,
    password varchar(255) not null,
    nickname varchar(255) not null,
    role varchar(255) not null,
    active int,
    activecode varchar(255)
)engine=innodb default charset=utf-8;

create table product (
    id int auto_increment primary key,
    name varchar(255) not null,
    category varchar(255) not null,
    markedprice double,
    estoreprice double,
    pnum int,
    imgurl varchar(255) not null,
    description varchar(255) not null
)engine=innodb default charset=utf-8;

create table orders(
    id varchar(255) primary key,
    totalprice double,
    receiverinfo varchar(255),
    paystate int,
    createtime timestamp,
    user_id int,
    constraint user_id_fk foreign key(user_id) references user(id)
)engine=innodb default charset=utf-8;

create orderitem(
   order_id varchar(255),
   product_id int,
   buynum int,
   constraint order_id_fk foreign key(order_id) references orders(id),
   constraint prodect_id_fk foreign key(product_id) references product(id)
)engine=innodb default charset=utf-8;
~~~~~~

# 建立实体类 #
~~~~~~
public class User {
    private int id;
    private String email;
    private String password;
    private String nickname;
    private String role;
    private int active;
    private String activecode;
}

public class Product {
    private int id;
    private String name;
    private String category;
    private int pnum;
    private double marketprice;
    private doublc estoreprice;
    private String imgurl;
    private String description;
}

public class Order{
    private String id;
    private double totalprice;
    private String receiverinfo;
    private int paystate;
    private Timestamp createtime;
    private Int user_id;
}

public class OrderItem {
    private String order_id;
    private int product_id;
    private int buynum
}
~~~~~~

# 用户注册功能实现 #
![注册页面](/img/register_ui.png)
重要知识点: 激活邮件功能

## JavaMail 技术入门 ##

### 邮件软件开发的概念### 
* 邮件服务器和电子邮箱

    必须有专门的计算机, 安装邮件服务程序  
    电子邮箱: 邮件服务器账户, 使用电子邮箱登录到邮件服务器
* 邮件传输协议

    SMTP协议: 发送邮件  
    POP3协议: 接收协议, 下载到本地操作  
    imap: 接收协议, 网络在线操作邮件, 很多邮件服务器不支持
* 邮件收发过程

    例如 aa@163.com 向 bb@sina.com 发送邮件
    1. aa 使用smtp协议登陆到163发送邮件服务器
    2. 发送邮件 163 smtp服务器将邮件投送到 新浪 smtp 服务器
    3. 新浪 smtp 服务器 将邮件保存到 bb 信箱
    4. bb使用 pop3 协议 登陆sina 收取邮件服务器
    5. sina 收取服务器 将邮件取出


可以用 易邮 在window上新建邮件服务器

### MX记录和A记录 ###

MX记录: 163 SMTP 服务器将信发送给 新浪 SMTP 服务器,
是如何知道 SMTP 服务器地址是什么?
解决 SMTP 服务器可以接受外来的邮件, 
~~~~~~
;; 查询域名
cmd> nslookup 连接到互联网
> set type=mx
> sina.com
~~~~~~

A记录: 个人用户, 使用 smtp 服务器发送邮件, 需要登陆 如果 163smtp 服务器
向 sina stmp 服务器投递邮件, 需不需要登陆? 存在于A记录中的服务器,
可以在向其他 stmp 服务器投递邮件时, 不需要登陆验证
~~~~~~
;; 查询A记录
cmd> nslookup 连接到互联网
> set type=a
> 163.com
~~~~~~

### JavaMail Demo ###

#### 描述一封邮件 ####
~~~~~~
// 使用javamail API 描述一份邮件内容
Session session = null; // 代表邮件服务器的一个连接会话
java.mail.Message msg = new MimeMessage(session);  // 一封邮件
// 使用message生成RFC822文档要求的邮件内容
msg.setFrom(new InternetAddress("aa@estore.com")); // 发件人
msg.setRecipient(RecipientType.TO, new InternetAddress("bb@estore.com")); // 设置收件人
msg.setSubject("测试邮件");
msg.setText("简单文字内容"); // or setContent();
// 定义复杂内容
msg.setContent("<a href='xxx'>激活</a>", "text/html;charset=utf-8");
~~~~~~

#### 发送一封邮件 ####

[更加详细的介绍](http://pringles.iteye.com/blog/125196)

JavaMail 核心类:
* Message 代表一封邮件
* Session 代表连接邮件服务器会话
* Transport 用于发送邮件
* Store 用于收取邮件

发送流程
1. 配置smtp邮件服务器, 连接参数
2. 编写邮件 Message
3. 使用Transport 发送邮件

~~~~~~
// 第一步配置 Session 
Properties p = new Properties();
p.setProperty("mail.smtp.host", "localhost");
p.setProperty("mail.transport.protocol", "smtp");
p.put("mail.smtp.auth","true"); // 如果是163需要验证, 要加上这句
Session session = Session.getInstance(p);

// 第二步, 编写邮件, 按照上一节代码

// 第三步: 连接服务器发送邮件
Transport transport = session.getTransport();
transport.connect("aaa", "111"); // 登陆,连接服务器
transport.sendMessage(msg, msg.getRecipient(RecipientType.TO));
~~~~~~


## 完成注册 ##
验证码技术原理: 在页面加载时, 服务器生成验证码,
将其保存在Session, 防止恶意提交

1. register.jsp 所有的表单元素, 添加 name 元素
2. 修改 form 的 Action 地址
3. 编写 表现层
~~~~~~
// cn.itcast.estore.servlet.RegistServlet.java
public void doGet(){
    // 判断密码是否一致
    String password = req.getParameter("password");
    String repassword = req.getParameter("repassword");
    if(!password.equals(repassword)){
        request.setAttribute("msg", "两次密码不一致");
        request.getRequestDispatcher("/regist.jsp").forward(req, res);
    }
    // 判断验证码是否正确
    String checkcode = req.getParameter("checkcode"); // 用户填写
    String key = req.getSession().getAttribute("key");
    if(key==null || !checkcode.equals(key)){
        request.setAttribute("msg", "验证码错误");
        request.getRequestDispatcher("/regist.jsp").forward(req, res);
    }
    req.getSession().removeAttribute("key");
    // 用户注册
    // 1. 将页面的数据封装到 User 对象, 采用BeanUtils
    User user = new User();
    BeanUtils.populate(user, req.getParameterMap());
    // 2. 将封装好的User对象传递给业务层处理
    UserService userService = new UserService();
    userService.regist(user);

    res.setContentType("text/html;charset=utf-8");
    res.getWrite().write("请到邮箱去激活");
}
~~~~~~
4. 编写业务层
~~~~~~
// cn.itcast.estore.service
public interface UserService {
    public void regist(User user);
}
// cn.itcast.estore.service.impl
public class UserServiceImpl implements UserService{
    public void regist(User user) {
        // 1. 生成激活码, 发送激活邮件
        String activeCode = UUID.randomUUID().toString();
        // 发送邮件, 请参照发送邮件章节
        Message msg = new MimeMessage(session);
        msg.setFrom(new InternetAddress("admin@estore.com"));
        // msg.setRecipient(); msg.setSubject();
        msg.setContent("<h2>欢迎注册</h2><a href='"+ activeCode+"' >激活</a>");
        // 2. 将用户信息保存到数据库
        user.setActivecode(activeCode);
        user.setActive(0);
        user.setRole("user"); //普通用户
        // 3. 调用数据层
        userDao.insert(user);
    }
}
~~~~~~
5. 编写数据库接口和实现类
~~~~~~
// cn.itcast.estore.dao
public interface UserDAO {
    public void insert(User user);
}
// cn.itcast.estore.dao.impl
public class UserDAOImpl {
    public void insert(User user){
        // 1. c3p0获取数据库连接池
        DataSource ds = new ComboPooledDataSource();
        // 2. 插入数据
        QueryRunner qs = new QueryRunner(ds);
        String sql = "insert into user values(null, ?, ?, ...)"
        // 用户的密码用密文保存
        qs.update(sql, MD5Utils.md5(user.getPassword()));
    }
}
~~~~~~

乱码问题处理: 请参照之前章节, 重写 `getParameterMap()` `getParameter()` `getValues()`

### 注册用户激活 ###
~~~~~~
// ActiveServlet
public void doGet(req, res){
    String activecode = req.getParameter("activecode");
    UserService userService = new UserServiceImpl();
    int result = userService.active(activecode);
    res.setContentType("text/html;charset=utf-8");
    if(result == Constants.ACTIVEMAIL_HASACTIVE){
        out.print("账户已经激活");
    } else if (result == Constants.ACTIVEMAIL_INVALIDATEACTIVCOD){
        out.println("激活码无效");
    } else if (result == Constants.ACTIVEMAIL_OK){
        out.println("激活成功");
    }
}
// UserService, 返回数字 
public int active(String activecode){
    // 判断激活码是由有效
    User user = userDAO.findByActiveCode(activecode);
    if(user == null) {
        // 激活码无效
        return Constants.ACTIVEMAIL_INVALIDATEACTIVCOD;
    } else {
        // 已经激活
        if(user.getActive()==1){
            return Constants.ACTIVEMAIL_HASACTIVE;
        } else {
            //需要激活
            userDao.updateActive(user.getId());
            return Constants.ACTIVEMAIL_OK;
        }
    }
}

// userDao
public User findByActiveCode(String activecode){
    DataSource ds = new ComboPooledDataSource();
    QueryRunner qr = new QueryRunner(ds);
    String sql = "select * from user where activecode=?";
    User user = qr.query(sql, new Beanhandler<User>(), activecode);
    return user;
}
public void updateActive(int id){
    DataSource ds = new ComboPooledDataSource();
    QueryRunner qr = new QueryRunner(ds);
    String sql = "update user set active = 1 where id=?";
    qr.update(sql, id);
}

// 存放操作常量
public class Constants {
    // 邮件激活的几种情况
    public static final int ACTIVEMAIL_TIMEOUT = 0;
    // 正常
    public static final int ACTIVEMAIL_OK = 1;
    // 已经激活过了
    public static final int ACTIVEMAIL_HASACTIVE = 2;
    // 无效
    public static final int ACTIVEMAIL_INVALIDATEACTIVCOD = 3;
}
~~~~~~

## 用户登陆功能实现 ##

~~~~~~
// servlet.LoginServlet
public class doGet(req, res){
    // 将请求的数据, 封装到user对象中
    User user = new User();
    BeanUtils.populate(user, request.getParameterMap());
    // 调用业务层, 进行登陆
    UserService userService = new UserServiceImpl();
    User loginUser = userService.login(User);
    // 判断登陆是否成功
    if(loginUser==null){
        // 失败
        req.setAttribute("msg", "用户名或者密码错误");
        req.getRequestDispatcher("/login.jsp").forward(req, res);
    } else {
        if(loginUser.getActive()==0){
            req.setAttribute("msg", "还没有激活");
            req.getRequestDispatcher("/login.jsp").forward(req, res);
        } else {
            request.getSession().setAttribute("user", loginUser);
            res.sendRedirect(req.getConentPath() + "/index.jsp");
        }
    }
}

// UserService
public User login(User user) {
    UserDao userDao = new UserDaoImpl();
    return userDao.login(user);
}

// UserDao
public User login(User user) {
    String sql = "select * from user where email=? and password=?";
    User loginUser = qr.query(sql, new Beanhandler<User>(), user.getEmail(), MD5Utiles.encode(user.getPassword()));
    return loginUser;
}
~~~~~~

## 注销功能 ##

~~~~~~
<!-- Invalidate.jsp -->
<%
    sesssion.invalidate();
    response.sendRedirect("login.jsp");
%>
~~~~~~

## 记住用户 ##
只发生在登陆成功时
~~~~~~
// LoginServlet doGet() 成功登陆后, 添加代码
if(req.getParameter("remember")!=null){
    Cookie cookie = new Cookie("email", loginUser.getEmail);
    cookie.setPath("/");
    cookie.setMaxAge(60*60*24); // 一天
    res.addCookie(cookie);
}

~~~~~~
