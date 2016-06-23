<<<<<<< HEAD
# Job Scheduling

##Overview

Spark有一些调度计算机资源的方式(facilities)。首先回忆一下在[cluster mode overview](http://spark.apache.org/docs/latest/cluster-overview.html)中描述的，每一个Spark的应用(包含一个SparkContext的实例)以独立的executor进程集合来运行。运行Spark的集群管理器提供了[ scheduling across applications](http://spark.apache.org/docs/latest/job-scheduling.html#scheduling-across-applications)的方法。其次，在每一个Spark应用中，如果被不同的线程提交，那么多个job(Spark action)可能会并发的运行。如果你的应用服务于网络请求的话，那么这样的情况是常见的。Spark包含了一个[fair scheduler](http://spark.apache.org/docs/latest/job-scheduling.html#scheduling-within-an-application)来在不用的SparkContext调度资源。

##Scheduling Across Applications

如果在集群上运行，每个Spark应用都会获取一批独占的executor JVM，来只运行其任务和存储数据。如果有多个用户共享集群，那么会有不同的选项，取决于集群管理器。
=======
#Job Scheduling

Spark有好几种计算资源调度方式。首先回忆下，在cluster mode overview中描述的，每一个Spark应用（SparkCotext的实例）
>>>>>>> 6a682f5af0f13a31a2be7eca6378fd7bc367d9c5
