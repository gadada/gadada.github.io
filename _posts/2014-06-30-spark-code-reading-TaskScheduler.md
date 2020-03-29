---
title: "Spark代码阅读(9) Task Scheduler"
categories:
  - Code Reading
tags:
  - Spark
---

[FIXME: revision is on the road]

关于Spark的调度，官方的[Job Scheduling](http://spark.apache.org/docs/latest/job-scheduling.html)给了非常详细的说明，可以先读那部分的文档。OSChina上有这篇文档的[翻译版本](http://www.oschina.net/translate/spark-job-scheduling)，如果不想看英文的可以考虑读翻译版本，当然啦，推荐官方文档。

在[Stage](./6.Stage.md)的分析中，我们了解了Task是由`DAGScheduler`创建Stage之后，通过[`submitMissingTasks`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L745)函数来生成Task的。生成的Task通过[`taskScheduler.submitTasks`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L789)来进行提交。所以接下来就是`TaskScheduler`如何调度Task的问题了。


# Trait 
[`TaskScheduler`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/TaskScheduler.scala)只是一个`trait`，它定义了一系列的接口，具体实现在目前的版本中，是由[`TaskSchedulerImpl`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/TaskSchedulerImpl.scala)完成的。`TaskScheduler`的具体接口包含如下：

``` scala
/**
 * Low-level task scheduler interface, currently implemented exclusively by TaskSchedulerImpl.
 * This interface allows plugging in different task schedulers. Each TaskScheduler schedulers tasks
 * for a single SparkContext. These schedulers get sets of tasks submitted to them from the
 * DAGScheduler for each stage, and are responsible for sending the tasks to the cluster, running
 * them, retrying if there are failures, and mitigating stragglers. They return events to the
 * DAGScheduler.
 */
private[spark] trait TaskScheduler {

  def rootPool: Pool

  def schedulingMode: SchedulingMode

  def start(): Unit

  // Invoked after system has successfully initialized (typically in spark context).
  // Yarn uses this to bootstrap allocation of resources based on preferred locations,
  // wait for slave registerations, etc.
  def postStartHook() { }

  // Disconnect from the cluster.
  def stop(): Unit

  // Submit a sequence of tasks to run.
  def submitTasks(taskSet: TaskSet): Unit

  // Cancel a stage.
  def cancelTasks(stageId: Int, interruptThread: Boolean)

  // Set the DAG scheduler for upcalls. This is guaranteed to be set before submitTasks is called.
  def setDAGScheduler(dagScheduler: DAGScheduler): Unit

  // Get the default level of parallelism to use in the cluster, as a hint for sizing jobs.
  def defaultParallelism(): Int
}
```

# TaskSchedulerImpl

``` scala
/**
 * Schedules tasks for multiple types of clusters by acting through a SchedulerBackend.
 * It can also work with a local setup by using a LocalBackend and setting isLocal to true.
 * It handles common logic, like determining a scheduling order across jobs, waking up to launch
 * speculative tasks, etc.
 *
 * Clients should first call initialize() and start(), then submit task sets through the
 * runTasks method.
 *
 * THREADING: SchedulerBackends and task-submitting clients can call this class from multiple
 * threads, so it needs locks in public API methods to maintain its state. In addition, some
 * SchedulerBackends synchronize on themselves when they want to send events here, and then
 * acquire a lock on us, so we need to make sure that we don't try to lock the backend while
 * we are holding a lock on ourselves.
 */
```

在[Glance with debug]({% post_url 2014-05-30-spark-code-reading-Glance-with-debug %})的分析中，我们分析过，每个Driver都首先要生成一个`SparkContext`环境，在这个过程中，`SparkContext`会调用TaskScheduler的[`initialize()`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/SparkContext.scala#L1178)以及[`start()`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/SparkContext.scala#L200)方法，完成TaskScheduler的初始化过程，所以会在这个时候去调用`TaskSchedulerImpl`的这初始化以及启动方法。

## initialize
`initialize`函数主要是负责TaskScheduler的初始化。初始化主要包含两个重要工作，其一是指定[`SchedulerBackend`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/SchedulerBackend.scala#L27)。Spark中包含了多种SchedulerBackend，包括[`LocalBackend`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/local/LocalBackend.scala#L82)， [`SparkDeploySchedulerBackend`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/cluster/SparkDeploySchedulerBackend.scala#L28)， [`CoarseGrainedSchedulerBackend`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/cluster/CoarseGrainedSchedulerBackend.scala#L45)， [`CoarseMesosSchedulerBackend`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/cluster/mesos/CoarseMesosSchedulerBackend.scala#L46)， [`MesosSchedulerBackend`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/cluster/mesos/MesosSchedulerBackend.scala#L42)， [`SimrSchedulerBackend`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/cluster/SimrSchedulerBackend.scala#L26)等几个负责任务调度的后端。Spark的任务调度后端是Spark中重要的组成部分，可能有必要单独进行分析，请转向[Spark中的SchedulerBackend](./SchedulerBackend.md)。其实这里所谓的Backend，其实就是官方文档中关于[Scheduling Across Applications](http://spark.apache.org/docs/latest/job-scheduling.html#scheduling-across-applications)的部分。TaskScheduler便是在initialize函数中指定了应用间调度的后端。[FIXME: Need to be comfirmed]

接下来，`initialize`方法会去生成一个rootPool：`rootPool = new Pool("", schedulingMode, 0, 0)`，并在这个rootPool的基础上实例化一个`SchedulerBuilder`。值得注意的是，SchedulerBuilder主要有两类，一类是基于FIFO调度方式的`FIFOSchedulableBuilder`和基于Fair调度算法的`FairSchedulableBuilder`。两者的区别在于，前者用队列的方式来调度加入的`TaskSetManager`，采用`FIFOSchedulingAlgorithm`进行应用内的任务调度。每个job有若干个stages(map和reduce的阶段)，如果前面的stage把mem和cpu占满了，那后续来的job里的stage可能就卡住不能跑了。所以spark-0.8里出了Fair Scheduler这个新的调度模式，即第二类任务调度方法，对jobs的tasks采用轮询的方式，短的任务在长任务跑的情况下也可以得到资源并行进行，适合多用户使用的情况。详细代码见[Pool>taskSetSchedulingAlgorithm](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/Pool.scala#L53)

``` scala
  var taskSetSchedulingAlgorithm: SchedulingAlgorithm = {
    schedulingMode match {
      case SchedulingMode.FAIR =>
        new FairSchedulingAlgorithm()
      case SchedulingMode.FIFO =>
        new FIFOSchedulingAlgorithm()
    }
  }
```
`FIFOSchedulingAlgorithm`与`FairSchedulingAlgorithm`两者主要是在`def comparator(s1: Schedulable, s2: Schedulable): Boolean`函数的实现上有所区别，FIFO的调度方法主要考虑priority和stageId，而Fair算法要同时考虑runningTasks、minShare以及weight等因素。算法细节详见[`SchedulingAlgorithm.scala`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/SchedulingAlgorithm.scala#L26)。

## submitTasks
另外，从上面的代码注释来看，`TaskSchedulerImpl`是通过`DAGScheduler`的[`submitMissingTasks`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L789)方法调用它的[`submitTasks`](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/TaskSchedulerImpl.scala#L137)来接手Task的调度的。
因此，咱们可以先从`TaskSchedulerImpl`的`submitTasks`函数来分析。

``` scala
  override def submitTasks(taskSet: TaskSet) {
    val tasks = taskSet.tasks
    this.synchronized {
      val manager = new TaskSetManager(this, taskSet, maxTaskFailures)
      activeTaskSets(taskSet.id) = manager
      schedulableBuilder.addTaskSetManager(manager, manager.taskSet.properties)

      ...

      hasReceivedTask = true
    }
    backend.reviveOffers()
  }
```

submitTask的主要工作是为一个Stage中的一组Task生成一个[TaskSetManager](https://github.com/apache/spark/blob/v0.9.1/core/src/main/scala/org/apache/spark/scheduler/TaskSetManager.scala#L51)，用于封装TaskSet，提供对单个TaskSet内的tasks的track和schedule。由此可知，一个TaskSetManager中的所有Task没有相互依赖关系，是可以完全并行的。

> Schedules the tasks within a single TaskSet in the ClusterScheduler. This class keeps track of each task, retries tasks if they fail (up to a limited number of times), and handles locality-aware scheduling for this TaskSet via delay scheduling. The main interfaces to it are resourceOffer, which asks the TaskSet whether it wants to run a task on one node, and statusUpdate, which tells it that one of its tasks changed state (e.g. finished).

> THREADING: This class is designed to only be called from code with a lock on the TaskScheduler (e.g. its event handlers). It should not be called from other threads.

