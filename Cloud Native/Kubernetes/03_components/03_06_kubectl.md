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

### 描述 - describe

  kubectl describe pod mynginx

### 日志 - logs

### 缠绕 - attach

### 执行 - exec

### 复制 - cp

### 首次展示 - rollout

### 伸缩 - scale

### 自动伸缩 - autoscale

## 高级命令

### 新增/更新 - apply

### 标签 - label

## 其他命令

### 集群信息 - cluster-info

### 版本 - version
