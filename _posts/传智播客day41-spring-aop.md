title: 传智播客day41-spring AOP
date: 2014-05-27 09:19:03
tags:
- 传智播客
- spring
---
# AOP 概念 #
Aspect Oriented Programing 面向切面编程, AOP是对OOP(面向对象编程) 思想的延伸,

AOP采取横向抽取机制，取代了传统纵向继承体系重复性代码(性能监视、事务管理、安全检查、缓存)

底层原理: **代理** 
* 传统继承, 纵向结构代码复用
~~~~~~
abstract class BaseDao{
    public void wirteLog(){doSome()}
}

class UserDao extends BaseDao{}
class ProductDao extends BaseDao{}
~~~~~~
* AOP, 使用代理机制, 将复用的代码放入代理中

## 相关术语 ##
* Joinpoint(连接点), 在代理过程中, 可以被拦截的点(指方法)
* Pointcut(切入), 要对哪些Jointpoint进行拦截定义, 指定拦截(个别方法)
* Advice(通知, 建议), 增强的代码逻辑(日志记录), 方法级别
* Introduction(引介), 特殊类型Advice, 对原有的类对象添加一个新的属性或者方法
* Target, 被代理对象
* Weaving(织入), 把增强应用到目标对象, 来创建对象的过程
* Proxy(代理), 一个类被AOP织入增强后，就产生一个结果代理类
* Aspect(切面), 是切入点(Joinpoint)和通知(Advice)的结合, 多个切点和多个通知组成


# AOP底层原理 #
AOP, 使用代理机制, 将复用的代码放入代理中

## JDK动态代理 ##
使用JDK动态代理, JDK1.3新特性

原理： 针对内存中Class对象，使用类加载器 动态为目标对象实现接口的创建代理类
* 代理类 是动态创建的， 代理类 和 被代理对象 实现相同接口 
* 被代理对象 必须要实现 接口(JDK代理 只能针对接口 进行代理)

## CGLIb ##

JDK动态代理, 为目标对象接口生成代理对象,
对于不使用接口的业务类, 无法使用JDK动态代理

CGLIB(Code Generation Library)是一个开源项目.
是一个强大的,高性能,高质量的Code生成类库,
它可以在运行期扩展Java类与实现Java接口.
Hibernate支持CGlib 来实现PO字节码的动态生成.
* Hibernate 默认PO 字节码生成技术  javassist

CGLIB 是一个第三方技术，使用时 ，需要下载 jar 包
* Spring3.2 版本， spring-core jar包 已经集成 cglib 开发类 

原理: CGlib采用非常底层字节码技术, 可以为一个类创建子类,
解决无接口代理问题
~~~~~~
public ProductDao createCglibProxy(){
    // 创建代理的核心对象
    Enhancer enhancer = new Enhancer();
    // 设置被代理类, 为类创建子类
    enhancer.setSuperclass(productDao.getClass());
    enhancer.setCallback( new MethodInterceptor(){
        public Object intercept(Object proxy, Method method, Object[] rags, MethodProxy methodProxy){
            // 为 addProduct 计算运算时间
            if (method.getName().equals("addProduct")) {// 当前执行方法
                long start = System.currentTimeMillis();
                Object result = methodProxy.invokeSuper(proxy, args);
                long end = System.currentTimeMillis();
                System.out.println("addProduct方法运行时间 : " + (end - start));
                return result;
            } else {
                // 不进行增强
                return methodProxy.invokeSuper(proxy, args);
            }
        }
    });
    // 返回代理
    return (ProductDao) enhancer.create();
}
~~~~~~

## 结论 ##
程序中应优先对接口创建代理，便于程序解耦维护
* 若目标对象实现了若干接口.spring使用JDK的java.lang.reflect.Proxy类代理
* 若目标对象没有实现任何接口.spring使用CGLIB库生成目标对象的子类

# 传统Spring AOP #
AOP 开发规范: AOP联盟为通知Advice定义了`org.aopalliance.aop.Interface.Advice`

Spring AOP 实现 AOP 联盟定义的规范

传统Spring AOP提供五类 Advice:
    前置通知(代码增强) org.springframework.aop.MethodBeforeAdvice
	* 在目标方法执行前实施增强
    后置通知 org.springframework.aop.AfterReturningAdvice
	* 在目标方法执行后实施增强
    环绕通知 org.aopalliance.intercept.MethodInterceptor
	* 在目标方法执行前后实施增强
    异常抛出通知 org.springframework.aop.ThrowsAdvice
	* 在方法抛出异常后实施增强
    引介通知 org.springframework.aop.IntroductionInterceptor （课程不讲 了解）
	* 在目标类中添加一些新的方法和属性

## Advisor ##
Advisor 就是对 PointCut 应用 Advise, 指一个 Point 和一个 Advise

类型：
* Advisor : 代表一般切面, Advice本身就是一个切面, 对目标类所有方法进行拦截(没有切点)
* PointcutAdvisor : 代表具有切点的切面, 可以指定拦截目标类哪些方法
* IntroductionAdvisor : 代表引介切面，针对引介通知而使用切面(不重要)

### 普通 Advisor ###
使用Advice作为一个切面, 不定义切点, 拦截目标类所有方法 
1. 导入jar包, `spring-aop.jar`, `com.springsource.org.aopalliance.jar`
2. 被代理接口和实现类
~~~~~~
public interface CustomerDao{ public void save();}
public class CustomerDAOImpl implements CustomerDao{ public void save(){};}
~~~~~~
3. 编写前置增强
~~~~~~
public class MyMethodBeforeAdvice implements MethodBeforeAdvice {
    public void before(Method method, Object[] args, Object target){
        pring("方法前")
    }
}
~~~~~~
4. 为目标对象配置代理
~~~~~~
<beans>
    <bean id="customerDao" class="zhpooer.CusotmerDao"> </bean>
    <bean id="mybeforeadvice" class="zhpooer.MyMethodBeforeAdvice"> </bean>
    <!-- 使用代理工厂类, 创建代理 -->
    <bean id="customerDAOProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <!--
        target : 代理的目标对象
        proxyInterfaces : 代理要实现的接口, 如果多个接口可以使用以下格式赋值
        proxyTargetClass : 是否对类代理而不是接口，设置为true时，使用CGLib代理
        interceptorNames : 需要织入目标的Advice
        singleton : 返回代理是否为单实例，默认为单例
        optimize : 当设置为true时，强制使用CGLib
        -->
        <!-- 目标 -->
        <property name="target" ref="customerDAO"></property>
        <!-- 针对接口代理,如果不用接口代理, 可以不写 -->
        <property name="proxyInterfaces" value="cn.itcast.aop.c_advisor.CustomerDAO"></property>
        <!-- 增强 
            interceptorNames 表示可以运用多个 Advice, 必须写value
            value 引用增强的名字
        -->
        <property name="interceptorNames" value="mybeforeadvice"></property>
    </bean>
</beans>
~~~~~~
5. 使用代理
~~~~~~
CustomerDao dao = context.getBean("customerDAOProxy");
dao.save();
~~~~~~

### PointcutAdvisor ###
带有切点的切面, 指定被代理对象哪些方法会被增强
* JdkRegexpMethodPointcut 构造正则表达式切点
* 使用正则表达式 切点切面 `org.springframework.aop.support.RegexpMethodPointcutAdvisor `

1. 创建被代理接口和对象 `public class OrderDaoImpl implements OrderDAO{}`
2. 环绕代码增强
~~~~~~
class MyMethodInterceptor extends MethodInterceptor{}
~~~~~~
3. 配置
~~~~~~
<beans>
    <bean id="OrderDao" class="zhpooer.OrderDaoImpl"> </bean>
    <bean id="mymethodinterceptor" class="zhpooer.MyMethodInterceptor"> </bean>
    <!-- 定义切点切面 -->
    <bean id="myadvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
        <!-- 正则表达式规则 -->
        <!-- pattern: zhooer\.OrderDao\.add.* -->
        <property name="pattern" value=".*"></property>
        <!-- 多个规则 -->
        <property name="patterns" value="*add, *delete"></property>
        <property name="advice" ref="mymethodinterceptor"></property>
    </bean>
    <!-- 创建代理 -->
    <bean id="orderDAOProxy" class="org.springframework.aop.framework.ProxyFactoryBean" >
        <!-- 目标 -->
        <property name="target" ref="orderDAO"></property>
        <!-- 针对类代理 -->
        <property name="proxyTargetClass" value="true"></property>
        <!-- 增强 -->
        <property name="interceptorNames" value="myadvisor"></property>
	</bean>
<beans>
~~~~~~


## 自动代理 ##
使用ProxyFactoryBean 创建代理，需要为每个Bean 都配置一次, 非常麻烦

自动代理和ProxyFactoryBean本质区别:
	ProxyFactoryBean, 先有被代理对象, 传递ProxyFactoryBean, 创建代理 
	自动代理, Bean构造过程中, 使用后处理Bean 创建代理, 返回构造完成对象就是代理对象 

自动代理原理： 根据xml中配置advisor的规则，得
知切面对哪个类的哪个方法进行代理 (切面中本身就包含 被代理对象信息) ,
就不需要ProxyFactoryBean ，使用BeanPostProcessor 完成自动代理 

* BeanNameAutoProxyCreator 根据Bean名称创建代理 
* DefaultAdvisorAutoProxyCreator 根据Advisor本身包含信息创建代理
* AnnotationAwareAspectJAutoProxyCreator 基于Bean中的AspectJ 注解进行自动代理

~~~~~~
<!-- 被代理对象 -->
<bean id="OrderDao" class="zhpooer.OrderDaoImpl"> </bean>
<bean id="customerDao" class="zhpooer.CusotmerDao"> </bean>
<!-- 增强 -->
<bean id="mymethodinterceptor" class="zhpooer.MyMethodInterceptor"> </bean>
<bean id="mybeforeadvice" class="zhpooer.MyMethodBeforeAdvice"> </bean>

<!-- 第一种BeanName自动代理  -->
<!-- 后处理, 不需要配置id -->
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <!-- 对所有DAO结尾Bean 进行代理 -->
    <property name="beanNames" value="*DAO"></property>
    <!-- 增强 -->
    <property name="interceptorNames" value="mymethodinterceptor"></property>
</bean>

<!-- 第二种,基于切面信息自动代理 -->
<!-- 切面 -->
<bean id="myadvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <!-- 切点拦截信息 -->
    <property name="patterns" value="zhpooer\.OrderDAO\.save.*"></property>
    <!-- 增强 -->
    <property name="advice" ref="mybeforeadvice"></property>
</bean>
<!-- 后处理会自动读取切面信息   -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"></bean>
~~~~~~

# AspectJ #
Spring 2.0 之后,

AspectJ是一个面向切面的框架,它扩展了Java语言.
AspectJ定义了AOP语法所以它有一个专门的编译器用来生成遵守Java字节编码规范的Class文件.

Spring2.0之后 为了简化 AOP编程,  整合了 AspectJ, 支持AspectJ 技术 
@AspectJ 是AspectJ1.5新增功能, 通过JDK5注解技术，允许直接在Bean类中定义切面

新版本Spring框架, 建议使用AspectJ方式来开发AOP, 而不需要使用传统 Spring AOP 编程

## 基于注解 ##
1. 导入jar包, `aspectj.weaver.jar`, `spring-aspects.jar`
2. spring配置文件, 需要 aop 名称空间, 
~~~~~~
<!-- 配置自动代理 -->
<!-- <bean class="... AnnotationAwareAspectJAutoProxyCreator" /> -->
<aop:aspectj-autoproxy/>
~~~~~~

常用注解

    @Aspect 定义切面 
    通知类型 
    @Before 前置通知，相当于BeforeAdvice
    @AfterReturning 后置通知，相当于AfterReturningAdvice
    @Around 环绕通知，相当于MethodInterceptor
    @AfterThrowing抛出通知，相当于ThrowAdvice
    @After 最终final通知，不管是否异常，该通知都会执行
    @DeclareParents 引介通知，相当于IntroductionInterceptor (不要求掌握)


切点使用指定哪些连接点 会被增强, 通过execution函数，可以定义切点的方法切入

    语法： execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)
	execution(public * *(..)) 匹配所有public修饰符 任何方法 
	execution(* cn.itcast.service.HelloService.*(..))  匹配HelloService类中所有方法 
	execution(int cn.itcast.service.UserService.regist(..))  匹配UserService中int返回类型 regist方法 
	execution(* cn.itcast.dao..*(..))  匹配cn.itcast.dao包下 （包含子包） 所有类 所有方法
	execution(* cn.itcast.dao.*(..)) 不包含子包  
	execution(* cn.itcast.dao.GenericDAO+.*(..))  匹配GenericDAO 所有子类 或者 实现类 所有方法

~~~~~~
public class UserDao{
    public void save(){}
    public void delete(){}
    public void search(){}
}

// 自定义切面类
@Aspect
// 声明一个切面类
public class MyAspect {

    //方式一 前置通知
    @Before("execution(public * *(..))")
    public void before(){
        // 前置通知不能拦截目标方法执行
    }
    @Before("execution(public * *(..)"))
    public void before(aspactj.Joinpoint joinpoint){
        joinpoint.toString(); // 获得拦截点的信息
    }
    
    // 方式二 后置通知
    @AfterReturning("execution(public * *(..))", returning="returnValue")
    // returnValue 是代理方法参数名, 前后两个参数名必须一致
    public void afterReturning(Object returnValue){
        // 获得方法的返回值
    }

    // 方式三 环绕通知
    @Around("execution()", )
    public Object around (ProceedingJoinPoint pjp){
        // 可以阻止 search 方法执行
        Object result = pjp.proceed();
        return result;
    }

    //方式四 抛出通知 , 出现异常后, 方法得到执行
    @AfterThrowing("execution()", throwing="e")
    public void afterThrowing(Throwable e){
    }

    //方式五 最终通知, 不管代码是不是抛出代码都执行, 可以用来释放资源
    @After("execution()")
    public void after(){
    }
}
// <bean id="userDao" class="zhpooer.UserDao"></bean>
// <bean id="myAspect" class="zhpooer.MyAspect"></bean>
~~~~~~

### 切点的定义 ###
直接在通知上定义切点表达式, 会造成切点的重复, 工作量大, 不易维护

~~~~~~
// 切点定义
@Pointcut("execution()")
// 方法名,就是切点名字
private void mypointcut(){}

// 应用切点
@After("MyAspect.mypointcut()")
public void after(){}
~~~~~~

advisor 和 aspect 区别 ？

	advisor 是 spring 中 aop定义切面，通常由一个切点和一个通知组成
    aspect 是规范中切面 ， 允许由多个切点和 多个通知组成 	

## 基于XML ##
~~~~~~
// 被代理对象
public class ProductDao {
    public void sell();
}

public class MyAspect {
    public void before(){
        // 前置增强
    }
    public void afterReturning(Ojbect returnValue){
        // 后置增强
    }
    public Objct around(ProceedingJoinPoint pjp){
        // 环绕增强
    }
}
~~~~~~

~~~~~~
<!-- 定义被代理对象 -->
<bean id="productDao" class="ProductDao"></bean>
<!-- 定义切面 -->
<bean id="myAspect" class="MyAspect"></bean>
<!-- 进行AOP配置 -->
<aop:config>
    <aop:aspect ref="myAspect">
        <aop:pointcut expression="exection" id="mypointcut"></aop:pointcut>
        <!-- 配置前置  -->
        <aop:before method="before" pointcut-ref="mypointcut"></aop:before>
        <!-- 后置增强 -->
        <aop:after-returning method="afterReturning" pointcut-ref="mypointcut"
             returning="returnValue"/>
        <!-- 环绕增强  -->
        <aop:around method="around" pointcut-ref="mypointcut"></aop:around>
        <!-- 抛出通知  -->
        <aop:after-throwing method="afterThrowing" throwing="ex"> </aop:after-throwing>
        <!-- 最终通知 -->
        <aop:after method="after" pointcut-ref="mypointcut"></aop:after>
    </aop:aspect>
</aop:config>
~~~~~~

# JDBC Template #

Spring 为各种支持的持久化技术, 都提供了简单的模板工具类和回调

| 不同持久化技术 | 模板工具 |
|-------------------|
| JDBC      | org.springframework.jdbc.core.JdbcTemplate |
| Hibernate | org.springframework.orm.hiberante3.HibernateTemplate |
| IBatis    | org.springframework.orm.ibatis.SqlMapClientTemplate |
| JPA       | org.springframework.orm.jpa.JpaTemplate |


JdbcTemplate 是用来简化JDBC操作的, 类似 DbUtils 框架.

## 快速入门 ##

导入jar包, `spring-jdbc.jar`, `spring-tx.jar`

### 手动运行Jdbc ###
~~~~~~
public void demo1(){
   // 数据库连接池
   DirverManagerDataSource ds = new DirverManagerDataSource();
   ds.setDriverClassName("com.mysql.jdbc.Driver");
   ds.setUrl("jdbc:mysql:///");
   ds.setUsername();
   ds.setPassword();
   JdbcTemplate jt = new JdbcTemplate(ds);
   jt.execute("");
}
~~~~~~


### 使用 xml 配置 ###

常用数据源
* Spring 数据源实现类 `DriverManagerDataSource`
~~~~~~
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
	<property name="url" value="jdbc:mysql:///spring3day2"></property>
	<property name="username" value="root"></property>
	<property name="password" value="abc"></property>
</bean>
<bean id="JdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"> </property>
</bean>
~~~~~~
* DBCP 数据源 BasicDataSource 
~~~~~~
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
	<property name="url" value="jdbc:mysql:///spring3day2"></property>
	<property name="username" value="root"></property>
	<property name="password" value="abc"></property>
</bean>
~~~~~~
* C3P0 数据源 ComboPooledDataSource
~~~~~~
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
	<property name="jdbcUrl" value="jdbc:mysql:///spring3day2"></property>
	<property name="user" value="root"></property>
	<property name="password" value="abc"></property>
</bean>
~~~~~~

外部属性文件引入, 在Spring 直接修改常用属性，不方便，
可以将属性抽取出来 建立单独 properties 文件，在Spring 中引入properties
~~~~~~
<!-- 方式一  -->
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="location" value="classpath:jdbc.properties"></property>
</bean>
<!-- 方式二 -->
<context:property-placeholder location="classpath:jdbc.properties">

<!-- 将连接池配置参数，使用 ${属性key} -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"></property>
	<property name="jdbcUrl" value="${jdbc.url}"></property>
	<property name="user" value="${jdbc.user}"></property>
	<property name="password" value="${jdbc.password}"></property>
</bean>
~~~~~~

### JdbcTemplate CRUD ###
UserDAO 实现数据库操作, 必须要使用 JdbcTemplate, Spring 将jdbcTemplate 注入 UserDAO  

Spring 为每种持久化技术 提供一个支持类, 支持类作用，在DAO 中注入 模板工具类

    JDBC: org.springframework.jdbc.core.support.JdbcDaoSupport
	Hibernate 3.0: org.springframework.orm.hibernate3.support.HibernateDaoSupport
	iBatis: org.springframework.orm.ibatis.support.SqlMapClientDaoSupport

用户自己编写DAO 只需要继承 JdbcDaoSupport, 就可以注入 JdbcTemplate
~~~~~~
public class UserDao extends JdbcDaoSupport {
    // 修改
    public void save(User user){
        String sql = "insert into user value(?, ?)";
        getJdbcTemplate().update(sql, user.getId(), user.getName());
    }
    // 简单查询查询, 返回原始类型, string类型
    public int count(){
        String sql = "select count(*) from user";
        return getJdbcTemplate().queryForInt(sql);
    }
    public String findNameById(int id){
        String sql = "select name from where id=?";
        return getJdbcTemplate().queryForObject(sql, String.class, id);
    }
    // 复杂查询, 手动完成对象封装
    public User findById(int id) {
        return this.getJdbcTemplate().queryForObject(sql, new UserRowMapper(),id);
    }
    public List<User> findAll(){
        return this.getJdbcTemplate().query(sql, new UserRowMapper());
    }

    class UserRowMapper implements RowMapper<User> {
        @Override
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        // rs 已经指向每一条数据，不需要自己调用 next，将rs指向数据 转换 User对象
            User user = new User();
            user.setId(rs.getInt("id"));
            user.setName(rs.getString("name"));
            return user;
        }
    }
}

~~~~~~
