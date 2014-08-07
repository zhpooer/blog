title: 传智播客day17-jdbc
date: 2014-04-22 09:29:44
tags:
- 传智播客
- jdbc
- 设计模式
---

# jdbc概述 #
sun 公司为了简化和统一数据库的操作, 定义了JDBC规范
1. Java Data Base Connective
2. 组成包: java.sql.\*, javax.sql.\*, 都包含在JDK
3. 还需要数据库的驱动, 这些驱动就相当于对JDBC规范的实现

# jdbc 编码步骤 #
准备数据:
~~~~~~
create table users(
   id int primary key auto_increment,
   name varchar(100),
   password varchar(40),
   email varchar(60),
   birthday date
);
insert into users(name,password,email,birthday) values('ab','123','ab@c.com','1980-02-22');
insert into users(name,password,email,birthday) values('cd','123','cd@e.com','1981-02-22');
insert into users(name,password,email,birthday) values('ef','123','ef@g.com','1982-02-22');
insert into users(name,password,email,birthday) values('gh','123','gh@k.com','1983-02-22');
~~~~~~
1. 注册驱动
2. 获取与数据库的连接
3. 得到代表数据库的语句
4. 执行语句
5. 如果执行的是查询语句, 就会有结果集, 处理
6. 释放占用的资源


~~~~~~
DriverManager.registerDriver(new com.mysql.jdbc.Driver);
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test?characterEncoding=utf-8", "root", "root");
Statement stmt = conn.createStatement();
// resultset 是一个游标, 该游标可以滚动, 默认指向第一行的前面
ResultSet rs = stmt.excuteQuery("select * from user");
// hs.next(); 指向移动下一条游标, 并告知有没有记录
while(rs.next()) {
   println(rs.getObject("id"));
   println(rs.getObject("name"));
   println(rs.getObject("password"));
   println(rs.getObject("email"));
   println(rs.getObject("birthday"));
}
rs.close();
stmt.close();
conn.close();
~~~~~~

# JDBC 主要接口或类 #

## DriverManager ##
作用:
* 注册驱动
~~~~~~
// 有缺陷, 会导致驱动注册两遍
DriverManager.registerDriver(new com.mysql.jdbc.Driver)
// 替代方案: 避免了依赖具体的驱动类
Class.fornName("com.mysql.jdbc.Driver")
~~~~~~
* 获取与数据库的连接
  1. 方式一: 
`DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");`
sun公司和数据库厂商定义的协议, 具体参考数据库的文档
  2. 方式二
~~~~~~
Properties info = new Properties();
info.setProperty("user", "root"); // key由什么定义,查数据库文档查
info.setProperty("password", "root");
info.setProperty("useUnicode", "true");
info.setProperty("characterEncoding", "utf8");
DriverManager.getConnection("jdbc:mysql://localhost:3306/test", info)
~~~~~~
  3. 方式三: `DriverManager.getConnection("jdbc:mysql://localhost:3306/test?user=root&password=root&...")`
  

## Connection ##
所有的与数据库的交互都是基于连接的基础之上的, 因此该对象相当重要

`Statement stmt = conn.createStatement()` 创建向数据库发送sql的`Statement`对象

## Statement ##
|  方法  |  详细信息 |
|------------------------|
| ResultSet excuteQuery(String, sql)| 只适合查询, 返回的是查询的结果集|
| int excuteUpdate(String sql) | 适合(insert update delete), 或者没有返回结果集的DDL, 返回影响的记录数目 |
| boolean excute(String sql) | 适合所有语句, 返回*有没有结果集*,  `if(true) stmt.getResultSet()` |


## ResultSet ##
代表着查询语句的查询结果集, 有一个游标
### 遍历结果集 ###
~~~~~~
// 方式一
while(rs.next()) {
   println(rs.getObject("id")); 
   println(rs.getObject("name"));
   println(rs.getObject("password"));
   println(rs.getObject("email"));
   println(rs.getObject("birthday"));
}
// 方式二 可以通过列的索引获取数据, 第一列索引是1,与框架有关
while(rs.next()) {
   println(rs.getObject(1));
   println(rs.getObject(2));
   println(rs.getObject(3));
   println(rs.getObject(4));
   println(rs.getObject(5));
}
~~~~~~

### 常用方法 ###
|   方法        |   描述     | 
| -----------------------------| 
| boolean next() | 向后移动游标, 返回该位置上有没有记录 | 
| boolean previous() | 向前移动游标, 返回该位置上有没有记录 | 
| boolean absolute(int)    | 绝对定位(第一行 1), 返回该位置上有没有记录 | 
| void beforeFirst()   | 游标定位在第一行的前面 | 
| void afterLast() | 游标定位在最后一行的后面 | 

### mysql&jdbc 类型对应 ###
| mysql类 | Jdbc对应方法  | 返回类型 |
|-----------------------------------------------------|
| Bit(1)           | getBoolean() getBytes() | Boolean byte[] |
| tinyint          | getByte()               | Byte |
|smallint          | getShort()              | Short |
| Int              | getInt()                | Int |
| bigint           | getLong()               | Long |
| char varchar     | getString()             | String |
| Text(cblob) Blob | getClob() getBlob()     | Clob Blob |
| date             | getDate()               | java.sql.Date |
| time             | getTime()               | java.sql.Time |
| timestamp        | getTimestamp()          | java.sql.Timestamp |


### 封装数据到 Javabean ###
~~~~~~
public class Users implements Serializable {
    private int id;
    private String password;
    private String name;
    private String email;
    private String birthday;
}
u.setId(rs.getInt("id"));
u.setName(rs.getString("name"));
u.setBirthday(rs.getDate("birthday"));
~~~~~~

## 释放占用的资源 ##
按照打开的顺序, 以相反的顺序进行释放

特别是 Connection 对象, 非常稀有的资源, 用完后必须马上释放,
**必须尽量晚创建和尽量早释放**

~~~~~~
Connection conn = null;
Statement = null;
ResultSet rs = null;
try {
} catch (Exception e){
} finally {
   try{if(rs!=null)   { rs.close(); rs = null;     }} catch (e){}
   try{if(stmt!=null) { stmt.close(); stmt = null; }} catch (e){}
   try{if(conn!=null) { conn.close(); conn = null; }} catch (e){}
}
~~~~~~

## 模板 ##
dbcfg.properties
~~~~~~
driverClass=**
url=**
usr=**
password=**
~~~~~~

~~~~~~
public class JdbcUtil {
    private static String driveClass;
    private static String url;
    private static String user;
    private static String password
    static {
      InputStream in = getClasLoader.getResourceAsStream("dbcfg.properties");
      Properties props = new Properties();
      props.load(in)
      String drClass = props.getProperty("driveClass");
      String url = props.getProperty("url");
      String user = props.getProperty("usr");
      String password = props.getProperty("password");
      Class.forName(drClass);
    }
    public static Connection getConnection() throws SQLException {
         return DriverManager.getConnection(url,user,password);
    }
    public static void release(ResultSet rs, Statement stmt, Connection conn) {
        try{if(rs!=null)   { rs.close(); rs = null;     }} catch (e){}
        try{if(stmt!=null) { stmt.close(); stmt = null; }} catch (e){}
        try{if(conn!=null) { conn.close(); conn = null; }} catch (e){}
    }
}
~~~~~~


# JDBC进行CRUD #

~~~~~~
@Test
public void testAdd(){
    Connection conn = null;
    Statement stmt = null;
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.createStatement();
        String query = "insert into users(name, password, email, birthday) values(''...)";
        stmt.excuteUpdate(query);
    }catch(e) {
        JdbcUtil.release(null, stmt, conn);
    }
}
@Test
public void testUpdate(){
    Connection conn = null;
    Statement stmt = null;
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.createStatement();
        String query = "udpate users set password='123' where id = 4";
        stmt.excuteUpdate(query);
    }catch(e) {
        JdbcUtil.release(null, stmt, conn);
    }
}
@Test
public void testDelete(){
    Connection conn = null;
    Statement stmt = null;
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.createStatement();
        String query = "delete from users where id = 4";
        stmt.excuteUpdate(query);
    }catch(e) {
        JdbcUtil.release(null, stmt, conn);
    }
}
~~~~~~

# 利用JDBC修改web项目 #
~~~~~~
ResourceBundle rb = ResourceBundle.getBundle("dbcfg");
try{
    Class.forName()
} catch (e) {
    throw new ExceptionInInitializerError("读取数据库配置文件失败.");
}
~~~~~~
~~~~~~
public class UserDaoMySQLImpl implements UserDao {
    public void save(User user) {
        if(findUserByUsername(user.getUsername) != null){
           throw new UserExistException();
        }
        Connection conn = null;
        Statement stmt = null;
        try{
            conn = JdbcUtil.getConnection();
            stmt = conn.createStatement();
            stmt.excuteUpdate("");
        } catch (Exception e) {
            throw new RuntimeException();
        } finally {
            JdbcUtil.release(null, stmt, conn)
        }
    }
    public User findUserByUsername(String username) {
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        try{
            conn = JdbcUtil.getConnection();
            stmt = conn.createStatement();
            rs = stmt.excuteQuery("");
            if(rs.next()) {
                Usesr user = new User();
                user.setUsername(rs.getString("username"));
                return user;
            }
            return null;
        } catch (Exception e) {
            throw new RuntimeException();
        } finally {
            JdbcUtil.release(null, stmt, conn);
        }
    }
    public User findUser(String username, String password) {
        // 同 findUserByName
    }
    // 注意单元测试
}
~~~~~~

# PreparedStatement #

特点和作用
* 支持SQL语句的预编译, 提高数据库的执行效率
* 防止SQL注入, 给数据库的已经不是字符串了
* 语句中的参数可以使用占位符(?)

**能用 PreparedStatement, 都要用 PreparedStatement**
~~~~~~
stmt = conn.preparedStatement("insert into user (username,password,email,birthday) values(?,?,?,?)");
stmt.setString(1, user.getUsername()); // 索引对应的语句中的?的位置, 第一个就是1
stmt.setDate(4, new java.sql.Date(user.getBirthday().getTime()));
stmt.excuteUpdate(); // or rs = stmt.excuteQuery();
~~~~~~

# 理解DAO解耦的好处 #
高类聚, 低耦合
~~~~~~
// 工厂模式 和 单例 
public class DaoFactory {
    private static DaoFactory instance;
    private DaoFactory(){}
    public static DaoFactory getInstance(){
        if(instance==null){
            synchronized(DaoFactory.getClass()){
                 if(instance==null)
                      instance = new DaoFactory();
                  return instance;
            }
        }
        return instance;
    }
    public UserDao getUserDao(){
        return Class.forName("").newInstance();
    }
}
~~~~~~
