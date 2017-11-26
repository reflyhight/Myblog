title: Java多线程系列（一）开启新线程
tags: 
	- Java
	- 多线程
	- Random

categories:
	- Java
-------------------
###    线程简介
**概念**：线程是程序执行路径，Java多线程是建立在Java虚拟机的线程调度的基础上的，线程是程序执行的最小单位，一个Java进程可以包含多个线程，从宏观上说多个线程之间是切换执行，完全并行，但具体到某一个时刻的并行度会受限于CPU能支持的并行度。
**优先级**：创建线程的时候，可以设置线程的优先级，通过设置高优先级，可以使得该线程获CPU的机会增加（但不代表绝对优先执行）。
**守护线程**：创建线程的时候，线程可以分为守护线程和非守护线程，当设置为守护线程，线程则为守护线程，默认情况即为非守护线程。Java程序启动时，Jvm首先会开启一个非守护线程，也就是main方法开启的线程
**进程结束**：Java进程结束的条件：
- 调用Runtime 的exit方法，并且安全管理器允许退出操作发生
- 非守护线程的所有线程都已经停止
 
**创建方式**：创建线程的两种方式：继承Thread类，实现Runnable接口
 
###    开启新线程
####    Thread方式
```java
//线程类
/**
* @author haima
* @date 2017年11月4日
* @com blog.mxjhaima.com
* @email jh624haima@126.com
*/
public class PrimeThread extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        super.run();
        System.out.println("this is a new thread");
    }
}
 
//测试类
public class MainTest {
    public static void main(String[] args) {
        PrimeThread t1= new PrimeThread();
        t1.start();
    }
}
```
主线程t1.start()；后，就开启了一个全新的线程了
 
####    Runnable方式
```java
/**
*
* @author haima
* @date 2017年11月4日
* @com blog.mxjhaima.com
* @email jh624haima@126.com
*/
public class RunableThread implements Runnable{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        System.out.println("this is a new thread");
    }
}
 
public class MainTest {
    public static void main(String[] args) {
        //        PrimeThread t1= new PrimeThread();
        //        t1.start();
        RunableThread t2= new RunableThread();
        new Thread(t2).start();
    }
}
```
主线程new Thread(t2).start();后，就开启了一个全新的线程了
 
####    关于start方法
两种开启线程的方式方式，但实际上都是通过Thread.start方法来开启的，start()方法的内部并没有直接调用run()，如下：
```java
public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
 
        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);
 
        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
 
    private native void start0();
```
调用start0()方法的底层我们就不去深究了，反正run()方法最后会被调用，完成我们写的业务。记得开启线程的方式是start()方法就好了。
**一个线程只能被开启一次**：start()方法如果再次被调用，将会报错，开启新线程需要一个新的Thread对象
```java
/**
*
* @author haima
* @date 2017年11月4日
* @com blog.mxjhaima.com
* @email jh624haima@126.com
*/
public class MainTest {
    public static void main(String[] args) {
                PrimeThread t1= new PrimeThread();
                t1.start();
                t1.start();
    }
}
```
执行结果：
```java
Exception in thread "main" this is a new thread
java.lang.IllegalThreadStateException
    at java.lang.Thread.start(Unknown Source)
    at com.mxj.thread.test.MainTest.main(MainTest.java:14)
```
####   Thread和Runnable的联系
查看Thread类的源码，我们可以看到Thread是实现了Runnable接口，而Runnable接口中只声明了一个方法就是run()方法
```java
//java.lang.Thread.class，截取部分
 
 * @since   JDK1.0
*/
public
class Thread implements Runnable {
```
 
```java
//java.lang.Runnable.class，截取部分
 
* @since   JDK1.0
*/
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```
再看看第二种方式创建线程的过程，构造方法Thread(Runnable target)，将target传递给Thread对象作为一个成员变量
```java
//java.lang.Thread.class，截取部分
public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
}
```
再看看Thread类的run()方法：
```java
//java.lang.Thread.class，截取部分
@Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```
明白第二种创建线程的方式了吧。通过start()开启后执行Thread()中的run()方法，而run()中直接调用传入的Runnable实例中的run()方法
**这两种方式的比较**：通常我们都使用第二种方式创建新线程，因为Java不支持多继承，直接继承Thread类，就无法继承其他的类了。而两种创建方式的本质其实都一样。
 
 
####    run方法
让我们来证明确实是开启了一个新线程，而不是平常普通方法的调用
修改PrimeThread类结构如下：
```java
public class PrimeThread extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        super.run();
        while(true){
 
            System.out.println("this is child thread");
            try {
                Thread.sleep(ThreadLocalRandom.current().nextLong(2000));
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}
```
修改MainTest为如下:
```java
public class MainTest {
    public static void main(String[] args) {
                PrimeThread t1= new PrimeThread();
                t1.start();
                while(true){
                    System.out.println("this is Main thread");
                    try {
                        Thread.sleep(ThreadLocalRandom.current().nextLong(2000));
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
    }
}
```
执行结果：
```
this is Main thread
this is child thread
this is child thread
this is Main thread
this is Main thread
this is child thread
this is child thread
this is Main thread
this is Main thread
this is child thread
```
从上面的执行结果我们可以看出，主线程和子线程是交替打印，说明这两者是并行的，而不是顺序执行
再将MainTest 类做一个小改动，直接用t1调用run()方法
```java
public class MainTest {
    public static void main(String[] args) {
                PrimeThread t1= new PrimeThread();
                t1.run();
 
                while(true){
                    System.out.println("this is Main thread");
                    try {
                        Thread.sleep(ThreadLocalRandom.current().nextLong(2000));
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
        }
}
```
执行结果如下：
```
this is child thread
this is child thread
this is child thread
this is child thread
this is child thread
this is child thread
this is child thread
```
**结果显示**：run()方法不能开启新的一个线程，而是单一的主线程阻塞在run()方法中，无法继续执行后面的内容了

