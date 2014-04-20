title: java正则学习
date: 2014-04-15 08:11:20
tags:
- java
- 正则表达式
---
# 正则表达式 #

用于操作字符串的数据
~~~~~~
// 定义一个功能对QQ号进行校验
// 长度 5-15, 只能是数字, 0不能开头
boolean match = "938393".match("[1-9][0-9]{4,14}");
~~~~~~

## 常见规则 ##
~~~~~~
Character classes
[abc] 	a, b, or c (simple class)
[^abc] 	Any character except a, b, or c (negation)
[a-zA-Z] 	a through z or A through Z, inclusive (range)
[a-d[m-p]] 	a through d, or m through p: [a-dm-p] (union)
[a-z&&[def]] 	d, e, or f (intersection)
[a-z&&[^bc]] 	a through z, except for b and c: [ad-z] (subtraction)
[a-z&&[^m-p]] 	a through z, and not m through p: [a-lq-z](subtraction)
 
Predefined character classes
. 	Any character (may or may not match line terminators)
\d 	A digit: [0-9]
\D 	A non-digit: [^0-9]
\s 	A whitespace character: [ \t\n\x0B\f\r]
\S 	A non-whitespace character: [^\s]
\w 	A word character: [a-zA-Z_0-9]
\W 	A non-word character: [^\w]

Boundary matchers
^ 	The beginning of a line
$ 	The end of a line
\b 	A word boundary
\B 	A non-word boundary

Greedy quantifiers 贪懒
X? 	X, once or not at all
X* 	X, zero or more times
X+ 	X, one or more times
X{n} 	X, exactly n times
X{n,} 	X, at least n times
X{n,m} 	X, at least n but not more than m times
 
Reluctant quantifiers
X?? 	X, once or not at all
X*? 	X, zero or more times
X+? 	X, one or more times
X{n}? 	X, exactly n times
X{n,}? 	X, at least n times
X{n,m}? 	X, at least n but not more than m times
~~~~~~

## 匹配 ##
`"String".matches(regex)`

## 切割 ##
~~~~~~
String[] sep = str.split("\\s");
str = "zhangaaaaaaaaboccccccshen"
String[] sep = str.split("(.)\\1+"); // 组 > zhang bo shen
~~~~~~

## 组 ##
`((A)(B(C)))` 从左括号开始,第一个括号就是第一组, 第二个括号就是第二组

## 替换 ##
~~~~~~
str = "zhangaaaaaaaaboccccccshen"
// 叠词替换成一个
str.replaceAll("(.)\\1+", "$1");   // zhangabocshen

str = "15800001111";
str.replaceAll("(\\d{3})\\d{4}(\\d{3})", "$1****$2");
~~~~~~

## 获取 ##
java.util.regex.Pattern: 正则表达式的对象形式

~~~~~~
// 将正则封装成对象
Pattern p = Pattern.compile("a*b");
// 通过正则对象的获取匹配器对象
Matcher m = p.matcher("aaaab");
// 使用Matcher对象对字符串进行操作
boolean b = m.matches();
while(m.find()){
    m.group();  // 获取匹配的子序列
    m.start();  // index of start
    m.end();
}
~~~~~~
