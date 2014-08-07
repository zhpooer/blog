title: 传智播客day61-医药集中采购平台 框架搭建
date: 2014-07-21 09:13:07
tags:
- 传智播客
---

# 项目背景 #

# 用户角色 #
监管单位: 卫生局, 卫生院
* 控制
* 监视

医院: 卫生室
TODO

供货商: 供货商

系统管理员

# 业务流程 #

角色的活动

# 功能模块 #

细化业务流程, 功能模块划分,
需求分析在项目当中, 要不断提炼, 维护

采购单管理
* 创建采购单
* 采购单维护
* 采购单递交
* 采购单审核
* 采购单受理
  * 发货
* 采购单入库

退货单管理
* 创建退货单
* 退货单维护
* 退货单递交
* 退货单受理

结算单管理
* 创建结算单
* 结算单维护
* 结算单递交
* 结算单受理

系统管理
* 角色管理
* 权限管理
* 用户管理
* 系统参数配置
* 系统日志
* 系统备份
* 数据字典维护
* 区域管理

统计分析
* 交易明细查询
* 按医院统计
* 按供货商统计
* 按药品统计


药品目录
* 维护药品目录
* 查询药品目录
* 医院采购目录
* 企业供货目录


# 用户需求 #


# 系统需求 #

得出 `需求规格说明书`, 开发人员文档

在需求不明确时, 使用原型开发模型

在需求非常明确, 周期不长, 使用瀑布模型: 需求, 设计, 编码

需求明确, 但是周期很大, 使用增量模型

# 项目开发 #

## 人员配置 ##
项目经理(1): 项目整体执行控制

架构师(1):
* 系统架构搭建
* 系统集成
* 技术预研

需求分析(2)
* 需求调研分析
* 比阿奴啊系统规格说明书

开发(5)
* 系统模块开发编码
* 单元测试
* 参与系统集成
* 配合系统测试, 修改bug
* 配合系统产品化工作

测试(3)
* 编写测试用例
* 系统集成测试
* 系统功能测试
* 系统性能测试

产品化(2)
* 制作系统安装包
* 制作系统升级包

系统(2)
* 系统安装调试
* 培训

## 开发周期 ##

瀑布模型
1. 需求分析
2. 设计
3. 实施
4. 测试

增量模型:
企业中常用的模型, 当系统很大需要几次开发完成, 每次开发做一个版本,
正常进行系统安装升级工作

原型模型


需求阶段 30 工作日  
开发阶段 90 工作日  
测试阶段 90 工作日  
部署阶段 30 工作日

# 生产环境 #

## oracle ##

~~~~~~
-- 新建表空间
create tablespace yycg0329;
-- oracle 用户名, 相当于 mysql 中的 数据库
-- 虚拟路径, 创建表空间, 
datafile `e:\oradata\yycgy\xxx.dbf`
size 32m
autoextend on
next 32m maxsize 2048m
extent management local;

-- 创建用户,并指定表空间
create user yycg0329 identified by yycg0329
default tablespace yycg0329
temporary tablespace tmp;

-- 授权
grant connect, resource, dba to yycg0329

-- 导入建表数据
@D:/sql.sql
~~~~~~

## 项目开发模块划分 ##

以技术框架纵向拆分, 技术层次清晰, 开发方便,
模块抽取困难(模块可重用性), 适用于小项目

    action
    service
    service.impl
    dao
    dao.mapper
    jsp/view

以业务模块横向拆分, 代码可重用性强, 代码的包多, 不易维护, 适用于中大型的项目.
模块耦合非常强, 工作时要将所有模块下载进行编译

    user.action
    user.service
    user.dao
    system.dao
    system.action
    system.service

通过maven解决模块耦合问题, 公司人力充足, 按模块分配人员, 适用于人力充足的公司
* 用户管理(maven工程), `user`
* 系统管理, `system`
* 工具类, `util`

## 本系统划分 ##

* base(基础模块), 系统管理 + business(业务管理), 药品目录, 采购单, 退货单, 统计分析
        db.propertyies
        spring/applictionContext.xml
        spring/springmvc-servlet.xml
        spring/applicationContext-base-service.xml
        spring/applicationContext-base-dao.xml
        mybatis/SqlMapConfig.xml
* util(工具工程)
* 技术架构模块(springmvc+mybatis), 引入springmvc 和 mybatis 的 jar 包

    yycg.base
      action
      service
      dao
    yycg.business
      action
      service
      dao

