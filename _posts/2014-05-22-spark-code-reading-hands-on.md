---
title: "Spark代码阅读(3) Hands On"
categories:
  - Code Reading
tags:
  - Spark
---

Spark上手，估计也就只能自己装上个Spark试试了吧。Spark的安装总的来说，还是比较简单的，主要有两总方法吧，二进制文件安装和源码编译安装。毕竟我的目的是要学习一下Spark的代码，所以如果直接使用二进制文件，似乎没法改改代码，看看效果了。所以从这个目的出发，我们也就需要自己去编译Spark的代码了。

获得Spark的代码，可以有两种来源：[Github](https://github.com/apache/spark)或者[打好的包](http://spark.apache.org/downloads.html)，因为git里面还包含了代码提交的一些[历史信息](https://github.com/apache/spark/network)，因此git下来的代码似乎更适合用来代码学习了。

{% include toc %}

## 获取git代码
``` shell
mkdir ~/workspace
git clone git@github.com:apache/spark.git
```

## Compile
``` shell
cd ~/workspace/spark
./sbt/sbt assembly
```

## For IntelliJ Idea IU
读代码有个好工具还是必须的，它可以很方便的定位所关注的代码，可以很方便的找到哪些代码用了你所关注的代码等等。
Eclipse以及Idea IU对Scala的支持还是不错的，咱们可以很方便的在IdeaIU里读Spark的代码。但是首先要生成一些支持文件，命令如下。

``` shell
./sbt/sbt update gen-idea
```

不过，这个过程对墙内的同学来说，可是个漫长的痛苦的等待啊！经过这步之后，就可以开始打开IdeaIU，然后打开这个Spark工程了。就可以在IdeaIU里面很happy的看代码啦！记得好好用好IdeaIU的Ctrl加鼠标的功能以及IdeaIU里的Find Usage功能。[^1]

[^1]: 不过需要注意的是，我当时碰到过两个问题，一个问题是总是编译不了，具体的出错信息是什么记不清了，当时主要是在64位的Ubuntu上进行的，好像说是home目录加密了还是什么的（可是我没有加密呀～～～），反正后来是通过把Spark代码的目录放到根目录或者opt目录下了。另外一个问题是，IdeaIU提示我说没有SDK，所以加一个JDK就好啦～

前面咱们也已经编译好Spark了，IdeaIU里也可以看Spark代码了，咱们还需要去跑个例子程序。用[Word Count]({% post_url 2014-05-06-spark-code-reading-an-example %})这个例子，咱们就可以看到里面相应的运行记录啦！
知乎上有一篇连城的[回答](https://www.zhihu.com/question/24869894/answer/97339151)，很详细的介绍了怎么搭阅读和调试Spark源码。

And good luck!
