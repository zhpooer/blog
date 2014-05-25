title: 传智播客day34-Hibernate性能
date: 2014-05-21 14:20:20
tags:
---
# 懒加载 #
懒加载主要解决一个问题: 类, 集合, many-to-one 在什么时候发出sql语句, 加载数据
## 类的懒加载 ##
* 利用session的load可以产生代理对象
* 在session.load 方法执行的时候并不发出sql语句
* 在得到一般属性的时候发出sql语句
* 只针对一般属性有效果, 针对标示符(id)没有效果
~~~~~~
// 得到一个代理对象
Classes c = (Classes) session.load(Classes.class, 1L);
c.getCname(); // 发出sql语句
c.getCid();// 标识符, 不会发出
session.close();
c.getDescription();  // 如果还没有初始化, 抛错 could not initialize proxy, no session
~~~~~~
## 集合的懒加载 ##
lazy 的三个变量:
* false: 当session.get时, 集合就被加载了
* true: 在集合遍历时才被加载
* extra: 针对集合做count, min, max, sum等操作

~~~~~~
<!-- 设置集合懒加载, 默认为 true,
     extra 更进一步的懒加载中, 如 students.size(), 会执行 select count()...-->
<set lazy="false|true|extra"></set>
~~~~~~
~~~~~~
// 执行 sql 生成 Classes 对象
// 如果 set.lazy=false, 会马上加载集合对象
Classes c = session.get(Classes.class, 1L); 
Set students = classes.getStudents(); 
for(Student s:students) {  // 执行sql, 生成 真实 students 对象(集合懒加载)
    s.getSname();
}
~~~~~~
~~~~~~
Classes c = session.load(Classes.class, 1L);
Set students = classes.getStudents(); // 执行 sql 生成 Classes 对象
for(Student s:students) {  // 执行sql, 生成 真实 students 对象(集合懒加载)
    s.getSname();
}
~~~~~~

## 单端关联的懒加载(多对一) ##
根据多的端加载一的一端, 就一个数据, 所以无所谓
~~~~~~
<!-- 默认值 no-proxy -->
<many-to-one lazy="false|no-proxy|proxy">
</many-to-one>
~~~~~~
# 抓取策略 #
抓取策略, 由一个对象如何抓取集合的策略(主要是Set集合如何提取数据)
1. 主要是研究 `set` 集合如何提取数据
2. 在 Classes.hbm.xml 文件中
~~~~~~
<!-- 外连接, 默认, 子查询 -->
<set fetch="join|fetch|subselect"></set>
~~~~~~

~~~~~~
// 默认 select, 如果学生有N个,要查询 N+1 次, N+1 查询
/* 如果 fetch=suselect, 会翻译成
*        select student.* from student where id in (select cid from classes)
*  先查询 Classes, 在查询 students, 两条语句 
*/

List cList = session.createQuery("from classes").list();
for(Classes classes:cList) {
    Set students = classes.getStudents();
    for(Student s: students) {
        println(s.getSname());
    }
}


/* 如果 fetch=join
* 发出语句变成一句, class left join student
* 如果把需求分析翻译成sql语句, 存在子查询, 用该策略不起作用(如上面的例子)
*/
Classes classes = session.get(Classes.class, 1L);
jSet students = classes.getStudents();
for(Student s: students) {
    println(s.getSname());
}
~~~~~~

# 懒加载和抓取策略的结合 #
| fetch | lazy | sql          |  sql的执行时机                            |  说明 |
|-------------------------|
| join  | false | 存在子查询   | 当查询classes时, classes和student全部查询 | join 没有用 |
| join  | true  | 存在子查询   | 当遍历student时查询 student               | join 没有用 | 
| join  | true  | 不是子查询   | 在session.get(classes)时, 全部查询        | lazy 没有用 |
| subselect| true/false| 存在子查询 | 发出两条语句 | 如果lazy为false, 在一开始就发出 |
| select | true/false | | 发出 n+1 条语句 | 同上 |

主要先看是不是子查询, 再看如果需要一下子全部加载, 就用`join`

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
二级缓存, 存放共有数据

适用场合: 数据不能频繁更新, 数据能公开, 私密性不是很强

hibernate 本身并没有提供二级缓存的解决方案, 依赖第三方供应商完成
* ehcache
* oschace
* jbosscache
* swamcache

二级缓存操作:
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
<!-- 方式二 在映射文件(*.hbm.xml)中 -->
<!-- 缓存策略 -->
<cache usage="read-only|read-write|transaction|nonstrict-read-write"> </cache>
~~~~~~

把数据存到一级缓存和二级缓存: `get` `load`,但是 `save` `update` 不一定


### 查询缓存(QueryCache) ###
~~~~~~
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

### ehcache 配置 ###
~~~~~~
<ehcache>
    <diskStore path=""></diskStore> <!-- 缓存在磁盘的目录 -->
    <defaultCache maxElementsInMemory=""
                  overflowToDisk="" <!-- 是否保存在硬盘 -->
                  maxElementsOnDisk="">
    </defaultCache>
     <!-- 设置二级缓存的硬盘保存策略-->
    <Cache name="ClassName"
        maxElementsInMemory=""
        overflowToDisk="true"
        maxElementsOnDisk="">
    </Cache>
</ehcache>
~~~~~~

