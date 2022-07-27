## 垃圾回收--后续的部分知识

> 往后访问不到的数据就是垃圾，需要回收
>
> 参考：https://www.bilibili.com/video/BV1D741177rV?p=2

---

#### 几种垃圾回收器的发展历史，与大致特点

> Serial、SerialOld、Parallel Scavenge、ParallelOld、ParNew和CMS组合、G1、ZGC、Shenandoah

下面1~5点的[参考链接](https://www.bilibili.com/video/BV1D741177rV?p=1&vd_source=b7e7d38107067bf58865928f03ef61a5)

1、首先GC的开始是新生代的GC回收(**Serial**)和老年代的GC(**SerialOld**)回收，串行的【要先停止用户程序，然后单线程进行垃圾回收】，这样会带来长时间的用户程序停止。

2、因为上面的是使用的单线程，因此又有人提出在原本单线程处换成多线程回收垃圾。【(**Parallel Scavenge**)、(**ParallelOld**)】

3、因为上面的1和2都要长时间停止用户程序，所以提出了CMS【使用的标记清除算法回收，所以不能回收新生代（新生代的回收应该是复制算法），只能回收老年代，通常要与新生代的垃圾回收器进行结合组合，又因为Parallel Scavenge无法和CMS结合（**<font color="orange">为什么无法结合？？</font>**），所以又提出一个**ParNew和CMS组合**】

> **<font color="orange">CMS无法与Parallel Scavenge结合的原因参考：</font>**https://blog.csdn.net/twt936457991/article/details/89736582
>
> 接口不兼容？Parallel Scavenge不是HotSpot的分代式GC框架，是由其他开发者提出来的，后来看起来效果不错，就直接将这个GC放到了HotSpot的VM里面了。所以HotSpot要维护两个功能几乎一样【Parallel Scavenge 和 ParNew】，只是具体细节不一样的GC回收器了

4、CMS不是JAVA的默认GC回收器，因为CMS仍然不够好【使用的标记清理，可能出现碎片化等问题】，后面又提出**G1**【G1从java9开始作为Java的默认回收器】

5、未来应该是**ZGC**和**Shenandoah**的发展

---

### G1   GC  --  最适合比较大的堆的垃圾回收

- 不是回收整个堆，而是选择与i个Collection Set（CS）进行回收  【理想是让每次回收以Region‘为单位回收，这样会让延迟比较低】
- 包含两种GC
  - Fully young GC
  - Mixed GC [包含一次Fully young GC]
- 估计每个Region中的垃圾比例，优先回收垃圾多的Rehion

#### 一些概念的理解-主要的一些问题

**跨代/跨Region引用问题**：会出现类似老年代中的对象引用了新生代中的对象，如果不考虑该新生代对象在老年代中的被引用，直接回收这个新生代的话，会出现不安全的情况。

基于以上跨Region引用的问题：提出Region内部使用Card进行构建，每个Region有一个Remember Set（RS）：记录有哪个Region中的card引用了当前的Region。【下图为Region1和Region3中的某个Card是引用的Region2，那么Region2中的RS就会记录Region1和Region3中Card的位置，那么当要回收Region2的时候，只要扫描Region1和Region3对应的引用的Card就可以了，避免整个堆的扫描】

![image-20220706224118537](C:\Users\10435\AppData\Roaming\Typora\typora-user-images\image-20220706224118537.png)

Card的标记变为dirty的时候：例子--原本新建一个对象刚开始的值是null，经过赋值引用了其他对象，就会使当前对象的card标记为dirty。比如一个对象使A a，在对对象B 赋值的时候，B有个子元素是A对象，会有b.a = a的赋值操作，这样就有了引用其他对象的操作。

一个card在所处的card table中，card叫做这个card table的一个entry

RS中的元素就是指向对应region中card table中的某个entry。并且可以找到具体的内存区域。【RS的底层数据结构可以理解为用C写的一个哈希表】

#### G1的一些流程

> Stop The World  ==>  STW

- **Fully young gc**

  是Stop The World的操作，会停止应用程序。与正常的新生代回收差不多，不过eden和survivor1复制到survivor2中的时候，整个survivor2是由多个分散的Region组成的。

- **Old GC**--先进行一次stop the world来进行和young gc？？？<font color="lightblue">【初始标记】</font>，然后并发进行三色标记算法<font color="lightblue">【并发标记】</font>，然后再来一次stop the world来进行**<font color="orange">remark</font>**<font color="lightblue">【最终标记】</font>，识别再三色标记时由于随着应用程序运行不断变化而**<font color="orange">导致出现的一些没有被标记上的活对象</font>**，避免错误清理,需要再停止一次应用程序，进行以此remark。最后再进行clean up/清理垃圾。<font color="lightblue">【筛选回收</font>：对每个 Region 的回收成本进行排序，根据用户期望的停顿时间来制定收回计划】

  并发进行，当堆用量达到一定程度时触发old gc

  **<font color="orange">三色标记算法</font>**： 刚开始所有region都是白色的，将gc root标为黑色，然后它里面引用的对象所在region标记为灰色，然后再把灰色标记为黑色，再查找原来是灰色的region所引用的对象标记为灰色，以此往下递进，直到最终找不到可往下递进的对象的时候【**可达性分析完成**】，玩车个标记，所有黑色的region都是活对象。

  只回收一个region中全都是垃圾的老年代区域，

  但是也会存在一个region中有一般是活的对象，一半是垃圾，为了避免产生碎片，那就再清理一半垃圾，之后再复制剩下的活对象到其他region？？【但是不是说只清理全都是垃圾的region吗？】---前面这些是在下面的mixed gc实现的，【优先选择垃圾比重所占最多的region进行清理，因为活对象越少，延迟stop the world/停止应用程序的延迟就越小。】，下面有个mixed gc

- **Mixed GC**  --  与Old GC有啥不同的？？？

  同时进行新生代和老年代的回收
