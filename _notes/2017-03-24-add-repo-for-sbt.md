---
title: "设置sbt源"
categories:
  - Spark实战FAQ
tags:
  - SBT
  - 配置
  - FAQ
---


在墙内使用SBT最大的痛苦莫过于等待了。
所以换一下全局sbt配置吧。

## 方法

给sbt换个国内的源。国内差不多最好的源估计是阿里云提供的。

在`~/.sbt/`下添加一个`repositories`文件，里面内容如下：

``` shell
cat ~/.sbt/respositories
```

``` ini
[repositories]
local
aliyun: http://maven.aliyun.com/nexus/content/groups/public/
typesafe: http://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
sonatype-oss-releases
maven-central
sonatype-oss-snapshots
```
