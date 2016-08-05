# Job Scheduling

## Overview

Spark有一些调度计算机资源的方式(facilities)。首先回忆一下在[cluster mode overview](http://spark.apache.org/docs/latest/cluster-overview.html)中描述的，每一个Spark的应用(包含一个SparkContext的实例)以独立的executor进程集合来运行。运行Spark的集群管理器提供了[ scheduling across applications](http://spark.apache.org/docs/latest/job-scheduling.html#scheduling-across-applications)的方法。其次，在每一个Spark应用中，如果被不同的线程提交，那么多个job(Spark action)可能会并发的运行。如果你的应用服务于网络请求的话，那么这样的情况是常见的。Spark包含了一个[fair scheduler](http://spark.apache.org/docs/latest/job-scheduling.html#scheduling-within-an-application)来在不用的SparkContext调度资源。

## Scheduling Across Applications

如果在集群上运行，每个Spark应用都会获取一批独占的executor JVM，来只运行其任务和存储数据。如果有多个用户共享集群，那么会有不同的选项，取决于集群管理器。

对于Spark支持的所有集群管理器而言，最简单的分配资源的方式就是静态的分配资源。对于这种方式，每个Spark的应用都设定一个最大的资源总量，并且在整个应用的生命周期内都占用这些资源。这种方式在Spark的standalone和YARN模式，还有Mesos的粗粒度模式下都可用。资源分配可以根据如下的集群类型来配置。

* Standalone mode: 默认情况下，Spark应用在Standalone模式下是按FIFO(first-in-first-out)提交的。并且spark应用会占用所有可用节点。不过你可以通过设置spark.cores.max或者spark.deploy.defaultcores来限制单个应用所占用的节点个数。最后除了可以控制CPU核数外，还可以通过spark.executor.memory来控制各个应用的内存占有量。

* mesos：在Mesos中使用静态分配的话，需要设置spark.mesos.coarse为true，你也可以选择性的设置spark.cores.max
