title: java对象工具API
date: 2014-04-12 19:38:05
tags:
- java
---

# System #
~~~~~~
System.long currentTimeMillis();  // 当前毫秒值
System.exit();                // 退出
System.getProperties(); 
System.getProperty("line.separator");
System.setProperty(key, value ); //给系统设置一些属性信息, 其他程序可以使用
~~~~~~

# java.lang.Runtime #
每个Java应用都有一个Runtime类实例, 与其运行的环境相连接

> 查API文档, 没有构造方法提要, 就是私有构造方法


~~~~~~
Runtime r = Runtime.getRunTime();
try {
    Process p = r.exec("*.exe");
} catch( IOException e ){} // 调用本地程序
p.destroy();

~~~~~~

# java.lang.Math #
提供了数学方法, 是静态的
~~~~~~
ceil();    // 取上值
floor();   // 取下值
round();   // 四舍五入
max();
pow(3, 2);     // 三的二次方
Math.random() // 0 到 1 的伪随机值
Random r = new Random();
r.nextInt();
~~~~~~

# java.util.Date #
> Date月份是由 0 到 11 表示


1. 毫秒值 -> 日期, 通过 `new Date(long time)` 完成  

    还可以通过 `setTime()` 设置.
2. 日期对象 -> 毫秒值 `getTime()`
~~~~~~
Date d = new Date(); //当前日期和时间

boolean after(Date when);
boolean before(Date when);
~~~~~~

从JDK1.1开始使用Calendar类实现日期和时间字段之间的转换,
使用DateFormat类来格式化和解析日期字符串.
## java.text.DateFormat ##
~~~~~~
// 日期 默认风格的日期
String myString = DateFormat.getDateInstance().format(myDate)

// 日期 + 时间 默认风格的日期和时间
String myString = DateFormat.getDateTimeInstance().format(myDate)

DateFormat.getDateInstance(DateFormate.LONG);
DateFormat.getDateInstance(DateFormate.MEDIUM);
DateFormat.getDateTimeInstance(DateFormate.LONG, DateFormate.LONG)

// 自定义风格
new SimpleDateFormat("yyyyMMdd")
~~~~~~

## 将日期格式的字符串 转成 日期对象 ##

~~~~~~
String str_date = "2012-4-19";
DateFormat format = DateFormat.getDateInstance(DateFormat.LONG);
Date date = format.parse(str_date);
~~~~~~

## 两个日期之间相隔多少天 ##
~~~~~~
long separatTime = **
Int day = separatTime/1000/60/60/24;
~~~~~~

# java.util.Calendar #
~~~~~~
Calendar c = Calendar.getInstance();

c.get(Calendar.YEAR);
c.get(Calendar.MONTH);
c.get(Calendar.DAY_OF_MONTH);

c.set(2001, 3, 19);
c.add(Calendar.YEAR, 2);
c.add(Calendar.MONTH, 2);
c.add(Calendar.DAY_OF_MONTH, 2);
~~~~~~
