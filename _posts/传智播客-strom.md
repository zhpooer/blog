title: 传智播客-Storm
date: 2014-09-02 21:50:48
tags:
- storm
- 传智播客
---

# Storm #

Twitter 开源的一个分布式实时计算系统
* 分布式
* 可扩展
* 高可靠性
* 高效实时
* 编程模型简单

使用场景: 数据的实时分析, 持续计算, 分布式RPC

常用的类
* BaseRichSpout(消息生产者)
* BaseBasicBolt(消息处理者)
* TopologyBuilder(拓扑构建者)
* Values(将数据存放到values, 发送到下个组建)
* Tuple(发送的数据被封装到Tuple, 可以通过Tuple接受上个组建发送的消息)
* StormSubmitter, LocalCluster(拓扑提交器)

# WordCount #
~~~~~~
Toplogybuilder builder = new TopologyBuilder();
builder.setSpout("word-reader", new WordReader);
builder.setBolt("word-spliter", new WorldSpliter().shuffleGrouping("word-reader"));
builder.setBolt("word-", new WorldSpliter().shuffleGrouping("word-spliter"));
~~~~~~

# Storm 集群 #

组成
* Nimbus, storm 主节点
* zookeeper
* Supervisor, 子节点

安装, 运行
1. 安装 zeromq, jzmq, python, zookeeper
2. 配置运行 storm, `storm.yaml`, `./storm ui`
~~~~~~
storm.zookeeper.servers:
  - "master"
nimbus.host: "master"
~~~~~~
3. 访问 `localhost:192.168.3.250:8080`

启动集群
1. 主节点运行 `storm nimbus` 和 `storm ui`
2. 子节点运行 `storm supervisor`
3. 提交任务 `storm jar *.jar mainClass`
4. 查看任务 `storm list`
5. 停止任务 `storm kill taskname`

提交作业会在相应的子节点启动 worker, 可以手动设定 worker 启动个数,
也可以手动设定worker上启动的 bolt 个数

* 每个 Supervisor 上运行着若干个 worker 进程
* 每个 worker 进程中运行着各个 Excutor 线程
* 每个 Excutor 线程里面运行着若干个相同的Task(Spout/Bolt)

# 数据分发策略 #

* Shuffle Grouping：随机分组，随机派发stream里面的tuple，保证每个bolt接收到的tuple数目相同。
* Fields Grouping：按字段分组，比如按userid来分组，具有同样userid的tuple会被分到相同的Bolts，而不同的userid则会被分配到不同的Bolts。
* All Grouping：广播发送，对于每一个tuple，所有的Bolts都会收到。
* Global Grouping: 全局分组，这个tuple被分配到storm中的一个bolt的其中一个task。再具体一点就是分配给id值最低的那个task。
* Non Grouping：不分组，这个分组的意思是说stream不关心到底谁会收到它的tuple。目前这种分组和Shuffle grouping是一样的效果，有一点不同的是storm会把这个bolt放到这个bolt的订阅者同一个线程里面去执行。
* Direct Grouping：直接分组,  这是一种比较特别的分组方法，用这种分组意味着消息的发送者指定由消息接收者的哪个task处理这个消息。只有被声明为Direct Stream的消息流可以声明这种分组方法。而且这种消息tuple必须使用emitDirect方法来发射。消息处理者可以通过TopologyContext来获取处理它的消息的taskid (OutputCollector.emit方法也会返回taskid)
* Local or shuffle grouping：如果目标bolt有一个或者多个task在同一个工作进程中，tuple将会被随机发生给这些tasks。否则，和普通的Shuffle Grouping行为一致

# 对象生命周期 #

spout 方法调用顺序
1. declareOutputField()
2. open()
3. active()
4. nextTuple()
5. deactive()

bolt 方法调用顺序
1. declareOutputFields()
2. prepare()
3. execute()



