# kubectl

## 常用命令

### 查看命名空间

  kubectl get namespace

### 创建pod

  kubelet run mynginx --image=nginx

### 查看pod（默认查看default命名空间的pod）

  kubectl get pod

### 描述pod

  kubectl describe pod mynginx
