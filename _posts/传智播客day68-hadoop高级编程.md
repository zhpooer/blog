title: 传智播客day68-Hadoop高级编程
date: 2014-08-14 08:52:42
tags:
- 传智播客
- hadoop
---


# Partitioner #

MapReduce 通过 Partitioner 来进行分区, 如按 月份, 地区进行分区

~~~~~~
// 对电话号码进行 服务商分区
// 在 Mapper 执行完成, Reducer 还没有开始时, 执行
public class ProviderPartitioner extends Partitioner<Text, DataBean>{
    private static Map<String, Integer> providerMap = new HashMap<String, Integer>();
    static {
        // 移动
        providerMap.put("135", 1);
        providerMap.put("136", 1);
        providerMap.put("137", 1);
        providerMap.put("138", 1);
        providerMap.put("139", 1);
        // 联通
        providerMap.put("150", 2);
        providerMap.put("159", 2);
        // 电信
        providerMap.put("182", 3);
        providerMap.put("183", 3);
        providerMap.put("187", 3);
    }
    // numPartitions 有几个 partition
    public int getPartition(Text key, DataBean value, int numPartitions) {
        String account = key.toString();
        String sub_acc = account.subString(0, 3);
        Integer code = providerMap.get(sub_acc);
        if(code == null) {
            code = 0;
        }
        // 如果返回值是0, 存到0号 reducer
        return code;
    }
}

// main
job.getJobInstance(conf);
// 本地模式, 只能启动一个 reducer, partitioner 会启动多个 reducer
job.setPartitionerClass(ProviderPartitioner.class);

// 设置 reducer 的数量, 如果不设置, 默认启动一个, 文件会被写到一个结果文件, 不会被分区
// 若设置的Reducer 超过分区数, 那么会产生 多余的结果文件, 且多余结果文件会是空的
// 如果 Reducer 小于分区数, 那么会抛错
job.setNumReduceTasks(4);
job.waitForCompletion();

~~~~~~


# 排序 #

*数据*

| 账号 | 收入 | 支出 | 日期 |
|-------------------------|
| zhangsan@163.com | 6000 | 0   | 2014-02-20 |
| lisi@163.com     | 2000 | 0   |2014-02-20  |
| lisi@163.com     |    0 | 100 | 2014-02-20 |

*求和过程, 中间结果*

| 账号 | 收入 | 支出 | 结余 |
|-------------------------|
| zhangsan@163.com | 6000 | 0   | 6000 |
| lisi@163.com     | 2000 | 100   | 1900 |

*根据收入进行排序, 在收入相同的情况下, 根据支出排序*

| 账号 | 收入 | 支出 | 结余 |
|-------------------------|
| lisi@163.com     | 2000 | 100   | 1900 |
| zhangsan@163.com | 6000 | 0   | 6000 |


## Maven 建立工程 ##


TODO: pom 文档

`pom.xml`
~~~~~~
<dependencies>
   <dependency>
       <groupId>org.apache.hadoop </groupId>
       <artifactId> hadoop-common</artifactId>
       <version> 2.4.1</version>
   </dependency>
   <dependency>
       <groupId>org.apache.hadoop </groupId>
       <artifactId>  hadoop-mapreduce-cl </artifactId>
       <version> 2.4.1 </version>
   </dependency>
</dependencies>
~~~~~~


~~~~~~
// 求和程序
public class SumStep {
    public static void main(){
        Configuration conf = new Configuration()
        Job job = job.getInstance(conf);
        job.setJarByClass(SumStep.class);
        job.setMapperClass(SumMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(InfoBean.class);
        FileInputFormat.setInputPaths(job, new Paht(args[0]));

        job.setReducerClass(SumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(InfoBean.class);
        FileInputFormat.setOutputPaths(job, new Paht(args[1]));
        
        job.waitForComletion(true);
    }
    
}

// 实现序列化和排序功能
public class InfoBean extends WritableComparable<InfoBean>{
    private String account;
    // 如果是账目, 应该用 BigDecimal
    private double income;
    private double expenses;
    // 结余
    private double surplus;
    @Override protected void write(DataOutput out) {
        out.writeUTF(account);
        out.writeDouble(income);
        out.writeDouble(expenses);
        out.writeDouble(surplus);
        
    }
    
    public void set(String account, double income, double expenses) {
        this.account = account;
        this.income = income;
        this.expenses = expenses;
        this.surplus = inomce - expenses;
    }
    
    @Override
    protected void readFields(DataInput in) {
        this.account = in.readUTF();
        this.income = in.readDouble();
        this.expenses = in.readDouble();
        this.surplus = in.readDouble();
    }

    @Override public int compareTo(InfoBean o) {
        if(this.income == 0.getIncome()) {
            return this.expenses > o.getExpenses() ? 1 : -1;
        } else {
            return this.income > o.getIncome() ? -1 : 1;
        }
    }
    
    @Override public String toString(){}
}

public class SumMapper extends Mapper<LongWritable, Text, Text, InfoBean> {
    // 虽然是有状态的变量, 但是它们设置完之后会马上序列化
    private Text k = new Text();
    private InfoBean v = new InfoBean();
    
    @Override protected void map(LongWritable key, Text value, Context context) {
        String line = value.toString();
        String[] fields = line.split("\t");
        String account = fields[0];
        double in = Double.parseDouble(fields[1]);
        double out = Double.parseDouble(fields[2]);
        k.set(account);
        v.set(account, in, out);
        context.write(k, v);
    }
}

public class SumReducer extends Reducer<Text, InfoBean, Text, InfoBean>  {
    @Override protected void reduce(Text key, Iterable<InfoBean> values, Context context){
        double in_sum = 0;
        double out_sum = 0;
        for(InfoBean bean : values) {
            in_sum += bean.getIncome();
            out_sum += bean.getExpenses();
        }
        v.set("", int_sum, out_sum);
        context.write(key, v);
    }
}
~~~~~~

排序方法
~~~~~~
public class SortStep {
    public static void main() {
        // TODO
    }
}

// 根据输出的 key 进行排序, NullWritable
public class SortMapper extends Mapper<LongWritable, Text, InfoBean, NullWritable> {
    private InfoBean k = new InfoBean();
    @Override protected void map(LongWritable key, Text value, Context context) {
        String line = value.toString();
        String[] fields = line.split("\t");
        String account = fields[0];
        double in = Double.parseDouble(fields[1]);
        double out = Double.parseDouble(fields[2]);
        k.set(account, in, out);
        context.write(k, NullWritable.get());
    }
}

public class SortReducer extends Reducer<InfoBean, NullWritable, Text, InfoBena> {
    private Text k = new Text();

    @Override protected void reduce(InfoBean bean, Interable<NullWritable> value, Context context) {
        String account = bean.getAccount();
        k.set(account);
        context.write(k, bean);
    }
}
~~~~~~

# Combiners 编程 #

Combiners 是一个特殊的 Reducer, 在Map端进行一次简单的 Reducer.
使Reducer端的减少计算. 如果所有结果都是reduce完成, 效率会相对低下.
使用combiner, 先完成的map会在本地聚合, 提升速度

Combiners 是可插拔的, 绝不能改变最终的计算结果

可以用 Combiners 来过滤数据

~~~~~~
job.setCombinerClass(WCReducer.class);
~~~~~~

# Shuffle #

MapReduce 的核心, 是指 reducer 获取 Mapper 的输出数据的过程

一个输入切片对应一个 Mapper, 一个Mapper 对应一个 数据缓存区(默认大小100M),
当数据缓存区数据达到域值(80%), 那么通过 partitioner 对数据进行分区, 按键排序, 写入磁盘,
生成分区且排序的小文件. 最后对小文件进行合并, 形成分区且合并的大文件, 并汇报给 MrAppMaster 

Reducer 通过 MrAppMaster(JobTracker) 获取数据信息,  一号 Reducer 取一号分区中的数据,
二号 Reducer 取二号分区中的数据, 然后进行合并, 执行 Reduce, 输出数据

在 Hadoop1.0 中, 由 JobTracker 来监控任务, 管理资源, JobTracker 下辖多个 TaskTrack

在 Hadoop2.0 中, 由 ResourceManager 来管理资源, ResourceManager 下辖多个 NodeManager,
任务启动时, 启动一个 MrAppMaster 监控任务运行

Hadoop2.0 中, 将 JobTracker 职能分成 MrAppMaster 和 ResourceManager,
提交一个一个任务, 就有一个 AppMaster

# 倒排索引 MapReduce #

统计某个单词在文章中出现了多少次

`a.txt`

    hello world
    hello Hadoop

`b.txt`

    hello ruby
    hello world

结果

    hello a.txt:2 b.txt:2
    world a.txt:1 b.txt:2
    Hadoop a.txt:1
    ruby a.txt:1


~~~~~~
public class InverseIndex {
    public static void main(String [] args){
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJarByClass(InverseIndex.class);

        job.setMapperClass(IndexMapper.class);
        job.setMapperOutputKeyClass(Text.class);
        job.setMapperOutputValueClass(Text.class);
        FileInputFormat.setInputPaths(job, new Path(args[0]));

        job.setReducerClass(IndexReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputKeyClass(Text.class);
        FileOutputFormat.setOutputPaths(job, new Path(args[1]));
        
        job.setCombinerClass(IndexCombiner.class);
        job.waitForCompletion();
    }
}

public class IndexMapper extends Mapper<LongWritable, Text, Text, Text> {
    private Text k = new Text();
    private Text v = new Text();
    @Override protected void map(LongWritable key, Text value, Context context) {
        String line = value.toString();
        String[] words = line.split(" ");
        // 得到输入切片
        FileSplit fileSplit = (FileSplit)context.getInputSplit();
        
        // 得到文件名 hdfs://itcast:9000
        String path = fileSplit.getPath().toString();
        for(String w:words) {
            k.set(w + "->" + path);
            v.set("1");
            context.write(k, v);
        }
    }
}

public class IndexCombiner extends Reducer<Text, Text> {
    private Text k = new Text();
    private Text v = new Text();
    @Override protected void Reduce(Text key, Iterable<Text> value, Context context) {
        String[] workdAndPath = key.toString().split("->");
        String word = wordAndPath[0];
        String path = wordAndPath[1];
        int counter = 0;
        for(Text t : values) {
            counter += Integer.parseInt(t.toString());
        }
        k.set(word);
        v.set(path + "->" + counter);
        context.write(k, v);
    }
}

public class IndexReducer extends Reducer {
    @Override protected void reduce(Text key, Iterable<Text> values, Context context) {
        String result = "";
        for(Text t: values) {
            result += t.toString() + "\t";
        }
        v.set(result);
        context.write(key, v);
    }
}
~~~~~~

# 切片大小 #

在任务运行任务时, 会下载分片下载数据默认3128M,
所以系统会根据 block 块生成对应个数的 Map 程序
