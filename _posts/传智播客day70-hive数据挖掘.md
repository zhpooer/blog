title: 传智播客day70-Hive数据挖掘
date: 2014-08-17 09:21:57
tags:
- 传智播客
- hadoop
---

# Hive #

Hive是建立在Hadoop上的数据仓库基础架构, 它提供了一系列的工具,
可以存储、查询和分析在 Hadoop中的大规模数据机制.
Hive 定义了简单的类 SQL 查询工具, 成为 QL, 将 QL 转换为 MapReduce.

可以结合 MapReduce 来处理复杂的分析工具

Hive表就是 HDFS 的目录, Hive数据就是目录里边的文件

## 系统架构 ##

* 用户接口, CLI, JDBC/ODBC, WebUI
* 元数据存储(Hive字段, 字段类型), 通常是存储在关系数据库如 mysql, derby 中
* 解释器, 编译器, 优化器, 执行器
* Hadoop, 用HDFS进行存储, 利用 MapReduce 进行计算

# 相关操作 #

启动Hive `hive`, 所有创建的表存在 `/user/hive/warehouse`

~~~~~~
show databases;
-- 默认使用 default
show tables;
create table student (
  id int,
  name string
);

-- /warehouse/itcast.db
create database itcast;
use itcast;
-- /warehouse/itcast.db/teacher
create table teacher (
  tid bigint,
  name string
);

-- 插入数据
load data local inpath '/home/a.txt' into table student;
-- 四行 `Null Null`, 因为没有指定分隔符
select * from student;
-- 执行 MapReduce
select count(*) from student;

load data local inpath '/home/a.txt' into table student;

-- 创建一张表, 以 \t 为分割
create table people(
  pid in,
  name string
) row format delimited fields terminated by '\t';

show create table people;
load data local inpath '/home/a.txt' into table people;
-- 不会产生 mapReduce
select * from people limit 2;

-- 会产生 mapReduce
select * from people groupby pid desc;

-- 执行 hadoop 命令
dfs -ls /
dfs -put /home/hadoop/b.txt /user/hive/warehouse/peoples

~~~~~~

`data.txt`
~~~~~~
1  zhangsan
2  lisi
3  wangwu
~~~~~~

可以将 `data2.txt` 放入 `/user/hive/warehouse/people`, 以至于也可以将文件加入数据库
`data2.txt`
~~~~~~
10 zhangsan
11  lisi
~~~~~~

# 设置数据库 #

hive默认数据库是 derby, 只允许连接一个用户

* 安装mysql `/usr/bin/mysql_secure_installation`
* 配置`hive-default.xml`
~~~~~~
<configuration>
    <!-- url -->
    <property>
        <name> javax.jdo.option.ConnectionURL
        </name>
        <value>?createDatabaseInNotExist=true
        </value>
    </property>
    <!-- 连接驱动 -->
    <!-- 用户名 -->
    <!-- 密码 -->
</configuration>
~~~~~~
* 拷贝jar文件到`lib`文件夹下
* 数据库授权
~~~~~~
grant all priveleges on *.* to `root`@`%` identity by 'root';
flush priveleges;
~~~~~~

# 外部表 #
~~~~~~
dfs -mkdir /data;
dfs -put /home/hadoop/*.txt /data
dfs -ls /data
-- 在其他目录创建 /data, 关联到表, 外部表
create external table users (
  uid int,row format delimited fields terminated by '\t' location '/data';
  uname string
) row format delimited fields terminated by '\t' location '/data';
~~~~~~

# 分区表 #

如按月份分区存储数据, 在数据量非常大的情况下

~~~~~~

-- 建立外部分区表
create external table beauties (
  id int,
  name string,
  size double
) row format delimited fields terminated by '\t' location '/beauty';

-- 指定分区 'China', 在hdfs上要增加一个 `notion=China` 的文件夹(dfs -mkdir /beauty/nation=China), 和一个字段 
load data local inpath '/home/hadoop/b.c' into table beauties partition (nation='China');

dfs -mkdir /beauty/nation=Japan;
alter table beauties add partition (nation='Japan') location `/beauty/nation=Japan`;

-- 建分区表是为了可以快速查询
select * from beauties where nation = 'china';
select nation, avg(size) si from beauties group by nation order by si;
~~~~~~

# sqoop 导入导出 #

mysql数据导入到 hdfs

~~~~~~
## 导入到 hdfs
## -m 启动 2个 mapreduce 一起导入
sqoop import --connect jdbc:mysql://mysql:3306/itcast
--username root --password 123 --table trade_detail --target-dir /sqoop -m 2

## 导入到 mysql
## create table td1 like trade_detail;

sqoop exoport --connect jdbc:mysql://mysql:3306/itcast --username root
--password 123  --password 123 --export-dir '/sqoop/td' --table td1 --fields-terminatd-by '\t'

~~~~~~


# 自定义函数(UDF) #



1. 导入jar包, `hive.jar`, `hadoop-common.jar`
2. 编写程序
~~~~~~
public class NationUDF extends UDF {
    private static map<String, String> nationMap = new HashMap<String, String>;
    static {
        nationMap.put("China", "天朝");
        nationMap.put("Japan", "小日本");
        nationMap.put("USA", "米国");
    }
    
    private Text t = new Text();
    
    // 返回 hadoop 支持的序列化类型
    public Text evaluate(Text ename) {
        String en = ename.toString();
        String cname = nationMap.get(en);
        if(cname == null) {
            cname = "火星人";
        }
        t.set(cname);
        return t;
    }
}
~~~~~~
3. 打包
4. 运行 hive 命令
~~~~~~
add jar /home/hadoop/udf.jar
-- 创建临时函数
create temporary function getNN as 'cn.itcast.hive.udf.NationUDF';
-- 调用
select id, name, getNN(nation) from beauties;
-- 将查询语句的结果存入表
create table result row format delimited fields terminated by '\t' as
     select id, name, getNN(nation) from beauties;
~~~~~~

# 数据采集框架(flume) #

注意版本(0.9和1.0)

Agent, 采集数据的单元, 包括 Source, Channel, Sink

Redis可用于分布式session存储

Flume 可用于收集多台机器的产生的数据(如 log), 写入到 HDFS 里面

1. 修改配置文件 `flume.env`
~~~~~~
JAVA_HOME=...
~~~~~~
2. `conf/a2.conf`
~~~~~~
## 设置 agent 的 源 通道 代理
a2.sources = r1
a2.channels = c1
a2.sinks = k1

# 定义具体的soruce
a2.sources.r1.type = exec   # 或 spooldir 监听数据变化
a2.sources.r1.command = tail -F /root/a.log # /home/hadoop/logs

# 定义具体的 channel
a2.channles.c1.type = memory
a2.channles.c1.capacity = 1000
a2.channles.c1.transactionCapacity = 100

# 定义具体的 sink
a2.sinks.k1.type = logger  # 或 hdfs

# 可以定义拦截器, 为消息添加时间戳

# 关联 channel
a2.sources.r1.channels = c1
a2.sinks.k1.channnels = c1
~~~~~~
3. 启动
~~~~~~
## -c 配置文件目录, -D jvm 配置
flume-ng agent -n a2 -f conf/a2.conf -c conf -Dflume.root.logger=INFO,console
~~~~~~

# 论坛数据分析 #

网站指标
* 浏览量PV: 每个页面访问页面数
* 访客数UV: 通常以 Cookie 为依据
* IP数
* 跳出率, 浏览一个页面的访问次数/全部访问次数汇总
* 版块热度排行

开发步骤
* flume 采集数据
* 对数据进行清洗(可以使用flume拦截器)
* 使用hive进行数据的多维分析
* hive分析结果通过 sqoop 导出到 mysql 中
* 提供视图工具供用户使用

hive 创建表
~~~~~~
create external table hmbbs(
  ip string,
  logtime string,
  url string
) partitioned by (logdate string) row format delimited fields terminated by '\t' location "/cleaned";
~~~~~~

定时运行脚本
~~~~~~
CURRENT=`date +%y%m%d`
hadoop jar /cleaner.jar /flume/$CURRENT /cleaned/$CURRENT

## 清洗数据 
hive -e "alter table hmbbs add partition (logdate=$CURRENT) location '/cleaned/$CURRENT'"
## PV
hive -e "select count(*) from hmbbs where logdate=$CURRENT";
## UV
hive -e "select count(distinct ip) from hmbbs where logdate=$CURRENT";
## 注册用户数
hive -e "select count(*) from hmbbs where logdate=$CURRENT and instr(url, 'member.php?mod=register')>0";

## 查找重点用户, 倒叙
hive -e "create table vip_$current row format delimited fields terminated by '\t'
    select ip, count(*) as vtimes from hmbbs groupby ip having vtimes >= 50 order by vtimes desc";

## 导出数据到 mysql
sqoop export --connect jdbc:mysql://mysql:3306/itcast --username root
  --password root "/user/hive/warehouse/vip_$CURRENT" --table vip --fields-terminatd-by '\t';
~~~~~~




# 建立简历 #

虚拟化: openStack, cloudStack

需要掌握技能:
* 集群的搭建
* MapReduce 编写


