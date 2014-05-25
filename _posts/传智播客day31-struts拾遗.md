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
<!-- 打印fengjie=fengjie -->
${name}=<s:property value="name"/><br/>
<!-- 打印[10, 6, 4, 0] -->
<s:property value="{5,3,2,0}.{#this*2}"/>
<!-- 打印[5, 3, 2] -->
<s:property value="{5,3,2,0}.{?#this*2}"/>
<!-- 打印 [5] -->
<s:property value="{5,3,2,0}.{^#this*2}"/>
<!-- 打印 [2] -->
<s:property value="{5,3,2,0}.{$#this*2}"/><br/>
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

## struts 配置文件中的 ognl 表达式 ##
~~~~~~
<!-- request.setAttribute("uri", "test") -->
<result name="test">
%{#request.uri}.jsp
</result>
~~~~~~


# 表单校验 #
普通式的开发，用声明式的，细致的控制，用编程式的验证
## 编程式验证 ##
用代码，表达式的方式

前提：动作类一般要求继承`ActionSupport`

struts.xml配置文件中,对要验证的动作,需要提供name="input"的结果视图(回显)
~~~~~~
<action name="RegUser" class="com.itheima.action.UserAction" method="RegUser">
    <result type="dispatcher" name="success">/WEB-INF/pages/main.jsp</result>
    <result type="dispatcher" name="error">/WEB-INF/pages/commons/error.jsp</result>
    <!-- 出现错误时转向的页面：回显 -->
    <result name="input">/WEB-INF/pages/regist.jsp</result>
</action> 
~~~~~~

针对所有动作进行验证：需要覆盖 `public void validate()`方法，
方法内部如果不满足要求，调用addFieldError填充信息.
~~~~~~
@SkipValidation//用在不需要验证的动作方法上
public String RegUserUI() {
    return SUCCESS;
}
 //对所有的动作方法进行校验
public void validate(){
//写你的校验代码ActionSupport里面有addFieldError()方法,把错误信息存起来.
    if(StringUtils.isEmpty(user.getUsername())){
        addFieldError("username", "请输入用户名");//向一个Map中存储错误消息。何时返回input视图，是由该Map中有无信息决定的。
    }
} 
~~~~~~

~~~~~~
//针对某个动作方法进行校验 public String regUser(){}
public void validateRegUser() {
    // 写你的校验代码
    if (user.getUsername() == null || user.getUsername().equals("")) {
        addFieldError("username", "请输入用户名");
    }
}
~~~~~~
## 声明式验证 ##
开发的时候，可以不写验证，后续增加声明式验证，
写配置文件即可，编码式验证需要开始就要写

struts2中已经内置一些验证器 `com.opensymphony.xwork2.validator.validators.default.xml`
~~~~~~
<!-- 打开xwork-core-2.*.jar包中的xwork-validator-1.*.dtd文件,复制表头 -->
<!DOCTYPE validators PUBLIC 
          "-//Apache Struts//XWork Validator 1.0//EN"
          "http://struts.apache.org/dtds/xwork-validator-1.0.2.dtd"> 
~~~~~~
如何使用内置验证器：
1. 对所有的动作方法都进行验证：在动作类相同的包中, 添加 `动作类名-validation.xml`使用。
2. 针对某些动作进行验证：动作类名-动作别名-validation.xml(动作别名指Struts.xml中的action的name))

~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE validators PUBLIC 
          "-//Apache Struts//XWork Validator 1.0//EN"
          "http://struts.apache.org/dtds/xwork-validator-1.0.2.dtd">
<validators>
    <!-- 针对字段的验证：方式一（建议使用）.在一个字段上加上多个验证规则-->
    <field name="username" >
    <!-- 不能为null或者""字符串, 默认会 trim -->
        <field-validator type="requiredstring"> 
            <message>用户名是必须的</message>
        </field-validator>
    </field>
     <!-- 针对字段的验证：方式二.
     <validator type="requiredstring">
     <!-- 校验器中有trim方法,当把这个方法设置为false,那么用户名前后可以有空格 -->
         <param name="trim">false</param>
         <param name="fieldName">username</param>
         <message>必须的用户名</message>
     </validator>-->
     <!-- 在使用配置文件的時候,只使用一种配置方式 -->
     <field name="password">
         <field-validator type="strongpassword">
             <message>密码不够强壮</message>
         </field-validator>
     </field>
</validators> 
~~~~~~

## 案例: 表单验证 ##

~~~~~~
<!-- 标签标示在此显示错误信息 -->
<!-- 里面的属性标示显示该属性的错误信息,如果不写就显示所有错误信息 -->
<s:fielderror fieldName="password" />
<s:form action="RegUser" namespace="/user">
    <!-- requiredLabel="true"表示在用户名旁边加 *(单纯加*,没有其他作用)； requiredPosition="left"表示 * 在左边 -->
    <s:textfield key="hello" label="用户名" name="username" requiredLabel="true" requiredPosition="left"></s:textfield>
    <s:textfield label="昵称" name="nick"></s:textfield>
    <s:password label="密码" name="password"></s:password>
</s:form> 
~~~~~~

~~~~~~
public class UserAction extends ActionSupport implements ModelDriven<User> {
    private User user = new User();
    private Map<String, String> hobbies = new HashMap<String, String>();
    public Map<String, String> getHobbies() {
        hobbies.put("eat", "吃饭");
        hobbies.put("sleep", "睡觉");
        hobbies.put("study", "学java");
        return hobbies;
    }
    public void setHobbies(Map<String, String> hobbies) {
        this.hobbies = hobbies;
    }
    public String RegUser() {
        try {
            System.out.println(user);
            // 调用service，保存数据
            System.out.println("调用后台service，保存数据到数据库中");
            return SUCCESS;
        } catch (Exception e) {
            e.printStackTrace();
            return ERROR;
        }
    }
    @SkipValidation//用在不需要验证的动作方法上
    public String RegUserUI() {
        return SUCCESS;
    }
     //对所有的动作方法进行校验
//     public void validate(){
//     //写你的校验代码
//         if(StringUtils.isEmpty(user.getUsername())){
//             addFieldError("username", "请输入用户名");//向一个Map中存储错误消息。何时返回input视图，是由该Map中有无信息决定的。
//         }
//     }
    //针对某个动作方法进行校验
//    public void validateRegUser() {
//        // 写你的校验代码
//        if (user.getUsername() == null || user.getUsername().equals("")) {
//            addFieldError("username", "请输入用户名");
//        }
//    }
    public User getModel() {
        return user;
    }
}
~~~~~~
使用声明式校验器
~~~~~~
<validators>
    <!-- 针对字段的验证：方式一（建议使用）.在一个字段上加上多个验证规则-->
    <field name="username" >
        <field-validator type="requiredstring">
            <message>用户名是必须的</message>
        </field-validator>
    </field>
     <!-- 针对字段的验证：方式二.
     <validator type="requiredstring">
     <!-- 校验器中有trim方法,当把这个方法设置为false,那么用户名前后可以有空格 -->
         <param name="trim">false</param>
         <param name="fieldName">username</param>
         <message>必须的用户名</message>
     </validator>-->
     <!-- 在使用配置文件的時候,只使用一种配置方式 -->
</validators>
~~~~~~


### 编写一个自定义校验器 ###
~~~~~~
public class StrongpasswordFieldValidate extends FieldValidatorSupport {
    //object就是当前执行的动作类的实例
    public void validate(Object object) throws ValidationException {
        String fieldName = getFieldName();
        Object value = getFieldValue(fieldName, object);
        //验证
        if(!(value instanceof String)){
            addFieldError(fieldName, object);
        }else{
            String s = (String)value;
            if(!isStrong(s)){
                addFieldError(fieldName, object);
            }
        }
    }
    //判断s是否强大
    private static final String GROUP1 = "abcdefghijklmnopqrstuvwxyz";
    private static final String GROUP2 = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private static final String GROUP3 = "0123456789";
    private boolean isStrong(String s) {
        boolean ok1 = false;
        boolean ok2 = false;
        boolean ok3 = false;
        int length = s.length();
        for(int i=0;i<length;i++){
            if(ok1&&ok2&&ok3)
                break;
            String character = s.substring(i,i+1);
            if(GROUP1.contains(character)){
                ok1 = true;
                continue;
            }
            if(GROUP2.contains(character)){
                ok2 = true;
                continue;
            }
            if(GROUP3.contains(character)){
                ok3 = true;
                continue;
            }
        }
        return ok1&&ok2&&ok3;
    }
}
~~~~~~
对自定义的校验器进行配置(validators.xml)
~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE validators PUBLIC
          "-//Apache Struts//XWork Validator Definition 1.0//EN"
          "http://struts.apache.org/dtds/xwork-validator-definition-1.0.dtd">
<validators>
    <validator name="strongpassword" class="com.itheima.validators.StrongpasswordFieldValidate"/>
</validators>
~~~~~~
使用自定义的校验器
~~~~~~
<field name="password">
    <field-validator type="strongpassword">
        <message>密码不够强壮</message>
    </field-validator>
</field>
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
<s:actionerrors></s:actionerrors>
~~~~~~

## 国际化显示中文信息 ##
~~~~~~
<!-- 引入token的资源文件 -->
<constant name="struts.custom.i18n.resources" value="io.zhpooer.token"> </constant>
~~~~~~
~~~~~~
## 默认的国际化文件在org.message.struts2.struts-message.properties
## io.zhpooer.token.properties
struts.messages.invalid.token=已经提交过了
struts.messages.error.content.type.not.allowed=不允许上传的文件类型
~~~~~~

# 文件上传 #
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
<!-- 文件上传的最大值, 默认大小为2M-->
<!-- <constant name="struts.multipart.maxSize" value="2097152"> </constant> -->
<action>
    <interceptor-ref name="defaultStack">
        <param name="fileUpload.maximumSize">2097152</param>
        <param name="fileUpload.allowedExtensions">txt,doc,pdf </param>
        <!-- 可以在web.xml中查看, mime类型 -->
        <param name="fileUpload.allowedTypes">application/msword </param>
    </interceptor-ref>
</action>
~~~~~~

# 文件的下载 #

~~~~~~
/*
<action>
    <result type="stream">
        <param name="inputName"> inputStream </param>
        <param name="contentDisposition">
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
        String encoded = URLEnocoder.encode(fileName, ""utf-8);
        ActionContext.getContext().put("fileName", encoded);
        inputStream = new FileInputStream(filePath + fileName);
    }
}
~~~~~~

# 类型转换 #
当数据类型及数据转换出现错误信息时, 框架自动会转向名称为input的结果集
~~~~~~
<!-- 输入 myname, mypassword -->
<s:textfield name="user"> </s:textfield>
<s:checkbox list="{'aa', 'bb'}" name="checkbox"/>
~~~~~~

~~~~~~
public UserConverter extends StrutsTypeConverter {
    // 从页面到Actioin的转换
    public Object convertFromString(Map context, String[] values, Class clazz) {
        String[] tmp = values[0].split(",");
        User user = new User();
        user.setUsername(tmp[0]);
        user.setPassword(tmp[1]);
        return user;
    }
    public String convertToString(Map context, Object o) {
        if(o instanceof User){
            return o.toString();
        }
        return null;
    }
}
public class ConverterAction {
    @BeanProperty private User user;
}
public class User {
    @BeanProperty private String userName;
    @BeanProperty pirvate String password;
}
~~~~~~
局部类型转换器：要转换的属性所在的类相同的包下, 建立类名-conversion.properties的配置文件
~~~~~~
## 转换的字段名称=验证类全面
birthday=
~~~~~~
全局类型转换器设置: 在根目录下建立 xwork-conversion.preperties
~~~~~~
## 转换后的类型=转换器
User=UserConverter
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
