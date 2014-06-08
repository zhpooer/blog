title: 传智播客day34-Hibernate性能
date: 2014-05-21 14:20:20
tags:
---

# 检索策略 #
## 类级别检索 ##

通过session直接检索 某一个类 对应数据表数据
~~~~~~
session.load(Customer.class , 1) ;
session.createQuery("from Order");
~~~~~~

## 关联级别检索 ##

程序内部已经获得持久对象，通过对象引用关系，进行数据检索
~~~~~~
Customer c = session.load(Customer.class , 1) ; // 类级别
c.getOrders().size() ;  // 关联级别检索 
order.getCustomer().getName() ; // 关联级别检索
~~~~~~

# 类的检索策略 #
* 类级别可选的检索策略包括立即检索和延迟检索, 默认为延迟检索  （针对load方法 ）
* 类级别的检索策略可以通过 `<class>` 元素的 lazy 属性进行设置

类级别检索 get 、Query 默认使用立即检索策略

load 默认使用延迟检索， 在hbm文件 `<class>` 配置 lazy=false 使类级别检索变为立即检索

* lazy=false 后 load方法效果 和 get方法相同，使用 立即检索 

# 关联级别的检索策略 #
~~~~~~
c.getOrders().size() ;
order.getCustomer().getName();  // 属于关联级别的检索 
~~~~~~

## 多对多和一对多 ##
`<set>` 元素提供 fetch 和 lazy 两个属性 决定检索策略 
* fetch 属性 （select、subselect、join） ，主要决定SQL语句生成格式 
* lazy 属性 （false、true、extra）主要决定集合被初始化的时机

fetch 和 lazy 共有9种组合 
* fetch 属性为 join， lazy属性会被忽略， 生成SQL将采用迫切左外连接查询 (left outer join fetch )
    * SQL语句 左外连接，采用 立即检索
    * 使用Query对象查询数据时，需要自己编写hql语句，(`session.get`是Hibernate生成sql)
    fetch=join 无效果，关联集合将根据lazy 设置进行 加载 
* fetch 属性为 select ，将生成多条简单SQL查询 
        lazy = false 立即检索
		lazy = true  延迟检索
		lazy = extra 加强延迟检索 （及其懒惰，比延迟更加延迟）
* fetch 属性为 subselect ，将生成子查询的SQL语句
        lazy = false 立即检索
		lazy = true  延迟检索
		lazy = extra 加强延迟检索 （及其懒惰，比延迟更加延迟）
* lazy=false 立即检索，检索类级别数据时，关联级别数据也进行检索 
* lazy=true 延迟检索，检索类级别数据时，不会检索关联级别的数据，用到了再进行检索 
* lazy="extra" 及其懒惰，当程序第一次访问 order 属性的 size(), contains() 和 isEmpty() 方法时, Hibernate 不会初始化 orders 集合类的实例 ，例如 查询size时，生成select count(*)  

结论: `session.load/session.get` ， fetch=join 生成迫切左外连接， lazy被忽略,
`session.createQuery(hql).list()`  将忽略 fetch=join， lazy 将重新产生效果 	 

## 多对一和一对一 ##
`<many-to-one>` 元素也有一个 lazy 属性和 fetch 属性,
fetch 决定SQL语句格式， lazy决定数据加载时间
* fetch取值 join、 select
* lazy取值 false、 proxy、no-proxy(不讲解)

fetch 和 lazy 共有4种组合 
* fetch 属性为 join, lazy属性会被忽略,
生成SQL将采用迫切左外连接查询(left outer join fetch )
* fetch 属性为 select， 产生多条SQL 
        lazy=false  立即检索
	    lazy=proxy  有关联对象类级别检索策略决定立即检索 或者 延迟检索 

Query的list 会忽略 fetch="join", lazy重新起作用

结论： 开发中能延迟都延迟，必须立即的 才立即的 

# 批量检索 #
解决n+1查询检索问题
1. Customer 一方设置检索, `<set batch-size=""/>` 设置批量检索
~~~~~~java:
// 查询每个客户的订单数
List<Customer> customers = session.createQuery("from Customers").list();
// 查询每个客户订单数, 一次查询所有客户,
// 每个客户订单数产生单独SQL查询, 如果有N个客户
// `<set batch-size="3"/>` 每次查三个
for(Customer c:customers){ // n+1
    customer.getOrders().size();
}

// 用订单查客户
List<Order> orders = session.createQuery("from Order").list();
// 产生 n+1 条缓存, n 为客户数
// 配置批量检索, Customer.hbm.xml, `<class batch-size=3>`
for(o <- orders) {
    o.getCustomer();
}
~~~~~~


# session #
~~~~~~
public void doService{
    // 如果 两个dao都要 begintransaction(), 如何管理?
    dao1.doSome();  
    dao2.doSome();
}
~~~~~~
## session 的产生 ##
不同java虚拟机的调用不考虑
* `sessionFactory.openSession()`, 每次都打开新的会话(session)
* `sessionFactory.getCurrentSession()`, 线程绑定一个session

hibernate 提供三种管理Session的方式 
* Session 对象的生命周期与本地线程绑定 (与ThreadLocal绑定)
  * 在hibernate配置文件中 配置 `hibernate.current_session_context_class=thread` 
* Session 对象的生命周期与 JTA 事务绑定 (分布式事务管理)
  * session面向多个数据库程序
* Hibernate 委托程序管理 Session 对象的生命周期,
在程序中获得Session对象，使用Session， 需要手动session.close() 



## 使用 getCurrentSession() ##
~~~~~~
<!-- 告诉hibernate, session由当前线程产生  -->
<property name="current_session_context_class"> thread</property>
~~~~~~
~~~~~~
<!-- CRUD 操作必须在事务的环境下运行 -->
<!-- session和事务绑定在一起 -->
Session s = session.getCurrentSession();
Transaction trans = session.beginTransaction(); // 如果不写会报错
Classes classes = session.get(Classes.class, 1L);
trans.commit();
// 可以不写, 因为session和事务绑定在一起, 在事务提交时会自动关闭
session.close();
~~~~~~

## session 的缓存 ##
客户端先从内存中查找数据, 再从数据库获取数据

### 一级缓存 ###
* 一级缓存的生命周期, 就是session的生命周期
* 一级缓存的存放的数据都是私有数据
  * 把session存放在threadlocal中, 不同的线程不能访问

~~~~~~
// 只发出一条语句
// session.get 方法把数据放在一级缓存中, load, save, update也一样
session.get(Classes.class, 1L);
session.get(Classes.class, 1L);

session.evict(classes); // 从一级缓存中清空
~~~~~~

~~~~~~
Classes classes = session.get(Classes.class, 1L);
session.clear(); // 如果不加这句话, 两个不同对象, 相同的id值, 得把缓存中的先清空
Classes c2 = new Classes();
c2.setCid(1L);
session.update(c2);
~~~~~~
### 数据库和缓存的交互 ###

~~~~~~
// 把数据库的缓存刷新的缓存中
Classes classes = session.get(Classes.class, 1L);
classes.setCname("66");
session.refresh(classes); // 查询classes, 重置classes, 把数据库中的数据同步到缓存中

// 把把缓存中的数据刷到数据库中
session.flush(); // 检查缓存内部, 对比快照, 执行数据库操作
~~~~~~

批量操作
~~~~~~
for(i <- 1 to 10000) {
    Classes c = new Classes();
    c.setCname(i);
    session.save(c);
    // 不加下面, 会内存溢出
    if(i%50==0) {
        session.flush(); // 只刷, 不清空
        session.clear();
    }
}
~~~~~~

## 二级缓存 ##

适用场合: 很少被修改，不是很重要，允许偶尔的并发问题，
放入二级缓存 考虑因素(二级缓存 监控，是否采用二级缓存主要参考指标)

hibernate 本身并没有提供二级缓存的解决方案, 依赖第三方供应商完成
* ehcache: 主要学习，支持本地缓存，支持分布式缓存
* oschace
* jbosscache
* swamcache

二级缓存并发策略: 
* transactional: 提供Repeatable Read事务隔离级别, 缓存支持事务, 发生异常的时候,缓存也能够回滚
* read-write: 提供Read Committed事务隔离级别， 更新缓存的时候会锁定缓存中的数据
* nonstrict-read-write ： 导致脏读， 很少使用 
* read-only： 数据不允许修改 ，只能查询 

### 二级缓存配置 ###
* 二级缓存存在 sessionFaction 中
* 生命周期和 sessionFaction 保持一致
* 使用步骤
  1. 开启二级缓存
~~~~~~
<!-- 开启二级缓存 -->
<property name="cache.use_second_level_cache"> true </property>
<!-- 导入ehcache包  -->
<property name="cache.provider_class">
    org.hibernate.cache.EhCacheProvider
</property>
~~~~~~
  2. 让一个对象进入到二级缓存中
~~~~~~
<!-- 方式一 在配置文件中 -->
<class-cache usage="read-only" class="Classes"> </class-cache>
<collection-cache usage="read-write" class="cn.Order"/>
<!-- 客户关联的订单 -->
<collection-cache usage="read-write" class="Customer.Order"/>
<!-- 方式二 在映射文件(*.hbm.xml)中 -->
<!-- 缓存策略 -->
<!-- 在类下面配置, 或在集合下面配置 -->
<cache usage="read-only|read-write|transaction|nonstrict-read-write"> </cache>
~~~~~~
  3. ehcache配置
~~~~~~
<ehcache>
    <diskStore path=""></diskStore> <!-- 缓存在磁盘的目录 -->
    <!-- 缓存属性配置, 全局default 对所有缓存都有效 -->
    <defaultCache maxElementsInMemory="" <!-- 内存中最大数量,超过数量, 内存 -->
                  overflowToDisk="" <!-- 是否保存在硬盘 -->
                  external="false"  <!-- 缓存是否永久 -->
                  maxElementsOnDisk="" <!-- 硬盘缓存最大对象数量 -->
                  >
    </defaultCache>
    <!-- 设置二级缓存的硬盘保存策略 , 自定义设置, 针对 ClassName 有效-->
    <Cache name="ClassName"
        maxElementsInMemory=""
        overflowToDisk="true"
        maxElementsOnDisk="">
    </Cache>
</ehcache>
~~~~~~

### 类缓存区域 ###
从二级缓存区返回数据地址都是不同的(散装数据), 
每次查询二级缓存，都是将散装数据 构造为一个新的对象
~~~~~~
session.get(Customer.class, 1); 
transaction.commit();
session.get(Customer.class, 1);// 地址与上一次不同
transaction.commit();

// 不能读取二级缓存数据, 但是会写入二级缓存数据
session.createQuery("from Customer").list();
transaction.commit();
session.createQuery("from Customer").list(); // 会直接查数据库
transaction.commit();
~~~~~~
get/load 方法都可以 读取二级缓存的数据, 
Query的list方法只能存，不能取

### 集合缓存区 ###

~~~~~~
Customer customer = session.get(Customer.class, 1);
customer.getOrders().size();
transaction.commit();
// collection-cache 要设置
// 如果去掉 class-cache usage="read-write" class="cn.itcast.domain.Order" 将会继续从数据库查询
// 因为 集合缓存区, 缓存的是集合的id, 然后才会从类缓存区查找
Customer customer = session.get(Customer.class, 1);
customer.getOrders().size();
~~~~~~

> 一级缓存操作会同步到二级缓存

### 更新时间戳区域 ###
作用：记录数据最后更新时间，确保缓存数据是有效的

更新时间戳其余，记录数据最后更新时间，在使用二级缓存时，比较缓存时间t1 与 更新时间 t2 ，
如果 t2 > t1 丢弃原来缓存数据，重新查询缓存

### Qeury的iterate()方法###
Query的`iterate` 方法返回 只有OID 代理对象
* 产生查询id的SQL 语句 `select id from orders;`
   
~~~~~~
// 访问每个 具体数据时，再生成SQL查询 N+1
// batch-size 无法优化，
// 可以使用 二级缓存优化, 先执行 session.query().list(), 把数据放入二级缓存
Query q = session.createQuery("from Customer");
Iterator i = q.iterate();
// 每次迭代都从数据库或者二级缓存拿
while(i.hasnext()){
}
~~~~~~


### 查询缓存(QueryCache) ###
可以说是三级缓存

二级缓存缓存的数据都是类对象数据, 数据都缓存在*类缓冲区*,
二级缓存缓存PO类对象，条件(key)是id

如果查询条件不是id查询, 缓存数据不是PO类完整对象, 不适合使用二级缓存

~~~~~~
<!-- 先配置二级缓存 -->
<!-- 启用查询缓存  -->
<property name="hibernate.cache.use_query_cache">true</property>
~~~~~~
~~~~~~
// key 是 sql 语句
Query q = session.createQuery("from Classes");
query.setCacheable(true); // classes里的所有数据要往查询缓存中存放
query.list(); // 查询, 并放入二级缓存
session.close();

Query q = session.createQuery("from Classes");
query.setCacheable(true); // classes在二级缓存中查询, 不加会查询数据库
query.list(); // 查询

Query q = session.createQuery("from Classes where id=?");
query.setCacheable(true); 
query.list(); // 查询, 但是会从数据库查, 因为语句变了
~~~~~~

### 二级缓存的性能监控 ###
**没有检测性能的优化是毫无意义的** 

SessionFactory 提供二级缓存监控方法，用来获得二级缓存命中次数 
*  `Statistics getStatistics()`  返回  Statistics 对象

Statistics 对象提供
* `long getQueryCacheHitCount()` 获取查询缓存命中次数
* `long getQueryCacheMissCount()`  获取查询缓存丢失次数
* `long getSecondLevelCacheHitCount()` 获取二级缓存命中次数
* `long getSecondLevelCacheMissCount()` 获取二级缓存丢失次数 

二级缓存监控 设置hibernate属性 
* 在配置期间，将 `hibernate.generate_statistics` 设置为 true或 false

命中率:`hitCount/(hitCount + missCount)` =


# 事务并发处理 #
事务四个特性 ACID: 原子性、一致性、隔离性、持久性 

1、隔离性引发问题： 脏读、不可重复读、虚读 、丢失更新(lost update)
* 脏读: 一个事务 读取 另一个事务 未提交的数据 
* 不可重复读: 一个事务中 连续读取 两次， 第二次读取另一个事务 已经提交 update修改数据(数据改变)
* 虚读: 一个事务 读取 另一个事务 已经提交 插入数据(记录条数改变)
* 丢失更新: 两个事务同时修改数据，后提交事务覆盖了先提交事务的结果 

2、数据库为了解决事务的隔离性问题，提供四种隔离级别（不是所有数据库都支持这四种级别）
* READ_UNCOMMITED : 读取未提交，引发所有隔离问题
* READ_COMMITTED : 读已提交，阻止脏读，发生不可重复读和虚读
* REPEATABLE_READ : 重复读 ，阻止脏读、不可重复读，发生虚读
* SERIALIZABLE : 串行处理，不允许两个事务 同时操作一个目标数据，根本不存在并发，不存在并发问题  （效率低下）

企业开发中主要 READ_COMMITTED （Oracle默认级别）、REPEATABLE_READ （MySQL默认级别）

设置hibernate事务隔离级别`hibernate.connection.isolation = 4`
* 1—Read uncommitted isolation
* 2—Read committed isolation
* 4—Repeatable read isolation
* 8—Serializable isolation

~~~~~~
<property name="hibernate.connection.isolation">2</property>
~~~~~~

# 解决丢失更新 #
## 悲观锁 ##
悲观锁： 采用数据库内部锁机制，在一个事务操作数据时，
为数据加锁，另一个事务无法操作 
* 排它锁 （写锁），数据库中每张表只能添加一个排它锁，
排它锁与其他锁互斥 
* 在修改数据时，自动添加排它锁 
* 在查询数据时 添加排它锁
`select * from customers for update;`

hibernate中使用悲观锁
~~~~~~
Customer customer = (Customer) session.load(Customer.class, 1,LockMode.UPGRADE);
Customer customer = (Customer) session.load(Customer.class, 1,LockMode.UPGRADE);
~~~~~~

* 悲观锁解决丢失更新，效率问题 ， 数据不能同时修改 

## 乐观锁 ##
与数据库锁无关，在数据表中为数据添加 版本字段，每次数据修改都会导致版本号+1 

hibernate 为Customer表 添加版本字段 
* 在customer类 添加 `private Integer version;` 版本字段
* 在Customer.hbm.xml 定义版本字段
`<version name="version"></version>`
