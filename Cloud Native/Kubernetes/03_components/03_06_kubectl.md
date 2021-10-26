# kubectl

## 基本命令

### 查询命令 - get

kubernete中包含非常多的资源，查看各种资源的情况可以通过：

    kubectl get [resource]

- 获取节点

        kubectl get nodes

- 获取kubernetes中支持的所有api资源

        # 得到pod、nodes、service、deployment set...等等
        kubectl get api-resources

- 获取kubernetes中支持的所有资源的api版本号

        kubectl get api-version

- 查看命名空间

        # namespace可以简写为 ns。支持单复数形式
        kubectl get namespace 

- 获取pod

        # pods pod都可以，支持单数和复数形式。也可以缩写为 po
        kubectl get pods -n dev  # -n指定命名空间，默认为default

- 获取service

        # service可以缩写为svc。支持单复数形式
        kubectl get service

- 获取pv

        kubectl get pv

- 获取config map

        # configmap 可以缩写为 cm
        kubectl get configmap 

### 创建命令 - create

### 编辑命令 - edit

### 更新命令 - patch

### 解释命令 - explain

### 删除命令 - delete

## 运行和调试命令

### 运行 - run

### 暴露 - expose

`port`: 通过clusterIp访问时用的端口  
`target-port`: 目标端口，pod中提供服务的端口  
`nodePort`: 集群外部访问时用的端口，<http://NodeIP:32035>

    kubectl expose deployment my-deploy --port=8080 --target-port=80     --type=NodePort -n my-namespace
    
    [root@k8s-master ~]# kubectl get svc my-svc -o yaml

    ...
    spec:
        clusterIP: 10.104.0.64
        clusterIPs:
        - 10.104.0.64
        externalTrafficPolicy: Cluster
        ports:
        - nodePort: 32035
          port: 80
          protocol: TCP
          targetPort: 80
        selector:
          app: web
    ...

### 描述 - describe

    kubectl describe pod mynginx -n my-namespace

### 日志 - logs

### 缠绕 - attach

### 执行 - exec

### 复制 - cp

### 首次展示 - rollout

### 伸缩 - scale

    kubectl scale --replicas=3 deployment/xxx-app -n my-namespace

### 自动伸缩 - autoscale

### 滚动升级 - rolling-update

滚动升级（Rolling Update）通过逐个容器替代升级的方式来实现无中断的服务升级：

    kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2

在滚动升级的过程中，如果发现了失败或者配置错误，还可以随时回滚：

    kubectl rolling-update frontend-v1 frontend-v2 --rollback

需要注意的是，kubectl rolling-update 只针对 ReplicationController（高版本已启用，使用Replicaset替代）。对于更新策略是 RollingUpdate 的 Deployment（Deployment 可以在 spec 中设置更新策略为 RollingUpdate，默认就是 RollingUpdate），更新应用后会自动滚动升级：

    spec:
    replicas: 3
    selector:
      matchLabels:
        run: nginx-app
    strategy:
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 1
      type: RollingUpdate

而更新应用的话，就可以直接用 kubectl set 命令：

    kubectl set image deployment/nginx-app nginx-app=nginx:1.9.1

滚动升级的过程可以用 rollout 命令查看:

    $ kubectl rollout status deployment/nginx-app

    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    Waiting for rollout to finish: 2 of 3 updated replicas are available...
    Waiting for rollout to finish: 2 of 3 updated replicas are available...
    Waiting for rollout to finish: 2 of 3 updated replicas are available...
    Waiting for rollout to finish: 2 of 3 updated replicas are available...
    Waiting for rollout to finish: 2 of 3 updated replicas are available...
    deployment "nginx-app" successfully rolled out

Deployment 也支持回滚：

    $ kubectl rollout history deployment/nginx-app
    deployments "nginx-app"
    REVISION    CHANGE-CAUSE
    1        <none>
    2        <none>

    $ kubectl rollout undo deployment/nginx-app
    deployment "nginx-app" rolled back

## 高级命令

### 新增/更新 - apply

### 标签 - label

## 其他命令

### 集群信息 - cluster-info

### 版本 - version
