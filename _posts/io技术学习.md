title: IO技术学习
date: 2014-04-13 18:47:16
tags:
- java
- IO
---

# IO初识 #

流按数据类型分类:
1. 字符流, 字节流读取文字自己数据后,
不直接操作而是先查指定的编码表, 获取指定的文字.
就是 字节流+编码表
2. 字节流

流按操作方式分类:
1. 输入流, 将外设中的数据读取到内存中
2. 输出流, 将内存中的数据写到外设中


# 字符流 #
字符流的两个顶层父类
1. Reader
2. Writer

如果操作文集数据, 优先考虑字符流
要将数据从内存写到硬盘上, 要使用字符流中的输出流.`Writer`



## FileWriter ##
~~~~~~
// 文件会自动创建或覆盖
FileWriter fw = new FileWriter("demo.txt");
// 数据写到内存临时缓冲区
fw.write("abce");
fw.flush(); //进行刷新, 将数据直接写到硬盘上
fw.close();  // 关闭会自动 flush

// 如果构造函数加入true, 可以实现文件的续写
FileWriter fw = new FileWriter("demo.txt", true);
~~~~~~


## IO 异常处理 ##

~~~~~~
FileWriter fw = null;
try{
}catch(IOExcetpion e){
} finally {
   if(fw!=null){ /*  重要   */
       try{
           fw.close()
       }catch(IOExcetpion){}
   }
}
~~~~~~


## 读取一个文件把字符串输入到控制台 ##

~~~~~~
// 方式一
FileReader fr = new FileReader("demo.txt");
int ch = 0;
while((ch=fr.read())!=-1){
   print(ch);
}
System.out.prinln(ch1);
// 方式二
char[] buf = new char[3];
int num  = 0;
while((num = fr.read(buf))!=-1){
    println(new String(buf,0,len));
}
~~~~~~


## 文件复制 ##

~~~~~~
// 1. 对一个已有文本
FileReader src = new FileReader("file1");
// 2. 创建一个目的, 用于存储到数据
FileReader dist = new FileWriter("file2");
// 3. 读写操作
char[] buf = new char[1024*4];
int len = 0;
while((len=src.read(buf)) != -1) {
    dist.write(buf, 0, len);
}
// 4. 关闭流资源
src.close();
dist.close();

~~~~~~


## 字符流的缓冲区 ##

缓冲区提高读写效率

缓冲区要结合流来读取


### BufferedReader ###
~~~~~~
FileWriter fw = new FileWriter("buf.txt");

BufferedWriter bufw = new BufferedWriter(fw); // or new BufferedWriter(fw, 1024);

bufw.write("abcde");
bufw.newLine(); // 新行
bufw.close();
// fw.write("");// 流已经被关了会抛错
~~~~~~

### BufferedWriter ###
`read()`是从缓冲区取出字符数据. 做一覆盖了父类的read方法

`readLine()`使用了读取缓冲区的read方法,将读取到的字符进行缓冲并判断换行标记,
将标记前的缓冲数据进行读取


## 装饰设计模式 ##
对一组对象的功能进行增强时, 就可以使用该模式进行问题的解决.

装饰比继承灵活

特点: 装饰类和被装饰类都必须是所属同一接口或父类


## LineNumberReader ##

是 BufferedReader 的子类
~~~~~~
LineNumberReader lnr = new LineNumberReader(new FileReader());

lnr.setLineNumber(100);
while((line=lnr.readLine())!=null){
    lnr.getLineNumber() + ":"  + lnr.readLine();
}
lnr.close();
~~~~~~


# 字节流 #
字符流的顶层父类
1. InputStream
2. OutputStream

字节流处理是字节数组, 字符流处理的是字符流

~~~~~~
FileOutputStream fos = new FileOutputStream();

fos.write("".getBytes());

fos.close();
~~~~~~

~~~~~~
fis.available(); // 返回文件大小
FileInputStream fis = new FileInputStream("");
byte[] buf = new byte[fis.available()]; // 小文件可以, 大文件慎用
fis.read(buf);
new String(buf);
~~~~~~


## 读取一个键盘录入的数据, 并打印在控制台上 ###
`System.in` 关了之后, 就不能再用了, *千万别* `close()`
~~~~~~
// 简单方法
InputStream in = System.in;
int ch = in.read(); // 会阻塞
println(ch);

// 方法一
StringBuilder sb = new StringBuilder();
while((ch=in.read())!=-1){
    if(ch=='\r'){ continue;}
    if(ch=='\n'){
        String temp = sb.toString();
        if("over".equals(temp)) break;
        sb.delete(0, sb.length());
    } else
        sb.append(ch);
}

// 方法二
InputStreamReader isr = new InputStreamReader(System.in);
BufferedReader bufr = new BufferedReader(isr);
bufr.readLine();
~~~~~~


## 转换流 ##

`InputStreamReader`, 将字节转换成字符的桥梁, 可以设置字符集
> `new InputStreamReader(in, charset)`


`OutputStreamWriter`, 是字符流通向字节流的桥梁, 可指示编码
~~~~~~
Writer osw = new BufferedWriter(new OutputStreamWriter(System.out));
osw.write();
osw.newLine();
osw.flush(); // 如果要输出, 就要刷新缓冲
~~~~~~
`FileWriter` 继承于 `OutputStreamWriter`


# 流的操作规律 #
1. 明确源和目的

    源: InputStream Reader  
    目的: OutputStream Writer
2. 明确数据是否纯文本

    1. 源是纯文本: reader
    2. 源不是纯文本: InputStream
    3. 目的是纯文本: Writer
    4. 目的不是纯文本: OutputStream

3. 明确具体的设备

    源设备, 目的设备
    1. 硬盘: File
    2. 键盘: System.in, System.out
    3. 内存: 数组
    4. 网络: Socket流

4. 是否需要其他额外功能

    如果需要高效缓冲区, 就加上Buffer


~~~~~~
new FileWriter("file"); // 以系统默认的编码写入文件
// 如果指定码表, 那么就不可以使用FileWriter,
// 他内部使用默认本地码表, 使用如下
new OutputStreamWriter(new FileOutputStream(), "UTF-8");
// 同理, FileReader 亦然
new InputStreamReader("file", "utf-8");
~~~~~~


# java.io.File类 #
对文件或*文件夹*的属性进行操作
~~~~~~
File.separator; 文件分隔符
File.pathSeparator; 路径夹分隔符

//获取属性
File f = new File("a.txt");
File f = new File(dir, filename);
f.getName();
f.getAbsolutePath();
File pathFile = f.getAbsoluteFile();  
f.getPath();
long len = f.length();
long time = file.lastModified();

// 文件的创建和删除
f.canWrite();
f.createNewFile();  // 如果文件存在就不创建
f.createTempFile();

f.delete(); 
file.deleteOnExist();

// 文件夹
f.mkdir(); // 创建单级目录
f.mkdirs(); // 创建多级目录

// 判断
f.exists();
f.isFile();  // 要先判断文件是否存在
f.isDirectory(); // 要先判断文件是否存在

// 重命名
File f2 = new File("2");
f.renameTo(f2);

// 列出可用的系统根
File.listRoots(); //如 c盘 d盘 e盘
File dRoot = new File("d:\\");
dRoot.getFreeeSpace();
dRoot.getTotalSpace();
dRoot.getUsableSpace();

// 获取当前目录下文件或文件夹的名称
String[] names = f.list(); // f必须是目录, 需要判断是否为空, 访问系统级目录也会空指针
File[] files = f.listFiles();

f.list(FilenameFilter); // 过滤器
~~~~~~



# java.util.Properties 集合 #

Properties 是一个 HashTable, 特点
1. 键值都是字符串
2. 集合中的数据可以保存在流中, 从流中获取

~~~~~~
Properties p = new Properties();  // 注意: 只采用88591-字符编码
p.load(new FileInputStream());
p.setProperty("1", "2");
p.stringPropertyNames();

// 输出到流
p.list(PrintStream out);
// 存储
p.store(OutputStream s, String comments)
p.store(Writer s, String comments)
~~~~~~


# 其他IO流 #

## PrintStream 和 PrintWriter ##
PrintStream 从来不抛出异常, 可以知己恩操纵文件

可以打印多种数据类型, 并保持数据的表示形式
~~~~~~
PrintStream ps = new PrintStream("filename");
ps.print(97);  // 先将97变成字符串

new PrintWriter(Writer w);
new PrintWriter(OuputStream w, true); // 默认是带缓冲的, 可以设置自动刷新
new PrintWriter(File w);
new PrintWriter(String path);
~~~~~~


## 序列流 SequenceInputStream ##
对多个流进行合并, 串联读取
~~~~~~
Vector<FileInputStream> v = new Vector<FileInputStream>();
Enumeration<FileInputStream> en = e.elements();
// 或
Enumeration<FileInputStream> en = Collections.enumerations(list)
new SequenceInputStream(en);
~~~~~~


## ObjectInputStream 与 ObjectOutputStream ##
~~~~~~
ObjectOutputStream oos = new ObjectOutputStream(FileOutputStream);
oos.writeObject(Serializable obj);  // obj 必须实现接口 java.io.Serializable

ObjectInputStream ois = new ObjectInputStream();
(Object) ois.readObject();

~~~~~~


### Serializable ###
使用 `serialVersionUID` 验证序列化对象的类版本信息


### transient ###

被标记了 transient 的字段, 不会被序列化

## RandomAccessFile ##
支持随机访问文件的读取和写入
1. 既能读,又能写
2. 内部维护了以 byte 数组, 通过指针操作数组中的元素
3. 可以通过 `getFilePoint()` 和 `seek()` 操作指针
4. 它封装了 输入流 和 输出流
5. 源和目的只能是 文件

~~~~~~
RandomAccessFile f = new RandomAccessFile("", "rw");
f.write("".getBytes());
f.seek(1*8);
f.readInt("");

f.close();
~~~~~~

## PipedStream ##

管道流, 输入和输出可以直接进行连接, *要结合线程使用*

~~~~~~
PipedInputStream pis = new PipedInputStream();
PipedOutputStream pos = new PipedOutputStream();
pis.connect(pos);
~~~~~~

## 其他输入输出流 ##
1. DataOutputStream 和 DataInputStream

    
2. ByteArrayInputStream 和 ByteArrayOutputStream

    源和目的都是内存, 关闭他没有效果
    ~~~~~~
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    ByteArrayInputStream bis = new ByteArrayInputStream("".getBytes());
    bis.read();
    bos.write();
    bos.toByteArray();
    ~~~~~~
3. CharArrayReader 和 CharArrayWriter
4. StringReader 与 StringWriter

# 编码解码 #
~~~~~~
String str = "你好";
str.getBytes();
byte[] buf = str.getBytes("GBK");

new String(buf, "GBK")
~~~~~~
