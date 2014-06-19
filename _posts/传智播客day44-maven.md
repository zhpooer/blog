title: 传智播客day44-maven
date: 2014-06-09 09:04:08
tags:
- 传智播客
- maven
---

# Maven技术简介 #
项目管理工具, 包含一个项目对象模型(Project Object Model), 一组标准集合,
一个项目生命周期(Lifecycle), 一个依赖管理系统(Dependency Managerment)
和用来运行定义在色和那个吗周期阶段(Phase)中的插件(Plugin)目标(goal)的逻辑

* Pom 项目管理周期
* 标准集合
* 项目生命周期
* 依赖管理系统
* 插件

开发思想: 约定优于配置

针对Java开发项目管理工具, 提供构建工具所提供功能的超集, 除了构建功能之外,
Maven还可以管理项目结构, 管理依赖关系

maven管理项目: 清理, 编译, 测试, 报告, 打包, 部署, 清理

# maven快速入门 #

目录结构
* `bin` 提供mvn命令
* `conf/setting.xml`, 提供核心配置文件

## 安装配置 ##

仓库, 用来管理打包后的项目

修改 `conf/settings.xml`, 配置maven
~~~~~~
<!-- 自定义本地仓库位置 -->
<localRepository> </localRepository>
~~~~~~

名词解释

| 名词  |  解释 |
|-----------------|
| Project | 何您想build的事物，Maven都可以认为它们是工程。这些工程被定义为工程对象模型(POM，Poject Object Model) |
| Pom   | 它是指示Maven如何工作的元数据文件,位于每个工程的根目录中 |
| GroupId | 组名(工程名), groupId是一个工程的在全局中唯一的标识符 |
| ArtifactId | 项目名称, artifact 是工程将要产生或需要使用的文件，它可以是jar文件, 每个artifact都由groupId和 artifactId组合的标识符唯一识别 |
| Dependency | 依赖 |
| plug-in   | 插件 |
| Repository | 仓库 |

~~~~~~shell
> mvn archetype:create -DgroupId=cn.itcast.maven.quickstart \
-DaritifactId=simple -DarichetypeArtifactId=maven-archetype-quickstart
~~~~~~

项目目录结构
* `pom.xml`, 位于工程根目录, 对项目进行配置
* `src/main/java`, 存放项目源码
* `src/test/java`, 存放测试源码

## 常用命令 ##
~~~~~~
> mvn compile  ;; 编译, 在工程目录生成target目录
> mvn test     ;; 运行测试
> mvn test -Dtest=${classname} ;; 测试运行单个类
> mvn clean    ;; 清除编译的结果, 删除target
> mvn package  ;; 打包
> mvn package –Dmaven.test.skip=true ;; 打包时不执行测试
> mvn install  ;; 打包后, 安装到本地仓库中 ${仓库根目录}/groupid/artifact/version/
> mvn deploy   ;; 发布到本地仓库或服务器(例如Tomcat、Jboss)
~~~~~~

生命周期，从上到下执行， 如果执行后面的命令，自动执行之前项目构建步骤

    validate
    generate-sources
    process-sources
    generate-resources
    process-resources     复制并处理资源文件，至目标目录，准备打包。
    compile     编译项目的源代码。
    process-classes
    generate-test-sources 
    process-test-sources
    generate-test-resources
    process-test-resources     复制并处理资源文件，至目标测试目录。
    test-compile     编译测试源代码。
    process-test-classes
    test     使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
    prepare-package
    package     接受编译好的代码，打包成可发布的格式，如 JAR 。
    pre-integration-test
    integration-test
    post-integration-test
    verify
    install     将包安装至本地仓库，以让其它项目依赖。
    deploy     将最终的包复制到远程的仓库，以让其它开发人员与项目共享。

### 转换其他工程项目 ##
转换Eclipse工程
> mvn eclipse:eclipse  
> mvn eclipse:clean  ;; 清除Eclipse设置信息

转换成IDEA工程
> mvn idea:idea  
> mvn idea:clean //清除idea设置信息

显示一个插件的详细信息(configuration, goals等):
> mvn help:describe -Dplugin=pluginName -Ddetail

# Eclipse基于maven构建 #
安装 m2eclipse, 更新配置文件, 点击 reindex对仓库中的项目重建索引

* 新建maven项目,选择archetype:maven-archetype-quickstart

## 建立web项目 ##

方式一: 选择archetype:maven-archetype-webapp,
* 需要在`src`下手动建立 `src/main/java`
* 使用 `tomcat-maven-plugin`, `mvn tomcat:run`, 运行tomcat

方式二: 跳过骨架选择, 使用默认的骨架, package 设置为 war
* 需要在 `src/main/webapp` 下建立 WEB-INF/web.xml
* 推荐： 跳过骨架选择方式，生成目录结构最完整

* `src/main/java` 存放项目源码
* `src/main/resources` 存放项目配置文件
* `src/test/java` 存放测试源码
* `src/test/resources` 存放测试配置文件
* `src/main/webapp` 存放页面代码 

# maven 配置详解 #

## 仓库 ##
maven项目管理, 依赖于三种仓库
* 本地仓库, 根据 settings.xml 配置, 可以缓存网络上的项目, 也可以自己的应用安装到仓库
* 远程仓库(网络上的仓库), 公司内部搭建服务器(私服)
* 中心仓库, 当本地项目 依赖其它项目, 而依赖的项目本地没有, 去网络上中央仓库找
  * Maven内置了远程公用仓库：http://repo1.maven.org/maven2
  * 这个公共仓库是由Maven自己维护, 里面有大量的常用类库,
  并包含了世界上大部分流行的开源项目构件.目前是以java为主.

中央仓库是最大网络远程仓库, 里面管理jar包,
也并不是企业开发全部(有些jar包需要到私服下载JBOSS私服、Spring 私服)

## POM 配置详解 ##

全景图
* 坐标: groupId、artifactId、version
* 聚合: modules 分模块
* 继承: parent、 dependencyManagement
* 依赖: dependences
* 工程信息： name、description、url
* 构建配置： properties、 packaging、 build 、reporting

用户编写maven项目，自定义 pom.xml 默认继承 "超级POM文件"

    自定义POM + 超级POM = 有效POM

~~~~~~
<project>
    <!-- 使用Pom对象版本4.0.0 -->
    <modelVersion>4.0.0</modelVersion>
    
    <!-- 当前项目的坐标, 唯一的锁定项目在仓库中的位置  -->
    <!-- 如果要寻找项目, 先要获得项目的坐标, 可以是 jar 或 war -->
    <groupId>io.zhpooer</groupId>
    <artifactId>mavendemo3</artifactId>
    <version>0.0.1-snapshot</version>
    
    <!-- Maven 构建项目默认提供常见的三种打包项目 -->
    <!-- * jar包, 简单java项目 -->
    <!-- * war包, web项目 -->
    <!-- * pom, 父项目，提供pom被子项目继承  -->
    <packageing>war</packageing>
    
    <!-- 如果使用tomcat:run 运行项目，访问项目通过 artifactId 属性 -->
    <!-- 在项目编译打包、安装部署，使用artifactId 属性 -->
    <!-- name属性用于生成文档  -->
    <name>demo</name>
</project>
~~~~~~

## 构建依赖 ##
在一个项目可以在 pom.xml 中 通过 `<dependencies>` 引入对其它项目的依赖

    坐标决定项目在仓库中唯一位置.
    通过坐标导入项目, 通过坐标导入jar包.

~~~~~~
<dependencies>
    <dependency>
        <!-- 坐标： groupId、artifactId、version, 找到jar位置 -->
        <groupId></groupId>
        <artifactId></artifactId>
        <version></version>
        <classifier></classifier>
        
        <!-- compile:编译范围,默认scope,在classpath中存在 -->
        <!-- provided:已提供范围,编译需要, 不会被打包, 比如容器提供Servlet API -->
        <!-- runtime:运行时范围,编译不需要,接口与实现分离 -->
        <!-- test:测试范围,单元测试环境需要 -->
        <!-- system:系统范围,自定义构件，指定systemPath -->
        <scope></scope>
        
        <!-- 导入项目类型：type 默认值 jar  -->
        <type></type>
        <systemPath></systemPath>
        
        <!-- 标记依赖是否可选, A->B->C，那么当A依赖于C时就可以设为可选 -->
        <optional></optional>
        
        <!-- 排除依赖： exclusions 某些依赖项目不想导入 -->
        <exclusions>
            <exclusion>
                <artifactId> </artifactId>
                <groupId> </groupId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
~~~~~~

## 传递性依赖 ##
如果两个jar包, 依赖不同版本的jar包, 传递依赖倒置jar版本冲突问题
* 第一原则：路径近者优先原则, x2.0传递给A最近
* 第二原则：第一声明者优先原则, 当路径相等时，则由POM声明的依赖顺序决定

解决方法
* 解决间接依赖最好方式就是直接依赖(直接声明)
* 锁定版本
~~~~~~
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.struts </groupId>
            <artifactId>struts2-core </artifactId>
            <version>${struts2.version} </version>
        </dependency>
    </dependencies>
</dependencyManagement>
~~~~~~

## 统一维护版本 ##
~~~~~~
<properties>
    <spring.version> 3.2.0.RELEASE </spring.version>
    <struts2.version> 2.3.15.2 </struts2.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.apache.struts </groupId>
        <artifactId>struts2-core </artifactId>
        <version>${struts2.version} </version>
    </dependency>
</dependencies>
~~~~~~

# 重构maven #
~~~~~~
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>cn.itcast.maven</groupId>
  <artifactId>mavenstore</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
  <name>mavenstore</name>
  <description>仓库管理系统</description>
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.6</source>
          <target>1.6</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
  <!-- 统一维护版本 -->
  <properties>
  	<spring.version>3.2.0.RELEASE</spring.version>
  	<struts2.version>2.3.15.2</struts2.version>
  	<hibernate.version>3.6.10.Final</hibernate.version>
  	<mysql.version>5.1.6</mysql.version>
  	<ehcache.version>2.6.6</ehcache.version>
  	<c3p0.version>0.9.1.2</c3p0.version>
  </properties>
  
  <dependencies>
  	<!-- spring -->
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-context</artifactId>
  		<version>${spring.version}</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-aspects</artifactId>
  		<version>${spring.version}</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-web</artifactId>
  		<version>${spring.version}</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-test</artifactId>
  		<version>${spring.version}</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-orm</artifactId>
  		<version>${spring.version}</version>
  	</dependency>
  	<!-- hibernate -->
  	<dependency>
  		<groupId>org.hibernate</groupId>
  		<artifactId>hibernate-core</artifactId>
  		<version>${hibernate.version}</version>
  	</dependency>
  	<!-- struts2 -->
  	<dependency>
  		<groupId>org.apache.struts</groupId>
  		<artifactId>struts2-core</artifactId>
  		<version>${struts2.version}</version>
  	</dependency>
  	<dependency>
  		<groupId>org.apache.struts</groupId>
  		<artifactId>struts2-json-plugin</artifactId>
  		<version>${struts2.version}</version>
  	</dependency>
  	<dependency>
  		<groupId>org.apache.struts</groupId>
  		<artifactId>struts2-spring-plugin</artifactId>
  		<version>${struts2.version}</version>
  	</dependency>
  	<!-- 日志 -->
  	<dependency>
  		<groupId>org.slf4j</groupId>
  		<artifactId>slf4j-log4j12</artifactId>
  		<version>1.7.2</version>
  	</dependency>
  	<!-- c3p0 -->
  	<dependency>
  		<groupId>c3p0</groupId>
  		<artifactId>c3p0</artifactId>
  		<version>${c3p0.version}</version>
  	</dependency>
  	<!-- mysql驱动 -->
  	<dependency>
  		<groupId>mysql</groupId>
  		<artifactId>mysql-connector-java</artifactId>
  		<version>${mysql.version}</version>
  	</dependency>
  	<!-- 缓存 -->
  	<dependency>
  		<groupId>net.sf.ehcache</groupId>
  		<artifactId>ehcache-core</artifactId>
  		<version>${ehcache.version}</version>
  	</dependency>
  	<!-- servlet jsp -->
  	<dependency>
  		<groupId>javax.servlet</groupId>
  		<artifactId>servlet-api</artifactId>
  		<version>2.5</version>
  		<scope>provided</scope>
  	</dependency>
  	<dependency>
  		<groupId>javax.servlet</groupId>
  		<artifactId>jsp-api</artifactId>
  		<version>2.0</version>
  		<scope>provided</scope>
  	</dependency>
  	<!-- junit4 -->
  	<dependency>
  		<groupId>junit</groupId>
  		<artifactId>junit</artifactId>
  		<version>4.9</version>
  		<scope>test</scope>
  	</dependency>
  	<!-- jstl -->
  	<dependency>
  		<groupId>javax.servlet</groupId>
  		<artifactId>jstl</artifactId>
  		<version>1.2</version>
  	</dependency>
  </dependencies>
</project>
~~~~~~

# 多模块和继承 #

多模块: 定义一组构建模块的聚集,
构建父模块的时候, 会自动构建子模块, 父模块的packaging必须为POM

继承: 复用配置, 子模块可以任意重写父模块的配置

1. 建立父工厂, 打包方式为 POM
~~~~~~
<!-- parent.pom -->
<packaging>pom</packaging>
~~~~~~
2. 新建两个子模块
~~~~~~
<!-- child.pom -->
<parent>
    <artifactId></artifactId>
    <groupId></groupId>
    <version></version>
    <relativePath>..</relativePath>
</parent>

<!-- parent.pom  -->
<!-- 在父工程中指定子模块 -->
<!-- 父工程编译同时编译子工程 -->
<modules>
    <module>child</module>
</modules>
<!-- 父工程引用的包子工程也可以使用 -->
<dependencies>
    <dependency> </dependency>
</dependencies>
~~~~~~

# 生命周期和插件使用 #
Maven提供三套生命周期
* `mvn clean` 清理生命周期
* `mvn install` 项目构建生命周期
* `mvn site` 生成站点生命周期

## 插件 ##
maven本身是一个框架, 三个生命周期每个环节都是依赖插件.

~~~~~~
<plugins>
<!-- maven-compiler-plugin 编译插件 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.6</source>
        <target>1.6</target>
        <encoding>UTF-8</encoding>
    </configuration>
</plugin>

<!-- maven-surefire-plugin 测试插件 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <!-- 跳过测试 -->
        <skip>true</skip>
        <testFailureIgnore>true</testFailureIgnore>
        <!-- 解决中文输出乱码  -->
        <argline>-Dfile.encoding=UTF-8 </argline>
    </configuration>
</plugin>

<!-- 使用tomcat:run命令, tomcat插件  -->
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>tomcat-maven-plugin</artifactId>
    <configuration>
        <!-- 可选，指定端口 -->
        <port>8080</port>
    </configuration>
</plugin>
</plugins>
~~~~~~

## profile 使用 ##
自定义一组配置，在运行时指定，覆盖默认配置
1. 配置profile
~~~~~~
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <jdbc.url>jdbc:mysql:///dev</jdbc.url>
        </properties>
    </profile>
    <profile>
        <id>product</id>
        <properties>
            <jdbc.url>jdbc:mysql:///product</jdbc.url>
        </properties>
    </profile>
</profiles>
~~~~~~
2. db.properties
~~~~~~
jdbc.url=${jdbc.url}
~~~~~~
3. 在项目构建时, 是pom生效
~~~~~~
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
~~~~~~
4. 运行
~~~~~~
> mvn -Pdev  ;; 或
> mvn -Pproduct
~~~~~~

# maven的私服(Nexus)搭建 #

maven仓库分为三种: 本地仓库, 远程仓库(私服【第三方仓库】、 中央仓库)
* 如果需要项目依赖, 优先从本地仓库下载
* 如果本地仓库没有项目依赖, 自动去中央仓库下载, 在本地仓库缓存

安装nexus
* 下载 `nexus.war`, 解压到 `tomcat/webapp`目录
* 启动tomcat访问, 用`user=admin;password=admin123`登陆
* 配置 `repository`(仓库列表), 具有七个仓库配置, 四种类型
  * group(仓库组), 一组仓库， 当使用坐标，按照组内顺序查找 
  * hosted(宿主), 私服
    * Releases: 指公司内部已经发布正式版本的项目
    * Snapshorts: 指公司内部项目还处于测试版本的项目
    * 3rd party: 私服配置指向第三方仓库, 由其它公司私服
  * proxy(代理): 通过私服去连接 apache或者 中央仓库
  * virtual(虚拟): 兼容maven1格式

## 私服安装后配置 ##

在运行私服后, 自动在 `$HOME/sonatype-work` 目录, 创建私服目录结构
* 设置 `downloadRemoteIndex=true`, 下载中央库索引
* `sonatype-work/storage/central`下放置中央仓库下载的包

## 将本地项目发布到私服 ##

1. 在本地maven的 `settings.xml` 配置连接私服用户名和密码,
`sonatype-work\nexus\conf\security.xml` 提供私服内部用户名和密码
~~~~~~
<!-- 正式发行仓库账号 -->
<server> 
    <id> releases </id>
    <username> admin </username>
    <password> admin123 </password>
</server>
<!-- 快照发行仓库账号 -->
<server> 
    <id> Snapshots </id>
    <username> admin </username>
    <password> admin123 </password>
</server>
~~~~~~
2. 修改本地pom.xml
~~~~~~
<!-- 发布到私服 -->
<distributionManagement>
    <repository>
        <id>releases</id><!--此处ID以上页server中的一致-->
        <name>Internal Releases</name>
        <url>releases仓库地址</url>
    </repository>
    <snapshotRepository>
        <id>Snapshots</id><!--此处ID以上页server中的一致-->
        <name>Internal Snapshots</name>
        <url>snapshots仓库地址</url>
    </snapshotRepository>
</distributionManagement>
~~~~~~
3. 发布, 根据版本自动发布到 snapshot 或 release
~~~~~~
> mvn deploy
~~~~~~

## 从私服下载文件 ##
默认情况下, 如果本地仓库没有,
去中央仓库找, 不会去私服找, 需要配置去私服下载项目

1. 配置镜像, setting.xml
~~~~~~
<mirrors>
    <mirror>
      	<id>nexus</id>
     	<mirrorOf>*</mirrorOf>
        <url>内部公共仓库地址</url>
    </mirror>
</mirrors>
~~~~~~
2. 激活
~~~~~~
<!-- setting.xml -->
<profiles>
    <!-- id: nexus 和 mirror的id一致 -->
    <profile>
        <id>nexus</id>
        <repositories>
            <repository>
                <id>central</id>
                <url>http://repo1.maven.org/maven2/</url>
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>true</enabled></snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>central</id>
                <url>http://repo1.maven.org/maven2/</url>
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>true</enabled></snapshots>
            </pluginRepository>
        </pluginRepositories>
    </profile>
</profiles>
<!-- profile要使用, 必须要激活 -->
<activeProfiles>
    <activeProfile>nexus</activeProfile>
</activeProfiles>
~~~~~~
3. 使程序更新 settings.xml
