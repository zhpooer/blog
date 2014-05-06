title: 传智播客day20-ORM简介
date: 2014-04-26 09:28:23
tags:
- 传智播客
---
# ORM简介 #

ORM(Object Relation Mapping):
* Object : JavaBean对象
* Raltion: 关系型数据库
* Mapping: 映射(存在在着对应关系)

框架: Hibernate、MyBatis(iBatis)、JPA(java持久化框架, 规范)

简单的JDBC框架: 这些框架只是对JDBC操作的简单封装. 比如DBUtils\SpringJDBCTemplate

# DBUtils 框架的使用 #
Dbutils 是Apache提供的JDBC简单框架

* QueryRunner
* `int[] batch(String sql, Object[][] params)`: 执行批处理
* sql: 执行的语句
* params: 二维数组,高纬表示执行的语句的条数, 低维表示每条语句的参数
* 返回每条语句影响到的记录行数
* <T> T query(String sql, ResultSetHandler<T> rsh)
* 适合执行sql查询, 把结果封装到JavaBean中
* <T> T query(String sql, ResultSetHandler<T> rsh, Object... params)
* int update(String sql)
* 执行 insert update delete 语句
* int update(String sql, Object... params)
* int update(String sql, Object param)


## Bean封装 ##
~~~~~~
create table user(
    id int primary key,
    name varchar(20),
    birthday date
);
~~~~~~

~~~~~~
public class User{
    private int id;
    private String name;
    private Date birthday;
}
~~~~~~

~~~~~~
private QueryRunner qr = new QueryRunner(datasource);
public void testAdd(){
    qr.update("insert into user values(?, ?, ?)", 1, "aaa", new Date());
}
public void testUpdate(){
    qr.update("update user set name=? where id=?", "aaa", new Date(), 1);
}
public void testDel(){
    qr.update("delete from user where id=?", 1);
}
public void testQueryOne(){
    User u = qr.query("select * from user where id=1", new BeanHandler<User>(User.class), 1);
}
public void testQuery1(){
    User u = qr.query("select * from user", new BeanHandler<User>(User.class));
}
public void testMulti(){
    List<User> us = qr.query("select * from user", new ListBeanHandler<User>(User.class));
}
public void testBatch(){
    Object params[][] = new Object[10][];
    for(int i=0;i<params.length;i++){
        params[i] = new Object[]{i+1, "aaa" + ()i+1), new Date()}
    }
    qr.batch("insert into user values(?,?,?)", params);
}
~~~~~~

## DBUtils 中的结果处理器详解 ##
~~~~~~

// 把结果集的第一条记录封装到数组中, 数组中的元素就是每一列的值
public void testArrayHandler(){
    Object objs[] = qr.query("select * from user", new ArrayHandler());
}

// 把结果集中的每条记录封装到一个List中, 元素是Object[],
// 适合查询结果由多条的记录
public void testArrayListHandler(){
    List<Object[]> list = qr.query("select * from user", new ArrayListHandler());
}

//把查询结果中的某一列封装到List中
public void testColumnListHandler(){
    List<Object> list = qr.query("select * from user", new ColumnListHandler("name"));
}

// 把查询结果封装到一个Map中, Map中的key值是你指定的列值
// value 也是一个Map, map的key是字段名, value是字段值
public void testKeyedHandler(){
    Map<Object,Map<String,Object>> bmap = qr.query("select * from user", new KeyedHandler("id"));
}

// 把结果集中的第一条记录封装到Map中, key是字段名, value是字段值
public void testMapHandler(){
    Map<String, Object> map = qr.query("select * from user", new MapHandler());
}
// 每个元的代表一条记录, 用Map 封装, key是字段名, value是字段值
public void testMapListHandler(){
    List<Map<String, Object>> list = qr.query("select * from user", new MapListHandler);
}

// 单行, 单列
public void testScalarHandler(){
    Object list = qr.query("select count(*) from user", new ScalarHandler(1));
}
~~~~~~

# ThreadLocal #

1. ThreadLocal 提供了线程局部变量
2. 分析原理

ThreadLocal 内部有一个Map, Map的Key是当前线程对象, value是一个Objct对象. *模拟*:
~~~~~~
public class ThreadLocal<T>{
    private Map<Runnable, T> map = new HashMap<Runnable, T>();
    public void set(T t){
        map.put(Thread.currentThread(), t); // 把传入的参数绑定到当前线程上
    }
    public void remove(){
        map.remove(Thread.currentThread());
    }
    public T get(){
        return map.get(Thread.currentThread()); // 获取当前线程绑定的脆响
    }
}
~~~~~~
# 真实案例中的事务控制 #
* QueryRunner(DataSource conn) // 这个每条语句执行,都是不同的连接(Connection), 不能在同一个事务中执行语句
* QueryRunner()  //因为需要所有语句执行要在同一个事务中执行, 在执行语句时, 要传入(Connection)

> DAO层: 只负责数据库的访问, 只有增删改查

> 事务控制的要求: 一般是业务上的要求, 一个事务就是一个业务, **事务控制**不能在放在DAO

## 版本一 ##

### DAO层 ###
~~~~~~
public void findAccontByName(acconntname){
    Account account = qr.query("select * from account where name=?", new BeanHandler<Account>(Account.class), accountName);
}

public void updateAccount(Account account){
    qr.update("udpate set ", account.id, account.name)
}
~~~~~~

### 业务逻辑 ###
~~~~~~
Connection conn = dataSource.getConnection(); // 不好, 在业务层有Dao层 的代码
conn.setAutoCommit(false);
dao.setConnection(conn);
Account sourceAccount = dao.findAccontByName(sourceAccountName);
Account targetAccount = dao.findAccontByName(targetAccountName);

sourceAccount.setMoney(any);
targetAccount.setMoney(any);

dao.update(sourceAccount);
dao.update(targetAccount);
conn.setAutoCommit(true);
~~~~~~

## 版本二 ##
~~~~~~
public class TransactionManager{
    private static ThreadLocal<Connection> t1 = new ThreadLocal<Connection>();
    public static Connection(){
        Connecton conn = t1.get(); // 从当前线程上获取
        if(conn==null){
            conn = dataSource.getConnection(); // 没有就从池中取一个
            t1.set(conn);
        }
        return conn;
    }

    public static void startTransaction(){
        Connection conn = getConnection();
        conn.setAutoCommit(false);
    }

    public static void commit(){
        Connection conn = getConnection();
        conn.commit();
    }

    public static void rollback(){
        Connection conn = getConnection();
        conn.rollback();
    }
    public static void release(){
        Connection conn = getConnection();
        conn.close();
        t1.remove();  // 从当前线程解绑, 因为服务器使用了线程池技术
    }
} 
~~~~~~
### 业务逻辑 ###
~~~~~~
TransactionManager.startTransaction();
dao.setConnection(conn);
Account sourceAccount = dao.findAccontByName(sourceAccountName);
Account targetAccount = dao.findAccontByName(targetAccountName);

sourceAccount.setMoney(any);
targetAccount.setMoney(any);

dao.update(sourceAccount);
dao.update(targetAccount);
TransactionManager.commit();

TransactionManager.rollback(); // catch it
TransactionManager.release(); // finally
~~~~~~
### DAO层 ###
~~~~~~
public void findAccontByName(acconntname){
    Account account = qr.query(TransactionManager.getConnection(), "select * from account where name=?", new BeanHandler<Account>(Account.class), accountName);
}

public void updateAccount(Account account){
    qr.update(transactionmanager.getconnection(), "udpate set ", account.id, account.name)
}
~~~~~~

## 版本三 ##
提取 service 中的关于 Transaction 的重复代码, 使用**动态代理**

~~~~~~
public Interface BusinessService{
    public void transform(){}
}
~~~~~~

~~~~~~
public class BeanFactory {
// 返回BusinessService的实现类的代理对象
// 面向切面编程
    public static BusinessService getBusinessService(){
    BusinessService s = new BusinessService();
    BusinessService proxy = Proxy.newProxyInstancee(
        getClassLoader(), s.getClass().getInterfaces(),
        new InvocationHandler() {
            @Override public Object invoke() {
                TransactionManager.startTransaction();
                Object rtValue = method.invoke(s, args);
                    TransactionManager.commit();
                    TransactionManager.rollback(); // try catch
                    TransactionManager.release(); // finally
                    return rtValue;
            }
        });
    }
}
~~~~~~

# 多表存取 #

## 一对多 ##
一个客户可以有许多订单
### 模型类 ###
~~~~~~
create table customer(
    id int primary key,
    name varchar(100)
)
create table orders(
    id int primary key,
    num varchar(100),
    money float(8,0),
    customer_id int,
    constraint customer_id_ref foreign key (customer_id) referrence customer(id)
)
~~~~~~
~~~~~~
public class Customer {
    private int id;
    private String name;
    private List<Orders> orders = new ArrayList<Orders>
}
public class Orders {
    private int id;
    private String num;
    private float money;
    private Customer customer;
}
~~~~~~
### Dao ###
~~~~~~
public class CustomerDaoImpl{
    private QueryRunner rq;
    public void addCustomer(Customer c){
        rq.update("", c.getId(), c.getName());
        List<Orders> os = c.getOrders();
        if(os!=null&&os.size()>0){
            for(Orders o:os){
                os.update("", o.getId...);
            }
        }
    }

// 查询客户时,要不要查询客户拥有的订单? 看需求(Hibernate用的是延迟加载)
    public Customer findByCustomerId(int id){
        Customer c = rq.query("select * from customer where id=?", new BeanHandler<Customer>(), id);
        if(c!=null){
            // 有查询结果, 查询该客户的订单
            List<Orders> os = qr.query("select * from orders where customer_id=?"), new BeanListHander<Orders>, c.getId());
            c.setOrders(os);
        }
        return c;
    }
}
~~~~~~

~~~~~~
public class CustomerDaoTest {
    public void testAdd(){
        Customer c = new Customer();
        c.setId(1);
        c.setName("");

        Orders o1 = new Orders();
        o1.setId(1);
        o1.setNum("201401");
        o1.setMoney(1000);
        c.getOrders().add(o1);

        Orders o2 = new Orders();
        o2.setId(1);
        o2.setNum("201401");
        o2.setMoney(1000);
        c.getOrders().add(o2);
        dao.addCustomer(c);
    }

    public void testQuery(){
        Customer c = dao.findByCustomerId(1);
        List<Orders> os = c.getOrders();
    }
}
~~~~~~

## 多对多 ##
学生和老师的关系是多对多
~~~~~~
create table teacher (
    id int primary key,
    name varchar(100),
    salary float(8,0)
);
create table student(
    id int primary key,
    name varchar(100),
    grade varchar(100)
);
create table teacher_student(
    t_id int,
    s_id int,
    primary key(t_id, s_id),
    constraint t_id_fk foreign key(t_id) referrences teacher(id),
    constraint s_id_fk foreign key(s_id) referrences student(id)
);

~~~~~~

~~~~~~
public class Teacher {
    private int id;
    private String name;
    private float salary;
    private List<Student> studnents = new ArrayList();
}
public class Student{
    private int id;
    private String name;
    private String grade;
    private List<Teacher> teachers = new ArrayList();
}
~~~~~~

### Dao 
~~~~~~
public void addTeacher(){
    // 保存老师的基本信息
    qr.update("insert int teacher values(?,?,?)", t.getId(), t.getName());
    List<Student> sts = t.getStudnets();
    // 查看学生的信息在不在, 不在才保存
    if(students!=null&& students.size()>0){
        for(Student s:students){
            Student dbs = qr.query("select * from student where id=?", BeanHandler(), s.getId());
            if(dbs==null){
                qr.update("insert into student values(?,?,?)");
            }
            qr.update("insert into teacher_student values(?,?)", t.getId(), s.getId());
        }
    }
    // 同时保存老师和学生的关联关系
}
// 要不要查询老师教过的学员? 看需求
public void findTeacherById(int id){
    Teacher t = qr.query("select * from teacher where id=?", BeanHandler, teacher.getId);
    if(t!=null){
        // String sql = "select * from student where id in(select s_id from teacher_student where t_id=?)"; // 子查询
        // String sql = "select s.* from teacher_student ts, student s where s.id=ts.s.id and ts.t_id=?"; // 隐式内连接
        String sql = "select s.* from teacher_student inner join student s on s.id=ts.s_id where ts.t_id=?"; // 显式内连接
        List<Student> ss = qr.query(sql, new BeanListHander<Student>);
        t.setStudents(ss);
    }
}
~~~~~~

~~~~~~
public void testAddTeacher(){
    Teacher t1 = new Teacher();
    t1.setId(1);
    t1.setName("qb");
    t1.setSalary(10);

    Teacher t2 = new Teacher();
    t2.setId(1);
    t2.setName("wzt");
    t2.setSalary(10);

    Student s1 = new Student();
    s1.setId(1);
    s1.setName("hcH");
    s1.setGrade("A");

    t1.getStudents().add(s1);
    t2.getStudents().add(s1);
}

public void testQuery(){
    Teacher t = dao.findTeacherById(1);
}
~~~~~~

## 一对一 ##
有主键关联 和外键关联

身份证和公民是一对一

~~~~~~
create table person (
    id int primary key,
    name varchar(100)
);
create table idcard (
    id int primary key,
    num varchar(100),
    constraint i_id_fk foreign key(id) references person(id)
);
~~~~~~

~~~~~~
public class Person{
    private int id;
    private String name;
    private IdCard idCard;
}

public class IdCard {
    private ind id;
    private String num;
    private Person person;
}
~~~~~~
## DAO层 ##

~~~~~~
public void addPerson(Person p){
    qr.update("insert into person values(?,?)");
    IdCard idcard = p.getIdCard();
    if(idcard!=null){
        qr.update("insert into idcard values(?,?)")
    }
}
//要不要查对应的身份证, 看需求(建议查)
public Person findPersonById(int id) {
    Person p = qr.query("select * from person where id=?", new BeanHandler<Person>, id)
    if(p!=null){
        IdCard idCard = qr.query("select * from idcard where id=?", BeanHandler<IdCard),id)
        p.setIdCard(idcard);
    }
}
~~~~~~
