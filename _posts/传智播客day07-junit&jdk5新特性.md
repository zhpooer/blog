title: 传智播客day07-junit&jdk5新特性
date: 2014-04-07 09:01:39
tags:
- 传智播客
- java
---
# Tips #
~~~~~~
// propoties 的快捷方法
ResourceBundle res = ResourceBundle.getBundle("db"); //文件名为 db.prop
res.getString("user")
~~~~~~
.properties 不支持中文。使用jdk工具字符串转码工具
> native2ascii


# junit 使用 #
1. 线程独立
2. 测试方法  
1.public  
2.没有返回  
3.方法不能有参数  
3. 独立测试体，调用指定对象

>类一般为Test结尾，如果类名没有以Test结尾，类继承TestCase类

# 泛型 #
~~~~~~
val entrys = maps.entrySet();
entrys.iterator();
~~~~~~

~~~~~~
// T 不能是基本数据类型
public static <T> T print(T [] arr){
    
}
~~~~~~

# 可变参数 #
~~~~~
public static int getSum(int... params) {

}
~~~~~

# for循环 #
~~~~~~
List<String> list = Arrays.asList(arr);
for (String str : list) println(str); // 实现的对象要实现 iterable 接口
~~~~~~
# 枚举 #

~~~~~~
enum Role {
    boss, manager, emp; //私有构造类
}
Role.values() //返回对象数组
~~~~~~

## 枚举的扩展 ##
~~~~~~
enum Week {
   Mon("星期一"){@Override public void getName(){}}, Tue("星期二");
   private String week;

   private Week(String week) {
       this.week = week;
   }
   // getWeek(); setWeek();
   
   public abstract void getName();
}

Week.Mon.name();
Week.Mon.ordinal(); //脚标，0
~~~~~~

# 反射 #
~~~~~~
Class clazz = Class.forName("");
Class clazz = ref.getClass();

// 字段
Object instance = clazz.newInstance();
Field field = clazz.getDeclaredField("field"); // or getField() 获取public的字段
field.setAccessible(true); // private 变为 public
field.set(instance, "list");

// 获取方法
Method m = clazz.getDeclaredMethod("print", String.class); //方法的名字和方法的参数
// or  clazz.getDeclaredMethod("print", null);
m.setAccessible(true); // private to public
m.invoke(instance, "parameters");

Constructor con = clazz.getConstructor(String.class, String.class);
con.newInstance("");
~~~~~~
用scala实现的调用的java反射
~~~~~~
package io.zhpooer.test

import org.scalatest.FlatSpec
import org.scalatest.matchers.ShouldMatchers
import scala.beans.BeanProperty

class Person {
  @BeanProperty var name: String = _
  @BeanProperty var age: Int = _
  override def toString = name + ", " + age 
}

class ReflectSpec extends FlatSpec with ShouldMatchers {
  it should "通过Java反射机制得到类的包名和类名" in {
    val p = new Person;
    println("包名字: " + p.getClass().getPackage().getName())
    println("类名: " + p.getClass().getSimpleName())
    println("完整类名: " + p.getClass().getName())
  }

  it should "通过Java反射机制，用 Class 创建类对" in {
    val clazz = Class.forName("io.zhpooer.test.Person")

    clazz.newInstance() match {
      case p: Person =>
        p.setAge(12)
        p.setName("lest")
        println(p)
    }
  }
  
  it should "通过Java反射机制，用 Field 设置属性" in {
    val clazz = Class.forName("io.zhpooer.test.Person")
    val p = new Person;
    p.setAge(12);
    p.setName("test")
    
    val field = clazz.getDeclaredField("name") 
    field.setAccessible(true)
    field.set(p, "change")
    println(field.get(p) +  " " + p)
  }
  
  it should "通过Java反射机制得到类的一些成员信息 方法等" in {
    val clazz = Class.forName("io.zhpooer.test.Person")

    for ( field <- clazz.getDeclaredFields()) {
      println(field.getModifiers() + " " + field.getName())
    }
    
    for ( method <- clazz.getDeclaredMethods()) {
      println(method.getModifiers() + " " + method.getReturnType() + " " + method.getName())
    }
  }
}
~~~~~~
