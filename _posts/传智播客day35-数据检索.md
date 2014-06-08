title: 传智播客day35-数据检索
date: 2014-05-22 14:17:00
tags:
- 传智播客
- hibernate
---

# Hibernate 检索 #
五种检索数据方式:
1. 导航对象图检索方式, 更具已经加载的对象导航的其他对象
~~~~~~
Customer c = session.get(Customer.class, 1);
c.getOrders().size();
~~~~~~
2. OID检索方式, `session.get()/load()`
3. HQL检索, `session.createQuery()`
4. QBC检索, `session.createCriteria()`
5. 本地SQL检索, `session.createSQLQuery()`

~~~~~~
// HQL,排序
String hql = "from Customer order by name asc";
Query query = session.createQuery(hql);
List<Customer> list = query.list();

// QBC
Criteria criteria = session.createCriteria(Customer.class)
// 排序
List customer = criteria.addOrder(org.hibernate.Order.asc("name")).list();

// SQL
String sql = "select orders.* from customer, orders where customers.id=orders.customer_id and customer.name=?";
SQLQuery sqlQuery = session.createSQLQuery(sql);
sqlQeury.setParameter(0, "mary");
sqlQuery.addEntity(Orders.class);

// 多态查询
// 返回所有 Object 子类的对应表数据
List list = session.createQuery("from java.lang.Object").list();

//使用聚集函数, 查询最大年龄的客户
// 如果查询结果只有一条记录或者 无记录, 使用 uniqueResult是没有问题,
// 但是如果结果大于一条记录, 报错
Integer age = session.createQuery("select max(age) from Customer").uniqueResult();
~~~~~~


# 单表查询 #
~~~~~~
// 返回的是 Classes 对象
List<Classes> cs = session.createQuery("from Classes").list();
// 带 select 查询, 出来的是 数组 
List cs = session.createQuery("select cid, cname from Classes").list();

// 带构造函数的查询, 必须先有构造函数
String sql = "select new io.zhpooer.Classes(cname, description) from Classes";
List<Classes> cs = session.createQuery(sql);

// 条件查询
Query query = session.createQuery("select cid, cname from Classes where cid=:cid");
query.setLong("cid" , 1L);
Classes cl = query.unique();
// 或
Query query = session.createQuery("select cid, cname from Classes where cid=?");
query.setLong(0 , 1L);

// QBC
Cuttomer c = session.createCriteria(Customer.class)
                    .add(Restrictions.eq("name", "tom")).uniqueResult();

// 查询一号用户的所有订单
List list = session.createQuery("from Order where customer.id=?").list();
// 把参数绑定到一个持久化对象
List list = session.createQuery("from Order where customer=?").setEntity(0, customer);

List list = session.createCriteria(Customer.class)
                   .add(Restrictions.eq("customer", customer)).list()

// 子查询
String sql = "select Classes where cid in (select cid in(select cid from Classes where cid in(1,2,3)))";
List<Classes> cs = session.createQuery(sql).list();

~~~~~~
# 一对多 #

~~~~~~
// 等值连接
// 返回 Array[tuple2<Classes, Student>]
String sql = "from Classes c, Student s where c.cid=s.classes.cid";
// 内连接, 放回结果和上面相同
String sql = "from Classes c inner join c.students";
// 迫切内连接, return List<Classes>
String sql = "from Classes c inner join fetch c.students";

// 左外连接, return Array[tuple2<Classes, Student>]
String sql = "from Classes c left join  c.students";
// 迫切左外连接, return List<Classes> 
String sql = "from Classes c left outer join fetch  c.students";
// 迫切左外连接, 数据会重复, 用distinct 排重
String sql = "select distinct c from Classes c left outer join fetch  c.students";

// 迫切左外连接, return List<Student>
String sql = "from Student s left outer fetch join s.classes";

// 多表条件查询, 隐式内连接
List list = session.createQuery("from Order o where o.customer.name=?")
                   .setParameter(0, "mary");
// QBC 连接查询
Criteria criteria = session.createCriteria(Order.class);
// 设置别名, 表关联
criteria.createAlias("customer", "c");
// 在关联时, 使用左外连接
criteria.createAlias("customer", "c", Criteria.LEFT_JOIN);
criteria.add(Restrictions.eq("c.name", "mary"));
criteria.list();


// 投影查询, 同理也可以查询几个字段, 生成模型类, 带构造函数的查询, 不能带 fetch
String sql = "from new io.zhooer.NewClasses(c.cname, s.sname) c left outer join  c.students";
String sql = "from count(c) from Customer c";
// QBC方式
List list = session.createCriteria(Customer.class)
                   .setProjection(
                       Projection.projectionList()
                                 .add(Property.forName("name"))
                                 .add(Property.forName("age"))
                   ).list();
~~~~~~
总结 :
1. 如果页面上要显示的数据和数据库中的数据相差甚远, 利用带select的构造器进行查询
2. 如果页面上要显示的数据和数据库中的数据相差不是很远, 用迫切连接
3. 如果采用迫切连接, from 后面跟的那个对象就是结构主体

# 报表查询 #

~~~~~~
Long count = (Long)session.createQuery("select count(*) from Customer");
List list = session.createQuery("select count(*) from Order group by customer").list();
~~~~~~

# 命名查询和离线查询 #

~~~~~~
<!-- hbm.xml -->
<query name="sql1">
    from Cusomer
</query>
<sql-query name="sql2">
    select * from cusomer
</sql-query>
~~~~~~

~~~~~~
session.getNamedQuery("sql1").list()
~~~~~~

## 离线条件对象 ##

在业务层得到 `Criteria`, 在没有session情况下, 构造条件查询对象
~~~~~~
// 业务层
DetachedCriteria deCriteria = DetachedCriteria.forClass(Customer.class);
deCirteria.add(Restrictions.eq("name", "kitty"));

// 数据层
Criteria criteria = deCriteria.getExecutableCriteria(session);
criteria.uniqueResult();
~~~~~~

# 多对多 #
~~~~~~
String sql = "from Student s inner join fetch s.courses c";
List<Student> cs = session.createQuery(sql).list();
~~~~~~

# 一对多结合多对多 #
如果多张表进行查询, 找核心表 
~~~~~~
String hql = "from Classes c inner join fetch c.students s inner join fetch s.courses co";
List<Classes> cs = session.createQuery(hql).list();

// 会有重复值, 要用Set容器装
String hql = "from Classes c outer join fetch c.students s outer join fetch s.courses co";
~~~~~~


