title: scala thoughtworks培训
date: 2014-06-08 13:06:47
tags:
- scala
---

# 基于scala的框架 #

play: web框架, 仿制rails

spark: 大数据分析

# 动态类型和静态类型 #

动态: 运行时, 进行类型检查

静态: 编译时, 进行类型检查

简单地说，在声明了一个变量之后，不能改变它的类型的语言，是静态语言；能够随时改变它的类型的语言，是动态语言. 


## 为什么要静态类型 ##
* 编译检查, 安全类型
* 工具提示
* 性能好
* Java/C#

## 为什么要动态特性 ##
* 轻量和灵活, 可根据框架自动生成代码
* 运行时改变程序的行为
* Python/Ruby, JavaScript

什么是码农? 人肉编译器, 把业务场景编译成代码

# 为什么要面向对象 #

解决问题更方便, 快捷?

面向对象更好地描述业务逻辑, 与人认知世界的方式相似(总结)

三大特征, 解决三个问题
* 封装, 规范, 可维护
* 继承, 可重用
* 多态, 可扩展([javascript如何实现?](http://www.zhihu.com/question/20177988))

Java数据和行为分开(Dao和Model), 不太面向对象. ruby on rails把数据和行为一同封装(面向对象).

TODO: 丧钟为谁而鸣?

# 为什么要函数式编程 #

[什么是纯函数](http://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E5%89%AF%E4%BD%9C%E7%94%A8#.E5.BC.95.E7.94.A8.E9.80.8F.E6.98.8E)

函数可以当做参数传递, 表述清晰

函数式编程特性, 如何解决 可维护性, 可重用性, 可扩展性?
* 无副作用
* [引用透明](http://www.jdon.com/42422)
* [高阶函数](http://zh.wikipedia.org/zh/%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0)

面向对象, 对象在做事情; 函数式, 函数在做



| 解决问题  |  面向对象 | 函数式  |
|-------------------|
| 重用单位  |  类       |   方法   |
| 可维护性  |  信息封装 |   信息不变化   |
| 可扩展性  |  类组合   |   方法组合   |

[深入函数式,请猛击](/2014/05/16/函数式编程手记)

## 为什么函数式编程没有流行起来? ##

图灵完备: 可计算的都可计算. (递归, 需要无限的计算资源)

物理限制, 计算资源有限, 内存有限

现今内存和 cpu 运算速度指数增长, 以及对并行运算的需求

# scala特性 #

动态性: REPL, 运行时编译

tuple: 可以存储不同类型的集合

<!-- **tessdemo** -->

# 学习资源 #

[twitter scala 指南](http://twitter.github.io/scala_school/zh_cn/)

[typesafe scala 交互学习](https://typesafe.com/platform/getstarted)

[scala官方文档中文](https://code.csdn.net/DOC_Scala/chinese_scala_offical_document)

