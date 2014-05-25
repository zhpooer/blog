title: 传智播客day35-HQL
date: 2014-05-22 14:17:00
tags:
- hibernate
---

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
// 迫切左外连接, return List<Student>
String sql = "from Student s left outer join s.classes";

// 同理也可以查询几个字段, 生成模型类, 带构造函数的查询, 不能带 fetch
String sql = "from new io.zhooer.NewClasses(c.cname, s.sname) c left outer join  c.students";
~~~~~~
总结 :
1. 如果页面上要显示的数据和数据库中的数据相差甚远, 利用带select的构造器进行查询
2. 如果页面上要显示的数据和数据库中的数据相差不是很远, 用迫切连接
3. 如果采用迫切连接, from 后面跟的那个对象就是结构主体


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
