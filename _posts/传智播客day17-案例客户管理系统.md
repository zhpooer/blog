title: 传智播客day17-案例客户管理系统
date: 2014-04-22 15:41:49
tags:
- 传智播客
---
# 客户信息管理系统 #
用之前所学完成一个对单表的CRUD JavaWeb 项目

## 项目需求 ##
![项目需求](/img/day17_demo2.png)

## 搭建开发环境, 写配置文件, Jar包 ##

## 写 Javabean ##

~~~~~~
package cn.itcast.domain
public class Customer {
    private String id;
    private String name;
    private String gender;  // 数据库中 1男,0女
    private Date birthday;
    private String email;
    private String cellphone; 
    private String hobby; // 爱好: 吃饭,睡觉,学Java
    private String type; // 客户类型, 普通客户 vip
    private String description;
}
~~~~~~

## 建数据库表 ##
~~~~~~
create table customer (
    id varchar(100) primary key,
    name varchar(100),
    gender varchar(10),
    birthday date,
    cellphone varchar(100),
    email varchar(100),
    hobby varchar(100),
    type varchar(100),
    description varchar(100)
);
~~~~~~

## 业务接口 ##
~~~~~~
package cn.itcast.service
public interface BusinessService {
    List<Customer> findAll();
    void addCustomer(Customer c);
    void delCustomer(String customerId);
    Customer findCustomerById(String customerId);
    // 如果传入 id 为 null, 抛出此异常
    void updateCustomer(Customer c) throws CustomerIdConnotBeEmpty;
}
~~~~~~
~~~~~~
package cn.itcast.exception
public class CustomerIdConnotBeEmpty extends Exception{ }
~~~~~~

### 业务实现 ##
~~~~~~
package cn.itcast.service.impl;
public class BusinessServiceImpl {
    Customer dao = new CustomerDaoImpl()
    public List<Customer> findAll(){
        return dao.findAll();
    }
    public void addCustomer(Customer c){
        c.setId(UUID.randomUUID().toString());
        dao.save(c);
    }
    public void delCustomer(String customerId){
         dao.delete(customerId);
    }
    public Customer findCustomerById(String customerId){
        dao.findById(customerId);
    }
    // 如果传入 id 为 null, 抛出此异常
    public void updateCustomer(Customer c) throws CustomerIdConnotBeEmpty{
         if(c.getId()==null) {
             throw new CustomerIdConnotBeEmpty("参数有误");
         }
         dao.update(c);
    }
}
~~~~~~

## Dao接口 ##
~~~~~~
public interface CustomerDao {
    List<Customer> findAll();
    void save(Customer c);
    void findById(String customerId);
    void delete(String customerId);
    void update(Customer c);
}
~~~~~~

### CustomerDaoImpl 实现 ###
~~~~~~
public List<Customer> findAll(){
    Connection conn = null;
    PreparedStatement stmt = null;
    ResultSet rs = null
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.PreparedStatement();
        re = stmt.excuteQuery();
        List<Customer> cs = new ArrayList<Customer>();
        while(rs.next()){
            Customer c = new Customer();
            c.setId(r.getString("id"));
            cs.add(c);
        }
        return cs;
    } catch (e) {
        throw new RuntimeException(e);
    } finally {
        JdbcUtil.release(rs, stmt, conn);
    }
}
public void save(Customer c) {
    Connection conn = null;
    PreparedStatement stmt = null;
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.PreparedStatement("");
        stmt.setString(1, c.getId());
        re = stmt.excuteUpdate();
    } catch (e) {
        throw new RuntimeException(e);
    } finally {
        JdbcUtil.release(rs, stmt, conn);
    }
}
public void delete(String customerId) {
    Connection conn = null;
    PreparedStatement stmt = null;
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.PreparedStatement("");
        stmt.setString(1, c.getId());
        re = stmt.excuteUpdate();
    } catch (e) {
        throw new RuntimeException(e);
    } finally {
        JdbcUtil.release(rs, stmt, conn);
    }
}
public void findById(String customerId) {
    Connection conn = null;
    PreparedStatement stmt = null;
    ResultSet rs = null
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.PreparedStatement("");
        re = stmt.excuteQuery();
        while(rs.next()){
            Customer c = new Customer();
            c.setId(r.getString("id"));
            return c;
        }
        return null;
    } catch (e) {
        throw new RuntimeException(e);
    } finally {
        JdbcUtil.release(rs, stmt, conn);
    }
}
public void update(Customer c) {
    Connection conn = null;
    PreparedStatement stmt = null;
    try{
        conn = JdbcUtil.getConnection();
        stmt = conn.PreparedStatement("");
        stmt.setString(1, c.getName());
        re = stmt.excuteUpdate();
    } catch (e) {
        throw new RuntimeException(e);
    } finally {
        JdbcUtil.release(rs, stmt, conn);
    }
}

~~~~~~

### 测试Service实现 ###

~~~~~~
public class BusinessServiceImplTest {
    public BusinessService s = new BusinessServiceImpl();
    @Test public void testAddCustomer() {
        Customer c = new Customer();
        c.setId("aaa");
        c.setName("xxx");
        c.setGender("1");
        c.setBirthday(new Date());
        c.setEmail("zhpo@gmai.com");
        c.setHobby("学习 吃饭");
        c.setType("vip");
        c.setDescription("xxx");
        s.addCustomer(c);
    }
    @Test public void testFindAll() {
        List<Customer> cs = s.findAll();
        assertEquals(2, cs.size());
    }
    @Test public void testFindCustomerById() {
       Customer c = s.findCustomerById("****");
       assertNotNull(c);
    }
    @Test(expected=CustomerIdConnotBeEmpty.class)
    public void testUpdateCustomer() {
        Customer c = new Customer();
        c.updateCustomer();
    }
    @Test public void testUpdateCustomer1() {
        Customer c = s.findCustomerById("aaa");
        c.setName("xxx");
        c.setGender("0");
        s.updateCustomer(c);
    }
    @Test public void testDelCustomer() {
        s.delCustomer("xx");
    }
}
~~~~~~

## 表现层 ##
~~~~~~
public class Controller extends HttpServlet {
    public void doGet(){
        req.setCharactorEncoding("utf-8");
        res.setContentType("text/html;charset=utf-8");
        String op = req.getParameter("op");
        if("showAllCustomer".equals(op)) {
            showAllCustomers(req, res);
        }if("addCustomer".equals(op)) {
            addCustomer(req, res);
        }
        
    }
    private void showAllCustomers(){
         List<Customer> cs = s findAll();
         req.setAttribute("cs", cs);
         req.setRequestDispather("ListCustomers.jsp");
    }
 
}
~~~~~~

~~~~~~
<body>
    <jsp:forward page="/servlet/Controller">
        <jsp:param value="showAllCustomer" name="op"> </jsp:param>
    </jsp:forward>
</body>
~~~~~~

~~~~~~
// ListCustomers.jsp
<style type="text/css">
    .odd{
        background-color: #c3f3c3;
    }
    .even{
       background-color: #f3c3f3;
    }
    body{
       text-align : center;
       font-size: 12px
    }
    table {
       font-size: 12px
    }
</style>
<div>
    <a href="${pageContext.request.contextPath}/addCustomer.jsp">添加</a>
    <a href="">删除</a>
</div>
<c:if test="${empty cs}">
   没有客户信息
</c:if>
<c:if test="${!empty cs}">
   <table border="1">
       <tr>
           <th> 选择 </th>
           <th> 姓名 </th>
           <th> 性别 </th>
           <th>出生日期 </th>
           <th> 电话 </th>
           <th> 邮箱 </th>
           <th> 爱好 </th>
           <th> 类型 </th>
           <th> 描述 </th>
           <th> 操作 </th>
       </tr>
       <c:forEach items="${cs}" var="c" varStatus="vs">
           <tr class="${vs.index%2==0?'odd':'even'}">
              <td>
                  <input type="checkbox" name="ids" value="${c.id}"/>
              </td>
              <td> ${c.name} </td>
              <td> ${c.gender} </td>
              <td> ${c.birthday} </td>
              <td> ${c.cellphone} </td>
              <td> ${c.email} </td>
              <td> ${c.hobby} </td>
              <td> ${c.type} </td>
              <td> ${c.description} </td>
              <td>
                  <a href=""> 修改</a>
                  <a href=""> 添加</a>
              </td>
           </tr>
       </c:forEach>
   </table>
</c:if>
~~~~~~

### 添加客户 ###

~~~~~~
<!-- addCustomer.jsp -->
<!-- post方式,用get方式提交都能提交, get方式就不行 -->
<form action="${pageContext.request.contextPath}/servlet/Controller?op=addCustomer" metho="post">
    <table boder="1">
        <tr>
            <td> 姓名 </td>
            <td> <input type="text" name="name"/> </td>
        </tr>
        <tr>
            <td> 性别 </td>
            <td>
            <input type="radio" name="gender" value="2" checked/>
            </td>
        </tr>
        <tr>
            <td> 出生日期</td>
            <td> <input type="text" name="birthday" value="1990-1-1"/> </td>
        </tr>
        <tr>
            <td> 电话 </td>
            <td> <input type="text" name="cellphone"/> </td>
        </tr>
        <tr>
            <td> 邮箱 </td>
            <td> <input type="text" name="email"/> </td>
        </tr>
        <tr>
            <td> </td>
            <td> <input type="text" name=""/> </td>
        </tr> 
        <tr>
            <td> 爱好: </td>
            <td>
               <input type="checkbox" name="hobbies" value=""/>
               <input type="checkbox" name="hobbies" value=""/>
               <input type="checkbox" name="hobbies" value=""/>
               <input type="checkbox" name="hobbies" value=""/>
            </td>
        </tr>
        <tr>
            <td> 类型</td>
            <td>
                <input type="radio" name="type" value="vip"/>
                <input type="radio" name="type" value="normal"/>
            </td>
        </tr>
        <tr>
            <td> 姓名 </td>
            <td>
                <textarea rows="3" cols="38" name="description">
                </textarea>
            </td>
        </tr>        
    </table>
</form>
~~~~~~

**Controller.java**
~~~~~~
   private void addCustomer(){
        // 类型封装到javabean中
        CustomerFormBean bean = WebUtil.fillBean(request, CustomerFormBean.class)
        // 验证用户信息
        if(!FormBean.validate()){
        // 不正确, 数据要回显
           request.setAttribute("FormBean", FormBean);
           request.getRequestDispather("/addCustomer.jsp").forward();
           return;
        }
        // 填充模型 FormBean => Javabean
        ConvertUtils.register(new DateLocaleConverter, Date.class);
        Customer c = new Customer()
        try{
           BeanUtils.copyProperties(c, bean);
        } catch{
            throw new RuntimeException(e);
        }
        // 单独处理 hobbies
        String hobbies = request.getParameter("hobbies");
        if(hobbies!=null && hobbies.length>0){
            StringBuffer sb = new StringBuffer();
            if(int i=0;i<hobbies.length; i++){
               if(i>0){
                   sb.append(",");
               }
               sb.append(hobbies[i]);
            }
            c.setHobby(sb.toString());
        }
        s.addCustomer(c);
        // out.wirte("注册成功");
        res.sendRedirect(request.getContextPath()); // 防止重复提交
    }
~~~~~~

~~~~~~
public class CustomerFormBean {
    private String id;
    private String name;
    private String gender;
    private String birthday;
    private String email;
    private String cellphone; 
    private String[] hobbies; 
    private String type; 
    private String description;
    private Map<String, String> errors = new HashMap<String, String>;
    public boolean validate() {}
}
~~~~~~

### 修改 ###
**Controller.java**
~~~~~~
public void editCustomerUI(){
    String customerId = req.getParameter("customerId");
    Customer c = s.findCustomerById(customerId);
    request.setAttribute("c", c);
    req.getRequestDispather("editCustomer.jsp").forward();
}
public void editCustomer(){
    // 参考saveCustomer
}
~~~~~~

~~~~~~
// editCustomer.jsp
<form action="${pageContext.request.contextPath}/servlet/Controller?op=editCustomer" metho="post">
    <input type="hidden" name="id" value="{c.id}"/>
    <table boder="1">
        <tr>
            <td> 姓名 </td>
            <td> <input type="text" name="name"/> </td>
        </tr>
        <tr>
            <td> 性别 </td>
            <td>
            <input type="radio" name="gender" value="2" checked/>
            </td>
        </tr>
        <tr>
            <td> 出生日期</td>
            <td> <input type="text" name="birthday" value="1990-1-1"/> </td>
        </tr>
        <tr>
            <td> 电话 </td>
            <td> <input type="text" name="cellphone"/> </td>
        </tr>
        <tr>
            <td> 邮箱 </td>
            <td> <input type="text" name="email" value="${c.email}"/> </td>
        </tr>
        <tr>
            <td> </td>
            <td> <input type="text" name=""/> </td>
        </tr> 
        <tr>
            <td> 爱好: </td>
            <td>
               <input type="checkbox" name="hobbies" ${fn:contains(c.hobby, '吃饭')?'checked':''}  value=""/>
               <input type="checkbox" name="hobbies" value=""/>
               <input type="checkbox" name="hobbies" value=""/>
               <input type="checkbox" name="hobbies" value=""/>
            </td>
        </tr>
        <tr>
            <td> 类型</td>
            <td>
                <input type="radio" name="type" value="vip"/>
                <input type="radio" name="type" value="normal"/>
            </td>
        </tr>
        <tr>
            <td> 姓名 </td>
            <td>
                <textarea rows="3" cols="38" name="description">
                </textarea>
            </td>
        </tr>
    </table>
</form>
~~~~~~

### 删除 ##
~~~~~~
<!-- list.jsp -->
<a href="javascript:delOne('${c.id}')">删除</a>
<script type="text/javascript">
    function delOne(customerId){
        var sure = window.confirm("确定要删除吗")?
        if(sure) {
            window.loacation.href="${pageContext.request.contextPath}/servlet/Controller?op=delOneCustomerUI&custermerId="+custmerId;
        }
    }
</script>
~~~~~~

~~~~~~
private void delOneCustomerUI(){
    String customerId = req.getParameter("customerId");
    s.delCustomer(customerId);
    response.sendRedirect(request.getContextPath());
}
~~~~~~
### 删除多个 ##
~~~~~~
<form atcion="${}/servlet/Controller?op=delMuti" method="post">
   ...
</form>

function delMulti(){
    // 首先判断用户哟没有选择要删除的记录
    var selected = false;
    var idsArray = document.getElementsByName("ids");
    for (int i=0;i<idsArray.length; i++){
        if(idsArray[i].checked) {
            selected = true;
            break;
        }
    }
        // 选了, 二次提示, 确定要删除吗
    if(selected){
        var sure = window.confirm("确定吗");
        // 二次提示: 确定,让表单提交即可
        if(sure) {
           document.forms[0].submit()
        }
    } else {
        // 没选, 提示, 请选择要删除的记录
        alert("请选择")
    }
}
~~~~~~

~~~~~~
private void delMulti(){
    String ids[] = request.getParameterValues("ids");
    if(ids!=null&&ids.length>0){
        for(String id:ids) s.delCustomer(id);
    }
    response.sendRedirect(request.getContextPath());
}
~~~~~~
