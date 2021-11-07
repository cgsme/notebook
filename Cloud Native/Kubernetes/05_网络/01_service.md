# Service

## Service类型

- ClusterIp: 默认。是Kubernetes系统自动分配的虚拟IP，**只能在集群内部访问**。
- NodePort: 将Service通过指定的Node上的端口暴露给外部，通过此方法就可以在集群外部访问服务。（可能导致节点端口不够用）
- LoadBlancer：使用外接负载均衡器完成服务的负载分发，此模式需要外部云环境支持。（需要集群外部的负载均衡设备）
- ExternalName：把集群**外部的服务引入集群内部**，直接使用。
- Headliness：自己控制负载均衡策略。这类service不会分配ClusterIp，如果想访问service，只能通过service的域名进行查询。

## Endpoint

EndPoint是Kubernetes中的一个资源对象，存储在etcd中，用来记录一个service对应的所有Pod的访问地址。是根据Service配置文件中的selector的描述产生的。

一个Service由一组Pod组成，这些Pod通过Endpoint暴露出来，Endpoint是实现实际服务的端点集合。也就是说，Service和Pod的联系是通过Endpoint实现的。

## 负载分发策略

对service的访问会分发到后端的Pod上去，目前Kubernetes提供了两种负载分发策略：

- 如果不定义，默认采用kube-proxy组件的策略，比如随机、轮询。
- 基于客户端地址的**会话保持**模式，即来自同一个客户端发起的所有请求都会转发到固定的一个Pod上。此模式可以在`spec`上添加`sessionAffinity: ClientIP`选项。
  