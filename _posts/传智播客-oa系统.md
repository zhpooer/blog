title: 传智播客-OA系统
date: 2014-06-20 09:02:49
tags:
- 传智播客
---

# 入门 #

OA: Office Automation, 办公自动化  
CRM: 客户关系管理系统  
ERP: 企业资源管理系统  
BBS: 论坛系统  
CMS: 内容管理系统

软件开发流程
1. 需求调研, 形成调研文档
2. 分析需求, 形成需求分析文档
3. 设计(概要设计, 详细设计), 形成设计文档
4. 编码
5. 测试, 测试用例, 测试计划, 测试报告, 性能测试, 压力测试(HP LoadRunner)
6. 运维

## 整体设计 ##
分层: 表现层, 业务层, 持久层

技术: Struts + Spring3.2 + Hibernate3.6 + Jquery1.8 + Ajax

代码规范:
* 常量字母都大写, 单词之间使用`_`隔开, 例如 `DEFAULT_PAGE_SIZE`
* 注释, 在代码中加入适当注释, 说明步骤, 与说明非简单逻辑
* 空行, 在代码中加入适当空行, 增加可读性
* 要格式化代码, 一个Java文件中代码不要过多, 一个方法中的代码不要过多

约定:
* 文件中都采用 UTF-8 编码, JDK版本, 编译环境统一
* 实体的主键属性的类型使用`Long`类型

# 搭建开发环境 #

1. 创建Web项目`itcastOA`
2. 导入jar包(struts2, spring3.2, hibernate3.6, 数据库驱动, c3p0)
3. 配置 web.xml
~~~~~~

<filter>
    <filter-name> openSessionInView</filter-name>
    <filter-class>OpenSessionInView</filter-class>
</filter>

<filter>
    <filter-name> struts</filter-name>
    <filter-class>StrtusPrepareAndExcutorFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>openSessionInView </filter-name>
    <url-pattern> /* </url-pattern>
</filter-mapping>

<filter-mapping>
    <filter-name>struts2 </filter-name>
    <url-pattern> /* </url-pattern>
</filter-mapping>

<!-- spring监听器 -->
<listener>
   <listener-class> org.spring.web.context.ContextLoaderListener </listener-class>
</listener>
<context-param>
    <param-name> contextConfigLocation</param-name>
    <param-value>classpath:beans.xml</param-value>
</context-param>
~~~~~~
4. 加入 `struts.xml`, `beans.xml`
~~~~~~
<beans>
    <!-- 读取属性文件的配置  -->
    <context:property-placeholder locaction="classpath:jdbc.properties"/>

    <!-- dataSource  -->
    <bean id="dataSrouce" class="com.mchange.ComboPooledDataSource">
        <property name="driverClass"></property>
        <property name="jdbcUrl"></property>
        <property name="user"></property>
        <property name="password"></property>
        <property name="initialPoolSize"></property>
        <property name="maxPoolSize"></property>
        <property name="minPoolSize"></property>
    </bean>
    <!-- 本地会话工厂bean, LocalSessonFactoryBean -->
    <bean id="sessionFactory" class="org.spring.hibernate.LocalSessionFactory">
        <property name="dataSource" ref="dataSource"> </property>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>
            </props>
        </property>
        <!-- hbm的映射文件 -->
        <property name="mappingDirectoryLocations" >
            <list>
                <value>classpath:cn/itcast/or/domain </value>
            </list>
        </property>
    </bean>
    <!-- 事务管理器, HibernateTransactonManager -->
    <bean id="transactionManager" class="HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"></property>
    </bean>
    <!-- 事务通知, AOP切面, 事务注解驱动, 组件扫描 -->
    <tx:advice id="oaAdvice" transaction-manager="transactionManager">
        <!-- 事务属性 -->
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIERD" isolation="DEFAULT">
            <tx:method name="update*" propagation="REQUIERD" isolation="DEFAULT">
            <tx:method name="delete*" propagation="REQUIERD" isolation="DEFAULT">
            <tx:method name="find*" read-only="true">
            </tx:method>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <!-- 切点  -->
        <aop:pointcut expression="execution(* cn.itcast.oa.service..*Service.*())">
        </aop:pointcut>
        <!-- 切面 -->
        <aop:adviser advice-ref="oaAdvice" pointcut-ref="oaPC"></aop:adviser>
    </aop:config>
    <!-- 组建扫描-->
    <context:compoent-scan bas-package="cn.itcast.oa"></context:compoent-scan>
    <!-- 支持注解  -->
    <context:annotation-config></context:annotation-config>
    <!-- 事务注解驱动 -->
    <tx:annotation-driver transaction-manager="transactionManager"></tx:annotation-driver>
</beans>
~~~~~~
5. 创建数据库
~~~~~~
create database itcastOA default character set utf8;
create user 'xiaoming' identified by '123';  -- 创建一个普通用户
grant all on itcast_oa.* to itcast@localhost identified by "123";
~~~~~~
6. 数据连接参数
~~~~~~
;; TODO
driverClass=
jdbcUrl=
user=
password=
initPoolSize=
maxPoolSize=
minPoolSize
~~~~~~
7. 创建包结构
        cn.itcast.oa.domain
        cn.itcast.oa.dao
        cn.itcast.oa.service
        cn.itcast.oa.web
        cn.itcast.oa.web.action
        cn.itcast.oa.web.filter
        cn.itcast.oa.web.interceptor
        cn.itcast.oa.web.listener
        cn.itcast.oa.utils
        cn.itcast.oa.base
8. 抽取通用DAO
~~~~~~
package cn.itcast.oa.base;
// 抽取通用DAO
public interface IBaseDao<T> {
    public void save(T entity);
    public void delete(Long id);
    public void update(T entity)
    public T findById(Long id);
    public List<T> findAll();
    public List<T> findByIds(Long[] ids);
}
// class UserDaoImpl extends BaseDaoImpl<User> implements IUserDao
public class BaseDaoImpl<T> implements IBaseDao<T> {
    // 在执行过程中, 获得实体类类型
    Class<T> clazz;
    
    public BaseDaoImpl(){
        // 获得父类类型的泛型
        ParameterziedType genericSuperclass = this.getClass().getGenericSuperclass();
        Type[] types = genericSuperclass.getActualTypedArguments();
        // 获得实体类类型
        clazz = (Class<T>)types[0];
    }

    @Resource
    private SessionFactory sessoinFactory;

    public Session getSession(){
        return sessionFactory.getCurrentSession();
    }

    public void save(T entity){
        getSessoin().save(entity);
    }
    public void delete(Long id){
        //先查询, 再删除
        getSession().delete(this.findById(id));
    }
    public void update(T entity){
        getSessoin().update(entity);
    }
    public T findById(Long id){
       return this.getSession().get(clazz, id)
    }
    public List<T> findAll(){
        String hql = "from " + clazz.getSimpleName();
        getSession().createQeury(query);
        return query.list();
    }
    public List<T> findByIds(Long[] ids){
        if(ids!=null && ids.length >0 ){
        String hql = "From " + clazz.getSimpleName() + " where id in (:ids)";
        Qeury query = getSession().createQuery(hql);
        query.setParameterList("ids", ids); // 为命名参数赋值
        return query.list();
        }
        Collections.EMPTY_LIST;
    }
}
~~~~~~
9. 抽取通用 Action
~~~~~~
class BaseAction<T> extends ActionSupport implements ModelDriver<T> {
    protected T model;
    public BaseAction(){
        // 构造方法中实例化模型对象
        ParameterzedTypes superclass = this.getClass().getGernericSuperclass();
        Type[] types = superclass.getActualTypedArguments();
        // 获得实体类类型
        Class<T> clazz = (Class<T>)types[0];
        model = clazz.newInstance();
    }
    public T getModel(){
        return model;
    }
    // 将集合属性压入值栈
    protected void set(String key, List<?> value) {
        ActionContext.getContext().getValueStack().set(key, value);
    }
    // 将单个属性压入值栈
    protected void push(Ojbect obj) {
        ActionContext.getContext().push(obj);
    }
}
~~~~~~

# 系统管理(3) #

抽取实体类: 岗位(Role), 部门(Department), 用户(User)
~~~~~~
public class Role {
    private Long id;
    private String name;
    private String description;
    private Set<User> users = new HashSet();
}
public class Department {
    private Long id;
    private String name;
    private String description;
    private Department parent;
    private Set<Department> children = new HashSet();
    private Set<User> users = new HashSet();
}
public class User {
    private Long id;
    private loginName:String
    private String name;
    private String password;
    private String email;
    private int gender;
    private String phoneNumber
    private Department department;
    private Set<Role> roles = new HashSet();
}
~~~~~~
~~~~~~
<hibernate-mapping>
    <class name="Role">
        <id name="id">
            <gernerator class="native">
            </gernerator>
        </id>
        <property name="name" length="32"></property>
        <property name="description"></property>
        <set name="user" table="itcast_user_role">
            <key column="roleId"></key>
            <many-to-many class="user" column="userId"></many-to-many>
        </set>
    </class>

    <class name="Department" table="itcast_department">
        <id name="id">
            <gernerator class="native">
            </gernerator>
        </id>
        <property name="name" length="32"></property>
        <property name="description"></property>
        
        <many-to-one name="parent" class="Department" column="parentId"></many-to-one>
        <set name="children">
            <key column="parentId">
            </key>
            <one-to-many class="Department"/>
        </set>
        <set name="users">
            <key column="department">
            </key>
            <one-to-many class="User"></one-to-many>
        </set>
    </class>
    <class name="User" table="itcast_user">
        <id name="id">
            <generator class="native"> </generator>
        </id>
        <property name="loginName" length="32"></property>
        <property name="name"></property>
        <property name="descrption"></property>
        <property name="gender"></property>
        <property name="email"></property>
        <property name="phoneNumber"></property>
        <property name="password"></property>
        <many-to-one name="department" class="Department"></many-to-one>
        <set name="roles" table="itcast_user_rols">
            <key column="userId">
            </key>
            <many-to-many class="Role" column="roleId"></many-to-many>
        </set>
    </class>
</hibernate-mapping>
~~~~~~
## 岗位管理 ##
~~~~~~
@Controller
@Scope("prototype")
public class RoleAction extends BaseAction<Role> {
    @Resource
    private IRoleService roleService;
    // 查询所有岗位列表
    @Action(value="role_List" results = {@Results(name="list", location="rolsList.jsp")})
    public String list(){
        List<Role> roleList = roleService.findAll();
        set("roleList", roleList);
        return "list";
    }
    
    @Action(value="role_delete" results = {@Results(name="list", location="roleList", type="redirectAction")})
    public String delete(){
        return "toList";
    }
    
    @Action(value="role_save" results = {@Results(name="list", location="roleList", type="redirectAction")})
    public String save(){
        return "toList";
    }

    // 跳转到updateUI页面
    public String updateUI(){
        Role role = roleService.findById(model.getId);
        push(role);
        return "updateUI";
    }

}

@Service
@Transaction
public class RoleServiceImpl implements IRoleService {
    @Resource
    private IRoleDao roleDao;
    // 查询岗位列表
    @Transactional(readOnly=true)
    public List<Role> findAll(){
        roleDao.findAll();
    }
}

// 岗位管理
@Repository
public class RoleDaoImpl extends BaseDao<Role> implements IRoleDao {
}
~~~~~~

~~~~~~
<!-- RoleAction-role_save-validation.xml -->
<validators>
    <field name="name">
        <field-validator type="requiedstring">
            <message key="roleNameNotNull"></message>
        </field-validator>
    </field>
</validators>
<!-- messages.properties -->
~~~~~~


~~~~~~
<table>
    <tr>
        <th>岗位名称</th>
        <th>描述</th>
        <th>相关操作</th>
    </tr>
    <s:iterator value="roleList">
        <tr>
            <td> ${name} </td>
            <td> ${description} </td>
            <td>  </td>
        </tr>
    </s:iterator>
</table>
~~~~~~

## 部门管理 ##
~~~~~~
@Controller
@Scope("prototype")
public class DepartmentAction extends BaseAction<Departemnt>{
    // 注意返回上一级时的设计
    private Long parentId; // 属性驱动, 上级部门的Id
    
    // 查询部门列表功能
    public String list(){
        if(parentId == null) {
            // 查询顶级部门列表
            // hql: from department where parent is null
            List list = departmentService.findTopTist();
        } else {
            // 查询下级部门
            // hql: from department where parent.id=:parentId;
            list = departmentService.findChildren(parentId);
        }
        
        set("list", list);
        return LIST;
    }
    // 删除上级部门同时删除下级部门, 配置cascade
    public String delete(){
        if(getParent().getId()!=null)parentId = model.getParent().getId();
        return TOLIST;
    }
    // 新建 删除时,要注意传过来 parentId
    public String saveUI(){
        // 准备部门列表数据, 用户填充下拉框
        // 使用递归, 包装成树形结构, 使用缩进
        // 空格 替换成 &nbsp;
    }
    public String save(){}
    public String updateUI(){
        // 准备部门列表数据, 用户填充下拉框
        // 使用递归, 包装成树形结构名字
        // 空格 替换成 &nbsp;
        // 在修改时展示的不应该是全的树
    }
    public String update(){}
}

~~~~~~


## 用户管理 ##

~~~~~~
@Controller
@Scope("prototype")
public class UserAction {
    private Long departmentId;
    private Long[] roleIds ;
    
    public String updateUI(){
        // 准备部门列表, 可以使用 json-lib和ajax, 异步获得
        // 准备岗位列表
    }
    public String save(){
        // 根据departmentid 和 roleids手动配置;

        // 某人密码是"1234"
        // 使用 md5 加密
    }

    // 初始化密码
    public void initPassword(){
    }
}

~~~~~~
### 用户名重复客户端校验 ##
~~~~~~
<script type="text/javascript">
    $(function(){
        // 绑定离焦时间
        $("#loginName").blur(function(){
            var v = this.value;
            $.post(${contextPath/user_checkLoginName}, {'loginName': v}, function(data){
                if(data==1) {
                    // 当前登陆名字已经存在
                    $("#showMsg").html("当前登陆名已经存在");
                    $("#saveBtn").hide();
                } else {
                    $("#showMsg").html("当前输入名可以使用");
                    $("#saveBtn").show();
                }
            })
        });
    })
</script>
~~~~~~

~~~~~~
// UserAction
public String checkLoginName(){
    String loginName = model.getLoginName().trim();
    List<User> list = userService.findByLoginName(loginName);
    response.setContentType("text/html;charset=utf-8");
    String flag = 0;
    if(list!=null && list.size()>0) {
        // 当前登录名已经存在
        flag = "1"
    }
    out.print(flag);
    return NONE;
}
~~~~~~

# 权限管理 #

角色就是权限的集合, 角色关联权限.
在进行任何一个操作之前, 先判断用户是否有权限

## 权限实体 ##
~~~~~~
public class Privilege {
    private Long id;
    private String name;
    private String url;        //权限对应的访问url地址
    private Privilege parent;  // 对应的上级权限
    private Set<Privilege> children = new HashSet();
    private Set<Role> roles = new HashSet();
}

public class Role {
    private Set<Privilege> privileges = new HashSet();
}
~~~~~~

### 初始化数据 ###
~~~~~~sql:
> source privilege.sql; -- 导入初始化数据
> insert into user(loginName, name, password) values("admin", "超级用户", md5(admin));
~~~~~~

### 显示层 ###
~~~~~~
public class RoleAction {
    public String setPrivilegeUI() {
        // 根据id 查询当前设置权限的角色
        Role role = roleService.findById();
        push(role);

        // 准备权限列表数据
        // 使用 ul li 可以来展示树形结构
        // 配合使用 jquery TreeView 插件
        List privileges = privilegeService.findTopPrivilege();

        // 设置privilegesIds属性驱动的值, 用于页面回显
        return "setPrivilegeUI";
    }
    
    private Long[] privilegeIds;
    public String setPrivilege(){
        Role role = roleService.findById();
        if(privilegeIds != null && privilegeIds.length>0) {
            List privilegeList = priprivilegeService.findByIds(privilegeIds);
        } else {
            
        }
        
        return TOLIST;
    }
}
~~~~~~


~~~~~~
<!-- set order-by=id 设置权限树有序  -->
<script>
function(){
    $("#root").treeview();
    //为复选框绑定事件
    // 当选中或取消某个权限时, 同时选中或者取消下级权限
    $("input[name=privilegeIds]").click(function(){
        $(this).siblings("ul").find("input").attr('checked', this.checked);
    });
    // 当选中某个权限时, 同时选中上级选项
    if(this.checked){
        $(this).parents("li").children("input").attr('checked', true);
    } else {
        if($(this).parent('li').siblings('li').children('input:checked').size()==0){
            $(this).parent().parent().siblings("input").attr("checked", false);
        }
    }
    // 当取消某个权限时, 如果兄弟权限没有选中, 取消上级选项
}
</script>
<ul id="root">
   <s:iterator value="children">
        <li>
            <input id="abc"/>
            <!--  label的案件效果和input绑定-->
            <label for="abc"></label>
        </li>
   </s:iterator>
</ul>
~~~~~~

# 应用主页面 #
~~~~~~
@Controller
public class HomeAction extends ActionSupport {
    public String index(){
        return "index";
    }
    public String top(){}
    public String left(){}
    public String right(){}
    public String bottom(){}
}
~~~~~~

# 用户登陆和注销 #

~~~~~~
// UserAction
public String login(){
    User user = userService.login(model);
    if(user != null) {
        session().put(key, value);
    } else {
        this.addActionError("用户名或者密码不正确");
        return "loginUI";
    }
    return "home";
}

public String logout(){
    session.remove("loginUser");
    return "loginUI";
}

// UserService
public User login(){
    model.setPassword(md5(model.getPassword()));
    // 根据用户名和密码查询用户
    return userDao.findUserByLoginNameAndPassword()
}
~~~~~~
~~~~~~
<global-result>
    <result name="loginUI">
        index.jsp
    </result>
</global-result>
~~~~~~

# 使用权限 #
左侧菜单数据从数据库中获取，编写监听器, 当项目启动时加载权限数据, 将权限数据放入 `application` 作用域

~~~~~~
// 在web.xml配置监听器
public class OAIniitListener implements ServletContextListener {
    public void contextInitialized(ServletContextEvent sce) {
        // 获得 spring 工厂对象
        // 方式一: application.getAttribute(WebApplicationContextUtil.ROOT_WEB_APPLICATION_CONTEXT);
        WebApplicationContext cxt =
            WebApplicationContextUtil.getWebApplicationContext(sce.getServletContext());
        IPrivilegeService ps = cxt.getBean("privilegeService");
        // 从工厂中获得一个权限的Service对象
        List<Privilege> privilegeTopList = ps.findTopList();
        // 将权限数据放入作用域, lazy要设置为 false
        sce.getServletContext().setAttribute("privilegeTopList", privilegeTopList);

        //需要全部权限对应的url
        List<String> allUrls = ps.findAllUrls();
        sce.getServletContext().setAttribute("allUrls", allUrls);        
    }
    public void contextDestroyed(ServletContextEvent sce){}
}

// User.class, role privileges都要立即加载
public boolean checkPrivilegeByName(String privilegeName){
    if(isAdmin()) return true;
    for(Role r: roles) {
       Set<Privilege> privileges = r.getPrivileges();
       for(Privilege privilege : privileges) {
           if(privilegeName.equals(privilege.getName())) {
               return true;
           }
       }
    }
    return false;
}

public boolean isAdmin(){
    return "admin".equals(loginName);
}
~~~~~~

~~~~~~
<!-- 侧边栏根据权限显示 -->
<s:iterator value="#application.privilegeTopList">
     <!-- 使用 ongl表达式调用对象方法 -->
     <s:if test="#session.loginUser.checkPrivilegeByName(name)">
     </s:if>
</s:iterator>
~~~~~~

## 权限拦截器 ##
~~~~~~
// 在struts中注册 interceptor
public class CheckPrivilegeInterceptor extends AbstractInterceptor {
    public String intercept(ActionInvocation ai) {
        ActionProxy actionProxy = ai.getProxy();  // 获得当前 Action的代理对象
        
        String namespace = actionProxy.getNamespace();
        String actionName = actionProxy.getActionName();
        
        String url = namespace + actionName;
        // 如果有UI结尾的去掉
        if(url.endsWith("UI")) {
            url.subString(0, url.length()-2);
        }
        
        User user = ActionContext.getSession().get("loginUser");
        //如果用户没有登陆
        if(user == null) {
            // 如果用户访问的是登陆页面,方向
            if("user_login".equals(url)) {
                return ai.invoke();
            } 
            // 如果用户访问的不是登陆, 跳转到登陆页面
            return "loginUI";
        } else {
            // 如果用户已经登陆
            List<String> allUrls = servletContext().getAttribute("allUrls");
            // 如果当前访问的功能不需要控制
            if(!allUrls.contains(url)) return ai.invoke();
            
            boolean hasPrivilege = usesr.checkPrivilegeByUrl(url);
            // 如果用户有权限
            if(hasPrivilege) return ai.invoke();
            // 如果用户没有,掉转到没有权限的提示页面
            return "noPrivilege";
        }

        return null;
    }
}
~~~~~~

## 页面嵌套显示 ##
~~~~~~
// index.jsp
if(window.parent != window) {
    window.parent.location.href = "index.jsp"
}
~~~~~~



# 个人信息 #
~~~~~~
insert into itcast_privilege (21, "个人设置", null);
insert into itcast_privilege (22, "个人信息", 21);
~~~~~~

~~~~~~
// userAction
// 设置个人信息
private File resource;
private String resourceFileName;
private resourceContentType;

public String setUserInfo(){
    if(model.getId()==null) {
        // 跳转的设置页面
        // 在是视图层, user 中的集合还没有初始化, opensessionInview,
        // 在一次请求中才生效
        User user = session.get("loginUser");
        push( userService.findById(user.getId()) );
        return "SetUserInfo"
    } else {
        // 设置信息
        User user = userService.findById(model.getId());
        // 可以抽到 BaseAction, 文件夹按日期存储文件
        // 获得uploadFiles的绝对磁盘路径
        String realPath = servletContext.getRealPath("uploadFiles");
        int position = resourceFileName.lastIndexOf(".");
        String suffix = resourceFileName.subString(position);

        String filename = UUID.randomUUID().toString() + suffix;

        File destFile = new File(realPath + File.separator+ filename);
        resource.renameto(destFile);
        user.setSavePath(filename);
        userService.update(user);
        return "toSetUserInfoUI";
    }
}
// User
class User {
    private String savePath;
}
~~~~~~

~~~~~~
<!-- 文件上传 -->
<form enctype="multipart/form-data">
    <input type="file" name="resource"/>
</form>
~~~~~~

## struts文件下载 ##
* Action中提供 `InputStream inputName`
* 配置结果集
~~~~~~
<result name="stream">
    <param name="inputName"></param>
    <param name="contentType"></param>
    <param name="contentDisposition"></param>
</result>
~~~~~~

# 论坛管理 #

1. 提取实体, 版块(form), 主题(Topic), 回复(Reply);
帖子类型, 普通帖子, 精华帖, 置顶帖, 热门帖;
不同类型的帖子会影响列表排序
~~~~~~
// FormManagerAction, 重写Dao, findAll按position排序
public class Form {
    Long id;
    String name;
    String description;
    int position = 0; // 默认为0, 在新建时等于id的值
    Set<Topic> topics;
    // 提高检索效率
    Int topicCount; // 主题数量
    Int articleCount; // 文章数量(回复数)
    Topic lastTopic;   // 最后发表主题
}

public class Topic {
    Long id;
    String title;
    String content;
    int type;
    User author;  // 多对一
    Date postTime;
    String ip;
    Date lastUpdateTime; // 针对主题最后回复时间
    Set<Reply> replies;
    // 提高检索效率
    Int replyCount;
    Reply lastReply;
}

public class Reply {
    Long id;
    String content;
    Date postTime;
    User author;
    String ip;
    Int deleted;
    Topic topic;
}
~~~~~~

## 论坛管理模块 ##
论坛版块管理, 添加, 删除, 上移下移
~~~~~~
class FormManagerAction{}
// 上移

// Service
public void moveUp(Long id) {
    // select * from form where position < :up desc limit 0,1
    // 相互交换位置信息
}
~~~~~~
~~~~~~
<s:if test="#s.first"> 上移 </s:if>
~~~~~~

## 论坛模块 ##
~~~~~~

class FormAction {
    // 排序按照 position
    // `form/list.jsp`
    // 版块列表
    public String list(){}

    // 主题列表
    // 需要排序: 置顶帖在最上方, 剩下的按时间降序排序
    // select * from itcast_topic order by (case type when 2 then 2 else 1 end) desc, postTime desc;
    // 注意 order by type desc, postTime desc 区别
    public String show(){
        // 根据版块id查询版块信息
        Form form = formManagerService.findById(model.getId);
        // 根据 版块id查询主题列表
        List topicList = topicService.findTopicListByForum(model);
        set("topicList", topicList);
        return "formShow";
    }
}
~~~~~~

## 发表主题 ##

~~~~~~
public class TopicAction extends BaseAction{
    private Long forumId;
    
    public String saveUI(){
        Forum form = findById(forumId);
        push(form);
    }

    public String save(){
        Forum form = findById(forumId);
        model.setForum(forum);
        model.setAuthor(getCurrentUser());
        model.setIp(getIpAddress()); //request.getRemoteAddress();
        model.setPostTime(new Date());
        model.setLastUpdateTime(model.getPostTime());
        model.setReplyCount(0);
        model.setType(0);
        
        topicService.save(model);
        return "toTopicList";
    }
}

// topicService
public void save(Topic model) {
    topicDao.save(model);
    Forum forum = model.getforum();
    // 版块主题
    forum.setTopicCount(forum.getTopicCount() + 1);
    // 版块文章 
    forum.setArticleCount(forum.getArticleCount() + 1);
    forum.setLastTopic(model);
}
~~~~~~

### CKEditor编辑内容 ###
使用 ckEditor 插件内容编辑器
~~~~~~
<textarea id='editor>
</textarea>
<script>
CKEDITOR.repalce('editor', {});
</script>
~~~~~~

## 显示单个主题(回复列表) ##
~~~~~~
// TopicAction
public String show(){
    Topic topic = topicService.findById(model.getId);
    push(topic);
    // 根据主题查询回复列表, order by postTime asc
    List replyList = findReplyListByTopic(model.getId());
    set("replyList", replyList);
    
    return "topicshow";
}
~~~~~~

## 回复功能 ##


~~~~~~
// ### 快速回复功能 ###
public class ReplyAction {
    private Long topicId;
    // 针对某个主题, 快速回复
    public String save(){
        Topic topic = topicService.findById(topicId);
        model.setTopic(topic);
        model.setAuthor(getCurrentUser());
        model.setDeleted(0);
        model.setIp(getIpAddress());
        model.setPostTime(new Date());

        replyService.save(model);
        return "toTopicShow";
    }
    //普通回复(单页)
    public String saveUI(){
        Topic topic = topicService.findById(model.getId());
        push(topic);
        return "SAVEUI";
    }
}
// replyService
public void save(Reply mode) {
    replyDao.save(model);
    Topic topic = model.getTopic();
    Forum forum = topic.getForum();
    topic.setLastUpdateTime(model.getPostTime());
    topic.getReplyCount(topic.setReplyCount() + 1);
    topic.setLastReply(model);
    // 文章数+1
    forum.setArticleCount(aticalAcount + 1);
}

~~~~~~


## 分页 ##
~~~~~~
// 封装分页信息
public class PageBean {
   // 页面传递参数
   private int currentPage; // 当前页码 
   private int pageSize;  // 每页显示记录数
   // 查询数据库获得
   private int recordCount; // 总记录数
   private List recorderList;

   private int pageCount;  // 总页数
   private int beginPageIndex; 
   private int endPageIndex;

    public PageBean(){}
    public PageBean(int currentPage, int pageSize, int recordCount, List recorderList){
        // 计算总页数
        this.pageCount = (this.recordCount + this.pageSize - 1)/pageSize;
        // 计算开始索引和结束索引
        if(tihs.pageCount <= 10) {
            this.beginPageIndex = 1;
            this.endPageIndex = this.pageCount
        } else {
            this.beginPageIndex = this.currentPage - 4;
            this.endPageIndex = this.currentPage + 5;
            if(this.beginPageIndex < 1) {
                beginPageIndex = 1;
                endPageIndex = 10
            }
            if(endPageIndex > pageCount) {
                endPageIndex = pageCount;
                beginPageIndex = endPageIndex - 9;
            }
        }
    }
}
~~~~~~

~~~~~~
// topicAction
// BaseAction{ private int currentPage } // 属性驱动当前页码
// 使用 form 表单提交分页信息(因为表单还有其他信息, 如排序条件, 过滤条件)
public String show(){
    push(topic);

    PageBean pb = replyService.findReplyListByTopic(model, currentPage);
    push(pb);
    return "topicShow";
}
// 手动配置 配置 pagesize
// <context:property-palceholder loacation="classpath:jdbc.properties,classpath:pageSize.properties">
// </context:property-palceholder>
~~~~~~


