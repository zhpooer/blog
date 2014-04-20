title: 传智播客day15-登陆案例
date: 2014-04-19 09:13:03
tags:
- 传智播客
---

# 用户注册和登陆案例 #

1. 技术架构: 三层架构(表现层MVC)
    
2. 要求: jap中不能出现一行java脚本或jiava表达式,出了指令<%@ %>
3. 数据库: 临时使用xml, 解析使用Dom4j
4. 必须知道要干什么
5. 开发步骤

    1. 建立工程, 搭建开发环境(dom4j, jaxen, commons-beanutils )
    2. 建立类所用的包    
      * cn.itcast.domain: 放javaBean
      * 弄出数据库. cn.itcast.dao: 放dao接口
      * cn.itcast.dao.impl: 放Dao接口的实现
      * cn.itcast.service: 业务接口
      * cn.itcast.service.impl: 业务接口实现
      * cn.itcast.comtroller: Servlet 控制器
      * /WEB-INF/pages: 放jsp, 用户无法访问(只能靠转发)
      * 把握两点, 1.domain中的javabean; 2.业务接口

~~~~~~
<users>
  <user username="**" password="**" email="**" birthday="2022-12-12">
  </user>
</users>

~~~~~~
