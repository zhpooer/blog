title: 传智播客day31-struts拾遗
date: 2014-05-17 09:04:32
tags:
- 传智播客
---

# ognl 详解 #
OGNL：Object Graph Navigation Language

* OGNL不仅可以调用属性，还可以调用普通方法
~~~~~~
<!-- 表达式可以调get和set,可以调方法,可以按索引取数组内容 （打印a）-->
<s:property value='"abcdefg".toCharArray()[0]'/><br/>
<!-- ognl表达式中：context代表contextMap （打印对应的value）-->
<s:property value="#context['com.opensymphony.xwork2.ActionContext.locale']"/>
<!-- 打印11111111111111111，直接量，直接写的值,H表示大数据？ -->
<!-- 111B  表示BigDecimal类型    1111H表示BigInteger -->
<s:property value="11111111111111111H"/><br/> 
~~~~~~
* OGNL获取属性等
~~~~~~
<%
request.setAttribute("str", new String[]{"a","b","c"});
%>
<s:debug></s:debug>
<!-- OGNL中可以使用伪属性  .length（以下打印3） -->
<s:property value="#request.str.length"/><br/>
<s:property value="#request.str['length']"/><br/>
<s:property value="#request.str['len'+'gth']"/><br/>
~~~~~~
* 链式表达式(Chained Subexpressions)：
~~~~~~
<!-- 链式表达式，#this表示前面的110H.intValue() -->
<s:property value="110H.intValue().(#this<112?#this*2:#this/2)"/><br/>
~~~~~~
* 访问静态资源, 取得静态常量，用@进行间隔，加小括号表示静态方法，不加小括号就是代表取get方法
~~~~~~
<!-- 使用静态资源，需要打开静态常量开关struts.ognl.allowStaticMethodAccess -->
<!-- 表示Integer中的MAX_VALUE常量 -->
<s:property value="@java.lang.Integer@MAX_VALUE"/><br/>
<!-- 表示使用System类中的currentTimeMillis()静态方法 -->
<s:property value="@java.lang.System@currentTimeMillis()"/><hr/>
~~~~~~
* 集合对象操作
~~~~~~
<!--{'a','b','c'} 创建了一个List集合  -->
<s:iterator value="{'a','b','c'}">
    <s:property/><br/><!-- value省略，打印a b c -->
</s:iterator>
<!-- 打印class java.util.ArrayList -->
<s:property value="{'a','b','c'}.class"/><br/>

<!-- 打印集合 -->
<s:iterator value="#{'a':'aa','b':'bb'}">
<!-- 打印a:aa  b:bb -->
    <s:property value="key"/>:<s:property value="value"/><br/>
</s:iterator>

<!-- in 表达式 -->
<!-- EL表达式在Struts2中已经被改写了：原有功能保持不变。只是在四大域范围内找不到的话，EL表达式就变成了OGNL表达式 -->
<s:iterator value="#{'eat':'吃饭','sleep':'睡觉','study':'学习'}">
    <input type="checkbox" name="hobby" <s:property value="key in {'java','sleep','study'}?'checked=\"checked\"':''"/> value="${key}"/>${value}
</s:iterator><br/>
~~~~~~
* 投影查询
~~~~~~
<s:property value="products.{name}"></s:property> <!-- 只要name属性 -->
<!-- 遍历集合, 只要价格大于 1500的-->
<s:property value="products.{?#this.price>1500}"></s:property>
<!-- 遍历集合, 第一个符合-->
<s:property value="products.{^#this.price>1500}"></s:property>
<!-- 遍历集合, 最后一个符合-->
<s:property value="products.{$#this.price>1500}"></s:property> 
~~~~~~
* 类型转换
~~~~~~
<!-- 打印[0, 2, 4, 6, 8] -->
<s:property value="(5).{#this*2}"/><br/>
<s:property value=”#{'name':'wzt','age':30,'gender':'male'}.{#key}”/>发现什么都没有输出
<s:property value="#{'name':'wzt','age':30,'gender':'male'}.{#this}"/>有输出。把Map转为了Value
<!-- 打印 [wzt, 30, male]-->
<s:property value="#{'name':'wzt','age':30,'gender':'male'}.{#this}"/><br/>
<!-- 打印当前日期 -->
<s:property value="new java.util.Date()"/>
~~~~~~

技巧：如果不确定是不是OGNL表达式，那么加上%{abc}，那么abc就是一个OGNL表达式而不是字符串了
~~~~~~
<s:form action="RegUser" namespace="/user">
    <s:textfield key="hello" label="用户名" name="username" ></s:textfield>
    <s:textfield label="昵称" name="nick"></s:textfield>
    <s:password label="密码" name="password"></s:password>
    爱好：
    <s:iterator value="hobbies" var="me">
        <!-- 如果要把一个字符串当做OGNL表达式对待，请使用%{}包括起来 -->
        <s:checkbox name="hobby" fieldValue="%{key}" label="%{value}"></s:checkbox>
    </s:iterator>
    <hr/>
    <s:checkboxlist list="hobbies" name="hobby" listKey="key" listValue="value" label="新爱好"></s:checkboxlist>
    <s:radio list="#{'male':'男','female':'女'}" name="gender" label="性别"></s:radio>
    <s:submit value="注册"></s:submit>
</s:form> 
~~~~~~

# 防止表单重复提交 #
可以利用 interceptor, 防止重复提交
1. 在输入表单中加入<s:token/>标签
1. 在要防止重复提交的动作配置中加入token拦截器
1. 提交会出错，需要配置一个结果
1. 如果在返回的页面中要显示错误消息提示，使用s:actionErrors
1. 要覆盖默认的提示，请在国际消息资源文件中加入struts.messages.invalid.token=你的提示信息


加入 token 拦截器
~~~~~~
<action name="addUser" class="com.itheima.action.UserAction" method="addUser">
     <interceptor-ref name="token"></interceptor-ref>
     <interceptor-ref name="defaultStack"></interceptor-ref>
    <result>/success.jsp</result>
    <!-- 第一个结果视图是invalid.token -->
    <result name="invalid.token">/regist.jsp</result>
</action> 
~~~~~~

~~~~~~
<!-- form.jsp -->
<s:form>
    <s:token> </s:token> 
    <s:submit/>
</s:form>

<!-- error.jsp, 显示错误信息 -->
<s:actionerror></s:actionerror>
~~~~~~

# 更改校验失败视图 #
~~~~~~
public class UserAction{
    // 如果这个方法校验失败,默认视图是"input", 可以修改为 "loginInput"
    @InputConfig(resultName="loginInput")
    public String login(){}
}
~~~~~~

# 使用标准插件--JFreeChart使用 #
导入 jfreechart.jar包
~~~~~~
public class ChartAction extends ActionSupport {
    private JFreeChart chart;
    public JFreeChart getChart() {
        return chart;
    }
    public String execute() throws Exception {
        ValueAxis xAxis = new NumberAxis("年度");
        ValueAxis yAxis = new NumberAxis("产值");
        XYSeries xySeries = new XYSeries("绿豆");
        xySeries.add(0,300);
        xySeries.add(1,200);
        xySeries.add(2,400);
        xySeries.add(3,500);
        xySeries.add(4,600);
        xySeries.add(5,500);
        xySeries.add(6,800);
        xySeries.add(7,1000);
        xySeries.add(8,1100);
        XYSeriesCollection xyDataset = new XYSeriesCollection(xySeries);
        XYPlot xyPlot = new XYPlot(xyDataset,xAxis,yAxis,new StandardXYItemRenderer(StandardXYItemRenderer.SHAPES_AND_LINES));
        chart = new JFreeChart(xyPlot);
        return SUCCESS;
    }
}
~~~~~~
~~~~~~
<action name="showchart" class="com.itheima.action.ChartAction">
    <result name="success" type="chart">
        <param name="height">400</param>
              <param name="width">600</param>
    </result>
</action>
<!-- jsp页面 -->
<body>
上传成功！<br/>
  <s:url action="showchart" var="url"></s:url>
  <img alt="图图" src="${url}">
</body>
~~~~~~
使用插件,必须要先知道插件的用途,一般只要组织数据就好了,没必要深究.


# 伪装URL地址(REST) #
~~~~~~
public String showAll(){
    return SUCCESS;
}
public String queryOne(){
    //根据id的值，调用serivce，找到那个用户
    List<User> us = getUsers();
    for(User u:us){
        if(id.equals(u.getId())){
            System.out.println(u);
        }
    }
    return null;
}
~~~~~~
~~~~~~
<constant name="struts.devMode" value="true" />
<constant name="struts.action.extension" value="html"></constant>
<constant name="struts.enable.SlashesInActionNames" value="true"></constant>
<!--     <constant name="struts.mapper.alwaysSelectFullNamespace" value="true"></constant> -->
<constant name="struts.ognl.allowStaticMethodAccess" value="true"></constant>
<package name="default" extends="struts-default">
    <action name="showAll" class="com.itheima.action.UserAction" method="showAll">
        <result>/list.jsp</result>
    </action>
    <action name="users/*" class="com.itheima.action.UserAction" method="queryOne">
        <param name="id">{1}</param>
    </action>
</package> 
~~~~~~
通过 `${pageContext.request.contextPath}/users/${id}.html` 访问

# 异常处理 #
如果发生错误, 要第一时间将错误转换成自定义异常错误抛出
~~~~~~
public class MySqlException extends Exception{
}
~~~~~~

通过拦截器, 获取自定义异常
~~~~~~
public class MyExceptioinInterceptor extends AbstractInterceptor{
    @Override
    public String intercept(ActionInvocation ai) {
        try {
            return ai.invoke();
        } catch(MySqlException e) {
            ActionSupport action = ai.getAction();
            action.addActionError("");
            return "error";
        }
    }
}
~~~~~~

可以设置全局错误页面
~~~~~~
<!-- 在package中设置 -->
<global-exception-mapping>
    <exception-mapping result="error" exception="java.lang.Exception"> </exception-mapping>
</global-exception-mapping>
~~~~~~
