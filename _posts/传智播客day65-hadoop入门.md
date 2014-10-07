title: 传智播客day65-Hadoop入门
date: 2014-08-10 09:04:39
tags:
- 传智播客
- Hadoop
---

# 云计算 #

<!-- QQ群 316336020 -->
<!-- about.com -->

熟悉java集合类(Vector, ArrayList, LinkedList), io, 并发编程(锁 Lock, synchronized)和熟悉jvm原理及内存管理,
对数据结构, 算法有深刻的理解

# 项目简介 #


作者 Doug Cuttng, 是 Lucene, Nutch, Hadoop 等项目发起人

解决问题
* 海量数据的存储(HDFS)
* 海量数据的分析(MapReduce), 分布式计算模型
* 资源调度(YARN)

受Google三篇论文的启发(GFS, MapReduce, BigTable)

hadoop 擅长日志分析, facebook利用Hive来进行日志分析, HiveSQL进行数据分析;
Pig可以做高级的数据处理, 推荐系统.

用廉价的服务器搭建搭建集群服务器

Storm + Hadoop 具有强大的优势

Hadoop缺点, 只能进行离线数据

Storm能进行实时数据处理

核心
* HDFS(Hadoop Distributed File System), 分布式文件系统,
通过水平扩展机器的数量来增加存放文件的能力, 将数据进行冗余存储(多分存储)
* YARN(Yet Another Resource Negotiator), 资源调度管理系统,
可以运行其他的编程模型, 使实时处理出现了可能

特点
* 扩容能力强(scalable)
* 成本低(Economical): 通过普通机器组成的服务器来分发以及处理数据
* 高效(Efficient), 分发数据, 并行计算
* 可靠(Reliable), 失败任务的转移


## 生态圈 ##

TODO

nutch 抓取数据, HDFS存储数据, Lucence检索分析 , Zookeeper 进行管理

版本: Apache(2.4.1), Cloudera, HDP(Hortonworks Data Platform)

# HDFS的架构 #

主从结构
* 主节点, namenode
* 从节点, 很多个: datanode

namenode 负责管理
* 接受用户操作请求
* 维护文件系统的目录结构
* 管理文件与block之间关系,

datanode 负责存储文件
* 存储文件
* 文件被分成block存储在磁盘上
* 保证安全

如何自己设计一个分布式文件系统?

> 客户端(Client) 查询 NameNode(记录文件存储信息), 将数据放入datanode(多个)或从中取出  
> 数据在上传过程中要进行冗余保存, datanode 自行进行水平复制.  
> 上传过程中, 文件会被分成8块, 每块128M, 其实是对块的冗余存储


如何解决海量数据的计算?

> 求和 1+2+3+4+5+6=?  
> map: 1+2, 3+4, 5+6  
> reduce: 3 + 7 + 11


# hadoop版本对比 #

`hadoop 1.0`: MapReduce + HDFS

`hadoop 2.0`: MapReduce + YARN(资源管理) + Others + HDFS

# 部署 Hadoop #

三种模式
* 本地模式
* 伪分布式
* 集群模式

## 伪分布模式 ##
在centos环境下

fileziler, windows下ftp工具

secureCRT, windows下ssh工具

* 修改主机名 `/etc/sysconfig/network`
~~~~~~
HOSTNAME=itcast
~~~~~~
* 修改 IP, `/etc/sysconfig/network-script/ifcfg-eth0`
~~~~~~
DEVICE="eth0"
BOOTABLETO="static"
HWADDR=""
IPV6INIT="yes"
NM_controlled="yes"
ONBOOT="yes"
TYPE="Ethernet"
UUID=""
IPADDR="192.168.1.101"
NETMASK="255.255.255.0"
GATEWAY="192.168.1.1"
NDS1="8.8.8.8"
NDS2="4.4.4.4"
~~~~~~
* 主机名和ip的映射关系, `/etc/hosts`
~~~~~~
192.168.1.101 itcast
~~~~~~
* 关闭防火墙
~~~~~~
service iptables stop
chkconfig iptables --list
chkconfig iptables off
~~~~~~
* 安装JDK 64位或32位
* 修改Hadoop配置文件
  1. `hadoop-env.sh`
~~~~~~
export Java_Home=...
~~~~~~  
  2. `core-site.xml`
~~~~~~
<configuration>
  <!-- namenode 地址 -->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://itcast:9000</value>
  </property>
  <!-- 运行时产生文件,非临时, 重要! -->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/tmp/hadoop</value>
  </property>
</configuration>
~~~~~~
  3. `hdfs-site`
~~~~~~
<configuration>
  <!-- hdfs副本的数量 -->
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
~~~~~~
  4. `mapred-site.xml.template` copy to `mapred-site.xml`
~~~~~~
<configuration>
  <!-- mapreduce运行在yarn上 -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
~~~~~~
  5. `yarn-site.xml`
~~~~~~
<configuration>
  <property>
    <!-- 指定yarn的 ResourceManager 所在地址 -->
    <name>yarn.resourcemanager.hostname</name>
    <value>itcast</value>
  </property>
  <!-- reduce 获取数据的方式 -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
~~~~~~
* 初始化 Hadoop, 初始化 HDFS, *只需要执行一次*
~~~~~~
## hadoop namenode -format, 已被舍弃
hdfs namenode -format
~~~~~~
* 启动 Hadoop
~~~~~~
# start-all.sh, 已过时
start-dfs.sh
start-yarn.sh

# stop-all.sh 停止进程
stop-dfs.sh
stop-yarn.sh
~~~~~~
* 执行`jps`, 查看运行进程

        NodeManager
        NameNode
        DataNode
        ResourceManager
        SecondaryNameNode
* 查看管理界面
  * hdfs `http://192.168.1.101:50070`
  * yarn `http://192.168.1.101:8088`



Hadoop目录结构
* `sbin`, 启动或停止hadoop相关进程的命令
* `bin`, 操作hadoop相关模块的一些命令
* `lib`, 动态库
* `share`, 相关jar包
* `etc`, 配置文件

## 测试 ##

hdfs操作
~~~~~~
## 上传文件命名为 jdk
hadoop fs -put /root/jdk_***.tar hdfs://itcast:9000/jdk

## 查看文件, 也可以通过 htfs管理页面查看目录
hadoop fs -ls hdfs://itcast:9000/

## 下载文件
hadoop fs -get hdfs://itcast:9000/jdk /root/jdk_***.tar
~~~~~~

运行 mapreduce, 词频统计 `share/mapreduce/hadoop-example`
~~~~~~
## 将文件放入hdfs
hadoop fs -put word-in.txt hdfs://itcast:9000/words

## 查看 hdfs
hadoop fs -cat hdfs://itcast:9000/words

## 运行 wordcount 程序, 将 words 文件进行词频统计, 输出到 hdfs://itcast:9000/words-out 文件夹
hadoop jar hadoop-mapreduce-example.jar wordcount hdfs://itcast:9000/words  hdfs://itcast:9000/words-out

## 查看 正在运行的命令
jps

## 查看运行后的结构
hadoop fs -cat hdfs://itcast:9000/words-out/part-r-00000
~~~~~~

# SSH 协议 #

~~~~~~
# 登陆
ssh 192.168.1.208

# 执行命令
ssh 192.168.1.208 mkdir /itcast1008

# 免密码登陆
# 生成私钥和公钥
ss-keygen -t rsa

# 拷贝公钥到远程服务器
ssh-copy-id 192.168.1.208
~~~~~~
