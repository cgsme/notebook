# controller-manager

**Master节点中的组件。**

内部管理控制中心。容器编排。

负责执行各种控制器：

- endpoint-controller

定期关联service和Pod（关联信息由endpoint对象维护），保证service到Pod的映射总是最新的。

- replication-controller

定期关联replicationController和Pod，保证replicationController定义的副本数量与实际运行的Pod数量一致。
