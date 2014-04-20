title: 传智播客day06-xml解析
date: 2014-04-06 08:47:01
tags:
- 传智播客
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
Node node = nodelist.item(0);

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
// 设置属性
elm.setAttribute( "出版社" ,  "黑马" );

//5. 回写
val transFac = TransformerFactory.newInstance()
val trans = transFac.newTransformer()
trans.transform(new DOMSource(doc), new StreamResult("filepath"))
~~~~~~

## SAX ##


~~~~~~
val saxFac = SAXParserFactory.newInstance()
val saxPaser = saxFac.newSAXParser()
val xmlReader = saxPaser.getXMLReader()
val handler = new DefaultHandler {
    override def characters(ch:Array[Char], start:Int, length:Int):Unit = println(ch)
    override def startDocument() = println("start doc")
    override def endDocument() = println("end doc")
    override def startElement(uri:String, localName:String, qname:String, attr:Attributes) = println(	"start attr")
    override def endElement(uri:String, localName:String, qname:String) = println("end elm")
}
xmlReader.setContentHandler(handler)
xmlReader.parse("book.xml")
~~~~~~
~~~~~~
public class  MySAX3_exercise {
//把书中的数据封装到JavaBeam中.
     public   static   void  main(String[] args)  throws  Exception {
         //得到解析器SAXSparer
        SAXParser parser = SAXParserFactory. newInstance ().newSAXParser();
         //得到读取器
        XMLReader reader = parser.getXMLReader();
         //定义一个集合,存储JavaBeam
        List<Book> books =  new  ArrayList() ;
         //给读取器注册事件处理器,使用
        reader.setContentHandler( new  MyContentHandler2(books));
    }
}
//创建注册事件注册器,把读到的数据存入到JavaBeam中.
class  MyContentHandler2  extends  DefaultHandler{
     //定义一个集合容器,引用主函数中的集合
     private  List<Book>  books ;
     public  MyContentHandler2(List<Book> books){
         this . books =books;
    }
     //定义一个容器类
     private  Book  book ;
     //定义一个容器字符串
     private  String  currentTagName ;
     public   void  startElement(String uri, String localName,
            String qName, Attributes attributes)  throws  SAXException {
         //如果读到的是书，创建book对象
         if ( "书" .equals(qName)){
             book  =  new  Book();
        }
         currentTagName  = qName;
    }
     public   void  characters( char [] ch,  int  start,  int  length)
             throws  SAXException {
         if ( currentTagName .equals( "书名" ))
         book .setName( new  String(ch,start,length));
         if ( currentTagName .equals( "作者" ))
         book .setAuthor( new  String(ch,start,length));
         if ( currentTagName .equals( "售价" ))
         book .setPrice( new  String(ch,start,length));
        }
     public   void  endElement(String uri, String localName, String qName)
             throws  SAXException {
         //如果读到的是书，把book对象加到集合中去
         if ( "书" .equals(qName)){
             books .add( book );
        }
         currentTagName  =  null ;
    } }

~~~~~~

## DOM4j ##

结合了SAX和jaxp

核心是Element，jaxp核心是Node

~~~~~~
// 得到解析器
SAXReader reader =  new  SAXReader();
// 得到Document,
Document document = reader.read( "src/book.xml" );
// 1.得到根元素
Element root = document.getRootElement();
// 2.得到根元素书里面所有(书)元素的集合(只得到儿子,不得到孙子),注意不是get方法,是element.
List<Element> books =  root.elements( "书" ) ;
// 3.得到第二本书
Element secondBook = books.get(1);
// 4.得到书中的(作者)元素
Element author = secondBook.element( "作者" );
// 修改主体内容
Element price = secondBook.element( "售价" );
price.setText( "1.00元" );

// 1.创建元素DocumentHelper,这个类可以创建好多东西;
Element e = DocumentHelper. createElement ( "内部价" ).addText( "48.00元" );
// 2.添加元素到文档中,有父亲才可以添加.
document.getRootElement().element( "书" ).add(e);

// 删除第三个元素
books.remove(2);

// 设置属性
Element book2 = (Element)root.elements( "书" ).get(1);
book2.addAttribute( "出版社" ,  "传智" );

for  ( int  i = 0, size = element.nodeCount(); i < size; i++) {
    Node node = element.node(i);
    if  (node  instanceof  Element) {
        test2 ((Element) node);
    }  else  {
        // do something....
    }
}

// 保存
OutputFormat format = OutputFormat. createCompactFormat (); // 不好看的,机器看的
format.setEncoding( "UTF-8" ); // 设置编码

XMLWriter writer =  new  XMLWriter( new  FileOutputStream( "src/book.xml" ), format);
writer.write(document);
writer.close();
~~~~~~
