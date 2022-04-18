#Java 
如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。今天我们就来详细讲解一下Java的线程池，首先我们从最核心的ThreadPoolExecutor类中的方法讲起，然后再讲述它的实现原理，接着给出了它的使用示例，最后讨论了一下如何合理配置线程池的大小。

# Java中的 ThreadPoolExecutor 类

　java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，构造器如下:

## 构造函数

```
 //1
 	public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
    
 //2
     public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }
    
 //3
  	public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
    
 //4
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
    
```

可以从代码看出，前三个的构造函数都是调用第四个构造函数来构造的，下面是各个形参：

**corePoolSize**：线程池的大小，有任务到来之前就创建corePoolSize个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；

**maximumPoolSize**：线程池所能建立的最大线程数量，它表示在线程池中最多能创建多少个线程；

**keepAliveTime**：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

workQueue：等待队列，用于存放等待执行的任务；这个队列有几个选择：

```
 ArrayBlockingQueue;//数组的先进先出队列，此队列创建时必须指定大小；
 LinkedBlockingQueue;//基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；
 SynchronousQueue;//这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。
 //一般我们会用 LinkedBlockingQueue和SynchronousQueue，这个参数的选择很重要。
```

threadFactory：线程工厂，主要用来创建线程；

handler：表示当拒绝处理任务时的策略，有以下四种取值：

```
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
```

- unit：参数keepAliveTime的时间单位



## ThreadPoolExecutor类的重要属性

### 线程池状态

```
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```

### 工作队列和任务等待队列

```
private final BlockingQueue<Runnable> workQueue;//等待队列
private final HashSet<Worker> workers = new HashSet<Worker>();//工作队列
```

### 线程的存活时间和大小

```
private volatile long keepAliveTime;// 线程存活时间   
private volatile boolean allowCoreThreadTimeOut;// 是否允许核心线程存活   
private volatile int corePoolSize;// 核心池大小   
private volatile int maximumPoolSize; // 最大池大小   
private volatile int poolSize; //当前池大小   
private int largestPoolSize; //最大池大小，区别于maximumPoolSize，是用于记录线程池曾经达到过的最大并发,理论上小于等于maximumPoolSize。
```

### 线程工厂和拒绝策略

```
private volatile RejectedExecutionHandler handler;// 拒绝策略，用于当线程池无法承载新线程是的处理策略。  
private volatile ThreadFactory threadFactory;// 线程工厂，用于在线程池需要新创建线程的时候创建线程 
```

