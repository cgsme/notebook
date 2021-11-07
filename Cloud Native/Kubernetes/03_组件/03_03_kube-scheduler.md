# scheduler

**Master节点中的组件。**

集群分发调度器。负责集群的资源调度，为新建的pod分配机器。

1、scheduler收集和分析当前kubernetes集群中所有node节点的资源负载情况，然后一次分发新建的pod到可用的节点中。
2、实时监测kubernets中未分发和已分发的所有运行的Pod。
3、监测Node节点信息，由于会频繁查找Node节点，scheduler会缓存一份最新的信息数据到本地。
4、scheduler在分发Pod到指定的node节点后，会把Pod相关的信息`Binding`写回API server。
