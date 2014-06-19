title: 传智播客day46-JQuery EasyUI
date: 2014-06-12 09:03:59
tags:
- 传智播客
---

# 简介 #
j2ee企业开发, 服务端界面没有门户网站界面要求那么复杂,
没有要求那么美观
* 种类一: 门户网站, 大量用户使用, 页面内信息非常多, 没有很高界面设计要求
* 种类二, 企业内网应用系统, 用户很少,页面内信息量不多, 页面设计要求不高

针对内网系统, 界面开发工作由java工程师来完成. java工程师设计这些系统,
基于页面设计框架技术完成(Flex技术, ExtJS, EasyUI), 这些框架不需要
关注页面的显示部分

# 目录结构 #
* Locale: 存放国际化js文件, 如果不知道 locale, 默认提示信息是英文, 导入
`easyui-lang-zh_CN.js`, 将提示信息变为中文.
* plugins: 插件目录
* themes: 内置了五种主题, 每种主题内部的文件结构是相同的,
每种主题都提供了`easyui.css`,
 *  icon.css + icons 文件夹, 用于图标显示(与主题无关)
* jquery.min.js, jquery框架js文件
* easyloader.js, 用于提供 easyui 框架核心加载器, 想使用 plugin,
可以用加载器去加载插件
* easyui.min.js, easyloader.js + 36个plugin

# 使用maven构建 maveneasyui #

新建JSP, 导入 `jquery.js`, `easyui.min.js`,
`easyui-lang-zh_CN.js`, `icon.css`, `easyui.css`

# 案例: 系统主界面设计 #

JavaEE主界面特点:后台应用系统, 填满屏幕: 顶部logo, 底部copyrights,中部 menu+content

门户网站: 高度是屏幕的两到三倍

## layout插件 ##
layout插件, 完成页面布局

~~~~~~
<!-- 可以对div和body使用 -->
<!-- 添加 class="easyui-layout" -->
<body class="easyui-layout">
<!-- 在layout中, 由东西南北中五个区域 -->
<!-- 过region属性，指定div属于哪个区域 ，region属性属于easyui对象属性 -->
<!-- 可以对东西南北四个区域 设置高、宽 -->
<!-- 剩下的就是center(center区域是必须的，其它区域可以省略) -->
<div region="north" >北部区域</div>
<!-- 可以通过title指定标题 -->
<div data-option="region:south'" style="height: 100px;">南区域</div>
<div data-option="region:'west', title:'员工管理系统'">西部区域</div>
<div data-option="region:east'">东部区域</div>
<div data-option="region:center'">中部区域</div>
</body>
~~~~~~

## 折叠面板(accordion) ##

~~~~~~
<!-- fit:true , 占满父容器窗口  -->
<div class="easyui-accordion" data-options="fit:true">
    <!-- 对每个折叠面板，设置title属性 ，用于面部标题  -->
    <div data-options="title:'基本菜单'"> 基本面板 </div>
    <!-- 可以对任何一个面部，添加iconCls属性，指定面板标题的图标, 自动寻找iconCss -->
    <div data-options="title:'系统菜单', iconCls:'icon-search'"> 系统面板 </div>
</div>
~~~~~~

## 选项卡插件(tabs) ##

~~~~~~
<!-- 选项卡面板 -->
<div id="mytabs" calss="easyui-tabs" data-options="fit:true">
    <!-- 内部提供多个面板 -->
    <!-- closeable: true, 设置是否可以被关闭 -->
    <div data-options="title:'百度', closable: true">面板1</div>
    <div data-options="title:'新浪'">面板2</div>
</div>
~~~~~~

~~~~~~
$("baduClick").click(function(){
    // 调用选项卡面板 add 方法
    $("#mytabs").tabs('add',{
        title:'百度',
        content:'',
        closable: true
    });
});'
~~~~~~

## ztree 树形菜单的制作 ##

jquery 树形结构第三方插件`ztree`

ztree功能介绍
* Core 核心功能，负责树显示
* Excheck 扩展勾选功能
* Exedit 扩展可编辑
* Exhide 扩展隐藏

`All = core + check + edit + hide` 只需要在项目导入 all.js 使用ztree所有功能

### 导入ztree ###
~~~~~~
<!-- ztree -->
<script type="text/javascript" src="jquery.ztree.all-3.5.js"></script>
<link href="zTreeStyle.css" type="text/css" rel="stylesheet"/>
~~~~~~

### 制作基本菜单 ###
基于标准数据菜单: 
~~~~~~
<ui class="ztree" id="standardtree">
</ui>
<script type="text/javascript">
// 标准数据
// 页面加载后, 进行ztree配置
var standardsettings = {}; // 标准数据树, 不进行任何配置
// 准备节点数据
var standardnodes = [
   {name: "节点一"},{name:"节点二", children:[ {name:"节点二一"} ]}
]
// 初始化ztree
$.fn.zTree.init($("#standardTree"), standardsettings, standardnodes);
</script>

~~~~~~
Ztree 标签数据树缺点：和easyui ztree一样, 通过children指定子节点,
如果子节点层次过多, 不利于编写和维护.

基于简单数据树
~~~~~~
<ui class="ztree" id="simpleTree">
</ui>
<script type="text/javascript">
// 标准数据
// 页面加载后, 激活简单ztree配置
var simplesettings = {
    data:{
        simpleData{
            enable:true
        },
        // 对所有节点增加单击时间
        callback:{
           onClick:function(event, treeId, treeNode, clickFlag){
               // treeNode 就是当前点击的节点 treeNode.curl
               if($("mytabls").tabs('exist', treeNode.name)){
                   // 如果存在, 切换到
                   $('#mytabs').tabs('select', treeNode.name);
                   
               } else {
                   // 如果不存在, 新增
                   $('#mytabs').tabs('add', {
                       title: treeNode.name,
                       content: treeNode.curl
                       '<div>
                           <iframe src="treeNode.curl">
                           </iframe>
                       </div>'
                   });
               }
           }
        }
    }
};
// 准备节点数据
var simplenodes = [
   // id是当前节点编号, pid是父节点编号
   {name: "节点一", id:1, pId:0},
   // icon: 设置图标呢, url: 打开新窗口
   {name:"节点二", id: 2, pId:0,
    icon: "*.png",
    url:"http://www.baidu.com"},
    // 自定义属性curl
   {name:"节点二一", id: 3, pId:2
    curl:"http://www.baidu.com"}
]
// 初始化ztree
$.fn.zTree.init($("#simpleTree"), simplesettings, simplenodes);
</script>
~~~~~~


# Maven完成ssh框架整合(注解方式) #

非注解 + jpa + struts2 convention 插件

	导入 servlet, jsp(provided范围)
	导入 junit4(test范围)
	导入 Spring
        context, aspects, orm, web, test
	导入 hibernate
		hibernate-core, jpa
	导入struts2
		Struts2-core, struts2-spring, struts2-json, struts2-convention
    导入 slf4j-log4j
	导入 c3p0, mysql驱动

## 配置文件  ##
web.xml配置, spring核心监听器,
struts2 核心过滤器, OpenSessionInViewFilter 延迟加载问题,
log4j.properties

<!-- applicationContext.xml -->
~~~~~~
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:cache="http://www.springframework.org/schema/cache"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx.xsd
	http://www.springframework.org/schema/cache 
	http://www.springframework.org/schema/cache/spring-cache.xsd">
	<!-- 引入外部 properties 文件 -->
	<context:property-placeholder location="classpath:jdbc.properties"/>
	<!-- 数据库连接池  -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.driver}"></property>
		<property name="jdbcUrl" value="${jdbc.url}"></property>
		<property name="user" value="${jdbc.username}"></property>
		<property name="password" value="${jdbc.password}"></property>
	</bean>
	<bean id="sessionFactory" 
		class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">
		<!-- 引用数据库连接池 -->
		<property name="dataSource" ref="dataSource"></property>
		<!-- 配置hibernate其它属性 -->
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">true</prop>
				<prop key="hibernate.hbm2ddl.auto">update</prop>
			</props>
		</property>
		<!-- 自动扫描注解类 -->
		<property name="packagesToScan">
			<list>
				<value>cn.itcast.domain</value>
			</list>
		</property>
	</bean>
	<bean id="hibernateTemplate" class="org.springframework.orm.hibernate3.HibernateTemplate">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>
	<!-- 事务管理 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>
	<!-- 注解管理事务 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>
	<!-- 扫描组件 -->
	<context:component-scan base-package="cn.itcast.web, cn.itcast.service, cn.itcast.dao" />
<beans>
~~~~~~

## 实体类 ##

~~~~~~
@Entity
@Table(name="department", catalog="easyui")
public class Department {
    @Id
    @GeneratedValue(strategy=Generation.IDENTITY)
    private Integer id;
    private String name;
    @OneToMany(mappedBy="department", targetEntity=Employee.class)
    private Set<Employee> emloyeees = new HashSet<Employee>();
}
@Entity
@Table(name="employee", catalog="easyui")
@NamedQueries(value={@NamedQquery(name="Employee.login",
              query="from Employee where name=? and password=?")})
public class Empoyee{
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private String password;
    private Integer age;
    private Date birthday;
    @ManyToOne(targetEntity=Department.class)
    privat Department department;
}
~~~~~~

# 表现层 #
~~~~~~
<s:actionerros/>
<form action="${contextPath}/login.action" method="post">
    用户名
    <input type="text" name="name"/>
    <input type="password" name="password"/>
    <input type="submit"/>
</form>
~~~~~~

~~~~~~
@ParentPackage("struts-default")
@Namespace("/")
@Controller
@Scope("prototype")
public class LoginAction extends ActionSupport implements ModelDriver<Employee>{
    @Resource
    private EmployeeService employeeService;
    
    private Employee employee = new Emplyee();
    @Action(value="login", results={@Result(name="success", location="/index.jsp")
    @Result(name="input", location="/login.jsp")})
    public String login(){
        log("用户登陆");
        Employee loginUser = employeeService.login(employee);
        if(loginUse = null) {
            this.addActionError("用户名或密码错误");
            return INPUT;
        } else {
           session.setAttribute("user", loginUser);
           return SUCCESS;
        }
    }
}
~~~~~~

# 业务层和数据层 #
~~~~~~
@Repository("employeeDao")
public classs EmployeeDao{
    @Resource(name="hibernateTemplate")
    private HibernateTemplate template;
    private Employee login(Employee employee) {
       List employees = template.findByNamedQuery("Employee.login",
             employee.getName, employee.getPassword());
       retirn employee.isEmpty()?:;
    }
}

@Service("emplyeeService")
@Transactional
public class EmployeeService{
   @Resource(name="employeeDao")
   private EmployeeDao employDao;
   public Employee login(Employee user) {
      empoyeeDao.login(user);
   }
}
~~~~~~

# message 消息框 #

easyUI提供了5种消息框
~~~~~~
// 警告框
$.messager.alert("标题", "内容", "warning|error|info|questiong(图标)")
// 确认框
$.messager.confirm("标题", "内容", function(r){ if(r){} else {} });
// 输入框
$.messager.prompt("标题", "内容", function(input){ log(input)});
// 进度条
$.messager.progress({interval: 1000});
window.setTimeout('$.messager.progerss("close");', 5000);
// 右下角提示框
$.messager.show({
    title:"",
    msg:"",
    timeout: 5000,
});
~~~~~~

# 用户退出功能 #

通过 menubutton 制作下拉菜单
~~~~~~
<!-- 下拉菜单 -->
<!-- 提供menu属性, 指向下拉菜单项的div的id -->
<a href="javascript:void(0)" class="easyui-menubutton" data-options="menu:'#menu'">控制面板</a>
<div id="menu">
    <div data-options="iconCls:'icon-edit'">修改密码 </div>
    <!--分割线 -->
    <div class="menu-sep"></div>
    <div onclick="logout();">用户退出</div>
</div>
<script type="text/javascript">
function logout(){
    $messager.confirm("确认窗口", "确认退出吗", function(isConfirm){
        if(isConfirm) {
            // 用户选择跳转
            location.href = "${contextPath}/logout.action"
        }
    })
}
</script>
~~~~~~

~~~~~~
@Action(value="logout", result={@Result(name="success", location="/index.jsp")})
public String logout(){
    getSession().invalidate()
    return SUCESS;
}
~~~~~~
