title: 传智播客-solr
date: 2014-08-28 14:00:50
tags:
- solr
- 传智播客
---

# compass #

数据库,以服务的形式对外公开, 代表跟数据进行交互走网络,
使其他的语言客户端都能调用

Compass 对 Lucence 进行封装, 以面向对象的方式操作 Lucence

# solr #

solr 当做搜索引擎服务器封装 LucenceDao, 底层是对索引库的操作,
对外提供 Web-service API 接口, 通过 http 协议进行调用.

通过各种 API 可以让你的应用使用搜索服务.也支持主从, 热换库的操作.

通过 `localhost:8983/solr` 索引进行 CRUD 操作

`post.jar`包可以与solr服务器进行交互
~~~~~~
# 提交索引 hd.xml
# localhost:8983/solr/collection1/select?q=高端&wt=json 查询
# localhost:8983/solr/collection1/select?q=高端&wt=json&fl=title 查询只返回title字段
# sort=id 按id排序
java -jar post.jar hd.xml

# 删除 id 为 1 的索引
java -Ddata=args -Dcommit=false -jar post.jar "<delete><id>1</id></delete>"
# 提交
java -jar post.jar -
~~~~~~

# solrj #

通过 solrj 向 solr 交互数据

~~~~~~
public void testInsert(){
  // 连接 solr 服务器
  SolrServer server = new HttpSolrServer("http://localhost:8983/solr");
  
  SolrInputDocument document = new SolrInputDocument();
  List<SolrInputDocument> collection = new ArrayList();
  
  document.addField("id", 1);
  document.addField("title", "csdn");
  document.addField("cctv", "bjtx");
  
  
  collection.add(document);
  
  server.add
  
  server.add(collection);
  server.commit();
}

public void textUpdate(){
  // 连接 solr 服务器
  SolrServer server = new HttpSolrServer("http://localhost:8983/solr");
  
  SolrInputDocument document = new SolrInputDocument();
  List<SolrInputDocument> collection = new ArrayList();
  
  document.addField("id", 1);
  document.addField("title", "更新值");
  
  collection.add(document);
  
  server.add
  
  server.add(collection);
  server.commit();
}

public void testDelete(){

}

public void testQuery(){
  // 连接 solr 服务器
  SolrServer server = new HttpSolrServer("http://localhost:8983/solr");
  SolrQuery param = new SolrQuery();
  params.setQuery(":*");
  params.addSort("id", ORDER.desc);
  QueryResponse response = server.query(param);
  SorlDocumentList list = response.getResult();
  for(SolrDocument document: List) {
    // 出来的是数组
    String title = document.get("title").toString();
  }
}

// 高亮
public void testHightLight(){
  // 连接 solr 服务器
  SolrServer server = new HttpSolrServer("http://localhost:8983/solr");
  SolrQuery param = new SolrQuery();
  params.setQuery("title:传智播客");
  // 开启高亮
  param.setHightLight(true);
  params.setHightLightSimplePre("<font color='red'>");
  params.setHightLightSimplePost("</font>");
  // 添加需要高亮的字段
  params.setParam("hl.fl", "title");

  params.addSort("id", ORDER.desc);
  QueryResponse response = server.query(param);
  SorlDocumentList list = response.getResult();
  //第一个map的 key 是document的id
  //第二个map的 key 是高亮的字段
  // Map<String, Map<String, List<String>>>
  for(SolrDocument document: List) {
    Map<String, List<String>> listmap = maplist.get(id);
    List<String> listhight = listmap.get("title");
    for(String s:listhight) {
      System.out.println(s);
    }
  }
}
~~~~~~

`schema.xml` 配置字段
~~~~~~
<!-- 通过 text进行 多个字段的搜索 -->
<copyField source="cat" dest="text"/>
<copyField source="dog" dest="text"/>
~~~~~~

`solrconfig` 配置solr缓存jar包
* 配置集群
* 索引库的存放位置
* 自定义分词器
* 请求处理器, 如将数据库导入到索引库的请求处理器

`data-config.xml` 配置数据库连接

TODO 配置文件说明
