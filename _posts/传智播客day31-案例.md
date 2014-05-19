title: 传智播客day31-案例
date: 2014-05-17 14:05:42
tags:
- 传智播客
---
# 项目需求分析 #

模块:
1. 登陆模块
2. 用户模块的搭建

框架的搭建:
* 页面用 JSP, 放在 WEB-INF 下
* 后台: struts2 + jdbc
* 后台的分层
  + action
  + service
  + dao
* struts2的配置文件
  + `struts.xml`, `struts-login.xml`, `struts-user.xml`
  + 简单样式, 开发模式为 true
* 数据库: DBUtils 封装数据库的链接

# 代码 #
~~~~~~
<!-- login.jsp -->
<s:form action="LoginAction_login.action">
    <s:textfield name="username" cssClass="text"> </s:textfield>
    <s:textfield name="password" cssClass="text"> </s:textfield>
    <s:submit/>
</s:form>
~~~~~~
配置文件
~~~~~~
<struts>
    <constants name="struts.devMode" value="true"/>
    <constants name="struts.ui.theme" value="simple"/>
    <include file="struts/struts-user.xml"/>
    <include file="struts-login.xml"/>
</struts>
~~~~~~
移动脚本
~~~~~~
create table s_user (
    userID int not null auto_increment,
    username varchar(50) null,
    password varchar(50) null,
    sex varchar(50) null,
    birthday varchar(50) null,
    education varchar(50) null,
    telephone varchar(50) null,
    interest varchar(50) null,
    path varchar(500) null,
    filename varchar(100) null,
    remark varchar(500) null,
    primary key(userID)
);
insert into s_user (userID, username, password) values(1, "admin", "admin");
~~~~~~
持久层
~~~~~~
// cn.itcast.dao
public interface UserDao {
    public List<User> getAllUser();
    public User getUserById(Serializable id);
    public void saveUser(User user);
    public void deleteUser(Serializable id);
    public void updateUser(User user);
    public User login(String username, String password);
}
public class UserDaoImpl implements UserDao {}
~~~~~~
模型类: 
~~~~~~
public class User {
    private String long userId;
    private String username;
    private String password;
    private String sec;
    private String birthday;
    private String education;
    private String phone;
    private String interest;
    private String path;
    private String filename;
    private String remark;
}
~~~~~~
业务层:
~~~~~~
public interface UserService {
    private UserDao userDao = new UserDaoImpl();
    public User login(String username, String password){
        return userdao.login(username, password);
    }
}
~~~~~~

~~~~~~
public class UserAction extends ActionSupport implements ModelDriven<User> {
    private UserService UserService = new UserServiceImpl();
    private User model = new User();
    @Override public User getModel(){return model;}
    public String login(){
        User user = userService.login(user.getUsername(), user.getPassword);
        if(user==null) {
            this.addActioinError("用户名或者密码错误")
            return "input"; // 跳到错误页面
        }
        return "home";
    }
}
~~~~~~
struts-login.xml
~~~~~~
<package name="login" namespace="" extends="struts-default">
    <action name="loginAction_*" method="{1}" class="UserAction">
       <result name="input"> login.jsp </result>
    </action>
</package>
~~~~~~
## 显示层 ##
~~~~~~
<!-- home.jsp -->
<frameset>
    <frame src="forwardAction_forward.action?method-top" name="topFrame" scrolling="NO" noresize="">
    </frame>
    <!-- ...脑补... -->
</frameset>
~~~~~~

~~~~~~
public class ForwardAction {
    private String method;
    public String forward() {
        // 配置 struts-forwards.xml
        // result name=top => top.jsp
        // result name=left => left.jsp
        // result name=bottom => bottom.jsp
        // result name=right => welcome.jsp
        return method;
    }
}
~~~~~~

~~~~~~
<!-- struts-login -->
<result name="home"> WEB-INF/frame/home.jsp </result>
~~~~~~

左侧的树是利用登陆一个框架 dTree 框架
~~~~~~
<!-- left.jsp -->
<link rel="stylesheet" href="dtree.css" type="text/css"/>
<script type="text/javascript" src="dtree.js"> </script>
<script type="txt/javascript">
d = new dTree('d')
d.add(0, -1, '系统菜单树'); // 设置树的父节点
// 第一个参数: 为该节点的 id
// 第二个参数: 父节点
// 三: 节点的名称
// 四: 链接到的页面
// 六: 需要显示的 frame
d.add(2, 0, '用户管理', userAction_showAllUser.action', '', 'right');
d.add(3, 2, '员工管理', 'list.jsp', '', right');
document.add(d);
</script>
~~~~~~

~~~~~~
// UserAction
public String showAllUser() {
    List<User> userList = userService.getAllUser();
    ActionContext.getContext().put("userList", userList);
    return "list.jsp"; // 添加struts-user.xml, 修改 user/list.jsp, 迭代userList
}
// UserService
public List<User> getAllUser() {
    userdao.getAllUser();
}
~~~~~~


## 添加用户 ##

~~~~~~
// UserAction
@BeanProperty private File upload; // 该属性用于文件上传
@BeanProperty private String uploadFileName;
@BeanProperty private String uploadContentType;
public void addUI(){
    return "add.jsp"
}
public String add(){
    String path = UploadUtils.saveUploadFile(upload);
    getModel.setPath(path);
    getModel.setFilename(uploadFileName);
    userService.addUser(getModel);
    
}

// userService
public void addUser(User user){
    userDao.saveUser(user);
}
~~~~~~

~~~~~~
// list.jsp
function addUser(){
    window.location.href="UserAction_addUI";
}
~~~~~~

## 修改客户信息 ##

~~~~~~
// list.jsp: <s:a action="userAction_showUserByID?userId=%{userId}">修改</s:a>
// 注意 在struts2标签中, 不能跟 el 表达式
// 在html标签中 不能加 ognl 表达式
// Useraction
private String[] interest; // 用来回显爱好
public String showUserByID {
   User user =  userService.getUserById(getModel.getUserId());
   interest = user.getInterest().split(",");
   ActionContext.getContext().getValueStack().push(user);
   return "updateUI";
}
public String updateUser(){
    userService.updateUser(model);
    return "list.jsp";
}

// UserService
public User getUserById(Serializable id){
    return userDao.getUserById();
}
~~~~~~

### interests 的回显 ###

~~~~~~
// UserAction

@BeanProperty private String[] ids; // 用来存储 页面上 checkboxlist
// 在代码中 处理 ids, 加入到 userAction中
~~~~~~
