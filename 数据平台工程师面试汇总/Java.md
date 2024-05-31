视频网址：[https://www.bilibili.com/video/BV1is421N7Cg/?spm_id_from=333.337.search-card.all.click&vd_source=d48f4dc9d6d078002e7de6c900378be5](https://www.bilibili.com/video/BV1is421N7Cg/?spm_id_from=333.337.search-card.all.click&vd_source=d48f4dc9d6d078002e7de6c900378be5)

P 1 创建线程的方法：

3 或者 4 种

1、继承 Thread 类

重写 run () 方法，而不是 start () 方法，占用了继承名额，Java 中的类是单继承的。（接口可以多继承）

2、实现 Runnable 接口

实现 Runnable 接口，实现 run () 方法，使用时依然要用到 Thread，该方法更常用

3、实现 Callable 接口

实现 Callable 接口，实现 call () 方法，使用时需要 Thread+FutureTask 配合，这种方式支持获取到异步执行任务的结果

4、利用线程池来创建线程

实现 Callable 接口或者 Runnable 接口都行，然后使用 ExecutorService 来创建线程

底层都是基于 Runnable 来实现的

P 2 为什么不建议使用 Executors 来创建线程池：

P 3：线程池有哪几种状态？每种状态分别表示什么？

P 4：Sychronized 和 ReentrantLock 区别

P 5：ThreadLocal 有哪些应用场景？底层如何实现的？

P 6：公平锁和非公平锁

P 7：Sychronized 的锁升级过程是怎样的

P 8：tomcat 为什么要自定义类加载器

P 9：【JVM】类的生命周期

P 10：【JVM】JVM 类加载机制

P 11：【JVM】JVM 内存区域

P 12：【JVM】对象创建过程

P 13：【JVM】对象内存分配方式

P 14：【JVM】JVM 创建对象是如何解决多线程内存抢占的

P 15：【JVM】对象的内存布局与大小计算

P 16：【JVM】JVM 内存泄漏的原因

P 17：【JVM】如何判断对象仍然存活

P 18：【JVM】垃圾回收算法

P 19：【JVM】三色标记法是什么？有什么优点？

P 20：【JVM】CMS 收集器的垃圾收集过程

P 21：【JVM】G 1 收集器解释、垃圾收集过程

