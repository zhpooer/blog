title: 传智播客day69-搭建Hadoop集群
date: 2014-08-16 09:17:30
tags:
- hadoop
- 传智播客
---

# ZooKeeper #

ZooKeeper 是 Google 的 Chubby 的一个开源实现, Hadoop的分布式协调服务

负载均衡, 分布式锁, 协调服务, 数据同步, 数据发布与订阅

为了保证服务安全, 可以将 ZooKeeper 布置位一个集群,
一个 Leader Server 和 多个 Follower Server,
一般要配置奇数台机器

客户端连接 ZooKeeper, 一个 Follower Server 数据发生变化,
会自动同步到其他 ZooKeeper 服务器

为什么使用
* 大部分分布式引用需要一个主控, 协调器或控制器来管理物理分布的子进程(如资源, 任务分配)
* 提供通用的分布式锁服务, 用以协调分布式应用

在Hadoop中保证整个集群只有一个活跃的 NameNode, 存储配置信息. 在Hbase集群中, 确保 HMaster

## 部署单节点 ZooKeeper ##

1. 切换用户到 hadoop
2. 安装 ZooKeeper 到 `/itcast` 目录, `chown -R hadoop:hadoop /itcast`
3. `mv zoo_sample.cfg zoo.cfg`
~~~~~~
dataDir=/itcast/data
~~~~~~
4. 启动 ZooKeeper 服务, `zkServer.sh`
5. 连接 ZooKeeper `zkcli.sh`
~~~~~~
# 创建 sh_0329 文件夹
create /sh_0329 $8000
# 获取 sh_0329 存储的信息
get /sh_0329
~~~~~~
6. 停止 `zkServer.sh stop`

目录结构
* `bin`, 可执行文件
* `conf`, 配置文件

## 搭建 ZooKeeper 集群 ##

1. 修改 `zoo.cfg`
~~~~~~
# 心跳时间
tickTime=2000
# 容忍心跳时间
initLimit=10
# 数据同步时间限制
syncLimit=5
# 运行时,目录
dataDir=/data
# 端口
clientport=2081

# ZooKeeper 之间的关系
# 数字代表id, 主机名或者ip:端口(learder follower 通信端口):端口(选举端口)
server.4=itcast04:2888:3888
server.5=itcast05:2888:3888
server.6=itcast06:2888:3888
~~~~~~
2. 在 数据保存文件夹 `/data`, 创建文件 `myid`, 对应 `server.N` 的 `N`
~~~~~~
4
~~~~~~
3. 拷贝文件到其他机器
~~~~~~
scp -r /itcast/zookeeper hadoop@itcast05:/itcast/
~~~~~~
4. 修改远程 ZooKeeper 的 `myip`
5. 分别启动各个机器上的 ZooKeeper `zkServer.sh start`

因为是通过选举来决定 Leader, 所以必须保证 `n/2+1` 台运行, 而且一定要配置奇数个 ZooKeeper

# hadoop 集群搭建 #

CDH 认证, 大数据专业认证

TODO: 集群分布
* 两台运行 NameNode, 和 zkfc(失败控制, ZooKeeper组件)
* 两台运行 ResourceManager
* 三台运行 DataNode, NodeManager, JournalNode(存放共享 edits), QuorumPeerMain(ZooKeeper主程序)

hadoop2.0 中, NameService = 主NameNode + 备NameNode, 元数据会进行同步(通过 JournalNode 或 nfs, 共享 edits),
通过 ZooKeeper服务进行协调, 多个 NameService 可以横向扩展

JournalNode, 用来存放共享 edits, 使 NameService 进行数据同步, 要求必须存在奇数个.

jkfc 负责监控 NameNode, 向 ZooKeeper 汇报信息, 一旦发现 主NameNode当机,
监视 副NameNode 的 jkfc 启动 副NameNode


修改Hadoop配置文件
1. `hadoop-env.sh`
~~~~~~
export Java_Home=...
~~~~~~  
2. `core-site.xml`
~~~~~~
<configuration>
  <!-- nameservice 地址 -->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://ns2</value>
  </property>
  <!-- 运行时产生文件,非临时, 重要! -->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/tmp/hadoop</value>
  </property>
  <!-- 设置 zookeeper 连接地址 -->
  <property>
    <name>ha.zookeeper.quorum</name>
    <value>itcast04:2181,itcast05:2181,itcast07:2181</value>
  </property>
</configuration>
~~~~~~
3. `hdfs-site`
~~~~~~
<configuration>
  <!-- 指定 nameservice  -->
  <property>
    <name>dfs.nameservice</name>
    <value>ns1</value>
  </property>
  <!-- ns1下的两个 namenode -->
  <property>
    <name>dfs.ha.namenodes.ns1</name>
    <value>nn1, nn2</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.ns1.nn1</name>
    <value>itcast01:9000</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ns1.nn1</name>
    <value>itcast01:50070</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.ns1.nn2</name>
    <value>itcast01:9000</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.ns1.nn2</name>
    <value>itcast01:50070</value>
  </property>
  <!-- journal 主机 -->
  <property>
    <name>dfs.namenode.shareed.edits.dir</name>
    <value>qjournal://itcast06:8485;itcast06:8485;itcast07:8485/ns1</value>
  </property>
  <!-- journal 自动备份在硬盘 -->
  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/itcast/journal</value>
  </property>

  <!-- 开启自动切换  -->
  <property>
    <name>dfs.ha.automatic-failover.enabled</name>
    <value>true</value>
  </property>
  <!-- TODO 自动切换实现方式 -->
  <property>
    <name>dfs.client.failover.proxy.provider</name>
    <value></value>
  </property>
  <!-- sshfence隔离机制 -->
  <!-- sshfence: 一台 nameNode 出错但是没有终止, 另一台 NameNode 主动通过ssh 终止他   -->
  <!-- TODO: shell: 一台 namenode 当掉, 从 namenode 通过 shell 切换到 active   -->
  <property>
    <name>dfs.ha.fencing.methods</name>
    <value>
      sshfence
      shell(/bin/true)
    </value>
  </property>
  <property>
    <name>dfs.ha.fencing.ssh.privaate-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa </value>
  </property>
  <!-- 隔离机制 超时时间 -->
  <property>
    <name>dfs.ha.fencing.ssh.connects-timeout</name>
    <value>30000 </value>
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
  <!-- 开启 ha  -->
  <property>
    <name>yarn.resourcemanager.ha.enabled</name>
    <value>true</value>
  </property>
  <!-- 指定集群id -->
  <property>
    <name>yarn.resourcemanager.cluster-id</name>
    <value>yrc</value>
  </property>
  <!-- RM 名字 -->
  <property>
    <name>yarn.resourcemanager.ha.rm-ids</name>
    <value>rm1, rm2</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname.rm1</name>
    <value>itcast3</value>
  </property>
  <property>
    <name>yarn.resourcemanager.hostname.rm2</name>
    <value>itcast04</value>
  </property>
  <!-- TODO zk集群个地址 -->
  <property>
    <name>yarn.resourcemanager.???</name>
    <value>itcast04:2181,itcast05:2181,itcast05:2181</value>
  </property>

  <!-- reduce 获取数据的方式 -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
~~~~~~
6. 添加`slaves`文件
~~~~~~
itcast05
itcast06
itcast07
~~~~~~
7. 拷贝配置好的文件到主机 `itcast01-06`

启动集群
1. 在05, 06, 07上启动 Zookeeper
2. 在05, 06, 07上启动, 启动 JournalNode, `hadoop-deamon.sh start journalnode`
3. 在 01 上初始化 namenode `hdfs namenode -format`, 并拷贝 `/hadoop/tmp` 到 02 机子上
4. 格式化 Zookeeper, `hdfs zkfc -formatZK`, 在一台机子上, 会在 zk 上产生目录 `hadoop-ha`
5. 在01上启动hdfs, `start-dfs.sh`, 所有机子上的 DataNode 服务都会自动启动, `itcast:50070` 查看 NameNode 状态
6. 在 03 上启动yarn(ResourceManager), `start-yarn.sh`, 在 04 上启动 `yarn-daemon.sh start resourcemanager`,
访问 `itcast:8088` 查看 ResourceManager 状态


# 动态增加节点 #

`hadoop-deamon.sh start datanode`

如果 一台datanode当掉, 那么其他datanode 会自动同步数据到新的一台 datanode,
以至于所有数据块都保持3份


# 程序连接 NameService #
~~~~~~
public static void main(){
    Configuration conf = new Configuration();
    conf.set("fs.defaultFS", "hdfs://ns1");
    conf.set("dfs.nameservices", ns1);
    conf.set("dfs.ha.namenodes.ns1", "nn1,nn2");
    conf.set("dfs.ha.rpc-address.ns1.nn1", "itcast1:9000");
    conf.set("dfs.ha.namenodes.rpc-address.ns1.nn2", "itcast2:9000");
    // TODO 
    conf.set("dfs.client.failover.proxy.provider.ns1", )
    // 使用 hadoop 用户登陆
    FileSystem fs = FileSystem.get(new URI("hdfs://ns1"), conf, "hadoop");
}
~~~~~~

TODO: 最后20分钟

