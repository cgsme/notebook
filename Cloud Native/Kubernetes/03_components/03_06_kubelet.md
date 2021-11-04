# kubelet

- 通过`CRL(Container Runtime Interface)`与`容器运行时`交互。（CRI：容器运行时接口，定义了容器运行时的各项核心操作，比如：启动一个容器需要的所有参数）
- 通过gRPC协议同一个叫做Device Plugin的插件进行交互。这个插件是Kubernetes项目用来管理GPU等宿主机物理设备的主要组件，也是基于Kubernetes项目进行机器学习训练、高性能作业支持等工作必须关注的功能。
- 调用网络插件和存储插件为容器配置网络和支持和存储。这两个插件与 kubelet 进行交互的接口分别是：CNI（Container Network Interface）和 CSI（Container Storage Interface）

默认情况下 Master 节点是不允许运行用户 Pod 的。实现这个的方式是通过 Kubernetes 的 Taint / Toleration 机制。原理很简单：一旦某个节点被加上了一个 Taint，即被“打上污点”，那么所有 Pod 就都不能在这个节点上运行，因为 Kubernetes 的 Pod 都有“洁癖”。除非 Pod 声明自己能“容忍”这个“污点”，即声明了 Toleration，它才可以在这个节点上运行。
