title: 传智播客-Lucence 入门
date: 2014-08-26 20:49:37
tags:
- lucence
- 传智播客
---

# 全文检索 #

全文检索是计算机程序通过扫描文章中的每一个词，对每一个词建立一个索引，
指明该词在文章中出现的次数和位置。当用户查询时根据建立的索引查找，
类似于通过字典的检索字表查字的过程


# Lucence 建立索引 #

~~~~~~
// 索引存放的目录
Directory directory = FSDirectory.open("indexDir/");
// 分词器
Analyzer analyzer = new StandardAnalyzer(Version.LUCENCE_44);
// 索引写入配置, lucence 使用版本
IndexWriterConfig config = new IndexWriterConfig(Version.LUCENCE_44, analyzer);

IndexWriter indexWriter = new IndexWriter(directory, config);

// lucence索引当中存储的都是 document

Document document = new Document();

IndexableField intfield = new IntField("id", 1, Store.YES);
IndexableField stringfield = new StringField("title", "客机失航", Store.YES);
IndexableField textfield = new TextField("content", "中国提供", Store.YES);
document.add(intfield);
document.add(stringfield);
document.add(textfield);

indexWriter.addDocument(document);
indexWriter.close();
~~~~~~

索引索引库中的数据

~~~~~~
// 索引在硬盘上存放的目录
Directory directory =  FSDirectory.open(new File("indexDIR/"));
// 索引读取器
IndexReader indexReader = DirectoryReader.open(directory);

// 构造索引搜索对象
IndexSearchr indexSearcher = new IndexSearcher(indexReader);

// query 它是一个查询条件
Query query = new TermQuery(new Term("content", "中"));

// 搜索符合 query 条件的N条记录
TopDocs topDocs = indexSearcher.search(query, 10);

ScoreDoc scoreDocs [] = topDocs.scoreDocs;
for(Score scoreDoc : scoreDocs) {
  // 返回 document的 id, 根据id找到对应的document
  Document document = indexSearcher.doc(scoreDoc.doc);
  document.get("content");
  document.get("title");
  document.get("id");
}
~~~~~~

过程解析:
1. IndexWriter 使用 分词器(Analyzer)创建索引(StringField 不会被分词, TextField 会被分词)
2. 通过 IndexSearch 搜索
  1. 通过 `IndexSearch.search(query, 10)` 检索索引目录
  2. 根据检索到的id, 检索源数据

## Lucence CRUD ##

~~~~~~
public class LucenceUtils {
   private static Directory directory;
   
   private static IndexWriterConfig config = null;

   private static Version version;
   // 构造分词器, 是一个抽象类, 
   private static Analyzer analyzer;
   
   static {
       try{
           directory = FSDirectory.open(new File("newsIndexDIR/"));
           version = version.LUCENCE_44;
           analyzer = new StandardAnalyzer(version);
           confi = new IndexWriterConfig(analyzer, version);
       } catch (){
       }
   }
   
   // 返回用于操作索引的对象
   public IndexWriter getIndexWriter() {
     IndexWrider indexWriter = new IndexWriter(directory, config);
     return indexWriter;
   }
   
   // 返回用于搜索索引的对象
   public static IndexSearch getIndexSearch() {
       IndexReader indexReader = DirectoryReader.open(directory);
       IndexSearch indexSearch = new IndexSearch(indexReader);
       return indexReader;
   }

   public static getCurrentVersion(){
       return version;
   }
   public static getAnalazer() {
       return analyzer;
   }
}

public class Article {
    public ind id;
    public String title;
    public String author;
    public String url;
}

public class AticleToDocument {
    public static Document articleToDocument(Article article){
        Document document = new Document();
        IntField idfield = new IntField("id", article.getId(), Store.YES);
        TextField titlefield = new TextField("title", article.getContent(), Store.YES);
        StringField authorfield = new StringField("author", article.getAuthor(), Store.YES);
        StringField urlField = new StringField("url", article.getUrl(), Store.YES);
        document.add(idfield);
        document.add(titlefield);
        document.add(authorfield);
        document.add(urlField);
        return document;
    }
    public static Article documentToArticle(Document document){
        Article article = new Article();
        article.setId(Integer.parseInt(document.get("id")));
        article.setTitle(document.get("title"));
        article.setContent(document.get("content"));
        article.setAuthor(document.get("author"));
        article.setUrl(document.get("url"));
        return article;
        
    }
}
public class LucenceDao {
   // 将 Article 添加到 索引库中
   public void addIndex(Article article){
       IndexWriter indexWriter = LucenceUtils.getIndexWriter();
       Document document = articleToDocument.articleToDocument(article);
       indexWriter.addDocument(document);
       indexWriter.close();
   }
   public void deleteIndex(String field, String value){
       IndexWriter indexWriter = LucenceUtils.getIndexWriter();
       // delete from document where title = ?
       indexWriter.deleteDocuments(new Term(field, value));
       indexWriter.close();
   }
   public void updateIndex(String field, String value, Article article) {
       IndexWriter indexWriter = LucenceUtils.getIndexWriter();
       Document doc = ArticleToDocument.articleToDocument(article);
       indexWriter.updateDocuments(new Term(field, value), doc);
       indexWriter.close();
   }
   
   // 通过关键字进行搜索, 进行分页
   public List<Article> queryIndex(String keywords, int firstResult, int maxResult){
       IndexSearch indexSearch = LucenceUtils.getIndexSearch();
       String fields[] = {"title", "content", "author", "url"};
       // 根据多个字段进行查询
       QueryParser parser = new MultiFieldQeuryParser(LucenceUtils.getCurrentVersion(), fields, LucenceUtils.getAnalazer());
       // 在查询时也会进行分词, 但 TermQuery 不会
       Query query = parser.parse(keywords);

       TopDocs topdocs = indexSearch.search(query, firstResult+maxResult);
       ScoreDoc scoreDocs[] = topdocs.scoredocs;
       int endResult = Math.min(firstResult+maxResult, scoreDocs.length);
       for(int i=firstResult;i<endResult; i++) {
           Document document = indexSearch.doc(scoreDocs[i], doc);
           article.add(ArticleToDocument.documentToArticle(Document));
       }
       return article;
   }
}
~~~~~~

# 数据库查询和全文检索对比 #

性能
* 数据库性能: `select * from document where title %ant%` 进行全表扫描, 性能差
* lucence: 全文检索通过扫描文章中的每一个词, 对此建立索引. 搜索的时候根据索引找到对应的数据

准确度
* 数据库不准确
* lucence: 准确非常高

lucence 具有相关度排序




# 原理分析 #

创建索引: 使用 indexWriter 对象
* Lucence 中存储的都是 document 对象
* 在创建索引的时候要用到分词器, 对文本当中的词进行提取, 然后建立索引
* Analyzer 是一个抽象类, 构造不同的子类, 相当于不同的分词规则
* 可以在 document 中添加字段
  * IndexableField 是一个接口, 要存储不同类型的数据, 构造不同的实现
  * StringField 和 TextField 都可以存储string 类型的数据, StringField 字段对应的值
  在索引库中不分词, textField 对应的数据会被分词

搜索索引, 使用 indexSearch 进行
* Query 是一个查询条件, 是一个抽象类, 构造不同的子类, 相当于不同的查询规则
* 根据 Query 查询索引, 获得 Document 的id


# 分词器 #

对文本进行分词, 创建索引, Analyzer是一个抽象类, 可以改造分词器来定义自己的分词规则

Analyzer（分词器）的作用是把一段文本中的词按规则取出所包含的所有词。
对应的是Analyzer类，这是一个抽象类，
切分词的具体规则是由子类实现的，
所以对于不同的语言（规则），
要用不同的分词器.

工作流程
1. 切分关键词
2. 去掉停用词, 有些词在文中出现的频率非常高, 但是对文本所携带的信息基本不产生影响.
3. 对于英文单词, 把所有字母转为小写

常用中文分词
* 单字分词(StandardAnalyzer, ChineseAnalyzer)
* 二分法分词(CJKAnayzer), 按两个字进行切分,
如：“我们是中国人”，效果：“我们”、“们是”、“是. "中国"
* 词库分词("极易分词”MMAnalyzer，或者是“庖丁分词”分词器、IKAnalyze, 按某种算法构造词，
然后去匹配已建好的词库集合，如果匹配到就切分出来成为词语。通常词库分词被认为是最理想的中文分词算法。

~~~~~~
public void test(){
    // 单词分词
    Analyzer a = new StandardAnalyzer(Version.LUCENCE_44);
    // 二分法分词
    Analyzer a = new CJKAnalyzer(Version.LUCENCE_44);
    // 词库分词 自定义的词, 自定义停用词(庖丁分词器, 对中文支持比较好)
    // 拷贝配置文件, 定义扩展词典
    Analyzer a = new IKAnalyzer();
}
public static void testAnalyzer(Analyzer analyzer, String text) {
    TokenStream tokenStream = analyzer.tokenSteam("content", new StringReader(text));
    tokenStream.addAttribute(ChartermAttribute.class);
    tokenStream.reset();
    while(tokenStream.incrementToken()) {
        ChartermAttribute chartermAttribute = tokenStream.getAttribute(ChartermAttribute.class);
        System.out.println(new String(chartermAttribute.toString));
    }
    tokenStream.close();
}
~~~~~~

# 相关度排序 #

搜索出的每个 document 都有一个得分, 得分越高, 排列顺序越靠前

~~~~~~
IndexSearch indexSearch = LucenceUtils.getIndexSearch();
QueryParser queryParser =
    new MultiFieldQeuryParser(LucenceUtils.getCurrentVersion(),new String[]{"title", "conent"}, LucenceUtils.getAnalazer());
Query query = queryParser.parse("");

TopDocs topdocs = indexSearch.search(query, 100);
ScoreDoc scoreDoc[] = topDocs.scoreDocs;

for(Score scoreDoc : scoreDocs) {
  // 返回 document的 id, 根据id找到对应的document
  Document document = indexSearcher.doc(scoreDoc.doc);
  // 得分
  scoreDoc.score;
  document.get("content");
  document.get("title");
  document.get("id");
}
~~~~~~

可以通过修改文档的得分(人工干预), 来影响文档的排列顺序
~~~~~~
// 修改得分为原来得分的4倍, 设置权重值, 默认为一倍
textFi1eld.setBoot(4f);
~~~~~~

可以在搜索的时候对查询结果进行排序
~~~~~~
// 排序条件, 升序/降序true(反转)
SortField sortField = new SortField("id", Type.INT, true);

Sort sort = new Sort(sortField);

indexSearch.search(query, 100, sort)
~~~~~~

# 索引优化 #

lucence 3.6 之后的版本会自动优化, 亦可以手动优化

~~~~~~
Directory directory = FSDirectory.open("indexDir/");
IndexWriterConfig config = new IndexWriterConfig(Version.LUCENCE_44, LucenceUtils.getAnalazer());
// 合并策略
LogMergePolicy mergePolicy = new LogMergePolicy();
config.setMergePolicy(mergePolicy);
// 合并频率
// 如果值越小, 搜索越快, 创建索引越慢
// 值越大, 代表搜索越慢, 创建索引越快
// 小值 3-10, 大值 >= 10
confg.setMergeFactor(10);

IndexWriter indexWriter = new IndexWriter(directory, config);
~~~~~~

可以从内存中搜索

~~~~~~
// 索引在硬盘上存放的位置
directory directory1 = FSDirectory.open(new File("newIndexDIR"));
// 通过此对象将数据放入内存
IOContext ioContext = new IoContext();
Directory directory = new RAMDirectory(directory1, ioContext);
IndexReader indexReader = DirectoryReader.open(directory);
~~~~~~

索引优化的四种方式:
1. 通过修改 indexWriterConfig 对象设置 document 的合并规则
2. 将硬盘上的索引数据读取到内存中, 减少io操作, 
3. 通过创建索引的时候定义分词规则
4. 分区存放(新闻放一个文件夹, 图片一个文件夹)

# Lucence 高亮 #

在查询的时候, 对查询结果所包含的关键字进行高亮

~~~~~~
// 对查询出来的document, 对当中的关键字进行高亮

IndexSearch indexSearch = LucenceUtils.getIndexSearch();
String fields[] = {"title", "content"};
// 根据多个字段进行查询
QueryParser parser = new MultiFieldQeuryParser(LucenceUtils.getCurrentVersion(), fields, LucenceUtils.getAnalazer());
// 在查询时也会进行分词, 但 TermQuery 不会
Query query = parser.parse(keywords);

// 创建一个高亮器
// 高亮的格式: <font color='red'>中国</font> 达人秀
HighLight highlighter = new Highlighter("<font color='red'>", "</font>");

// 跟 query 条件进行关联
Score fragmentScorer = new QueryScorer(query);

TopDocs topdocs = indexSearch.search(query, 100);
ScoreDoc scoreDocs[] = topdocs.scoredocs;

for(int i=0;i < scoreDocs.length; i++) {
    Document document = indexSearch.doc(scoreDocs[i].doc);

    String title = document.get("title");
    if(title!=null) {
      // 返回高亮过后的文本
       hightlighterTitle =
          hightlighter.getBestFragment(LucenceUtils.getAnalazer(), "title", document.get("title"));
    }
    
   // 如果高亮过好的文本为null, 说明属性字段对应的值当中没有包含关键字
    if(hightlighterTitle==null) {
        document.get("title");
    } else {
        hightlighterTitle;
    }
}
~~~~~~

# Score.YES & Score.NO #

~~~~~~
// 根据关键字查找到记录, 但是返回具体值"some content"
IndexableField stringFiled =
    new StringField("title", "some content", Store.YES);

// 根据关键字查找到记录, 但是不会返回具体值, 分词, 但是不存储
IndexableField stringFiled1 =
    new StringField("title", "some content", Store.NO);
~~~~~~

# 过滤器 #

对搜索结果进行过滤, 获得更小的范围结果

~~~~~~
// 添加过滤条件,
// 是否包含最小值, 是否包含最大值
Filter filter = NumbericRangerFilter.newIntRange("id", 1, 9, true, false);
TopicDocs topdocs = indexSearch.search(query, filter, 100);
~~~~~~

# 查询 #

~~~~~~


public static void query(Query query) {
    IndexSearch indexSearch = LucenceUtils.getIndexSearch();
    TopDocs topdocs = indexSearch.search(query, 100);
}

// 单字段查询
Query query = new TermQuery(new Term("author", ""));
query(query);

// 多字段查询, 会被分词
QueryParser queryParser = new MultiFieldQeuryParser(
    LucenceUtils.getCurrentVersion(), new String[]{"title", "content"}, LucenceUtils.getAnalazer);
Query query = queryParser.parse("cotent");
query(query);

// 通配符查询, ? 代表单个字符, * 代表多个字符
Query query = new WildcardQuery(new Term("content", "ja*"));
query(query);

// 范围查询, 使用查询可以代替过滤器
Query query = NumbericRangerQuery.newIntRange("id", 1, 10, true, false)

// 查询所有
Query query = new MatchAllDocsQuery();

// 模糊查询
// 取值在 0-2 之间, 最大可编辑数, 运行查询条件错误几个字符
Query query = new FuzzyQuery(new Term("author", "爱新觉罗"), 1);

// 短语查询
PhraseQuery phraseQuery = new PhraseQuery();
// 添加短语, 需要在短语后面设置元素角标
phraseQuery.add(new Term("conent", "学"));
phraseQuery.add(new Term("conent", "扑"));
// 设置两个短语之间的最大间隔数
phraseQuery.setSlop(11111);

// 布尔查询
BooleanQuery booleanQuery = new BooleanQuery();
// 必须满足的条件
booleanQuery.add(fuzzyQuery, Occur.MUST);
// 必须不满足
booleanQuery.add(phraseQuery, Occur.MUST_NOT);


~~~~~~
