# 创建多线程方式
- 实现 Runnable 接口
- 实现 Callable 接口
- 继承 Thread 类

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。
## 实现 Runnable 接口
```Java
public class MyRunnable implements Runnable(){
    @Override
    public void run(){ ... }
}
//使用
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```
## 实现 Callable 接口
与 Runnable 相比，Callable 可以有返回值，返回值通过 `FutureTask` 进行封装。
```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {//可以有返回值
        return 123;
    }
}
//使用
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```
## 继承 Thread 类
```java

public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
//使用
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```
## 接口 VS 继承类
实现接口会更好一些，因为：
- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。
## 线程池
线程池的优点：
- 降低资源消耗，通过重复利⽤已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度，当任务到达时，无需等待线程创建即可立即执行。
- 提高线程的可管理性，线程池可以进⾏统⼀的分配线程，调优和监控。

线程池核心实现类 `ThreadPoolExcutor` 

[线程池实现原理-美团团队](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

###  1 excute() 和 sumbit() 方法的区别
1. excute() 用于提交不需要返回值的任务，无法判断任务是否被线程池执行成功。
2. sumbit*()用于需要返回值的任务。线程池会返回一个 `Future` 对象。sumbit() 最终调用的还是 excute()。
### 2 ThreadPoolExcutor 构造函数
`ThreadPoolExcutor` 有四个构造方法，以下为最长的，剩下三个指定了一些默认值：
```java
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize, 
    long keepAliveTime,TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,RejectedExecutionHandler handler);
```
构造函数中参数分析：
1. `corePoolSize`： 核心线程数量。最小可以同时运行的线程数量
2. `maxmumPoolSize`：线程池最大线程数，表示在线程池中最多能创建多少个线程。
3. `workQueue`：阻塞队列，当新任务来的时候会先判断当前运⾏的线程数量是否达到核⼼线程数，如果达到的话，新任务就会被存放在队列中。
4. `keepAliveTime`: 当池中的线程数量大于 `corePoolSize` 时，核心线程外的线程销毁前等待的时间。
5. `threadFactory`：`excutor` 创建线程时用到
6. `handler`：饱和策略，如果当前同时运⾏的线程数量达到最⼤线程数量并且队列也已经被放满了任
时， ThreadPoolTaskExecutor 定义的⼀些策略:
    - 抛出异常，拒绝新任务的处理；
    - 调用自己的线程执行任务，如果程序可以承受延迟并且不能丢弃任何一个任务请求时使用；
    - 不处理，直接丢弃；
    - 丢弃最早的未处理的任务请求。

### 3 `ThreadPoolExcutor` 运行状态分为5种：
![](ThreadPoolExcutorLife.png)
生命周期转换：
![](ThreadPoolExcutorLifeChange.png)
### 4 excute() 方法的任务调度过程
所有任务的调度都是由execute方法完成的，执行过程：
1. 检测线程池运行状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执行任务。
2. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务。
3. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
4. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
5. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

流程图如下:   
![](ThreadPoolExcutor_Excute.png)

## 线程间的协作
### join()
join()方法的作用是等待**调用该方法的线程**在执行完 `run()` 方法后在执行后面的方法。

例如：线程 B 中的 `a.join()` 会保证先执行完 A 线程，再执行 B 线程。
```java
 class B extends Thread {
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
````
### wait()