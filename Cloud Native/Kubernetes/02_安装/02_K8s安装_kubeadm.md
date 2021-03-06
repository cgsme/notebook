# k8s安装

[官方文档](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

---

- [安装方式](#安装方式)  
- [通过kubeadm安装k8s（CentOs7）（未测试）](#通过kubeadm安装k8scentos7)  
  - [1 环境准备](#1-环境准备)
    - [1.1 关闭防火墙](#11-关闭防火墙)
    - [1.2 关闭selinux](#12-关闭selinux)
    - [1.3 关闭swap分区（K8s禁止虚拟内存以提高性能）](#13-关闭swap分区k8s禁止虚拟内存以提高性能)
    - [1.4 在master添加hosts](#14-在master添加hosts)
    - [1.5 设置网桥参数](#15-设置网桥参数)
    - [1.6 时间同步](#16-时间同步)
  - [2 K8s具体安装步骤](#2-k8s具体安装步骤)
    - [2.1 安装Docker](#21-安装docker)
      - [2.1.1 更新Docker的yum源](#211-更新docker的yum源)
      - [2.1.2 安装指定版本的Docker](#212-安装指定版本的docker)
      - [2.1.3 配置加速器加速下载](#213-配置加速器加速下载)
    - [2.2 安装k8s](#22-安装k8s)
      - [2.2.1 添加k8s的阿里云yum源](#221-添加k8s的阿里云yum源)
      - [2.2.2 安装kubeadm/kubelet/kubectl](#222-安装kubeadmkubeletkubectl)
      - [2.2.3 部署kubernetes master主节点](#223-部署kubernetes-master主节点)
      - [2.2.4 将node节点加入集群](#224-将node节点加入集群)
      - [2.2.5 部署网络插件](#225-部署网络插件)

---

## 安装方式

- minikube
- Kind
- kubeadm
- kubespray
- 二进制包
- yum （版本太旧）

## 通过kubeadm安装k8s（CentOs7）

### 1 环境准备

#### 1.1 关闭防火墙

    systemctl stop firewalld
    systemctl disable firewalld

#### 1.2 关闭selinux

    #永久关闭
    sed -i 's/enforcing/disabled/' /etc/selinux/config  

    # 或者 将 SELinux 设置为 permissive 模式（相当于将其禁用）
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

    setenforce 0  # 临时关闭

#### 1.3 关闭swap分区（K8s禁止虚拟内存以提高性能）

    swapoff -a  # 临时关闭
    sed -ri 's/.*swap.*/#&/' /etc/fstab  # 永久关闭

#### 1.4 在master添加hosts

    #每台机器分别设置hostname
    hostnamectl set-hostname k8smaster
    hostnamectl set-hostname k8snode01
    #hostnamectl set-hostname k8snode02

    cat >> /etc/hosts << EOF
    192.168.0.10 k8smaster
    192.168.0.11 k8snode01
    192.168.0.12 node2
    EOF

#### 1.5 设置网桥参数

    # 确保 br_netfilter 模块被加载。这一操作可以通过运行 lsmod | grep br_netfilter 来完成
    # 若要显式加载该模块，可执行 sudo modprobe br_netfilter。
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    br_netfilter
    EOF

    # 确保节点上的 iptables 能够正确地查看桥接流量，增加以下内容
    cat >> /etc/sysctl.d/k8s.conf << EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-call-iptables = 1
    EOF
  
    sysctl --system   # 生效
  
#### 1.6 时间同步

    yum install ntpdate -y
    ntpdate time.windows.com
  
### 2 K8s具体安装步骤

所有服务器节点安装：`Docker/kubeadm/kubelet`

k8s默认容器运行环境是Docker，因此首先需要安装Docker；

#### 2.1 安装Docker

##### 2.1.1 更新Docker的yum源

    wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo-0/ /etc/yum.repos.d/docker-ce.repo

##### 2.1.2 安装指定版本的Docker

    yum install docker-ce-19.03.13 -y
    # yum install docker -y # (这个版本可能偏旧)

    # 启动docker
    sudo systemctl enable docker
    sudo systemctl daemon-reload
    sudo systemctl restart docker

##### 2.1.3 配置加速器加速下载

    vi /etc/docker/daemon.json    # 不存在则手动创建
    # 加入以下内容
    {
        "registry-mirrors": ["https://registry.docker-cn/com"]
    }
    # 生效 启动docker
    systemctl enbale docker.service & systemctl start docker

#### 2.2 安装k8s

##### 2.2.1 添加k8s的阿里云yum源

默认从谷歌的源下载，国内无法访问。添加以下配置：

    cat > /etc/yum.repos.d/kubernetes.repo << EOF
    [kubernetes]
    name=Kubernetes
    baseUrl=https://mirrors.aliyum.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=0
    repo_gpgcheck=0
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
    https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF

##### 2.2.2 安装kubeadm/kubelet/kubectl

    yum install kubelet-1.19.4 kubeadm-1.19.4 kubectl-1.19.4 -y
    # 然后执行
    systemctl enable kubelet.service
    
    # 查看安装情况
    yum list installed | grep kubelet
    yum list installed | grep kubeadm 
    yum list installed | gerp kubectl
    
    # 查看安装的版本号
    kubelet --version

    # 开机启动
    sudo systemctl enable --now kubelet

- kubelet: 运行在所有的cluster节点上，负责启动pod和容器。
- kubeadm：用于初始化cluster的工具。
- kubectl：kubectl是kubernetes的命令行工具，通过kubectl可以部署和管理应用，查看各种资源，创建、删除和更新组件。

##### 2.2.3 部署kubernetes master主节点

在master主节点执行：

    kubeadm init --apiserver-advertise-address=192.168.0.39 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.20.5 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16

**说明**：service-cidr的选取不能和PodCIDR及本机网络有重叠或者冲突，一般可以选择一个本机网络和PodCIDR都没有用到的私有网段地址，比如PodCIDR使用10.244.0.0/16，那么service-cidr可以选择10.96.0.0/12，网络无重叠冲突即可。

然后在master节点执行(上一个步骤中的日志中有描述)：

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    # 查看k8s集群中的节点
    kubectl get nodes

##### 2.2.4 将node节点加入集群

在work node中执行上面init命令输出的日志中的最后一部分内容，大致如下：

    kubeadm join 192.168.0.10:6443 --token xxx --discovery-token-ca-cert-hash adfagasdljlj23jr18u12j213jxxxxxx

##### 2.2.5 部署网络插件

    wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

应用kube-flannel.yml文件，得到运行时容器：

    kubectl apply -f kube-flannel.yml

查看运行时pod：

    kubectl get pods -n kube-system

## kubeadm 工作原理

**`kubeadm init`工作流程**：

- 执行 kubeadm init 之后，kubeadm会先执行一系列的检查工作，确保机器可以用来部署Kubernetes。这一步检查称为 `Preflight Checks`。
- 生成Kubernetes对外提供服务所需的各种证书和对应的目录。

    Kubernetes 对外提供服务时，除非专门开启 “不安全模式”，否则都要通过 https 才能访问 kube-apiserver 。生成的证书文件都放在 Master 节点的 `/etc/kubernetes/pki` 目录下。这个目录下最主要的证书文件时 ca.crt 和对应的私钥 ca.key 。

    此外，用户使用 kubectl 获取容器日志等 `streaming` 操作时，需要通过 kube-apiserver 向 kubelet 发起请求，这个连接也必须时安全的。kubeadm 为这一步生成的时 apiserver-kubelet-client.crt 文件。对应的私钥是 apiserver-kubelet-client.key。

    ... 还有其他证书。

- 证书生成后，kubeadm 为其他组件生成方位 kube-apiserver所需的配置文件。
  
    这些文件的路径为：/etc/kubernetes/xxx.conf:

        ls /etc/kubernetes/

        admin.conf  controller-manager.conf  kubelet.conf  scheduler.conf

- 为 Master 组件生成 Pod 配置文件
  
    Master 组件包括：kube-apiserver、kube-controller-manager、kube-scheduler。它们都会以 Pod 的方式部署。

    此时 Kubernetes 集群还未创建，所以这些组件都会以 `Static Pod` 的形式启动。（即：将需要部署的 Pod 的 YAML 文件放在一个指定的目录里。当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在这台机器上启动它们）YAML 文件会被生成在 /etc/kubernetes/manifests 路径下。

    可以看出 kubelet 在 Kubernetes 中的地位非常高，在设计上它就是一个完全独立的组件，而其他 Master 组件，则更像是辅助性的系统容器。

- 生成 etcd 的 Pod YAML 文件，同样通过 static pod 的方式启动 etc
- Master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 Master 组件的健康检查 URL，等待 Master 组件完全运行起来。
- 为集群生成一个 bootstrap token。只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 kubeadm join 加入到集群中。这个 token 在 kubeadm init 执行结束之后打印在控制台。
- token 生成之后，kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，供后续部署 Node 节点使用。这个 ConfigMap 的名字是 cluster-info。
- 安装默认插件。Kubernetes 默认kube-proxy 和 DNS 这两个插件是必须安装的。分别用来提供整个集群的服务发现和 DNS 功能。这两个插件也只是两个容器镜像。

**`kubeadm join`工作流程**

为什么 `kubeadm join` 需要一个 token？

因为任何一台机器想要成为 Kubernetes 集群中的一个节点，就必须在集群的 kube-apiserver 上注册。需要跟 apiserver 通信就必须要获取到相应的证书文件（CA文件）。为了一件安装，就不能让用户手动去 Master 节点上手动拷贝这些文件。

所以 kubeadm 至少需要发起一次“不安全模式”的访问到 kube-apiserver，从而拿到保存在 ConfigMap 中的 cluster-info（它保存了APIServer的授权信息）。而 token 就是这个过程中的安全验证的角色。

只要有了 cluster-info 里的 kube-apiserver 的地址、端口、证书，kubelet 就可以以“安全模式”连接到 apiserver 上，这样一个新的节点就部署完成了。
