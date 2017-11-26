title: Java多线程系列（二）Thread类常见方法和属性
tags: 
	- Java
	- 多线程
	- static

categories:
	- Java
	
	
-------------------

###    interrupt()
**public void interrupt()**  
**梦想只是一阵风，中断只是一个信号**
下面我们来学习一下Thread中的interrupt()方法，该方法不会影响到该线程的生命周期，也就是说线程并不会直接停止，如下：



<!-- more -->



```java
//线程类
public class InterruptThread extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        super.run();
        while(true){
            if(2<1)break;//避免死循环造成编译错误
            System.out.println("I am aliving");
 
        }
        System.out.println("game over");
    }
}
 
 
//测试类
public class MainTest {
    public static void main(String[] args) {
 
        InterruptThread interruptThread = new InterruptThread();
        interruptThread.start();
        try {
            Thread.sleep(5000);//保证线程启动，五秒后，主线程发起中断信号
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("interrupt");
        interruptThread.interrupt();
 
 
   }
}
```
执行结果：
```java
...省略部分
I am aliving
I am aliving
interrupt
I am aliving
I am aliving
I am aliving
I am aliving
I am aliving
I am aliving
I am aliving
I am aliving
I am aliving
...进程一直未结束
```
那怎么使用interrupt()方法达到停止线程的目的呢？interrupt()其实给线程传递了一个中断信号，可以通过isInterrupted()获得，修改InterruptThread 代码如下：
```java
public class InterruptThread extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        super.run();
        while(true){
            if(this.isInterrupted()){
                System.out.println("catched interrupt signal");
                break;//检测中断信号，跳出循环，结束线程
            }
            System.out.println("I am aliving");
 
        }
 
        System.out.println("game over");
 
   }
}
```
执行结果为：
```java
...省略部分
I am aliving
I am aliving
I am aliving
I am aliving
I am aliving
I am aliving
interrupt
I am aliving
catched interrupt signal
game over    //跳出循环run()方法执行完，线程结束，进程也就执行结束
```
###   interrupted() vs isInterrupted()
**public static boolean interrupted() ** 静态方法（不属于对象，属于类），表示当前线程的中断信号
**public boolean isInterrupted()  ** 是成员方法，表示该线程对象的中断信号
 
上面的阐述可能让人有点理解，关键在于“当前线程”和“该线程对象”的比较。
```java
//线程类
public class InterruptThread extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        super.run();
        while(true){
            //啥也不做，让线程模拟挂起状态
        }
    }
}
 
//测试类
public class MainTest {
    public static void main(String[] args) {
        //当前线程名
        String currentThreadName = Thread.currentThread().getName();
        System.out.println("currentThread:"+currentThreadName);
        InterruptThread t1= new InterruptThread();
        t1.start();
        //t1线程对象线程名
        String t1Name = t1.getName();
        System.out.println("t1ThreadName:"+t1Name);
 
        //中断请求信号
        t1.interrupt();
        System.out.println("t1 is interrupted:"+t1.isInterrupted());
        System.out.println("currentThread is interrupted:"+Thread.interrupted());
 
   }
}
```
执行结果：
```java
currentThread:main    //当前线程名为:main
t1ThreadName:Thread-0   
t1 is interrupted:true
t1 is interrupted:true
currentThread is interrupted:false    //当前线程并没有被中断信号
```
这里我们还要注意，因为方法*interrupted()*和方法*currentThread()*都是静态方法，所以无论你用线程类实例t1.interrupted()还是用线程类Thread.interrupted()调用，结果都不会改变，调用结果也和t1没有关系
现在我们再来测试一下静态方法interrupted()，加深一下我们对这个方法的理解，我们将测试类做如下修改：
```java
public class MainTest {
    public static void main(String[] args) {
        Thread.currentThread().interrupt();//当前线程中断信号
        System.out.println("currentThread is interrupted:"+Thread.interrupted());//当前线程是否中断
        System.out.println("currentThread is interrupted:"+Thread.interrupted());
 
   }
}
```
执行结果：
```java
currentThread is interrupted:true
currentThread is interrupted:false
```
是不是有点让人意外，为什么第二次再调用interrupted()时，结果发生了变化。让我们看看interrupted()和isInterrupted()的源码：
```java
//interrupted()
public static boolean interrupted() {
        return currentThread().isInterrupted(true);
}
 
//isInterrupted()
public boolean isInterrupted() {
        return isInterrupted(false);
    }
 
/**
     * Tests if some Thread has been interrupted.  The interrupted state
     * is reset or not based on the value of ClearInterrupted that is
     * passed.
     */
private native boolean isInterrupted(boolean ClearInterrupted);
```
从重载方法isInterrupte(boolean ClearInterrupted)可以看出，interrupted()会清除中断信号，所以调用完interrupt()，当前线程不再是中断状态了
 
###   sleep( long millis) && InterruptedException
**public static native void sleep(long millis) throws InterruptedException**
可能你已经注意到Thread.sleep 可能会抛出InterruptedException 异常，所以需要try-catch一下，Thread.sleep(long millis)为静态方法，同样也是代表当前线程的操作，表示让当前线程暂停一会儿,来看看源码
```java
//sleep方法
  * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public static native void sleep(long millis) throws InterruptedException;
```
sleep使用native修饰，是一个调用c,c++的方法，不深究底层实现。我们来看一下这个方法的注释**@throws  InterruptedException  if any thread has interrupted the current thread. The   <i>interrupted status</i> of the current thread is  cleared when this exception is thrown.**，阐述得很明了，抛出中断异常InterruptedException时，中断状态将被清除（前文说过的中断信号），来做个测试证明一下这个结论：
```java
//线程类
public class InterruptedExceptionTest extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        super.run();
        while(true){
            try {
                Thread.sleep(100000L);//让线程一直暂停休眠
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
                System.out.println("throw the InterruptedException");
            }
        }
    }
}
public class MainTest {
    public static void main(String[] args) {
 
        InterruptedExceptionTest t1= new InterruptedExceptionTest();
        t1.start();
        try {
            Thread.sleep(1000);//主线程暂停1秒，保证t1线程已经被启动了
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        t1.interrupt();
        System.out.println("I want to interrupt");
        System.out.println("t1's interrupt status is :"+t1.isInterrupted());
   }
}
```
执行结果：
```java
I want to interrupt
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at com.mxj.thread.test.InterruptedExceptionTest.run(InterruptedExceptionTest.java:12)
t1's interrupt status is :false    //抛出InterruptedException后，中断状态（信号）被清除了
throw the InterruptedException
```
如上，抛出InterruptedException后，中断状态（信号）被清除了，打印**false**，但进程依然没停止
让我们再看看InterruptedException的源码：
```java
* <pre>
 *  if (Thread.interrupted())  // Clears interrupted status!
 *      throw new InterruptedException();
* </pre>
*
 * @author  Frank Yellin
 * @see     java.lang.Object#wait()
 * @see     java.lang.Object#wait(long)
 * @see     java.lang.Object#wait(long, int)
 * @see     java.lang.Thread#sleep(long)
 * @see     java.lang.Thread#interrupt()
 * @see     java.lang.Thread#interrupted()
 * @since   JDK1.0
*/
public
class InterruptedException extends Exception {
    private static final long serialVersionUID = 6700697376100628473L;
```
看注释已经很清楚了吧，**如果当前线程中断状态为true，那么清除中断状态，后面列表是参考更多的相关方法说明的列表。通过eclipse中快捷ctrl+shift+g键搜索方法被引用，就能找到哪些方法抛出了InterruptedException异常，可能抛出该异常的方法挺多，列出java.lang包下几个常见的：
- java.lang.Object#wait
- java.lang.Thread#sleep
- java.lang.Thread#join
- java.lang.ProcessImpl#waitFor
 
###   isAlive()
**public final native boolean isAlive() **
这个方法没有什么需要特别说明的，非静态方法，代表这个方法的线程是否是存活状态。鉴于启动和死亡之间
```java
//线程类
public class AliveTest extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        super.run();
        System.out.println("t1's alive status is:"+this.isAlive());   
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
 
//测试类
public class MainTest {
    public static void main(String[] args) {
 
        AliveTest t1= new AliveTest();
        t1.start();
        try {
            Thread.sleep(200);//保证t1被启动了
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("t1's alive status is:"+t1.isAlive());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("t1's alive status is:"+t1.isAlive());
   }
}
```
执行结果：
```java
t1's alive status is:true
t1's alive status is:true
t1's alive status is:false
```
稍微改动一下测试类，将start()改为run()，执行结果：
```java
t1's alive status is:false
t1's alive status is:false
t1's alive status is:false
```
在次证明start才能开启一个线程，直接调用run()方法，和普通方法无异
 
###   getId()
**public long getId() **
获得线程的ID号
```java
/**
     * Returns the identifier of this Thread.  The thread ID is a positive
     * <tt>long</tt> number generated when this thread was created.
     * The thread ID is unique and remains unchanged during its lifetime.
     * When a thread is terminated, this thread ID may be reused.
     *
     * @return this thread's ID.
     * @since 1.5
     */
    public long getId() {
        return tid;
    }
```
###   stop()
**public final void stop()  **
该方法能强制停止线程，暴力停止，但目前已经过时，不建议使用。
stop方法造成数据不一致的情况发生。stop方法暴力停止，在方法执行过程中任意位置停止，不在开发者掌控之中，会造成方法执行不完全，造成数据不一致
```
//线程类
public class StopThread extends Thread {
 
    private String m1;
    private String m2;
 
    public String getM1() {
        return m1;
    }
 
    public void setM1(String m1) {
        this.m1 = m1;
    }
 
    public String getM2() {
        return m2;
    }
 
    public void setM2(String m2) {
        this.m2 = m2;
    }
 
    public void printM1M2() {
        System.out.println("m1:" + m1);
        System.out.println("m2:" + m2);
    }
 
    @Override
    public void run() {
        super.run();
        // TODO Auto-generated method stub
        setM1("rob");
        try {
            Thread.sleep(5000L);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        setM2("jiang");
    }
}
public class MainTest {
    public static void main(String[] args) {
 
        StopThread st= new StopThread();
        st.start();
        try {
            Thread.sleep(500L);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
 
        st.stop();
        st.printM1M2();
   }
}
```
执行结果
```
m1:rob
m2:null
```
 
###    currentThread()
**public static native Thread currentThread()**
该方法可以获得当前线程，但一定要明白什么是当前线程，因为该方法是静态方法，所以如果是对象调用，也会和这个对象无关，在此用代码说明一下：
```java
//线程类
public class CurrentThreadTest extends Thread{
        CurrentThreadTest(String threadName){
            this.setName(threadName);//设置线程名
        }
        @Override
        public void run() {
            // TODO Auto-generated method stub
            super.run();
 
        }
}

public class MainTest {
    public static void main(String[] args) {
 
        CurrentThreadTest t1= new CurrentThreadTest("haima");
        String threadName1=t1.currentThread().getName();
        String threadName2=Thread.currentThread().getName();
        String t1ThreadName = t1.getName();
 
        System.out.println(threadName1);
        System.out.println(threadName2);
        System.out.println(t1ThreadName);
   }
}
```
因为t1.currentThread()和Thread.currentThread()获得都是当前线程，而当前线程是主线程，所以前两次打印都是main,执行结果：
```
main
main
haima
```
###    getName() && setName(String name)
**public final String getName()**  
**public final synchronized void setName(String name)**  
通过这两个方法可以设置线程的名字，线程名不是唯一的，两个线程名可以相同，相对简单，不赘述了

 
###   suspend()&&resume()
**public final void suspend()**    暂停线程
**public final void resume()**   从暂停中恢复
以上两个方法都是成员方法，两个方法都已经过时，不建议使用了

###    setPriority()
**public final void setPriority(int newPriority)** 设置线程优先级
首先说说线程优先级概念，线程的调度是JVM，具有随机性，线程优先级越高获得JVM调度的机会要大一些，从而拥有更多机会获得cpu资源。
优先级在1-10之间，设置不在这个范围会抛出IllegalArgumentException，Thread类自带三个常量**MIN_PRIORITY**,**NORM_PRIORITY**,**MAX_PRIORITY**分布代表最小优先级1，默认优先级5，最大优先级10。经博主测试MAX_PRIORITY和MIN_PRIORITY都没多大区别（喔晶）
子线程的优先级会从发起线程的父线程那里继承下来
```java
public class MainTest {
    public static void main(String[] args) {
        Thread.currentThread().setPriority(6);//主线程设置优先级为6
        PriorityThread t1= new PriorityThread();
        t1.start();
        System.out.println(t1.getPriority());//打印子线程优先级
   }
}
```
执行结果：
```
6  
```

###    yield()
**public static native void yield();**    将资源让出，让线程从运行状态转换为可运行状态，这个能够很大程度的将自己的优先级降低


###    setDaemon()
**public final void setDaemon(boolean on)**    设置为守护线程
守护线程概念。线程分为守护线程和非守护线程。守护线程不能独立存在，如果非守护线程都执行结束了，那么守护线程也不会存在了。典型的守护线程是Java垃圾回收线程。
```java
//线程类
public class DaemonThread extends Thread{
    @Override
    public void run() {
        // TODO Auto-generated method stub
        while (true) {
            System.out.println("aliving");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}
//测试类
public class MainTest {
    public static void main(String[] args) {
 
        DaemonThread t1= new DaemonThread();
        t1.setDaemon(true);
        t1.start();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
   }
}
```
执行结果：
```
//...省略部分日志
aliving
aliving
aliving
aliving
game over    //进程停止
```



