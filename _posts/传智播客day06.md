title: 传智播客day06
date: 2014-04-06 08:47:01
tags:
- xml
- DOM
- SAX
---
# xml解析 #

1. dom  
文档放入都内存中，方便CRUD
2. sax  
逐行解析，方便查询，速度快，但不能CRUD

## DOM, jaxp ##
解析包：javax.xml.parsers
1. 获取解析器的工厂对象  
`DocumentBuilderFactory fac = DocumentBuilderFactory.newInstance();`
2. 获取文档解析对象  
`DocumentBuilder docBuilder = factory.newDocumentBuilder;`
3. 加载xml文件  
`docBuilder.parse("xml")`
4. 获取Document对象
~~~~
import org.w3c.dom.Document;
Document document = documentBuilder.newDocument();
~~~~
5. 操作  
节点Node： 文本是一个节点，属性也是一个节点
~~~~~~
NodeList nodeList = document.getElementByTagName("book");
Node node = nodelist.item();

~~~~~~
### Node继承结构 ###
Node是中心  
`nodelist.getNodeType()`  
继承Node： Text，Attr，Comment，Document, Element...  
Text: 空白行也算  
`fac.setIgnoringElementContentWhitespace(true)// 忽略回车换行`,只对有约束的文档起作用

### 操作元素
~~~~~~
//1.创建元素
val elm  = doc.createElement("div")
//2.创建文本
val txtElm = doc.createTextNode("hello dom")
//3. 添加
elm.appendChild(txtElm)
//4. 删除
txtElm.getParentNode().removeChild(txtElm);
//5. 回写
val transFac = TransformerFactory.newInstance()
val trans = transFac.newTransformer()
trans.transform(new DOMSource(doc), new StreamResult("filepath"))
~~~~~~

## SAX ##


~~~~~~
val saxFac = SAXParserFactory.newInstance()
val saxPaser = saxFac.newSAXParser()
val handler = new DefaultHandler {
    override def characters(ch:Array[Char], start:Int, length:Int):Unit = println(ch)
    override def startDocument() = println("start doc")
    override def endDocument() = println("end doc")

    override def startElement(uri:String, localName:String, qname:String, attr:Attributes) = println(	"start attr")
    override def endElement(uri:String, localName:String, qname:String) = println("end elm")
}
saxPaser.parse("filepath.xml", handler)
~~~~~~

## DOM4j ##

结合了SAX和japx  
核心是Element，japx核心是Node
