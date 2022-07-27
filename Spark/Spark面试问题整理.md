https://zhuanlan.zhihu.com/p/433682238

https://blog.csdn.net/weixin_45727359/article/details/122098250

---

## MapReduceShuffle与SparkShuffle

[参考1、MapReduce Shuffle 和 Spark Shuffle 原理概述](https://www.cnblogs.com/xiaodf/p/10650921.html)

### MapReduce Shuffle

> https://blog.csdn.net/ASN_forever/article/details/81233547

### Spark Shuffle

> https://zhuanlan.zhihu.com/p/67061627
>
> [参考2、Spark中的Spark Shuffle详解](https://www.cnblogs.com/xueqiuqiu/articles/12979864.html)

- **HashShuffle**

  HashShuffleManager的运行机制主要分成两种，一种是**普通运行机制**，另一种是**合并的运行机制--【利用File Consolidation 制将一个Executor中所有MapTask相同的分区文件合并，与普通运行机制相比减少了文件数量】**。

  ***普通运行机制***生成的文件的数量：mapTask数量*reducer个数

  ***合并的运行机制***生成的文件数量：CPU核数*reducer个数【可以假设一个Executor由一个CPU Core】

  > **Hash shuffle普通机制的问题**
  >
  > 1).Shuffle前在磁盘上会产生海量的小文件，建立通信和拉取数据的次数变多,此时会产生大量耗时低效的 IO 操作 (因為产生过多的小文件)
  >
  > 2).可能导致OOM，大量耗时低效的 IO 操作 ，导致写磁盘时的对象过多，读磁盘时候的对象也过多，这些对象存储在堆内存中，会导致堆内存不足，相应会导致频繁的GC，GC会导致OOM。由于**内存中需要保存海量文件操作句柄和临时信息**，如果数据处理的规模比较庞大的话，内存不可承受，会出现 **OOM 等问题**。

- Spark2.0之后的**SortShuffle**--参考MapReduce的Shuffle

  SortShuffleManager的运行机制主要分成两种，一种是**普通运行机制**，另一种是**bypass运行机制**。【当shuffle read task的数量小于等于spark.shuffle.sort.bypassMergeThreshold参数的值时(默认为200)  /  或者不是聚合类的shuffle算子(比如reduceByKey)，就会启用bypass机制。】

  

由HashShuffle改为SortShuffle的原因主要是因为HashShuffle会产生大量的中间磁盘文件，大量的磁盘IO会影响操作性能。而SortShuffle会利用排序将相同的分区的临时文件合并成一个文件。

**SortShuffleManager相较于HashShuffleManager来说**，有了一定的改进。主要就在于，每个Task在进行shuffle操作时，虽然也会产生较多的临时磁盘文件，但是最后会将所有的临时文件合并(merge)成一个磁盘文件，因此**每个Task就只有一个磁盘文件**。在下一个stage的shuffle read task拉取自己的数据时，只要根据索引读取每个磁盘文件中的部分数据即可。

=======================================================================================================

#### shuffle过程中排序的作用是什么？

> https://www.cnblogs.com/xiaodf/p/10650921.html

- 提前将相同的key放到一起，排序之后相同的key放到一块可以方便做合并操作。
- 如果没有排序那么合并的时候需要全盘扫描，但是排序后一旦发现有不同的key的时候，说明当前key已经查找完。--以便更好的合并？
- 在map阶段的排序和合并【排序之后合并可以快一点？？】可以缓解reduce阶段的排序的内存消耗，map阶段可以溢出写道磁盘中。
- **使用sort的好处是能够*降低内存使用量***
- **但是有时候排序并不是必要的，甚至会带来不必要的消耗，因为排序是消耗资源的**

---

**一个mapTask是一个分区的数据进行map的操作？**

参照MR中mapTask和reduceTask的理解：Spark中每个RDD中的一个分区块可当作一个mapTask？下一个stage开始时的RDD有多少个分区作为有多少个reduce？

---

