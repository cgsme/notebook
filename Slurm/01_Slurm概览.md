# Slurm

## 基本介绍

Slurm是一个开源、容错、高度可扩展的集群管理和作业调度系统，适用于大小linux集群。作为集群关键负载管理器，Slurm具有三个关键功能：

- 在一段时间内为用户分配对资源（计算节点）的独占和/或非独占的访问权限，以便他们可以执行工作。
- 提供了一个框架，用于在已分配的节点集合上启动、执行和监控工作（通常是并行的job）。
- 通过管理待办的工作队列来处理资源的争抢。

有很多可选的插件用于计费、高级预留、资源调度（并行作用的时间共享）、回填调度、已经拓扑优化的资源选择、用户或银行账户的资源限制以及复杂的多因素作业优先级算法。

## Slurm架构

Slurm有一个集中管理`slurmctld`来监控资源和工作，如果发生故障的话也可以有一个备节点来保证可用性。每一个**计算服务器节点**都有一个可以被认为是一个远程shell的`slurmd`守护进程：它一直等待工作、执行工作、返回状态并等待更多的工作，它提供了容错分层通信。可选的`slurmdbd`（Slurm数据库守护程序），可用于在单个数据库中记录多个Slurm管理的集群的计费信息。可选的`slurmrestd`（slurm的REST API守护进程）用户通过Slurm的REST API与Slurm进行交互。

用户工具包括（所有命令都可以在集群的任何节点上执行）：

- `srun`: 初始化jobs。
- `scancel`：终止队列中或运行中的jobs。
- `sinfo`: 报告系统状态。
- `squeue`：报告队列中的jobs的状态。
- `sacct`：获取运行结束或正在运行中的jobs和job steps的信息。
- `sview`：以图形方式报告系统和作业的状态，包括网络拓扑。
- `scontrol`：管理工具，监控或修改集群上的配置和状态信息。
- `sacctmgr`：管理工具，用来管理数据库。可以用户识别集群、有效用户、有效银行账户等等。

SLurm组件，如下图：

![SLurm组件](https://slurm.schedmd.com/arch.gif 'SLurm组件')

Slurm有一个通用的插件机制，可以轻松支持各种基础设施。这运行使用构建块的方式进行各种Slurm配置。有以下插件：

- `Accounting Storage`：todo
- `Account Gather Energy`：todo
- `Authentication of communications`：todo
- `Containers`：todo
- `Credential (Digital Signature Generation)`：todo
- `Generic Resources`：todo
- `Job Submit`：todo
- `Job Accounting Gather`：todo
- `Job Completion Logging`：todo
- `Launchers`：todo
- `MPI`：todo
- `Preempt`：todo
- `Priority`：todo
- `Process tracking (for signaling)`：todo
- `Scheduler`：todo
- `Node selection`：todo
- `Site Factor (Priority)`：todo
- `Switch or interconnect`：todo
- `Task Affinity`：todo
- `Network Topology`：todo

![Slurm实体](https://slurm.schedmd.com/entities.gif 'Slurm实体')

上图展示的是被这些Slurm守护管理的实体，包括节点（**nodes**）,Slurm中的计算资源，**分区**（**partition**，将节点按逻辑分组），jobs或在指定时间内分配给用户的资源，和**job steps**（job中的任务及（可能是并行的））。

**分区**可以被当作是job的队列，每个分区都有各种各样的限制，比如作业大小限制、job时间限制、允许使用他的用户等等。在**分区**内为已经按照优先级排序的jobs分配节点，直到该分区的资源（节点、处理器、内存等）耗尽。

一旦为jobs分配了一组节点，用户就可以在已分配的资源内以job steps的形式初始并行工作。例如，一个单个的job step可以利用已分配的资源中的所有节点来开启，或者多个job steps可以独立地使用已分配的资源的一部分。Slurm为分配给job处理器提供了资源管理，以便于多个job steps可以被同时提交可排队，直到已分配的资源中有可用资源。

## 可配置性

监控的节点状态包括：**处理器数量**，**真实内存的大小**，**临时磁盘空间大小**，和**状态**（UP，DOWN等）。附加节点信息包含了权重（分配工作的优先级）和特征（任意信息，如处理器速度或类型）。节点被分组到分区中，其中可能包含重复的节点，所以最好把分区视为作业队列。分区信息包括：名称，关联节点列表，状态（UP或DOWN），job的最大时间限制，每个job的最多节点数，组访问列表，优先级（如果节点在多个分区中很重要），和具有可选的拥有超额订阅级别的资源调度的共享节点访问策略（例如：YES，NO或者FORCE:2）。Bit maps用于表示节点，并且可用通过执行少量的比较和一系列快速的位图操作来做出调度决策。一个例子（slurm.conf的一部分）：

    #
    # Sample /etc/slurm.conf
    #
    SlurmctldHost=linux0001  # Primary server
    SlurmctldHost=linux0002  # Backup server
    #
    AuthType=auth/munge
    Epilog=/usr/local/slurm/sbin/epilog
    PluginDir=/usr/local/slurm/lib
    Prolog=/usr/local/slurm/sbin/prolog
    SlurmctldPort=7002
    SlurmctldTimeout=120
    SlurmdPort=7003
    SlurmdSpoolDir=/var/tmp/slurmd.spool
    SlurmdTimeout=120
    StateSaveLocation=/usr/local/slurm/slurm.state
    TmpFS=/tmp
    #
    # Node Configurations
    #
    NodeName=DEFAULT CPUs=4 TmpDisk=16384 State=IDLE
    NodeName=lx[0001-0002] State=DRAINED
    NodeName=lx[0003-8000] RealMemory=2048 Weight=2
    NodeName=lx[8001-9999] RealMemory=4096 Weight=6 Feature=video
    #
    # Partition Configurations
    #
    PartitionName=DEFAULT MaxTime=30 MaxNodes=2
    PartitionName=login Nodes=lx[0001-0002] State=DOWN
    PartitionName=debug Nodes=lx[0003-0030] State=UP Default=YES
    PartitionName=class Nodes=lx[0031-0040] AllowGroups=students
    PartitionName=DEFAULT MaxTime=UNLIMITED MaxNodes=4096
    PartitionName=batch Nodes=lx[0041-9999]
