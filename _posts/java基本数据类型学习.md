title: java基本数据类型学习
date: 2014-04-11 16:09:58
tags:
- java
- 基本数据类型
---
# 基本数据类型对象包装类 #
为了方便操作基本数据类型, 将其封装成了对象,
在对象中定义了属性和行为丰富了数据的操作.
用于描述该对象的类就称为基本数据包装类型.

| 数据类型 | 包装类 |
|-------------------|
|byte    | Byte|
|short   | Short |
|int     | Integer |
|long    | Long |
|float   | Float|
|double  | Double|
|char    | Char |
|boolean | Boolean |

## 用法 ##
~~~~~~
Integer.parseInt("123");
Integer.parseInt("121", 2); // 以2进制读取
Boolean.parseBoolean("true");
Integer.MAX_VALUE;
Integer.toBinaryString(-6);
Integer.toOctalString(-6);
Integer.toString(60, 2); // 转换成2进制
new Integer(2).intValue(); // 转换成基本数据类型
~~~~~~

## 自动装箱 ##
~~~~~~
Integer x = 129;
Integer y = 129;
println(x==y); // false
println(x.equals(y)); //true

// jdk1.5以后, 自动装箱如果是一个字节, 那么数据不会创建一个新的空间
Integer x = 127; 
Integer y = 127;
println(x==y); // true
~~~~~~

