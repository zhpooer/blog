title: 传智播客day57-Mybatis
date: 2014-07-15 09:12:17
tags:
- 传智播客
- Mybatis
---

# 概况 #
![mybatis api](/img/mybatis_api.png)

mybatis配置
1. SqlMapConfig.xml，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。
2. mapper.xml文件即sql映射文件，文件中配置了操作数据库的sql语句。
此文件需要在SqlMapConfig.xml中加载。
3. 通过mybatis环境等配置信息构造SqlSessionFactory即会话工厂
4. 由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行。
5. mybatis底层自定义了Executor接口操作数据库，
Executor接口有两个实现，一个是基本实现一个是缓存实现。
6. Mapped Statement也是mybatis一个底层对象，
它包装了mybatis配置信息及sql映射信息等。
mapper.xml文件中一个sql对应一个Mapped Statement对象，sql的id即是Mapped statement的id。
7. Mapped Statement对sql执行输入参数进行定义，包括HashMap、基本类型、pojo，
Executor通过 Mapped Statement在执行sql前将输入的java对象映射至sql中，
输入参数映射就是jdbc编程中对preparedStatement设置参数。
8. Mapped Statement对sql执行输出结果进行定义，包括HashMap、基本类型、pojo，
Executor通过 Mapped Statement在执行sql后将输出结果映射至java对象中，
输出结果映射过程相当于jdbc编程中对结果的解析处理过程。

# Demo #
log4j.properties
~~~~~~
# Global logging configuration
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
~~~~~~

SqlMapConfig.xml是mybatis核心配置文件，上边文件的配置内容为数据源、事务管理
~~~~~~
<configuration>
    <environments default="development">
        <environment id="development">
            <!-- 配置事务  -->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
			    <property name="driver" value="com.mysql.jdbc.Driver" />
			    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
			    <property name="username" value="root" />
			    <property name="password" value="mysql" />
			</dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="sqlmap/User.xml"></mapper>
    </mappers>
</configuration>
~~~~~~

~~~~~~
<!-- User.xml -->
<mapper namespace="test">
    <!-- 根据用户账号查询单个用户信息
         将定义的select 理解为一个sql, 这个sql对应statement
         可以将sql的id理解为statement的id-->
    <select id="findUserById" parameterType="java.lang.Integer" resultType="po.User">
       select * from user where id=#{id}
    </select>
    <insert id="insertUser" parameterType="po.User">
        <!-- 指定主键映射的pojo对象的属性  -->
        <!-- order selectKey, 的执行顺序 -->
        <!-- 一般不用自增, 而是用 uuid -->
        <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            select last_insert_id()
        </selectKey>
        insert into user(username, birthday, sex, address, detail, score)
        values(#{username}, #{birthday}, #{sex}, #{address}, #{detail}, #{score})
    </insert>
    <update id="updateUserUserById" parameterType="po.User">
        update user set username=#{username}, birthday=#{birthday},
        sex=#{sex}, address=#{address}, detail=#{detail}, score=#{score}
        where id=#{id}
    </update>
    <delete id="deleteUserById" parameterType="java.lang.Integer">
        delete from user where id=#{value}
    </delete>
</mapper>
~~~~~~

~~~~~~
String resource = "sqlMapConfig.xml";
// 通过输入流读取配置文件
InputStream inputStream = Resources.getResourceAsStream(resource);

// 通过sqlSessionFactoryBuilder, 获取SqlSessonFactory
SqlSessoinFactory fac = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession session = fac.openSession();

// 查询用户
// 命名空间 + id
User user = session.selectOne("test.findUserById", 1);

// 插入数据
session.insert("test.insertUser", user);
sqlSession.commit(); // 提交事务

// 更新记录
session.update("test.updateUserById", user);
sqlSession.commit(); // 提交事务

// 删除记录
session.delete("test.deleteUserById", user);
sqlSession.commit(); // 提交事务

session.close();
~~~~~~

# API 介绍 #

## SqlSessionFactoryBuilder##

SqlSessionFacoty是通过SqlSessionFactoryBuilder进行创建，
SqlSessionFactoryBuilder只用于创建SqlSessionFactory，
可以当成一个工具类，在使用时随时拿来使用不需要特殊处理为共享对象。

## SqlSessionFactory ##

SqlSessionFactory是一个接口，
接口中定义了openSession的不同方式，
mybatis默认使用DefaultSqlSessionFactory作为接口实现，
SqlSessionFactory一但创建后可以重复使用，实际应用时通常设计为单例模式。

## SqlSession ##
SqlSession是一个接口，定义了数据库操作。

执行过程如下：
* 加载数据源等配置信息
`Environment environment = configuration.getEnvironment();`
* 创建数据库链接
* 创建事务对象
* 创建Executor，SqlSession所有操作都是通过Executor完成，mybatis源码如下：
~~~~~~
if (ExecutorType.BATCH == executorType) {
    executor = new BatchExecutor(this, transaction);
} else if (ExecutorType.REUSE == executorType) {
    executor = new ReuseExecutor(this, transaction);
} else {
    executor = new SimpleExecutor(this, transaction);
}
if (cacheEnabled) {
    executor = new CachingExecutor(executor, autoCommit);
}
~~~~~~
* SqlSession的实现类即DefaultSqlSession，此对象中对操作数据库实质上用的是 Executor

结论：每个线程都应该有它自己的SqlSession实例。
SqlSession的实例不能共享使用，它也是线程不安全的。
因此最佳的范围是请求或方法范围。
绝对不能将SqlSession实例的引用放在一个类的静态字段甚至是实例字段中。

## Namespace ##
命名空间除了对sql进行隔离，mybatis中对命名空间有特殊的作用，
用于定义mapper接口地址

## Mapper 接口 ##
// 原生实现
~~~~~~
public interface UserDao {
    public User findUserById(String id);
    public void inserUser(User user);
    public void pudateUserById(User user);
    public void deleteUserById(String id);
}
public class UserDaoImpl implements UserDao {
    private SqlSessionFactory fac;
    
    public User findUserById(String id){
        SqlSession session = fac.openSession();
        User u = session.selectOne("test.selectUserById", 1);
        session.close();
    }
    public void inserUser(User user){
        SqlSession session = fac.openSession();
        User u = session.insert("test.insertUser", user);
        session.commit();
        session.close();
    }
    public void pudateUserById(User user){
        SqlSession session = fac.openSession();
        session.update("test.updateUser", user);
        session.commit();
        session.close();
    }
    public void deleteUserById(String id){
        SqlSession session = fac.openSession();
        session.update("test.deleteUserById", i);
        session.commit();
        session.close();
    }
}
~~~~~~

第一个例子中，在访问sql映射文件中定义的sql时需要调用sqlSession的selectOne
方法，并将sql的位置(命名空间+id)和参数传递到selectOne方法中，
且第一个参数是一个长长的字符串，第二个参数是一个object对象，
这对于程序编写有很大的不方便，很多问题无法在编译阶段发现。
虽然上边对提出的面向接口编程问题进行解决，
但是dao实现方法中仍然是调用sqlSession的selectOne方法，且重复代码多。

解决方法:
~~~~~~
// 将 namespace 修改为 cn.mapper.UserMapper
public interface UserMapper {
    public User findUserById(String id);
    public void inserUser(User user);
    public void pudateUserById(User user);
    public void deleteUserById(String id);
}

public void testFindUserById {
    sqlSession sqlSession = sqlSessionFactory.openSession();
    // 指定mapper接口类型, mybatis生成一个代码对象实现mapper接口
    UserMapper userMapper = sqlSessioin.getMapper(UserMapper.class);
    userMapper.findUserById(1);
}

~~~~~~

* Mapper接口方法名和mapper.xml中定义的每个sql的id相同
* Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同
* Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同
* Mapper.xml文件中的namespace即是mapper接口的类路径。

# 配置详解 #

## properties ##
~~~~~~
# db.properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis
username=root
password=mysql
~~~~~~

~~~~~~
<properties resource="db.properties" />
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC" />
        <dataSource type="POOLED">
            <property name="driver" value="${driver}" />
            <property name="url" value="${url}" />
            <property name="username" value="${username}" />
            <property name="password" value="${password}" />
        </dataSource>
    </environment>
</environments>
~~~~~~

## settings ##

全局配置参数, 会影响到 mybatis 的运行行为

~~~~~~
<configuration>
    <settings>
        <setting name="cacheEnabled" value="false"/>
    </settings>
</configuration>
~~~~~~

## 别名 ##

| 别名    |  	映射的类型  |
|------------|
| _byte    |  	byte  |
| _long    |  	long  |
| _short    |  	short  |
| _int    |  	int  |
| _integer    |  	int  |
| _double    |  	double  |
| _float    |  	float  |
| _boolean    |  	boolean  |
| string    |  	String  |
| byte    |  	Byte  |
| long    |  	Long  |
| short    |  	Short  |
| int    |  	Integer  |
| integer    |  	Integer  |
| double    |  	Double  |
| float    |  	Float  |
| boolean    |  	Boolean  |
| date    |  	Date  |
| decimal    |  	BigDecimal  |
| bigdecimal    |  	BigDecimal  |

定义别名
~~~~~~
<typeAliases>
    <typeAlias alias="user" type="cn.itcast.mybatis.po.User"></typeAlias>
    <!-- 批量别名定义，扫描整个包下的类 -->
	<package name="cn.itcast.mybatis.po"/>
    <!-- 就可以使用 po.User -> user -->
</typeAliases>
~~~~~~

使用注解方式
~~~~~~
@Alias("user")
public class User{
}
~~~~~~


## 类型处理器 ##
将java类型和 sql 映射文件进行映射,
mybatis自带的类型处理器基本上满足日常需求，不需要单独定义。
~~~~~~
<!-- parameterType：指定输入数据类型为int，即向statement设置值 -->
<!-- resultType：指定输出数据类型为自定义User，即将resultset转为java对象 -->
<select id="selectUserById" parameterType="int" resultType="user">
    select * from user where id = #{id}
</select>

~~~~~~

## environments(环境集合属性对象) ##
MyBatis可以配置多种环境. 这会帮助你将SQL映射应用于多种数据库之中.

多数据库配置

~~~~~~
<environments default="development">
    <environment id="development_mysql">
        <transactionManager type="JDBC" />
        <dataSource type="POOLED">
            <property name="driver" value="${mysql.driver}" />
            <property name="url" value="${mysql.url}" />
            <property name="username" value="${mysql.username}" />
            <property name="password" value="${mysql.password}" />
        </dataSource>
    </environment>
        <environment id="development_oracle">
        <transactionManager type="JDBC" />
        <dataSource type="POOLED">
            <property name="driver" value="${oracle.driver}" />
            <property name="url" value="${oracle.url}" />
            <property name="username" value="${oracle.username}" />
            <property name="password" value="${oracle.password}" />
        </dataSource>
    </environment>
</environments>
~~~~~~

~~~~~~
mysql.driver=com.mysql.jdbc.Driver
mysql.url=jdbc:mysql://localhost:3306/mybatis
mysql.username=root
mysql.password=mysql

oracle.driver=oracle.jdbc.driver.OracleDriver
oracle.url=jdbc:oracle:thin:@127.0.0.1:1521:yycg
oracle.username=yycg
oracle.password=yycg
~~~~~~

~~~~~~
// 需要用到 oracle 数据库
sqlSessionFactory_oracle = new SqlSessionFactoryBuilder()
    .build(Resources.getResourceAsStream(resource), "development_oracle");
~~~~~~

~~~~~~
<mapper>
    <!-- 只对 mysql 有效 -->
    <select id="getNow" resultType="date">
        select now()
    </select>
    <!-- 只对 oracle 有效 -->
    <!-- 重新定义一个 getNow_Oracle的接口 -->
    <select id="getNow_Oracle" resultType="date">
        select sysdate from dual
    </select>
</mapper>
~~~~~~

可以用 databaseIdProvider 提供多数据库支持

### databaseIdProvider(数据库ID提供者) ###

~~~~~~
<!-- SqlMapConfig.xml, 定义 databaseIdProvider -->
<databaseIdProvider type="DB_VENDOR">
    <property name="SQL Server" value="sqlserver"/>
    <property name="DB2" value="db2"/>
    <!-- oracle的 databaseid 事务 oracle-->
    <property name="Oracle" value="oracle" />
    <property name="MySQL" value="mysql" />
</databaseIdProvider>

<mapper>
    <!-- 只对 mysql 有效 -->
    <select databaseid="mysql" id="getNow" resultType="date">
        select now()
    </select>
    <!-- 只对 oracle 有效 -->
    <select databaseid="oracle" id="getNow" resultType="date">
        select sysdate from dual
    </select>
</mapper>
~~~~~~


## 指定mapper文件位置 ##
~~~~~~
<!-- 指定mapper文件的配置 -->
<mappers>
    <!-- 使用相对于类路径的资源 -->
    <mapper resource="sqlmap/user.xml" />
    
    <!-- 使用完全限定路径 -->
    <mapper url="file:///D:\workspace_spingmvc\mybatis_01\config\sqlmap\user.xml" />
    
    <!-- 使用mapper接口类路径 -->
    <!-- 此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中 -->
    <mapper class="cn.itcast.mybatis.mapper.UserMapper"/>
    
    <!-- 注册指定包下的所有mapper接口 -->
    <!-- 此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中 -->
    <package name="cn.itcast.mybatis.mapper"/>
</mappers>
~~~~~~

## Mapper.xml ##


输入类型
~~~~~~
<select id="selectUserById"  parameterType="int" resultType="user">
    select * from user where id = #{id}
</select>

<!-- ${}只显示内容, 不管类型 -->
<!-- ${}和#{}不同，${}是将参数值不加修饰的拼在sql中，
     相当中用jdbc的statement拼接sql，使用${}不能防止sql注入，
     但是有时用${}会非常方便 -->
 <!--${}中只能写value -->
<select id="selectUserByName"  parameterType="int" resultType="user">
    select * from user where username like '${value}%'
</select>

<!-- 如果传递的是模型类 ${} 要写属性名  -->
<select id="selectUserByUser" parameterType="user" resultType="user">
    select * from user where id=#{id} and username like '%${username}%'
</select>

<!-- 传递hashmap综合查询用户信息 -->
<select id="selectUserByHashmap" parameterType="hashmap" resultType="user">
    select * from user where id=#{id} and username like '%${username}%'
</select>
~~~~~~


输出类型:
* 输出pojo对象和输出pojo列表在sql中定义的resultType是一样的。
* 返回单个pojo对象要保证sql查询出来的结果集为单条，
使用session.selectOne方法调用，mapper接口使用pojo对象作为方法返回值。
* 返回pojo列表表示查询出来的结果集可能为多条，
只能使用session.selectList方法调用，
mapper接口使用List<pojo>对象作为方法返回值。

~~~~~~
<!-- 获取用户信息列表, 返回hashmap -->
<!-- 输出pojo对象可以改用hashmap输出类型，
     将输出的字段名称作为map的key，value为字段值 -->
<!-- 输出hashmap类型必须保证sql查询出来的记录为单条 -->
<select id="selectUserByHashmap" parameterType="user" resultType="hashmap">
    select * from user where id=#{id} and sex=#{sex}
</select>

<!-- resultMap -->
<!-- 返回类型 Person{userid:int, name:String, addr:String} -->
<select id="selectUserByHashmap" parameterType="user" resultType="resultMapPerson">
    select * from user where id=#{id} and sex=#{sex}
</select>

<!-- 定义rsultMap -->
<resultMap type="person" id="resultMapPerson">
    <!-- 结果集的属性  -->
    <id property="userid" column="id"></id>
    <result property="name" column="username"></result>
    <result property="addr" column="address"></result>
</resultMap>
~~~~~~
