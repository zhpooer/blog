title: scala和Dom4j的xml操作
date: 2014-04-17 07:29:49
tags:
- xpath
- scala
- dom4j
---

# dom4j的xpath接口 #
[dom4j具体操作, 猛击](/2014/04/06/传智播客day06-xml解析)

* `document.selectNodes("")` 查找多个匹配的节点

* `document.selectSingleNode("")` 查找只有一个匹配的节点, 如果没有返回null

* `node.valueOf("@name");` 获取节点中name属性的值



|   xpath              |   意义       |
|----------------------------------------------|
|`//BBB`               |  选择所有的BBB|
|`//DDD/BBB`           |  选择所有DDD下面的BBB|
|`/AAA/CCC/DDD/*`      |  选择/AAA/CCC/DDD 下面所有的标签|
|`/*/*/*/DDD`          |  选择第四个层次的DDD|
|`//* 所有的标签`     |
|`/AAA/BBB[1]`        |   选择第1个节点|
|`/AAA/BBB[last()]`   |
|`//@id`               |  选择所有的id属性 |
|`//BBB[@name]`        |  选择所有有name属性的BBB节点 |
|`//BBB[@*]`           |  选择所有有属性的BBB节点|
|`//BBB[not(@*)]`      |  选择所有没有属性的BBB节点|
|`//[@id='b1']`        |  选择所有属性id=bi的属性|
|`//*[count(BBB)=2]`   |  选择有两个BBB子节点的节点|
|`//*[count(*)=2]`     |  选择由两个子节点的节点 |
|`//*[contains(name(), "AAA")]` |   |
|`//*[string-length(name(),3)]` |   |
|`/AAA/EEE  //DDD/CCC`  |  集合起来 |

## 读入xml ##
~~~~~~
SAXReader reader = new RAXReader();
Document document = reader.read("*.xml");
~~~~~~

## dom4j 输出 ##
~~~~~~
org.dom4j.io.OutputFormat format = OutputFormat.createCompactFormat();
format.setEncoding("UTF-8");
XMLWriter writer = new XMLWriter(new FileOutputStream(), format);
writer.write(document);
writer.close();
~~~~~~

## xpath实际案例 ##
~~~~~~
<books>
    <book>
        <author>
        </author>
    </book>
</books>
~~~~~~

~~~~~~
Node node = document.selectSingleNode("//book[2]/author"); // 会选中其中一个
node.getText();

Node node = document.selectSingleNode("//book[2]");
node.valueOf("@id");  // 获取属性ID
~~~~~~

# scala xml 操作 #

## 加载和导出xml ##
~~~~~~
val bookstore =
      <bookstore>
        <book>
          <title lang="eng">Harry Potter</title>
          <price>29.99</price>
        </book>
        <book>
          <title lang="eng">Learning XML</title>
          <price>39.95</price>
        </book>
      </bookstore>
XML.save("bookstore1.xml", bookstore, "utf-8", true, null) // 第三个参数明是否要写xml头
// 加载
XML.loadFile("bookstore.xml")
~~~~~~

## 查询 ##
由于`scala.xml.Node` 继承于 Seq[Node], 所以可以用 scala 的 `Seq` 中的方法来操作xml,

[详细继承图,及案例](http://www.codecommit.com/blog/scala/working-with-scalas-xml-support)
~~~~~~
val bookstore = XML.loadFile("bookstore.xml")
val price =
  bookstore match {
    case <bookstore>{ books @ _* }</bookstore> =>
      books.collectFirst {
        case book @ <book>{ _* }</book> if (book \ "title").text == "Learning XML" =>
          (book \ "price").text
        }
    }
~~~~~~

## CRUD ##
注: 还有另一种更新方法, 感觉不是很直观, 有兴趣的可以[猛击我](http://stackoverflow.com/questions/970675/scala-modifying-nested-elements-in-xml)
~~~~~~
  it should "append xml" in {
    val bookstore = XML.loadFile("bookstore.xml")
    val book =
      <book>
        <title lang="zh">Learn Scala</title>
        <price>40.11</price>
      </book>
    bookstore \ "book" ++ book
  }

  it should "remove child" in {
    val bookstore = XML.loadFile("bookstore.xml")
    val afterDel = bookstore.child.filter(node => (node \ "price").text == "29.99")
  }

  it should "update xml" in {
    val bookstore = XML.loadFile("bookstore.xml")
    val updated =
      (bookstore \ "book") map {
        _ match {
          case book @ <book> { _* }</book> if (book \ "title").text == "Learning XML" =>
            <book><title>{ (book \ "title").text }</title><price>3</price></book>
          case x => x
        }
      }
  }
~~~~~~
