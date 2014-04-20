title: 传智播客day13-EL表达式
date: 2014-04-16 11:42:31
tags:
- java
- EL表达式
- 传智播客
---

# EL表达式 #
* 作用, 向界面输出内容, 只适合显示数据
* 基本语法形式: ${EL表达式}
* **不支持字符串连接操作**
* **永远没有空指针**

~~~~~~
<jsp:JavaBean id="p" class="cn.itcast.Person"></jsp:JavaBean>
<%=p.getName()%> <!--Java表达式, 官方不建议使用-->
${p.name} <!--EL表达式, 官方推荐-->
~~~~~~
* 作用分解

    1. 获取数据, 替换JSP中的`<%= %>`
    2. 执行简单的数学或逻辑运输
    3. EL隐式对象(难点, 容易和JSP的隐式对象混淆)
    4. 调用Java中的静态方法

## 获取数据 ##
只能获取四大域对象中的数据
~~~~~~
<%
   pageContext.setAttribute("p", p, page|request|session|application);
%>

${p} <!--依次查找名字为p的对象, 没有找到页面什么都不显示你-->
${p.name} <!-- 获取对象属性的值,显示到页面上 -->
${p.addreess.province}

<!-- 使用`[]`运算符, 获取 List 和 Map 中的数据
   但是无法获取 Set 中的数据
-->
<%
    List<Person> ps = new ArrayList<Person>
    ps.add(new Person("a"));
    ps.add(new Person("b"));
    ps.add(new Person("c"));
    ps.add(new Person("d"));
    pageContent.setAttribute("ps", ps);
    Map<String, Person> m = new HashMap<String, Person>;
    m.put("aa", new Person("a"));
%>

${ps[2].name}  <!-- output: d -->
${ps[2]['name']}

${m.aa.name}   <!-- output: a -->
${m['a'].name} <!-- output: a -->
~~~~~~

## 执行数据 ##
* empty 运算符: 判断一个对象是不是null,
还能判断一个集合中是否有元素, 是否为空字符串
* 三元判断符: `${empty user? "true":"false"}`

~~~~~~
${1+1}      2
${1==2}     false

${empty p}  p="" => true; p=null => true; p=List() => true; p=List("dfjk") => false

${empty user?"您还没有登陆":"欢迎您"}
${"a" + "b"} <!--报错!!!!!!!!!!!!不能字符串相加--->
~~~~~~

## EL中的11个隐式对象 ##

|   内置对象|        表示类型               |     备注      |对应的JSP内置对象|
|--------------------------------------------------------------|
| *pageContext* | javax.servlet.jsp.PageContext     |    和jsp的内置对象完全一样   | pageContext     |
| *requestScope*| java.util.Map             | 代表servletRequest中的那个Map     | 没有     |
| *pageScope*   | java.util.Map             | 代表pageContext中的那个Map     | 没有     |
| *sessionScope*| java.util.Map             | 代表session中的那个Map     | 没有     |
| *applicationScope* | java.util.Map        | 代表application中的那个Map     | 没有     |
| *param*       | Map<String,String>        | 获取单一请求参数              |  没有    |
| paramValues   | Map<String,String[]>      | 获取重名请求参数            |  没有    |
| header        | Map<String,String>        | 单一请求消息头              | 没有     |
| headerValues  | Map<String,String[]>      | 重名请求消息头              | 没有     |
| initParam     | Map<String,String>        | 代表web.xml中的全局变量     | 没有     |
| cookie        | Map<String,Cookie>        | 获取Cookie对象              |    没有  |

~~~~~~
<!-- pageContent -->
<%= request.getContextPath() %>
${pageContext.request.contextPath}  <!-- request.contextPath 非法 -->

${param.name} <!-- 获取单一请求参数的值 -->
${paramValues.password[0]}  <!-- 获取多个请求参数的值 -->

${header['Accept-Language']}  <!-- 请求消息头 -->
${headerValues['Accept-Language'][0]}  <!-- 重名的请求消息头 -->

${cookie.JSESSIONID} <!-- cookie对象 -->
${cookie.JSESSIONID.value} <!-- cookie对象的value -->
${cookie.JSESSIONID.name} <!-- cookie对象的name -->
~~~~~~

## 调用Java中的静态方法 ##
产生原因: EL表达式不支持字符串的相关功能

EL 能调用普通Java类的静态方法, 但是不能调用实例的方法

开发步骤:
1. 定义一个静态类, 提供静态方法
2. 在WEB-INF目录下建立一个扩展名为tld的xml文件
3. 在jsp中使用

~~~~~~
<%@ taglib uri="http://www.itcast.cn/jsp/myfunction" prefix="myfn" %>
${myfn:toUppercase(s)}
~~~~~~
WEB-INF/myfn.tld
~~~~~~
<taglib>
    <tlib-version>1.0 </tlib-version>
    <short-name>myfn </short-name>
    <!-- 定位符 -->
    <uri>http://www.itcast.cn/jsp/myfunction </uri>
    <function>
        <description>toUppercase the string</description>
        <name>toUppercase </name>
        <function-class>cn.itcast.MyFunctions </function-class>
        <function-signature>java.lang.String toUppercase(java.lang.String)</function-signature>
    </function>
</taglib>
~~~~~~
## JSTL ##
sun提供了标准的EL函数

* JSTL标签库

    * core: 核心
    * fmt: 国际化
    * sql: 数据库
    * xml: 操作xml
    * functions: EL函数
* 由 Apache 实现, 需要导入 jstl.jar, standard.jar

   所有的java开发规范由JCP.org发布的, 代号 JSR-XXX
   JCP组织由: Oracle\Apache\Jboss 等知名开源组织构成
* 使用JSTL中的EL函数
~~~~~~
<% taglib uri="http://java.sun.com/jsp/jstl/functions" prefix=fn %> <!--导入-->
${fn:toLowerCase(s1)}
${fn:toUpperCase(s1)}
${fn:trim(s1)}
${fn:length(s1)}
${fn:escapeXml(s1)}  <!-- <hr/> 转义成 &lt;hr/&gt; -->
${fn:split(s1, "-")[0]}
${fn:join(s1, "-")[0]}  <!-- 联接 -->
${fn:indexOf(s1, "f")} <!--打印字符索引-->
${fn:contains(s1, "a")}  <!--是否包含字母-->
${fn:subString(s1, 0, 2)}  <!-- abcdefg => abc  -->
${fn:subString(s1, 5, 1000)}  <!-- abcdefg => fg, 数组不会越界, 只能是三个参数-->
<!-- startWith -->
<!-- endsWith -->
<!-- replace -->
<!-- subStringAfter -->
<!-- subStringBefore -->
~~~~~~
