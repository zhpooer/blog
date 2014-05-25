title: 传智播客day33-Hibernate关系传递
date: 2014-05-20 15:28:47
tags:
- 传智播客
- Hibernate
---
# Hibernate 关系传递 #

## 一对多的单向关联 ##
可以通过Classes 到 Student, 但不能从 Student 到 Classes.
~~~~~~
public class Classes {
    @BeanProperty private Long cid;
    @BeanProperty private String cname;
    @BeanProperty private String description;
    @BeanProperty private Set<Student> students; // Set可以避免重复
}
public class Student{
    @BeanProperty private Long sid;
    @BeanProperty private String sname;
    @BeanProperty private String description;
}
~~~~~~
映射文件
~~~~~~
<!-- Classes.hbm.xml -->
<hiberante-mapping>
    <class name="Classes">
        <id name="cid" length="5" type="long">
            <gernerator class="increment"> </gernerator>
        </id>
        <property name="cname" length="20" type="string"> </property>
        <property name="description" length="100" type="java.lang.String"> </property>
        <!-- set 元素对应类中的set集合
             通过set元素使classes表和student表建立关联-->
        <!-- key是通过外键的形式让两张表建立关联  -->
        <!-- one-to-many 是通络类的形式让两个类建立关联 -->
        <set name="students">
            <key> <!-- 用来描述外键 -->
                <column name="cid"> </column>
            </key>
            <one-to-many class="Student"/>
        </set>
    </class>
</hiberante-mapping>
<!-- Student.hbm.xml -->
<hibernate-mapping>
    <class name="Student">
        <id name="sid" length="5">
            <generator class="increment"> </generator>
        </id>
        <property name="sname" length="20"> </property>
        <property name="descscription" length="100"> </property>
    </class>
</hibernate-mapping>
~~~~~~
### 执行操作 ###
~~~~~~
Session session = sessionFactory.openSession();
Transaction transaction = session.beginTransaction();

Classes classes = new Classes();
classes.setCname("第八期");
classes.setDescription("很牛");
session.save(classes);

Student student = new Student();
student.setSname("班长");
student.setDescription("很牛");
// 如果学生很多, 要执行很多次, 可以用级联(Cascade)来解决
session.save(student);
transaction.commit();
session.close();
~~~~~~

### 级联(Cascade) ###
~~~~~~
<!-- 配置级联操作,
     save-update: 在保存(save) 或更新(update)班级时,
     也保存(save)和更新(update)学生, 即使学生是临时状态
     delete: 在删除班级的时候, 也删除学生
     all: save-update, delete
     -->
<!-- inverse 维护关系
             true 不维护(classes 不维护与 student 的关系)
             false 维护
             default false-->
<set name="students" cascade="save-update" inverse="true">
    <!-- any -->
</set>
~~~~~~
在保存 Classes 时, 同时保存 students
* cascade 通过更新班级 级联保存学生(决定插入学生, 但不决定建立外键关联)
* inverse 建立班级和学生之间的关系(决定是不是建立外键关联)
~~~~~~
Classes classes = new Classes();
classes.setCname("第八期");
classes.setDescription("很牛");

Student student = new Student();
student.setSname("班长");
student.setDescription("很牛");

Set students = new HashSet<Student>();
students.add(student);
// 建立 classes 和 student 之间的关联
classes.setStudents(students); 
// student 还是临时状态对象, 在保存班级的时候同时保存学生
// 进行了级联操作
session.save(classes);

// 删除一个学生
session.delete(student);
~~~~~~
把一个学生从一个班级转到另一个班级
~~~~~~
// 可以不写, 原因如下
// Classes c1 = session.get(Classes.class, 1L);
Classes c2 = session.get(Classes.class, 2L);
Student s = session.get(Student.class, 1L);
// 会执行数据库删除外键操作, 而随后一句会执行更新外键操作, 就变成 2条 sql 语句
// 所以 第一条可以不写
// c1.getStudents().remove(s);
c2.getStudents().add(s);
// 因为都是持久化状态, 所以不用 `save` 或 `update`
transaction.commit();
session.close();
~~~~~~
删除Classes中的所有学生
~~~~~~
Classes classes = session.get(Classes.class, 1L);
// 方式一: 效率不高, 因为要先查询
Set students = classes.getStudents();
students.clear();
// 方式二
classes.setStudents(null);
session.close();
~~~~~~
删除一个班级
~~~~~~
Classes c = session.get(Classes.class, 5L);
// 如果 inverse=true, classes不会维护和student的关系, 外键不会删除, 会报错
// 必须 inverse=false, 或者 cascade=delete(会执行多条"delete * from Student where sid=?"语句)
session.delete(c);
~~~~~~

~~~~~~
// 没有在 hbm.xml 设置 cascade, 就不会执行级联,
// 如果两个关联的对象中有一个的状态是 临时, 会报错,
// `TransientObjectException`: object references an unsaved transient instance
Classes classes = new Classes();
classes.setCname("xxx");
Student student = new Student();
classes.getStudents().add(student);
session.save(classes); //  如果 cascade 不设置, 报错
~~~~~~

把 `session.save()/update()` 一个对象的操作是显示操作,
级联一个对象是一个隐式操作

## 一对多的双向 ##

~~~~~~
<!-- Class Student -->
<!-- @BeanProperty private Classes classes; -->
<class name="student">
    <!-- any other property -->
    <many-to-one name="classes" class="io.zhpooer.Classes" column="cid"/> 
</class>
~~~~~~
一对多涉及到关系操作, 在多的一方操作效率高, 因为多的一方不会执行`update`外键操作
~~~~~~
Student student = new Student();
Classes classes = session.get(Classes.class, 3L);
classes.getStudents.add(student); // 如果 inverse 为 false 会发出 update外键 语句
student.setClasses(classes);
session.save(student); // 不会发出 update 外键 语句
~~~~~~

## 删除关系的错误 ##

~~~~~~
Classes c = session.get(Classes.class, 1L);
Set students = classes.getStudents();
for(Student s:students) {
    // student.setClass(null);   不能解决下面的问题, 因为是 classes 维护着关系
    // 解决问题思路: students.remove(student);, 但是不能在迭代中删除
    session.delete(student); // 会报错, 因为 student 还关联着 classes
}
~~~~~~

## 一对多总结 ##
* 如果让一的一方维护关系，取决于的因素有
  1. 在一的一方的映射文件中，set元素的inverse属性为default/false
  2. 在客户端的代码中，通过一的一方建立关系
  3. `session.save/update`方法是用来操作表的，和操作关系没有关系
* 怎么样采用级联的方法通过保存一个对象从而保存关联对象
  1. 如果`session.save`操作的对象是A，这个时候应该看A.hbm.xml,看set元素中cascade是否设置有级联保存
  2. 在客户端通过A建立关联
  3. 在客户端执行`session.save(A)`
* 一对多的情况，多的一方维护关系效率比较高
  1. 在多的一方many-to-one中没有inverse属性
  2. 在客户端通过多的一方建立关联

## 多对多 ##
学生和课程的关系是多对多
~~~~~~
public class Student {
    @BeanProperty private Long sid;
    @BeanProperty private String name;
    @BeanProperty private Set<Course> courses;
}
public class Course {
    @BeanProperty private Set<Student> students;
    @BeanProperty private Long cid;
    @BeanProperty private name;
}
~~~~~~

~~~~~~
<class name="Student">
    <id name="sid">
        <generator class="increment"> </generator>
    </id>
    <!-- table 就是用来描述第三张表的 -->
    <set name="courses" table="student_course">
        <key>
           <!-- student_course 通过sid, 与student表建立关联 -->
            <column name="sid"> </column>
        </key>
        <!-- colomn 也是描述要链接过去的外键 -->
        <many-to-many class="Cource" column="cid"> </many-to-many>
    </set>
</class>
<class name="Course">
    <id name="cid">
        <generator class="increment"> </generator>
    </id>
    <set name="students" table="student_course">
        <key>
            <column name="cid"> </column>
        </key>
        <many-to-many class="Student" column="sid"> </many-to-many>
    </set>
</class>
~~~~~~
关系操作:
* 多对多, 谁操作, 效率都一样
* 解除关系, 把第三张表一行数据删除掉
* 建立关系, 把第三张表数据增加一行记录
* 变更关系, 对第三张表 执行update操作, 先删除后增加

级联操作: 都是对象针对集合的操作

## 一对一 ##
~~~~~~
<!-- 方式一 -->
<many-to-one unique="true"></many-to-one>
<!-- 方式二: 主键关联, person 的主键是外键 -->
<class name="Person">
    <id name="pid">
        <generator class="foreign">
            <param name="property"> address </param>
        </generator>
    </id>
    <one-to-one name="address" constrained="true" class="Address"></one-to-one>
</class>
~~~~~~

