title: 传智播客day64-NoSql
date: 2014-08-09 09:01:04
tags:
- 传智播客
- NoSql
---

# NoSql 简介 #

Not Only SQL, 一系列非关系型数据库的总称

关系型数据库中的表存储一些格式化数据结构,
每条记录的字段的组成都一样, 即使不是每条记录都需要所有字段

非关系型数据库以键值对存储, 结构不固定, 每一条记录可以有不一样的键,
每条记录可以根据需要增加一些自己的键值对.

常见的Nosql
* CouchDB
* Redis
* Neo4j
* HBase
* BigTable
* Tair

优势
* 简单扩展
* 快速读写
* 低廉成本
* 灵活的数据模型 json

不足
* 不提供对SQL的支持
* 特性不够丰富
* 现有产品不够成熟
* 事务支持不够好

# MongoDB 数据库 #
C++语言编写, 高性能 易部署 易使用
* 面向集合存储, json
* 模式自由
* 支持动态查询
* 支持完全索引
* 支持复制和故障恢复
* 使用高效的二进制数据存储, 包括大型对象
* 文件存储格式为 BSON

关系型数据库和 mongodb 对应情况

| 关系数据库 | mongodb |
|------------------|
| 数据库     | 数据库 |
| 表    | 集合 |
| 行    | 文档 |

文档(document) 是 Mongodb 的最基本对象, 都有一个 objectId, `{"id": ObjectId()}`

集合(collection)是文档的聚合

0 和 1 也可以表示 true 或 false

数字类型64位浮点数, 整数都会被转换成64位浮点数

## 安装 ##
~~~~~~
mongod --dbpath=/var/any
mongo
~~~~~~

~~~~~~
## 创建或者切换
use mydb1;
## 查看当前数据库
db;
## 删除数据库
db.dropDatabase();

## 查看所有数据库信息
show dbs;
~~~~~~

## 建表 ##
~~~~~~
## 查看集合
show collections;
show tables;


## 创建集合
db.createCollections("erdan");
## 也可以这样创建集合, 隐式创建
db.collectionName.insert({});

## 删除集合
db.collectionName.drop();

## 插入文档
db.erhuo.insert({name:"xxx", age:18, city:"shanghai"});

## 删除文档
db.erhuo.remove({name:"xxx", age:18, city:"shanghai"});

## 查询
db.erhuo.find();

for(var i=1;i<=10000;1++) {
   db.drhuo.insert({name: "erhuo"+i, age: i, city: "tokyo"});
}

## 显示name字段
db.erhuo.findOne({}, name:true);
## 除了不显示name, 其他都显示
db.erhuo.findOne({}, {name: 0});

## 删除名字为 erhuo1
db.erhuo.remove({name: "erhuo1"});

var m = db.erhuo.find();
m.next();

## 条件查找
## 大于
db.collection.find({field:{$gt:value}});
## 小于
db.collection.find({field:{$lt:value}});
## 大于等于
db.collection.find({field:{$gte:value}});
## 小于等于
db.collection.find({field:{$lte:value}});
## 不等于
db.collection.find({field:{$ne:value}});

## 统计
db.erhuo.count();
db.erhuo.find().count();
## 按age升序排序, -1 为降序
db.erhuo.find().sort({age:1});

## 分页
db.erhuo.find().skip(3).limit(4);

## 不管分页, 列出所有数据的大小
db.erhuo.find().skip(3).limit(4).count(0);
## 列出分页数据
db.erhuo.find().skip(3).limit(4).count(1);

db.c2.insert({name:"user1", post:[1,2,3]});
## 查询包含关系
## 全部满足
db.c2.find({post:{$all:[1, 2]}})
## 只要有一个满足
db.c2.find({post:{$in:[1, 2]}})
## notin
db.c2.find({post:{$nin:[1, 2]}})

## 常用操作
db.customer.find({$or:[{name: "a"}, {age: 11}]})

## 1 表示存在, 0 表示不存在, 存在name字段的文档
db.customer.find({name: {$exist: 1}});
~~~~~~

## 更新 ##

`db.collection.update(criteria, objNew, upsert, multi)`

* `criteria`, 查询条件的对象
* `objNew`, 根系内容的对象
* `upsert`, 如果记录已经更新, 更新它, 否则新增一个记录
* `multi`, 如果符合多个记录是否全部更新

~~~~~~
## 覆盖更新
db.itcast.update({address:"abc"}, {address: "bj"});

db.itcast.update({name:"user1"}, {$set:{address: "bj"}}, 0, 1)

## age + 1
db.itcast.update({name:"user1"}, {$inc:{age: 1}}, 0, 1)

## 删除键
db.itcast.update({name:"user1"}, {$unset:{address: 1}}, 0, 1)
~~~~~~

# 索引 #

用来加速查询速度的

~~~~~~
## 查询性能
db.erhuo.find({name: "erhuo12"}).explain();

## 给 name 创建普通索引
db.erhuo.ensureIndex({name:1});

## 查看系统表相关参数
db.user.state();

db.erhuo.find({name: "erhuo12"}).explain();

## 查看生成的索引
db.system.indexes.find();

## 删除索引
## 如果删除集合, 那么也会删除索引
db.user.dropIndex({name:1});

## 创建唯一索引, 相当于唯一约束
db.user.ensureIndex({name:1}, {unique:1});
~~~~~~

## 固定集合 ##

事先创建而且大小固定的集合, 如果空间不足最早的文旦会被删除,
适用于任何想要自动淘汰过期的属性的场景, 如日志

~~~~~~
## size指定文档大小 单位KB, max 指文档数量
db.createCollection("collectionName", {capped: true, size:10000, max: 100});
~~~~~~



# 数据库备份和恢复 #

~~~~~~
## 备份数据
mongodump -h dbhose -d dbname -o dbdirectory

## 恢复数据
mongorestore -h localhost -d test dbdirectory
~~~~~~

# 导入和导出 #
~~~~~~
mongoexport -h dbhost -d dbname -c collectioName -o output

mongoimport -h dbhost -d dbname -c collectioName output

~~~~~~

# 安全和认证 #

~~~~~~
use admin;
## 创建超级管理员
db.addUser("root", "root");

use test;
db.addUser("zhangshan", "123");
## 创建只读用户
db.addUser("lisi", "123", true);

## 运行服务, 并且开启安全检查
mongod --dbpath --auth

## 登陆, 验证用户
db.auth("zhangshan", "123");
~~~~~~

# 配置集群 副本集 #
实时更新 复制 备份

## 主从复制 ##
主从复制是MongoDB最常用的复制方式。这种方式非常灵活，可用于备份、故障恢复、读
扩展等。

~~~~~~
## 创建主节点
mongod --dbpath=/tmp/master --port 10000 --master
## 创建从节点
mongod --dbpath=/tmp/slave --port 10001 --slave --source localhost:10000
~~~~~~
启动成功后就可以连接主节点进行操作了，而这些操作会同步到从节点.

## 副本集 ##

主从集群和副本集最大的区别就是副本集没有固定的“主节点”； 
整个集群会选出一个“主节点”，当其挂掉后，又在剩下的从节点 
中选中其他节点为“主节点”，副本集总有一个活跃点(primary)
和一个或多个备份节点(secondary)。
~~~~~~
## 启动节点一, 默认为主节点
mongod --dbpath E:\mogodb\dbs\node1 --logpath E:\mogodb\logs\node1\logs.txt --logappend
--port 10001 --replSet itcast/localhost:10002,localhost:10003 --master

## 启动节点二
mongod --dbpath E:\mogodb\dbs\node2 --logpath E:\mogodb\logs\node2\logs.txt --logappend
--port 10002 --replSet itcast/localhost:10001

## 启动节点三
mongod --dbpath E:\mogodb\dbs\node3 --logpath E:\mogodb\logs\node3\logs.txt --logappend
--port 10003 --replSet itcast/localhost:10001,localhost:10002

## 连上 节点一
mongo localhost:10001/admin

## 初始化副本集
db.runCommand({"replSetInitiate":{"_id":"itcast","members":[
   {"_id":1,"host":"localhost:10001","priority":3},
   {"_id":2,"host":"localhost:10002","priority":2},
   {"_id":3,"host":"localhost:10003","priority":1}]}});

## 查询是否主库
db.$cmd.findOne ( {ismaster: 1 } );
~~~~~~

## 分片 ##
分片(sharding)是指将数据拆分，将其分散存在不同的机器上的过程。有时也用分区
(partitioning)来表示这个概念。将数据分散到不同的机器上，不需要功能强大的大型计算机
就可以储存更多的数据，处理更多的负载。 

分片之前要运行一个路由进程，该进程名为 mongos。这个路由器知道所有数据
的存放位置，所以应用可以连接它来正常发送请求。对应用来说，它仅知道连接了一个普通
的 mongod。路由器知道数据和片的对应关系，能够转发请求到正确的片上。如果请求有了
回应，路由器将其收集起来回送给应用。

config 指明数据存放规则

~~~~~~
## 启动 config 配置服务
mongod --dbpath E:\mogodb\sharding\config_node --port 2222

## 启动分片服务, 连接config服务
mongos --port 3333 --configdb=127.0.0.1:2222

## 启动分片一 和 二
mongod --dbpath E:\mogodb\sharding\mongod_node1 --port 4444
mongod --dbpath E:\mogodb\sharding\mongod_node2 --port 5555

## 连接分片服务器, 并添加分片一 和 分片二
mongo localhost:3333/admin
db.runCommand({"addshard":"localhost:4444","allowLocal":true});
db.runCommand({"addshard":"localhost:5555","allowLocal":true});

## 开启数据库 test 的分片服务
db.runCommand({"enablesharding":"test"});

## 指定分片的片键
## 对test.person表进行分片, 根据 name 键, 进行分片
db.runCommand({"shardcollection":"test.person","key":{name:1}});

## 查看分片情况
db.printShardingStatus();
~~~~~~

# Java 操作 MongoDB #

导入 `mongo.jar`

查找
~~~~~~
Mongo mongo = new Mongo("localost", 27017);
DB db = mongo.getDB("test");

// 获得集合
DBCollection collection = db.getCollection("customers");
// 获得结果集
DBCursor dbCursor = collection.find();
while(dbCursor.hasNext()) {
  DBObject dbObject = dbCursor.next();
  Object name = dbObject.get("name");
  Object age = dbObject.get("age");
}
// 关闭资源
mongo.close();
~~~~~~

增加
~~~~~~
Mongo mongo = new Mongo("localost", 27017);
DB db = mongo.getDB("test");

// 获得集合
DBCollection collection = db.getCollection("customers");

DBObject db0 = new BasicDBObject();
db0.put("name", "leo");
db0.put("age", test);
collection.insert(db0);

mongo.close();
~~~~~~

更新
~~~~~~
Mongo mongo = new Mongo("localhost", 5555);

DB db = mongo.getDB("test");

DBCollection personCollection = db.getCollection("person");

DBObject o= new BasicDBObject();
o.put("name", "erdan");
o.put("age", 30);

//更改的时候 ,如果以”_id去更改”,那么同样的要以传递ObjectId,
// 不能直接将id值以字符串的形式传过去
DBObject q = new BasicDBObject("_id", new ObjectId("525f495bf422c198b9a69bbc"));

// update方法重载了很多次,这里以这个为例.
WriteResult result = personCollection.update(q , o);

mongo.close();
~~~~~~
