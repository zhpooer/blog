title: 传智播客day05
date: 2014-04-04 09:20:29
tags:
- 课堂笔记
- xml
- dtd
- schema
---

# xml语言 #
* 作用：传输数据 存储数据

## xml结构 ##
* xml1.1 发一次请求，多次响应
* xml1.0 发一次请求，一次响应
~~~~~~xml:
<?xml version="1.0" encoding="utf-8"?> <!--声明-->

<books>  <!--必须要有根元素-->
    <book id="first"> cnontent </book>
</books>
~~~~~~
** 注意： 大小写敏感 **

## dtd xml约束 ##
* DOCTYPE: 内部声明
* ELEMENT: 元素声明

~~~~~~dtd:
<?xml version="1.0" encoding="gbk"?>
<!ELEMENT books (book?)>  <!-- + * ?: 出现一次或多次-->
<!ELEMENT book (name, price)>
<!ELEMENT name (#PCDATA)>
<!ELEMENT price (#PCDATA)>
<!ATTLIST book bid CDATA #REQUIRED >
~~~~~~

~~~~~~xml:
<?xml version="1.0" encoding="gbk"?>
<!DOCTYPE books [
   <!ELEMENT books(book)>
   <!ELEMENT book(name, price, author|publish)> <!-- 必须按顺序写 -->
   <!ELEMENT name(#PCDATA)>
   <!ELEMENT price(#PCDATA)>
]> <!-- 声明在头部 or -->

<!DOCTYPE books SYSTEM "**path.dtd**"> <!-- 从本地引入dtd文件 -->

<!DOCTYPE books PUBLIC "//UNKNOWN/" "unknown.dtd"> <!-- 从互联网引入 -->
~~~~~~
### 类型 ###
* CDATA: 字符数据
* ID: id

### 约束 ###
* \#REQUIRED
* \#FIXED value: 固定
* \#IMPLIED: 不是必须

### CDATA ###
> `<![CDATA[will be not parsed content]]>`


### xml css ###

> `<?xml-stylesheet type="text/css" href="book.css"?>`


### 实体 entity ###

> `<!ENTITY site "www.itcast.cn">`


> `<publish>&my;</publish> <!-- 引用 --> `

### 混合型 ###

> `<!ELEMENT note (#PCDATA|to|from|header|message)*>`

## schema ##

Schema是一个xml文件扩展为xsd，替代dtd

~~~~~~
<schema xmlns=""
        targetNamespace="*any*"i
        xml ns:tns="*any*"
        elementForDefault="qualified">
    <element name="books">
        <complexType>
            <sequence>
                <element name="book" minOcdcurs="2" maxOccurs="3">
                    <complexType>
                        <sequence>
                            <element name="name" type="string"> </element>
                            <element name="price" type="tns:myprice"> </element> <!--注意：自定义类型使用 -->
                            <element name="publishtime" type="date"> </element>
                            <element name="tel" type="tns:mytel"> </element>
                        </sequence>
                    </complexType>
                </element>
            </sequence>
            <attribute name="bid" type="int" use="option">
            </attribute>
        </complexType>
    </element>
    <!-- 自定义元素 -->
    <simpleType name="mytel">
        <restriction base="striing"> <!-- 指定元素类型 进行加强 -->
            <parttern value="\d{3}-\d{4}-\d{4}">
            </parttern>
        </restriction>
    </simpleType>
    <simpleType name="myprice">
        <restriction base="double">
            <maxInclusive value="200"> </maxInclusive>
            <minInclusive value="100"> </minInclusive>
        </restriction>
    </simpleType>
</schema>

~~~~~~

~~~~~~
<books xmlns="" xmlns:xsi="" xsi:schemaLocation="">
</books>
~~~~~~
