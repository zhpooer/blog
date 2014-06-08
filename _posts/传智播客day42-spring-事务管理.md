title: 传智播客day42-Spring 事务管理
date: 2014-05-29 09:01:42
tags:
- 传智播客
- spring
---

# Spring事务管理 #
在Java开发中, 事务管理代码放到业务层

## 事务管理 API ##
* `PlatformTransactionManager` 提供事务管理方法, 核心接口
~~~~~~
// 提交事务
void commit(TransactionStatus status);
// 根据事务定义信息，获得当前状态 
TransactionStatus getTransaction(TransactionDefinition definition);
// 回滚事务
void rollback(TransactionStatus status)
~~~~~~
* `TransactionDefinition` 事务的定义信息(隔离级别, 传播行为, 超时时间, 只读)
  * ISOLATION_xxx 事务隔离级别 
  * PROPAGATION_xxx  事务传播行为 
  * int getTimeout()  获得超时信息
  * boolean isReadOnly()  判断事务是否只读
* TransactionStatus 事务具体行为, 每一个时刻点, 事务具体状态信息

关系: PlatformTransactionManager 根据 TransactionDefinition 进行事务管理, 
管理过程中事务存在多种状态, 每个状态信息通过 TransactionStatus 表示

## PlatformTransactionManager ##

Spring 为不同的持久化框架提供了不同的PlatformTransactionManager接口实现 ,
针对不同的持久层技术, 要选用对应的事务管理器

| 不同平台事务管理器实现 | 说明 |
|-----------------|
| org.springframework.jdbc.datasource.DataSourceTransactionManager  | 使用Spring JDBC或iBatis 进行持久化数据时使用 |
| org.springframework.orm.hibernate3.HibernateTransactionManager |	使用Hibernate3.0版本进行持久化数据时使用 | 
| org.springframework.orm.jpa.JpaTransactionManager	 | 使用JPA进行持久化时使用 |
| org.springframework.jdo.JdoTransactionManager	| 当持久化机制是Jdo时使用 |
| org.springframework.transaction.jta.JtaTransactionManager | 	使用一个JTA实现来管理事务，在一个事务跨越多个资源时必须使用|

# 事务的隔离级别 #
四大特性: ACID, 原子性, 一致性, 隔离性, 持久性

隔离性引发并发问题：脏读 不可重复读, 幻读
* 脏读 一个事务读取另一个事务 未提交数据
* 不可重复读 一个事务读取另一个事务 已经提交 update 数据
* 虚读  一个事务读取另一个事务 已经提交 insert 数据

事务的隔离级别: 为了解决事务隔离性引发的问题
* DEFAULT 默认级别  mysql REPEATABLE_READ 、 oracle READ_COMMITTED
* READ_UNCOMMITED  导致所有问题发生
* READ_COMMITTED 防止脏读、 发生不可重复读和虚读
* REPEATABLE_READ 防止脏读、不可重复读，发生虚读
* SERIALIZABLE 防止所有并发问题 

# 事务的传播行为 #

为什么要有事务的传播行为?(Why)

    实际开发中, 业务层方法间相互调用, 如在删除客户信息时, 要先删除订单信息
    那么删除订单出错,客户要不要删除?

什么是事务的传播行为?(what)

    一个业务层事务调用令一个业务层事务, 事务间之间关系如何处理

七种传播行为:
* PROPAGATION_REQUIRED 支持当前事务, 如果不存在 就新建一个
* PROPAGATION_SUPPORTS 支持当前事务, 如果不存在，就不使用事务
* PROPAGATION_MANDATORY 支持当前事务, 如果不存在，抛出异常
* PROPAGATION_REQUIRES_NEW 如果有事务存在, 挂起当前事务, 创建一个新的事务
  * 生成订单, 发送通知邮件, 通知邮件会创建一个新的事务, 如果邮件失败, 不影响订单生成
* PROPAGATION_NOT_SUPPORTED	以非事务方式运行，如果有事务存在，挂起当前事务
* PROPAGATION_NEVER 以非事务方式运行, 如果有事务存在, 抛出异常
* PROPAGATION_NESTED 如果当前事务存在, 则嵌套事务执行
  * 依赖于 JDBC3.0 提供 SavePoint 技术 
  * 删除客户 删除订单, 在删除客户后, 设置SavePoint, 执行删除订单, 删除订单和删除客户在同一个事务,
  删除订单失败， 事务回滚 SavePoint , 由用户控制是事务提交 还是 回滚


# 事务管理方式 #

编程式事务管理 
* 在代码中通过 TransactionTemplate 手动进行事务管理, 在实际开发中很少被用到

声明式事务管理
* 在配置文件中, 对 Bean 的方法进行事务管理, 基于AOP思想, 无需写代码 

## 实际案例: 转账 ##
如果没有进行事务管理, JdbcTemplate DAO 每一个操作, 都是一个事务

数据库脚本
~~~~~~
CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `money` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
INSERT INTO `account` VALUES ('1', 'aaa', '1000');
INSERT INTO `account` VALUES ('2', 'bbb', '1000');
~~~~~~

~~~~~~
<!-- 最全的spring 模板 -->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!-- 导入外部属性文件, 配置连接池 -->
    <bean id="accountService" class="zhpooer.AccountServiceImpl">
         <property name="accountDao" ref="accountDao"></property>
         <property name="transactionTemplate" ref="transactionTemplate"> </property>
    </bean>
    <bean id="accountDao" class="zhpooer.AccountDaoImpl">
    <!-- 将连接池注入给DAO, JdbcTemplate会自动 创建 -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!-- 编程式事务管理配置 -->
    <!-- 事务管理模板 -->
    <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager" ref="tractionManager"> </property>
    </bean>
    <!-- 事务管理器 -->
    <!-- org.springframework.jdbc.datasource.DataSourceTransactionManager 用来管理jdbc事务操作 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSrouce"> </property>
    </bean>
</beans>
~~~~~~

~~~~~~
public class AccountServiceImpl implements AccountService{
    // 自动注入
    private AccountDao accountDao;
    private TransactionTemplate transactionTemplate;
    
    // 转账
    public void transfer(String outAccount, String inAccount, Double money) {
            // 编程式事务管理, 使用事务模板管理事务
        transactionTemplate.execute(new TransactionCallbackWithoutResult(){
            @Override
            protected void doInTransactionWithoutResult(){
                accountDao.outMoney(outAccount, money);
                accountDao.inMoney(inAccount, money);
            }
        });
    }
}

public class AccountDaoImpl extends JdbcDaoSupport implements AccountDao {
    public void outMoney(String outAccount, Double money) {
        String sql = "update account set money = money-? where name=?";
        getJdbcTemplate().update(sql, money, outAccount);
    }

    public void inMoney(String inAccount, Double money) {
        String sql = "update account set money = money+? where name=?";
        getJdbcTemplate().update(sql, money, inAccount);
    }
}
~~~~~~


### 声明式事务管理 ##

#### TransactionProxyFactoryBean ####
通过 TransactionProxyFactoryBean 对业务类创建代理,
实现声明式事务管理, 无需修改 Service 代码

缺点, 需要为每个Bean 都创建单独代理对象，开发量巨大

~~~~~~
<!-- 事务管理器 -->
<bean id="transactionManager" class="">
    <proerty name="dataSource" ref="dataSource">
    </proerty>
</bean>
<!-- 为目标Servicee创建代理 -->
<bean name="accountServiceProxy" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <!-- 目标 -->
    <property name="target" ref="accountService"> </property>
    <!-- 针对接口代理 -->
    <property name="proxyInterfaces" value="zhpooer.AccountService"></property>
    <!-- 增强 事务管理 -->
    <property name="transactionManager" ref="transactionManager"></property>
    <!-- 事务管理属性 -->
    <property name="transactionAttributes">
        <props>
           <!-- key 就是方法名  -->
           <!-- value prop格式：PROPAGATION,ISOLATION,readOnly,-Exception,+Exception -->
           <!-- PROPAGATION 事务传播行为 -->
           <!-- ISOLATION, 事务隔离级别 -->
           <!-- readOnly  表示事务是只读的，不能进行修改操作  -->
           <!-- -Exception 发生这些异常事务进行回滚(默认发生任何异常事务都会回滚) -->
            <!-- +Exception 事务将忽略这些异常，仍然提交事务  -->
            <prop key="transfer"> PROPAGATION_REQUIRED, readOnly, +java.lang.ArithmeticException </prop>
        </props>
    </property>
</bean>
<!-- 注入代理对象: @Qualifier("accountServiceProxy") -->
~~~~~~

#### tx, 自动代理 ####
~~~~~~
<!-- 定义事务管理增强 -->
<tx:advice id="txAdvicd" transaction-manager="transactionManager">
    <tx:attribute>
        <!--
        name="transfer" 事务管理方法名
        isolation="DEFAULT" 默认隔离级别
        qpropagation="REQUIRED"  默认传播行为
        read-only="false"  是否只读
        no-rollback-for=""  发生异常不会滚  类似+Exception
        rollback-for=""  发生异常回滚 类似-Exception
        timeout="-1"  不超时
        -->
        <tx:method name="transfer" isolatioin="DEFAULT" propagation="REQUIRED"
                   read-only="false" timeout="-1"></tx:method>
    </tx:attribute>
</tx:advice>
<!-- 使用Aop 进行自动代理 -->
<aop:config>
    <!-- 定义切点 -->
    <aop:pointcut expression="execution(public * * (..))" id="mypointcut"></aop:pointcut>
    <!-- 定义切面 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="mypointcut"></aop:advisor>
</aop:config>
~~~~~~

#### 注解实现事务管理 ####
1. 在需要管理的类或者方法添加 `@Trasactional`
~~~~~~
// isolation 隔离级别
// propagation 传播行为
// readOnly 是否只读
// noRollbackFor 发生异常不回滚
// rollbackFor 发生异常回滚
// timeout 超时时间 -1 不超时

@Transactional(isolation=Isolation.DEFAULT,propagation=,
               readyOnly=, noRollbackFor=, rollbackFor, timeout=)
public void transfer(){}
~~~~~~

2. 在 applicationContext.xml
~~~~~~
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
~~~~~~

# SSH 框架整合 #
导入Jar包, 及配置文件
* 表现层框架 struts2, `struts2-json-plugin.jar`,
`struts2-spring-plugin.jar`, `struts2-conversion.jar`(注解), 以及 web.xml(filter), struts2.xml
* 业务层 spring3
~~~~~~
配置 web.xml
<listener>
    <listener-class> ContextLoaderListner </listener-class>
</listener>
~~~~~~
* 持久层框架 hibernate3

## Spring 和 Hibernate 整合 ##

### 零障碍整合 ###
通过 Spring 提供 LocalSessionFactoryBean, 注解引入hibernate配置文件,
在 Spring 容器中获得 SessionFactory对象, 将 SessionFactory, 注入到 DAO 程序

图书添加

移动脚本
~~~~~~
create table(
    id int not null AUTO_INCREMENT PRIMARY key,
    bookname varchar(20) not null,
    price double not null
);
~~~~~~

~~~~~~
public class Book {
     private Integer id;
     private String bookname;
     private double price;
}

public class BookDao extends HibernateDaoSupport{
    
    public void saveBook(){}
}
~~~~~~

~~~~~~
<!-- hibernate.cfg.xml -->
<mapping resource="domain.Book.hbm.xml"/>

<!-- Book.hbm.xml -->
<hibernate-mapping>
    <class name="domain.Book" table="book" catalog="">
        <id name="id">
            <generator class="identity"> </generator>
        </id>
        <property name="bookname"> </property>
        <property name="price"> </property>
    </class>
</hibernate-mapping>

<!-- applicationContext.xml -->
<!-- 配置 sessionFactory -->
<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <!-- 加载hibernate配置文件 -->
    <property name="configLocation" value="classpath:hibernate.cfg.xml">  </property>
</bean>
<!-- 注入 sessionFactory -->
<bean id="bookDao" class="dao.BookDao">
    <property name="sessionfactory" ref="sessionfactory"> </property>
</bean>

<bean id="bookService" class="service.BookService">
    <property name="bookDao" ref="bookDao"> </property>
</bean>
<tx:advice id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <tx:attribute>
        <tx:method name="*"> </tx:method>
    </tx:attribute>
</tx:advice>
<aop:config>
    <aop:advisor advice-ref="transactionManager" pointcut="execution()"> </aop:advisor>
</aop:config>
~~~~~~

~~~~~~
// BookDao.java
// HibernateTemplate 常用 api
public void saveBook(Book book){
    this.getHibernateTemplate().save(book);
}
public void updateBook(Book book) {
    this.getHibernateTemplate().update(book);
}
public void deleteBook(Book book) {
    this.getHibernateTemplate().delete(book);
}
public void findById(Integer id) {
    return this.getHibernateTemplate().get(Book.class, id);
}p
public List<Book> findAll(){
    // this.getSession().createQuery("from Book").list();
    return this.getHibernateTemplate().find("from Book");
}
public Book findByName(String name){
    //  this.getSession().createQuery("from Book where name=?").setParameter(0, name).uniqueResult();
    return this.getHibernateTemplate().find("from Book where name=?", name);
}
~~~~~~

业务层
~~~~~~
public class BookService{
    private BookDao bookDao;
    public void saveBook(Book book) {
        bookDao.saveBook(book);
    }
    public List<Book> findAllBooks(){
        return bookDao.findAll();
    }
}
~~~~~~

### 将 Hibernate 参数配置到 Spring ###
将 hibernate框架的所有参数, 都配置到 applicationContext.xml

~~~~~~
<!-- applicationContext.xml -->
<context:property-placeholder location="classpath:"></context:property-placeholder>
<!-- c3p0连接池 -->
<bean id="dataSource" class="com.ComboPooledDataSource">
    <!-- any other configuration  -->
</bean>

<!-- 配置 sessionFactory -->
<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="hibernateProperties">
        <props>
            <!-- 数据库方言 -->        
            <prop key="hibernate.dialect"> org.hibernate.dialect.MySQLDialect </prop>
            <prop key="hibernate.hbm2dll.auto">update </prop>
            <prop key="hibernate.format_sql">true </prop>
            <prop key="hibernate.show_sql">true </prop>
        </props>
    </property>
    <!-- 配置 hbm 文件 -->
    <property name="mappingResources | mappingLocations | mappingDirectoryLocations">
        <!-- mappingResources -->
        <list>
            <value>cn/zhpooer/Person.hbm.xml </value>
        </list>
        
        <!-- mappingLocations -->
        <list>
            <value>classpath: cn/zhpooer/Person.hbm.xml </value>
        </list>
        
        <!-- mappingDirectoryLocations, 查找所有目录下的配置文件 -->
        <list>
            <value>classpath: cn/zhpooer/domain </value>
        </list>
    </property>
</bean>
~~~~~~
### HibernateTemplate 用法 ###

~~~~~~
template.save(obj);
template.update(obj);
template.saveOrUpdate();
template.delete(obj);
template.get(class, id);
template.load(class, id)

// 查询
template.find(hql, Object... args); // QBC查询, 等价于 session.createQeury(hql);

template.findByCriteria(detachedCriteria) // 完全面向对象查询
template.findByNamedQuery(queryName) // 命名查询, 查询语句写入配置文件, 方便项目管理

public void testQBC {
    HibernateTemplate template = new HibernateTemplate();
    DetachedCriteria criteria = DetachedCriteria.forClass(Book.class); // 生成 select * from book;
    criteria.add(Restrictions.like("bookname", "%为什么%"));
    List<Book> books = template.findByCriteria(criteria);
}
/* 配置文件
* <hibernate-mapping>
*     <!-- 配置 hql -->
*     <query name="book.findbyname"> from book where bookname like ?</query>
*     <!-- 配置 sql -->
*     <sql-query name="book.findbynamesql">
*         <return class="domain.Book"></return> <!-- 将结果集封装到book对象 -->
*         select * from book where bookname like ?
*     </sql-query>
* </hibernate-mapping>
*/ 
public void testFindByNamedQuery(){
    List<Book> books = template.findByNamedQuery("book.findbyname", "%为什么%")

    List<Book> books = template.findByNamedQuery("book.findbynamesql", "%为什么%")
}
~~~~~~

## Spring 和 Struts 整合 ##
Struts2 整合 Spring 框架原理: 修改 struts2 默认对象工厂(struts -> spring),
导入 `struts2-spring-plugin`

### 自动装配 Service ###
由 struts2 自己装配Action, 再由 Spring
~~~~~~
public class BookaddAction extends ActionSupport{
    private Book book = new Book();
    // 自动装配, 不需要提供任何配置, 只需要在Action里提供 service的 set方法
    // 原理: struts配置文件中
    // `struts.objectFactory.spring.autoWire = true`(根据名字自动注入), 生效 

    private BookService bookService;
    public String execute(){
        pringln("添加图书")
        return NONE;
    }
}
~~~~~~

~~~~~~
<form action="bookadd.action" method="post">
    <input type="text" name="bookname"/>
    <input type="text" name="price"/>
    <input type="submit"/>
</form>

<struts>
    <package name="default" namespace="/" extends="struts-default">
        <action name="bookadd" class=BookaddAction"">
        </action>
    </package>
</struts>
~~~~~~

### Action由 Spring 管理 ###
将 struts2 的Action配置spring管理的Bean对象

1. 将Action配置为 Spring 中的一个Bean对象
~~~~~~
<!-- applicationContext.xml 加入ActionBean -->
<!-- 必须设置为 prototype -->
<bean id="bookaddAction" class="action.BookaddAction" scope="prototype">
    <property name="bookService" ref="bookService"></property>
</bean>
~~~~~~
2. Struts.xml 配置 Spring bean 的 id, 作为 class 的属性
~~~~~~
<!-- 不是真实类名, 只是只给伪类名 -->
<package>
    <action name="bookadd" class="bookaddAction">
    </action>
</package>
~~~~~~

### 结论 ###
第一种整合方式, Action 由 Struts2 自己管理, Service对象采用自动装配

第二种整合方式, Action 由 spring 自己管理, 依赖注入service对象, struts2 需要配置伪类名

第二种方式, 可以对 Action 进行 AOP 增强

# 延迟加载问题 #

懒加载对象, 但是对象已经脱管, 报异常 `no session`

解决方案
1. 设置为立即加载 lazy=false, 缺点, 每次查询客户, 都要查询订单
2.  在业务层, 事务关闭前手, 动初始化
`Hibernate.initialize(customer.getOrders())`, 缺点, 需要写代码
3. OpenSessionView, session 不随事务关闭而关闭,
将 session 延迟到表现层, 存在性能问题, 通过配置, 无需编码
~~~~~~
<!-- web.xml -->
<!-- 在Struts2的过滤器前, 配置 OpenSessionInViewFilter -->
<filter>
    <filter-name>openfilter</filter-name>
    <filter-class>org.springframework.orm.hiberante3.support.OpenSessionInViewFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>openfilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
~~~~~~

# 注解整合 #

导入 `struts-conversion-plugin.jar`

~~~~~~
// hiberante注解 是对 jpa注解的扩展
@Entity
@Table(name="book", catalog="")
public class Book {
    @Id
    @GeneratedValue(strategy = GeneratedType.IDENTITY)
    private Integer id;
    // 如果列名和属性名不一致, 可以不写
    @Column(name="", nullable=true)
    private String bookname;
    private double price;
}
@Repository("bookDao")
public class BookDao {
    @Resource(name="hiberanteTemplate")
    private HiberanteTemplate template;
}
@Controller("bookaddAction")
@Scope("prototype")
@Namespace("/")
@ParentPackage("struts-default")
public class BookaddAction{
    @Action("bookadd")
    public String execute(){
    }
}
~~~~~~

~~~~~~
<!--  AnnotationSessionFactoryBean 支持注解功能-->
<bean id="sessionFactory" class="org.springframework.orm.hiberante3.annotation.AnnotationSessionFactoryBean ">
    <!-- 其他配置同上  -->
    <property name="packageToScan">
        <list>
            <value>cn.zhpooer</value>
        </list>
    </property>
</bean>
<context:component-scan base-package="cn.itcast, io.zhpooer."/>
<context:annotation-config/>
<!-- 注解事务管理 -->

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSrouce"> </property>
</bean>
<tx: annotation-driven transaction-manager="transactionManager"/>
~~~~~~
