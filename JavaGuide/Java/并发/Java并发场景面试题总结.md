# 线程
## 什么是线程和进程
### 进程
进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

在 Java 中，当我们启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称主线程。

在 Windows 中通过查看任务管理器的方式，我们就可清楚看到 Windows 当前运行的进程。

### 线程
线程与进程相似，但是线程是一个比进程更小的执行单位。一个进程在其执行过程中可以产生多个线程。与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但是每个线程都有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间做切换工作时，负担要比进程小得多，也正因为如此，线程也被称为**轻量级进程**。

Java 程序天生就是多线程程序，可通过 JMX 来看一个普通的 Java 程序有哪些线程，代码如下：
```java
public class MultiThread {
  public static void main(String[] args) {
    // 获取 Java 线程管理 MXBean
    ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
    // 不需要获取同步的 monitor 和 synchronizer 信息，仅获取线程和线程堆栈信息
    ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
    // 遍历线程信息，仅打印线程 ID 和线程名称信息
    for (ThreadInfo threadInfo : threadInfos) {
      System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
    }
  }
}
```
上述程序输出如下（输出内容可能不同，不用太纠结下面每个线程的作用，只用知道 main 线程执行 main 方法即可）：
```plain
[5] Attach Listener //添加事件
[4] Signal Dispatcher // 分发处理给 JVM 信号的线程
[3] Finalizer //调用对象 finalize 方法的线程
[2] Reference Handler //清除 reference 线程
[1] main //main 线程,程序入口
```
从上面输出内容可见：**一个 Java 程序的运行是 main 线程和多个其他线程同时运行的。**

## Java 线程和操作系统线程有什么区别
JDK 1.2 前 Java 线程基于绿色线程，是用户线程。绿色线程比原生线程多一些限制，如绿色线程不能直接使用操作系统提供的功能如异步 I/O、只能在一个内核线程上运行无法利用多核等。
JDK 1.2 后 Java 线程改为基于原生线程（Native Threads）实现，JVM 直接使用操作系统原生内核级线程（内核线程）来实现 Java 线程，由操作系统内核进行线程调度与管理。

- 用户线程：由用户空间程序管理和调度的线程，运行在用户空间，专门给应用程序使用。
- 内核线程：由操作系统内核管理和调度的线程，运行在内核空间，只有内核程序可以访问。
用户线程创建和切换成本低，但不可以利用多核。内核态线程，创建和切换成本高，但可以利用多核。

**现在的 Java 线程的本质其实就是操作系统的线程。**

线程模型是用户线程和内核线程之间的关联方式，常见的线程模型有这三种：
1. 一对一（一个用户线程对应一个内核线程）
2. 多对一（多个用户线程映射到一个内核线程）
3. 多对多（多个用户线程映射到多个内核线程）
![[Pasted image 20240626174253.png]]

Windows、Linux 等主流操作系统中，Java 线程采用 “一对一” 线程模型，一个 Java 线程对应一个系统内核线程。Solaris 系统是特例，它使用 “多对多” 线程模型。

虚拟线程在 JDK 21 顺利转正，关于虚拟线程、平台线程（也就是我们上面提到的 Java 线程）和内核线程三者的关系可以阅读我写的这篇文章：[Java 20 新特性概览](https://javaguide.cn/java/new-features/java20.html)。

## 线程与进程的关系、区别及优缺点
![[Pasted image 20240626175348.png]]

一个进程可以有多个线程，多个线程共享进程的 **堆** 和 **方法区** （JDK 1.8 之后的元空间）资源，但是每个线程有自己的 **程序计数器**、**虚拟机栈** 和 **本地方法栈**。

总结：**线程是进程划分成的更小运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会互相影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反。**

> 扩展：为什么 **程序计数器、虚拟机栈和本地方法栈** 是线程私有的呢？为什么 **堆和方法区** 是线程共享的呢？

程序计数器主要有下面两个作用：
1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理等。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

需要注意的是：如果执行的是 Native 方法，那么程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址。

所以，程序计数器私有主要是为了 **线程切换后能恢复到正确的执行位置。**

虚拟机栈和本地方法栈私有原因：
- 虚拟机栈：每个 Java 方法在执行之前会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。
- 本地方法栈：和虚拟机栈所发挥的作用非常相似，区别是 **虚拟机栈为虚拟机执行 Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。**

所以，为了 **保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的。

**堆和方法区** 是所有线程共享的资源，其中堆是进程中最大的一块内存，注意用于存放新创建的对象（几乎所有对象都在这里分配内存），方法区主要用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

## 如何创建线程
创建线程方式：
- 继承 `Thread` 类
- 实现 `Runnable` 接口
- 实现 `Callable` 接口
- 使用线程池
- 使用 `CompletableFeture` 类
- …………

不过，这些方式其实并没有真正创建出线程。准确来说，这些属于是 Java 代码中使用多线程的方法。

严格来说，Java 只有一种方式可以创建线程，`new Thread().start()` 创建。不管哪种方式，最终还是依赖于 `new Thread().start()`。

关于这个问题的详细分析可以查看这篇文章：[大家都说 Java 有三种创建线程的方式！并发编程中的惊天骗局！](https://mp.weixin.qq.com/s/NspUsyhEmKnJ-4OprRFp9g)。

## 线程的生命周期和状态
Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态：
- NEW：初始状态，线程被创建出来但没有被调用 `start()`。
- RUNNABLE：运行状态，线程被调用了 `start()` 等待运行的状态。
- BLOCKED：阻塞状态，需要等待锁释放。
- WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）。
- TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。
- TERMINATED：终止状态，表示该线程已经运行完毕。
线程在生命周期中并不是固定处于某一个状态，而是随着代码的执行在不同状态之间切换。

Java 线程状态变迁图：
![[Pasted image 20240626182030.png]]
由上图可以看出：

- 线程创建之后它将处于 NEW 状态，调用 `start()` 方法后开始运行，线程这时处于 READY 状态。可运行状态的线程获得了 CPU 时间片后就处于 RUNNING 状态，

>在操作系统层面，线程有 READY 和 RUNNING 状态；而在 JVM 层面，只能看到 RUNNABLE 状态（图源：[HowToDoInJava](https://howtodoinJava.com/ "HowToDoInJava")：[Java Thread Life Cycle and Thread States](https://howtodoinJava.com/Java/multi-threading/Java-thread-life-cycle-and-thread-states/ "Java Thread Life Cycle and Thread States")），所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。
**为什么 JVM 没有区分这两种状态呢？** （摘自：[Java 线程运行怎么有第六种状态？ - Dawell 的回答](https://www.zhihu.com/question/56494969/answer/154053599) ） 现在的时分（time-sharing）多任务（multi-task）操作系统架构通常都是用所谓的“时间分片（time quantum or time slice）”方式进行抢占式（preemptive）轮转调度（round-robin 式）。这个时间分片通常是很小的，一个线程一次最多只能在 CPU 上运行比如 10-20ms 的时间（此时处于 running 状态），也即大概只有 0.01 秒这一量级，时间片用后就要被切换下来放入调度队列的末尾等待再次调度。（也即回到 ready 状态）。线程切换的如此之快，区分这两种状态就没什么意义了。

![[Pasted image 20240626182444.png]]

- 当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）** 状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态。
- **TIMED_WAITING(超时等待)** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将线程置于 TIMED_WAITING 状态。当超时时间结束后，线程将会返回到 RUNNABLE 状态。
- 当线程进入 `synchronized` 方法/块或者调用 `wait` 后（被 `notify`）重新进入 `synchronized` 方法/块，但是锁被其它线程占有，这个时候线程就会进入 **BLOCKED（阻塞）** 状态。
- 线程在执行完了 `run()`方法之后将会进入到 **TERMINATED（终止）** 状态。

相关阅读：[线程的几种状态你真的了解么？](https://mp.weixin.qq.com/s/R5MrTsWvk9McFSQ7bS0W2w) 。

## 什么是线程上下文切换
线程在执行过程中会有自己的运行条件和状态（也称上下文），比如上文所说到过的程序计数器、栈信息等。当出现如下情况时，线程会从占用 CPU 状态中退出。
- 主动让出 CPU，比如调用了 `sleep()` ，`wait()` 等。
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用 CPU 导致其他线程或者进程饿死。
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。
- 被终止或者结束运行。

这其中前三种都会发生线程切换，线程切换意味着需要保存当前线程上下文，留待线程下次占用 CPU 的时候恢复现场。并加载下一个将要占用 CPU 的线程上下文。这就是 **上下文切换**。

上下文切换是现代操作系统的基本功能，因为每次需要保存信息恢复信息，这将会占用 CPU、内存等系统资源进行处理，也就意味着效率会有一定损耗，如果频繁切换会造成整体效率低下。

## `Thread#sleep()` 方法和 `Object#wait()` 方法对比
**共同点：** 两者都可以暂停线程的执行。

**区别：**
- `sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁。
- `wait()` 通常被用于线程间交互/通信，`sleep()` 通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()` 或者 `notifyAll()` 方法。`sleep()` 方法执行完成后，线程会自动苏醒，或者也可以使用 `wait(long timeout)` 超时后线程会自动苏醒。
- `sleep()` 是 `Thread` 类的静态本地方法，`wait()` 是 `Object` 类的本地方法。

## 为什么 wait () 方法不定义在 Thread 中
`wait()` 是让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁。每个对象 `Object` 都拥有对象锁，既然要释放当前线程占有的对象锁并让其进入 WAITING 状态，自然要操作对应的对象 `Object`，而非当前线程 `Thread`。

而 `sleep()` 是让当前线程暂停执行，不涉及到对象类，因此也不需要获得对象锁。

## 可以直接调用 Thread 类的 run 方法吗？
New 一个 `Thread`，线程进入了新建状态（new）。调用 `start()` 方法，会启动一个线程并使线程进入就绪状态，当分配到时间片后就开始运行了。`start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。但是，直接执行 `run()` 方法，会把 `run()` 方法当中一个 main 线程下的普通方法区执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结：调用 `start()` 方法可以启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**

# 多线程
## 并发和并行的区别
- 并发：两个及两个以上的作业在同一 **时间段** 内执行。
- 并行：两个及两个以上的作业在同一 **时刻** 执行。

关键在于是否 **同时** 执行。

## 同步和异步的区别
- 同步：发出一个调用之后，在没有得到结果之前，该调用就不可以返回，一直等待。
- 异步：调用在发出之后，不用等待返回结果，该调用直接返回。

## 为什么要使用多线程
总体上：
- 从计算机底层来说：线程可以比作是轻量级的进程，是程序执行的最小单位，线程之间的切换和调度成本远小于进程。另外，多核 CPU 时代意味着多个线程可以同时运行，减少了线程上下文切换的开销。
- 从当代互联网发展趋势来说：现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

从计算机底层来探讨：
- 单核时代：在单核时代多线程注意为了提高单进程利用 CPU 和 IO 系统的效率。假设只运行了一个 Java 进程的情况，当我们请求 IO 的时候，如果 Java 进程中只有一个线程，此线程被 IO 阻塞则整个进程被阻塞。CPU 和 IO 设备只有一个在运行，那么可以简单地说系统整体效率只有 50%。当使用多线程的时候，一个线程被 IO 阻塞，其他线程还可以继续使用 CPU。从而提高了 Java 进程利用系统资源的整体效率。
- 多核时代：多核时代多线程主要是为了提高进程利用多核 CPU 的能力。举个例子：假如我们要计算一个复杂的任务，我们只用一个线程的话，不论系统有几个 CPU 核心，都只会有一个 CPU 核心被利用到。而创建多个线程，这些线程可以被映射到底层多个 CPU 核心上执行，在任务中的多个线程没有资源竞争的情况下，任务执行的效率会有显著性的提高，约等于（单核时执行时间/CPU 核心数）。

## 使用多线程可能带来什么问题
并发编程的目的就是为了提高程序的执行效率，进而提高程序的运行速度。但是并发编程并不总是能提高程序运行速度的，而且并发编程可能会遇到很多问题，比如：内存泄露、死锁、线程不安全等。

## 如何理解线程安全和不安全
线程安全和不安全是在多线程环境下对于同一份数据的访问是否能够保证其正确性和一致性的描述。
- 线程安全指的是在多线程环境下，对于同一份数据，不管有多少个线程同时访问，都能保证这份数据的正确性和一致性。
- 线程不安全则表示在多线程环境下，对于同一份数据，多个线程同时访问时可能会导致数据混乱、错误或者丢失。

## 单核 CPU 上运行多个线程效率一定会高吗
单核 CPU 同时运行多个线程的效率是否会高，取决于线程的类型和任务的性质。

一般来说，有两种类型的线程：CPU 密集型和 IO 密集型。CPU 密集型线程主要进行计算和逻辑处理，需要占用大量的 CPU 资源。IO 密集型线程主要进行输入输出操作，如读写文件、网络通信等，需要等待 IO 设备的响应，而不占用太多的 CPU 资源。

在单核 CPU 上，同一时刻只能有一个线程在运行，其他线程需要等待 CPU 的时间片分配。如果线程是 CPU 密集型的，那么多个线程同时运行会导致频繁的线程切换，增加了系统的开销，降低了效率。而 IO 密集型线程，多个线程同时运行可以利用 CPU 在等待 IO 时的空闲时间，提高了效率。

因此，对于单核 CPU 来说，CPU 密集型任务，开较多线程会影响效率；IO 密集型任务，开较多线程会提高效率。较多线程需要适度，不能超过系统课承受上限。

# 死锁
## 什么是线程死锁
多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。

如下图，A 持有资源 2，B 持有资源 1，它们同时想申请对方的资源，所以这两个线程就会互相等待而进入死锁状态。
![[Pasted image 20240627002304.png]]
下面通过一个例子来说明线程死锁,代码模拟了上图的死锁的情况：
```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```
Output：
```plain
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```
线程 A 通过 `synchronized(resource1)` 获得 `resource1` 的监视器锁，然后通过 `Thread.sleep(1000)` 让线程 A 休眠 1s 为的是让线程 B 得到执行，然后获取到 `resource2` 的监视器锁。

线程 A 和线程 B 休眠结束后都开始请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，也就产生了死锁。

上面的例子符合产生死锁的四个必要条件：
1. 互斥条件：该资源任意一个时刻只由一个线程占用。
2. 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。

## 如何检测死锁
- 使用 `jmap`、`jstack` 登命令查看 JVM 线程栈和堆内存的情况。如果有死锁，`jstack` 的输出中通常会有 `Found one Java-level deadlock` 的字样，后面会跟着死锁相关的线程信息。另外，实际项目中还可以搭配使用 `top`、`df`、`free` 等 Linux 命令查看操作系统的基本情况，出现死锁可能会导致 CPU、内存等资源消耗过高。
- 采用 VisualVM、JConsole 等工具进行排查。

这里以 JConsole 工具为例进行演示。找到 JDK 的 bin 目录，找到 jconsole 并双击打开。连接上对于 Java 进程，然后到线程子 tab，就能看到【检测死锁】的按钮，进行死锁检测。
![[Pasted image 20240627004243.png]]
结果：
![[Pasted image 20240627004336.png]]

## 如何预防和避免线程死锁
**预防死锁**，破坏死锁产生的必要条件即可：
1. **破坏请求与保持条件：** 一次性申请所有的资源。
2. **破坏不剥夺条件：** 占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
3. **破坏循环等待条件：** 靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

**避免死锁**

避免死锁就是在资源分配时，借助于算法对资源分配进行计算评估，使其进入安全状态。

> **安全状态** 指的是系统能够按照某种线程推进顺序（P1、P2、P3……Pn）来为每个线程分配所需资源，直到满足每个线程对资源的最大需求，时每个线程都可以顺利完成。称 `<P1、P2、P3……Pn>` 序列为安全序列。

对上述代码进行修改，线程 2 的代码修改成下面这样就不会产生死锁了。
```java
new Thread(() -> {
    synchronized (resource1) {
        System.out.println(Thread.currentThread() + "get resource1");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread() + "waiting get resource2");
        synchronized (resource2) {
            System.out.println(Thread.currentThread() + "get resource2");
        }
    }
}, "线程 2").start();

```
输出：
```plain
Thread[线程 1,5,main]get resource1
Thread[线程 1,5,main]waiting get resource2
Thread[线程 1,5,main]get resource2
Thread[线程 2,5,main]get resource1
Thread[线程 2,5,main]waiting get resource2
Thread[线程 2,5,main]get resource2

Process finished with exit code 0
```
上面的代码为什么避免了死锁的发生?

线程 1 首先获得到 resource1 的监视器锁,这时候线程 2 就获取不到了。然后线程 1 再去获取 resource2 的监视器锁，可以获取到。然后线程 1 释放了对 resource1、resource2 的监视器锁的占用，线程 2 获取到就可以执行了。这样就破坏了破坏循环等待条件，因此避免了死锁。

# JMM（Java 内存模型）
JMM（Java 内存模型）相关的问题比较多，也比较重要，单独抽了一篇文章来总结 JMM 相关的知识点和问题：[[JMM（Java内存模型）详解]]。

# Volatile 关键字
## 如何保证变量的可见性
在 Java 中，`volatile` 关键字可以保证变量的可见性，如果我们将变量声明为 `volatile`，这就指示 JVM 这个变量是共享且不稳定的，每次使用它都到主存中进行读取。
![[Pasted image 20240627005452.png]]
![[Pasted image 20240627005457.png]]
JMM(Java 内存模型)强制在主存中进行读取

`volatile` 关键字其实并非是 Java 语言特有的，在 C 中也有，最原始的意义就是禁用 CPU 缓存。如果将一个变量使用 `volatile` 修饰，这就指示编译器，这个变量共享且不稳定，每次使用它都到主存中进行读取。

`volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。

## 如何禁止指令重排序
**在 Java 中，`volatile` 关键字除了可以保证变量的可见性，还有一个重要的作用就是防止 JVM 的指令重排序。** 如果我们将变量声明为 `volatile`，在对这个变量进行读写操作时，会通过插入特点的 **内存屏障** 的方式来禁止指令重排序。

在 Java 中，`Unsafe` 类提供了三个开箱即用的内存屏障相关的方法，屏蔽了操作系统底层的差异：
```java
public native void loadFence();
public native void storeFence();
public native void fullFence();
```

理论上来说，通过这三个方法也可以实现和 `volatile` 禁止重排序一样的效果，只是会麻烦一些。

以一个常见的面试题为例讲解一下 `volatile` 关键字禁止指令重排序的效果。

面试中面试官经常会说：“单例模式了解吗？来给我手写一下！给我解释一下双重检验锁方式实现单例模式的原理呗！”

**双重校验锁实现对象单例（线程安全）**：
```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```
`uniqueInstance` 采用 `volatile` 关键字修饰也是很有必要的， `uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：
1. 为 `uniqueInstance` 分配内存空间
2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。

## `volatile` 可以保证原子性吗？
**`volatile` 关键字能保证变量的可见性，但不能保证对变量的操作是原子性的。**

通过下面的代码即可证明：
```java
public class VolatileAtomicityDemo {
    public volatile static int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        VolatileAtomicityDemo volatileAtomicityDemo = new VolatileAtomicityDemo();
        for (int i = 0; i < 5; i++) {
            threadPool.execute(() -> {
                for (int j = 0; j < 500; j++) {
                    volatileAtomicityDemo.increase();
                }
            });
        }
        // 等待1.5秒，保证上面程序执行完成
        Thread.sleep(1500);
        System.out.println(inc);
        threadPool.shutdown();
    }
}
```
正常情况下，运行上面的代码理应输出 `2500`。但你真正运行了上面的代码之后，你会发现每次输出结果都小于 `2500`。

为什么会出现这种情况呢？不是说好了，`volatile` 可以保证变量的可见性嘛！

也就是说，如果 `volatile` 能保证 `inc++` 操作的原子性的话。每个线程中对 `inc` 变量自增完之后，其他线程可以立即看到修改后的值。5 个线程分别进行了 500 次操作，那么最终 inc 的值应该是 5 \* 500=2500。

很多人会误认为自增操作 `inc++` 是原子性的，实际上，`inc++` 其实是一个复合操作，包括三步：
1. 读取 inc 的值。
2. 对 inc 加 1。
3. 将 inc 的值写回内存。

`volatile` 是无法保证这三个操作是具有原子性的，有可能导致下面这种情况出现：
1. 线程 1 对 `inc` 进行读取操作之后，还未对其进行修改。线程 2 又读取了 `inc`的值并对其进行修改（+1），再将`inc` 的值写回内存。
2. 线程 2 操作完毕后，线程 1 对 `inc`的值进行修改（+1），再将`inc` 的值写回内存。

这也就导致两个线程分别对 `inc` 进行了一次自增操作后，`inc` 实际上只增加了 1。

其实，如果想要保证上面的代码运行正确也非常简单，利用 `synchronized`、`Lock`或者`AtomicInteger`都可以。

1）使用 `synchronized` 改进：
```java
public synchronized void increase() {
    inc++;
}
```
2）使用 `AtomicInteger` 改进：
```java
public AtomicInteger inc = new AtomicInteger();

public void increase() {
    inc.getAndIncrement();
}
```
3）使用 `ReentrantLock` 改进：
```java
Lock lock = new ReentrantLock();
public void increase() {
    lock.lock();
    try {
        inc++;
    } finally {
        lock.unlock();
    }
}
```