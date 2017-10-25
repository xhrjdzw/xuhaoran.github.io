抽象类和接口的区别
重载和重写的区别
重载方法是否可以根据返回类型进行区分
Threadpool中线程是如何做到重用的



# JAVA线程池知识总结

## 简介

>线程的使用在java中占有极其重要的地位，jdk1.5之后加入了java.util.concurrent包，这个包中主要介绍java中线程以及线程池的使用。线程池是一种多线程处理形式，处理过程中将任务添加到队列，然后在创建线程后自动启动这些任务。



## 线程池

### 线程池的作用

当一个服务器接受到大量短小线程的请求时，使用线程池技术是非常合适的，它可以大大减少线程的创建和销毁次数，提高服务器的工作效率。<br>

线程池主要用来解决线程生命周期开销问题和资源不足问题。通过对多个任务重复使用线程，线程创建的开销就被分摊到了多个任务上了，而且由于在请求到达时线程已经存在，所以消除了线程创建所带来的延迟。这样，就可以立即为请求服务，使用应用程序响应更快。另外，通过适当的调整线程中的线程数目可以防止出现资源不足的情况。<br>



### 为什么要用线程池

+ 1.减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。

+ 2.可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。



### 线程池的优点

+ 降低资源消耗：通过重用已经创建的线程来降低线程创建和销毁的消耗;<br>

+ 提高响应速度：任务到达时不需要等待线程创建就可以立即执行;<br>

+ 提高线程的可管理性：线程池可以统一管理、分配、调优和监控;<br>



## JAVA多线程池讲解

### java自带线程池--ThreadPoolExecutor

java的线程池支持主要通过ThreadPoolExecutor来实现，我们使用的ExecutorService的各种线程池策略都是基于ThreadPoolExecutor实现的，所以ThreadPoolExecutor十分重要。要弄明白各种线程池策略，必须先弄明白ThreadPoolExecutor。<br>



ThreadPoolExecutor的完整构造方法：<br>



```java

Executor executor = new ThreadPoolExecutor(int corePoolSize,

int maximumPoolSize, long keepAliveTime,

TimeUnit unit,

BlockingQueue<Runnable> workQueue,

ThreadFactory threadFactory, RejectedExecutionHandler handler

);



```



具体参数详解：<br>

<table>

<tr>

<th>参数名</th><th>作用</th>

</tr>

<tr> <th>corePoolSize</th>

<th> 核心线程池大小 </th>

</tr>

<tr>

<th>maximumPoolSize</th> <th>线程池最大容量大小</th>

</tr>

<tr>

<th>keepAliveTime </th>

<th>线程池中超过corePoolSize数目的空闲线程最大存活时间</th>

</tr>



<tr>

<th>TimeUnit </th>

<th>时间单位</th>

</tr>



<tr>

<th>workQueue </th>

<th>阻塞任务队列</th>

</tr>



<tr>

<th>threadFactory </th>

<th>新建线程工厂</th>

</tr>



<tr>

<th>handler </th>

<th> 线程拒绝策略（当提交任务数超过maxmumPoolSize+workQueue之和时，任务会交给RejectedExecutionHandler来处理）</th>

</tr>



</table>





<br>



### java线程池和线程比较

执行一个异步的任务，之前通常的方式是new Thread；但对于多个重复性的任务，这样可能会导致无限制的新建线程，过多的占用系统资源有死机的风险。

相比new thread而言，线程池的好处在于：减少了对象的创建、消亡的开销



```java

new Thread(new Runnable() {

public void run() {

}

}

).start();

```

这种用法的弊端在于每次新建的对象性能很差，并且缺乏统一的管理，可能无限制的新建线程，当线程过多会导致系统资源死机等问题。<br>

相比new Thread, java提供的线程池可以重复使用存在的线程，减少了对象的创建消亡过程，同时可以有效控制最大线程并发数，提高了系统资源的使用率。



### java四种线程池的用法

根据以上比较总结，java线程池的基本概念已经有所了解，这部分着重以代码形式来展示java的四种线程池的示例<br>

MyThread.java<br>



```java

public class MyThread extends Thread {

public void run() {

System.out.println(Thread.currentThread().getName() + "正在执行。。。");

}

}

```



+ **newSingleThreadExecutor**<br>

创建一个单线程的线程池，这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。<br>



```java

public class TestnewSingleThreadExecutor {

public static void main(String[] args) {

//创建一个可重用固定线程数的线程池

ExecutorService pool = Executors. newSingleThreadExecutor();

//创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口

Thread t1 = new MyThread();

Thread t2 = new MyThread();

Thread t3 = new MyThread();

Thread t4 = new MyThread();

Thread t5 = new MyThread();

//将线程放入池中进行执行

pool.execute(t1);

pool.execute(t2);

pool.execute(t3);

pool.execute(t4);

pool.execute(t5);

//关闭线程池

pool.shutdown();

}

}

```

输出结果：<br>



```

pool-1-thread-1正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-1正在执行。。。

```



+ **newFixedThreadPool**



创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。<br>



```java

public class TestNewFixedThreadPool {

public static void main(String[] args) {

//创建一个可重用固定线程数的线程池

ExecutorService pool = Executors.newFixedThreadPool(2);

//创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口

Thread t1 = new MyThread();

Thread t2 = new MyThread();

Thread t3 = new MyThread();

Thread t4 = new MyThread();

Thread t5 = new MyThread();

//将线程放入池中进行执行

pool.execute(t1);

pool.execute(t2);

pool.execute(t3);

pool.execute(t4);

pool.execute(t5);

//关闭线程池

pool.shutdown();

}

}

```

输出结果<br>



```

pool-1-thread-2正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-2正在执行。。。

pool-1-thread-1正在执行。。。

pool-1-thread-2正在执行。。。

```



+ **newCachedThreadPool**



创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，

那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。<br>



```java

public class TestNewCachedThreadPool {

public static void main(String[] args) {

//创建一个可重用固定线程数的线程池

ExecutorService pool = Executors.newCachedThreadPool();

//创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口

Thread t1 = new MyThread();

Thread t2 = new MyThread();

Thread t3 = new MyThread();

Thread t4 = new MyThread();

Thread t5 = new MyThread();

//将线程放入池中进行执行

pool.execute(t1);

pool.execute(t2);

pool.execute(t3);

pool.execute(t4);

pool.execute(t5);

//关闭线程池

pool.shutdown();

}

}

```



输出结果：



```java

pool-1-thread-1正在执行。。。

pool-1-thread-2正在执行。。。

pool-1-thread-3正在执行。。。

pool-1-thread-4正在执行。。。

pool-1-thread-5正在执行。。。

```



+ **newScheduledThreadPool**



创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。<br>



```java

public class TestNewScheduledThreadPool** {

public static void main(String[] args) {

ScheduledThreadPoolExecutor exec = new ScheduledThreadPoolExecutor(1);

exec.scheduleAtFixedRate(new Runnable() {// 每隔一段时间就触发异常

@Override

public void run() {

System.out.println("================");

}

}, 1000, 5000, TimeUnit.MILLISECONDS);

exec.scheduleAtFixedRate(new Runnable() {// 每隔一段时间打印系统时间

@Override

public void run() {

System.out.println(System.nanoTime());

}

}, 1000, 2000, TimeUnit.MILLISECONDS);

}

}

```

输出结果：



```java

================

88820986619756

88822977207505

88824978506276

================

88826977188305

88828976688253

```



### 线程的提交

+ ThreadPoolExecutor被初始化好之后便可以提交线程任务，线程的提交方法主要是execute,通过这个方法可以向线程池提交一个任务，交由线程池去执行.<br>



```java

executor.execute(new Runnable() {

@Override

public void run() {

runTask(group, scanTime);

}

});

```



+ 第二种提交方式为 submit，submit有返回值，而execute没有<br>



```java

/**

* submit(Runnable x) 返回一个future。可以用这个future来判断任务是否成功完成。

*/

Future future = pool.submit(new RunnableTest("Task2"));

try {

if(future.get()==null){//如果Future's get返回null，任务完成

System.out.println("任务完成");

}

```



### 线程池的关闭



ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()：<br>



+ shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务<br>

+ shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务



## 总结



虽然线程池能大大提高服务器的并发性能，但使用它也会存在一定风险。与所有多线程应用程序一样，用线程池构建的应用程序容易产生各种并发问题，如对共享资源的竞争和死锁。此外，如果线程池本身的实现不健壮，或者没有合理地使用线程池，还容易导致与线程池有关的死锁、系统资源不足和线程泄漏等问题。<br>



+ **死锁**

任何多线程应用程序都有死锁风险。

虽然任何多线程程序都有死锁的风险，但线程池还会导致另外一种死锁。在这种情形下，假定线程池中的所有工作线程都在执行各自任务时被阻塞，它们都在等待某个任务A的执行结果。而任务A依然在工作队列中，由于没有空闲线程，使得任务A一直不能被执行。这使得线程池中的所有工作线程都永远阻塞下去，死锁就这样产生了。<br>



+ **系统资源不足**

如果线程池中的线程数目非常多，这些线程会消耗包括内存和其他系统资源在内的大量资源，从而严重影响系统性能。<br>



+ **并发错误**

最好使用现有的、比较成熟的线程池。例如，直接使用java.util.concurrent包中的线程池类。<br>



+ **线程泄漏**

使用线程池的一个严重风险是线程泄漏。对于工作线程数目固定的线程池，如果工作线程在执行任务时抛出 RuntimeException 或Error，并且这些异常或错误没有被捕获，需要做捕获异常处理，否则遇到此问题，未做处理，



## 参考资料及网站

+ [聊聊并发（三）Java线程池的分析和使用](并发编程网 - ifeve.com)

+ [深入理解java线程池—ThreadPoolExecutor](深入理解java线程池—ThreadPoolExecutor)

+ [ThreadPoolExecutor机制 ](JAVA进阶----ThreadPoolExecutor机制 - 无量的IT生活 - ITeye博客)

+ [服务器处理多个用户请求的解决方案(返回多个用户输入的信息)](java网络编程二：服务器处理多个用户请求的解决方案(返回多个用户输入的信息) - 王爵的技术博客 - 博客园)

+ [java多线程详解(7)-线程池的使用](java多线程详解(7)-线程池的使用 - weiguo21 - 博客园)

+ [ShutDown以及ShutDownNow解析](Java线程池---ShutDown以及ShutDownNow解析)

+ [java 线程池execute()原理](Java线程池ThreadPoolExecutor使用和分析(二) - execute()原理)