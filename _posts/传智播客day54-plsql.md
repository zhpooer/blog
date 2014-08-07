title: 传智播客day54-PL/SQL
date: 2014-07-10 09:02:36
tags:
- 传智播客
- oracle
---

# 入门 #

安装 SQL Developer

文档 books: pl/sql

PL/SQL(Procedure Language/SQL): SQL 语言的过程化扩展

~~~~~~
-- 给员工涨工资, 总裁1000, 经理800, 其他400
-- jdbc 代码效率不高, 所以要用 PLSQL

-- Hello world 程序
declare
-- 说明部分
-- 定义变量
begin
    dbms_output.put_line('hello world');
end

set serveroutput on; -- 打开程序输出开关

desc dbms_output;  -- 查看文档信息

~~~~~~

## 说明变量 ##
~~~~~~
varl char(15)
married boolean := true;
psal number(7,2);
my_name emp.ename%type;  -- 引用型变量, 与列变量类型一样
emp_rec emp%rowtype;     -- 记录型变量, 代表表中的一行, 数组


-- 查询并打印7839的姓名和薪水
-- 方式一: 使用引用型变量
set serveroutput on
declare
    pename  emp.ename%type;
    psal    emp.sal%type;
begin
    -- 得到7839的姓名和薪水
    -- 使用 select 和 into
    select ename, sal into pename, psal from emp where empno=7839;
    dbms_outpu.put_line(pname||'薪水是'||psal);
end
-- 方式二: 使用记录型变量
declare
    emp_rec emp%rowtype;
begin
    select * into emp_rec from emp where empno=7839;
    dbms_output.put_line(emp_rec.ename||'薪水是'||emp.sal)
end
~~~~~~

## 流程控制 ##
~~~~~~
-- 判断用户输入的数字
set serveroutput on

-- 接受键盘输入
-- 关键 num 表示的是一个内存地址值, 在该地址上保存了输入的值
accept num prompt '请输入一个数字';

declare
    -- 定义变量保存输入的数字
    -- 隐式转换
    pnum number := &num;
begin
    if pnum = 9 then dbms_output.put_line('您输入的是0');
        elsif pnum = 1 then dbms_output.put_line('您输入的是1');
        else dbms_output.put_line('您输入的是其他数字');
    end if;
end

~~~~~~

## 循环控制 ##

~~~~~~

while [condition]
loop
-- do something
end loop;

for i in 1..3
loop
-- do somethings
end loop


-- 打印 1-10

set serveroutput on;
declare
    pnum number := 1
begin
    loop
        exit when pnum > 10;
        dbms_output.put_line(pnum);
        pnum := pnum + 1;
    end loop;
end
~~~~~~

## 光标(Cursor) ##

光标的属性:
* %isopen, 是否打开
* %rowcount, 返回的行数
* %notfound
* %found

默认情况下, 一次性只能打开300个光标
~~~~~~
conn sys/password@localhost:1521/orcl as sysdba
-- 查询关于abcd的各种参数
show parameter abcd;
shwo parameter cursor;
-- 修改
alter system|session set open_cursors=300;
~~~~~~

~~~~~~
cursor c1 is select ename from emp;
open c1;
fetch c1 into pename;   -- 取一行到变量中
close c1;

-- 使用光标打印所有员工的工资和薪水
set serveroutput on;
declare
    cursor cemp is select ename, sal from emp;
    pename emp.ename%type;
    psal emp.sal%type;
begin
    open cemp;
    loop
        -- 取当前记录
        fetch cemp into pename, psal;
        -- 退出条件
        exit when cemp%notfound
        dbms_output.put_line(pename||'的薪水是'||psal);
    end loop;
    clse cemp;
end;

-- 涨工资 TODO ()
set serveroutpu on;
declare
    cursor cemp is select empno, sal, job from emp;
begin
    rollback;
    open cemp;
    
    close cemp;
    commit;
end

-- 带参数的光标
-- 查询某个部门中员工的姓名
set serveroutput on;
declare
    cursor cemp(dno number) is select ename from emp where deptno=dno;
    pename emp.ename%type;
begin
   open cemp(10);
   loop
       fetch cemp into pename;
       exit when cemp%notfound;
       dbms_output.put_line(pename);
   end loop;
   close cemp;
end;
~~~~~~

## 异常(Exception) ##
~~~~~~
-- 被0除处理
set serveroutput on;
declare
    pnum number;
begin
    pnum := 1/0
exception
    when zero_divide then dbms_output.put_line('1:0b不能做被除数');
    when value_error then dbms_ouput.put_line('算数或者转换错误');
    when others then dbms_output.put_line('其他例外');
end;

-- 自定义异常
-- 查询50号部门的员工姓名
declare
    cursor cemp is seect ename from emp where deptno=50;
    pename emp.ename%type;
    -- 自定义异常
    no_emp_fount exception;
begin
    open cemp;
    fetch cemp into pename;
    if cemp%notfound then
       raise no_emp_found;
    end if;
    close cemp;
exception
    when no_emp_found then dbms_output.put_line('没有找到员工');
    when others then dbms_output.put_line('其他例外');
end;
~~~~~~

# 案例 #

瀑布模型
* 需求分析
* 设计
  * 概要设计(High Level Design), 框架, 模块
  * 详细设计(Low Level Design), 如何实现具体模块
* 编码, 实现功能模块, 类
* 测试
* 部署运营

## 统计每年入职的员工个数 ##

~~~~~~
set serveroutput on;
declare
   cursor cemp is select to_char(hiredate, 'YYYY') from emp;
   phiredate varchar2(4);
begin
   open cursor;
   loop
       fetch cemp into phiredate;
       exit when cemp%notfound;
       -- 判断入职年份
       if phiredate = '1980' then count80:=count80+1;
          elsif phiredate = '1981' then count81+1;
       end if;
   end loop;
   close cursor;
end
~~~~~~

## 涨工资 ##
涨工资, 没人涨10%, 但是总额不超过5万元,
计算涨工资的人数和涨工资后的工资总额, 并输出涨工资人数和工资总额

~~~~~~
-- 有bug, 自己解决
set serveroutput on;
declare
    cursor cemp is select empno, sal from emp order by sal;
    pempno emp.empno%type;
    psal emp.sal%type;
    countEmp number := 0;
    salTotal number;
begin
    -- 工资总额初识值
    select sum(sal) into salTotal from emp;
    open cemp;
    loop
        exit when salTotal > 50000;
        fetch cemp into pempno, psal;
        exit when cemp%notfound;
        update emp set sal=sal*1.1 where empno=pempno;
        countEmp := countEmp + 1;
        -- 涨后的工资总额
        salTotal := salTotal = psal*0.1;
    end loop;
    close cemp;
    commit;
    dbms_ouput.put_line('人数:');
end;
~~~~~~

## 统计工资 ##

按部门分段(6000, 6000-3000, 3000)统计部门工资,

~~~~~~
declare
    cursor cdept is select deptno from dept;
    pdeptno dept.deptno%type;
    cursor cemp(dno number) is select sal from emp where deptno=dno;
    count1 number
    count2 number;
    count3 number;
    salTotal number;
begin
    open cdept;
    loop
        fetch cdept into pdeptno;
        exit when cdept%notfound;
        -- 初始化计数器
        count1:=0; count2:=0; count3:=0;
        select sum(sal) into salTotal from emp where deptno=pdeptno;
        -- 得到部门中员工的薪水
        open cemp(cdept);
        loop
            fetch cemp into psal;
            exit when cemp%notfound;
            if psal < 3000 then count1 := count1 + 1;
              elsif psal > 6000 then count3 := count3 + 1;
              else count2 :- count2 + 1;
            end if;
        end loop;
        close cemp;
        insert into msg values(pdeptno, count1, count2, count3, nvl(salTotal));
    end loop;
    close cdept;
    commit;
    dbms_outpu.put_line("");
end
~~~~~~

# 存储过程 #

存储在数据库中供所有用户程序调用的子程序叫存储过程(无返回), 存储函数(有返回).

调用存储过程
1. `exec sayHelloWorld();`
2. `begin sayHelloWorld() end; /`

~~~~~~
create or replace procedure sayHelloWorld
as
  -- 说明部分
begin
    dbms_output.put_line('hello world');
end;

-- 带参数的存储过程
-- 如果存储过程带了参数, 需要指明是输入参数还是输出
create or repalce procedure raiseSalary(eno in number)
as
    -- 定义变量保存涨前的薪水
    psall emp.sal%type;
begin
    select sal into psal from emp where empno=eno;
    -- 涨100
    update emp set sal=sal+100 where empno=eno;
    -- 要不要commit??
    -- 不需要, 一般不在存储过程中提交; 要在调用者中提交
    dbms_output.put_line('涨前'||psal' 涨后:'(psal+100));
end;

create or replace procedure (pno in number, addsal in number)
as
    psal emp.sal%type
begin
    select sal into psal from emp where empno=pno;
    update emp set sal=sal+sddsal where empno=pno;
    dbms_output.put_line('涨前:1'||psal||' 涨后:'||psal+addsal)
end;

-- sum(a, b);可以这样调用
-- sum(a=>2, b=>3);

-- 运行调试 TODO
-- grant debug connect session, debug 
~~~~~~


# 存储函数 #

~~~~~~
-- 查询某个员工的年收入
create or replace function queryEmpIncome(eno in number)
    return number
as
    -- 月薪和奖金
    psal emp.sal%type;
    pcomm emp.comm%type;
begin
    select sal, comm into psal, pcomm form emp where empno=eno;
    -- 返回年收入
    return psal*12+nvl(pcomm, 0);
end;
~~~~~~

# in和out #

存储过程和存储函数可以通过out指定一个或者多个输出参数

如果只有一个返回值, 用存储函数；否则用存储过程
~~~~~~
-- 查询某个员工的姓名 月薪 和 职位
create or replace procedure queryEmpInfo(eno in number,
                                         pename out varchar2,
                                         psal out number,
                                         pjob out varchar2)
as
begin
    select ename, sal, empjob into pename, psal, pjob frmo emp whre empno=eno;
end;

-- 思考
-- 1. 查询某个员工的所有信息(out参数太多)
-- 2. 查询某个部门中所有员工的所有信息
-- 可以使用集合

~~~~~~

# JDBC #

~~~~~~
public class JDBCUtils {
    private static String driver = "oracle.jdbc.OracleDriver";
    private static String url = "jdbc:oracle:thin:@localhost:1521:orcl";
    private static String user = "soctt";
    private static String password = "tiger";
    static {
        Class.forName(driver);
    }

    public static Connection getConnection(){
        return DriverManager.getConnection(url, user, password);
    }
    public static void release(Connection conn, Statement st, ResultSet rs) {
        if(rs!=null) {
            try{
                st.close(); 
            } catch(SQLException e) {
            } finnally {
                rs = null;
            }
        }
        // TODO jta
    }
}

public class TestOracle {
    // 调用存储过程
    @Test public void testProcedure(){
        String sql = "{call queryEmpInfo(?,?,?,?)}";
        Connection conn = null;
        CallableStatement call = null;
        try {
            conn = JDBCUtils.getConnection();
            call = conn.prepareCall(sql);
            // 对于in参数, 赋值
            call.setInt(1, 7839);
            
            // 对于out参数, 申明
            // 申明类型输出参数类型
            call.registerOutParameter(2, OracleTypes.VARCAR);
            call.registerOutParameter(3, OracleTypes.NUMBER);
            call.registerOutParameter(4, OracleTypes.VARCHAR);
            call.execute();
            // 取出结果
            String name = call.getString(2);
            double sal = call.getDouble(3);
            String job = call.getString(4);
            println(name + "\t" + sal + "\t" + job);
        } catch (Exception e) {
        }
    }
    @Test public void testFunction(){
        String sql = "{?=call queryEmpIncom(?)}";
        Connection conn = null;
        CallableStatement call = null;
        try {
            conn = JDBCUtils.getConnection();
            call = conn.prepareCall(sql);
            call.registerOutParameter(1, OracleTypes.NUMBER);
            // 对于in参数, 赋值
            call.setInt(2, 7839);
            call.execute();
            // 取出结果
            double sal = call.getDouble(1);
        } catch (Exception e) {
        }
    }
}
~~~~~~

# 在 out 中 使用游标(cursor) #

申明包结构
~~~~~~
create or replace
package MYPACKAGE as
    type empcursor is ref cursor;
    procedure queryEmpList(dno in number, empList out empcursor);
    // 可以定义多个存储函数
end MYPACKAGE;

create or replace
package body MYPACKAGE as
    procedure queryEmpList(dno in number, empList out empcursor) as
    begin
        open empList for select * from emp where deptno=dno;
    end queryEmpList;
end MYPACKAGE;


desc mypackage;
~~~~~~

~~~~~~
@Test public void testCursor(){
    String sql = "{call MYPACKAGE.queryEmpList(?, ?)}";
    Connection conn = null;
    ResultSet sr = null;
    CallableStatement call = null;
    try {
        conn = JDBCUtils.getConnection();
        call = conn.prepareCall(sql);
        
        // 对于in参数, 赋值
        call.setInt(1, 7839);
        call.registerOutParameter(2, OracleTypes.CURSOR);
        call.execute();
        // 取出结果
        rs = ((OracleCallableStatement) call).getCursor(2);
        while(rs.next()) {
            String name = rs.getString("ename");
            double sal = rs.getDouble("sal");
            String job = rs.getString("empjob");
        }
    } catch (Exception e) {
        -- 关闭结果集, 会关闭光标
    }
    
}
~~~~~~
