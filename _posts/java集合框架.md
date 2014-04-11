title: java集合框架
date: 2014-04-11 16:38:52
tags:
- java
- 集合
---
# 集合初识 #

集合类的由来:
> 对象用于封装特有的数据, 对象多了需要存储, 如果对象不确定.
> 存储就使用集合容器进行


集合特点:
1. 用于存储对象的容器
2. 集合长度是可变的
3. 集合不可存储基本数据类型

集合体系的共同父类 `java.util.Collection`
~~~~~~
boolean add(E e);
boolean contains(Object o);
void clear()
boolean containsAll(Collection<?> c);
boolean remove(Object o);
boolean removeAll(Collection<?> c);
int size();
Iterator<E> iterator(); // 迭代器
boolean retainAll(Collection coll); // 取交集
Object[] toArray(); // 集合转数组
~~~~~~
## Iterator ##
Iterator 对象必须依赖于具体容器, 因为每一个容器的数据结构都不同.
所以该迭代器对象在容器中进行内部实现的.
**Iterator用法**
~~~~~~
for(iterator it = coll.iterator(); it.hasNext();){
    it.next();
}
~~~~~~

## Collection 的继承结构图 ## 
![Collection 的继承结构图](/img/collections_hierachy.png)
# List 和 Set #
List 和 Set 都从 Collection 继承

List: 有序,元素有索引, 元素可以重复

Set: 元素不能重复, 无序

## java.util.List ##
~~~~~~
void add(index, element);
void add(index, collection);

Object remove(index);
Object set(index, element);

Object get(index);
int indexOf(object);
int lastIndexOf(object);
List subList(from, to);
~~~~~~
Iterator 接口在迭代过程中不能执行添加操作,
可以使用Iterator的子接口ListIterator进行操作.
~~~~~~
ListIterator it = list.listIterator();
it.add("xx");
it.previous();
it.hasPrevious();//允许逆向便利
~~~~~~

### Vector ###
内部是是数组数据结构, 是线程安全的, 长度可变, 增删查询都慢
### ArrayList ###
内部是是数组数据结构, 是线程不同步的, 长度可变, 查询速度快
### LinkedList ###
内部是链表, 是不同步的, 增删元素的速度快

## Set ##
Set接口中的方法和 Collection 接口的方法一致

### HashSet ###
内部数据结构是哈希表, 是不同步的

使用元素的hashCode方法来确定位置, 如果位置相同,
在通过元素的equals来确定是否相同.
所以在使用一个新类时要重写类的方法 `hashCode()` 和 `equals()`
#### LinkedHashSet ####
有序的HashSet

### TreeSet ###

可以对Set集合中的元素进行排序(从小到大), 是不同步的.
存储在里面的元素必须实现 `java.lang.Comparable` 接口,
他根本不看哈希码, 他判断元素的唯一性, 是看比较结果
~~~~~~
import java.util.Comparator;
public class ComparatorByName implements Comparator {
   @Override public int compare(Object obj1, Object obj2){
       return 0;
   }
}

ThreeSet ts = new TreeSet(new ComparatorByName());

~~~~~~

# Map #

Map: 一次添加一对元素. Collection 一次添加一个元素.
Map集合中存储的是键值对, **必须保证键的唯一性**.

~~~~~~
void clear();

V put(K key, V value); //返回前一个和key关联的值, 如果没为null
V remove(K key); // 删除键值对, 返回值

boolean containsKey(key);
boolean containsValue(value);
boolean isEmpty();

value get(key); // 如果没有则返回null
int size();

Set keySet();
Set<Map.Entry> entrySet();
Collection<V> values();
~~~~~~

## Hashtable ##
内部结构是哈希表, 是同步的, 不支持空键和空值.
> Properties: 用来存储键值对型的配置文件信息, 可以和IO技术结合
## HashMap ##
内部结构是哈希表, 不是同步的. 允许空值和空键.

## TreeMap ##
内部是二叉树, 不是同步的. 可以对Map中的键进行排序. 
