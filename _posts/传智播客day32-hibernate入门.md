title: 传智播客day32-Hibernate入门
date: 2014-05-20 10:45:34
tags:
- 传智播客
- Hibernate
---


# jdbc的优缺点 #
缺点
* 查询代码繁琐
* 重复性代码特别多(频繁的try catch)
* 没有数据缓存
* sql的移植性不好

优点
* 速度快
* 把控性比较好

# OR mapping 优缺点 #
优点
* 简单
* 数据缓存, 一级缓存 二级缓存 查询缓存
* 可移植性比较好

缺点
* 因为sql语句是内部生成的, 程序不可控
* 如果数据量特别大, 程序不可控

# 流行数据库框架 #
* JPA Java Persistence API  通过注解描述对象与数据表映射关系(只有接口规范)
* Hibernate 最流行ORM框架,通过对象-关系映射配置,可以完全脱离底层SQL, Hibernate实现JPA规范 
* MyBatis  本是apache的一个开源项目 iBatit, 支持普通 SQL查询, 存储过程和高级映射的优秀持久层框架(企业主流)
  * MyBaits 并不是完全ORM,  需要在xml中配置SQL语句
* Apache DBUtils, Spring JDBCTemplate


# ORmapping 概念 和 Hibernate #
对象关系映射: 让数据库和对象发生关联, 通过操作对象, 完成对数据库的操作

通过 `*.hbm.xml`(映射文件),
* 类与表的对应关系
* 类上属性的名称和表中的字段的名称的对应关系
* 类上属性的*类型*和表中的字段的*类型*的对应关系
*  把一对多和多对多的关系转化成面向对象的文件

Hibernate的配置文件: 用来连接数据库

# hibernate入门案例 保存对象 #
Hibernate: 根据客户端代码, 参照映射文件, 生成sql语句,
利用jdbc即使进行数据库操作


## 编写持久化类 ##
POJO: 持久化类
~~~~~~
/*
* 属性不能使用 数据库的关键字 如`table`
* 对象序列化作用, 让对象在网络上传输, 以二进制形式传输
*/
public class Person implements Serializable {
    @BeanProperty private long pid;
    @BeanProperty pirvate String pname;
    @BeanProperty pirvate String psex;
    // 必须要写默认的构造函数, hibernate 查询对象时
    public Person(){}
}
~~~~~~
## 准备映射文件 ##
`Person.hbm.xml` 类映射文件
~~~~~~
<!-- import hibernate-mapping-3.0.dtd-->
<hibernate-mapping>
    <!-- 用来描述一个持久化类  -->
    <!-- name 类的全名 -->
    <!-- table 可以不写, 默认值和类名一样 -->
    <!-- catalog 数据库名称, 一般不写 -->
    <class name="cn.itcast.Person">
        <!-- name 属性名称, 是根据字段, 而不是属性(get, set)-->
        <!-- column 列的名称 -->
        <id name="pid" length="5" type="java.lang.Long">
            <!-- 主键产生器, 告诉hibernate容器用什么样的方式产生主键 -->
            <gernerator class="increment"> </gernerator>
        </id>
        <property name="pname" column="pname" length="20" type="java.lang.String"> </property>
        <property name="psex" column="psex" >
            <column name="psex" sql-type="varchar(20)"> </column>
        </property>
    </class>
</hibernate-mapping>
~~~~~~
## 准备配置文件 ##
`hibernate.cfg.xml`, 如果是默认加载, 必须放在根目录下
~~~~~~
<!-- import hibernate-configuration-3.0.dtd -->
<Hibernate-configuration>
    <session-factory>
        <!-- 还有其他练级信息 -->
        <!-- 数据库连接信息 -->
        <!-- 可以省略 hibernate.  -->
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql:///database</property>
        <property name="hibernate.connection.username"></property>
        <property name="hibernate.connection.password"></property>
        <!-- 数据库方言 -->
        <property name="hibernate.dialect"> org.hibernate.dialect.MySQLDialect </property>
        <!-- 根据持久化类和映射文件生成表
            validate 只验证, 不创建表, 如果不一致 就抛出异常
            create-drop 启动创建, 结束删除
            create 每次创建
            update 检查, 如果没有就创建
        -->
        <property name="hbm2ddl.auto">update</property>
        <!-- 格式化输出sql语句 -->
        <property name="format_sql">true</property>
        <!-- 显示 hibernate 内部生成的sql语句, 默认false -->
        <property name="show_sql">true </property>
        <!-- 没有开事务的情况下事务是否自动处理 -->
        <property name="hibernate.connection.autocommit">true</property>
        <!-- 导入映射文件 -->
        <mapping resource="cn/itcast/hibernate/sh/domain/Person.hbm.xml" />
    </session-factory>
</Hibernate-configuration>
~~~~~~

## 测试运行 ##
~~~~~~
Configuration  conf = new Configuration();
conf.configure();  // 加载默认配置文件 `/hibernate.cfg.xml`
SessionFactory fac = configuration.buildSessionFactory();
Session sessioin = fac.openSession();
Transaction trans = session.beginTransaction();
Person p = new Person();
p.setPname("p");
p.setPsex("p");
session.save(p);
trans.commit();
session.close();
~~~~~~

# Hibernate 核心API #
1. Configuration
2. SessionFactory
3. Session, 是一个单线程对象, 线程不安全
~~~~~~
session.save();
session.update();
session.delete();
// 根据主键查询
session.get/load();
// 创建查询对象, Query 接受 SQL, sqlQuery 接收 SQL
session.createQeury();
session.createSQLQuery();
// 面向对象查询
session.createCriteria();
~~~~~~
4. Transaction
~~~~~~
// 如果没有开启事务, 那么每个Session的操作, 都相当于一个独立的事务
// 开启事务
sessoin.beginTransaction()
// 提交事务
transaction.commit();
// 回滚
transaction.rollback();
transaction.wasCommitted();
~~~~~~
5. Query, 代表面向对象的Hibernate查询
~~~~~~
// 分页查询
Query query = session.createQeury("form Person");
// 11-30
query.setFirstResult(10);
query.setMaxResults(20);
~~~~~~
6. Criteria, 主要为了解决多条件查询问题，以面向对象的方式添加条件，无需拼接HQL语句 
~~~~~~
Criteria criteria = session.createCriteria(Customer.class);

// 分页
criteria.setFirstResult(1);
criteria.setMaxResults(2);

// 条件查询
criteria.add(Restrictions.eq("name", "李四"));
criteria.add(Restrictions.lt("age", 20));
criteria.add(Restrictions.like("name", "李%"));

criteria.list();
~~~~~~

# hiberante 加载流程分析 #
~~~~~~
// 创建 Configuration 对象
Configuration  conf = new Configuration();
// 包括 数据库连接信息 和 加载映射文件
conf.configure();  // 或 conf.configure("myownconfig");
// 手动加载hbm
conf.addResource("Person.hbm.xml"); // conf.addResource(Person.class); 自动搜索
// 数据库链接信息, 和映射文件信息, 持久化类信息 封装在 sessionFactory, 是一个连接池
// 是一个单例模式,一般情况下 hibernate 应该只有一个数据库连接且线程安全
SessionFactory fac = configuration.buildSessionFactory();
// 打开一个数据库连接, 进行数据库操作
Session sessioin = fac.openSession();
// 如果进行查询, 不用开启事务
// 如果进行cud操作, 要开启事务, 事务是由session开启的
// 事务不是自动提交的, 必须由session开启, 和当前session绑定
Transaction trans = session.beginTransaction();
Person p = new Person();
p.setPname("p");
p.setPsex("p");
//  改变对象状态
// 会根据映射文件, 生成sql语句
session.save(p);
// 执行sql语句, 提交事务
trans.commit();
// 关闭数据连接
session.close();
sessionFactory.close(); // 关闭连接池
~~~~~~
得到一个持久化类
  * 加载配置文件
  * 在配置文件中加载类映射文件
  * 解析 映射文件中的class标签 name 属性, 找到对应类

hibernate错误概览
* `Unknown entity`, 在映射文件中没有配置该类
* `PropertyNotFound`, 映射文件中类字段错误, `property name` 写错
* `resource not found`, 映射文件名没有找到
* `entity class not found`, 在配置文件中导入文件映射文件错误, `class name`写错

# 配置c3p0连接池 #
1. 导入 c3p0-0.9.1.jar
2. 在hibernate.cfg.xml 修改连接提供者
~~~~~~
<property name="connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</property>
~~~~~~
3. 配置c3p0连接池属性 
~~~~~~
<property name="c3p0.min_size">5</property>
<property name="c3p0.max_size">20</property>
<property name="c3p0.timeout">120</property>
<property name="c3p0.idle_test_period">3000</property>
~~~~~~

# CRUD 的操作 #
~~~~~~
public void testQueryPerson(){
    Session session = sessionFactory.openSession();
    List personList = session.createQuery("from Person").list();
    session.close();
}
public void testQueryPersonById() {
    Session session = sessionFactory.openSession();
    // 按照主键的方式查询数据库表中的记录
    // session.get(Person.class, 1); 会报错, 需要 long 类型
    session.get(Person.class, 1L);
    session.close();
}
public void testQuerySQL(){
    String sql = "select * from person";
    SQLQuery sqlQuery = session.createSQLQuery(sql);
    sqlQuery.addEntity(Person.class);
    sqlQuery.list();
}
public void testDeletePerson(){

    Session session = sessionFactory.openSession();
    Transaction transaction = session.beginTransaction();
    // 方式一: 
    Person person = session.get(Person.class, 1L);
    session.delete(person);
    
    // 方式二
    Person person = new Person();
    // hibernate 内部会检查标示符(id), 看标示符的值在数据库响应的表中有没有对应的行
    // 和其他属性不相关
    person.setId(1L);
    session.delete(person);
    // 如果删除一个不存在的对象, 会报错: SaleStateException, unexpect row
    transaction.commit();
    session.close();
}
public void testUpdatePerson(){
    Session session = sessionFactory.openSession();
    Transaction transaction = session.beginTransaction();
    Person person = session.get(Person.class, 1L);
    person.setPsex("女");
    session.update(person);
    transaction.commit();
    session.close();
}
public void testIdentity(){
    Session session = sessoinFactory.openSession();
    Transaction trans = session.beginTransaction();
    Person person1 = session.get(Person.class, 1L);
    Person person2 = new Person();
    person.setPid(1L);
    session.update(person2); // 报错, 因为Hibernate不允许两个持久化对象持有同一个 标示符
    transaction.commit();
    session.close();
}
~~~~~~
# 类型匹配 #
在Hibernate映射文件中type属性支持两种类型, 一种是 java类型, 另一种hibernate类型,
java类型效率更高

以后开发中，PO类属性 都使用包装类型 
* 使用基本类型 ，无法区分 0 和 null,
使用int类型分数，如果学生分数为0 可以没有考试, 也可能考试得了0分
* 使用包装类型, 如果不设置数据, 数据表存放null, 而不是默认值 0	


| hibernate类型 | java类型->数据库类型 |
|-----------|
| integer, long, short, float, double, character, byte, boolean, yes_no, true_false | java 原始类型到sql的字段类型 |
| string | java.lang.String 到 VARCHAR(或者 Oracle 的 VARCHAR2) |
| date, time, timestamp | java.util.Date 和其子类到 SQL 类型 DATE，TIME 和 TIMESTAMP（或等价类型）的映射 |

# 产生器(Gernerator) #
~~~~~~
<gernerator class="increment"> </gernerator>
~~~~~~
## increment ##
* 主键类型必须是数字
* 主键生成是由hibernate生成的, 程序员不需要干预
* 效率相对低, 且多线程会冲突

~~~~~~
// 没有意义, hibernate是根据主键生成器来生成主键的,
/// 在客户端设置主键不一定起作用. 
Person p = new Person();
p.setId(1L);   
session.save(p);
~~~~~~

## identity ##
* 生成的主键是由数据库完成的
* 该表必须支持自动增长机制(auto_increment)
* 效率高
* mysql支持自动增长, oracle不支持自动增长

## assigned ##
自然主键的生成策略, 由程序(手动)设置主键

## sequence ##
标识符生成器利用底层数据库提供的序列来生成标识符

原理: 依赖数据库序列支持, 和hibernate程序无关
* Oracle 支持序列, Mysql 不支持序列
* 序列原理
~~~~~~
create sequence customer_seq;
insert into customer(id) values(customer_seq.nextval); // 自动序列+1 
~~~~~~

## native ##
native 标识符生成器依据底层数据库对自动生成标识符的支持能力,
来选择使用 identity, sequence 或 hilo 标识符生成器.
* Mysql 自动选择 identity ， oracle 自动选择 sequence

## uuid ##
* UUID是由hibernate内部生成
* 主键的类型必须是字符串

## 复合主键 ##
复合主键(联合主键)，一个数据表中多列共同作为主键, 复合主键是一种特殊 assigned 策略
~~~~~~
<composite-id>
    <!-- 配置多列 -->
	<key-property name="firstname"></key-property>
	<key-property name="secondname"></key-property>
</composite-id>
~~~~~~

**复合主键类必须实现序列化接口**

# 对象的状态 #
Hibernate中的对象共有三个状态: 临时(瞬时, transient), 持久化(persist), 托管状态(detach)

持久化对象不允许随意修改OID

~~~~~~
// 临时状态对象,
// 不存在持久化标识OID, 尚未与hibernate session关联
Person p = new Person();
// 持久化状态, 只有在持久化状态, 对象的状态变化才会被Hibernate捕捉
session.save(p);
transaction.commit();
// 脱管状态, 存在OID, 未与session关联
session.close();
~~~~~~

把一个临时状态对象, 变成持久化状态的方法:
* `Session.save`
* `Session.update`

当`Session.get`方法得到一个对象的时候, 是不需要再执行`Session.update`语句的,
因为已经是持久化状态.

当一个对象是持久化对象时, 当进行提交时, 会根据内存中*快照*看是否需要执行update语句.

## 对象状态变化 ##
![hibernate状态转换](/img/hibernate_state.png)

`session.update()`, 脱管对象更新, 变成持久化对象
* 调用该方法, 默认会直接生成 update 语句, 如果数据没有改变,
也会更新, 可以设置`<class select-before-update="true"/>`, 设置改变
* 一级缓存不允许存在两个相同OID的对象

`session.saveOrUpdate()`, 如果参数是一个瞬时对象, 就用save方法,
如果是脱管对象, 就用 update 方法. `<id unsaved-value="-1"/>`,
如果对象id是 -1 , 那么也是瞬时对象

`session.clear()`, 把所有的对象从session中清空(变成托管对象)

`session.evict(p)`, 把一个对象变成托管状态

`session.flush()`, 将缓存的变化同步到数据库, 缓存数据与快照不同时, 才执行

`session.refresh()`, 重新更新缓存, 用数据库的内容更新缓存

`session.setFlushMode(FlushMode)` 设置 缓存刷新时间
* FlushMode.ALWAYS, 每次查询都会 flush
* FlushMode.AUTO, 有些查询时 flush
* FlushMode.COMMIT, 提交时 flush
* FlushMode.MANUAL, 手动 flush

一个对象是否是持久化对象是针对某一个session而言的
~~~~~~
Session session = sessionFactory.getSession();
Person person = session.get(Person.class, 1L);
sessoin.close();
// person 相对于 session2 来说是临时状态, 所以不会生成sql语句
Sessoin session2 = sessionFactory.getSession();
person.setName("");
session.close();
~~~~~~

当 `transaction.commit()` 时, Hibernate 会检查session,
* 如果一个对象是临时状态对象, session不会管
* 如果是一个持久化状态对象, 那么先把该对象和快照对比, 来决定是否执行sql update操作
* 如果是一个没有Id的持久化对象, 则执行save操作
