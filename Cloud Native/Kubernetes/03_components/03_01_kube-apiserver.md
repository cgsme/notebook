# API Server

API Server是Kubernetes控制平面的核心。

API服务器负载提供HTTP API，用于用户、集群中的不同部分和集群外部组件相互通信。

通过API可以查询和操作kubernetes API中的对象（比如：Pod、Namespace、ConfigMap和Event）的状态。

大部分的操作都可以通过kubectl命令或者kubeadm这类命令行工具来执行，但是这些工具底层也是调用的API。我们也可以通过REST调用来访问这些API。

客户端可依赖的类库可以参考：[客户端库](https://kubernetes.io/zh/docs/reference/using-api/client-libraries/)  
其中有官方支持的库，比如：dotnet、Go、Haskell、Java、Javascript、Python。  
以及很多由社区维护的客户端库：NodeJs、Java、Go、PHP、Perl、Ruby、Scala...等
