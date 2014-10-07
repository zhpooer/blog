title: HBase-入门
date: 2014-09-19 09:19:13
tags:
- HBase
---


# HBase #

Hadoop Database, 建立在 Hadoop 之上的数据库， 高可靠, 面向列, 可伸缩的
分布式存储系统.

利用 Hbase 可以利用廉价的机器搭建大规模结构化存储集群.

# 相关概念 #

## Row Key ##

主键， 用来检索记录的主键， 访问 hbase table中的行，只有三种方式
* 通过单个 row key 访问
* 通过 row key 的 range

## Column Family ##

列族在创建表的时候申明， 一个列族可以包含多个列， 列中的数据都是以二进
制形式存在， 没有数据类型

## 时间戳(timestamp) ##

HBase中通过row和columns确定的唯一一个存储单元称为cell（可以存多个数据），每个cell都
保存这同一个数据的多个版本。版本通过时间戳来索引。

# 安装运行 #

* 修改hbase配置文件 `hbase-env.xml`, JDK 目录
* `hbase-site.xml`
~~~~~~
<property>
    <name>hbase.rootdir</name>
    <value>file:///root/habse</value>
</property>
~~~~~~
* 启动hbase，`start-hbase.sh`
* 启动运行命令行 `hbase shell`

# HBase shell #

~~~~~~
# 创建表， 以及两个列族
create 'people', {NAME=>'INFO', VERSIONS => 3}, {NAME => 'data',
VERSIONS => 1};

# 显示表
list;

# 显示描述信息
describe 'people';

help 'dml';
help 'dll';

# 插入数据 在 row: rk0001, column: info=>name 插入 value： cls
put 'people', 'rk0001', 'info:name', 'cls';
scan 'people';

# 添加数据
put 'people', 'rk0001', 'info:gender', 'male';
put 'people', 'rk0001', 'info:size', '36';

# 给data列族添加数据
put 'people', 'rk0001', 'data:torrent', 'seed';

put 'people', 'rk0002', 'info:name', 'bdyjy';

# 返回两行
scan 'people';
# 可以在列族下任意添加列

# 根据 version 查找 info 列族下想信息
scan 'people', {columns => 'info', version => 3};

# 超过版本号的版本会被删除， 但是内存中还是会保留， 还是可以被查询
# 查询内存中的所有信息
scan 'people', {columns => 'info', RAW => true};

~~~~~~

# 搭建 HBase 集群 #

1. 启动zookeeper
2. 启动 hdfs
3. 配置 HBase, 
  1. `hbase-site.xml`
~~~~~~
<property>
  <name>hbase..rootdir</name>
  <value>hdfs://ns1/hbase</value>
</property>
<-- 开启分布式 -->

<-- 指定zookepper地址 -->
~~~~~~
  2.`hbase-env.conf`
~~~~~~
# 不使用自带 zk
export HBASE_MANAGERS_ZK=false
~~~~~~  
  3. `regionservers`, 指定从服务器
~~~~~~
itcast03
itcast04
itcast05
itcast06
~~~~~~
  4. 将 hadoop的配置文件 `core-site.xml` `hdfs-site.xml` 复制到 hbase 的配置文件目录下
4. 启动Hbase， `start-hbase.sh`, 会分别启动 03 04 05 06 上分别启动从服
务器， 可以在 02 机器上 启动备份 Master `hbase-deamon.sh start master`
5. 通过 `localhost:60010` 访问管理界面

# HBase 基础知识 #

物理存储：
Table 的行的方向上分割多个 HRegion， 一个 region 由[startkey, endkey]
表示，按照字典顺序读取，每个 HRegion 分散在不同的 RegionServer

# Java操作HBase #

~~~~~
Configuration conf = HBaseConfiguration.create();
conf.set("hbase.zookeeper.qurorum", "itcast04f:2181;itcast05:2181");

HBaseAdmin admin = new HbaseAdmin(conf);
// 创建 people 表
HTableDescriptior htd = new
HTableDescriptor(TableName.valueOf("people"));
// 列族
HColumnDescriptor hcd_info = new HColumnDescriptor("info");
HColumnDescriptor hcd_data = new HColumnDescriptor("data");
htd.addFamily(hcd_info);
htd.addFamily(hcd_data);

admin.createTable(htd);
admin.close();
~~~~~

~~~~~
public void testPut(){
  Htable table = new Htable(conf, "people");
  Put put = new Put(Bytes.toBytes("kr0001"));
  put.add(Bytes.toBytes("info"), Bytes.toBytes("name"),Bytes.toBytes("cjk");
  put.add(Bytes.toBytes("info"), Bytes.toBytes("age"), Bytes.toBytes("14");
  table.put();
  table.close();
}

public void testPutAll(){
  Htable table = new Htable(conf, "people");
  List<Put> puts = new ArrayList<Put>();
  // 如果数据量过多， 内存会溢出
  for(int i=1;i<=1000; i++) {
    Put put = new Put(Bytes.toBytes("kr" + i));
    put.add(Bytes.toBytes("info"), Bytes.toBytes("name"),Bytes.toBytes("cjk")
    puts.add(put)
  }

  table.put(puts);
  table.close();
}

public void testGet(){
  Htable table = new Htable(conf, "people");
  Get get = new Get(Bytes.toBytes("kr999"));
  Result result = table.get(get);
  String r = Bytes.toString( result.getValue(Bytes.toBytes("info")) );
  table.close();
}

public void testGet(){
  Htable table = new Htable(conf, "people");
  // [)
  Scan scan = new Scan(Bytes.toBytes("kr800"), Bytes.toBytes("kr900"));
  ResultScanner scanner = table.getScanner(scan);
  for( Result result : scanner {
  }
  table.close();
}

public void testDel() {
  Htable table = new Htable(conf, "people");
  Delete delete = new Delete(Byte.toBytes("rk99"));
  table.delete(delete);
  table.close();
}
~~~~~

[HBase 结构介绍](http://jiajun.iteye.com/blog/899632)
