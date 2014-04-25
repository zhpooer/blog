title: 传智播客day18-大结果集和大数据处理
date: 2014-04-23 09:26:45
tags:
- 传智播客
- jdbc
---

# 大结果集分页 #
1. 分页靠SQL支持, 不同数据库的分页语句是不通过的
2. MySQL 分页语句 `limit M,N`  
  *M: 开始记录的数据, 第一页的第一条索引是0*  
  *N: 每次取出多少条*
  
~~~~~~
-- 取第一页, 每次取10条
select * from customer limit 0, 10;
-- 取第二页, 每次取10条
select * from customer limit 10, 10;
-- 取第n页, 每次取10条
select * from customer limit (n-1)*10, 10;

-- 总共的页数(每次取10条)
-- = 总记录条数%N==0?总记录条数/N:((总记录条数/N)+1)
~~~~~~
## 对客户信息进行分页 ##
### 新建Page类, 封装分页信息 ###
~~~~~~
// 专门封装与分页有关的数据
public class Page{
    private int pageSize; // 每页显示的记录条数
    private List recorder; // 每页显示的记录          DAO
    private int pageNum;  // 当前页码              用户传
    private int totalPage;  // 总共页码            计算
    private int startIndex; // 每页开始记录的索引    计算
    private int totalRecords; // 总记录的条数        Dao

    public Page(intPageNum, int totalRecords){
        this.pageNum = pageNum;
        this.totalRecords = totalRecords;

        this.totalPage = totalRecords%pageSize==0?totalRecords/pageSize:totalRecords/pageSize+1;
        startIndex = (pageNum-1)*pageSize;
    }
}
~~~~~~

### 改造接口DAO ###
~~~~~~
public interface CustomerDao {
    @Deprecated
    List<Customer> findAll();
    int getTotalRecordsNum();
    List<Customer> findPageCustomers(int offset, int size);
}
// 还要修改实现, SQL语句用 select * form ** where ** limit offset, size
~~~~~~


### 改造业务代码 ###

~~~~~~
public interface BusinessService{
    @Deprecated
    List<Customer> findAll();
    Page findPage(String pageNum)
}
public class BusinessServiceImpl{
    public Page findPage(String num) {
        int pageNum = 1;
        if(num!=null){
            pageNum = Integer.parseInt(num);
        }
        int totalRecords = dao.getTotalRecordsNum();
        Page page = new Page(pageNum, totalRecords);
        List<Customer> records = dao.findPageCustomers(page.statedIndex, page.pageSize);
        page.setRecords(records);
        return page;
    }
}
~~~~~~

### 改造Servlet ###
~~~~~~
private void showAllCustomer(){
   String num = request.getParameter("num");
   Page page = s.findPage(num);
   request.setAttribute("page", page);
   request.getRequestDispather("/listCustomer.jsp").forward(req, res);
}
~~~~~~

### 改造JSP显示页面 ###

~~~~~~
<c:if test="${!empty page}">
    <c:forEach items="page.records" var="c" varStatus="vs">
    </c:forEach>
</c:if>

第${page.pageNum}页&nbsp;&nbsp; 共${page.totalPage}页

<a href="${}/servlet/Controller?op=showAllCustomers&num=${pageNum-1<1?1:pageNum-1}">
</a>
<a href="${}/servlet/Controller?op=showAllCustomers&num=${pageNum+1>totalPage?totalPage:pageNum+1}">
</a>
// 跳页 方式一
<select id="num" name="num" onchange="jump(this)">
    <c:foreach begin="1" end="${}" var="i">
        <option value="${n}">${n}</option>
    </c:foreach>
</select>
function jump(select){
   window.location.href = "${}/servlet/Controller?op=showAllCustomers&num=" + select.value
}
// 跳页 方式二
<input type="text" size="3" name="num" id="num" value="${page.Naum}" /><a href="javascript:jump()"></a>
function fump(){
    var num = document.getElementsById("num").value
    var regObj = /^[1-9][0-9]*$/
    if(!regObj.test(num)) {
        alert("请正确输入")
        return;
    }
    if(num>${page.totalPage}){
        alert("页码超出范围");
        return;
    }
    window.location.href = "${}/servlet/Controller?op=showAllCustomers&num=" + num
}
~~~~~~
#### 提取代码 ####
**可以用静态包含, 提取公共jsp代码, 来复用代码**
~~~~~~
<%@ include file=""%>
public class Page {
    private String url = ""; // plus
}
public class Controller {
    page.setUrl("");
}
"${}/servlet/Controller?op=showAllCustomers&num=" + num // change to 
"${}${page.url}&num=" + num

~~~~~~


#JDBC 大数据(LOB)的存取 #

## 大文本 CLOB(text)##
~~~~~~
create table t1(
    id int primary key,
    content longtext
)
~~~~~~
### 存 ###
~~~~~~
Statement stmt = conn.PreparedStatement("insert into t1(id, content) value(?,?)");
stmt.setInt(1, 1);
// 使用字符输入流的形式, 提高效率
File f = new File("test.txt");
Reader reader = new FileReader(f);
// 如果不强转, 会报错 说明一个问题MySQL驱动实现不支持  setCharacterStream(int, reader, long)
// 不支持 long, 数据库本身就不支持那么大的数据(最大4G)
stmt.setCharacterStream(2, f, (int)f.length);
~~~~~~
### 取 ###
~~~~~~
stmt = conn.PreparedStatement("select * from t1 where id=1");
stmt.excuteQuery();
if(rs.next()){
   Reader reader = re.getCharacterStream("content");
   Writer writer = new FileWriter("");
   ...
}
~~~~~~

## 大二进制数据 BLOB ##
~~~~~~
create table t1(
    id int primary key,
    content longblob
)
~~~~~~
存, 取同 CBLOB, 核心方法:
* `stmt.setBinaryStream(2, InputStream, length)`
* `stmt.getBinaryStream(2)`


# 批处理 #
把sql语句缓存起来, 一起发给数据库, 减少数据库访问次数, 提高效率

~~~~~~
create table t3 (
    id int primary key,
    name varchar(100)
)
~~~~~~
* Statement
~~~~~~
Statement s1 = conn.createStatement();
String sql1 = "insert into t3 values(1, 'aaa1')"
String sql2 = "insert into t3 values(2, 'aaa2')";
String sql3 = "delete from t3 where id=1";
stmt.addBatch(sql1);
stmt.addBatch(sql2);
stmt.addBatch(sql3);
int ii[] = stmt.excuteBatch();  // 每条语句影响到的行数
~~~~~~
* PreparedStatement
~~~~~~
PreparedStatement stmt = conn.prepareStatement("insert into t3 values(?,?)");
for(int i=1; i<=1000; i++) {
   stmt.setInt(1, i);
   stmt.setString(2, "aaa" + i);
   stmt.addBatch();
   // 把准备好的参数加入到缓存中: 如果数据量太大, 内存可能溢出. 解决方案: 分批次执行
   if(i%100) {
      stmt.excuteBatch();
      stmt.clearBatch();
   }
}
stmt.excuteBatch();
~~~~~~

# 调用存储过程 #

## 存储过程简介 ##
~~~~~~
-- 修改语句结束符号 ; => $$
delimiter $$
-- 创建一个存储过程, 名字为 demoSp,
--- 括号里面是参数, 形式: in|out|inout(输入或输出) 参数名字 参数类型
create procedure demoSp(In inputParam varchar(255), inout inOutParam varchar(255))
begin
    select concat('welcome to:', inputParam) into inOutParam;
end

delimiter ;
~~~~~~
## jdbc 调用存储过程 ##
~~~~~~
Connection conn = JdbcUtil.getConnection();
// 获取执行存储过程的对象
CallableStatement stmt = conn.prepareCall("{call demoSp(?,?)}");;
// 设置参数, 输入参数要给一个值, 输出参数注册SQL数据类型
// 输入参数要给一个值
stmt.setString(1, "hch")
stmt.registerOutParameter(s, java.sql.Types.VARCHAR);

// 执行
stmt.excute(); // 这里不能使用结果集
String result = stmt.getString(2); // 取位置为2的那个结果
~~~~~~


# 事务入门 (数据安全)#

TPL: 事务(Transaction)处理语言
 
数据库有可能是自动提交事务的,(MySQL就是自动提交事务的),
每一条语句都是一个事务

* `start Transaction`: 开启事务
* `rollback`: 回滚, 回到最开始的地方
* `commit`: 提交, 永久存储到硬盘上

## JDBC 操作事务 ##

~~~~~~
try {
    Connection conn = JdbcUtil.getConnection();
    conn.setAutoCommit(false);
    PreparedStatement stmt = conn.prepareStatement("")
    stmt.excuteUpdate();
    conn.commit();
    conn.setAutoCommit(true); // 恢复现场
} finally{
   conn.rollback();
}

~~~~~~
