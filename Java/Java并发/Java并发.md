#JAVA 并发
* [一.使用线程](#1)
  * [继承Thread类](#1.1)
  * [实现Runnable接口](#1.2)
  * [实现Callable接口](#1.3)
* [二.基础线程机制](#2)
  * [Excutor](#2.1)
  * [Daemon](#2.2)
  * [sleep](#2.3)
  * [yield](#2.4)
* [三.中断](#3)
  * [InterruptedException](#3.1)
  * [interupted](#3.2)
  * [Excutor的中断操作](#3.3)
* [四.互斥同步](#4)
  * [synchronized](#4.1)
  * [ReentrantLock(JUC)](#4.2)
  * [比较](#4.3)
* [锁](#lock)
* [五.线程之间的协作](#5)
  * [join()](#5.1)
  * [wait() notify() notifyAll()](#5.2)
  * [wait() sleep()方法的区别](#5.3)
  * [await() signal() signalAll() (JUC)](#5.4)
* [六.线程状态](#6)
  * [New](#6.1)
  * [Runable](#6.2)
  * [Blocked](#6.3)
  * [WAITING](#6.4)
  * [TIMED_WAITING](#6.5)
  * [TERMINATED](#6.6)


## <span id="1">一.使用线程</span>
### <span id="1.1">继承Thread类</span>
重写run()方法，
当调用start()方法时启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的run()方法。

```java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

### <span id="1.2">实现Runnable接口</span>
需要重写Runnable接口中的run()方法
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // ...
    }
}
```
使用Runnable实例再创建一个Thread实例，然后调用Thread实例的start()方法来启动线程。
```java
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

### <span id="1.3">实现Callable接口</span>
与Runnable相比，Callable 可以有返回值，返回值通过FutureTask进行封装
```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}

```


### 接口实现与继承Thread
实现接口会更好一点，因为：
* JAVA不支持多重继承，因此继承了Thread类就无法继承其它类，但是可以实现多个接口
* 类可能只要求可执行就行，继承整个Thread类开销过大。

## <span id="2">二.基础线程机制</span>
### <span id="2.1">Excutor</span>
Excutor管理多个异步任务的执行，而无需程序员显式地管理线程的声明周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种Excutor：
* CachedThreadPool:一个任务创建一个线程
* FixedThreadPool:所有任务只能使用固定大小的线程
* SingleThreadExcutor:相当于大小为1的FixedThreadPool。
```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

### <span id="2.2">Daemon</span>
守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。
```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```


### <span id="2.3">sleep()</span>
Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。
```java

public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

### <span id="2.4">yield()</span>
对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。
```java
public void run() {
    Thread.yield();
}
```


## <span id="3">三、中断</span>

### <span id="3.1">InterruptedException</span>
通过调用一个线程的interrupt()来中断该线程，如果该线程处于阻塞，限期等待或者无限期等待状态，那么就会抛出InterruptedException,从而提前结束该线程。但是不能中断I/O阻塞和synchronized 锁阻塞。

对于以下代码，在main()中启动一个线程之后再中断它，由于线程中调用了Thread.sleep()方法，因此会抛出一个InterruptedException,从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

### <span id="3.2">interrupted</span>
如果一个线程的run()方法执行一个无限循环,并且没有执行sleep()等会抛出InterruPTException的操作，那么调用线程的interrupt()方法就无法使线程提前结束。

但是调用interrupt()方法会设置线程的中断标记，此时调用interrupted()方法会返回true。因此可以在循环体使用interrupted()方法来判断线程是否处于中断状态，从而提前结束线程。

```java
public class InterruptExample {

    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            while (!interrupted()) {
                // ..
            }
            System.out.println("Thread end");
        }
    }
}
public static void main(String[] args) throws InterruptedException {
    Thread thread2 = new MyThread2();
    thread2.start();
    thread2.interrupt();
}


Thread end

```

### <span id="3.3">Executor的中断操作</span>
调用Executor的shutdown()方法会等待线程都执行完毕之后再关闭，但是如果调用的是shutdownNow()方法，则相当于调用每个线程的interrupt()方法。
以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。
```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)

```
如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。
```java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

##<span id="4">四、互斥同步</span>
Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是JVM实现的synchronized,而另一个是JDK实现的ReentrantLock.

###<span id="4.1">synchronized</span>
####1. 同步一个代码块
```java
public void func() {
    synchronized (this) {
        // ...
    }
}
```
它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用ExecutorService执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，
```java
public class SynchronizedExample {

    public void func1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}


public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
}
```
>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。
```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}

```
>0 0 1 2 3 4 5 6 7 1 8 2 9 3 4 5 6 7 8 9 
#### 2.同步一个方法
```java
public synchronized void func () {
    // ...
}
```
它和同步代码块一样，作用于同一个对象。

#### 3.作用一个类
```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```
作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。
```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
```
>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 

#### 4.同步一个静态方法
```java
public synchronized static void fun(){
    //...作用于整个类
}

```



###<span id="4.2">ReentrantLock</span>
ReentrantLock 是 java.util.concurrent ==（J.U.C）== 包中的锁。

```java
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁。
        }
    }
}
public static void main(String[] args) {
    LockExample lockExample = new LockExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> lockExample.func());
    executorService.execute(() -> lockExample.func());
}
```

>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9


###<span id="4.3">比较</span>
####1.锁的实现
sychronized 是JVM实现的，而ReentrantLock是JDK实现的。

#### 2.性能
新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

#### 3.等待可中断
当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

#### 4.公平锁
公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

#### 5.锁绑定多个条件
一个 ReentrantLock 可以同时绑定多个 Condition 对象。
Condition 用来通知其他

###使用选择
除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## <span id="5">五.线程之间的写作</span>
当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其他部分之间完成，那么就需要对线程进行协调。
###<span id="5.1"> join()</span>
在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。
```java
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
A
B
```

###<span id="5.2">wait() notify() notifyAll()
调用wait()使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其他下次呢过那会调用notify()或者notifyAll()来唤醒挂起的线程。


它们都属于Object的一部分，而不属于Thread。

只能用在==同步方法或者同步代码块==中使用，否则会在运行时抛出 **IllegalMonitorStateException。**


使用wait()挂起期间，线程会释放锁。这是因为，如果没有释放锁，**其他线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行notify(),或者nofityAll()来唤醒挂起的线程，造成死锁**。
```java
public class WaitNotifyExample {

    public synchronized void before() {
        System.out.println("before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after");
    }
}
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    WaitNotifyExample example = new WaitNotifyExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}

```
>before
after

#### <span id="5.3">wait()和sleep()的区别</span>

* wait()是Object的方法，而sleep()是Thread的静态方法;
* wait()会释放锁，sleep()不会
* wait()方法只能在同步方法或同步代码块中使用， 而sleep()可以在任何地方使用


###<span id="5.4">await() signal() signalAll() </span>
java.util.concurrent （JUC）类库中提供了Condition 类实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。
```java
public class AwaitSignalExample {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}

```
before
after

### Condition的优点
可以实现精准唤醒

## <span id="lock">锁</span>

### synchronized同步一个方法时 锁的对象是方法的调用者

```java
class Phone{
    // synchronized 锁的对象是方法的调用者！、
// 两个方法用的是同一个锁，谁先拿到谁执行！s
    public synchronized void sendSms(){
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public synchronized void call(){
        System.out.println("打电话");
    }
}
```
一个对象
```java
public class Lock1 {
    public static void main(String[] args) {
        Phone phone=new Phone();
        new Thread(()->{
            phone.sendSms();
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(()->{
            phone.call();
        }).start();

    }
}
```
输出结果
>发短信
打电话

两个不同对象
```java
public class Lock2 {
    public static void main(String[] args) {
        Phone2 phone2=new Phone2();
        Phone2 phone1=new Phone2();

        new Thread(()->{
            phone1.sendSms();
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(()->{
            phone2.hello();
        }).start();
    }
}
```
输出结果
>打电话
发短信


### 同步一个静态方法时， 锁的对象是类的模板

```java
class Phone3{

    public static synchronized void sendSms(){
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public static synchronized void call(){
        System.out.println("打电话");
    }

}
```

两个对象，两个静态同步方法
```java
public class Lock3{
    public static void main(String[] args) {
        Phone3 phone1=new Phone3();
        Phone3 phone2=new Phone3();

        new Thread(()->{
            phone1.sendSms();
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(()->{
            phone2.call();
        }).start();
    }
}
```
输出结果
>发短信
打电话

### 同步一个静态方法和 普通方法时， 锁不是同一把
```java
class Phone4{
    public static synchronized void sendSms(){
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public synchronized void call(){
        System.out.println("打电话");
    }
}
```

一个对象
```java
public class Lock4 {
    public static void main(String[] args) {
        Phone4 phone1 =new Phone4();
        Phone4 phone2 = new Phone4();

        new Thread(()->{
            phone1.sendSms();
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone1.call();
        }).start();

    }
}
```
实验结果
>打电话
发短信

两个对象
```java
public class Lock4 {
    public static void main(String[] args) {
        Phone4 phone1 =new Phone4();
        Phone4 phone2 = new Phone4();

        new Thread(()->{
            phone1.sendSms();
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone2.call();
        }).start();

    }
}
```
输出结果
>打电话
发短信


##<span id="6"> 六.线程状态</span>
一个线程只能处于一种状态，并且这里的线程状态特制Java虚拟机的线程状态，不能反应线程在特定操作系统下的状态。

###<span id="6.1">初始(New)</span>
新创建了一个线程对象，但还没有调用start()方法。

###<span id="6.2">运行(Runable)</span>
Runable包括了操作系统线程状态中的Running 和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着CPU为它分配执行时间。

###<span id="6.3">阻塞（Blocked）</span>
请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。

###<span id="6.4">无限期等待(Waitting)</span>
等待其它线程显式地唤醒。
阻塞和等待的区别在于，阻塞是被动的，它是在等待获取monitor lock。而等待是主动的，通过调用Object.wait()等方法进入。
| 进入方法| 退出方法 |  
| :---         |     :---:      |  
| 没有设置Timeout参数的Object.wait()方法   | Object.notify()/Object.notifyAll()     | 
|没有设置 Timeout 参数的 Thread.join() 方法    | 被调用的线程执行完毕      | 
|LockSupport.park()方法| LockSupport.unpark(Thread)|


###<span id="6.5">限期等待(TIMED_WAITING)</span>
无需等待其他线程显式地唤醒，在一定时间之后会被系统自动唤醒。
|进入方法|退出方法|
|:--|:--:|
|Thread.sleep()方法|时间结束|
|设置了Timeout参数的Object.wait()方法|时间结束/Object.notify()/Object.notifyAll()|
|设置了Timeout参数的Thread.join()方法|时间结束/被调用的线程执行完毕|
|LockSupport.parkNanos()方法|LockSupport.unpark(Thread)|
|LockSupport.parkUntil()方法|LockSupport.unpark(Thread)|


调用 Thread.sleep()方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用Object.wait()方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。


###<span id="6.6">结束