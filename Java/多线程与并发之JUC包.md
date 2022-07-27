***<font color="gree">JUC包里面的锁、线程池、工具类、类、集合没看懂</font>***

### CAS的三个问题  --  属于非阻塞同步的一种？因为通常会有重试机制，利用自旋[不放弃cpu/不阻塞]方式。

1、ABA问题

2、自旋时间/循环时间长，开销大

3、只能保证一个共享变量的原子操作

---

### LockSupport

> park()  &  unpark()

- park()/parkNanos()/parkUntil() 阻塞进程  【调用unpark、中断或者设置的阻塞时间到的时候就释放进程了】【oarkNanos可以设置阻塞时间。parkUntil可以在指定的时限前禁用当前线程。】
- unpark() 释放进程，该函数是不安全的，调用前需要确定线程是否仍然存活

<font color="orange">**注意park与wait的区别；**</font>【https://pdai.tech/md/java/thread/java-thread-x-lock-LockSupport.html】

- object.wait()与LockSupport.park的区别：

  > hello，想获取下《Spark权威指南中文版》的PDF

- **Thread.sleep()、Object.wait、Conditation.await()、LockSupport.park()之间的关系与区别**--<font color="red">重点</font>

  > 

**park/unpark要比wait/notify更灵活**

在park之前执行unpark，没啥影响，线程不会阻塞，会跳过park那里的代码

在wait前执行notify，会导致，后续没有notify，导致线程一直处于挂起状态。

---

### AQS

> 构建锁和同步器的框架
>
> 主要是：构建一套阻塞等待以及唤醒时锁的分配的机制？

简单的来说：

- AQS其实就是一个可以给我们实现锁的**框架**
- 内部实现的关键是：**先进先出的队列、state 状态**
- 定义了内部类 ConditionObject
- 拥有两种线程模式
- - 独占模式
  - 共享模式
- 在 LOCK 包中的相关锁（常用的有 ReentrantLock、 ReadWriteLock ）都是基于 AQS 来构建
- 一般我们叫 AQS 为同步器。

<font color="red">内部原理，看不懂。。。。。还有其底层数据结构。。。双向链表，每个节点是一个线程。</font>

---

### <font color="orange">ThreadLocal</font>

> 实现线程隔离



**ThreadLocal造成的内存泄漏问题**

使用线程池来操作ThreadLocal的时候，是有可能会造成内存泄漏问题的。因为线程池中那些没有被销毁的线程中，有些ThreadLocal并没有释放。为了避免 内存泄露的情况，ThreadLocal提供了一个清楚线程中对象的方法。ThreadLocalMap的remove()方法。**<font color="lightblue">【详细的看pdai.tech】</font>**

---

