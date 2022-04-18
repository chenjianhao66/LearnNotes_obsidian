#Java 

# 一、进程和线程

## 1.1 什么是进程和线程？

详情请参考文章：

**简单的来说**

进程是为了刻画系统内部动态状况、描述运行程序活动规律而引进的；为了使多个程序并发执行，以便改善资源利用率和提高系统效率。

如果说进程是为了实现使多个程序并发执行，以便改善资源利用率和提高系统效率，那么**引入线程**是为了减少程序并发执行时所付出的时空开销，使得并发粒度更细、并发性更好。

总之，进程和线程的提出极⼤的提⾼了操作提供的性能。进程让操作系统的并发性成为了可能，⽽线程让进程的内部并发成为了可能。

**多进程的⽅式也可以实现并发，为什么我们要使⽤多线程？**

多进程⽅式确实可以实现并发，但使⽤多线程，有以下⼏个好处：

- 进程间的通信⽐较复杂，⽽线程间的通信⽐较简单，通常情况下，我们需要使⽤共享资源，这些资源在线程间的通信⽐较容易。
- 进程是重量级的，⽽线程是轻量级的，故多线程⽅式的系统开销更⼩。

## 1.2 进程和线程的区别

进程是⼀个独⽴的运⾏环境，⽽线程是在进程中执⾏的⼀个任务。他们两个本质的区别是是否单独占有内存地址空间及其它系统资源（⽐如I/O）：

- 进程单独占有⼀定的内存地址空间，所以进程间存在内存隔离，数据是分开的，数据共享复杂但是同步简单，各个进程之间互不⼲扰；⽽线程共享所属进程占有的内存地址空间和资源，数据共享简单，但是同步复杂。
- 进程单独占有⼀定的内存地址空间，⼀个进程出现问题不会影响其他进程，不影响主程序的稳定性，可靠性⾼；⼀个线程崩溃可能影响整个程序的稳定性，可靠性较低。
- 进程单独占有⼀定的内存地址空间，进程的创建和销毁不仅需要保存寄存器和栈信息，还需要资源的分配回收以及⻚调度，开销较⼤；线程只需要保存寄存器和栈信息，开销较⼩。

另外⼀个**重要区别是，进程是操作系统进⾏资源分配的基本单位，⽽线程是操作系统进⾏调度的基本单位**，即CPU分配时间的单位 。





# 二、Java的多线程实现

## 2.1 Thread类和Runnable接⼝

JDK提供了  `Thread`  类和  `Runnalble`  接⼝来让我们实现⾃⼰的“线程”类。

1. 继承  `Thread`  类，并重写  `run`  ⽅法；

2. 实现  `Runnable`  接⼝的  `run`  ⽅法；

   

### 2.1.1 继承 Thread 类

```
public class Demo {
    public static class MyThread extends Thread {
        @Override
        public void run() {
        	System.out.println("MyThread");
        }
    }
    public static void main(String[] args) {
        Thread myThread = new MyThread();
        myThread.start();
    }
}
```

> 一个线程要调用了 start方法之后才能算是启动该线程。



### 2.1.2 实现 Runnable 接口

```
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

public class Demo {
public static class MyThread implements Runnable {
    @Override
    public void run() {
    	System.out.println("MyThread");
    }
}
    public static void main(String[] args) {
        new MyThread().start();
            // Java 8 函数式编程，可以省略MyThread类
        new Thread(() -> {
        	System.out.println("Java 8 匿名内部类");
        }).start();
    }
}

```

### 2.1.3 Thread类的构造方法

实际情况下，我们会直接使用下面的两个方法：

```
Thread(Runnable target)
Thread(Runnable target, String name)
```



### 2.1.4 Thread类的常用方法

这⾥介绍⼀下Thread类的⼏个常⽤的⽅法：

- currentThread()：静态⽅法，返回对当前正在执⾏的线程对象的引⽤；

- start()：开始执⾏线程的⽅法，java虚拟机会调⽤线程内的run()⽅法；

- yield()：yield在英语⾥有放弃的意思，同样，这⾥的yield()指的是当前线程愿意让出对当前处理器的占⽤。这⾥需要注意的是，就算当前线程调⽤了yield()⽅法，程序在调度的时候，也还有可能继续运⾏这个线程的；

- sleep()：静态⽅法，使当前线程睡眠⼀段时间；

- join()：使当前线程等待另⼀个线程执⾏完毕之后再继续执⾏，内部调⽤的是Object类的wait⽅法实现的；



## 2.2 Callable、Future与FutureTask

通常来说，我们使⽤  `Runnable`  和 `Thread`  来创建⼀个新的线程。但是它们有⼀个弊端，就是  **`run`  ⽅法是没有返回值的**。⽽有时候我们希望开启⼀个线程去执⾏⼀个任务，并且这个任务执⾏完成后有⼀个返回值。

JDK使用了 `Callable`  接⼝与  `Future`  类解决了这个问题。

### 2.2.1 Callable接口

该接口与 `Runnable` 接口类似，只有一个方法，不同于 `Runnable` 接口，它有返回值且支持泛型。

```
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

它会返回⼀个 `Future` 类实例 ，我们后续的程序可以通过这个  `Future`  的  `get`  ⽅法得到结果。

```
// ⾃定义Callable
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        // 模拟计算需要⼀秒
        Thread.sleep(1000);
        return 2;
    }
    public static void main(String args[]){
        // 使⽤
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        // 注意调⽤get⽅法会阻塞当前线程，直到得到结果。
        // 所以实际编码中建议使⽤可以设置超时时间的重载get⽅法。
        System.out.println(result.get());
    }
}
```

输出结果：

```
2
```

### 2.2.2 Future接口

接口方法

```
public interface Future<V> {
	boolean cancel(boolean mayInterruptIfRunning);
	boolean isCancelled();
	boolean isDone();
	V get() throws InterruptedException, ExecutionException;
	V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

 `cancel`  ⽅法是试图取消⼀个线程的执⾏。
注意是试图取消，并不⼀定能取消成功。因为任务可能已完成、已取消、或者⼀些其它因素不能取消，存在取消失败的可能。  boolean  类型的返回值是“是否取消成功”的意思。参数  `paramBoolean`  表示是否采⽤中断的⽅式取消线程执⾏。

所以有时候，为了让任务有能够取消的功能，就使⽤  `Callable`  来代替 ` Runnable ` 。如果为了可取消性⽽使⽤  `Future`  但⼜不提供可⽤的结果，则可以声明  `Future<?>`形式类型、并返回  `null`  作为底层任务的结果。

### 2.2.3 FutureTask实现类

`FutureTask`类实现了`RunnableFuture<V>`接口，`RunnableFuture<V>`接口同时继承了 `Runnable`和`Future`接口；Future只是一个接口且具有很多的方法，实现起来比较麻烦，Java帮我们实现了这个Furture接口并且包装了起来，这个类就是 `FutureTask` 类。

后面如果想要有返回值的线程，通过FutureTask类和线程池类Executors配合使用。



## 三、Java线程之间的通信

合理的使⽤Java多线程可以更好地利⽤服务器资源。⼀般来讲，线程内部有⾃⼰私有的线程上下⽂，互不⼲扰。但是当我们需要多个线程之间相互协作的时候，就需要我们掌握Java线程的通信⽅式。

说线程就绕不过进程，那么进程之间的关系通信又分为竞争和同步；线程于进程大同小异，用的最多的就是同步，那么Java是怎么实现线程同步的呢？

主要方式有3种，分别是：锁、等待/通知、信号量

## 3.1 锁

在Java中，锁的概念都是基于对象的，所以我们⼜经常称它为对象锁。线程和锁的关系，我们可以⽤婚姻关系来理解。⼀个锁同⼀时间只能被⼀个线程持有。也就是说，⼀个锁如果和⼀个线程“结婚”（持有），那其他线程如果需要得到这个锁，就得等这个线程和这个锁“离婚”（释放）。

使用`对象锁`

```
public class ObjectLock {
private static Object lock = new Object();
    static class ThreadA implements Runnable {
        @Override
        public void run() {
           synchronized (lock) {
               for (int i = 0; i < 100; i++) {
               		System.out.println("Thread A " + i);
           		}
    		}
    	}
    }
    static class ThreadB implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 100; i++) {
                System.out.println("Thread B " + i);
                }
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(10);
        new Thread(new ThreadB()).start();
    }
}
```

这里声明了一个 `static` 修饰的变量，一个类只有一个，而且使用了 `synchronized` 对这个对象上锁，在执行过程中，线程B就会等待线程A的锁释放掉，才能执行线程B的代码。

> 这里休眠了10毫秒，是因为怕程序使用重排序把线程B先启动允许，保证线程A获取到锁先执行。



## 3.2 等待/通知

上⾯⼀种基于“锁”的⽅式，线程需要不断地去尝试获得锁，如果失败了，再继续尝试。这可能会耗费服务器资源。⽽等待/通知机制是另⼀种⽅式。Java多线程的等待/通知机制是基于  Object  类的  wait()  ⽅法和  notify()  , notifyAll()  ⽅法来实现的。

> 需要注意的是等待/通知机制使⽤的是使⽤同⼀个对象锁，如果你两个线程使⽤的是不同的对象锁，那它们之间是不能⽤等待/通知机制通信的。


未完待续.....

