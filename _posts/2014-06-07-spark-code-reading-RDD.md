---
title: "Spark代码阅读(5) Resilient Distributed Dataset"
categories:
  - Code Reading
tags:
  - Spark
---

RDD是Spark中的核心概念。Spark中的高性能、高可靠等等性质全部依赖于RDD概念。

{% include toc %}

## RDD

> A Resilient Distributed Dataset (RDD), the basic abstraction in Spark. Represents an immutable, partitioned collection of elements that can be operated on in parallel.
> 
> Internally, each RDD is characterized by five main properties:
> 
>- A list of partitions
>- A function for computing each split
>- A list of dependencies on other RDDs
>- Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
>- Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)
> 
> All of the **scheduling** and **execution** in Spark is done based on these methods, allowing each RDD to implement its own way of computing itself.

## RDD的设计初衷
先抛开RDD是什么、RDD里面包含了什么这些问题，我们先来理解一下RDD的设计初衷。

Map Reduce类的运算框架的逻辑，说白了就是如何高可扩展、高容错的利用大量的机器来做并行化的处理。所以在这些框架中，很大的工作量是如何来保证容错性、如何来保证并发性和扩展性。比如Hadoop，它就通过将运算结果存储到HDFS中，通过在HDFS中保存多份结果来保证系统的容错性。至于并发性，则主要是通过如何做好任务调度来完成。但是，Hadoop这样框架的一个问题是性能不高，系统相对较慢，这里有一篇关于[《传统的MapReduce框架慢在那里》](http://jerryshao.me/architecture/2013/04/15/%E4%BC%A0%E7%BB%9F%E7%9A%84MapReduce%E6%A1%86%E6%9E%B6%E6%85%A2%E5%9C%A8%E5%93%AA%E9%87%8C/)的文章，可以看看，很重要的一个原因是MapReduce框架中，`map`以及`reduce`的操作输入和输出通常是HDFS，所以每个过程都会涉及到分布式文件系统的读写过程，这些框架的运算性能会受到分布式文件系统性能的影响。

RDD的设计初衷则是，考虑如何通过避免或减少对分布式文件系统的依赖，通过内存计算来提高运算性能。所以Spark一个很关键的地方就是如何利用内存。但是利用内存进行并行计算和通过分布式文件系统作为交换数据的中介不同，内存并行计算存在两个问题：

 1. 如何并行得管理内存中的数据集以便进行分布式计算
 2. 如何保证内存计算的可靠性

### RDD的Partition
Spark以RDD来管理分区数据，这个粒度是很大的。这么大的数据通常很难在一台机器上处理，这里便涉及到了如何对数据进行分区。所以在RDD里面需要有如何对数据进行分区的分区方法，如`partitioner`，以及`partitions`，除此之外，还需要有一个用来计算每个partition（即RDD源代码中的`split`）的函数`compute`。另外，为了考虑locality，RDD还提供了计算的优先位置选择的接口`getPreferredLocations`方法。

### RDD的可靠性设计
可靠性主要考虑的是数据没了，系统怎么把数据给恢复出来。而内存作为一种易失型的存储介质，一旦断电或者什么的，里面的数据就没了。所以如何保证内存中数据的可靠性呢？在RDD中，作者引入了`Lineage`的概念。什么是`Lineage`呢？字面上是*血统、家系、世袭*的意思。所以，就比较好理解了，也就是说RDD是怎么得来的一个记录，就如同[麦兜是麦子仲肥的第十八代传人一样](http://movie.douban.com/review/6588749/)，你爸爸的爸爸的爸爸的爸爸的爸爸的爸爸的爸爸的爸爸，生了你爸爸的爸爸的爸爸的爸爸的爸爸的爸爸的爸爸，生了....，最后你爸爸生了你一样。（纸包鸡什么的最好吃了。）虽然你爸爸的爸爸的爸爸的爸爸的爸爸的爸爸的爸爸的爸爸早就已经不在了，但是只要有这张家族图谱，你就可以知道你是怎么来的。RDD的`Lineage`也就是这个意思，RDD需要有一种方式，在数据不存在的时候，如何把它恢复出来。但是呢，只有这张图谱还不行，麦兜还得知道他所有妈妈们（因为麦兜有那么多爸爸吗！^_^），所以在RDD里面就有咱们上面提到的`dependencies`的概念存在，也就是说`RDD`要记住谁生了它，每个RDD也就是麦兜的每个祖先都得知道他们各自的爸爸妈妈（dependencies）是谁。这还不够，虽然知道爸爸妈妈是谁了，但是同一对爸爸妈妈生出来的孩子不一定一样啊，比如你和你哥就是不一样的，所以咱们还得知道怎么生的，这就涉及到基因决定形态的算法了，所以在RDD里需要有个`compute`函数来计算它是怎么从parents转换而来的。

Spark面向的是大数据，数据有那么多，要以多大的粒度来记住这么多的数据的继承关系呢？如果太细，比如以数据块来记录，那么通常需要以大量的内存在记录这些转换关系，这本身将会是很大的开销。所以，Spark的策略是以较大的粒度来记录这些关系，这个粒度就是RDD。大数据处理者们，Spark只能帮咱们到这里了。所以Spark不太适合处理一些细粒度的需求。比如需要对数据进行大量不一样的操作。

RDD的checkpoint机制。checkpoint可以把数据存放到hdfs中，这就是所谓的数据落地，虽然这违背了Spark的计算原则，但在这种数据量面前我们还是需要屈服，最完美得是，Spark知道checkpoint这个东西存在，不用我们特别指定位置，Spark在计算的时候会按需使用[ref](http://blackniuza.blogspot.com/2013/10/spark_27.html)。另外，[这里](http://www.cnblogs.com/fxjwind/p/3514203.html)有篇关于checkpoint的代码分析，不过还是挺粗的，等分析到RDD真正执行过程的时候，再来分析。

``` scala
  /**
   * Mark this RDD for checkpointing. It will be saved to a file inside the checkpoint
   * directory set with SparkContext.setCheckpointDir() and all references to its parent
   * RDDs will be removed. This function must be called before any job has been
   * executed on this RDD. It is strongly recommended that this RDD is persisted in
   * memory, otherwise saving it on a file will require recomputation.
   */
  def checkpoint() {
    if (context.checkpointDir.isEmpty) {
      throw new Exception("Checkpoint directory has not been set in the SparkContext")
    } else if (checkpointData.isEmpty) {
      checkpointData = Some(new RDDCheckpointData(this))
      checkpointData.get.markForCheckpoint()
    }
  }

  /**
   * This class contains all the information related to RDD checkpointing. Each instance of this
   * class is associated with a RDD. It manages process of checkpointing of the associated RDD,
   * as well as, manages the post-checkpoint state by providing the updated partitions,
   * iterator and preferred locations of the checkpointed RDD.
   */
  private[spark] class RDDCheckpointData[T: ClassTag](@transient rdd: RDD[T])
    extends Logging with Serializable {
    // The checkpoint state of the associated RDD.
    var cpState = Initialized

    // The file to which the associated RDD has been checkpointed to
    @transient var cpFile: Option[String] = None

    // The CheckpointRDD created from the checkpoint file, that is, the new parent the associated RDD.
    var cpRDD: Option[RDD[T]] = None

    ...
  }

  // Mark the RDD for checkpointing
  def markForCheckpoint() {
    RDDCheckpointData.synchronized {
      if (cpState == Initialized) cpState = MarkedForCheckpoint
    }
  }

```

RDD的persist操作只是让`SparkContext`在`persistentRdds`里进行记录，将会在RDD第一次被计算的时候才会进行persist到相应的Storage Level中。

``` scala
  /**
   * Set this RDD's storage level to persist its values across operations after the first time
   * it is computed. This can only be used to assign a new storage level if the RDD does not
   * have a storage level set yet..
   */
  def persist(newLevel: StorageLevel): RDD[T] = {
    // TODO: Handle changes of StorageLevel
    if (storageLevel != StorageLevel.NONE && newLevel != storageLevel) {
      throw new UnsupportedOperationException(
        "Cannot change storage level of an RDD after it was already assigned a level")
    }
    sc.persistRDD(this)
    // Register the RDD with the ContextCleaner for automatic GC-based cleanup
    sc.cleaner.foreach(_.registerRDDForCleanup(this))
    storageLevel = newLevel
    this
  }

  /**
   * Register an RDD to be persisted in memory and/or disk storage
   */
  private[spark] def persistRDD(rdd: RDD[_]) {
    persistentRdds(rdd.id) = rdd
  }
```

## RDD.scala的代码结构
RDD相关的代码主要分布在`package org.apache.spark.rdd`中，包括RDD基类`RDD.scala`; 一些列的派生RDD类，如`MappedRDD` `FilteredRDD`等等; 某些RDD类特有的方法，如`DoubleRDDFunctions` `OrderedRDDFunctions` `PairRDDFunctions` `SequenceFileRDDFunctions`; 以及一些特殊的RDD Actions (`AsyncRDDActions.scala`)和用于Checkpoint的(`RDDCheckpointData.scala`)类。

`package org.apache.spark.rdd`的组织方式大致就是按照以上所描述的结构来划分的，rdd包的结构简化图如下：
![package org.apache.spark.rdd]({{site.url}}{{site.baseurl}}/assets/images/spark/RDD.jpg)

## Transformation and Action
> RDDs support two types of operations: transformations, which create a new dataset from an existing one, and actions, which return a value to the driver program after running a computation on the dataset. For example, map is a transformation that passes each dataset element through a function and returns a new distributed dataset representing the results. On the other hand, reduce is an action that aggregates all the elements of the dataset using some function and returns the final result to the driver program (although there is also a parallel reduceByKey that returns a distributed dataset).

从上面RDD的代码结构来看，RDD中除了上面提到的DeveloperApi（`getPartitions`, `getDependencies`, `partitioner`, `getPreferredLocations`, `compute`）以及`persist`、`checkpoint`等操作以外，剩下的最为重要的两块便是`Transformation`以及`Action`类操作了。

### Transformation
> All transformations in Spark are lazy, in that they do not compute their results right away. Instead, they just remember the transformations applied to some base dataset (e.g. a file). The transformations are only computed when an action requires a result to be returned to the driver program. This design enables Spark to run more efficiently – for example, we can realize that a dataset created through map will be used in a reduce and return only the result of the reduce to the driver, rather than the larger mapped dataset.

RDD的Transformation操作主要就是将一种RDD变换成另外一种RDD，比如将`HadoopRDD`变换到`MappedRDD`，这种变换操作只会记录两个(组)RDD之间的相互关系，计算方法等。也就是在这种操作之后，这种操作中所涉及到的计算是不会马上执行的，所以也就有了咱们在运行Word Count例子时，进行transformation操作时没有相应输出。

Spark中RDD有许多种，咱们在前面的rdd包的结构图中已经列出了目前(0.9.1)core中大部分的RDD类型了，通过查看每个RDD的实现，会发现实现一个新的RDD的主要工作就是重载RDD基类中给出的那些DeveloperApi（`getPartitions`, `getDependencies`, `partitioner`, `getPreferredLocations`, `comp    ute`）。在这些RDD实现过程中，极可能要再次实现的还有其对应的Partition类。具体的看代码吧。相对来说实现一个新的RDD还是比较简单的。

### Actions
Spark的Action操作，如`reduce`、`fold`等操作，才是真正触发Spark集群开始干活的直接原因。纵观RDD中所有的action操作，他们都会直接或者间接的调用`sc.runJob(...)`函数，如：

``` scala
  /**
   * Aggregate the elements of each partition, and then the results for all the partitions, using a
   * given associative function and a neutral "zero value". The function op(t1, t2) is allowed to
   * modify t1 and return it as its result value to avoid object allocation; however, it should not
   * modify t2.
   */
  def fold(zeroValue: T)(op: (T, T) => T): T = {
    // Clone the zero value since we will also be serializing it as part of tasks
    var jobResult = Utils.clone(zeroValue, sc.env.closureSerializer.newInstance())
    val cleanOp = sc.clean(op)
    val foldPartition = (iter: Iterator[T]) => iter.fold(zeroValue)(cleanOp)
    val mergeResult = (index: Int, taskResult: T) => jobResult = op(jobResult, taskResult)
    sc.runJob(this, foldPartition, mergeResult)
    jobResult
  }
```

所以，Action就开始了Spark的真正工作了。

从`sc.runJob`开始，我们一路跟踪代码，最终这些action操作会去调用

``` scala
  /**
   * Run a function on a given set of partitions in an RDD and pass the results to the given
   * handler function. This is the main entry point for all actions in Spark. The allowLocal
   * flag specifies whether the scheduler can run the computation on the driver rather than
   * shipping it out to the cluster, for short actions like first().
   */
  def runJob[T, U: ClassTag](
      rdd: RDD[T],
      func: (TaskContext, Iterator[T]) => U,
      partitions: Seq[Int],
      allowLocal: Boolean,
      resultHandler: (Int, U) => Unit) {
    partitions.foreach{ p =>
      require(p >= 0 && p < rdd.partitions.size, s"Invalid partition requested: $p")
    }
    val callSite = getCallSite
    val cleanedFunc = clean(func)
    logInfo("Starting job: " + callSite)
    val start = System.nanoTime
    dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, allowLocal,
      resultHandler, localProperties.get)
    logInfo("Job finished: " + callSite + ", took " + (System.nanoTime - start) / 1e9 + " s")
    rdd.doCheckpoint()
  }
```

所以真正开始执行action操作的，是由DAGScheduler的runJob函数来完成的。而`dagScheduler.runJob`又会进一步去调用它的`submitJob`函数并等待任务完成返回：

``` scala

  /**
   * Submit a job to the job scheduler and get a JobWaiter object back. The JobWaiter object
   * can be used to block until the the job finishes executing or can be used to cancel the job.
   */
  def submitJob[T, U](
      rdd: RDD[T],
      func: (TaskContext, Iterator[T]) => U,
      partitions: Seq[Int],
      callSite: String,
      allowLocal: Boolean,
      resultHandler: (Int, U) => Unit,
      properties: Properties = null): JobWaiter[U] =
  {
    // Check to make sure we are not launching a task on a partition that does not exist.
    val maxPartitions = rdd.partitions.length
    partitions.find(p => p >= maxPartitions).foreach { p =>
      throw new IllegalArgumentException(
        "Attempting to access a non-existent partition: " + p + ". " +
          "Total number of partitions: " + maxPartitions)
    }

    val jobId = nextJobId.getAndIncrement()
    if (partitions.size == 0) {
      return new JobWaiter[U](this, jobId, 0, resultHandler)
    }

    assert(partitions.size > 0)
    val func2 = func.asInstanceOf[(TaskContext, Iterator[_]) => _]
    val waiter = new JobWaiter(this, jobId, partitions.size, resultHandler)
    eventProcessActor ! JobSubmitted(
      jobId, rdd, func2, partitions.toArray, allowLocal, callSite, waiter, properties)
    waiter
  }
```

由此可见，RDD的action操作会去调用`SparkContext`的`runJob`，并最终由`DAGScheduler`的`submitJob`操作来完成，而`submitJob`操作则主要为这个action设定先设定一个`jobID`，并`eventProcessActor`发起一个`JobSubmitted`事件，从而触发`eventProcessActor`去处理这个事件，从而为RDD完成这一次action操作。至于DAGScheduler中的`eventProcessActor`是如何处理这个`JobSubmitted`事件的则要看它的`private[scheduler] def processEvent(event: DAGSchedulerEvent): Boolean = {}`函数了。咱们等分析DAGScheduler的时候再来讨论。不过值得注意的是，在`processEvent`中，`JobSubmitted`这个事件中，是通过提交所谓的Stage来完成的`submitStage(finalStage)`。所以接下来咱们在讨论完RDD后，先来了解以下Stage是什么个东西。

