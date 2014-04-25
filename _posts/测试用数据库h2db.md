title: 测试用数据库H2db
date: 2014-04-24 20:59:45
tags:
- java
---

在写上一个项目时, 在运行测试和真实运行项目时, 连接的是同一个Mysql数据库.
但是运行单元测试必须要有可重复性和隔离性, 测试时连接真实数据库,
造成了单元测试不能重复运行, 所以需要一个机制, 在运行测试时,
新建一个隔离的数据库, 等测试完, 把这个数据库清空或者废除. 详细需求如下:
1. 根据配置文件, 让系统选择加载数据库, 如在测试时连接一个数据库, 运行时连接另一个数据
2. 如果是测试, 那么系统在加载数据库之后需要运行一些SQL脚本, 初始化数据库
3. 运行完测试, 这个数据库要重置, 或者废除
4. 数据库不能是Mysql的外部安装的数据库, 这样可以在没有Mysql的机子上重复测试

so, 根据谷歌搜索, 找到了java内嵌数据库`Derby`, 它是用纯Java写的数据库,
只用加载一个Jar包, 就可以在Java程序中内嵌运行一个数据库. 运行代码如下
~~~~~~
Class.forName("org.apache.derby.jdbc.EmbeddedDriver");
DriverManager.getConnection("jdbc:derby:sample;create=true");
~~~~~~

这样就连接了一个Sample数据库, 且Derby会在工作目录下新建一个文件夹,
存储Sample数据库的数据. 所以可以通过指定新建数据库文件夹在一个临时文件夹下, 来满足条件三.
如Linux可以把新建文件建在 `/tmp` 目录下, 代码如下
~~~~~~
DriverManager.getConnection("jdbc:derby:/tmp/sample;create=true");
~~~~~~
因为每次运行测试不能连同一个数据库文件, 所以要用Java生成一个临时文件夹
~~~~~~
DriverManager.getConnection(makeDerbyTempURL("sample"));

public static String makeDerbyTempURL(String db) {
    Path p = makeTmpDir();
	return "jdbc:derby:" + p.toUri().getPath() + "/" + db + ";create=true";
}
private static Path makeTmpDir() {
    Path path = new File("/tmp").toPath();
	Path p = null;
	try {
	    p = Files.createTempDirectory(path, "derby");
	} catch (IOException e) {
	    throw new RuntimeException(e);
	}
	return p;
}
~~~~~~

但是如何满足条件二呢? 我猜想Derby提供了在内嵌数据库模式下直接导入SQL脚本的API,
就一直通过谷歌搜索, 找这个解决方案, 可惜做了很多无用功.
后来发现, 其实可以换个思路, 通过JDBC, 来直接执行SQL语句, 但是如何执行脚本呢?
要我重新写一个程序, 加载脚本, 分析出SQL语句, 然后通过JDBC一条一条执行? no!
有其他人已经解决过这个问题, 并且有Jar包可以直接使用, 通过这个思路,
两三下就直接找到了[解决方案](http://www.cnblogs.com/zencorn/archive/2011/01/27/1946348.html).

~~~~~~
public static void execSQL(String scriptPath, String driveClass,
        String url, String user, String passwd) {
	SQLExec sqlExec = new SQLExec();
	sqlExec.setDriver(driveClass);
	sqlExec.setUrl(url);
	sqlExec.setUserid(user);
	sqlExec.setPassword(passwd);
	sqlExec.setSrc(new File(scriptPath));
	sqlExec.setOnerror((SQLExec.OnError) (EnumeratedAttribute.getInstance(
	        SQLExec.OnError.class, "abort")));
	sqlExec.setPrint(false);
	sqlExec.setProject(new Project());
	sqlExec.execute();
}
~~~~~~

所以说如何看待一个问题, 是解决一个问题的关键点.

至于需求一, 可以通过配置`properties`来解决, 感兴趣的童鞋可以
[猛击这里](https://github.com/zhpooer/itcast-customer-demo/blob/master/src/java/io/zhpooer/util/ConnManager.java)

到这里, 已经基本解决了问题, 不过又出现了一个问题, Derby 不支持语句 `create table if exists tablename()...`, 
而且每次运行测试时, 发现Derby创建数据库时, 总会造成硬盘忙的状态, 卡一小段时间.
那么是不是可以有一个数据库直接在内存里运行, 测试运行完之后, 数据库直接删除.

这个可以有, 那就是`H2db`, 它既支持内存数据库, 又支持内嵌数据.
不过经我测试, 在H2内存数据库模式下, 只要运行完 `connection.close()`, 数据库就会被重置(有谁指正一下如何可以避免这个问题).
这和我的项目需求不一致, 但是我发现他在内嵌模式下启动速度比Derby快. 所以直接换掉Derby.

[具体代码猛击这里](https://github.com/zhpooer/itcast-customer-demo/blob/master/src/java/io/zhpooer/util/ConnManager.java)
