title: java线程学习
date: 2014-04-10 20:14:27
tags:
- java
- 线程
---
# 运行java程序两个线程 #
1. 执行main函数的线程
    该线程的任务代码都定义在main函数中

2. 负责垃圾回收的线程, 调用 `System.gc()`

# 多线程的创建方法 #
1. 继承Thread
~~~~~~
class MyThread extends Thread {
    public void run(){
       // Do some thing
       /* 获取线程的名称 Thread-编号
       * 在创建的同时已经给予编号
       */
       getName();
       Thread.currentThread().getName(); //当前运行线程的名称
    }
}
Thread t = new MyThread();
t.start(); //开启线程
~~~~~~
2. 实现runable接口

    好处: 1. 将线程任务从线程的子类中分离出来, 进行单独封装  
    2. 避免单继承的局限性
~~~~~~
class Demo implements Runnable{
   public void run(){}
}
Thread t = new Thread(new Demo());
t.start();
t.start(); // 多次启动会报异常
~~~~~~

# 线程的生命周期 #
1. 被创建 `new()`
2. 运行 `start()`
3. 消亡 `stop()` 或 等任务结束
4. 冻结 `sleep()` 或 `wait()`

    `wait()`不占有锁, 使用`notify()`唤醒;
    `sleep()`占有锁, 自己唤醒.

# 线程安全 #
产生的原因:
1. 多个线程在操作共享的数据
2. 操作共享数据的线程代码有多条

当一个线程在执行操作共享数据的多条代码的过程中,
其他线程参与了运算, 就会产生线程安全问题.
~~~~~~
public void run(){if(num>0) num++;}
~~~~~~
## 解决思路 ##
必须在当前代码执行完以后, 其他线程才能参与运算
~~~~~~
synchronized (this) {
    if(num>0) num++;
}
~~~~~~
弊端: 降低了效率

前提: 多个线程在同步中必须使用同一个锁
~~~~~~
private obj = new Object();
synchronized(obj){
    if(num>0) num++;
}
~~~~~~

### 同步函数 ###
~~~~~~
public synchronized void run(){
    if(num>0) num++;
}
~~~~~~
同步函数和和没有指定this对象同步锁, 锁定的是同一个对象this

~~~~~~
public static synchronized void run(){} // 锁定的对象是this.getClass()
~~~~~~

## 单例下的多线程安全 ##
~~~~~~
class Single{
    private static Single s = null;
    private Single(){}
    public static Single getInstance(){
        if(s==null) s = new Single();    // 线程安全
        return s;
    }
    // 解决方方法一: 写成同步函数
    // 解决方法二:
    public static Single getInstance(){
        if(s==null){  // 解决效率问题
            synchronized(Single.class){  // 解决线程安全问题
                if(s==null) s = new Single()
            }
        }
        return s;
    }
}

~~~~~~

## 死锁 ##
~~~~~~
public synchronized void run(){
    synchronized(obj){
    }
}
public void run(){
    synchronized(obj){
       synchronized(this){}
    }
}
~~~~~~


# 多线程通信 #
多个线程在处理同一资源, 但是任务却不同.

## 等待唤醒机制 ##
1. `wait()`: 让线程处于冻结状态, 被wait的线程会存储在线程池中.
2. `notify()`: *唤醒线程池中的一个线程.*
3. `notifyAll()`: 唤醒线程池中的所有线程.

这些方法必须定义在同步中,  
因为这些方法是用于操作线程状态的方法.  
必须要明确到底操作的是哪个锁上的线程. 
~~~~~~
//Thread1:
synchronized(r){
    while(flag) r.wait(); //不能是if, 会出现数据错误
    doSome();
    flag = true;
    r.notifyAll();  // 如果是用notify, 可能会阻塞
}

//Thread2:
synchronized(r){
    while(!flag) r.wait();
    doSome();
    flag = false;
    r.notifyAll(); 
}
~~~~~~


~~~~~~
//都必须捕捉
try{
    wait();
    sleep();
}catch(InterruptedException e) {}
~~~~~~

# java.util.concurrency.locks.*#
jdk1.5以后将同步和锁封装成了对象

Lock 替代了 synchronized 方法和语句, 可以加上多组监视器.

Condition 替代了 `notify()` 和 `wait()`
~~~~~~
// Lock是接口
Lock lock = new ReentrantLock(); // 互斥锁

lock.lock();
try{
  doSome();
} finally{
    lock.unlock();
}

Condition cond = lock.newCondition();
cond.await();
cond.singnal();
cond.singnalAll();
~~~~~~
From API Referrence: 
~~~~~~
class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length)
         notFull.await();
       items[putptr] = x;
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0)
         notEmpty.await();
       Object x = items[takeptr];
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   }
 }
~~~~~~

# wait 和 sleep 的区别 #
1. wait可以指定时间也可以不指定, sleep必须指定时间
    
2. 在同步中时, 对cpu的执行权和锁的处理不同.

    wait: 释放执行权,释放锁
    sleep: 释放执行权, 不释放锁

# 停止线程的方法 #
1. 调用 `stop()` `susppend()`方法, 已经过时, 由安全问题
2. 等 `run()` 方法结束
~~~~~~
// 控制, 但是线程处于冻结状态, 无法读取标志
public void run(){
    while(flag){ doSome(); }
}
~~~~~~
3. 调用 `interrupt()`,让线程从冻结状态中强制恢复过来,`sleep()` 和 `wait()` 会抛出异常

# 守护线程 #
`thread.setDeamon(true)` 必须在启动线程钱调用, 当正在运行的
的线程都是守护线程时, java虚拟机退出.

# join #
`thread.join()` 主线程等待`thread`线程终止, 再执行.

# 线程其他设置 #
~~~~~~
/**设置线程优先级**/
thread.setPriority(THREAD.MAX_PRIORITY); // 最大为10

/**设置线程组**/
new Thread(TreadGroup tg)
tg.interrupt();

/* yield */
thread.yield(); //暂时释放执行权
~~~~~~

