title: 传智播客day30-struts响应数据
date: 2014-05-15 09:01:10
tags:
- 传智播客
- struts
---

# Struts2 上传下载 #
企业常用文件上传技术: jspSmartUpload(JSP model时代), fileupload(apache commons), servlet3.0 集成文件上传

## 文件上传 ##
真正干活的是 `FileUploadInterceptor`
~~~~~~
<s:form enctype="multipart/form-data">
    <s:file name="resource"> </s:file>
</s:form>
~~~~~~

~~~~~~
public class UploadAction {
// 如果是多个文件上传, 那么就用定义数组, 如果 File[] resource;
    @BeanProperty private File resource;    // 得到文件
    @BeanProperty private String resourceContentType; // 得到文件类型 MIME
    @BeanProperty private String resourceFileName; // 得到文件的名称
    public String execution(){
        String bashPath = ServletContext.getServletContext().getRealPath("WEB-INF/upload");
        File dir = new File(bashPaht+subPath);
        dir.mkdirs();
        String path  = dir + UUIDName();
        FileUtils.copyFile(resource, new File(dir)); // common-io
        return null;
    }
}
~~~~~~
配置上传文件信息
~~~~~~
<!-- 上传出错后, 可以配置input视图 -->
<!-- 设置全局, 文件上传的最大值, 默认大小为2M, 超出大小后, actionError -->
<!-- 错误信息提示 可以根据 struts-messages.properties 来设置 -->
<constant name="struts.multipart.maxSize" value="2097152"> </constant>
<!-- 配置解析技术, jakara或pell  -->
<constant name="struts.multipart.parser" value="cos"> </constant>
<action>
    <interceptor-ref name="defaultStack">
        <!-- 设置局部  -->
        <param name="fileUpload.maximumSize">2097152</param>
        <param name="fileUpload.allowedExtensions">txt,doc,pdf </param>
        <!-- 可以在web.xml中查看, mime类型 -->
        <param name="fileUpload.allowedTypes">application/msword </param>
    </interceptor-ref>
</action>
~~~~~~

## 文件的下载 ##

~~~~~~
// http://contextPath/download.action&filename=xx.mp3
/*
<action>
    <result type="stream">
        <!-- 指定下载的文件流  -->
        <param name="inputName"> inputStream </param>
        <param name="ContentType">${contentType}</param>
        <!-- 解析下载文件名字  -->
        <param name="contentDisposition"> <!-- 浏览器打开方式 -->
            attachment;filename=%{#fileName}.txt
        </param>
    </result>
</action>
*/
public class DownloadAction {
    @BeanProperty private InputStream InputStream;
    public String download(){
        String fileName = "";
        String filePath = "";
        String agent = ServletContext.getRequest().getHeader("user-agent");
        String encoded = encodeDownloadFilename(filename, agent);
        // 附件乱码问题,(IE和其他浏览器: URL编码, 火狐: Base64编码)
        ActionContext.getContext().put("fileName", encoded);
        inputStream = new FileInputStream(filePath + fileName);
    }
    // 根据下载文件动态获取 MIME 文件类型
    public String getContentType(){
        return ServletActionContext.getServletContext().getMimeType(filename);
    }
    
    /*
	 * 下载文件时，针对不同浏览器，进行附件名的编码
	 * @param filename 下载文件名
	 * @param agent 客户端浏览器
	 * @return 编码后的下载附件名
	 * @throws IOException
	 */
     public String encodeDownloadFilename(String filename, String agent) throws IOException{
        if(agent.contains("Firefox")){ // 火狐浏览器
            filename = "=?UTF-8?B?"+new BASE64Encoder().encode(filename.getBytes("utf-8"))+"?=";
        }else{ // IE及其他浏览器
            filename = URLEncoder.encode(filename,"utf-8");
        }
        return filename;
    }
}
~~~~~~


# 访问 struts2 静态资源 #
文档中介绍了，可以把静态资源放到org.apache.struts2.static或者template包中，
可以直接访问，例如 访问template.aaa中的bbb.css，则
`http://localhost:8080/day22_03_strutsStatics/struts/aaa/bbb.css`

或者自己对静态资源访问的地址进行设置，在web.xml中设置
~~~~~~
<!-- 则访问com.itheima.statics中的资源ccc.css，的地址为 -->
<!-- http://localhost:8080/day22_03_strutsStatics/struts/ccc.css -->
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <init-param>
        <param-name>packages</param-name>
        <param-value>com.itheima.statics</param-value>
    </init-param>
</filter>
~~~~~~

# struts2 数据存储和显示 #

OGNL是Object Graphic Navigation Language(对象图导航语言)的缩写，
它是一个开源项目.Struts2框架使用OGNL作为默认的表达式语言。
* xwork 提供 OGNL表达式 
* ognl-3.0.5.jar

OGNL 是一种比EL 强大很多倍的语言 

OGNL 提供五大类功能 
1. 支持对象方法调用，如xxx.doSomeSpecial();
2. 支持类静态的方法调用和值访问
3. 访问OGNL上下文（OGNL context）和ActionContext: (重点 操作ValueStack值栈)
4. 支持赋值操作和表达式串联
5. 操作集合对象。

## 方法调用 ##
~~~~~~
<%@taglib uri="/struts-tags" prefix="s" %>
<!-- 通过 s:property 标签操作 ognl 表达式 -->
<!-- 实例方法 -->
<s:property value="'hellof world'.length()"/>
<!-- 静态方法 -->
<!-- @[类全名（包括包路径）]@[方法名]  -->
<!--  使用 静态方法调用 必须 设置 struts.ognl.allowStaticMethodAccess=true -->
<s:property value="@java.lang.String@format('您好,%s','小明')"/>
~~~~~~

## OGNL Context ##

什么是值栈(ValueStack)?

    接口, 是struts2 提供的一个接口, 实现类是`OgnlValueStack`,
    OGNL 是从值栈中获取数据的, 每个Action实例都有一个ValueStack 对象(一次请求,一个ValueStack对象)
    在其中保存当前Action对象保存名为 'struts.valueStack'的请求属性中, request中

值栈的内部结构?

    值栈由两部分组成 

    ObjectStack: Struts  把动作和相关对象压入 ObjectStack(List) 中
    
	ContextMap: Struts 把各种各样的映射关系(一些 Map 类型的对象) 压入 ContextMap 中
	    Struts 会把下面这些映射压入 ContextMap 中
		parameters: 该 Map 中包含当前请求的请求参数
		request: 该 Map 中包含当前 request 对象中的所有属性
		session: 该 Map 中包含当前 session 对象中的所有属性
		application:该 Map 中包含当前 application  对象中的所有属性
		attr: 该 Map 按如下顺序来检索某个属性: request, session, application

~~~~~~
ServletActionContext.getRequest().getAttribute("struts.valueStack");
~~~~~~

OGNL表达式, 访问root中数据时 不需要 '#',
访问 request、 session、application、 attr、 parameters 对象数据 必须写  `#`

值栈对象的创建, ValueStack 和 ActionContext 是什么关系?

    值栈对象 是请求时 创建的 
	doFilter中`prepare.createActionContext(request, response)`
		* 创建ActionContext 对象过程中，创建 值栈对象ValueStack 
		* ActionContext对象 对 ValueStack对象 有引用的(在程序中 通过 ActionContext 获得 值栈对象)
	Dispatcher类 serviceAction 方法中 将值栈对象保存到 request范围
		request.setAttribute(ServletActionContext.STRUTS_VALUESTACK_KEY, proxy.getInvocation().getStack());

### ValueStack ###
解决 Action 向 JSP 传递数据的问题
* 在struts2中所有的数据都在 ValueStack 中(核心)
* ValueStack里面有两个东西，一个是根,就是CompoundRoot，这是一个List集合，还有一个contextMap
* ValueStack 的生命周期是一次请求, 被存在 request 域`"struts.ValueStack"`中
* 获得 ValueStack 有三种方式
~~~~~~
public class ValueStackAction extends ActionSupport {
    public String testValueStack() {
        ValueStack vs1 = ActionContext.getContext().getValueStack();
        ValueStack vs2 = ServletActionContext.getContext().getValueStack();
        ValueStack vs3 = ServletActionContext.getRequest().getAttribute("struts.ValueStack");
        return "";
    }
}
~~~~~~


### ValueStack中常用方法详解 ###
对值栈的操作, 主要是对 root 的操作
* `pop` 把栈顶的remove
* `push` 放到栈顶
* `peek`获取栈顶
* `set` 往根栈里面放东西，如果栈顶是Map，则直接放进去，
如果不是Map，则建一个Map放进去, set方法不能设置栈顶的普通JavaBean对象的属性。
~~~~~~
vs.set("p", "pp");//向栈顶压入一个Map(如果栈顶就是一个Map，直接使用了)。map中的元素的key是p，value是pp
~~~~~~
* `setValue` 设置值
~~~~~~
//相当于从栈顶依次查找谁有setItheima("牛13"), 都没有则报错。不会去大Map中找
vs.setValue("itheima", "牛13");
vs.setValue("date", 18);//某个对象的属性   
~~~~~~
*  `findValue(String expr)`, 如果expr以#开头，从contextMap中取。
如果不加#，先从根栈找属性，没有找到。则从contextMap中，作为key来找了
* `findString` 如果找的是Date对象，内部会转换成String对象
~~~~~~
ac.put("now", new Date());
obj = vs.findValue("#now");
out.write(obj+" "+obj.getClass().getName());
//类型转换器把java.util.Date--->java.lang.String.确定表达式返回值就是String类型的才能用
String str = vs.findString("#now");
~~~~~~

## 值栈在开发中应用 ##
主流应用: 值栈 解决 Action 向 JSP 传递 数据问题 

Action 向JSP 传递数据处理结果 ，结果数据有两种形式 
* 消息 String类型数据
~~~~~~
// 针对某一个字段错误信息(常用于表单校验)
this.addFieldError("msg", "字段错误信息");
// 普通错误信息，不针对某一个字段 登陆失败
this.addActionError("Action全局错误信息");
// 通用消息
this.addActionMessage("Action的消息信息");

// 在jsp中使用 struts2提供标签 显示消息信息
// <s:fielderror fieldName="msg"/>
// <s:actionerror/>
// <s:actionmessage/>
~~~~~~
* 数据(复杂类型数据), 使用值栈`valueStack.push(products)`

哪些数据默认会放入到值栈?
* 每次请求，访问Action对象 会被压入值栈, `DefaultActionInvocation` 的 init方法 `stack.push(action);`
* Action如果想传递数据给 JSP，只有将数据保存到成员变量，并且提供get方法就可以了
* 如果Action 实现ModelDriven接口，值栈默认栈顶对象 就是model对象
~~~~~~
<!-- 在拦截器中 ，将model对象 压入了 值栈 stack.push(model); -->
<interceptor name="modelDriven" class="com.opensymphony.xwork2.interceptor.ModelDrivenInterceptor"/>
~~~~~~


## EL 访问值栈的数据 ##

struts对`Request`对象进行了包装, 重写request的 `getAttribute`
~~~~~~
Object attribute = super.getAttribute(s);
// 访问request范围的数据时，如果数据找不到，去值栈中找 
if (attribute == null) {
    attribute = stack.findValue(s);
}
~~~~~~
* request 对象 具备访问值栈数据的能力(查找root的数据)


## struts2显示(ognl) ##
ognl: struts2的标签, 把 valueStack 中的数据显示到页面上

常见符号 `#` `%` `$`
* `%`的使用
~~~~~~
<!-- 用法一： 结合struts2 表单表单使用， 通过%通知struts，
     %{}中内容是一个OGNL表达式，进行解析  -->
<s:textfield name="username" value="%{#request.username}"/>
<!-- 用法二： 设置ognl表达式不解析 %{'ognl表达式'} -->
<s:property value="%{'#request.username'}"/>
~~~~~~
* `$`的使用 
  * 用法一: 用于在国际化资源文件中，引用OGNL表达式 
~~~~~~
<!-- 在properties文件 msg=欢迎您, ${#request.username} -->
 <!-- 自动将值栈的username 结合国际化配置信息显示  -->
<s:i18n name="messages">
    <s:text name="msg"></s:text>
</s:i18n>
~~~~~~
  * 用法二 ：在Struts 2配置文件中，引用OGNL表达式
~~~~~~
<!-- 在Action 提供 getContentType方法 -->
<!-- 读取值栈中contentType数据,在Action提供 getContentType -->
<!-- 因为Action对象会被压入值栈,contentType是Action属性,从值栈获得 -->
<param name="contentType">${contentType}</param>
~~~~~~

结论: `#`用于写ognl表达式获取数据,`%` 控制ognl表达式是否解析,
`$` 用于配置文件获取值栈的数据 

### 调试 ###
~~~~~~
<!-- 导入标签,位置在 struts2-core/META-INF -->
<%@taglib prefix="s" uri="/struts-tags"%>

<!-- 超级链接, 显示ValueStack中的内容 -->
<s:debug/> 
~~~~~~

### 访问 valuestack 中的数据 ###
可通过`<s:property/>` 标签访问栈顶元素及其栈中所有对象的属性,

~~~~~~
<!-- 对象栈, 栈中元素: Person{name="zhang", pid=1L},
Person2{nickname=zhang2, pid=2L}
如果不写value属性, 则输出对象栈的栈顶元素全部 -->
<s:property /> <!-- 输出: Person@1245af -->
<s:property value="name"/> <!-- 输出: zhang -->
<s:property value="nickname"/> <!-- 输出: zhang2 -->
<!-- 如果对象栈中特有相同属性对象, 则访问最先找到的对象的属性 -->
<s:property value="pid"/>  <!--  输出: 1 -->

<!-- 将map放入对象栈中 -->
<!-- valueStack.set("key", "value1"); -->
<s:property value="key"/>  <!--  输出: value1 -->
~~~~~~

访问Map栈中的值,通过`<property value="#key"/>`
~~~~~~
<!-- 键值对放入 map 栈中 -->
<!-- ActionContext().getContext().put("key", "value1") -->
<s:property value="#key"/> <!-- 输出: value1 -->

<!-- 访问Request域中的数据 -->
<!-- ServletActionContext.getRequest().setAttribute("key", "value2") -->
<s:property value="#request.key"/> <!-- 输出: value2 -->

<!-- person = Person{name=zhang, id=3L}
ServletActionContext.getRequest().setAttribute("person", person) -->
<s:property value="#request.person.name"/> <!-- 输出: zhang -->
~~~~~~

### struts标签库 ###

#### property 标签用法 ####
`<s:property>`标签用于输出某个OGNL表达式的值，并进行HTML和XML实体转换,  
可以认为其内部使用的是ValueStack对象的findString()方法.  
如果没有设置value属性，则输出ValueStack栈顶的对象,  
等效于输出“top”这个特殊的OGNL表达式，”top”表示栈顶的对象.  
如果采用不加#前缀的方式输出Context中的某个键值对,  
这个对象必须是String类型，以此可以说明该标签内部调用的是ValueStack.findString()方法。

~~~~~~
<s:property value="[0].top"/>   <!-- [0].top 第一个元素对象, [1].top第二个元素对象 -->
<s:property value="[1].name"/><br/><!-- 取栈顶第二个对象中的name值  -->
<s:property value="name"/><br/><!-- 相当于ValueStack.findString() -->
<s:property value="top.name"/><br/><!-- top代表栈顶对象,取栈顶对象中的name值  -->
<s:property/><br/><!-- 取栈顶对象 -->
<s:property default="木有" value="abc"/><br/><!-- 表示如果没有abc这个属性,则返回default的木有 -->
<s:property value="'<hr/>'" escape="false"/><!-- escape默认的是ture,表示打印<hr/> -->

<!--请求url:  url?a=hello  -->
<s:property value="#parameters.a[0]"/> <!-- 输出: hello -->

<s:property value="#session.key"/>

<s:property value="#application.key"/>

<!-- ServletActionContext.getServletContext().setAttribute("key", "value2") -->
<!-- 从 request session application 域中依次查找-->
<s:property value="#attr.key"/> <!-- 输出: value2 -->
~~~~~~

### ognl 数组输出 ###
~~~~~~
<!-- p1=Person{name=乙, pid=1L} -->
<!-- p2=Person{name=甲, pid=2L} -->
<!-- p3=Person{name=丙, pid=3L} -->
<!-- personlist = ArrayList(p1, p2, p3) -->
<table>
    <th>
        <td> name </td>
        <td> id </td>
    </th>
    <!-- 迭代输出集合(Collection), 数组, Map
    value 指向valueStack中要遍历的集合当前正在迭代的元素会放到栈顶
    var 指向当前遍历的元素, 放在Map栈中
    status 指向当前元素的遍历信息, 有方法
    getIndex(), isLast(), getCount(), isEven(), isOdd(), isFirst()
    begin 开始位置
    step 步长
    end 结束位置
    -->
    <!-- 方式一, 把数据放在Map栈中 -->
    <!-- ActionContext.getContext().put("personList", personList) -->
    <!-- p 在根栈, 和 map栈都放一份-->
    <s:iterator value="#personList" var="p" status="st">
        <tr class="<s:property value="#st.odd?'even':'odd'"/>">
            <td> <s:property value="name"/> </td>
            <td> <s:property value="#p.pid"/> </td>
        </tr>
    </s:iterator>
    
    <!-- 方式二, 把数据放在Map栈中 -->
    <!-- 如果value值不写, 则默认迭代对象栈栈顶元素
        valueStack.push(personList)-->
    <s:iterator>
        <tr>
            <td> <s:property value="name"/> </td>
            <td> <s:property value="pid"/> </td>b
        </tr>
    </s:iterator>
</table>
~~~~~~

### 其他容器的迭代 ###
~~~~~~
<!-- p1=Person{name=乙, pid=1L} -->
<!-- p2=Person{name=甲, pid=2L} -->
<!-- p3=Person{name=丙, pid=3L} -->

<!-- 迭代输出Map栈中的map -->
<!-- personMap = Map("person1" -> p1, "person2" -> p2, "perseon3" -> p3) -->
<!-- ActionContext.getContext().put("map", personMap) -->
<s:iterator value="#request.map">
    <s:property value="key"/>
    <s:property value="value.name"/>
</s:iterator>

<!-- 迭代输出Map栈中的Set -->
<!-- personSet = Set(p1, p2, p3) -->
<!-- ActionContext.getContext().put("set", personSet) -->
<s:iterator value="#request.set">
    <s:property value="name"/>
    <s:property value="pid"/>
</s:iterator>

<!-- 迭代输出Map栈中的List<Map<String, Person>> -->
<!-- listMap = List(Map("person1"->p1, "person2"->p2)) -->
<!-- ActionContext.getContext().put("listMap", listMap) -->
<s:iterator value="#request.listMap">
    <s:iterator>            <!-- map 被放入对象栈栈顶 -->
        <s:property value="key"/>
        <s:property value="value.name"/>
    </s:iterator>
</s:iterator>

<!-- 迭代输出map栈中的Map<String, List<Person>> -->
<!-- 看官可以脑补, 关键点: 迭代时, 当前迭代对象会被放到栈顶, 所以不用声明 iterator.value -->

<!-- 输出 List<Map<String, List<Person>>>, 看官亦可脑补 -->
~~~~~~

### 其他标签 ###

~~~~~~
<!-- <s:set>标签用于将某个值存入指定范围域中，
通常用于将一个复杂的ognl表达式用一个简单的变量来进行引用。 -->
<!-- scope属性：指定变量被放置的范围，该属性可以接受
application、session、request、 page或action。该属性的默认值为action -->
<!-- 给 request 域中的键值对取别名 -->
<s:set value="#request.request_username" var="newname" scope="request"/>
<s:set value="%{'张三'}" var="newname" scope="request">/
<s:property value="#request.newname"/>

<!-- 将request域的元素压入对象栈, 并在用完之后, 从对象栈中删除初-->
<s:push value="#request.request_username"> 
    <s:property/> <!--因为是在栈顶也可以用 s:property 读取元素-->
</s:push>

<!-- Var:赋值给变量的值。放置在request作用域中。
     如果没有设置该属性，对象被设置到栈顶。 -->
<s:bean name="className" var="myperson">
    <s:param name="pid" value="#request.pid"/>
    <!--因为是在栈顶也可以用 s:property 读取元素-->
</s:bean>

<s:if test="#person.pid<3"> 小于三 </s:if>
<s:elseif test="#person.pid>3"> 大于三 </s:esleif>
<s:else test="#person.pid==3"> 等于三 </s:esle>

<!-- 编码转义 -->
<s:url action="" namespace="/" >
    <s:param name="" value="%{'good'}"> </s:param>
</s:url>

<!-- 是一个链接, 可以对中文进行转义 -->
<s:a action="" namespace="">
    <s:param name="" value="%{'test'}"> </s:param>    
</s:a>

<!-- 为值栈中的数据进行赋值 -->
<s:property value="price=1000, name='冰箱"></s:property>
~~~~~~

#### ui标签 ####
struts2 除了支持JSP之外，支持两种模板技术 Velocity (扩展名 .vm ),
Freemarker (扩展名 .ftl)
~~~~~~
<!--
    action 为提交form表单时的url
    method 默认为post
-->
<s:form action="" id="" name="" method="" theme="xhtml">
   <!-- struts 解析后, 会自动插入表格标签,
       所以插入 <table> <tr> <tr> 标签时, 会与其发生冲突
       可以通过设置在struts.xml中 struts.ui.theme 为simple, 来避免冲突
       -->
    <s:label value="" for="password"/>
    <s:textfield name="name" id="" value="" label=""/>
    <s:password id="" name="" showpassword="" lable="" value=""/>
    <s:hidden name="" id="" value=""/>
    <s:submit type="submit|button|image " value="" name=""/>
    <s:reset type="input|button" value="" name=""/>
    <s:texterea name="" id="" value="" cols="" rows="" lable=""/>

</s:form>
~~~~~~

案例
~~~~~~
<!-- p1=Person{name=乙, pid=1L} -->
<!-- p2=Person{name=甲, pid=2L} -->
<!-- p3=Person{name=丙, pid=3L} -->
<!-- personlist = ArrayList(p1, p2, p3) -->
<!-- ActionContext.getContext().put("personList", personList) -->
<!--
     listKey 为option中的value
     listValue 为option中的标签内容
     headerKey 表示第一个option中的value
     headerValue 表示第一个option标签中的内容
-->
<s:select list="#personList" listKey="pid" listValue="name" headerKey="" headerValue=""/>
<s:checkboxlist list="#personList" listValue="name" listKey="pid" name="pid"/>


<!-- 在文本框中把person中的name属性进行回显 -->
<!-- person = Person{name="test" pid=1L} -->

<!-- ActionContext.getContext.put("person", person) -->
<s:textfield name="name" value="%{#person.name}"/> <!--输出: test -->

<!-- valueStack().push(person), 放入对象栈,
     可以不加value, 根据name属性进行回显
     s:textfield, s:textarea, s:password 都可以用-->
<s:textfield name="name" /> <!--输出: test -->

<!-- TestAction{ @Beanproperty aa}
action = new testAction(); action.setAa("test2");
-->
<s:textfield name="aa" /> <!--输出: test2-->
~~~~~~
