# Java多线程和并发

- [一、进程和线程的区别](#一-进程和线程的区别)
- [二、Thread中的start()和run()方法的区别](#二-Thread中的start()和run()方法的区别)
- [三、Thread和Runnable是什么关系](#三-Thread和Runnable是什么关系)
- [四、如何实现处理线程的返回值](#四-如何实现处理线程的返回值)
- [五、线程的状态](#五-线程的状态)
- [六、sleep和wait的区别](#六-sleep和wait的区别)
- [七、notify和notifyAll的区别](#七-notify和notifyAll的区别)
- [八、Yield函数](#八-Yield函数)
- [九、如何中断线程](#九-如何中断线程)
- [十、synchronized](#十-synchronized)
- [十一、synchronized底层实现原理](#十一-synchronized底层实现原理)

## 一、进程和线程的区别

### 进程和线程的由来

- 串行：初期的计算机智能串行执行任务，并且需要长时间等待用户输入
- 批处理：预先将用户的指令，集中成清单，批量串行处理用户指令，仍然无法并发执行
- 进程：进程独占内存空间，保存各自运行状态，相互间不干扰且可以相互切换，并为并发处理任务提供了可能
- 线程：共享进程的内存资源，相互间切换更快速，支持更细粒度的任务控制，使进程内的子任务得以并发执行

### 进程和线程的区别

**进程是资源分配的最小单位，线程是CPU调度的最小单位**

- 所有与进程相关的资源，都被记录在PCB（进程控制块,是系统为了管理进程设置的一个专门的数据结构。系统用它来记录进程的外部特征，描述进程的运动变化过程）中；
- 进程是抢占处理机的调度单位；线程属于某个进程，共享其资源；
- 线程只由堆栈寄存器、程序计数器和TCB（线程控制块）组成

**总结：**

- 线程不能看做独立应用，而进程可以看做独立应用；
- 进程有独立的地址空间，相互不影响，线程只是进程的不同的执行路径；
- 线程没有独立的地址空间，多进程的程序要比多线程的程序健壮；
- 进程的切换比线程的切换开销要大

### Java进程与线程的关系

- Java对操作系统提供的功能进行封装，包括进程和线程
- 运行一个程序，会产生一个进程，一个进程最少包含一个线程
- 每个进程对应一个JVM实例，多个线程共享JVM里的堆
- Java采用单线程编程模型，程序会自动创建主线程（一个Ui应用，最好把耗时的任务放入子线程，以免影响用户的体验）
- 主线程可以创建子线程，原则上要后于子线程完成执行

## 二、Thread中的start()和run()方法的区别

- 调用start() 方法，会创建一个新的子线程并启动
- run() 方法，只是Thread的一个普通方法的调用

代码比较：

```java
private static void attack() {
    System.out.println("Attack");
    System.out.println("Current Thead: " + Thread.currentThread().getName());
}

public static void main(String[] args) {
    Thread t = new Thread() {
        @Override
        public void run() {
            attack();
        }
    };

    System.out.println("Current Main Thread: " + Thread.currentThread().getName());
    t.start();  // Current Thread: Thread-0
    // t.run(); // Current Thread: main
}
```

## 三、Thread和Runnable是什么关系

- Thread 是实现了Runnable接口的类，使得run支持多线程
- 因类的单一继承原则，推荐使用Runnable接口，使普通类实现Runnable接口从而获得多线程的能力

## 四、如何实现处理线程的返回值

### 如何给run()方法传参

- 构造函数传参
- 成员变量传参
- 回调函数传参

### 如何实现处理线程的返回值

- 主线程等待法

  即，结果不对，一直主线程等待轮询，直到结果正确

- 使用Thread类的join()阻塞当前线程以等待子线程处理完毕

  可以更精准的控制，实现更简单，但是粒度不够细

- 通过Callable接口实现，通过FutureTask Or 线程池获取

  线程池的好处：可以多个Callable类提交，执行并发操作

```java
public class CycleWait implements Runnable {

    private String value;

    @Override
    public void run() {
        try {
            Thread.currentThread().sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        value = "we have data now";
    }

    public static void main(String[] args) throws InterruptedException {
        CycleWait cw = new CycleWait();
        Thread thread = new Thread(cw);
        thread.start();
        // 主线程等待法
        while (cw.value == null) {
            thread.sleep(100);
        }
//        thread.join(); // 阻塞当前线程
        System.out.println("value: " + cw.value);
    }
}
```

MyCallable.java

```java
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        String value = "test";
        System.out.println("Ready to work");
        Thread.currentThread().sleep(5000);
        System.out.println("task done");
        return value;
    }
}
```

FutureTaskDemo.java：FutureTask获取线程返回值

```java
public class FutureTaskDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<String>(new MyCallable());
        new Thread(futureTask).start();
        if (!futureTask.isDone()) {
            System.out.println("task has not finished, please wait");
        }
        System.out.println("task return: " + futureTask.get());
    }
}
```

线程池获取返回值：

```java
public class ThreadPoolDemo {

    public static void main(String[] args) {
        // 创建一个线程池
        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
        // 向线程池提交Callable
        Future<String> future = newCachedThreadPool.submit(new MyCallable());
        if (!future.isDone()) {
            System.out.println("task has not finished, please wait");
        }
        try {
            System.out.println(future.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            // 线程池最后要关闭
            newCachedThreadPool.shutdown();
        }
    }
}
```

## 五、线程的状态

六个状态：

- New（新建）：创建后尚未启动的线程的状态
- Runnable（运行）：线程start之后的状态，包含Running和Ready
  - Running：位于可运行线程池中，等待被线程调度选中，获取CPU的使用权
  - Ready：位于线程池中，等待被线程调度选中，获取CPU的使用权，获得CPU时间之后，就变成Running状态
- Waiting（无限等待）：不会被分配CPU执行时间，需要被其他线程显式唤醒
  - 没有设置TimeOut参数的Object.wait()方法
  - 没有设置TimeOut参数的Thread.join()方法
  - LockSupport.park()方法
- Timed Waiting（限期等待）：在一定时间后会由系统自动唤醒
  - Thread.sleep()方法
  - 设置了TimeOut参数的Object.wait()方法
  - 设置了TimeOut参数的Thread.join()方法
  - LockSupport.parkUntil()方法
  - LockSupport.parkNanos()方法
- Blocked（阻塞）：等待获取排它锁
- Terminated（结束）：已终止线程的状态，线程已经结束执行
  - 线程已终止了，不能再start，如果start，会抛出IllegalThreadStateException异常
  - 当线程的run方法结束时，或者main方法结束时，我们就认为它线程结束状态

## 六、sleep和wait的区别

### 基本的差别

- sleep是Thread类的方法，wait是Object类中定义的方法
- sleep在任何地方都可以使用
- wait只能在synchronized方法或者synchronized块中使用

### 最本质的差别

- Thread.sleep只会让出CPU，不会导致锁行为的改变
- Object.wait不仅会让出CPU，还会释放已经占有的同步资源锁

```java
final Object lock = new Object();
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("thread A is waiting to get lock");
        synchronized (lock) {
            try {
                System.out.println("thread A get lock");
                Thread.sleep(20);
                System.out.println("thread A do wait method");
                lock.wait(1000);
                System.out.println("thread A is done");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}).start();

// 让 main sleep 10毫秒
try {
    Thread.sleep(10);
} catch (InterruptedException e) {
    e.printStackTrace();
}

new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("thread B is waiting to get lock");
        synchronized (lock) {
            try {
                System.out.println("thread B get lock");
                System.out.println("thread B is sleeping 10 ms");
                Thread.sleep(10);
                System.out.println("thread B is done");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}).start();
```

结果：

```java
thread A is waiting to get lock
thread A get lock
thread B is waiting to get lock
thread A do wait method
thread B get lock
thread B is sleeping 10 ms
thread B is done
thread A is done
```

## 七、notify和notifyAll的区别

### 两个概念

- 锁池：EntryList

  假设线程A已经拥有了某个对象（不是类）的锁，而其他线程B、C想要调用这个对象的某个synchronized方法（或者块），由于B、C线程在进入对象的synchronized方法（或者块）之前必须先获取该对象锁的拥有权，而恰巧该对象的锁目前正被线程A所占用，此时B、C线程就会被阻塞，进入一个地方去等待锁的释放，这个地方便是该对象的锁池。

- 等待池：WaitList

  假设线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁，同时线程A就进入到了该对象的等待池中，进入到等待池中的线程不会去竞争该对象的锁，优先级高的线程就容易竞争到锁，之后该线程的synchronized块运行结束或者出现异常才会释放锁；没有竞争到锁的线程，会留到锁池中，并不会继续留在等待池中。锁池中的线程会继续竞争锁

- 锁池和等待池都是针对对象而言的

### notify和notifyAll的区别

- notifyAll会让所有处于等待池的线程全部进入锁池去竞争获取锁的机会
- notify只会随记选取一个处于等待池中的线程进入锁池去竞争获取锁的机会

```java
public class NotificationDemo {
    /**
     * volatile 多个线程对一个对象修改，线程A修改了对象的值，其他的线程都能立即看到改动
     */
    private volatile boolean go = false;

    public static void main(String[] args) throws InterruptedException {
        final NotificationDemo test = new NotificationDemo();

        Runnable waitTask = new Runnable() {
            @Override
            public void run() {
                try {
                    test.shouldGo();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " finished Execution");
            }
        };

        Runnable notifyTask = new Runnable() {
            @Override
            public void run() {
                test.go();
                System.out.println(Thread.currentThread().getName() + " finished Execution");
            }
        };

        Thread t1 = new Thread(waitTask, "WT1");
        Thread t2 = new Thread(waitTask, "WT2");
        Thread t3 = new Thread(waitTask, "WT3");
        Thread t4 = new Thread(notifyTask, "NT1");

        t1.start();
        t2.start();
        t3.start();

        Thread.sleep(200);

        t4.start();

    }

    private synchronized void shouldGo() throws InterruptedException {
        while (!go) {
            System.out.println(Thread.currentThread() + " is going to wait on this object");
            wait();
            System.out.println(Thread.currentThread() + " is woken up");
        }
        go = false;
    }

    private synchronized void go() {
        while (!go) {
            System.out.println(Thread.currentThread() + " is going to notify all or one thread " +
                    "waiting on this thread");
            go = true;
//            notify();
            notifyAll();
        }
    }
}
```

## 八、Yield函数

当调用Thread.yield()函数时，会给线程调度器一个当前线程愿意让出CPU使用的暗示，但是线程调度器可能会忽略这个暗示。

对锁无影响。

## 九、如何中断线程

- 已经被抛弃的方法：
  - stop()，会马上释放锁，可能影响数据未同步的问题
  - suspend() 和 resume()
- 目前使用的方法：
  - interrupt()，通知线程应该中断了
    1. 如果线程处于被阻塞状态，那么线程将立即退出被阻塞状态，并抛出一个InterruptdException异常
    2. 如果线程处于正常活动状态，那么会将该线程的中断标志设置为true，被设置中断标志的线程将继续正常运行，不受影响。
  - 需要被调用的线程配合中断
    1. 在正常运行任务时，经常检查本线程的中断标志位，如果被设置了中断标志 就自行停止线程
    2. 如果线程处于正常活动状态，那么会将该线程的中断标志设置为true，被设置中断标志的线程将继续正常运行，不受影响

```java
public static void main(String[] args) throws InterruptedException {
    Runnable interruptTask = new Runnable() {
        @Override
        public void run() {
            int i = 0;
            try {
                //在正常运行任务时，经常检查本线程的中断标志位，如果被设置了中断标志就自行停止线程
                while (!Thread.currentThread().isInterrupted()) {
                    Thread.sleep(100); // 休眠100ms
                    i++;
                    System.out.println(Thread.currentThread().getName() + " (" + Thread.currentThread().getState() + ") loop " + i);
                }
            } catch (InterruptedException e) {
                //在调用阻塞方法时正确处理InterruptedException异常。（例如，catch异常后就结束线程。）
                System.out.println(Thread.currentThread().getName() + " (" + Thread.currentThread().getState() + ") catch InterruptedException.");
            }
        }
    };
    Thread t1 = new Thread(interruptTask, "t1");
    System.out.println(t1.getName() +" ("+t1.getState()+") is new.");

    t1.start();                      // 启动“线程t1”
    System.out.println(t1.getName() +" ("+t1.getState()+") is started.");

    // 主线程休眠300ms，然后主线程给t1发“中断”指令。
    Thread.sleep(300);
    t1.interrupt();
    System.out.println(t1.getName() +" ("+t1.getState()+") is interrupted.");

    // 主线程休眠300ms，然后查看t1的状态。
    Thread.sleep(300);
    System.out.println(t1.getName() +" ("+t1.getState()+") is interrupted now.");
}
```

## 十、synchronized

### 线程安全问题的主要诱因

- 存在共享数据，也称临界资源
- 存在多条线程共同操作这些共享数据
- 解决问题的根本方法：同一时刻有且只有一个线程在操作共享数据，其他线程必须等到该线程处理完数据后再对共享数据进行操作

### 互斥锁的特性

- 互斥性：即在同一时间内，只允许一个线程持有某个对象锁，通过这种特性来实现多线程的协调机制，这样在同一时间只有一个线程对需要同步的代码块（复合操作）进行访问。互斥性也称为操作的原子性。
- 可见性：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁的同时，应获得最新共享变量的值），否则另一个线程可能是在本地缓存的某个副本上继续操作，从而引起不一致。
- synchronized，锁的不是代码，锁的是对象

### 根据获取的锁的分类：获取对象锁和获取类锁

#### 获取对象锁的两种方式：

- 同步代码块（`synchronized(this)`, `synchronized(类实例对象)`），锁是 `()` 中的实例
- 同步`非静态`方法（`synchronized method`）synchronized作修饰符，锁是当前对象的实例对象

#### 获取类锁的两种方式：

- 同步代码块（`synchronized(类.class)`），锁是`()`中的类对象（`Class对象`）
- 同步`静态`方法（`synchronized static method`），锁是当前对象的类对象（Class对象），类锁

### 对象锁和类锁的总结

1. 有线程访问对象的同步代码块时，另外的线程可以访问该对象的非同步代码块；
2. 若锁住的是用一个对象，一个线程在访问对象的同步代码块时，另一个访问对象的同步代码块的线程会被阻塞；
3. 若锁住的是同一个对象，一个线程在访问对象的同步方法时，另一个访问对象同步方法的线程会被阻塞；
4. 若锁住的是同一个对象，一个线程在访问对象的同步代码块时，另一个访问对象同步方法的线程会被阻塞，反之亦然；
5. 同一个类的不同对象的对象锁互不干扰；
6. 类锁由于也是一种特殊的对象锁，因此表现和上述1,2,3,4一致，而由于一个类只有一把对象锁，所以同一个类的不同对象使用类锁将会是同步的；
7. 类锁和对象锁互不干扰。

## 十一、synchronized底层实现原理

### 对象在内存中的布局：

- 对象头
- 实例数据
- 对齐填充

| 虚拟机位数 | 头对象结构             | 说明                                                         |
| ---------- | ---------------------- | ------------------------------------------------------------ |
| 32/64 bit  | Mark Word              | 默认存储对象的hashCode，分代年龄，锁类型，锁标志位等信息     |
| 32/64 bit  | Class Metadata Address | 类型指针指向对象的类元数据，JVM通过这个指针确定该对象是哪个类的数据 |

### Monitor锁：内部锁，每个Java对象天生自带了一把看不见的锁

### 什么是重入

从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入

### 自旋锁和自适应自旋锁

#### 自旋锁

- 许多情况下，共享数据的锁定状态持续时间较短，切换线程不值得；
- 通过让线程执行忙循环等待锁的释放，不让出CPU；
- 缺点：若锁被其他线程长时间占用，会带来许多性能上的开销

#### 自适应自旋锁（java6）

- 自旋的次数不再固定
- 由前一次在用一个锁上的自旋时间及锁的拥有者的状态来决定

### 锁消除，更彻底的优化

- JIT编译时，对运行上下文进行扫描，去除不可能存在竞争的锁

### 锁粗化，另外一种极端

- 通过扩大加锁的范围，避免频繁的加锁和解锁

### synchronized的四种状态

无锁，偏向锁，轻量级锁，重量级锁

锁膨胀方向：无锁 ---> 偏向锁 ---> 轻量级锁 ---> 重量级锁

- 无锁：没有加入任何锁

- 偏向锁：减少同一线程获取锁的代价，CAS（Compare And Swap）

  如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，即获取锁的过程只需要检查Mark Word的锁标记位为偏向锁以及当前线程Id等于Mark Word的ThreadId即可，这样就省去了大量有关锁申请的操作。

- 轻量级锁：

  轻量级锁是由偏向锁升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁

  适应：线程交替执行同步块

  若存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁

### 锁的内存语义

当线程释放锁时，Java内存模型会把该线程对应的本地内存中的共享变量刷新到主内存中；

而当线程获取锁时，Java内存模型会把该线程对应的本地内存置为无效，从而使得被监视器保护的临界区代码必须从主内存中读取变量

| 锁       | 优点                                                         | 缺点                                                         | 使用场景                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| 偏向锁   | 加锁和解锁不需要CAS操作，没有额外的性能消耗，和执行非同步方法相比仅存在纳秒级的差距 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗               | 只有一个线程访问同步块或者同步方法的场景         |
| 轻量级锁 | 竞争的线程不会阻塞，提高了效应速度                           | 若线程长时间抢不到锁，自旋会消耗CPU性能                      | 线程交替执行同步块或者同步方法的场景             |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU                              | 线程阻塞，响应时间缓慢，在多线程下，频繁的获取释放锁，会带来巨大的性能消耗 | 追求吞吐量，同步块或者同步方法执行时间较长的场景 |

