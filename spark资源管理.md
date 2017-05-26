spark的调度分为资源调度和任务调度。
* 资源调度支持standalone、yarn、mesos，也可以运行在云上(Amazon EC2或者IBMBulemix)。
* 任务调度一般跟executor相关。

### 调度流程

与spark资源调度相关的有：driver、master、worker、executor。
