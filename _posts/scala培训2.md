title: scala培训2
date: 2014-06-15 10:11:33
tags:
- scala
---

# 大数据 #

数据来源:
* 网站点击量(分析用户行为习惯)
* 广告
* 反洗钱

实时分析

Hadoop 依赖硬盘, 对内存消耗小

硬件发展趋势, 硬盘 cpu 数据交换瓶颈, 发展速度慢, 故尽量使用内存做数据分析

Spark的中间数据放到内存中，对于迭代运算效率更高

storm 流式计算

shark(sql) depends on spark

~~~~~~
// wordcount
val sc = new SparkContxt("", "myjob", "");

// RDD Resilent Distribute dataset
// 弹性分布式数据集
val file = sc.textFile("hdfs://...");
val errors = file.filter(_.contains("error"));
errors.cache();
// take Action, 延迟执行
errors.count();


// RDD Objects, 基于图
rdd1.join(rddd2).groupBy(..).filter(..);
~~~~~~

TODO neo4j mahout

杭州数源 alyanbo/sliner/restful-hub

Erik Meijer

scalaz

netty

RESTful, spray


