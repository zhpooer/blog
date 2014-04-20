title: 传智播客day14-自定义标签
date: 2014-04-17 09:04:00
tags:
- 传智播客
- 自定义标签
---

# 自定义标签 #

作用: 替换JSP页面中的java脚本 `<%%>`

且每次调用JSP都会生成新的标签对象
~~~~~~
<%
Date time = new Date();
out.write(time.toLocaleString());
%>
~~~~~~
## 开发步骤 ##

1. 编写一个类 `javax.sevlet.jsp.tagext.SimpleTag`,
覆盖掉 `doTag()` 方法   **ShowTimeSimpleTag.java**:
~~~~~~
public class ShowTimeSimpleTag extends SimpleTagSupport{
   public void doTag() throws JspException, IOExcetpion {
       Date time = new Date();
       PageContext pc = (PageContext) getJSPContext();
       pc.getOut().write(time.toLocaleString());
   }
}
~~~~~~

2. **itcast.tld**:
~~~~~~
<taglib>
    <tlib-version>1.0</tlib-version>
    <short-name>myfn</short-name>
    <!-- 定位符 -->
    <uri>http://www.itcast.cn/jsp/myfunction </uri>
    <tag>
        <name>showTime</name>
        <tag-class>cn.itcast.ShowTimeSimpleTag </tag-class>
        <body-content> empty </body-content>
    </tag>
</taglib>
~~~~~~
3. 通过Taglib指令导入外部标签库, 以及使用
~~~~~~
<%@ taglib uri="http://" prefix="itcast" %>
<itcast:showTime>
~~~~~~

## 实现简单功能 ##
1. 控制页面中某部分内容不显示
~~~~~~
<itcast:hidden>隐藏的内容</itcast:hidden>
public doTag(){
   // 不让标签主体内容显示, 就设么都不写
   // 要让主体内容显示, 就这么写
   JspFragment jf = getJSPBody();
   jf.invoke(out);  //  与 jf.invoke(null) 相同
}
<tag>
    <body-content>scriptless</body-content> <!--标签里面有内容单, 不写<% %>-->
</tag>
~~~~~~
2. 控制结束标签后的JSP内容不执行
~~~~~~
<itcast:hidden/>隐藏的内容
public doTag(){
    throw new SkipPageException(); // 忽略结束标签后的内容
}
<tag>
    <body-content>empty</body-content> <!--标签里面有内容单, 不写<% %>-->
</tag>
~~~~~~
3. 控制主题内容重复执行
~~~~~~
<itcast:repeat count="10">重复的内容</itcast:repeat>
public void setCount(int count); // 注入, 自动转换,仅限基本类型
public doTag(){
    for(1 to count){
        getJSPBody().invoke(null);
    }
}
<tag>
    <body-content>scriptless</body-content> <!--标签里面有内容单, 不写<% %>-->
    <attribute>
        <name>count</name>
        <required>true</required>
        <!-- 是否支持表达式 count="${3+3}"-->
        <rtexprvalue> true </rtexprvalue>
        <!-- rt:RunTime expr:Expression value -->
    </attribute>
</tag>
~~~~~~
4. 获取标签主题内容, 改变后再输出
~~~~~~
<itcast:upcase>lower<itcast:upcase>
public doTag(){
    JspFragment jf = getJSPBody();
    // 带有缓冲的数据字符输出流
    StringWriter sw = new StringWriter();
    jf.invoke(sw);
    String content = sw.getBuffer().toString();
    PageContext pc = (PageContext) getJSPContext();
    pc.getOut().write(content);
}
<tag>
    <body-content> scriptless </body-content> <!--标签里面有内容单, 不写<% %>-->
</tag>
~~~~~~



## 执行原理 ##
SimpleTag 接口中的执行原理:
* `void doTag()`: 有服务器调用. 在JSP
* `JspTag getParent()`: 由程序员调用, 获取该标签的父标签对象, 没有返回null
* `void setJspBody(JspFragment jspBody)`: 服务器, 传入标签的内容
* `void setJspContext(JspContext pc)`: 服务器, 传入当前页面的的 pageContext
* `void setParent(JspFragment jf)`: 由服务器调用, 设置该标签的父标签对象, 没有返回null


## tld 文件中的一些配置 ##
body-content的取值内容
* JSP: 不考虑, (给传统标签处理用到)
* empty: 传统和简单标签都可以使用
* scriptless: 给简单标签用的, 开始标签和结束标签内不能写 `<%%>`,
但是可以有`${1+2}`
* tagdependent: 给简单标签用的, 告诉标签类, 主体只是普通文本

## 开发属于自己的标签库 ##

### 实现if功能的标签 ###
如果test为true 就输出主题内容, 如果是false就不输出
~~~~~~
<itcast:if test="true">你好<itcast:if>
public setTest(boolean test){}
public doTag(){
    JspFragment jf = getJSPBody();
    if(test) jf.invoke(null);

}
<tag>
    <body-content> scriptless </body-content> <!--标签里面有内容单, 不写<% %>-->
    <attribute>
        <name>test</name>
        <required>true</required>
        <rtexprvalue> true </rtexprvalue>
    </attribute>
</tag>
~~~~~~

### 实现if else功能的标签库###

~~~~~~
<itcast:choose>
    <itcast:when test="true"> this is if </itcast:when>
    <itcast:otherwise> this is else </itcast:otherwise>
</itcast:choose>

// chooseTag
@BeanProperty var flag:Boolean;

// whenTag
@BeanPropertyvar test:Boolean;
def doTag(){
    if(test){
        getJSPBody().invoke(null);
        (ChooseTag)getParent().setFlag(true);
    }
}
// otherwise
def doTag(){
    if((ChooseTag)getParent().isFlag()){
        getJSPBody().invoke(null);
    }
}
~~~~~~

### 实现 for 功能的标签 ###
简单版本
~~~~~~
<itcast:forEach items="${list}" var="s">
    ${s}<br/>
</itcast:forEach>

@BeanProperty List items;
@BeanProperty String var;
def doTag(){
    val pc = (PageContext)getJSPContext();
    if(items!=null){
        for(Object obj:items){
            pc.setAttribute(var, obj);
            getJSPBody.invoke(null);
        }
    }
}
~~~~~~
复杂版本
~~~~~~
<!--or items=<%= list %> -->
<itcast:forEach items="${list}" var="s"> 
    ${s}<br/>
</itcast:forEach>

@BeanProperty Object items;
@BeanProperty String var;
val collection:Collection = new ArrayList();
def setItems(Object items) {
    if(items instanceof List){
        collection = (List)items;
    } else if(items instanceof Map){
        collectoin = ((Map)items).entrySet();
    } else if(items.getClass().isArray()) {
        int len = Array.getLength(items);
        for(int i=0;i<len;i++){
            collection.add(java.reflect.Array.get(items, i));
        }
    } else {
        throw new RuntimeException("不支持的类型");
    }
}
def doTag(){
    val pc = (PageContext)getJSPContext();
    for(Object obj:collection){
        pc.setAttribute(var, obj);
        getJSPBody.invoke(null);
    }

}
~~~~~~
#### 数组的反射 ####
~~~~~~
int ii[] = {1,2,3};
String strs[] = {"a", "b"};
Class clazz1 = ii.getClass();
Class clazz2 = strs.getClass();
clazz1.isArray();

int len = Array.getLength(items);
for(int i=0;i<len;i++){
    collection.add(java.reflect.Array.get(items, i));
}
~~~~~~
### 实现html转义标签 ###

~~~~~~
<% pageContext.setAttribute("s", "<hr/>") %>
<itcast:htmlFilter>
    ${s}
</itcast:htmlFilter>
def doTag(){
    StringWriter sw = new StringWriter();
    getJSPBody().invoke(sw);
    String content = sw.getBuffer().toString();
    content = filter(content);
}
~~~~~~


# JSTL中的核心标签 #

~~~~~~
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<!-- 输出, 可转义 -->
<c:out value="${p}" default="没有值" escapeXml="true"> </c:out>

<!-- 设置值 -->
<c:set value="上海" var="s" scope="page"></c:set>
<!-- 设置JavaBean的属性 -->
<jsp:useBean id="person" class="cn.itcast.domain.Person"></jsp:useBean>
<c:set property="name" target="${person}" value="xxx"> </c:set>
<!-- 设置Map的key和value的值 -->
<c:set property="b" value="bbb" target="${map}"> </c:set>

<!-- 从指定域范围中删除数据, 不指定scope会全删除 -->
<c:remove var="s1" scope="page"> </c:remove>

<!-- 相当于java的catch -->
<c:catch var="e"> <%=1/0%> </c:catch> ${e.message}

<!-- foreach -->
<c:forEach begin="1" end="10" var="s"> ${s} </c:forEach>
<!-- strs = abcdefg -->
<c:forEach items=${strs} begin="1" end="10" step="2" var="s"> ${s} </c:forEach>
<table border="1" width="438">
    <tr>
        <th> 姓名 </th>
        <th> 性别 </th>
        <th> 城市 </th>
        <th>成绩 </th>
    </tr>
    <!-- vs指向一个对象, 该对象记录着当前遍历 的元素的一些信息 -->
    <!-- int getIndex(): 当前遍历的元素的索引号,从零开始 -->
    <!-- int getCount(); 当前遍历元素的位数, 从一开始 -->
    <!-- boolean isLast(); -->
    <!-- boolean isFirst(); -->
    <c:forEach items="${ps}" var="p" varStatus="vs">
        <th> ${p.name}  </th>
        <th> ${p.gender} </th>
        <th> ${p.city} </th>
        <th> ${p.grade} </th>
    </c:forEach>
</table>

<!-- 遍历字符串 20140417-->
<c:set var="s3" value="2014-04-17"> </c:set>
<c:forToken items="${s3}" delims="=-" var="s"> ${s}</c:forToken>

<!-- 可以包含任何页面,包括任何界面 -->
<c:import url="/3.jsp"> </c:import>
<c:import url="http://www.baidu.com"> </c:import>

<!-- 转发 -->
<c:redirect url="**.jsp">
</c:redirect>

<!-- 对中文参数进行url编码, 能对其进行URL重写 -->
<!-- pageContext.setAttribute("url", request.getContextPath()+"/3.jsp?") -->
<c:url value="3.jsp" var="url" scope="page">
    <c:param name="username" value="我擦"> </c:param> 
</c:url>
~~~~~~
