title: java泛型
date: 2014-04-11 20:28:59
tags:
- java
- 泛型
---
# 泛型入门 #
Jdk1.5 出现安全机制.

好处:
1. 将运行时期的问题 ClasscastException 到了编译时期
2. 避免了强制类型转换的麻烦

什么时候用<>, 当操作的引用数据类型不确定的时候.就使用<>.
他表示一个用于接受具体引用数据类型的参数范围.

泛型即使是给编译器使用的技术, 用于编译时期, 确保了编译的安全.
运行时, 会将泛型去掉, 生成的class是不带泛型的, 称为泛型的擦除.

泛型的补偿: 运行时, 通过获取元素的类型进行转换动作,
不用进行强制转换.
~~~~~~
/** 泛型定义在类上 **/
class Tool<e1, e2> {
   /** 泛型定义在方法上 **/
   public <W> void show(W str) {}

   public static <T> void method(T t) {}
}

class Person implements Comparable<Person>{
}

/**泛型接口**/
interface Inter<T>{
   public void show(T t)
}

class InterImpl<T> implements Inter<T> {}
~~~~~~


# 泛型的通配符号 #

泛型的通配符: ?,代表未知类型

~~~~~~
public static void printCollection(Collection<?> al){}
~~~~~~

# 泛型的限定 #
~~~~~~
/**这里只能传Person类的对象**/
public static void printCollection(Collection<Person> al){}

/**这里只可以传可以传Person类的子类对象  上限 **/
public static void printCollection(Collection<? extends Person> al){}

/**这里只可以传可以传Person类的父类对象 下限 **/
public static void printCollection(Collection<? super Person> al){}

class MyCollection<E>{
   public void addAll(MyCollection<? extends E> e){}
}
/** 当对集合中的元素进行取出操作时, 可以用下限 **/
TreeSet(Comparator<? super E> comparator)
~~~~~~
