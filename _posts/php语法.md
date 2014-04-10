title: php基本语法
date: 2014-04-07 19:51:06
tags:
- php
---

php语法
========

## php标签 ##

php是嵌入式语言，意味着代码的混编，为了区别，所以有四种标签
* 两种标准标签
~~~~~~
<?php  `content` ?>

<script language=“php">
echo 'php';
</script>
~~~~~~
* 兼容标记  
短标记,修改php配置,`open_short_tags=on`
~~~~~~
<? echo 'h'; ?> 
~~~~~~
* asp标签风格, 修改配置 `aps_tags=on`
~~~~~~
<% echo 'h';  %>
~~~~~~

## html模式和php模式 ##
~~~~~~
<html><? ?></html>
~~~~~~

## 语句结束符 ##
语句结束符是`;`
> `echo "Yes";`

## 注释 ##
* 单行: `//` 或 `#`
* 多行: `/**  **/`


## 解释器zend ##

> php是编译型语言和解释型语言

流程: 读入源代码 -> 词法分析 -> 语法分析 -> 生成opcode -> 由zendEngine执行

## 变量 ##
`$` 不是变量的一部分,只是语法,使用变量用`$`来引用
~~~~~~
<?php
$name = 'itcast';
echo $name;
functioni name(){
}
?>
~~~~~~
### 变量名 ###
定义语法, 由字母 下划线 数字 组成
> $student_name = ''

### 变量的操作 ###
* 增加 `$name = 'itcast';`
* 删除 `unset($name)`

### 变量的赋值 ###
#### 值传递 ####
~~~~~~
$var_name = "value";
$new_var_name = $var_name;
$var_name = "change";
echo $new_var_name;  // value
~~~~~~

#### 引用传递 ####
~~~~~~
$var_name = "value";
$new_var_name = &$var_name;
$new_var_name = "change"
echo $var_name; // change
~~~~~~

### 可变变量 ###
~~~~~~
$name = "hello";
$hello = "itcast-php";
echo $$name //itcast-php

function f1() {}
$f_name = 'f1';
$f_name();
~~~~~~

## 常量 ##
~~~~~~php:
define('pai', 3.1415926);
echo pai;

define("-_-", "haha");
// echo -_-, 非法
echo constant("-_-");
~~~~~~

> Define('变量名','值','是否区分大小写') //0 区分, 1 不区分


~~~~~~
defined("pai"); //常量是否被定义
~~~~~~

### 常量相关函数  ###
~~~~~~
get_defined_constants(); //获取所有定义的常量
//预定义常量
echo PHP_OS;
echo PHP_VERSION;
//魔术常量
echo __FILE__;  //文件在的路径
echo __LINE__;  //当前行数
echo __DIR__;  //目录绝对地址
~~~~~~

### const 定义 ###
~~~~~~
const SOME = "some";
echo SOME;
~~~~~~

### 数据类型 ###
php是弱类型语言,指变量没有类型之分,一个变量可以存储任意格式的数据,而数据是有类型之分的.
> `var_dump()` 输出数据的值和类型


1. 整型, int, integer  
存储需求: 4byte, 32bit  
`PHP_INT_MAX` int最大值  
如果再增加,会变为其他数据类型,`PHP_INT_MAX+1; //float()`  
`echo time();` 获取时间戳
进制间转换, 16-hex, 10-Dec, 8-oct, 2-bin,
`hexdec(); deccbin(); decoct();`
~~~~~~
$int1 = 10;
$int2 = 010; //进制
$int3 = 0x16; //16进制
~~~~~~
2. 浮点  
php能存储精度为14位的有效数字, 理论最大值是1.8E308  
不能相信浮点数的比较
~~~~~~
<?php
$f1 = 1.23*10^10;
$f2 = 1.23E10;
$f3 = 1234E-10;
?>
~~~~~~
3. 布尔  
~~~~~~
$b1 = true|flase|TRUR|FALSE;
~~~~~~
4. 字符串  
定义方法: 单引号,双引号,heredoc,nowdoc(定界符)  
单引号: 所有内容都当字符看待  
双引号: 可以解析一切特殊的  
`chr(65)` 转成字符  
heredoc: 定义复杂字符  
nowdoc: 不可以解析变量
~~~~~~
<?php
$s1 = "itcast";
var_dump($s1); // String(6) "itcast"

$s2 = "hello, $s1";
$s2 = "hello, ${s1}";
$s3 = "\122"; // ascii

/* heredoc */
$s4 = <<<END
$变量名
content0
END;

/* nowdoc */
$s4 = <<<'END'
content
content0
END;
?>
~~~~~~
5. 数组  
数据的组合, 数据的集合  
一个元素也称为一个键值对, 键只能是整形和字符串
~~~~~~
<?php
$a1 = array('key' => 'value', 'key0' => 'value0');
echo $a1["key"];

$a2['new'] = 'new value';

$a3 = array('a', 'b', 'c');
echo $a3[0]; // 'a'
?>
~~~~~~
6. 对象  
一个对象可以保存多个值, 每个数据称之为属性.
~~~~~~
$o1 = new Stdclass();

$o1->name = "name";
$o1->age = 22;
$o2->gender='male';

class Stu{
    public $name;
    public $age;
    public function addAge(){
        $this->age++;
    }
}
$o2 = new Stu;
$o2->addAge();
~~~~~~
7. NULL  
表示什么数据都没有  
常见的输出Null, 只有一个值, 不区分大小写.
~~~~~~
echo $not_defined;

function f1(){};
echo f1();
~~~~~~
8. 资源  
外部资源, 不能手动创建, 只能通过php内置函数得到.
~~~~~~
$link = mysql_connect('127.0.0.1'...);
var_dump($link);    // resource
~~~~~~

## 类型转换 ##

* 期望类型和得到类型不一致
~~~~~~
$cond = 1243;
if($cond) {  //转换为True
}
~~~~~~
* 参与运算的不一致
~~~~~~
$s1 = "12abc";
$i1 = 23;
echo $s1 + $i1; // 12abc23

$cond1 = "abc"; // => 0
$cond2 = 0; 
echo $cond1 == $cond2; //true
~~~~~~
转换的方式:
1. php自己完成, 称之为自动类型转换  
按照默认的行为完成
2. 用户强制完成, 称之为强制类型转换  
~~~~~~
(bool)'abc';
~~~~~~

1. 不同类型转换
2. 布尔类型转换
~~~~~~
<?php
// 只有空数组,0,null 转为false
(bool) 'abce'; //true
(bool) '0'; //false
(bool) '0.0'; //true
(bool) '00'; //true
(bool) 'null'; //true
(bool) array(); //false
(bool) 0.0 ; //false
(bool) array(false) ; //true
(bool) array(null) ; //false
?>
~~~~~~

### 类型和变量中常用的函数 ###
`var_dump()` `unset()` `isset()` `empty()`
~~~~~~
$a = array('a' => 10, 20);
var_dump($a[0]); // 20
unset($a[0]);
var_dump($a);

$c = NULL;
isset($c); // false

empty(0); // null array() 0.0 '' '0' ==> true (bool)反义词

is_array();
is_int('10'); // true
is_numeric('10'); // true
~~~~~~
