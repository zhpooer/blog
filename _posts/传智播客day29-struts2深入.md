title: 传智播客day29-struts深入
date: 2014-05-14 09:01:10
tags:
- 传智播客
- struts
---

# Action的模式 #
* action 是多实例的, 每请求一次, 将会创建一个对象
* 不存在线程安全问题 

# struts2 结果集(result) #
结果集的处理方式有多种, 通过 `type=**` 设置, 最常用的有
`dispatcher` `redirect` `redirectAction`
~~~~~~
<!-- 请求转发到 jsp 或 其他资源 -->
<result name="*" type="dispatcher"> </result>

<!-- 请求重定向到 jsp 或 其他资源-->
<result name="*" type="redirect"> </result>

<!-- 重定向到action -->
<result name="*" type="redirectAction"> </result>
~~~~~~

# struts2 数据存储和显示 #

在 servlet 中, 把数据放在**request, session, application*域中,
在页面上利用el表达式来解决数据的存储和显示

## ValueStack ##
* 在struts2中所有的数据都在 ValueStack 中
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
* ValueStack 结构分析
~~~~~~java:
// OnglValueStack implements ValueStack
ValueStack vs = ActionContext.getContext().getValueStack();
// CompundRoot extends ArrayList, 对象栈, 存放的是对象
// root[0] -> ValueStackAction; 当前请求的action
// root[1] -> DefaultTextProvider; 国际化
CompundRoot root = vs.getRoot();

// context isInstanceof OnglContext, Map 栈
// CompundRoot root = OnglContext.getRoot()
// context["request"] -> requestMap
// context["session"] -> sessionMap
// context["application"] -> applicationMap
// context["action"] -> 当前请求的 action
Map<String, Object> context = vs.getContext();
~~~~~~
* 操作对象栈数据
~~~~~~
ValueStack vs = ActionContext.getContext().getValueStack();
//1. 添加到栈顶
vs.push(obj);
//2. 添加到栈尾
vs.getRoot.add(obj);
//3. 把一个map放入到了对象栈的栈顶
vs.set("aa", obj);

// 取出数据
// 1. 取栈顶元素
vs.getRoot().get(0);
vs.peek();
// 2. 移除栈顶元素
vs.getRoot().remove(0);
vs.pop();
~~~~~~
* 操作Map栈数据
~~~~~~
ValueStack vs = ActionContext.getContext().getValueStack();
ServletActionContext.getRequest().setAttribute("key", "value");

// 存键值对到Map
ActionContext.getContext().put("key", "value");
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
~~~~~~
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
