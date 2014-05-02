title: 传智播客day19-事务和数据库连接池
date: 2014-04-25 11:17:21
tags:
- 传智播客
---
# 事务特性 (AICD)#
* 原子性：处于同一个事务中的多条语句，要么全都成功，要么全都不成成功。
* 一致性：事务必须使数据库从一个一致性状态变换到另外一个一致性状态,在事务中的状态不会被保存. 比如转账：转账前a+b=2000，转账后a+b=2000
* 隔离性：多线程并发时，一个事务不能被其他事务所干扰。
* 持久性：数据应该被永久性的保存起来。（硬盘，而不是内存）

## 事务的隔离性专题 ##

如果不考虑事务的隔离性，会导致以下不正确的问题:
* 脏读: 指一个事务读到了另外一个事务中未提交的数据
* 不可重复读: 指一个事务读到了另外一个事务update后（事务提交了）的数据
* 虚读: 指在一个事务中读到了另外一个事务insert(事务提交了)的数据

数据库有四个隔离级别：
* READ UNCOMMITTED:脏读、不可重复读、虚读都有可能发生。
* READ COMMITTED:防止脏读发生；不可重复读、虚读都有可能发生。
* REPEATABLE READ:（MySQL默认级别）防止脏读、不可重复读；虚读有可能发生。
* SERIALIZABLE:防止脏读、不可重复读、虚读的发生

特点：从上到下，隔离级别越高，数据越安全，但是效率越低

~~~~~~
select @@tx_isolation;		查看当前数据库的隔离级别, 必须在事务中进行操作
set transaction isolation level 四个级别之一;更改当前事务的隔离级别
~~~~~~

## JDBC 控制事务隔离机制 ##
~~~~~~
Connection.TRANSACTION_READ_UNCOMMITTED
Connection.TRANSACTION_READ_COMMITTED
Connection.TRANSACTION_REPEATABLE_READ
Connection.TRANSACTION_SERIALIZABLE

Connection.setTransactionIsolation(int Level) // 必须放在事务开启之前

~~~~~~

# 数据库连接池 #
给数据库连接提供缓存, 提供访问效率

## 版本一 ##
~~~~~~
// cn.itcast.pool.SimpleConnectionPoll
public static SimpleConnectionPoll{
    private List<Connection> pool = new ArrayList<Connection>();
    private int size = 10;
    public SimpleConnectionPoll () {
        for(i <- 1 to size) {
            Connection conn = JDBCUtil.getConnection();
            pool.add(conn)
        }
    }
}
public synchronized Connection getConnection(){
    if(pool.size()>0){
        Connection conn = pool.remove(0);
        return conn;
    } else {
        throw new RuntimeException("服务器");
    }
}
public synchronized void release(Connection conn) {
    pool.add(conn);
}
~~~~~~

## 版本二 ##
~~~~~~
public MyDataSource implements javax.sql.DataSource{
    private static List<Connection> pool = new ArrayList<Connection>();
    static {
        for(i <- 1 to size) {
            Connection conn = JDBCUtil.getConnection();
            pool.add(conn)
        }
    }
    @Override
    public synchronized  Connection getConnection() throws SQLException{
        if(pool.size()>0){
            Connection conn = pool.remove(0);
            return conn;
        } else {
            throw new RuntimeException("服务器");
        }
    }
}
~~~~~~
DataSource 中没有提供 `release()`, 可以修改 `Conneciton` 的 `close()`方法,
让他不要关闭连接, 而是还回池中. 

## 方式三、四(装饰器, 适配器模式) ##

### 修改已知类的原有功能 ###
* 继承: 子类可以覆盖父类的方法(现在不合适)
* 装饰器模式:
  * 编写一个类, 实现与被包装类相同的接口(具有相同的行为)
  * 定义构造方法, 传入被包装类对象
  * 对于要改写的方法, 书写你的代码即可
  * 对于不要改写的方法, 调用原有类的方法
  ~~~~~~
  public class MyConnection implements java.sql.Connection{
      private Connection conn;
      private List<Connection> pool;
      public MyConnection(Connection conn, List<Connection> pool){
          this.conn = conn;
          this.pool = pool;
      }
      @Override
      public void close() throws SQLException {
         pool.add(conn);
      }
  }
  // DataSource.java
  public synchronized  Connection getConnection() throws SQLException{
      if(pool.size()>0){
          Connection conn = pool.remove(0);
          return new MyConnection(conn, pool);
      } else {
          throw new RuntimeException("服务器");
      }
  }
  ~~~~~~
* 适配器模式
~~~~~~
public class ConnectionAdapter implements Connection {
    private Conneciton conn;
    private List<Connection> pool;
    public ConnecitonAdapter(Connection conn, List<Connection> poo){
        this.conn = conn;
        this.pool = pool;
    }
    Override ...// 跟上一个方式的区别是, 没有重新实现close()方法
}
public class MyConnection extends ConnecitonAdapter {
    private Conneciton conn;
    private List<Connection> pool;
    public ConnecitonAdapter(Connection conn, List<Connection> pool){
        super(conn, pool)
        this.conn = conn;
        this.pool = pool;
    }
    @Override
    public void close() {
        pool.add(conn);
    }
}
~~~~~~
* 动态代理
`Proxy.newProxyInstancee(classloader, Class<?>[] interfaces, InvocationHandler)`
> inerfaces: 代理类实现了那些接口, 和被代理对象一样即可,(双方具有相同的行为)  
> InvacationHandler: 具体怎么代理(策略设计模式)

~~~~~~
new InvocationHandler(){
    // 匿名内部类, 实现接口
    /*
    * Object proxy 代理对象本身的丢向
    * Method mehtod, 当前调用的是哪个方法
    * Object[] args, 当前调用用到的参数
    */
    public Object invoke(Object proxy, Method m, Object[] args) {
        // before method handler
        // invoke method
        // after method handler
    }
}
~~~~~~
## 方式五 (动态代理)##
~~~~~~
public getConnection(){
    if(pool.size() > 0) {
        Connection conn = pool.remove(0);
        Connection proxyConn = (Conneciton) Proxy.newInstance(
            conn.getClass().getClassloader(),
            conn.getClass().getInterfaces(),
            new InvocationHandler(){
                 public Object invoke(){
                     if("close".equals(method.getName())){
                         return pool.add(conn)
                     } else{
                         method.invoke(conn, args);
                     }
                 }
            }
        );
        return proxyConn;
    }
}
~~~~~~

## 总结 ##
* 日后尽量使用标准的数据源(一般都带有连接池, 为的就是提高效率)
* 当用户调用`Connection.close()`方法, 并不能关闭连接, 而是还回池中

# 开源数据源 #
`DataSource`, 数据源, 数据源中都包含了数据库连接池的实现

## DBCP ##
简介: APache组织实现. Database Connect Pool

如何使用:
* 拷贝Jar包
* 配置 dbcpconfig.properties, 加入构建路径
~~~~~~
initialSize=10
maxActive=50
maxIdle=20
minIdle=5
maxWait=6000 # 单位毫秒
connectionProperties=useUnicode=true;
# 可以配置数据库数据隔离特性
~~~~~~
* 建立工具类
~~~~~~
public class DBCPUtil{
    private static DataSource dataSource;
    static {
        // 读取配置文件
        InputStream in = DBCPUtil.class.getClassloader().getResourceAsStream('properties');
        Properties props = new Properties();
        props.load(in);
        dataSource = BasicDataSourceFactory.createDataSource(props);
    }
    public static Conneciton getConnection(){
        return dataSource.getConnection();
    }
    public static getDataSource(){
        return dataSource;
    }
}
~~~~~~

## C3P0 ##

* 拷贝Jar包
* 手工配置C3p0
~~~~~~
private static ComboPooledDataSource ds = new ComboPooledDataSource();
ds.setDriverClass("com.mysql.jdbc.Driver");
ds.setJdbcUrl("jdbc:mysql://localhost:3306/database");
ds.setUser("");
ds.setPassword("");
ds.setMaxPoolSize(40);
ds.setMinPoolSize(5);
ds.setInitialPoolSize(20);
ds.setMaxStatements(); // 批处理
~~~~~~
* 可以用xml配置`c3p0-config.xml`, 或`c3p0.properties`来配置

## 获取Tomcat管理的数据源 ##

Tomcat在启动时,会按照用户的配置, 创建数据源, 并把数据源对象绑定到一个名字上(用的是JNDI).

JNDI: Java Naming and Directory Interface(JDK:javax.naming.*)

相当于window系统的注册表, 是一个Map结构. key是一个由路径和名称组成的字符串,
value就是一个绑定的对象

### 利用Tomcat管理数据源(dbcp) ###
1. 数据库连接驱动Jar包, 放到Tomcat的lib文件夹下
2. 配置Context, 在应用目录里面的 META-INFO 建立 context.xml
~~~~~~
<context>
    <resources name="jdbc/day19" auth="Container"
    type="javax.sql.DataSource" username="root"
    password="sorry" driverClassName="com.mysql.jdbc.Driver"
    uri="jdbc:mysql" maxActive="8" maxIdle="4">
    </resources>
</context>
~~~~~~
3. 启动Tomcat, 数据源就已经建好了
4. 在应用中获取数据源, 获取JNDI容器中的数据源
~~~~~~
Context initCtx = new InitialContext();
Context envCtx = initCtx.lookup("java:comp/env");
Datasource ds = envCtx.lookup("jdbc/dya19");
Conneciton conn = ds.getConnection();
~~~~~~

# 数据库元数据的获取 #
元数据: 数据库, 表, 列等定义的信息
~~~~~~
DatabaseMetaData dmd = conn.getMetaData();
// 数据库综合信息获取
dmd.getDatabaseProductName();   // MySQL
dmd.getDatabaseProductVersion();   // 5.0.18

// 获取 preparedStateMent 中占位符的语句信息
PreparedStateMent stmp = conn.prepareStateMent("insert into user values(?,?,?)")
ParameterMetaData pmd = stmp.getParameterMetaData; // 得到的是问号的个数
pmd.getParameterCont();    // 3

// 结果集元数据信息获取
PreparedStateMent stmp = conn.prepareStateMent("select * from user")
ResultSet rs = stmp.excuteQuery();
ResultSetMetaData rsmd = rs.getMetaData();
int num = rsmd.getColumnCount();    // 查询结果有多少列
for(int i=0;i<num;i++){
    String columnName = rsmd.getColumnName(i+1);
    int columnType =  rsmd.getColumnType(i+1);  // java.sql.Types
}
~~~~~~

# 自定义框架 #

~~~~~~
public class User{
    private int id;
    private int String ;
    private String gender;
}
~~~~~~

~~~~~~
public class UserDaoImpl{
    public void addUser(User user){
        db.update("insert into user values(?,?,?))", new Object{user.getId,...})
    }
    public void updateUser(User user){
        db.update("update into user set name=?, gender=? where id=?", new Object{"abc", "0", user.getId()})
    }
    public void delUser(User user){
        db.update("delete from user where id=?", new Object{user.getId()});
    }
    public Object query(){
        db.query("select * from user where id=?", new Object[]{1},
        // 可提取一个方法类
            new ResultSetHandler(){
                public Object handle(ResultSet rs){
                    if(rs.next()){
                        ResultSetMetaData rsmd = rs.getMetaData();
                        int columnCount = rsmd.getColumnCount();
                        User user = new User();
                        for(int i=0;i<colmnCount;i++){
                            String fieldName = rsmd.getColumnName(i+1);
                            String fieldValue = rs.getObject(i+1);
                            Field field = User.class.getDeclaredField(fieldName);
                            field.setAccessible(true);
                            field.set(user, fieldValue);
                        }
                    } else {
                        return null;
                    }
                }
            }
        );
    }
}
~~~~~~

~~~~~~
// 结果处理策略
public Interface ResultSetHandler {
    Object handler(ResultSet rs);
}
public class DBAssist{
    private DataSource ds;
    public DBAssist(Datasource ds){
        this.ds = ds;
    }
    // 执行DML语句: insert update delete
    public void update(String sql, Object[] params){
        Conneciton conn = null;
        PreparedStateMent stmt = null;
        ResultSet rs = null;
        try{
            conn = dataSource.getConnection();
            stmt = conn.prepareStateMent(sql);
            ParameterMetaData pmd = stmt.getParameterMetaData();
            int paramCount = pmd.getParameterCount();
            if(paramCount>0){
                if(params==null || params.length==0){
                    throw new RuntimeException("参数有误");
                }
                if(paramCount!=params.length){
                    throw new RuntimeException("参数个数不匹配");
                }
                for(int i=0; i<paramCount;i++){
                    stmt.setObject(i+1, param[i])
                }
                stmt.excuteUpdate();
            }
        }catch(e){
        }finally{
            release(rs,stmt,conn);
        }
    }
    //使用前提 javabean中的字段名和数据库中的列名必须一致
    public Object query(String sql, Object[] param, ResultSetHandler h){
        Conneciton conn = null;
        PreparedStateMent stmt = null;
        ResultSet rs = null;
        try{
            conn = dataSource.getConnection();
            stmt = conn.prepareStateMent(sql);
            ParameterMetaData pmd = stmt.getParameterMetaData();
            int paramCount = pmd.getParameterCount();
            if(paramCount>0){
                if(params==null || params.length==0){
                    throw new RuntimeException("参数有误");
                }
            }
            if(paramCount!=params.length){
                throw new RuntimeException("参数个数不匹配");
            }
            for(int i=0; i<paramCount;i++){
                stmt.setObject(i+1, param[i])
            }
            rs = stmt.excuteQuery();
            h.handler(rs);
        }catch(e){
        }finally{
            release(rs,stmt,conn);
        }
    }
}
~~~~~~
