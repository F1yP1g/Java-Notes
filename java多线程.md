# 1 创建多线程方式
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
## 2 线程池
线程池的优点：
- 降低资源消耗，通过重复利⽤已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度，当任务到达时，无需等待线程创建即可立即执行。
- 提高线程的可管理性，线程池可以进⾏统⼀的分配线程，调优和监控。

线程池核心实现类 `ThreadPoolExcutor` 

[线程池实现原理-美团团队](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

###  1 excute() 和 sumbit() 方法的区别
1. excute() 用于提交不需要返回值的任务，无法判断任务是否被线程池执行成功。
2. sumbit()用于需要返回值的任务。线程池会返回一个 `Future` 对象。sumbit() 最终调用的还是 excute()。
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
6. `handler`：饱和策略，如果当前同时运⾏的线程数量达到最⼤线程数量并且队列也已经被放满了任务时， ThreadPoolTaskExecutor 定义的⼀些策略:
    - 抛出异常，拒绝新任务的处理；
    - 调用自己的线程执行任务，如果程序可以承受延迟并且不能丢弃任何一个任务请求时使用；
    - 不处理，直接丢弃；
    - 丢弃最早的未处理的任务请求。

### 3 `ThreadPoolExcutor` 运行状态分为5种：
![](pic/ThreadPoolExcutorLife.png)
生命周期转换：
![](pic/ThreadPoolExcutorLifeChange.png)
### 4 excute() 方法的任务调度过程
所有任务的调度都是由execute方法完成的，执行过程：
1. 检测线程池运行状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执行任务。
2. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务。
3. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
4. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
5. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

流程图如下:   
![](pic/ThreadPoolExcutor_Excute.png)

## 3 线程间的协作
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
```

# 2 synchronized 关键字
1. **修饰实例方法**: 作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁
2. **修饰静态方法**: 也就是给当前类加锁，会作用于类的所有对象实例，因为静态成员不属于任何一个实例对象，是类成员（ static 表明这是该类的一个静态资源，不管new了多少个对象，只有一份）。所以如果一个线程 A 调用一个实例对象的非静态 synchronized 方法，而线程 B 需要调用这个实例对象所属类的静态 synchronized 方法，是允许的，不会发生互斥现象，因为访问静态 synchronized 方法占用的锁是当前类的锁，而访问非静态 synchronized 方法占用的锁是当前实例对象锁。
3. **修饰代码块**: 指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

Java 早期版本中，synchronized 属于重量级锁，效率低下，JDK 1.6 之后官方从 JVM 层面实现了较大优化，所以现在 synchronized 锁的效率也不错，引入的优化只要有自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少开销。

双重校验锁实现对象单例（线程安全）：
```java
    public class Singleton{
        //volatile 关键字修饰很重要
        private volatile static Singleton uniqueInstance;
        //构造函数私有化
        private Singleton(){};
        //静态方法
        public static Singleton getUniqueInstance(){
            //首先判断对象是否已经实例过
            if(uniqueInstance == null){
                //类对象枷锁
                synchronized(Singleton.class){
                    if(uniqueInstance == null){
                        uniqueInstance = new Singleton();
                    }
                }
            }
            return uniqueInstance;
        }
    }
```
其中，uniqueInstance 采用 volatile 关键字修饰也是很有必要。`uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：
1. 为 uniqueInstance 分配内存空间
2. 初始化 uniqueInstance
3. 将 uniqueInstance 指向分配的内存地址

由于 JVM **指令重排**的特性，顺序执行可能变为1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

使用 volatile 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

## synchronized 和 volatile 的区别
1. volatile 关键字是线程同步的轻量级实现，所以 volatile 性能肯定比 synchronized 关键字要好。但是 volatile 关键字只能用于**变量**而 synchronized 关键字可以修饰方法以及代码块。
2. 多线程访问 volatile 关键字不会发生阻塞，而 synchronized 关键字可能会发生阻塞。
3. volatile 关键字能保证数据的可见性，但不能保证数据的原子性。synchronized 关键字两者都能保证。
4. volatile 关键字主要用于解决变量在多个线程之间的可见性，而 synchronized 关键字解决的是多个线程之间访问资源的同步性。

# 3 ThreadLocal
通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。如果想实现每一个线程都有自己的专属本地变量该如何解决呢？ JDK 中提供的 ThreadLocal 类正是为了解决这样的问题。 ThreadLocal 类主要解决的就是让每个线程绑定自己的值，可以将 ThreadLocal 类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据。

如果你创建了一个 ThreadLocal 变量，那么访问这个变量的每个线程都会有这个变量的本地副本，这也是ThreadLocal 变量名的由来。他们可以使用 get（） 和 set（） 方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题。

# 4 CAS
比较并交换，,CPU 并发原语，操作前检查内存中的值与自己期望的值是否相等，相等则操作。
```java
    //原子类
    AtomicInteger atomicInteger = new AtomicInteger(1);
    //比较并交换操作，先检查值是否为1，是则将 atomicInteger 的值设为2.
	atomicInteger.compareAndSet(1, 2);
```
`compareAndSet` 在 `AtomicInteger` 的源码如下：
```java
    private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
    private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");

    //函数，可发现调用的 Unsafe类下的方法。
    public final boolean compareAndSet(int expectedValue, int newValue) {
        return U.compareAndSetInt(this, VALUE, expectedValue, newValue);
    }
```
**`Unsafe`** 类是 CAS 的底层实现。
```java
    //实现i++的功能的底层源码。
    //没锁，单纯的while循环
    public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);//获取对象o的地址offset内存偏移量下的值，给v
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));//如果v 的值等于o在ofset的值，那么v=v+delta，并返回true退出循环，否则继续循环知道更新完成
        return v;
    }
```
CAS 的缺点：
1. 循环时间长，开销大
2. 只能保证一个共享变量的原子操作
3. ABA问题

ABA 问题

相当于写覆盖，假设有一个变量值为 A，线程 T1 和 T2。
T1 和 T2 都拿到了 A, 在 T1 还没反应过来时， T2 将 A 修改为 B, 然后又修改成 A。 此时 T1 比较自己拿的值和内存中的值发现都是 A，他就认为没人动过，则操作。 

ABA 解决办法：

AtomicStampedReference 版本号原子引用，加了版本号用于识别，每次修改值都有修改版本号。虽然值可能一样，但版本号不同则不行。
```java
//定义整型的时间戳源自引用，默认值 10，初始版本号 1；
    AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<Integer>(10, 1);
    //获取当前的版本号
	int stamp = atomicStampedReference.getStamp();
    //CAS 期望值是1，要修改的新值是2，期望的时间戳是 stamp，更新后的时间戳是2.
   atomicStampedReference.compareAndSet(1, 2, stamp, 2);
```

# java锁
## 公平锁和非公平锁
公平锁：多个线程按照申请锁的顺序来获取锁，类似排队打饭

非公平锁：可能后申请的线程比先申请的线程优先获取锁，在高并发下有可能造成优先级反转或饥饿现象
