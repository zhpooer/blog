title: 传智播客day29-struts深入
date: 2014-05-14 09:01:10
tags:
- 传智播客
- struts
---

# Struts的工作原理及核心过滤器 #

StrutsPrepareAndExecuteFilter过滤器其实是包含2部分的
1. StrutsPrepareFilter:做准备
2. StrutsExecuteFilter：进入Struts2的核心处理。
如果是Struts2的请求就会进入该过滤器，处理完后，不放行（由结果类负责显示）。
如果是非Struts2的请求，比如默认jsp的请求，直接放行。

如果用不到其他过滤器，配置StrutsPrepareAndExecuteFilter即可;
如果用到其他过滤器，还需要使用Struts2准备好的环境，
使用`StrutsPrepareFilter`，`StrutsExecuteFilter`个过滤器，其他过滤器放在两者之间.
~~~~~~
<filter-mapping>
    <filter-name> struts-prepare </filter-name>
    <url-pattern>/* </url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name>sitemesh </filter-name>
    <url-pattern>/* </url-pattern>
</filter-mapping>
<filter-mapping>
    <filter-name> struts-execute </filter-name>
    <url-pattern>/* </url-pattern>
</filter-mapping>
~~~~~~

![struts core](/img/struts_core.png)
showcase: 各种应用的案例，在struts2-showcase里面找各种案例

## 访问 struts2 静态资源 ##
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

# Action的模式 #
* action 是多实例的, 每请求一次, 将会创建一个对象
* 不存在线程安全问题

## 自动注入 ##
~~~~~~
<form>
    <input type="text" name="address.city"/>
    <input type="checkbox" name="hobby" value="吃饭"/>
    <input type="checkbox" name="hobby" value="睡觉"/>
</form>
~~~~~~
~~~~~~
public class Action extends ActionSupport {
     @BeanProperty private Address address;
     @BeanProperty private String[] hobby;
     @Override public String execute(){
         println(address.getCity());
         println(hobby); // struts2 会自动注入
     }
}
~~~~~~

## 在动作类中获取 servlet 相关对象引用 ##
1. ServletActionContext的静态方法可以得到Servlet相关的对象
~~~~~~
//用Servlet相关的对象request response servletContext HttpSession
HttpServletRequest request = ServletActionContext.getRequest();
HttpServletResponse response = ServletActionContext.getResponse();
ServletContext sc = ServletActionContext.getServletContext();
HttpSession session = request.getSession();
~~~~~~
2. 实现ServletRequestAware接口，struts2框架就会把request对象注入进来，通过拦截器servletConfig。

    Action实现如下接口，struts框架则会为其注入相应的Servlet API对象：
`ServletRequestAware`, `ServletResponseAware`, `ServletContextAware`, 实现其他对象或者功能，参考拦截器servletConfig。

# struts2 数据存储和显示 #


在 servlet 中, 把数据放在**request, session, application*域中,
在页面上利用el表达式来解决数据的存储和显示

OGNL表达式就是针对一个称之为OGNL根对象和一个称为OGNL Context的Map对象进行操作的语言。

OGNL表达式可以寻址Context内部的对象和直接调用根对象的属性或方法。

## 获取域数据 ##

~~~~~~
ActionContext ac = ActionContext.getContext();
ac.put("p", "request Scope"); // 相当于 req.setAttribute("", "")
Map applicationMap = ac.getApplication()
applicationMap.put("p", "application scope");  // servletContext.setAttribute();
Map sessionMap = ac.getSession();
sessionMap.put("p", "session scope");
~~~~~~

## ValueStack ##
* 在struts2中所有的数据都在 ValueStack 中
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
### ValueStack 结构分析 ###

OGNL是从一个称之为根栈和contextMap中取数据
* CompoundRoot就是根的类型。它是List。一般称之为根栈.取对象的某个属性，OGNL表达式不需要加任何东西，直接写属性即可
~~~~~~
<!-- 取得valueStack 中name的值,从上往下找-->
name:<s:property value="name"/><br/>
<!-- 打印的结果是name这个字符串 -->
<s:property value="'name'"/>
~~~~~~
* OGNL中的contextMap：key和value. 使用#开头，表示从Context里面取数据。
~~~~~~
<s:property value="#session.p"/>
~~~~~~
* ActionContext常用的方法(Map)
  * `put()`方法和`get()`方法就是往该Context Map对象中添加数据和取数据。
  * `getApplication()`得到application域中的所有attribute的map对象;
  * `getSession()`得到代表session域中的所有attribute的map对象；
  * `getParameters()`得到代表所有请求参数的map对象；
  * `getLocale()`()得到当前用户的Locale信息，是综合了session中保存的Locale与浏览器请求消息中的Locale的结果。
  
~~~~~~
ActionContext ac = ActionContext.getContext();
Map<String,Object> map = ac.getSession();
out.write((String)map.get("p"));
ValueStack vs = ac.getValueStack();
String s = (String)((Map<String,Object>)vs.getContext().get("session")).get("p");
ac.put("username", "cll");//向contextMap中放数据
s = (String)ac.get("username");//取数据
~~~~~~

ServletActionContext类继承了ActionContext类，它额外再提供了一些方便的方法，主要是直接返回Servlet有关的API，
例如，返回HttpServletRequest和HttpServletResponse等，它内部还是调用ActionContext内部保存的那个OGNL Context Map对象。

### ValueStack中常用方法详解 ###
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


## struts2显示(ognl) ##
ognl: struts2的标签, 把 valueStack 中的数据显示到页面上

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


#### property 标签用法 ####
`<s:property>`标签用于输出某个OGNL表达式的值，并进行HTML和XML实体转换，
可以认为其内部使用的是ValueStack对象的findString()方法。
如果没有设置value属性，则输出ValueStack栈顶的对象，
等效于输出“top”这个特殊的OGNL表达式，”top”表示栈顶的对象。
如果采用不加#前缀的方式输出Context中的某个对象，
这个对象必须是String类型，以此可以说明该标签内部调用的是ValueStack.findString()方法。

~~~~~~
<s:property value="name"/><br/><!-- 相当于ValueStack.findString() -->
<s:property value="top.name"/><br/><!-- top代表栈顶对象,取栈顶对象中的name值  -->
<s:property/><br/><!-- 取栈顶对象 -->
<s:property value="[1].name"/><br/><!-- [1]表示砍掉1个.再取第一个的name属性,后面的不管 -->
<s:property default="木有" value="abc"/><br/><!-- 表示如果没有abc这个属性,则返回default的木有 -->
<s:property value="'<hr/>'" escapeHtml="false"/><!-- escapeHtml默认的是ture,表示打印<hr/> -->

<!--请求url:  url?a=hello  -->
<s:property value="#parameters.a[0]"/> <!-- 输出: hello -->

<s:property value="#session.key"/>

<s:property value="#application.key"/>

<!-- ServletActionContext.getServletContext().setAttribute("key", "value2") -->
<!-- 从 request session application 域中依次查找-->
<s:property value="#attr.key"/> <!-- 输出: value2 -->
~~~~~~

### ognl 的方法调用 ###
支持方法调用, 也可以支持静态方法调用
~~~~~~
<!-- person = Person{name=zhang, id=3L}
     ServletActionContext.getRequest().setAttribute("person", person) -->
<s:property value="#request.person.getName()"/> <!-- 输出: shang -->
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
<s:iterator value="#map">
    <s:property value="key"/>
    <s:property value="value.name"/>
</s:iterator>

<!-- 迭代输出Map栈中的Set -->
<!-- personSet = Set(p1, p2, p3) -->
<!-- ActionContext.getContext().put("set", personSet) -->
<s:iterator value="#set">
    <s:property value="name"/>
    <s:property value="pid"/>
</s:iterator>

<!-- 迭代输出Map栈中的List<Map<String, Person>> -->
<!-- listMap = List(Map("person1"->p1, "person2"->p2)) -->
<!-- ActionContext.getContext().put("listMap", listMap) -->
<s:iterator value="#listMap">
    <s:iterator>            <!-- map 被放入对象栈栈顶 -->
        <s:property value="key"/>
        <s:property value="value.name"/>
    </s:iterator>
</s:iterator>

<!-- 迭代输出map栈中的Map<String, List<Person>> -->
<!-- 看官可以脑补, 关键点: 迭代时, 当前迭代对象会被放到栈顶, 所以不用声明 iterator.value -->

<!-- 输出 List<Map<String, List<Person>>>, 看官亦可脑补 -->
~~~~~~

### 其他操作标签 ###
~~~~~~
<!-- <s:set>标签用于将某个值存入指定范围域中，
通常用于将一个复杂的ognl表达式用一个简单的变量来进行引用。 -->
<!-- scope属性：指定变量被放置的范围，该属性可以接受
application、session、request、 page或action。该属性的默认值为action -->
<!-- 给 request 域中的键值对取别名 -->
<s:set value="#request.request_username" var="newname" scope="request"/>
<s:property valut="#request.newname"/>

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
~~~~~~

#### ui标签 ####

~~~~~~
<!--
    action 为提交form表单时的url
    method 默认为post
-->
<s:form action="" id="" name="" method="">
   <!-- struts 解析后, 会自动插入表格标签,
       所以插入 <table> <tr> <tr> 标签时, 会与其发生冲突
       可以通过设置在struts.xml中 struts.ui.theme 为simple, 来避免冲突
       -->
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
<checkboxlist list="#personList" listValue="name" listKey="pid" name="pid"/>


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

### struts 参数设置 ###
struts 在启动时, 会自动加载 /org/apache/struts2/default.property,
来配置 struts 的运行参数, 也可以在`struts.xml`中配置

* struts.i18n.encoding=UTF-8 struts2 默认的编码时utf8, 所以可以不用编码过滤器
* struts.action.extension=action  url默认扩展名
* struts.devMode=false  开发模式, 若为true, 则修改配置文件后, strut2会自动加载
* struts.ui.theme=xhtml  ui的主题, 可以设为 simple

~~~~~~
<struts>
    <constant name="struts.devMode" value="true/>
    <constant name="struts.ui.theme" value="simple/>
</struts>
~~~~~~
