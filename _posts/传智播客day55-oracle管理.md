title: 传智播客day55-Oracle管理
date: 2014-07-11 10:08:35
tags:
- 传智播客
- oracle
---

# DBCA #

DBCA 是一个管理Oracle数据库的工具
* ASM, 自动存储管理, 数据库文件在外设文件存储介质管理
* 创建数据库
* 配置数据库选项
* 删除数据库
* 管理模板

DBA工作:
* 评测数据库服务器硬件
* 安装Oracle数据库软件
* 规划数据库
  * 计划数据库的逻辑存储结构
    * 多少磁盘驱动器
    * 需要多少数据文件
    * 需要用多大的表空间
    * 专用存储的类型和尺寸, 哪种类型才信息将被存储
  * 数据库总设计
  * 数据库备份策略
* 创建并且打开数据库
* 数据库备份
* 注册用户
* 实现数据库计划
* 全库备份/增量备份
* 调整数据库性能

数据仓库, 分析, 一般只做查询

归档, 如果在非归档模式下,
数据库是不能做联机备份的,
只能脱机备份, 定期压缩备份
~~~~~~
arhive log list
~~~~~~

数据库四个状态
~~~~~~
shutdown
unmount
mount
open
~~~~~~

# 闪回(flashback) #

使用情景
* 错误的删除了数据, 并且commit
* 错误地删除了表`drop table`
* 如何获取表上的历史记录
* 如何撤销一个已经提交了的事务

闪回类型
* 闪回表, 将表会退到过去的一个时间上
* 闪回删除, 操作oracle回收站
* 闪回版本查询, 表上的历史记录
* 闪回事务查询, 回去一个 undo_sql
* 闪回数据库, 将数据库会退到过期的一个时间上
* 闪回归档日志

传统的恢复技术缓慢, 闪回命令很容易

## 闪回表 ##

将表中的数据快速恢复到过去的一个时间点, 使用到与撤销表空间相关的
undo信息

用户的表数据的修改操作, 都记录在撤销表空间中, 如某个操作在提交之后
被记录在撤销表空间中, 保留时间为900秒,  用户可以在900秒内对表进行闪回操纵

~~~~~~
show parameter undo;
-- 单位是秒
-- scope取值有三个:
-- memory(只改当前数据库, 重启恢复)
-- spfile(重启以后才生效)
-- both

alter system set undo_retention=1200 scope=both;


-- 赋予权限
grant flash any table to scott;

-- 系统该变号 scn
-- 得到时间SCN 222222
select to_char(sysdate, 'yyyy-mm-dd hh24:mi:ss:mm') 时间,
timestamp_to_scn(sysdate) SCN from dual;


-- do some insert
commit;

-- 必须先打开表的行移动
alter table flashback_table enable rowmovement;

flashback table falshback_table to SCN 222222;
-- 如何获取离该操作最近的一个时间点 或者 scn
~~~~~~

* 系统表不能被闪回
* 不能跨越 DDL操作(如`create table`)
* 会被写入警告日志
* 产生撤销和重做的数据

## 闪回删除 ##

实际上从系统的回收站将已经删除的对象, 恢复到删除之前的状态.
只对普通用户有效

~~~~~~
-- 回收站
show recyclebin;
-- 清空回收站
purge recyclebin;

-- 彻底删除表, 不经过回收站
drop table testtable purge;

-- 闪回删除
flashback table test1 to before drop;
-- 通过回收站中的名字闪回删除
flash back table "BIN$a343=$xxx" to before drop;

-- 如果如果存在重名的表, 会先闪回后删除的

-- 如果已经存在重复命名的表
flash back table "BIN$a343=$xxx" to before drop rename to test1new;
-- 放到回收站时, 默认会禁掉 触发器
flash back table "BIN$a343=$xxx" to before drop enable triggers;
~~~~~~

## 闪回版本查询 ##

~~~~~~
-- versions_xid 事务号
select vid, vname, versions_operation, versions_starttime, versions_endtime,
versions_xid
from versions_table
versions between timestamp minvalue and maxvalue;
~~~~~~


## 闪回事务查询 ##
闪回事务查询实际上是闪回版本查询的一个补充,
通过它可以审计某个事务甚至撤销一个已经提交的事务
~~~~~~
-- 事务视图, 从该视图中可以获取事务的历史操作记录以及撤销语句
desc flashback_transaction_query;

grant select any transaction to scott;

-- 通过闪回版本查询得到 versions_xid
select operation, undo_sql from flashback_transaction_query
where xid="xidxxxxx";
-- 执行 undo_sql, 就可以撤销这个事务
~~~~~~

# 导入和导出 #

~~~~~~
-- 表方式导出
cmd > exp scott/tiger file=d:/a.dmp log=d:/log.lgo tables=dept,emp
-- 用户方式导出
cmd > exp scott/tiger file=d:/a.dmp log=d:/log.lgo
-- 库方式导出
exp system/password@localhost:1521/orcl file=d:/temp/full.dmp log=d:/temp/log.log full=y

-- 导入一张表
cmd > imp emi/password file=d:/a.dmp log=d:/log.log table=dept,emp
fromuser=scott touser=emi commit=y ignore=y
-- 导入用户下的表
cmd > imp emi/password file=d:/a.dmp log=d:/log.log
fromuser=scott touser=emi commit=y ignore=y
-- 导入数据库
cmd > imp system/passowrd@localhost file=d:/cc.dmp log=d:/log.log full=y ignore=y destroy=y
~~~~~~

# 管理方案(Schema) #

启动 `OracleDBConsolord`, 访问 `http://locahost:1158`,
`grant select_catalog_role to scott`

方案: 用户和方案是一一对应的关系, 表, 存储过程以及View等都是管理方案

通过网页管理数据库

临时表

# 管理用户安全 #

概览
* 用户
* 角色
* 权限

~~~~~~
-- 创建用户 itcast0509/password
conn / as sysdba
-- 三种认证方式 password(密码验证)/external(以主机用户登陆)/global(生物认证,token方式)
-- 预定义账户: SYS账户(数据库拥有者), SYSTEM账户(DBA)
create user itcast0509 identified by password;
-- 授予登陆权限
grant create session to itcast0509;

-- 解锁
alter user scott account unlock;
-- 改密码
alter user scott identified by newpassword;
~~~~~~

用户权限由两种
* 系统权限, 允许用户执行对于数据库的特定操作, 如创建表, 创建用户等
* 对象权限, 允许用户访问和操作一个特定的对象, 如其他方案下的表的查询
~~~~~~
grant select on employee to scott;
~~~~~~


## 权限的级联 ##

ADMIN OPTION, 只对系统权限而言, 撤销权限不能级联撤销
~~~~~~

create user jeff identified by password;
create user emi identified by password;

grant create session to jeff;
grant create session to emi;

alter user jeff quota unlimited on users;
alter user emi quota unlimited on users;

-- 给予权限
grant create table to jeff with admin option;

-- login as jeff, jeff 授权给emi
grant create table to emi;

-- login as admin
-- 撤销了jeff的权限, 但是没有级联撤销emi的权限
revoke create table from jeff;
~~~~~~

GRANT OPTION, 只对对象权限而言, 会产生级联的效果

~~~~~~
-- login as scott
grant select on emp to jeff with grant option;

-- login as jeff
grant select on scott.emp to emi;

-- login as scott, 级联撤销权限
revoke select on emp from jeff;
~~~~~~



## 角色 (Role) ##

~~~~~~
-- 角色
create role hr_mgr;
create role hr_clerk;

grant create session to hr_clerk;

-- 继承关系
grant create table, hr_clerk to hr_mgr;

-- 将角色赋予给用户
grant hr_mgr to jeff;
~~~~~~

数据库已经预定义了各种常用角色:
* CONNECT, 登陆
* RESOURCE, 创建表, 触发器, 视图, 分配用户空间等权限

~~~~~~
-- 创建新用户
create user testuser identified by password;
grant connect, resource to testuser;
~~~~~~

# 概要文件(profile) #

* 定义用户创建规则, 如密码复杂度, 登陆失败处理, 密码修改策略,
* 以及对资源的使用控制


# 分布式数据库 #

物理上被存放在网络的多个节点上, 逻辑上是一个整体

独立性, 不必关系数据如何分割和存储, 只关心数据本身

分布操作, 创建数据库链路, 本地服务名
~~~~~~
-- 通过工具, 创建本地服务名配置文件 remoteorcl
-- 授予远程登陆权限
grant create database link to scott;
-- 创建数据库链路
create database link l2 connect to scott identified by
tiger using 'remoteorcl';

-- 分布式数据库查询
-- 查询员工信息, emp在远端, dept在本地
create synonym  remoteemp for emp@L2;

select ename, dname
from dept, remoteemp
where remoteemp.deptno=dept.deptno;

-- 建立远程表的本地视图
create view emp
as
    select * from emp1@L1
    union
    select * from emp2@L2;

select * from emp;

~~~~~~

## 分布式数据库跨节点更新 ##
快照, 定义快照维护关系表的异步副本.
主表修改后的指定时间内刷新副本, 用户主表修改少, 单查询查询的表.
定义在备份端

触发器, 同步完成, 定义在主数据库端

~~~~~~
-- 定义快照
create snapshot emp refresh
-- 第一次刷新刷新
start with sysdate
-- 到下个星期刷新一次
next next_day(sysdate, 'Monday')
as select * from emp@L1;


-- 定义触发器
-- 实现数据的同步备份
create ro replace trigger syncsal
after update
on emp
for each row
begin
   -- 将涨后的薪水备份到远端数据库中
   update remoteemp set sal=:new.sal where empno=:new.empno;
end;
~~~~~~
