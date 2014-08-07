title: 传智播客day58-Mybatis提高
date: 2014-07-16 09:09:13
tags:
- 传智播客
---

~~~~~~
public class User{
    id: Int;
    username: String;
    birthday: Date;
    sex: Int;
    address: String;
    detail: String;
    score: Float;
    orders:List;
}
public class Order {
    id:Int;
    user:User;
    orderNumber:String; // 订单号
    orderDetails: List;
}
public class OrderDetail {
    id:Int;
    order:Order;
    item:Item;
    num:Int;n
    itemPrice:Float;
}

public class Item {
    name:String;
    price:Float;
    detail:String;
}
~~~~~~

~~~~~~
<!-- 一对一查询 -->
<!-- 订单用户关联 -->
<select id="findOrdersUsers" resultMap="userOrderMap">
    select orders.*, user.username, user.sex, user.address
    from orders, user where orders.user_id = user.id;
</select>
<resultMap id="userOrderMap" type="order">
    <!-- order信息  -->
    <id property="id" column="id"/>
    <result property="user_id" column="user_id"> </result>
    <!-- 用户信息 -->
    <!-- 将用户信息封装成一个user对象 -->
    <association property="user" javaType="po.User">
        <!-- 指定user表的id  -->
        <id property="id" column="user_id"></id>
        <result property="username" column="username"></result>
        <result property="sex" column="sex"></result>
        <result property="address" column="address"></result>
    </association>
</resultMap>

<!-- 一对多查询 -->
<!-- 订单 及 订单明细 -->
<select id="" resultMap="userOrdersDetailsMap">
    select orders.*, orderdetial.id orderdetail_id,
    orderdetail.item_id, orderdetail.item_num, orderdetail.item_price
    from orders, orderdetail where orders.id=orderdetail.orders_id;
</select>

<resultMap type="order" id="userOrdersDetailsMap">
    <!--  订单id -->
    <!-- id是指定结果集的主键, 指定mybatis返回最终list的结果集主键 -->
    <id property="id" column="id"></id>
    <result property="orderNumber" column="order_number"></result>
    <!-- 集合中用 orderDetail -->
    <collection property="orderdetails" ofType="orderDetail">
        <id property="id" column="orderdetail_id"> </id>
        <id property="itemNum" column="item_num"> </id>
        <id property="itemPrice" column="item_price"> </id>
        <id property="itemDetail" column="item_detail"> </id>
    </collection>
    <!-- 订单明细信息 -->
</resultMap>

<!-- 三表关联, 使用继承 -->
<!-- 订单 用户 订单明细 -->
<select id="" resultMap="userOrdersDetailsMap">
select orders.*, user.username, user.sex, user.address,
       orderdetial.id orderdetail_id,
       orderdetail.item_id, orderdetail.item_num, orderdetail.item_price
       from orders, user, orderdetail
       where orders.id=orderdetail.orders_id and  orders.user_id = user.id;
</select>

<resultMap type="order" id="userOrdersDetailsMap" extends="userOrdersMap">
    <!--  订单id 继承 订单用户信息 -->
    <!-- 集合中用 orderDetail -->
    <collection property="orderdetails" ofType="orderDetail">
        <id property="id" column="orderdetail_id"> </id>
        <id property="itemNum" column="item_num"> </id>
        <id property="itemPrice" column="item_price"> </id>
        <id property="itemDetail" column="item_detail"> </id>
    </collection>
</resultMap>

<!-- 查询用户及用户下的订单 -->
<select id="findUserInOrders" resultMap="">
    select orders.*
    user.username,
    user.sex,
    user.address
    from user, order where user.id = order.user_id
</select>

<resultMap type="user" id="userInOrdersMap">
    <id property="username" column="username"></id>
    <result property="id" column="user_id"></result>
    <!-- 订单信息 -->
    <collection property="orders" ofType="order">
        <result property="order_number" column="order_number"/>
        <result property="id" column="id"/>
    </collection>
</resultMap>

<!-- 查询用户及用户下的订单, 和订单明细 -->
<!-- 查询用户及用户下的订单 -->
<select id="findUserInOrderDetailInOrder" resultMap="">
    TODO: sql
</select>
<!-- 一对多 嵌套 一对多 -->
<resultMap type="user" id="userInOrdersMap">
    <id property="username" column="username"></id>
    <result property="id" column="user_id"></result>
    <!-- 订单信息 -->
    <collection property="orders" ofType="order">
        <result property="order_number" column="order_number"/>
        <result property="id" column="id"/>
        <collection property="orderdetails" ofType="orderDetail">
            ...
        </collection>
    </collection>
</resultMap>

<!-- 多对多 -->
<!-- 查询用户下的订单及订单下的明细商品信息 -->
<select id="findUserOrdersItemList" resultMap="userOrdersItemListMap">
  select orders.*
    user.username,
    user.sex,
    user.address,
    orderdetial.id orderdetail_id,
    orderdetail.item_id,
    orderdetail.item_num,
    orderdetial.item_price,
    items.item_name
    items.item_price item_price_,
    items.item_detail
  from orders, user, orderdetail,items
  where orders.user_id = user.id
    and orders.id=orderdetail.orders_id
    and items.id = orderdetail.item_id
</select>

<resultMap type="user" id="userOrdersItemListMap">
    <id property="username" column="username"></id>
    <result property="id" column="user_id"></result>
    <!-- 订单信息 -->
    <collection property="orders" ofType="order">
        <result property="order_number" column="order_number"/>
        <result property="id" column="id"/>
        <collection property="orderdetails" ofType="orderDetail">
            ...
            <association property="items" javaType="items">
                <id property="id" column="item_id"></id>
                <result property="itemName" column="item_name"></result>
                <result property="itemPrice" column="item_price"></result>
                <result property="itemDetail" column="item_detail"></result>
            </association>
        </collection>
    </collection>
</resultMap>

~~~~~~

# 延迟加载 #
~~~~~~
select orders.*
    user.username,
    user.sex,
    user.address,
    orderdetial.id orderdetail_id,
    orderdetail.item_id,
    orderdetail.item_num,
    orderdetial.item_price,
--    items.item_name
--    items.item_price item_price_,
--     items.item_detail,
  (select items.item_name from items where orderdetail.item_id = items.id) item_name,
  (select items.item_price from items where orderdetail.item_id = items.id) item_price,
  (select items.item_detail from items where orderdetail.item_id = items.id) item_detail
  from orders, user, orderdetail
  where orders.user_id = user.id
    and orders.id=orderdetail.orders_id
    and items.id = orderdetail.item_id
-- 解决子查询问题, 如下>    
~~~~~~

    
~~~~~~
<!-- select 子查询, column 根据 column 进行子查询 -->
<!-- 可以通过设置 懒加载 来延迟查询, 一对一 不推荐使用 -->
<association property="items" javaType="items"
      select="findItemsById" column="item_id">
    <!-- 使用子查询, 可以通过 findItemsById 查询 -->
    <!-- 一一对应? TODO: 属性和值的对应关系如何设置!!!! -->
    <!-- <id property="id" column="item_id"></id> -->
    <!-- <result property="itemName" column="item_name"></result> -->
    <!-- <result property="itemPrice" column="item_price"></result> -->
    <!-- <result property="itemDetail" column="item_detail"></result> -->
</association>

<select id="findItemsById" parameterType="int" resultType="items">
    select * from items where id=#{id}
</select>
~~~~~~
开启全局延迟加载
~~~~~~
<settings>
    <!-- 全局性设置懒加载。如果设为‘false’，则所有相关联的都会被初始化加载 -->
    <!-- 默认为false -->
    <setting name="lazyLoadingEnable" value="true"></setting>
    <!-- 当设置为‘true’的时候，懒加载的对象可能被任何懒属性全部加载。
         否则，每个属性都按需加载。 -->
    <!-- 默认为true -->
    <setting name="aggressiveLazyLoading" value="false"></setting>
</settings>

~~~~~~

# 动态 SQL #

## if ##

~~~~~~
<select id="findUserInOrderDetailInOrder" parameterType="user" resultMap="">
  select orders.*
      user.username,
      user.sex,
      user.address,
      orderdetial.id orderdetail_id,
      orderdetail.item_id,
      orderdetail.item_num,
      orderdetial.item_price,
    from orders, user, orderdetail
    where orders.user_id = user.id
      and orders.id=orderdetail.orders_id
    
    <!-- 注意不是 user.id -->
    <if test="id!=null and id!=''">
        AND user.id=#{id}
    </if>
</select>
~~~~~~


## where ##

~~~~~~
<select id="findUserList" parameterType="user" resultType="user">
    select * from user
    <!-- 解决如果条件都不符合, 会多出一个 where -->
    <where>
        <if test="id!=null and id!=''">
            and user.id=#{id}
        </if>
    </where>
</select>
~~~~~~

## foreach ##

向sql传递数组或List，mybatis使用foreach解析

~~~~~~
<!-- User{ids:Array[Int]} -->
<select id="findUserList" parameterType="user" resultType="user">
    select * from user
    <!-- 解决如果条件都不符合, 会多出一个 where -->
    <where>
        <!-- ids是集合, item 是集合中的项目 -->
        <!-- 生成 user.id in (1, 2, 3) -->
        <foreach collection="ids" item="items" open="user.id in(" close=")" separator=",">
            #{items}
        </foreach>
        
        <!-- 第二种写法  -->
        <!-- 生成 and user.id=1 or user.id=1 -->
        <foreach collection="ids" item="items" open="and (" close=")" separator="or">
            user.id=#{items}
        </foreach>

        <foreach collection="array" index="index" item="item"
             open="and id in(" separator="," close=")" >
		    #{item.id} 
		 </foreach>
    </where>
</select>
~~~~~~

## set ##

TODO

# Sql 片段 #

抽取重用的 sql 片段
~~~~~~
<sql id="user_query">
    <if test="id!=null and id!=''">
      and user.id=#{id}
    </if>
</sql>

<select>
    ...
    <where>
        <include refid="user_query"> </include>
    </where>
</select>
<!-- 如果引用其他mapper.xml的sql片段, 需要及啊家辉命名空间-->
<include refid="namespace.sql片段名">
</include>
~~~~~~

# 缓存 #

## 一级缓存 ##

~~~~~~
SqlSession sqlSession = sqlSessionFactory.openSession();
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
// 从数据库中取出来, 写入缓存
user.findUserById(1);
// 同一个sql中, 相同的两次次查询发出一个sql
user.findUserById(1);

// 第二次测试
// open new session
user.findUserById(1);
// 任何数据库更新, 删除, 插入之后都会将 sqlSession 的缓存清空
// 更新
User user_update = new User;
user_update.setId(1);
user_update.setUsername("");
// update 会清除本地缓存
userMapper.updateUserById(user_update);
sqlSession.commit();

user.findUserById(1);
~~~~~~

## 二级缓存 ##

一个项目中肯定会存在很多共用的查询数据，对于这一部分的数据，没必要
每一个用户访问时都去查询数据库，因此配置二级缓存将是非常必要的

Mybatis 的二级缓存即查询缓存, 它的作用域是一个 mapper 的 namespace,
即在同一个 namespace 中查询 sql 可以从缓存中获取数据

二级缓存是可以跨 SqlSession 的

~~~~~~
<!-- 开启二级缓存 -->
<!-- 全局设置 -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>

<!-- 开启mapper的二级缓存 -->
<mapper>
    <cache ></cache>
</mapper>

~~~~~~
二级缓存测试
~~~~~~
SqlSession session1 = openSession();
SqlSession session2 = openSession();

UserMapper userMapper1 = session1.getMapper(UserMapper.class);
UserMapper userMapper2 = session2.getMapper(UserMapper.class);

userMapper1.findUserList();
session1.close();
// 必须先要关闭 session1, 关闭session的时候, session1的缓存会被写入到二级缓存
// 如果开启二级缓存, 那么不会再发出sql语句
userMapper2.findUserList();


// 测试二:
SqlSession session1 = openSession();
SqlSession session2 = openSession();
SqlSession session3 = openSession();

UserMapper userMapper1 = session1.getMapper(UserMapper.class);
UserMapper userMapper2 = session2.getMapper(UserMapper.class);
UserMapper userMapper3 = session2.getMapper(UserMapper.class);

User user1 = userMapper1.findUserById(1);
session1.close();

user1.setUsername("changed");
session2.updateUserById(user1);
session2.commit();

// 如果 <update id="updateUserById" flushcache="true">, 那么更新时, 会刷新二级缓存
session3.findUserById(1);
~~~~~~

### 配置二级缓存 ###
~~~~~~
<!-- 默认60秒刷新一次, 存数结果对象或列表 512个引用,
     返回的对象认为是只读的 -->
<!-- LRU 最近最少使用的:移除最长时间不被使用的对象。 -->
<!-- FIFO 先进先出:按对象进入缓存的顺序来移除它们。 -->
<!-- SOFT 软引用:移除基于垃圾回收器状态和软引用规则的对象。 -->
<!-- WEAK 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。      -->
<!-- size 默认值是1024, readOnly 默认是false -->
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
~~~~~~

### Ehcache 配置 ###

`ehcache.xml` defaultCache 配置说明
* maxElementsInMemory 内存中最大缓存对象数.
当超过最大对象数的时候,ehcache会按指定的策略去清理内存
* eternal 缓存对象是否永久有效,一但设置了,timeout将不起作用.
* timeToIdleSeconds 设置Element在失效前的允许闲置时间.仅当element不是永久有效时使用,
可选属性,默认值是0,也就是可闲置时间无穷大.
* timeToLiveSeconds：设置Element在失效前允许存活时间.
最大时间介于创建时间和失效时间之间.
仅当element是永久有效时使用,默认是0.,也就是element存活时间无穷大.
* overflowToDisk 配置此属性,当内存中Element数量达到maxElementsInMemory时,
Ehcache将会Element写到磁盘中.
* diskSpoolBufferSizeMB 这个参数设置DiskStore(磁盘缓存)的缓存区大小.
默认是30MB.每个Cache都应该有自己的一个缓冲区.
* maxElementsOnDisk 磁盘中最大缓存对象数,若是0表示无穷大.
* diskPersistent 是否在重启服务的时候清楚磁盘上的缓存数据.
true不清除.
* diskExpiryThreadIntervalSeconds 磁盘失效线程运行时间间隔.
* memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时,
Ehcache将会根据指定的策略去清理内存.
默认策略是LRU(最近最少使用).
你可以设置为FIFO(先进先出)或是LFU(较少使用).

~~~~~~
<!-- 这样设置 echcache -->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
~~~~~~

# Mybatis 和 SpringMVC 整合 #
导入 `mybatis-spring1.2.2.jar`

~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
    http://www.springframework.org/schema/mvc 
    http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd 
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context-3.1.xsd 
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-3.1.xsd 
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-3.1.xsd ">

    <!-- 引用配置文件 -->
    <!-- 加载数据源配置文件 -->
    <context:property-placeholder location="classpath:db.properties" />  
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
           destroy-method="close">
        <property name="driverClassName" value="${mysql.driver}" />
        <property name="url" value="${mysql.url}" />
        <property name="username" value="${mysql.username}" />
        <property name="password" value="${mysql.password}" />
        <property name="maxActive" value="30" />
        <property name="maxIdle" value="5" />
    </bean>
</beans>
~~~~~~

applicationContext-dao.xml
~~~~~~
<!-- SqlSessioniFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="configLocation" value="classpath:mybatic/SqlMapConfig.xml"></property>
</bean>
~~~~~~

~~~~~~
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="db.properties" />
    <!-- 全局参数配置  -->	
    <settings>
        <!--开启全局性设置懒加载 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!-- 设置为按需加载 -->
        <setting name="aggressiveLazyLoading" value="false"/>
        <!-- 开启二级缓存 -->
        <setting  name="cacheEnabled" value="true"/>
    </settings>
    <!-- 定义别名 -->
    <typeAliases>
        <!-- <typeAlias alias="user" type="cn.itcast.mybatis.po.User" /> -->
        <package name="cn.itcast.mybatis.po"/>
        <!-- <typeAlias alias="" type=""/> -->
    </typeAliases>
	
    <!-- 定义databaseIdProvider -->
    <databaseIdProvider type="DB_VENDOR">
        <property name="SQL Server" value="sqlserver"/>
        <property name="DB2" value="db2"/>
        <property name="Oracle" value="oracle" /><!-- oracle的databaseid为"oracle" -->
        <property name="MySQL" value="mysql" /><!-- mysql的databaseid为"mysql"，此名称和mapper.xml文件中下定义databaseId一致 -->
    </databaseIdProvider>
	
    <!-- 指定sql映射文件 -->
    <mappers>
        <!-- 指定mapper映射文件的位置 
             <mapper resource="sqlmap/User.xml" /> -->
        <!-- 
          通过class指定mapper接口，必须保证mapper接口名称和mapper映射文件名称相同，且放在同一个目录中
        <mapper class="cn.itcast.mybatis.mapper.UserMapper"/> -->
        <!--
          通过package指定扫描mapper包的路径 ，mybatis自动将此包下的mappre接口与mapper映射文件绑定
          必须保证mapper接口名称和mapper映射文件名称相同，且放在同一个目录中
        -->
        <package name="cn.itcast.mybatis.mapper"/>
    </mappers>
</configuration>
~~~~~~

## 方式一 编写 dao 方式 ##
~~~~~~
// <bean id=userDao class=UserDaoImpl>
public class UserDaoImpl extends SqlSessionDaoSupport{
}
~~~~~~

## 方式二 mapper bean ##

~~~~~~
<!-- 使用工厂bean -->
<bean id="userMapper" class="org.mybatis.pring.mapper.MapperFactoryBean">
    <property name="mapperInterface" value="UserMapper"></property>
    <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
</bean>
~~~~~~

## 方式三 mapper 注解 ##

mapper.xml文件编写，注意：
* mapper.xml中的namespace为mapper接口的地址
* mapper接口中的方法名和mapper.xml中的定义的statement的id保持一致
* 如果将mapper.xml和mapper接口的名称保持一致则不用在 sqlMapConfig.xml 中进行配置

~~~~~~
<!-- mapper 扫描器 -->
<!-- 对象名是 bean 的类名的头字母小写  -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="mapper接口包地址"></property>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/> 
</bean>
~~~~~~
