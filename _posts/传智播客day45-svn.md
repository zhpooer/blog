title: 传智播客day45 SVN
date: 2014-06-10 21:18:45
tags:
- 创智播客
---

# SVN简介 #

Maven: 项目管理工具, 完成项目构建

SVN (Subversion)版本管理工具, 属于 SCM 部分软件配置管理

项目中为什么要版本控制?
* 在多人团队同时进行项目开发, 每个人写好代码,
需要同步给其它人, 使用U盘工具, 容易出现版本更新不及时的问题
* 里程碑: 项目节点管理, 通过标签tag进行记录, 即时项目备份、恢复,
在项目升级 出现问题，很容易恢复
* 分支概念, 搭建基础架构, 对项目进行分支, 每个分支独立开发, 支持合并

# Subversion服务器 #

工作方式
1. 单独安装Subversion服务器程序,
直接使用客户端连接SVN Server(没有对用户名和密码加密，不安全)
2. Subversion服务器和Apache服务器整合, Apache负责密码加密(更加安全)

数据存储方式
* BDB (Berkeley DB), 数据库方式
* FSFS, 采用文件系统来存储(推荐)

SVN 管理版本文件, 就是硬盘上文件夹, 里面存放不同版本的文件
* 复制, 修改, 合并方案 (Subversion 服务器默认方式):
这种方案，允许多个人，对服务器副本文件，进行同时修改，在修改后，再合并 
* 锁定, 修改, 解锁方案: 只允许一个人修改文件

## 安装服务器 ##

1. 下载安装 Subversion

        svnadmin : 管理员创建仓库管理命令
        mod_dav_svn : 和Apache整合
        svnserve : 启动svn服务器
        svn : 执行svn客户端命令
2. 建立仓库, `svnadmin create itcast`, 会生成目录结构: 
  * conf下存放配置文件, authz授权文件、passwd 用户密码文件、svnserve.conf 核心配置
  * db 存放版本管理一些文件
  * hooks 钩子事件
  * locks 锁文件
3. 配置Subversion服务
~~~~~~
// 方式一
> svnserve -d –r 文档仓库路径
// 方式二 制作windows服务
> sc create SVN-Service binpath="\bin\svnserve.exe --service -r C:\Repositories\svn" displayname= "SVN-Service" start= auto depend=Tcpip
> net start SVN-Service // 启动服务
> net delete SVN-Service // 停止服务
~~~~~~
4. 客户端操作

        svn checkout -从版本库取出一个工作拷贝 
        svn commit -将改动的文件提交到版本库
        svn update -更新你的工作拷贝 
        svn add -向版本库中添加新文件
        svn delete -从版本库中删除文件
        svn revert -取消所有的本地编辑
        svn info -显示本地或远程条目的信息 
        svn list -列出版本库目录的条目 
        svn status -查看当前工作区状态
        svn help -获取帮助信息
        
~~~~~~
> svn checkout svn://localhost/itcast
> svn add index.jsp  ;; 在工作副本加入新的文件, 并通知服务器
> svn commit index.jsp -m "first commit" ;; 提交到中心服务器, 每一次提交版本加一
> svn update ;; 将本地的文件更新到最新文件
> svn revert ;; 取消本地副本的编辑
~~~~~~

## 权限配置 ##
~~~~~~
;;; senserve.conf
;; 匿名用户不能进行读写, 认证用户可以写入
anon-access = none
auth-access = write

;; 密码文件生效
password-db=passwd

;; 授权文件生效
authz-db = authz

;; 修改 passwd, 创建用户和密码
;; 修改 authz, 配置用户权限
~~~~~~

# TortoiseSVN #


# 冲突解决 #
多个人, 同时对一份代码进行修改, 再同时去提交产生问题
后提交用户, 出现文件过时错误, 服务器文件发生变化

解决方法: 修改时对文件上锁, 或: 

~~~~~~
// 在tortoise上操作
svn update
edit confilict
mark resolved
commit
~~~~~~

# eclipse svn 插件 #

# svn+apache整合配置 #

为什么 Subversion 整合 Apache
* 使用HTTP协议访问SVN服务
* Apache提供密码加密功能

1. 下载与svn相对应的apache安装程序, 安装之
2. 修改 httpd.conf, 复制 `mod_dav_svn.so` `mod_authz_zvn`
~~~~~~
;; 将下列2行前方的#移除
LoadModule dav_module modules/mod_dav.so
LoadModule dav_fs_module modules/mod_dav_fs.so
;; 并同时在上面两行下面增加以下两行
LoadModule dav_svn_module modules/mod_dav_svn.so
LoadModule authz_svn_module modules/mod_authz_svn.so
~~~~~~
3. 创建密码
~~~~~~
;; 复制产生的文件到 svn/conf
htpasswd -cb [filename] [username] [password]
~~~~~~
4. 修改httpd.conf, 将svn的服务映射为 http 服务
~~~~~~
#配置虚拟目录#
<location /svn/itcast>
#引用远程访问模块
DAV svn
#项目版本库路径#
SVNPath F:/software/repository/svn/itcast
#授权文件#
AuthzSVNAccessFile F:/software/repository/svn/itcast/conf/authz
#所有用户都需要身份验证#
Satisfy Any
Require valid-user
#验证方式#
AuthType Basic
#项目的名称#
AuthName "itcast"
#用户文件#
AuthUserFile F:/software/repository/svn/itcast/conf/passwd.apache
</location>
~~~~~~

## 快速整合 ##

可以下载集合包

# 分支, 标签, 切换, 合并使用 #
在实际企业开发中，SVN仓库中应该存在三个顶级目录
* `/trunk` 存放开发的“主线”
* `/branches` 存放支线副本
* `/tags` 存放标签副本

trunk: 存放主项目(项目主线)

tag: 在项目开发遇到里程碑(某些模块已经开发完后，记录当前时间 项目版本),
在企业开发中对于tag ，只读 不写
* 新建tag, 新建tag, 文件推送到 `TAGS`目录, 命名为 `***_v1.0`
* 如果发现之前版本存在bug, 切换到tag, 不在tag上开发，创建分支 `branch/***_v1.0`
* 切换到分支, 修改bug, 再打标签 v1.1 
* 切换到主线, 对分支进行合并 
