title: 传智播客day29-struts请求数据处理
date: 2014-05-14 09:01:10
tags:
- 传智播客
- struts
---

实际场景： 表现层，获得请求参数，将请求参数封装到JavaBean对象，
传递JavaBean对象 给业务层

# Action 如何接受请求参数 #

struts2 和 MVC 定义关系
* StrutsPrepareAndExecuteFilter : 控制器
* JSP : 视图
* Action : 可以作为模型，也可以是控制器 

## 属性驱动一 ##
Action 本身作为model对象，通过成员setter封装

~~~~~~
// 用户名  <input type="text" name="username" /> <br/>
public class RegistAction1 extends ActionSupport {
	private String username;
	public void setUsername(String username) {
		this.username = username;
	}
}
~~~~~~

问题一： Action封装数据，会不会有线程问题 ？ 
* struts2 Action 是多实例 ，为了在Action封装数据(struts1 Action 是单例的)
  
问题二：在使用第一种数据封装方式，数据封装到Action属性中，不可能将Action对象传递给 业务层
* 需要再定义单独JavaBean ，将Action属性封装到 JavaBean 

## 属性驱动二 ##
创建独立model对象，页面通过ognl表达式封装
~~~~~~
//	用户名  <input type="text" name="user.username" /> <br/> 
public class RegistAction2 extends ActionSupport {
    private User user;
    public void setUser(User user) {
	    this.user = user;
	}
	public User getUser() {
	    return user;
	}
}
~~~~~~

问题： 谁来完成的参数封装 
`<interceptor name="params" class="com.opensymphony.xwork2.interceptor.ParametersInterceptor"/>`

## 模型驱动 ##
使用ModelDriven接口，对请求数据进行封装(主流)
~~~~~~
// 用户名  <input type="text" name="username" /> 
public class RegistAction3 extends ActionSupport implements ModelDriven<User> {
	private User user = new User(); // 必须手动实例化
	public User getModel() {
		return user;
	}
}
~~~~~~
struts2 有很多围绕模型驱动的特性, 为模型驱动提供了更多特性
`<interceptor name="modelDriven" class="com.opensymphony.xwork2.interceptor.ModelDrivenInterceptor"/> `

模型驱动只能在Action中指定一个model对象，属性驱动二可以在Action中定义多个model对象 

# 封装复杂类型参数 #

## Collection 对象 ##
~~~~~~
 // <input type="text" name="products[0].name" /><br/>
public class ProductAction extends ActionSupport {
	private List<Product> products;
	public List<Product> getProducts() {
		return products;
	}
    public void setProducts(List<Product> products) {
		this.products = products;
	}
}
~~~~~~

## Map 对象 ##

~~~~~~
// <input type="text" name="map['one'].name" /><br/>  =======  one是map的键值
public class ProductAction2 extends ActionSupport {
	private Map<String, Product> map;
    public Map<String, Product> getMap() {
		return map;
	}
    public void setMap(Map<String, Product> map) {
		this.map = map;
	}
}

~~~~~~

# 类型转换 #
struts2 内部提供大量类型转换器，用来完成数据类型转换问题
* boolean 和 Boolean
* char和 Character
* int 和 Integer
* long 和 Long
* float 和 Float
* double 和 Double
* Date 可以接收 yyyy-MM-dd格式字符串
* 数组  可以将多个同名参数，转换到数组中
* 集合  支持将数据保存到 List 或者 Map 集合

## 自定义转换器 ##
1. 实现TypeConverter接口
`convertValue(java.util.Map<java.lang.String,java.lang.Object> context, java.lang.Object target, java.lang.reflect.Member member, java.lang.String propertyName, java.lang.Object value, java.lang.Class toType)`
2. 继承 DefaultTypeConverter
`convertValue(java.util.Map<java.lang.String,java.lang.Object> context, java.lang.Object value, java.lang.Class toType)`
3. 继承 StrutsTypeConverter
`convertFromString(java.util.Map context, java.lang.String[] values, java.lang.Class toClass)`
`convertToString(java.util.Map context, java.lang.Object o)`

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
## 转换的字段名称=验证类全名
birthday=
~~~~~~
全局类型转换器设置: 在根目录下建立 xwork-conversion.preperties
~~~~~~
## 转换后的类型=转换器
User=UserConverter
~~~~~~

## 类型转换中错误处理 ##

通过分析拦截器作用，得知当类型转换出错时，
自动跳转input视图 ，在input视图页面中 `<s:fieldError/>` 显示错误信息

在Action所在包中，创建 ActionName.properties，
在局部资源文件中配置提示信息 ：
~~~~~~
invalid.fieldvalue.属性名 = 错误信息
~~~~~~


# 请求数据有效性校验 #

1. 客户端数据校验: 通过JavaScript 完成校验()改善用户体验，使用户减少出错)
2. 服务器数据校验, 使用框架内置校验功能(struts2 内置校验功能)


## 代码校验 ##
在服务器端通过编写java代码，完成数据校验

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

## 配置校验 ##
XML配置校验(主流) 和 注解配置校验

开发的时候，可以不写验证，后续增加声明式验证，写配置文件即可，编码式验证需要开始就要写


struts2中已经内置一些验证器 `com.opensymphony.xwork2.validator.validators.default.xml`
* required (必填校验器,要求被校验的属性值不能为null)
* requiredstring (必填字符串校验器,要求被校验的属性值不能为null，并且长度大于0,默认情况下会对字符串去前后空格)
* stringlength (字符串长度校验器，要求被校验的属性值必须在指定的范围内，否则校验失败,minLength参数指定最小长度，maxLength参数指定最大长度，trim参数指定校验field之前是否去除字符串前后的空格)
* regex (正则表达式校验器，检查被校验的属性值是否匹配一个正则表达式，expression参数指定正则表达式，caseSensitive参数指定进行正则表达式匹配时，是否区分大小写,默认值为true)
* int(整数校验器，要求field的整数值必须在指定范围内，min指定最小值，max指定最大值)
* double(双精度浮点数校验器,要求field的双精度浮点数必须在指定范围内,min指定最小值,max指定最大值)
* fieldexpression (字段OGNL表达式校验器,要求field满足一个ognl表达式，expression参数指定ognl表达式,该逻辑表达式基于ValueStack进行求值,返回true时校验通过，否则不通过)
* email(邮件地址校验器，要求如果被校验的属性值非空，则必须是合法的邮件地址)
* url(网址校验器,要求如果被校验的属性值非空,则必须是合法的url地址)
* date(日期校验器,要求field的日期值必须在指定范围内,min指定最小值,max指定最大值)

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
     <!-- 针对字段的验证：方式二 -->
     <validator type="requiredstring">
     <!-- 校验器中有trim方法,当把这个方法设置为false,那么用户名前后可以有空格 -->
         <param name="trim">false</param>
         <param name="fieldName">username</param>
         <message>必须的用户名</message>
     </validator>
</validators> 
~~~~~~

~~~~~~
<!-- required  必填校验器 -->
<field-validator type="required">
       <message>性别不能为空!</message>
</field-validator>

<!-- requiredstring  必填字符串校验器 -->
<field-validator type="requiredstring">
       <param name="trim">true</param>
       <message>用户名不能为空!</message>
</field-validator>

<!-- stringlength：字符串长度校验器 -->
<field-validator type="stringlength">
	<param name="maxLength">10</param>
	<param name="minLength">2</param>
	<param name="trim">true</param>
	<message><![CDATA[产品名称应在2-10个字符之间]]></message>
</field-validator>

<!-- int：整数校验器 -->
<field-validator type="int">
	<param name="min">1</param>
	<param name="max">150</param>
	<message>年龄必须在1-150之间</message>
</field-validator>

<!-- date: 日期校验器 -->
<field-validator type="date">
	<param name="min">1900-01-01</param>
	<param name="max">2050-02-21</param>
	<message>生日必须在${min}到${max}之间</message>
</field-validator>

<!-- url:  网络路径校验器 -->
<field-validator type="url">
	<message>传智播客的主页地址必须是一个有效网址</message>
</field-validator>

<!-- email：邮件地址校验器 -->
<field-validator type="email">
	<message>电子邮件地址无效</message>
</field-validator>

<!-- regex：正则表达式校验器 -->
<field-validator type="regex">
     <param name="expression"><![CDATA[^13\d{9}$]]></param>
     <message>手机号格式不正确!</message>
</field-validator>

<!-- fieldexpression : 字段表达式校验 -->
<field-validator type="fieldexpression">
       <param name="expression"><![CDATA[(password==repassword)]]></param>
       <message>两次密码输入不一致</message>
</field-validator>
~~~~~~

## 编写一个自定义校验器 ##
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

# 国际化信息显示 #
国际化原理同一款软件 可以为不同用户，提供不同语言界面

需要一个语言资源包(很多properties文件，每个properties文件 针对一个国家或者语言 ，通过java程序根据来访者国家语言，自动读取不同properties文件)

## 资源包编写 ##
properties文件命名: 基本名称_语言（小写）_国家（大写）.properties
~~~~~~
// messages_zh_CN.properties 中国中文
// messages_en_US.properties 美国英文
ResourceBundle bundle = ResourceBundle.getBundle("messages", Locale.US);
~~~~~~




## struts2 框架国际化配置 ##
### 全局国际化信息文件 ###
所有Action都可以使用, 最常用
* properties文件可以在任何包中
* 需要在struts.xml 中配置全局信息文件位置

struts.xml
~~~~~~
<!-- messages.properties 在src根目录 -->
<constant name="struts.custom.i18n.resources" value="messages"></constant>
<!-- messages.properties 在 cn.itcast.resources 包 -->
<constant name="struts.custom.i18n.resources" value="cn.itcast.resources.messages"></constant>   
~~~~~~

国际化信息
	* 在Action中使用: `this.getText("msg");`
	* 在jsp中使用: `<s:text name="msg" />`
	* 在配置文件中(校验xml): `<message key="agemsg"></message>`

### Action范围信息文件
只能在某个Action中使用
无需配置, 数据只能在对应Action中使用，在Action类所在包 创建 Action类名.properties

### package范围信息文件
package中所有Action都可以使用

无需配置, 数据对包(包括子包)中的所有Action 都有效 ， 在包中创建 package.properties

### 临时信息文件
主要在jsp中 引入国际化信息

在jsp指定读取 哪个properties文件
~~~~~~
<s:i18n name="cn.itcast.struts2.demo7.package">
    <s:text name="customer"></s:text>
</s:i18n>
~~~~~~

### 动态消息文本 
向信息中传递参数 
~~~~~~
// MessageFormat 动态消息文本, required = {0} 错了
this.getText("required", new String[] { "用户名" });
~~~~~~
