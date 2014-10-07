title: 传智播客day67-MapReduce
date: 2014-08-13 09:04:09
tags:
- hadoop
- 传智播客
---

# MapReduce #

MapReduce 计算海量数据, 将一个任务切分成多个小任务, 分给多个进程计算

* MapReduce 是一种分布式计算模型, 用户搜索领域, 解决海量数据计算的问题
* MR由两个项目阶段组成: Map和Reduce, 用户只需要实现 `map()`和`reduce`两个函数,
即可以实现分布式计算
* 两个函数的形参是key、value对, 表示函数的输入信息

## Demo ##
~~~~~~
# 启动 NameNode DataNode
start-dfs.sh
# 启动 ResourceManager
start-yarn.sh

# 启动 DataNode 上启动 RunJar 和 NodeManager(DataNode 上)
# 过程中启动 MRAppMaster, 监控任务(每个任务都会对应有一个), 监控 YarnChild
# YarnChild(DataNode上), 运行计算任务
hadoop jar hadoop-mapreduce-example.jar wordcount hdfs://itcast:9000/words  hdfs://itcast:9000/words-out
~~~~~~

运行流程
1. RunJar 把jar包上传到 HDFS, 默认写10份.
2. 客户端提供任务描述信息
3. ResourceManager 任务初始化, 将任务放到任务调度器
4. NodeManager 主动申请任务, 采用心跳机制(RPC)
5. NodeManager 启动 YarnChild进程, 进行计算, YarnChild 有Map, Reduce 对象

## 编写 MapReduce ##

TODO 插入 MapReduce 图片

`[K1, V1]` Map `[k2, V2]` Shuffle `[K2, {V2...}]` Reduce `[K3, V3]`

Map执行处理
1.读取输入文件的内容, 解析出 key value 对, 对输入的问价的每一行, 解析成key value对
每一个键值对调用一次 `map()`

reduce任务处理 TODO



    输入数据
      hello world
      hello hadoop
    Map阶段(key: 字符偏移量)
      <0, "hello world">
      <11, "hello hadoop">
      代码
        map(){
          String line = v1;
          String[] words = ling.split(" ");
          for(String w: words) {
            context.write(w, 1);
          }
        }
    Reduce 阶段(数据已经按key字符排序)
      <"hello", 1, 1>
      <"hadoop", 1>
      <"world", 1>
      代码:
        reduce(){
          String word = K2
          List list = V2
          for(int i : list) {
            counter += 1
          }
          context.write(word, counter)
        }
    输出数据
      hello 2
      world 1
      hadoop 1

## 编写代码 ##

导入jar包 `hadoop-common.jar`, `hadoop-mapreduce.jar`

~~~~~~
public class WordCount {
    public class void main() {
        // 可以通过 conf 设置, 副本拷贝数 默认是 10
        Configuration conf = new Configuration();
        
        Job job = Job.getInstance(conf);
        
        // 重要, 将 main 方法所在的类注册
        job.setJarByClass(WordCount.class);
        // Mapper 类
        job.setMapperClass(WCMapper.class);
        // 设置 K2 V2
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);
        // 设置读取文件
        FileInputFormat.setInputPaths(job, new Path("/word.txt"));
        
        job.setReducerClass(WCReducer.class);
        // 最终将数据到hdfs 的key的类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);
        // 设置输出文件, 如果文件已经存在, 抛错
        FileOutputFormat.setOutputPath(job, new Path("/wcount1"));
        
        // 不太好的方法job.submit();
        // true 打印执行详情
        job.waitForComletion(true);
    }
}

// Hadoop 提供了自己序列化机制
public class WCMapper extends Mapper<LongWritable, Text, Text, LongWritable>{
    protected void map(LongWritable key, Text value, Context context) {
        String line = value.toString();
        String[] words = line.split(" ");
        for(String w : words) {
            context.write(new Text(w), new LongWritable(1))
        }
    }
}

public class WCReducer extends Reducer<Text, LongWritable, Text, LongWritable> {
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) {
        // super.reduce
        long count = 0;
        for(LongWritable l : values) {
            counter += l.get();
        }
        context.write(key, new LongWritable(counter));
    }
}
~~~~~~

打成Jar包, 运行 `hadoop jar /root/mrs.jar`

# Debug MapReduce 程序 #

## 本地调试 ##

在eclipse中调试, 并没有提交到集群

修改 `Path`
~~~~~~
// 读取本地
new Path("/root/words")
// 或 读取 hdfs 上的
new Path("hdfs://itcast:9000/words")
~~~~~~

## MR流程 ##

* 代码编写
* 作业配置
  * 在 `etc/mapred-site.xml` 配置全局变量
  * 在 `Configuration` 中设置局部变量
* 提交作业, `hadoop jar **.jar`
* 初始化作业
* 分配任务
* 执行任务
* 更新状态和任务
* 完成作业

TODO 运行状态图

# 序列化 #

序列化是把结构化对象转换为字节流

反序列化(Deserialization)将字节流转换为结构化的对象

因为 JDK 序列化机制效率太低, JDK序列化要记录额外的数据, 如继承结构, 而 Hadoop 需要只需要传递数据,
以至于 Hadoop 要重新实现序列化

Hadoop 序列化特点
* 紧凑, 高效的使用存储空间
* 快速, 读写数据的额外开销小
* 可扩展, 透明读取老的数据格式
* 互操作, 支持多语言交互

Hadoop 序列化要实现接口 `Writable`

# Demo 统计用户上网流量 #

测试数据

    电话号码     上行流量 下行流量
    13888888888  2000      1000
    13988888888  1000      6000
    13988888888  2000      5000
    13988888888  3000      4000

运行结果

    号码   上行流量    下行流量    总流量
    138    2000        1000        3000
    139    6000        15000       21000

~~~~~~
public class DataCount {

    public static void main(String[] args) {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJarByClass(DataCount.class);

        job.setMapperClass(DCMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(DataBean.class);
        FileInputFormat.setInputPaths(job, new Path(args[0]));

        job.setReducerClass(DCReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(DataBean.class);
        FileInputFormat.setOutputPaths(job, new Path(args[1]));

        job.waitForCompletion(true);
    }
}

public class DataBean implements Writable {
    private String account;
    private String upPayLoad;
    private String downPayLoad;
    private String totalPayLoad;

    public DataBean(){}
    
    // 构造函数
    public DataBean(String account, long upPayLoad, long downPayLoad){
        super();
        this.account = account;
        this.upPayLoad = upPayLoad;
        this.downPayLoad = downPayLoad;
        this.totalPayLoad = upPayLoad + downPayLoad;
    }
    
    // 在序列化时要注意 1. 类型, 2. 顺序
    @Override
    public void write(DataOutput out) {
        out.writeUTF(account);
        out.writeLong(upPayLoad);
        out.writeLong(downPayLoad);
        out.writeLong(totalPayLoad);
    }
    
    @Override
    public void readFields(DataInput in) {
        account = in.readUTF();
        upPayLoad = in.readLong();
        downPayLoad = in.readLong();
        totalPayLoad  = in.readLong();
    }
    
    @Override public void toString(){
        // TODO
    }
}

public class DCMapper extends Mapper<LongWritable, Text, Text, DataBean> {
    @Override protected void map(LongWritable key, Text value, Context context) {
        String ling = value.toString();
        String[] fields = line.split("\t");
        String account = fields[1];
        long up = Long.parseLong(fields[8]);
        long down = Long.parseLong(fields[9]);
        context.write(new Text(account), new DataBean("", up, down));
    }
}

public class DCReduce extends Reducer<Text, DataBean, Text, DataBean> {
    @Override protected void reduce(Text key, Iterable<DataBean> values, Context context) {
        long up_sum = 0;
        long down_sum = 0;
        for(DataBean bean : values) {
            up_sum +=  bean.getUpPayLoad();
            down_sum +=  bean.getDownPayLoad();
        }
        context.write(key, new DataBean(key, upPayLoad, downPayLoad));
    }
}
~~~~~~

如果想要在集群上运行, 安装 hadoop 插件

# 源码分析 #

1. 初始化DistributeFS, 获取 HDFS 代理对象(DFSClient)
2. 启动Job, 获取 ResourceManager 代理对象(Cluster), 通过ResourceManager 获取 jar包路径, 和jobId
3. 提交jar包, 默认10份
4. 提交信息给 ResourceManager, jar包路径, jobId
5. NodeManager 通过心跳来申请任务
6. 启动 YarnChild, 下载Jar包
7. 启动任务

一个 Reducer 对应一个结果文件, 可以通过配置来设置 Reducer 个数. Mapper不能设置个数.
