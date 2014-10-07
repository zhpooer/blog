title: 传智播客day66-Hadoop HDFS
date: 2014-08-11 08:54:29
tags:
- 传智播客
- hadoop
---

# 分布式文件系统与HDFS #

客户端(Client) 查询 NameNode(记录文件存储信息), 将数据放入datanode(多个)或从中取出  
数据在上传过程中要进行冗余保存, datanode 自行进行水平复制.(流水线复制)  
上传过程中, 文件会被分块, 每块128M, 其实是对块的冗余存储


* 数据量越来越多, 在多个操作系统中协调
* 允许文件通过网络在多台主机上分享文件系统, 增加机器数量来扩张
* 通透性, 在用户和程序看来, 就像访问本地的磁盘一样
* 容错, 某些节点脱机, 可以持续运作, 不会由数据损失
* 分布式文件管理系统有很多种, hdfs 只是其中一种. 适用于一次写入,
多次查询的情况, 不支持并发写(同一个文件分块同时上传), 小文件不合适

分布式文件系统
* GFS
* HDFS
* Lustre
* Ceph
* GriddFs
* TFS

谷歌三大论文(BigTable, MapReduce, GFS)

# HDFS的shell操作 #

~~~~~~
# 启动 HDFS
start-dfs.sh

# 列出文件
hadoop fs -ls hdfs://itcast:9000/

# 取出文件
hadoop fs -get hdfs://itcast:9000/jdk ~/temp/jdk
# 也可以这么写
hadoop fs -get /jdk ~/temp/jdk

hadoop fs -cat /words.avi

# 追加文件到 word.avi
hadoop fs -appendToFile ~/appendix /words.avi

# 查看ls 帮助
hadoop fs -help ls
hadoop fs -ls /wc* -h
# 递归显示
hadoop fs -ls /wc* -R

# 改变所属组和所属用户
hadoop fs -chown supergroup:supergroup  /jdk
hadoop fs -chgrp root  /jdk

# 改变权限
hadoop fs -chmode u+w /wcout
# 递归改变权限
hadoop fs -chmode u+w /wcout -R

# 拷贝本地文件到 hdfs 相当于 put
hadoop fs -copyFromLocal /home/hadoop/a.txt /
# 获得远程文件到本地
hadoop fs -copyToLocal /a.txt /tmp/

# 列出文件数, 文件夹数, 总大小
hadoop fs -count /

# 远程拷贝
hadoop fs -cp /a.txt /b.txt

# 剪切
hadoop fs -cp /a.txt /b.txt

hadoop fs -du
hadoop fs -df

hadoop fs -mkdir -p /abc
hadoop fs -rm /abc -r

# 合并 a.txt b.txt 到 c.txt
hadoop fs -getmerge /a.txt /b.txt /tmp/c.txt

# 查看尾部内容
hadoop fs -tail /a.txt

hadoop fs -text /a.txt | more

# 设置3个副本, 如果只有一台机器, 还是存了一份
hadoop fs -setrep 3 /a.txt
~~~~~~

老版本中所有命令都包含在 `hadoop` 命令中, 新版本中`hdfs dfs`代替 `hadoop fs`

# 体系结构和基本概念 #


* NameNode, 索引节点, 存放文件的描述性信息(medadata)
* DataNode, 数据节点, 可以有许多个
* Secodary NameNode, Name node的帮助节点, 在Hadoop 2.0中已经去除, 但是伪分布式中还会存在

## 元数据存储细节 ##

NameNode包含
* FileName
* replicas
* block-ids
* id2host
* 其他


    metadata: /test/a.log, 3, {blk_1, blk_2}, [{blk_1: [h0, h1, h3]}, {blk_2: [h0, h2, h4]}]
    文件/test/a.log, 有三个副本, 分成两块{blk_1, blk_2}, 分别被存在 [h0....]


NameNode 是整个文件系统的管理节点. 维护着整个节点的目录树.
文件/目录的元数据和每个文件对应的数据块列表, 接受用户的操作请求.

文件包括:
* fsimage 元数据镜像文件, 存储着某一时段 NameNode 内存元数据信息.
序列化写入到磁盘; 1.0非实时同步, 但是2.0可以通过设置实现
* edits, 操作日志文件, 记录用户的操作日志
* fstime, 保存最近一次checkpoint的时间, 上一次数据同步的时间点; 内存数据和磁盘数据同步的时间点;


工作特点
* 始终在内存中保存metedata
* 有写请求到来时, namenode会首先写editlog到磁盘. 成功返回后, 修改内存(metedata), 返回客户端
* namenode 维护 fsimage 文件, 不会随时与 metedata 同步, 每隔一段时间通过合并edits 文件来更新内容.
SecondaryNameNode 合并 fsimage 和 edits 来完成工作

## SecondaryNameNode ##

Hadoop2 中已经不使用这种方法同步

HA(高可靠性) 解决方案, 不支持热备份(数据实时同步)

执行过程, 从Namenode下载数据信息(fsimage, edits), 然后把二者合并,
生成新的fsimage, 返回给NameNode. 通常部署到两个节点

以下两个任意两个参数满足, 就会启动合并
* `fs.checkpoint.period` 指定两次checkpoint的最大时间间隔, 默认 3600秒
* `fs.checkpoint.size`, edits size 的最大值, 默认为64M


## DataNode ##

提供真实文件的存储服务

文件块(block): 最基本的存储单位. Hdfs1.0默认大小为64M, Hdfs1.0默认大小为128M.

HDFS中, 如果一个文件不小于数据块大小, 并不会占用整个block

replication, 多副本, 默认3块

# java接口与常用API #

导入jar包 `hdfs.jar common.jar`

~~~~~~
public class HDFSDemo {

    FileSystem fs = null;
    @Before public void init() {
        fs = FileSystem.get(new URI("hdfs://itcast:9000"), new Configuration());
    }
    
    @Test public void testDownload{
        InputStream in = fs.open(new Path("/jdk7"));
        OutputStream out = new FileOutputStream("/tmp/jdk4");
        // true, 拷贝完成自动关闭
        IOUtils.copyBytes(in, out, 4096, true);
    }

    @Test public void testUpload() {
        InputStream in = new FileInputStream("/tmp/test");
        OutputStream out = fs.create(new Path("/in.log"));
        IOUtils.copyBytes(in, out, 4096, true);
    }

    @Test public void testMkDir() {
        fs.mkdirs(new Path("/itcast/shanghai"));
    }

    @Test public void testDel() {
        // 是否递归删除
        Boolean flag = fs.delete(new Path("/jdk7"), false);
    }

   @Test public void testExist() {
       fs.exists(new Path("/jdk7"));
   }

}
~~~~~~

# RPC 机制 #

RMI 效率低

Remote Procedure Call, 远程过程调用协议

datanode 与 namenode 之间通信(心跳检测)使用RPC

Client 与 namenode 之间通信使用RPC

Client 与 datanode 之间使用 HTTP

~~~~~~
public interface Bizable {
    // 初始化时需要版本号
    public static final long versionID = 10010l;
    public String sysHi(String name);
}

public class RPCServer implements Bizable{
    public String sysHi(String name) {
        return "Hi ~" + name;
    }
    
    public static void main() {
        Server serveer = new RPC.Builder(new Configuration())
                               .setInstance(new RPCServer())
                               .setProtocol(Bizable.class)
                               .setBindAddress("192.168.1.101")
                               .setPort(9527)
                               .build();
        Server.start();
    }
}

public class RPCClient() {
    public static void main() {
        Bizable proxy = RPC.getProxy(Bizable.class, 10010,
                               new InetSocketAddress("192.168.1.101", 9527), new Configuration());
        String result = proxy.sysHi("world");
        RPC.stopProxy(proxy);
    }
}
~~~~~~

NameNode 和 DataNode 都是一个 main 方法

# 源码分析 #

`FileSystem` 通过反射生成 实际子类.


# 远程debug #

JDPA, java远程调试框架

~~~~~~
hadoop-deamon.sh start dataNode
~~~~~~
