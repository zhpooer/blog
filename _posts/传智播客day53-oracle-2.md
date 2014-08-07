title: 传智播客day53-oracle 2
date: 2014-07-08 19:06:52
tags:
- 传智播客
---

# 子查询 #
解决不能一步求解的问题

注意问题
1. 括号
2. 合理的书写风格
3. 可以在主查询的 where select from having 后面放子查询
4. 不可以在主查询的 group by后面放置子查询
5. 强调from后面的子查询
6. 主查询和子查询可以不是同一张表, 只要子查询返回结果
7. 一般不在子查询中使用 order by, 但在 Top-N 分析问题中使用排序
8. 一般先执行子查询, 在执行主查询；但相关子查询除外
9. 单行子查询只能使用单行操作符, 多行子查询只能使用多行操作符
10. 子查询中的null

~~~~~~
select * from emp where sal > (select sal from emp where ename='scott');

-- 单行子查询, 只返回一条记录
select ename, sal, (select job from emp where empno=7389) from emp;

-- 强调 from 后面的子查询
-- 查询员工的姓名和薪水
select * from (select ename, sal from emp);
-- 查询员工的姓名和薪水,年薪
select * from (select ename, sal, sal*12 annlsal from emp);

-- 6. 主查询和子查询可以不是同一张表, 只要子查询返回结果
-- 查询部门名称为sales的员工信息
select * from emp where deptno =
    (select depton from dept where dname='sales');
-- 也可以使用多表查询
-- 优化5: 理论上, 尽量使用多表查询

-- 9. 单行子查询只能使用单行操作符, 多行子查询只能使用多行操作符
-- 单行操作符: > = <
select department_id, min(salary)
from employees
group by department_id
having min(salary) >
    (select min(salary) from employees where department_id=50);

-- 多行比较操作: in 在集合中
-- 查询部门名称是 sales 和 accounting 的员工
select * from emp where deptno in
    (select deptno from dept where dname='sales' or dname='accounting');
    
-- any: 在集合中的任意一个值比较
-- 查询工资比30号部门任意一个员工高的部门( > ( min() ))
select * from emp where sal > any (select sal from emp where deptno=30);

-- all: 和集合的所有值比较
-- 查询工资比30号部门所有工资高的员工信息( > ( max() ))
select * from emp where sal > all (select sal from emp where deptno=30);


-- 10. 子查询中的null
-- 多行中的null

-- 是老板的员工
select * from emp where empno in (select mgr from emp);
-- 不是老板的员工, not in (10, 20, null)
-- not in 意思是 不等于 所有, a!=null 永远为假
-- in 操作符 等同于 any
select * from emp where empno not in (select mgr from emp where mgr is not null);
~~~~~~

# 集合运算 #

操作两个或者多个集合
* 并集: UNION/UNION ALL(相同的部分出现两次)
* 交集: INTERSECT
* 差集: MINUS

注意问题
1. 参与运算的各个集合必须列数相同且类型一致
2. 采用第一个集合的表头作为最后的表头
3. 如果排序, 必须在每个排序后使用相同的 order by

~~~~~~
select * from emp where deptno=10
union
select * from emp where deptno=20;

select deptno, job, sum(sal) from emp group by rollup(deptno, job);
-- 等同于
select deptno, job, sum(sal) from emp group by deptno, job
union
select deptno, to_char(null), sum(sal) group by deptno
union
select to_number(null), to_char(null), sum(sal) from deptno;

break on deptno skip 2;

-- 优化6, 尽量不要使用集合运算 

-- 打开记录语句执行时间
set timing on
~~~~~~

# 课堂练习 #

~~~~~~
-- rownum 行号, 伪列
-- 1. 按照默认的顺序生成, 从小到达
-- 2. rownum 只能使用 <=, <, 不能使用 >, >=, 行号永远从1开始(像是 iterator )
-- 下面这条语句是错误的!!!!, 行号在 order by 后会乱序
select rownum, empno, ename, sal from emp where rownum<=3 order by sal desc;

-- 一: 找到员工表中工资最高的前三名, 分页显示可以这样用, 可以把里层的 rownum 取别名
select rownum, empno, ename, sal
     (select empno, ename, sal from emp order by sal desc) where rownum<=3 ;

-- 二: 找到员工表中薪水大于本部门的薪水员工
select e.empno, e.ename, e.sal, d.avgsal
from emp e, (select deptno, avg(sal) avgsal from emp group by deptno) d
where e.deptno=d.deptno and s.sal>d.avgsal;

-- 相关子查询, 将主查询中的某个值, 作为参数传递给子查询
select empno, ename, sal, (select avg(sal) from emp where deptno=e.deptno) avgsal
from emp e
where sal > (select avg(sal) from emp where deptno=e.deptno);

-- 三, 统计每年入职的员工个数(不使用子查询)

select count(*) Total, sum(decode(to_char(hiredate, 'RR'), '81'),1,0),
sum(decode(to_char(hiredate, 'RR'), '82'),1,0) from emp;

-- 四. 相关子查询


-- 五 1 => 1,2,3,4; 1 =>张三, 2=> 李四..; 1 => 张三, 李四
-- 组函数 wm_concat
-- select deptno, wm_concat(ename) names from emp group by deptno;
set linesize 200;

select ci_id, wm_concat(stu_name) names
from (
    select ci_id, stu_name
    from pm_stu s, pm_ci c
    where instr(c.stu_ids, s.stu_id) > 0
) group by ci_id;

~~~~~~


# 处理数据 #

SQL 的类型
1. DML(数据操作语句) : insert update delete select
2. DDL(数据定义语句) : create alter drop truncate 
3. DCL(数据控制语言) : grant revoke

## insert ##

~~~~~~
insert into emp(empno, ename, sal, dptno) values (100, 'finance', NULL, NULL);

-- 地址符 &, PreparedStatement
insert into emp(empno, ename, sal, deptno) values (&empno, &ename, &sal, &deptno);

select empno, ename, &t from emp;

select * from &t;

-- 批处理
create table emp10 as select * from emp where 1=2;
-- 一次性将emp中所有10号部门的员工插入到emp10
-- 将insert中加入子查询
insert into emp10
  select * from emp where deptno=10;

-- 如果插入海量数据, 如何
-- 1. 数据bang (data dump)
-- 2. SQL*LOADER
-- 3. 外部表(External table)

~~~~~~

## update ##

## delete & truncate ##
delete和truncate区别
1. delete 逐条删除; truncate先摧毁表, 再重建
2. delete是DML(可以回滚), truncate是DDL(不可以回滚)
3. delete不会释放空间, truncate 会
4. delete会产生碎片, truncate不会; 碎片会影响查询数据
        去掉碎片
        1. alter table <表名> move; -- 如果数据庞大, 会消耗大量时间
        2. 数据导入和导出
5. delete可以闪回(被提交了,反悔), truncate不可以

Oracle 中 delete 比 truncate 快

~~~~~~
-- 导入外部数据
set feedback off; -- 关闭回显
@d:\test.sql      -- 导入
-- 开启运行时间回显
set timing on;
~~~~~~

# Oracle 中的事务 #
1. 起始标志: 该事务中的第一条DML语句
2. 结束标志
        提交: 
          1. 显式, commit
          2. 隐式, 正常退出exit或DDL, DCL
        回滚:
          1. 显式, rollback
          2. 隐式, 非正常退出

## 控制事务 ##

保存点(savepoint)
~~~~~~
create table testsavepoint
(tid number
tname varchar2(20));
set feedback on;
insert into testsavepoint values(1, 'tom');
insert into testsavepoint values(2, mary');
-- 定义保存点
savepoint a;
-- some operator
rollback to savepoint a;

~~~~~~

## 数据的隔离级别 ##
Oracle 只支持四个隔离级别中的两个
1. READ COMMITED, 读已经提交事务
2. SERIALIZABLE, 串行化

额外提供级别: `READONLY`
~~~~~~
-- 设置只读
set transaction read only;
~~~~~~

# DDL 语句 #
管理数据库的对象

必须条件:
1. `create table` 权限
2. 存储空间

~~~~~~
create table test1(
    tid number,
    tname varchar2(20),
    hiredate date default sysdate
);

-- rowid 行地址, 伪列, 相当于指针(指向数据)
select rowid, empno, ename from emp;

-- 用子查询创建表
-- 用一个永远为假的条件创建一个新的空表
create table emp10
as
select * from emp where 1=2;

-- 修改表
-- 追加新列
alter talbe test1 add photo blob;
-- 修改列
alter table test1 modify tname varchar2(40);
-- 删除列
alter table test1 drop column tname;
-- 重命名列
alter table test1 rename column tname to username;

-- 重命名表
rename test1 to test2;

-- 删除表
select * from tab;
drop table test2;  -- 并不是真的删除, 只是放到回收站

-- oracle的回收站
-- 查看
show recyclebin;
-- 清空
purge recyclebin;

-- 不是每个用户都由回收站, 管理员没有回收站

-- 以管理员方式登陆
-- 以密码认证登陆
sqlpus sys/password as sysdba;
-- 主机认证登陆
sqlplus / as sysdba
~~~~~~

## 约束 ##

类型
* CHECK, 检查性约束
~~~~~~
create table test3 (
tid number,
tname varchar2(20),
-- 给约束起名字
gender varchar2(2)
    constraint emp_gender 
    check (gender in('男', 女)),
salary number check (salary > 0) 
);

~~~~~~
* NOT NULL,
* UNIQUE,
* PRIMARY KEY,
* FOREIGN KEY, 主表的外键必须是附表的主键
~~~~~~
on delete cascade; 删除父表时, 级联删除子表记录
on delete set null; 删除父表时, 相关外键记录设子为 null
~~~~~~


~~~~~~
create table student(
    sid number constraint student_PK primary key,
    sname varchar2(20) constraint studnet_namenotnull not null,
    gender varcher2(2) constraint student_gender_check (gender in('男', '女')),
    email varchar2(40) constraint student_email_unique unique
                       constraint student_email_notnull not null,
    deptno number constraint student_FK refereces dept(deptno) on delete set null;
);
-- 可以把约束和表的创建分开
~~~~~~

## 视图(view) ##

从表中抽出的逻辑上相关的数据集合,
视图是基于表创建, 理解为存储起来的 select 语句,
*为了简化复杂查询*, 但是不能提高性能

~~~~~~
-- 需要 create view 权限
grant create view to scott;

-- 创建视图
create view empinfo
as
select * from emp

create or replace view  empinfo
as
select * from emp
with read only;

create view view10
as
select * from emp where deptno=10
with check option;
-- with check option 会使下面语句不成功
insert into view10 values(****, 20);

-- 不建议通过视图对表进行修改
-- 通过视图修改数据有限制!!!!

-- 删除视图, 只是删除视图的定义, 不会删除表的数据
drop view viewname;
~~~~~~


## 序列(sequence) ##

可供多个用户用来产生唯一数值的数据库对象
(内存中的数组,默认长度20), 可以提高访问效率
~~~~~~
create sequence myseq
increment by 2
start with 2
maxvalue 10000
minvalue 1
[cycle|nocycle]
[cache 20 | nocache];

create sequence myseql increment by 5 nochace;
create table testseq(
    tid number,
    tname varchar2(20)
);
-- nextval 必须在 currval 之前
select myseql.nextval from dual;
select myseql.currval from dual;

insert into testseq values(myseq.nextval, 'aa');

-- 修改序列
-- alter sequence ....;
~~~~~~

序列在某些情况下会出现裂缝
* 回滚
* 系统异常
* 多个表同时使用同一序列

## 索引(index) ##

通过指针加速查询

Oracle中的索引
1. B树索引, 默认
2. 位图索引

什么时候建立索引
1. 列中数据值分布范围很广
2. 列经常在where子句或连接条件中出现
3. 表经常被访问且数据量很大


不适合建立索引
1. 经常更新
2. 表非常小

~~~~~~
-- 基于部门号创建索引表, 存储 rowid
create index myindex on emp(deptno);

-- 删除
drop index myindex;
~~~~~~

## 同义词, 别名 ##
~~~~~~
select count(*) from hr.employees;

grant select on hr.employees to scott;

-- 为 hr.employees 取别名
grant create synonym to scott;
create synonym hremp for hr.employees;
-- 共有同义词
create public synonym hremp for hr.employees;

select * from hremp;
~~~~~~

# 触发器 #

数据库触发器是一个表相关的, 存储的 PL/SQL程序,
每当一个特定你的数据操作语句(Insert, update, delete)在指定的表上发出时,
Oracle 在动地执行触发器中定义的语句序列

触发器可用于:
* 数据确认
* 实施复杂的安全性检查
* 做审计(日志), 跟踪表上所作的数据操作等
* 数据的备份和同步

~~~~~~
-- 每当插入语句之后, 打印一条语句
create or replace trigger sayNewEmp
after insert on emp
declare
begin
    dbms_output.put_line('成功插入语句');
end;

create [or replace] trigger
{before | after}
{delete | insert | update [of 列名]}
on 表名
[for each row [when 条件]] -- 开始行级触发器
end
~~~~~~

触发器的类型
* 语句级触发器, 针对的是表
* 行级触发器, 针对的是行, 提供伪列 :new :old

~~~~~~
-- 语句级触发一次
-- 行级触发器触发n次
insert into emp10 select * from emp where emp=10;
~~~~~~

## 触发器应用 ##

实施复杂安全性检查:
禁止在非工作时间插入新员工

~~~~~~
-- 周末: to_char(sysdate, 'day') in ('星期六', '星期日')
-- 上班前,  to_number(to_char(sysdate, 'hh24')) not between 9 and 18;
create or replace trigger securityemp
before insert
on emp
begin
    if to_char(sysdate, 'day') in ('星期六', '星期日') or
       to_number(to_char(sysdate, 'hh24')) not between 9 and 18 then
       -- 错误代码必须在 20000 - 50000
       raize_application_error(-20001, '不能在非工作时间插入新员工')
    end if;
end;
~~~~~~


数据确认: 涨后的工资不能少于涨前的工资
~~~~~~
create or replace trigger checksal
before update
on emp
for each row 
begin
    if :new.sal < :old.sal then
        raise_application_error(-20002, ' 涨后的工资不能少于涨前的工资');
    end if;
end;
~~~~~~

# 数据字典 #

需要知道自己的权限, 自己的信息, 数据库本身的信息, 这些信息被放在 数据字典

* 基本表: 描述数据库信息
* 用户表: 用户自定义信息

~~~~~~
select * from dictionary; -- 描述基本表的信息

select * from user_tables;
~~~~~~

| 数据字典的命名规则 | 
|------|
| USER |  用户自己的 |
| ALL |  用户可以访问到的 |
| DBA |  管理员视图 |
| V$ |   性能相关数据 |

~~~~~~
-- 加注释
comment on table emp is '这是员工信息表'
-- 查看注释
select * from user_tab_comments where table_name='EMP';
~~~~~~

<!-- 老师号码: 13488899975 -->
