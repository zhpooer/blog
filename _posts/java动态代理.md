title: java动态代理
date: 2014-04-18 20:14:52
tags:
- java
- 动态代理
---
# java.lang.reflect.Proxy #
它提供了一组静态方法来为一组接口动态地生成代理类及其对象
~~~~~~
// 方法 1: 该方法用于获取指定代理对象所关联的调用处理器
static InvocationHandler getInvocationHandler(Object proxy) 

// 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象
static Class getProxyClass(ClassLoader loader, Class[] interfaces) 

// 方法 3：该方法用于判断指定类对象是否是一个动态代理类
static boolean isProxyClass(Class cl) 

// 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例
static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)
~~~~~~

# java.lang.reflect.InvocationHandler #
每次生成动态代理类对象时都需要指定一个实现了该接口的调用处理器对象（参见 Proxy 静态方法 4 的第三个参数）
~~~~~~
// 该方法负责集中处理动态代理类上的所有方法调用。第一个参数既是代理类实例，第二个参数是被调用的方法对象
// 第三个方法是调用参数。调用处理器根据这三个参数进行预处理或分派到委托类实例上发射执行
Object invoke(Object proxy, Method method, Object[] args)
~~~~~~

# 实例 #
注意: 这是Scala实现
~~~~~~
trait Stub {
  def doSome(): Unit;
}

class StubImpl extends Stub {
  override def doSome() {
    println("hello world");
  }
}
// 定义处理对象的代理方法
class ProxyHandler(proxied: Object) extends InvocationHandler {
  def this() = this(null)
  override def invoke(proxy: Object, method: Method, args: Array[Object]): AnyRef = {
    println("before invoke")
    val result = method.invoke(proxied)  // 调用主体方法, 可以在调用方法上下插入处理
    println("after invoke")
    result
  }
}
//主方法
it should "通过反射实现实现动态代理" in {
  val proxy = Proxy.newProxyInstance(this.getClass().getClassLoader(), Array(classOf[Stub]), new ProxyHandler(new StubImpl))
  proxy match {
    case p: Stub => p.doSome();
  }
}
~~~~~~
