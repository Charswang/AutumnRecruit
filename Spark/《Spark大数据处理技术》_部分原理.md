**<font color="red">Akka Actor消息传递机制？？</font>**

**这本书里是有很多东西没有讲到的。。。**

Spark 有几个核心抽象: **<font color="orange">Datasets</font>**、**<font color="orange">DataFrames</font>**、**<font color="orange">SQL 表</font>**和**<font color="orange">弹性分布式数据集(RDDs)</font>**  

---

## Spark概述

虽然Spark通用引擎在一定程度上是用来**取代MapReduce**系统的，但是***Spark能够完美兼容Hadoop生态中的HDFS和YARN等其他组件***，使得现有的Hadoop用户能够非常容易地迁移到Spark系统中。

Spark利用DAG进行调度执行规划，在多任务计算和迭代计算中能够大量计算磁盘IO的时间

对每个任务启动一个线程，而不是进程。

### RDD

> 一种抽象的弹性数据集

RDD本质：并行计算的各个阶段进行有效的数据共享，这种共享就是RDD的本质

RDD可以进行：迭代算法的运行【机器学习】、关系型查询【数据库引擎特性】、MR批处理、流处理

---

## RDD以及编程接口

> 相比MapReduce，Spark不只局限于Map和Reduce两个操作

RDD的初始创建都是由SparkContext来负责的 -- >> 使用内存中的集合或者外部文件系统作为输入源。【另外一种生成方法就是由一个RDD转换而来】

执行程序中通常会将一个RDD转换成另一个RDD，【转换操作】

<font color="orange">RDD持久化</font>：将RDD存储到内存或者磁盘中，一以便重复使用。【eg：cache()i接口会将RDD缓存到内存中】

Spark是<font color="orange">惰性计算</font>的，只有执行Action操作，才会触发spark作业的执行。[**<font color="orange">Action操作</font>一般会将操作变为Scala集合/常量，或者输出到文件中/数据库中**]

通过记录作用在这些RDD上的转换操作，不需要将实际的RDD数据进行记录，每一个RDD在丢失或者操作失败之后，是可以重建的，如果再加上持久化，会很快恢复。

### 弹性体现在：

> ### 7、RDD的弹性表现在哪几点？（☆☆☆☆☆）
>
>   1）自动的进行内存和磁盘的存储切换；
>   2）基于Lineage(血缘)的高效容错；
>   3）task如果失败会自动进行特定次数的重试；
>   4）stage如果失败会自动进行特定次数的重试，而且只会计算失败的分片；
>   5）checkpoint和persist，数据计算之后持久化缓存；
>   6）数据调度弹性，DAG TASK调度和资源无关；
>   7）数据分片的高度弹性

### 宽窄依赖【箭头出发点为父RDD，箭头指向子RDD】

窄依赖（OneToOneDependency）：每一个父RDD的分区最多只被子RDD的一个分区所使用

宽依赖（ShuffleDependency）：多个子RDD的分区会依赖于同一个父RDD的分区；【没有经过co-partition的join操作】

> 在Spark中需要**明确地区分这两种依赖关系**有两个方面的**原因**：
>
> 第一，1、**窄依赖**可以在集群的一个节点上如流水线一般地执行，可以计算所有父RDD的分区，2、相反的，**宽依赖**需要取得父RDD的所有分区上的数据进行计算，将会执行类似于MapReduce一样的Shuffle操作。***【宽依赖计算较复杂】***
>
> 第二，1、对于**窄依赖**来说，节点计算失败后的恢复会更加有效，只需要重新计算对应的父RDD的分区，而且可以在其他的节点上并行地计算，2、相反的，在有**宽依赖**的继承关系中，一个节点的失败将会导致其父RDD的多个分区重新计算，这个代价是非常高的。***【宽依赖恢复代价高】***

### RDD 分区

> 两种分区函数(Partitioner)：**HashPartitioner（哈希分区）** & **RangePartitioner（区域分区）**。只有当RDD类型是（K,V）类型的时候才有Partitioner

有了RDD分区，是有可能减少Shuffle次数的。

## <font color="orange">API还是的重点搞一下， 但是目前还没仔细学习练习</font>

### <font color="orange">RDD的创建操作</font>【1、从集合创建；2、从存储文件/数据库系统中读取然后创建】

> 1、**parallelize**和**makeRDD**两类函数可以从集合中生成RDD。
>
> 2、textFile、hadoopFile、sequenceFile、objectFile、newAPIHadoopFile等等函数可以从文件中创建RDD；具体怎么使用，有哪些参数等用到的时候再查吧。

### <font color="orange">RDD的转换操作</font>

> map、distinct、flatMap
>
> repartition、coalesce --  【对RDD的分区进行重新划分】【两者不同之处在于是否shuffle，repartition是coalesce的shuffle为true的情况】--**假设RDD有N个分区，需要重新划分成M个分区**
>
> 

程序运行的并行度越高，shuffle数据写磁盘的消耗就越高？

### <font color="orange">RDD的持久化操作</font>

> 

### <font color="orange">RDD的行动操作</font>

>

---

## Spark运行模式及原理

### - 基本工作流程与运行模式

> 一些运行模式：Local、Local Cluster、Standalone模式、Mesos、YARN等
>
> 作业调度模块在各种运行模式下差不多，逻辑上是相同的。--DAGScheduler
>
> **这些运行模式的区别主要体现在任务调度模块**。不同的部署和运行模式，根据底层资源调度方式的不同，各自实现了自己特定的任务调度模块，用来将任务实际调度给对应的计算资源。
>
> ***<font color="orange">任务调度模块就是：Yarn，Mesos等这种资源管理和任务调度框架。</font>***

Spark应用程序由**一个Driver线程【驱动程序】**和**一组Executor**组成。

- YARN就是一个集群管理器来跟踪可用的资源，监控Executor资源吧。

- Drive驱动程序负责执行驱动程序的命令，在Executor执行器中完成给定任务。

#### - 基本工作流程

Spark应用程序都离不开**<font color="orange">SparkContext和Executor两部分</font>**，***Executor负责执行任务，运行Executor的机器称为Worker节点***，***SparkContext由用户程序启动，通过资源调度模块和Executor通信。***

![image-20220705175852494](C:\Users\10435\AppData\Roaming\Typora\typora-user-images\image-20220705175852494.png)

SparkContext和Executor这两部分的核心代码实现在各种运行模式中都是公用的，在它们之上，根据运行部署模式的不同，包装了不同调度模块以及相关的适配代码。

> 上面两段，说明了Local、Local Cluster、Standalone模式、Mesos、YARN等这些运行模式都是基于spark的一个核心运行基础上进行改造的。

- DAGScheduler -- 作业调度【负责将作业拆分成不同阶段的具有依赖关系的多批任务】

  根据shuffle划分stage。为每个stage构建一组具体任务。

- TaskScheduler -- 任务调度【负责每个具体任务的实际物理调度】

  然后以TaskSets的形式提交给任务调度模块来具体执行。

  TaskScheduler的实现主要用于**与DAGScheduler交互，负责任务的具体调度和运行**，其核心接口是submitTasks和cancelTasks。

- SchedulerBackend -- 任务调度

  SchedulerBackend的实现是**与底层资源调度系统交互（如Mesos/YARN）**，配合TaskScheduler实现具体任务执行所需的**资源分配**，核心接口是receiveOffers--这个接口用于任务资源调度请求。

- TaskSchedulerImpl

  TaskSchedulerImpl实现了TaskScheduler接口，提供了大多数本地和分布式运行调度模式的任务调度接口。

#### - 几种运行模式<font color="orange">要理解几种方式的运行方式和不同点？？？</font>

> Local、Local Cluster、Standalone模式、Mesos、YARN等

- Local模式

  Local本地模式使用**LocalBackend**配合TaskSchedulerImpl

- Standalone模式

  Spark Standalone模式中，Spark集群由Master节点和Worker节点构成，用户程序通过与Master节点交互，申请所需的资源，Worker节点负责具体Executor的启动运行

  Standalone模式使用**SparkDeploySchedulerBackend**配合TaskSchedulerImpl工作。SparkDeploySchedulerBackend本身拓展自**CoarseGrainedSchedulerBackend**：一个基于Akka Actor实现的粗粒度的资源调度类。CoarseGrainedSchedulerBackend会监听并持有注册给它的Executor资源，在接受Executor注册、状态更新、响应Scheduler请求等各种时刻，根据现有Executor资源发起任务调度流程。

- Local Cluster模式

  **伪分布模式**是基于Standalone模式来实现的，除了启动Master（主进程）和Worker（工作进程）的位置**与Standalone不同（全部在本地启动）**，在集群启动完毕后，后续应用程序的**工作流程及资源的调度流程与Standalone模式完全相同。**

- Mesos模式

  Mesos模式中，资源的调度可以分为**粗粒度调度**和**细粒度调度**两种

  Spark分别使用**CoarseMesosSchedulerBackend（粗粒度）**和**MesosSchedulerBackend**来配合TaskSchedulerImpl工作。

- Yran Standalone & YARN Cluster

- Yarn Client

---

## <font color="orange">Spark调度管理</font>

### Spark作业调度管理

1、利用**DAG有向无环图**来判断并识别多个作业任务之间的依赖关系(因果关系--有些任务必须先获得执行，然后相关的依赖任务才能执行)

2、DAGScheduler--DAG【Directed Acyclic Graph】有向无环图的调度类。

3、Spark调度中的概念

- **Task（任务）**：**单个分区**数据集上的**最小处理流程单元**。

- **TaskSet（任务集）**：由一组关联的，但互相之间没有Shuffle依赖关系的任务所组成的任务集。

- **Stage（调度阶段）**：一个任务集对应的调度阶段。

- **Job（作业）**：由**<font color="orange">一个RDD Action生成的</font>**一个或多个调度阶段所组成的一次计算作业。

- **Application（应用程序）**：Spark应用程序，由一个或多个作业组成。

![image-20220706152008000](C:\Users\10435\AppData\Roaming\Typora\typora-user-images\image-20220706152008000.png)

4、作业调度模块的**顶层逻辑**

DAGScheduler暴露两个入口：**submitJob**和**runJob**。**前者**返回一个Jobwaiter对象，可以用在异步调用中，用来判断作业完成或者取消作业，而**后者**则在内部调用submitJob，阻塞等待直到作业完成（或失败）==相当于runJob在submitJob外又包了一层。

**<font color="orange">DAGScheduler在SparkContext初始化的过程中被实例化，一个SparkContext对应创建一个DAGScheduler</font>**

DAGScheduler在初始化过程中会创建一个**DAGShedulerEventProcessActor实例来处理各种DAGSchedulerEvent事件**，这些事件包括作业的提交、任务状态的变化、监控，等等

DAGShedulerEventProcessActor的**Receive函数**是DAGScheduler中**处理这些事件的入口**，可以看到各种待处理的事件。

5、作业调度模块的具体工作流程

- 调度阶段的拆分：以ShuffleDependency为依据划分Stage

- 调度阶段的提交：**<font color="orange">直接触发作业</font>的RDD关联的调度阶段被称为<font color="orange">FinalStage</font>**。提交一个调度阶段的时候，先判断FinalStage其所有的父调度阶段的结果是否可用，如果可用则直接提交，否则，重复提交不可用的父调度阶段，如果没有提交成功，则放入等待队列，等待以后提交。【这里的等待应该是：当前父调度阶段的前面的某个阶段还没有完成，所以就需要等待，前面的阶段都完成之后当前调度阶段才可以再次提交。

  这就引出一个问题：**<font color="orange">什么时候等待列表中的调度阶段会被重新提交呢？</font>**

  > DAGScheduler会检查对应的调度阶段是否还有任何依赖的调度阶段没有完成，如果没有，说明该调度阶段处于就绪状态，就可以再次尝试提交该调度阶段了

- 任务集的提交：DAGScheduler通过TaskScheduler接口提交任务集。这个任务集最终会触发**TaskScheduler**构建一个**TaskSetManager**的实例来管理这个任务集的生命周期。至此，DAGScheduler的提交阶段就完成了，剩下的就交给TaskScheduler和TaskSetManager来获取计算资源，并将具体任务发送到对应的Executor节点上了。

- 状态的监控：监控当前作业调度甚至具体到任务的完成情况。主要通过对TaskScheduler暴露一些**回调函数**实现。通过回调函数返回的信息，根据任务的生命周期维护作业和调度阶段的状态信息。【回调函数甚至可以返回Executor的生命状态】

- 任务结果的获取：一个具体的任务执行完成之后，需要将结果返回给DAGScheduler。、

  FinalStage对应的任务叫做**<font color="orange">ResultTask</font>**。**返回给DAGScheduler的是一运算结果本身**

  中间调度阶段对应的任务**<font color="orange">ShuffleMapTask</font>**，**返回给DAGScheduler的是一个MapStatus对象**

  > MapStatus对象管理了ShuffleMapTask的运算输出结果在BlockManager里的相关存储信息，而非结果本身，这些存储位置信息将作为下一个调度阶段的任务获取输入数据的依据。

  根据不同的任务结果的大小，可以按照大小分为两类，分别按照不同的方式进行返回。【具体看书本Spark大数据处理技术的4.4.5，这里不赘述】

6、调度池和调度模式：每个SparkContext可能会同时出现多个没有关系的任务集【多个任务集之间互不相关】。**这些任务集之间如何调度呢：**由**调度池**来决定，调度池分为两种类型：（1）FIFO：先进先出调度；（2）FAIR：公平调度；

​	  **<font color="orange">这里说的调度池只是在单个Spark程序内部的调度逻辑，而非Spark应用之间的调度关系</font>**

7、Spark应用(等价于SparkkContext)之间的调度关系：SparkContext之间的调度关系，按照Spark不同的运行模式，就不一定是归Spark管理的了。在Mesos和YARN模式下，底层资源调度系统的调度策略由Mesos和YARN所决定。

8、调度过程中的数据本地性问题：TaskSetManager在适配任务时，会根据资源的情况和任务的数据来源，尽量选择最佳的Locality进行匹配。

> Spark的应用程序由于**首先需要申请资源运行Executor**，**而后再调度任务**，所以，如果第一步在申请资源运行Executor时没有任何数据本地性的信息，那么除非应用程序申请的计算资源足够多，从而保证了在每个节点上都有Executor，否则，在第二步调度任务时，**是可能无法做到数据的本地处理的**（因为数据所在的节点上并没有运行Executor）。
>
> 在**YARN相关模式**下，是可以在为Spark应用创建资源申请时就额外**传递数据文件的位置信息**的，其他模式却暂时没有实现相关功能。【申请资源的时候，同时传递数据文件的位置信息，以此能够更好的将申请的Executor位于数据文件所在的节点】

---

## <font color="orange">Spark的存储管理</font>

> **BlockManager**操作类是在存储管理模块中的。通过BlockManager对存储管理模块进行操作。

RDD的存取是以数据块为单位的，本质上分区（partition）和数据块（block）是等价的，

在Spark中，分区和数据块是一一对应的，一个RDD中的一个分区对应着存储管理模块中的一个数据块【感觉不对，一个分区有可能很大，但是一个block是固定大小的呀】

RDD的ID号与RDD中每个分区的索引号作为数据块的名称来与每个RDD上的每个分区**<font color="orange">建立映射关系</font>**

### RDD持久化

#### - 内存缓存：

- 内部维护了一个一个以数据块名称为键，块内容为值的哈希表：

- 默认情况下，当所占用的内存大于60%的时候，会采取一些策略：将一些数据块丢弃或者存入磁盘【丢弃还是放入磁盘取决于这些数据块的持久化选项，若持久化选项中包含了磁盘缓存，则会将这些块移入磁盘进行缓存，反之则直接删除。】

### - 磁盘缓存：

- 在磁盘缓存中，一个数据块对应着文件系统中的一个文件，**文件名和块名称的映射关系**是通过**哈希算法**计算所得的

### - 持久化选项：

> - persist()
>
> - cache()
>
> 持久化选项：MEMORY_ONLY 、  MEMORY_AND_DISK、  MEMORY_ONLY_SER、   MEMORY_AND_DISK_SER、  DISK_ONLY、  MEMORY_ONLY_2 、  MEMORY_AND_DISK_2

### - 选择不同持久化选项的建议

> Spark为我们提供了不同的持久化选项，这意味着在CPU利用率和内存使用量上提供了不同的选择方案，用户可以根据程序的情况和机器的配置选择合理的持久化方式，使得在增加CPU利用率的同时兼顾内存的使用量。以下是对于持久化方式选择的几条建议：
>
> • 如果使用默认的持久化方式（MEMORY_ONLY）完全能够缓存RDD，那么就无须选择其他的持久化方式，因为这是CPU利用率最高的一种方式，能够确保程序跑得尽可能快。
>
> • 如果默认的持久化方式无法完全缓存所需的RDD，可以尝试使用MEMORY_ONLY_SER这种持久化方式，并选用更快速度的序列化方式，这能够更高效地使用内存，同时确保程序跑得相对较快。
>
> • 除非对于RDD的重算所带来的开销比缓存到磁盘来得更大，一般情况下不要选择将RDD缓存到磁盘上。**<font color="lightblue">通常情况下，对于某一分区的重算会比从磁盘中读取要快。</font>**
>
> • 如果想要确保快速的错误恢复机制，应尽可能地选用带有备份的持久化方式。虽然前面也说到所有的持久化方式都能通过重算丢失数据从错误中恢复，但是备份可以使得任务继续进行而无须等待丢失分区的重新计算。
>
> 同时，Spark的持久化选项也提供了一些接口供用户扩展，用户可以通过StorageLevel的apply()函数来实现。具体的实现方式可以参看源码。
>
> 当然，最适合用户的持久化方式需要用户结合自身应用的特点以及机器的硬件性能做出权衡，用户可以通过实验和测试选择自身所需的持久化方式。

### Shuffle持久化

> Shuffle数据块必须是在**磁盘上进行缓存**的，而不能选择在内存中缓存；
>
> 两种存储方式：1、像rdd那样映射成文件；2、映射成一个文件的一部分，多个数据块组成一个文件，这样避免文件很多造成磁盘/文件系统的性能影响。【如果按照1来映射文件的话，1000个Map任务和1000个Reduce任务，会产生1000 000个Shuffle文件】

- Shuffle是将一组任务的输出结果重新组合作为下一组任务的输入这样的一个过程

- **Shuffle数据的读取和传输**有两种方式，一种是**基于NIO以socket连接**去获取数据，另一种是**基于NIO通过Netty服务端**获取数据。前者是默认的获取方式，通过配置spark.shuffle.use.netty为true，可以启用第二种获取方式。

### 广播变量持久化

- 广播变量：能把本地的数据广播到所有的执行节点上，从而加速程序的运行。

- 广播变量也是由存储管理模块进行管理。

- 广播变量数据块是以MEMORY_AND_DISK的持久化方式存入本节点的存储管理模块中的。

---

## 监控--<font color="orange">后面用到再说吧</font>

### 实时UI监控

- 端口号：4040；如果在一台机器上有多个SparkContext启动，那么监听的端口顺延，如4041、4042，等等。
- ...

### 历史UI监控

> 历史UI管理是指当Spark程序运行结束后，用户仍然可以利用UI查询程序的运行状态。

- ....

---

## Shark架构 -- Hive on spark【hql】

....可能用不到，后面再说

> ### 7.9.3 Shark查询比Hive慢
>
> 通常来说，相同条件下，Shark查询不会比Hive慢，但有时确实会发生Shark较慢的情况，可能的原因如下（不限于）：
>
> • 内存设置不当，或者数据有倾斜，导致Shark不可用或者大量任务失败或者长时间的Full GC。
>
> • Hive在物理执行上有很多的优化，但是Shark并没有完全移植过来，比如，在写数据表时，合并数据表的小数据块文件（可以极大减少将来读取该数据表MapTask的数量），根据输入文件大小动态改变mapred.reduce.tasks等。
>
> • SQL语句中调用的UDF直接或者间接用到了内存锁。Hive执行是完全多进程的，这些锁并不起作用，而Shark执行是基于多线程的，有些内存锁就会造成性能急剧下降。
>
> Shark查询运行时的问题定位和性能优化，本身比较复杂，原因也多种多样，很难一言概之，需要广大读者朋友们在使用中积累和总结，在社区中积极参与分享讨论。

---

## Spark SQL --- 先有的Shark再有的Spark SQL





#### Shark与SparkSQL的区别



## Spark Streaming部分

**StreamingContext对象**与Spark初始需要创建**SparkContext对象**一样，使用Spark Streaming就需要创建StreamingContext对象。

```java
StreamingContext：new StreamingContext(master, appName, batchDuration, [sparkHome], [jars]);
// master参数所设置的是标准的Spark URL
// appName参数来为我们的应用程序命名
// 程序部署在集群上时，我们可以指定sparkHome和jars这两个参数，具体的使用方法可以参考SparkContext的初始化
// batchDuration[批处理间隔]--Spark Streaming以多少时间间隔为单位来提交任务逻辑
```

### 时间和窗口

**1、批处理间隔（batch duration）：数据处理和作业提交的最小单位**

将一个批处理时间间隔收集到的数据汇总起来作为一批数据让系统来处理

**2、 滑动间隔（slide duration）和窗口间隔（window duration）**

滑动间隔和窗口间隔的设置必须是批处理间隔的***整数倍***。**因为在Spark Streaming内部，数据处理和作业提交的最小单位是批处理间隔，因此不同的间隔设置必须为批处理间隔的整数倍。则窗口间隔的计算将在两个批次之间结束。**

**3、批处理间隔和滑动间隔、窗口间隔之间的关系**

假定我们设置的滑动间隔是批处理间隔的2倍【滑动间隔，是窗口操作的一个重要参数，它指的是经过多长时间窗口滑动一次形成新的窗口】，窗口间隔是批处理间隔的4倍，那么现在处理数据的间隔就变为了原先设定的2倍，而每次处理的数据量则有窗口间隔决定，为之前的4倍。

- 窗口操作会使得部分数据重复被计算，spark做了优化，会记录重复计算的结果。

- 为什么要有滑动窗口：

  以TCP引入滑动窗口为例：TCP 引入滑动窗口的最直接的原因是**“接收方的缓存是有限的”**<font color="orange">发送方不能假设接收方缓存无限大，一直发包，造成接收方丢包。</font>[如果按照批处理间隔那样到了指定时间就发送数据，很有可能会出现数据丢失的情况。]

### DStream【离散数据流】--  最终还是被映射到RDD的操作上，是建立在RDD的基础上的。

DStream是通过**一组时间序列上连续的RDD**来表示的，每一个RDD都包含了特定时间间隔内的数据流，**每个RDD里面是一个批次间隔的数据吗？**

![image-20220709152820716](C:\Users\10435\AppData\Roaming\Typora\typora-user-images\image-20220709152820716.png)

DStream也是可以像RDD那样通过map操作转换成另一个DStream的。【DStream的转换操作，最终会被映射到内部基于RDD的操作，操作结束后我们将得到一个新的DStream，我们可以在此DStream上继续进行操作。】

在DStream中有三种类型的操作，一种是**普通的转换操作**，一种是**基于窗口的变换操作**，还有一种是**输出操作**

**基于窗口的变换操作？？**

> 窗口的变换操作的理解：一些**转换操作与窗口操作合并**后所得到的一系列基于窗口的转换操作
>
> **<font color="orange">窗口操作是什么，有哪些操作？？</font>**

### DStream持久化

与RDD相似的是，DStream也可以将其内容持久化在内存中  --  persist()

对于一些基于窗口的操作，例如reduceByWindow或者基于状态的操作：updateStateByKey，会在其内部**隐式地将DStream中的RDD持久化在内存中**而无须用户显式调用。

**对于输入源的数据，它们会自动地持久化在内存中以供后续的操作使用，同时会将数据复制到不同的节点上以保证容错。**



Spark Streaming内维护了周期性的定时器定时器超时间隔就是预先设置的批处理间隔，每当定时器超时后【在每个批处理间隔的时间点到来时】，就会请求DStreamGraph产生Streaming作业，放进JobScheduler中进行调度并提交。

Spark Streaming程序-->DStreamGraph->JobScheduler(维持有线程池，默认大小为1)->转换成Spark作业提交给DAGScheduler->TaskScheduler

**<font color="orange">Spark Streaming作业会被转换成Spark作业</font>并提交到DAGScheduler中分解和执行，从本质和表现结果来看，最终还是RDD之间的转换操作**



#### Streaming作业与Spark作业之间的关系

> Spark Streaming作业最终会被翻译成Spark作业并提交和执行

当我们的程序**在构建DStream操作链时，在Spark Streaming内部会隐式地构建RDD操作链**。当完整的DStream操作链构建完成后，与之相应的RDD操作链也相应地被构建完成，这个RDD操作链最终会在其内部产生作业并提交运行，-- 这背后的一切都依赖于**DStream是建立在RDD基础之上的**这一设计理念。





---

### Checkpoint与持久化的区别：【https://blog.51cto.com/u_14693305/4765789】

1、Persist 和 Cache 只能保存在本地的磁盘和内存中(或者堆外内存)；
     Checkpoint 可以保存数据到 HDFS 这类可靠的存储上

2、Cache和Persist的RDD会在程序结束后会被清除或者手动调用unpersist方法；
     Checkpoint的RDD在程序结束后依然存在，不会被删除；

3、Persist和Cache，不会丢掉RDD间的依赖链/依赖关系，因为这种缓存是不可靠的，如果出现了一些错误(例如 Executor 宕机)，需要通过回溯依赖链重新计算出来；
     Checkpoint会斩断依赖链，因为Checkpoint会把结果保存在HDFS这类存储中，更加的安全可靠，一般不需要回溯依赖链；

![image-20220707205209573](C:\Users\10435\AppData\Roaming\Typora\typora-user-images\image-20220707205209573.png)

---

