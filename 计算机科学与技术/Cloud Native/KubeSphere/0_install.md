# 安装KubeShphere

## 目录

- [安装KubeShphere](#安装kubeshphere)
  - [目录](#目录)
  - [1 安装Kubernetes](#1-安装kubernetes)
  - [2 安装KubeSphere前置环境（默认存储环境）](#2-安装kubesphere前置环境默认存储环境)
    - [2.1 安装NFS文件系统](#21-安装nfs文件系统)
      - [2.1.1 安装nfs-server](#211-安装nfs-server)
      - [2.1.2 配置nfs-client（可选操作）](#212-配置nfs-client可选操作)
      - [2.1.3 配置默认存储](#213-配置默认存储)
    - [2.2 安装metrics-server](#22-安装metrics-server)
  - [3 安装KubeSphere](#3-安装kubesphere)

> 参考

- [在Kubernetes上安装KubeSphere](https://kubesphere.com.cn/docs/quick-start/minimal-kubesphere-on-k8s/)

## 1 安装Kubernetes

参考[通过kubeadm安装Kubernetes](../Kubernetes/02_install/02_K8s安装_kubeadm.md)

## 2 安装KubeSphere前置环境（默认存储环境）

### 2.1 安装NFS文件系统

#### 2.1.1 安装nfs-server

1）在每个节点中运行：

    yum install -y nfs-utils

2）在master节点中执行：

    # 配置nfs服务器挂载点（共享目录）
    echo "/nfs/data *(insecure,rw,sync,no_root_squash) > /etc/exports 

    # 创建共享目录
    mkdir -p /nfs/data

    # 启动nfs相关服务
    systemctl enable rpcbind
    systemctl enable nfs-server
    systemctl start rpcbind
    systemctl start nfs-server

    # 使配置生效
    exportfs -r

    # 检查配置是否生效
    exportfs

#### 2.1.2 配置nfs-client（可选操作）

在需要挂载nfs共享目录的节点中执行：

    # showmount -e nfs服务器IP

    # 创建用于挂载的目录
    mkdir -p /nfs/data

    mount -t nfs nfs服务器IP:/nfs/data /nfs/data

#### 2.1.3 配置默认存储

配置动态供应的默认存储类。

:tent: FIXME :joy:

### 2.2 安装metrics-server

## 3 安装KubeSphere

> 参考：

- [在Kubernetes上安装KubeSphere](https://kubesphere.com.cn/docs/quick-start/minimal-kubesphere-on-k8s/)