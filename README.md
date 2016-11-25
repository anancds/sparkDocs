# Spark的一些资源

## 课程

## 书籍

[Mastering Apache Spark](https://www.gitbook.com/book/jaceklaskowski/mastering-apache-spark/details "Mastering Apache Spark")

## 网络资源
[Spark on yarn的内存分配问题](http://www.voidcn.com/blog/wisgood/article/p-5974273.html)

## Spark summit

[spark summit](https://spark-summit.org/)

## spark启动参数

    /bigdata/salut/components/spark/bin/spark-submit --class com.uniview.salut.streaming.DealStreaming --name "StreamingProcessing" --master yarn-cluster --driver-memory 4g --num-executors ${NUM_EXECUTORS} --executor-memory 2g --executor-cores 6 --queue root.streaming --conf spark.streaming.concurrentJobs=3 --conf spark.yarn.executor.memoryOverhead=2g --conf spark.yarn.driver.memoryOverhead=2g /bigdata/salut/lib/salut-streaming*.jar ${IPS} ${IS_CLUSTER}
