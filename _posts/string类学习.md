title: string类学习
date: 2014-04-11 14:34:24
tags:
- java
- String
- StringBuffer
---
# 字符串初识 #
1. 字符串是一个特殊的对象
    
2. 字符串一旦初始化, 就不可以被改变

    字符串初始化在字符串常量池
~~~~~~
String s = "abc"; // 在常量池当中
String s1 = new String("abc"); // 在堆内存中生成
println(s==s1); // false
println(s.equals(s1)); // true
~~~~~~
3. 字符串的构造
~~~~~~
byte[] arr = {65, 66, 67, 68};
new String(arr); // ABCD
new String({'a', 'b', 'c'});// abcd
~~~~~~

# 字符串方法 #
~~~~~~
/** 获取字符的长度 **/
int length(); 

/** 根据位置获取字符 **/
char charAt(int pos)

/** 字符第一次出现位置
* 没有找到返回-1
**/
int indexOf(int ch);
int indexOf(int ch, int fromIndex);
int indexOf(String str, int fromIndex);

int lastIndexOf(int ch);
int lastIndexOf(int ch, int from Index);

/* 获取部分子串 */
String substring(int beginIndex);
String substring(int beginIndex, int endIndex);

/* 变成字符数组 */
char[] toCharArray();

/* 变成Byte */
byte[] getBytes();

/* 变大小写 */
String toUpperCase();
String toLowerCase();

/* 替换 */
String replace(char old, char new);
String replace(CharSequence old, CharSequence new);

/* 去掉前后空白字符 */
String trim();

/* 将字符串进行连接 */
String concat(String str);

/* valueOf */
String.valueOf(4); // toString

/* 字符串比较 */
boolean equalsIgnoreCase(String str);
boolean equals(Object obj);

/* 是否包含 */
boolean contains();

/* 是否指定开头和结尾 */
boolean endsWith(String s);
boolean startsWith(String s);
~~~~~~
## 字符串切割 ##
> String [] split(String regex);


~~~~~~
String s = "a,b,c";
s.split(",")
~~~~~~

## intern方法 ##
对字符串操作, 当返回常量池中的字符串, 如果常量池中没有, 就插入
~~~~~~
String s = new String("abc").intern();
/* equals */
String = "abc";
~~~~~~

# StringBuffer #
StringBuffer 是字符串缓冲,用于存储数据的容器

1. 长度是可变的
2. 可以存储不同类型的容器
3. 最终转换成字符串
4. 可对字符串进行修改, 可添加 `append(data)`,
可插入 `insert(pos, data)`, 删除 `delete(pos, pos)`
~~~~~~
//初始化一个长度为40的空间
StringBuffer sb = new StringBuffer("40"); 
sb.append(4).append(false).append("haha");
sb.insert(1, "hehe");


StringBuffer sb = new StringBuffer("abc");
sb.insert(1, "hehe"); // ahehebc

sb.delete(1, 4); // abc
sb.deleteCharAt(0); //bc

// 清空
sb.delete(0, sb.length());

// 查找, 类似String
// 修改
sb.replace(1,3,"123");
sb.setCharAt(0, 'c');

sb.setLength(0); // 可清空缓冲区
~~~~~~

## StringBuilder ##
StringBuilder 和 StringBuffer 功能和用法一模一样,
StringBuffer 线程安全, StringBuilder 线程不安全,
单线程推荐使用 StringBuilder

