title: java反射学习
date: 2014-04-14 21:33:07
tags:
- java
- 反射
---

# 反射 #
动态获取类中的信息, 就是java反射. 可以理解为对类的解剖.

对任意一个对象, 都能够调用他的任意一个方法和属性

## 获取Class ##
~~~~~~
// 获取Class
// 方式一
Person p = new Person();
Class clazz = p.getClass();
// 方式二
Class clazz = Person.class;
// 方式三
Class clazz = Class.forName("xx.xx.Person");
~~~~~~

## 构造函数 java.lang.Constructor##
~~~~~~
Object obj = clazz.newInstance();

//构造函数要传参数
Constructor cons = clazz.getConstructors();
clazz.getConstructor(String.class, String.clazz);
cons.newInstance()
~~~~~~

## 获取字段 java.lang.Field ##
~~~~~~
Field f = clazz.getField("age"); // 获取公共的字段
Field f = clazz.getDeclaredField("age"); // 获取所有的字段
f.setAccessible(true); // 私有的变为可访问
f.get(obj);
~~~~~~

## 获取方法 java.lang.Method ##
~~~~~~
Method[] ms = clazz.getMethods(); // 获取都是公有的方法
Method m = clazz.getDeclaredMethod("show", null); // 获取所有的方法
m.invoke(obj, null);
~~~~~~
