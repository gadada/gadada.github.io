---
title: "sbt assembly时碰到的错误"
categories:
  - Spark实战FAQ
tags:
  - Spark
  - FAQ
---

以前使用Spark的时候数据量比较小，都直接在笔记本上跑Spark任务，最近因为有了一个集群可以玩，所以开始在集群上跑任务了。

在集群上跑任务，首先需要把Spark工程文件进行打包，然后通过`spark-submit`提交到spark集群中`spark-submit --master spark://xxxx:port ...`。
打包的方法主要由你使用的工程文件管理工具决定，如果用的是maven，那么使用`mvn package`就可以对文件进行打包。

我一直使用的sbt，所以这里主要记录sbt的打包配置和方法。
对于spark的打包方法。在[Learning Spark](http://shop.oreilly.com/product/0636920028512.do)的第七章P127中，作者做了详细介绍。
另外一个值得参考的地方是这本书github中的[build.sbt](https://github.com/databricks/learning-spark/blob/master/build.sbt)文件。

## 正确解决方案

主要步骤包括：

 - Add plugin info in $PROJECT/project/plugins.sbt

 ``` scala
 addSbtPlugin("com.github.gseitz" % "sbt-protobuf" % "0.3.3")
 ```

 - Add assembly related info in build.sbt

``` scala
organization := "com.github.huajianmao"
assemblyOption in assembly :=
  (assemblyOption in assembly).value.copy(includeScala = false)
mergeStrategy in assembly <<= (mergeStrategy in assembly) { (old) =>
  {
    case m if m.toLowerCase.endsWith("manifest.mf") => MergeStrategy.discard
    case m if m.startsWith("META-INF") => MergeStrategy.discard
    case PathList("javax", "servlet", xs @ _*) => MergeStrategy.first
    case PathList("org", "apache", xs @ _*) => MergeStrategy.first
    case PathList("org", "jboss", xs @ _*) => MergeStrategy.first
    case "about.html"  => MergeStrategy.rename
    case "reference.conf" => MergeStrategy.concat
    case _ => MergeStrategy.first
  }
}
```

我在尝试这个的过程中碰到了这么几个问题。

## 主要碰到的问题

### AssemblyKeys && assemblySettings

开始的时候我是严格按照Learnign Spark的build.sbt来配置的，即在文件开头添加了

``` scala
import AssemblyKeys._

assemblySettings
```

但结果告诉我说

``` shell
┌─hjmao at myhost in ~/workspace/myproject <master*>
└─± sbt assembly
[info] Loading project definition from /home/hjmao/workspace/myproject/project
/home/hjmao/workspace/myproject/build.sbt:1: error: not found: object AssemblyKeys
import AssemblyKeys._
       ^
/home/hjmao/workspace/myproject/build.sbt:3: error: not found: value assemblySettings
assemblySettings
^
[error] Type error in expression
Project loading failed: (r)etry, (q)uit, (l)ast, or (i)gnore? 
```

经过[搜索](https://github.com/sbt/sbt-assembly/issues/71#issuecomment-126944801)，说在新版本的sbt中，这两行其实是可以不用的。
所以我也就去掉了，问题解决！

### deduplicate: different file contents found

还一个坑是打包时的合并策略所造成的冲突。

``` shell
deduplicate: different file contents found in the following:
/home/hjmao/.ivy2/cache/commons-beanutils/commons-beanutils/jars/commons-beanutils-1.7.0.jar:overview.html
/home/hjmao/.ivy2/cache/org.codehaus.janino/janino/jars/janino-3.0.0.jar:overview.html
	at sbtassembly.Assembly$.applyStrategies(Assembly.scala:140)
	at sbtassembly.Assembly$.x$1$lzycompute$1(Assembly.scala:25)
	at sbtassembly.Assembly$.x$1$1(Assembly.scala:23)
	at sbtassembly.Assembly$.stratMapping$lzycompute$1(Assembly.scala:23)
	at sbtassembly.Assembly$.stratMapping$1(Assembly.scala:23)
	at sbtassembly.Assembly$.inputs$lzycompute$1(Assembly.scala:67)
	at sbtassembly.Assembly$.inputs$1(Assembly.scala:57)
	at sbtassembly.Assembly$.apply(Assembly.scala:83)
	at sbtassembly.Assembly$$anonfun$assemblyTask$1.apply(Assembly.scala:240)
	at sbtassembly.Assembly$$anonfun$assemblyTask$1.apply(Assembly.scala:237)
	at scala.Function1$$anonfun$compose$1.apply(Function1.scala:47)
	at sbt.$tilde$greater$$anonfun$$u2219$1.apply(TypeFunctions.scala:40)
	at sbt.std.Transform$$anon$4.work(System.scala:63)
	at sbt.Execute$$anonfun$submit$1$$anonfun$apply$1.apply(Execute.scala:228)
	at sbt.Execute$$anonfun$submit$1$$anonfun$apply$1.apply(Execute.scala:228)
	at sbt.ErrorHandling$.wideConvert(ErrorHandling.scala:17)
	at sbt.Execute.work(Execute.scala:237)
	at sbt.Execute$$anonfun$submit$1.apply(Execute.scala:228)
	at sbt.Execute$$anonfun$submit$1.apply(Execute.scala:228)
	at sbt.ConcurrentRestrictions$$anon$4$$anonfun$1.apply(ConcurrentRestrictions.scala:159)
	at sbt.CompletionService$$anon$2.call(CompletionService.scala:28)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
[error] (*:assembly) deduplicate: different file contents found in the following:
[error] /home/hjmao/.ivy2/cache/javax.inject/javax.inject/jars/javax.inject-1.jar:javax/inject/Inject.class
[error] /home/hjmao/.ivy2/cache/org.glassfish.hk2.external/javax.inject/jars/javax.inject-2.4.0-b34.jar:javax/inject/Inject.class
[error] deduplicate: different file contents found in the following:
[error] /home/hjmao/.ivy2/cache/javax.inject/javax.inject/jars/javax.inject-1.jar:javax/inject/Named.class
[error] /home/hjmao/.ivy2/cache/org.glassfish.hk2.external/javax.inject/jars/javax.inject-2.4.0-b34.jar:javax/inject/Named.class
[error] deduplicate: different file contents found in the following:
[error] /home/hjmao/.ivy2/cache/javax.inject/javax.inject/jars/javax.inject-1.jar:javax/inject/Provider.class
[error] /home/hjmao/.ivy2/cache/org.glassfish.hk2.external/javax.inject/jars/javax.inject-2.4.0-b34.jar:javax/inject/Provider.class
[error] deduplicate: different file contents found in the following:
...
```

[Stackoverflow](http://stackoverflow.com/questions/30446984/spark-sbt-assembly-deduplicate-different-file-contents-found-in-the-followi)也碰到了这个问题。
看了Learning Spark的build.sbt，就直接拷贝了他的合并策略了。

``` scala
mergeStrategy in assembly <<= (mergeStrategy in assembly) { (old) =>
  {
    case m if m.toLowerCase.endsWith("manifest.mf") => MergeStrategy.discard
    case m if m.startsWith("META-INF") => MergeStrategy.discard
    case PathList("javax", "servlet", xs @ _*) => MergeStrategy.first
    case PathList("org", "apache", xs @ _*) => MergeStrategy.first
    case PathList("org", "jboss", xs @ _*) => MergeStrategy.first
    case "about.html"  => MergeStrategy.rename
    case "reference.conf" => MergeStrategy.concat
    case _ => MergeStrategy.first
  }
}
```
