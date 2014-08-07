title: 传智播客day52-oracle
date: 2014-07-08 09:03:35
tags:
- 传智播客
- oracle
---

# jdbc连接方式 #
~~~~~~
# 普通连接方式, 使用 sql develope
jdbc:oracle:thin:@localhost:1521:orcl
# 通过客户机, 连接oracle, 可用于用于集群
# 可以使用 pl/sql develope
jdbc:oracle:oci:@localhost:1521:orcl

# 网页管理员登陆 user: sys
# 从命令行登陆
sqlplus scott/tiger@localhost:1521/orcl
~~~~~~

# 基本概念 #
* 一个 oracle 服务器是一个数据管理系统(RDBMS), 提供完整的信息管理
* 有一个 Oracle 数据库(物理存在于硬盘上的文件)和多个 Oracle 实例组成(通过内存实例操作物理数据库)
* 可以多个内存实例(不同机器上集群RAC)操作同一个物理数据库

集群优点:
* 负载均衡(load balance)
* 失败迁移(fail over)

基本通行
* 内存实例通过同时操作系统读,写进程与物理数据库通信
* 两阶段提交, 客户端提交到 PGA(内存实例中), PGA提交到SGA(system global area), SGA最终提交到物理数据库

# Oracle 三级考试 #
* OCA:
  1. SQL
  2. 管理1
* OCP:
  1. 管理2
* OCM

# select语句 #

~~~~~~
-- spool 保留操作步骤到文本文件
spool d:\基本查询.txt
-- 执行操作内容
spool off   -- 关闭操作

show user;  -- 当前用户
select * from tab; -- 当前用户下的表
desc emp; -- 员工表的结构

-- 查询员工的所有记录
select * from from emp;
-- 设置行宽
set linesize 150;
-- 设置列宽
col ename format a8   -- a字符串, 8代表8位
cal sal for 9999  -- 四个数字
-- 执行上一条语句
/
-- 清屏
host cls

-- 改变上一条语句中的错误
change
2             -- 选择第二行
c /form/from  -- 改变
/             -- 执行

/* sql 空值: 是无效的, 未指定的值都为空值, 不是0, 也不是空格
1. 包含空值的表达式都为空, 可以使用 `nvl(a, b)` 函数
2. 空值永远不等于空
*/
-- 查询 年工资, 奖金, 年收入
-- 如果奖金为空, 年收入为空
select empno, sal*12, comm, sal*12+comm;
-- 应该这么写
select empno, sal*12, comm, sal*12+nvl(comm,0);

-- 空值永远不等于空
-- 这句话查询返回永远为空
select * from emp where comm=null;
-- 应该这样
select * from emp where comm is null;

-- 导出上一条语句到文件
ed

-- 别名
select empno as "员工号", ename "姓名", sal 月薪 from emp;

-- 去除重复, distinct 作用与后面所有的列
select distinct job form emp
~~~~~~

SQL优化
1. 尽量使用列名代替 `*`

# 连接符 #
连接符: `||`
~~~~~~
-- oracle 中 select 必须有 from 关键字
-- 所以为满足如下要求, dual 为 伪表, 
select concat('hello', 'world') from dual;

-- 查询员工信息, ***的薪水是***
select ename||'薪水是'|| sal from emp;

-- 字符串
~~~~~~
字符串: 表示一个字符, 数字, 日期
* 日期 只能用单引号表示

# SQL 和 SQL*Plus #
SQL命令, 一种语言, 标准语句, 不能缩写

    insert
    update
    delete
    select

plus: 一种环境, oracle特性, 能缩写

    edit        ed
    change      c
    description desc

`iSQL*plus`: SQL*PLUS 命令行的网页版功能, 只在 9i 和 10g 提供,
使用 sql-develope 提供类似功能
* `home1iSQL*Plus` 服务, 提供5560端口网页 `isql-plus` 服务
* `DBConsolerorcl`, 提供1158端口网页登陆


* 字符串大小写敏感
* 日期格式敏感
~~~~~~
-- 不符合格式, 是字符串
select * from emp where hiredate='1981-11-17';
-- 符合日期格式 DD-MON-RR
select * from emp where hiredate='17-11月-81';

-- 在数据字典中, 查看日期格式
select * from v$nls_parameters;
-- 改变日期格式
alter session|system set NLS_DATE_FORMATE='yyyy-mm-dd';
~~~~~~

赋值使用 `:=`

比较运算符
~~~~~~
-- 查询 薪水1000~2000的用户
-- between 2000 and 1000, 错误!!
select * from emp where sal between 1000 and 2000;

-- not in,如果集合含有 null, 不能使用 not in, 但是可以使用 in

-- like 模糊查询, 查询以S打头的员工
select * from emp wereh ename like 'S%';
-- 名字是四个空格
select * from emp wereh ename like '    ';

-- 查询名字中含有下划线的员工
select * from emp where ename like '%_%'; -- 错误, 代表任意长度的任意字符串
-- 使用转义字符
select * from emp where ename like '%\_%' escape '\';

-- Oracle 自动开始事务, 所有的操作都自动开启
~~~~~~


sql优化2, `where` 解析顺序, 从右往左, 尽量为假的放到右边
~~~~~~
-- 先 condition2
where condition1 and condition2
-- 先 conditiong1
where condition2 and condition1
~~~~~~

# 排序 #
~~~~~~
-- order by + 列, 表达式, 别名, 序号(下标从1开始)
-- 查询员工信息, 按照月薪排序
select * from emp order by sal asc;

select empno, ename, sal, sal*12 from emp order by 4 desc;

-- 多个列的排序
select * from emp order by deptno , sal desc;
select * from emp order by deptno desc, sal desc;

-- 查询员工信息, 按照奖金排序
-- null的排序, 升序 空值排在最后, 降序 控制排在前面

-- 指定所有的空值排在最后
select * from emp order by comm desc nulls last;

~~~~~~

# 单行函数 #
函数接受输入, 产生输出分为单行函数和多行函数

单行函数, 只对一行进行变换, 产生结果
* 字符
~~~~~~
-- 字符函数
select lower('HELLO WORLD') 转小写, upper('hello world') 大写, initcap('hello') 首字母大写 from dual;

-- substr(a,b),从a中 第b位开始取值
select substr('hello world', 3) from dual;
-- 第三位开始取, 取四位
select substr('hello world', 3, 4) from dual;

-- length 字符数, lengthb 字节数
select length("上海") 字符, lengthb("上海") 字节数 from dual;

-- instr(a, b), 在 a 中, 查找b, 找到返回下标, 否则返回0
select instr('hello world', '11') from dual;

-- lpad 左填充, rpad 右填充
select lpad('abcd', 10, '*'), rpad('abcd' 10, '*') from dual;

-- trim 去掉前后指定的字符
select trim('H' from 'Hello WorldH') from dual;

-- replace 替换
select replace('hello world', 'l', '*') from dual;
~~~~~~
* 数值
~~~~~~
-- round 四舍五入
select round(45.926, 2) from dual;  -- 45.93
select round(45.926, 1) from dual;  -- 45.9
select round(45.926, 0) from dual;  -- 46
select round(45.926, -1) from dual; -- 50
select round(45.926, -2) from dual; -- 0
-- trunc 截断
select trunc(45.926, 2) from dual;  -- 45.92
select trunc(45.926, 1) from dual;  -- 45.9
select trunc(45.926, 0) from dual;  -- 45
select trunc(45.926, -1) from dual; -- ??
select trunc(45.926, -2) from dual; -- ??
-- mod 求余数
~~~~~~
* 日期, date = date + time
~~~~~~
-- 系统时间 mysql:: select date();
select sysdate from dual;

-- 格式化显示时间
select to_char(sysdate, 'yyyy-mm-dd hh24:mi:ss') from dual;

-- 日期加上和减去一个数字结果仍然为数字, 但日期加日期没有意义
select (sysdate-1) 昨天, sysdate 今天, (sysdate+1) 明天 from dual;

-- 计算员工工龄, 天, 星期, 月, 年
select ename, hiredate, (sysdate-hiredate) 天, (sysdate-hiredate)/7 星期, (sysdate-hiredate)/30 月 from emp;

-- months_between
select months_between(sysdate, hiredate) from emp;
-- add_months
select months_between(sysdate, 78) from dual;
-- last_day, 日期所在月份的最后一天
select last_day(sysdate) from dual;
-- next_day, 指定日期的下一个日期
select next_day(sysdate,'星期二') from dual; -- 下个星期二
-- 可以使用 next_day 每个星期一自动备份数据

-- 四舍五入和截断
select round('25-JUL-95', 'MONTH') from dual; 
select round('25-JUL-95', 'YEAR') from dual;
select trunc('25-JUL-95', 'MONTH') from dual;
select trunc('25-JUL-95', 'YEAR') from dual;
~~~~~~
* 转换, 分为隐私(oracle自己完成)和显示
~~~~~~
select * from dept where deptno='5'; -- 隐式转换, ok

-- 转换函数
-- TO_CHAR
-- TO_NUMBER 反过来
-- TO_DATE

select to_char(sysdate, 'yyyy-mm-dd hh24:mi:ss"今天是"day') from dual;

-- 查询员工函数
select to_char(sal,'L9999.99') from emp;

~~~~~~
* 通用函数
~~~~~~
-- NVL(expr1, expr2), 当为空时, 返回被圈
select nvl(ename, 'no name') from dual; 
-- NVL2(a, b, c);  a 为空时, 返回c, 否则返回b
-- NULLIF(a, b)  当a=b时, 返回null, 否则返回a
-- coalesce(expr1, expr2, ..., exprn), 从左往右返回不为空的值
~~~~~~
* 条件表达式, `case`和`decode`(ORACLE)
~~~~~~
-- 给员工涨工资, 总裁 1000, 经理800, 其他 400
select ename, job, sal 涨前,
    case job when 'prresident' then sal + 1000
             when 'manager' then sal + 800
             else sal + 400
    end 涨后
from emp;
select ename, job, sal 涨前,
    decode(job, 'president', sal+1000
                 'manager', sal + 800
                 sal+400)
    涨后
from emp;
~~~~~~

| 日期的格式化 |
|-------|
| YYYY |  数字年 | 2001 |
| YEAR |   年的全称 | two Thousand year |
| MM   | 两位数字     | 08 |
| MONTH|  月的全称 | JULY |
| DY   |  三个字符串表示 | 星期二 |
| DAY  |天气的全称  | 星期二 |
| DD   |   数字    | 02  |

</br>

| 数字格式化 | 
|------|
| 9 | 一位数字 |
| 0 | 零 |
| $ | 美元符 |
| L | 本地货币福 |
| . | 小数点 |
| , | 千位符 |

oracle 数据备份演示
~~~~~~
rman target /
> backup database;
> recover database;
~~~~~~

# 多行函数 #

组函数
~~~~~~
select sum(sal) from emp;
select count(*) from emp;

-- 两条语句效果一样
-- 平均工资
select sum(sav)/count(*) , avg(sav)from emp;

-- 平均奖金, 第一个函数 和 后面两个结果不一样
-- count(comm) 奖金不为空
select sum(comm)/count(*), sum(comm)/count(comm), avg(comm) form emp;

-- 组函数自动滤空;可以嵌套虑空函数, 屏蔽过滤
select count(nvl(comm,0)) from emp;

select count(distinct comm) from emp
-- max
-- min
~~~~~~

分组数据
~~~~~~
-- 每个部门的平均工资
select deptno, avg(sal) from emp group by deptno;

-- 在select 列表中, 所有未包含在组函数的列必须包含在 group by 中
select deptno, job, avg(sal) from emp group by deptno, job;

-- 求平均工资大于
-- where 和 having 最大的区别是 where 不能存在组函数
select deptno, avg(sal) from emp group by deptno having avg > 1000;

-- group by 语句增强
select deptno, job, sum(sal) emp group by deptno, job;
select deptno, sum(sal) emp group by deptno;
select sum(sal) from dept;
-- 合成这三句, 分类汇总
select deptno, job, sum(sal) from emp group by rollup(deptno, jo)
-- group rollup(a, b) == group by a, b + group by a , group by null
-- 对上面的语句, 设置显示模式
break on deptno skip 2;
break on null;
~~~~~~

# 多表查询 #

~~~~~~
-- 等值连接
-- 查询员工信息, 员工号, 姓名, 月薪, 部门名称
select e.empno, e.ename, e.sal, e.dname from emp e, dept d where e.deptno=d.deptno;
-- 不等值连接, 查询员工的级别
select e.empno, e.ename, e.sal, e.salgrade from emp e, salgrade s where s.sal beween s.losal and s.hisal;

-- 外连接
-- 按部门统计员工信息: 部门号, 部门名称, 人数

-- 错误查询语句,
select d.deptno, d.dname, count(e.empno) 
from emp e, dept d where e.deptno=d.deptno
group by d.deptno, d.dname;
-- 正确查询语句
-- 左外连接: where e.deptno=d.deptno(+)
-- 右外连接: where e.deptno(+)=d.deptno   (上面的语句要这这个语句)

-- 自连接
-- 查询员工信息: 员工姓名, 老板姓名
select e.ename 员工, b.emane 老板
from emp e, emp b
where e.mgr=b.empno;

-- 只要是多表查询, 就会产生笛卡尔积
-- 自连接不适合操作大表

-- 层次查询, 只有一张表, 是一个单表查询
-- 当我们查询数据满足一颗树, 可以用层次查询
-- prior: 上一层, start:从这个节点遍历子节点, 可以设置任意节点, 如果是全部节点, 选根节点
-- start with mgr is null; 层次提供 level 伪列
select level, empno, ename, sal, mgr from emp connect by prior empno=mgr start with empno=7566;
~~~~~~
