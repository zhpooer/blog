title: 传智播客day30-struts提高
date: 2014-05-15 09:09:51
tags:
- 传智播客
- struts
---

# 拦截器 #

拦截器的目的: 如果一个业务逻辑方法中涉及到的逻辑相当复杂,
可以把这些业务分离开, 如 启动日志, 权限检查, 文件上传, 保存用户,
把这四方面全面分开, 实现松耦合

意义: 把一些和业务逻辑没有关联的代码放入拦截器, 以实现业务逻辑和其他代码的松耦合

## 拦截器初探 ##
~~~~~~
public class InterceptorAction extend ActionSupport {
    public String saveUser() {
        ActionContext().getContext().put("message", "userSaved");
        return "privilege"
    }
}
// 拦截器
public class PrivilegeInterceptor implements Interceptor {
    public void destroy(){}
    public void init(){}
    public String intercept(ActionInvocation action) {
         String username = ServletActionContext.getRequest().getParameter("username")
         if("admin".equals(username)) {
             return action.invoke();
         } else {
             ActionContext().getContext().put("message", "权限不足");
             return "privilege"
         }
    }
}
~~~~~~
声明拦截器
~~~~~~
<package>
    <interceptors>
        <interceptor name="privilege" class="cn..**"> </interceptor>
        <!-- 声明一个拦截器栈 -->
        <interceptor-stack name="privilegeStack">
            <!-- 引用默认拦截器栈 -->
            <interceptor-ref name="defaultStack"> </interceptor-ref>
            <!-- 引用自己的连接器 -->
            <interceptor-ref name="privilege"> </interceptor-ref>
        </interceptor-stack>
    </interceptors>
    <!-- 让struts 执行声明的拦截器栈, 和拦截器 -->
    <default-interceptor-ref name="privilegeStack"/>
</package>
~~~~~~

## 概念 ##
1. 拦截器, 实现 Interceptor 接口的一个类
2. 拦截器栈, 把很多个拦截器集中在一起
3. struts有一个默认拦截器栈, 该栈在 `struts-default.xml` 中声明, 结构为: 
~~~~~~
<package name="struts-default">
    <interceptors>
        <!-- 声明拦截器 -->
        <interceptor name="name1" class=""> </interceptor>
        <interceptor name="name2" class=""> </interceptor>
        <!-- 声明拦截器栈 -->
        <interceptor-stack name="defaultStack">
            <!-- 引用拦截器栈 -->
            <interceptor-ref name="name1"> </interceptor-ref>
            <interceptor-ref name="name2"> </interceptor-ref>
        </interceptor-stack>
    </interceptors>
    <!-- 让struts 内部执行默认的拦截器栈, 和拦截器 -->
    <default-interceptor-ref name="defaultStack"/>
</package>
~~~~~~

## 属性驱动 ##
在Action中声明一些属性, 这些属性能够获取表单中的值

在浏览器提交一个url请求时, 先创建一个action, 并且把action放入对象栈中,
这个时候action的属性会出现在对象栈中,
然后经过一个拦截器ParametersInterceptor拦截器,
1. 获取页面上的表单中的name和value值
2. 把上述name和value的值封装成一个map
3. 根据 valueStack.setValue(name, value); 把页面上的值放到页面栈中

~~~~~~
<!-- form.jsp -->
<!-- 提交表单后, action重新返回form.jsp, username会被设为 "aaa"
     也可以回显表单元素如 property.username 和 property.password
-->
<form action="{contextPath}/PropertyDriveAction_actionMethod">
    <s:textfield name="username"> </s:textfield>
    <s:password name="password"> </s:password>
    <s:textfield name="phone"> </s:textfield>
    
    <s:property name="username"> </s:property>
    <s:property name="password"> </s:property>
    <s:property name="phone"> </s:property>
</form>
~~~~~~
~~~~~~
public class PropertyDiveActionAction {
    @BeanProperty private String useranme;
    @BeanProperty private String password;
    @BeanProperty private String phone;
    public String actionMethod(){
       println(username); // 可以得到 表单元素
       println(password);
       println(phone);
       ActontionContext().getContext().getValueStack().setParameter("username", "aaa");
       return "form.jsp";
    }
}
~~~~~~

## 模型驱动 ##
1. 创建一个javabean, javabean中的属性和页面中表单的name属性的内容保持一致
2. 在Action中声明一个接口 ModelDriven<Person>
3. 在action中声明一个属性 `model = new Person`
4. 在action中重写 `getModel(){return model;}`, 返回 model 对象

原理, 经历两个拦截器
1. ModelDrivenIntercetper
  1. 得到action
  2. 由action强制转化成ModelDriver
  3. 由`ModelDriver.getModel()` 获取模型对象
  4. 把模型对象放入栈顶
2. ParameterInterceptor, 把form表单的数据封装到相应的对象栈中的属性

~~~~~~
public class Person {
    private String name;
    private String password;
    private String phone;
}
public ModelDriverAction extends ActionSuppoet implements ModelDriven<Person> {
    private Person model = new Person();
    public Person getModel() {
       return this.model;
    }
    public String testModel(){
        println(model.getName()); // 可以打印出
        println(model.getPassword());
        println(model.getPhone());
        return "";
    }
}
~~~~~~

## 拦截器执行 ##
执行顺序: 按照拦截器的声明顺序, 从上到下执行, 执行完拦截器以后, 再执行 `Action`
~~~~~~
<interceptor-stack name="privilegeStack">
<!-- 从上到下执行 -->
    <interceptor-ref name="defaultStack"> </interceptor-ref>
    <interceptor-ref name="privilege"> </interceptor-ref>
</interceptor-stack>
~~~~~~

一个 pacakge 中可以有
1. 结果集
2. 拦截器
3. action

可以通过继承package, 来复用 `interceptor` 配置信息
~~~~~~
<!-- 声明intercept包,在内部使用公用拦截器 -->
<package name="intercept" extends="struts-default"> </package>
<!-- 复用intercept拦截-->
<package extends="intercept"> </package>
<package extends="struts-default"> </package>
~~~~~~

# struts2 校验(validate)机制 #
~~~~~~
public class ValidateAction extends ActionSupport {
    @BeanProperty private String username;
    @BeanProperty private String password;

    public String testValidate(){
        return "form.jsp"
    }

    @Override
    public void validate(){
        if("".equals(this.getUsername()) ||
           "".equals) {
           // 方式一:
           // 会在对象栈中加入数组: actionErrors: []
           // 通过设置 actionerror 
           this.addActioinError("用户名不能为空");
           // 方式二:
           // 设置 fielderror
           this.addFieldError("username", "用户名不能为空");
           this.addFieldError("password", "密码不能为空");
           this.addFieldError("password", "长度不能小于6"); // 不会覆盖前面的错误
        }
    }

    // 验证方式二, 框架会自动执行 validateXXX() 方法
    public void validateUserName(){}
    public void validatePassword(){}
}
~~~~~~

~~~~~~
<!-- form.jsp -->
<s:form>
    <s:actionerror/>   <!-- 用于回显, 显示: 用户名不能为空 -->
    <s:textfield name="username"> </s:textfield> <s:fielderror fieldname="username"> </s:fielderro>
    <s:password name="password"> </s:password>  <s:fielderror fieldname="password"> </s:fielderror>
</s:form>
~~~~~~

# Tip #
属性驱动了利用了对象栈, 可以利用 `valueStack.setValue/setParameter`
方法给对象中的属性赋值
~~~~~~
person = Person{name=hello};
valueStack.push(person);
valueStack.setValue("name", "hello2");
assert(person.name == "hello2")
~~~~~~
