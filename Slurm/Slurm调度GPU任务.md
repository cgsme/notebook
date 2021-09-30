# Slurm 调度GPU任务 - 测试（CUDA）

简单概述Slurm中管理gpu任务。

## 安装CUDA

### 0、依赖

gcc gcc-g++ 等等

### 1、查看GPU驱动版本以及对应的CUDA版本

```shell
[root@slurm10007 547016]# nvidia-smi
Thu Sep 30 10:17:03 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.40.04    Driver Version: 418.40.04    CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           Off  | 00000000:00:07.0 Off |                    0 |
| N/A   42C    P8    26W / 149W |      0MiB / 11441MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

nvidia-smi命令需要安装好nvidia驱动才有  
输出第一行中包含了nvidia驱动版本418.40.04、CUDA版本10.1

### 2 下载对应版本的CUDA run或者rpm文件并安装

#### 2.1 下载

比如CUDA版本为10.1，则在官网中找到对应的文件下载即可。（内网无法下载，可以在外网下载之后复制）

[CUDA ToolKit 10.1 官网下载地址](https://developer.nvidia.com/cuda-10.1-download-archive-update2?target_os=Linux&target_arch=x86_64&target_distro=CentOS&target_version=7&target_type=runfilelocal)

#### 2.2 安装

```shell
sudo sh cuda_10.1.243_418.87.00_linux.run
```

安装过程中会有一些确认，按提示操作即可。过程中可以选择需要安装的组件，比如驱动、依赖、TookKit、案例等等。  
若不希望重新安装驱动，则去掉驱动的选项即可。

#### 2.3 配置环境变量

```shell
[root@slurm10007 547016]# export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64/:$LD_LIBRARY_PATH
[root@slurm10007 547016]# export PATH=/usr/local/cuda-10.1/bin/:$PATH
```

#### 2.4 验证安装是否成功

1）终端输入，查看CUDA版本号。有输出版本号则代表正常

```shell
nvcc -V

nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Sun_Jul_28_19:07:16_PDT_2019
Cuda compilation tools, release 10.1, V10.1.243

```

2）编译sample例子

CUDA默认安装目录为/usr/local/cuda-10.1/  

```shell

# 编译并执行测试设备查询
cd /usr/local/cuda-10.1/samples/1_Utilities/deviceQuery
sudo make
./deviceQuery

# 编译并测试宽带
cd /usr/local/cuda-10.1/samples/1_Utilities/bindwidthTest
sudo make
./bindwidthTest
```

若输出的测试结果均为Result=PASS，即说明安装成功。

### 3 运行官方示例，测试GPU占用情况

找一个官方示例，比如，/usr/local/cuda-10.1/samples/1_Utilities/UnifiedMemoryPerf/

```shell
cd /usr/local/cuda-10.1/samples/1_Utilities/UnifiedMemoryPerf/UnifiedMemoryPerf
sudo make
./UnifiedMemoryPerf
```

这个案例的运行时间大概十几秒，可以通过以下命令查看GPU使用情况

```shell
# 每隔1秒刷新结果
nvidia-smi -l 1


Thu Sep 30 10:51:14 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.40.04    Driver Version: 418.40.04    CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           Off  | 00000000:00:07.0 Off |                    0 |
| N/A   53C    P0    84W / 149W |     67MiB / 11441MiB |     89%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0     14882      C   ./UnifiedMemoryPerf                           56MiB |
+-----------------------------------------------------------------------------+




Thu Sep 30 10:51:15 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.40.04    Driver Version: 418.40.04    CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           Off  | 00000000:00:07.0 Off |                    0 |
| N/A   52C    P0    83W / 149W |     67MiB / 11441MiB |    100%      Default |
+-------------------------------+----------------------+----------------------+
                                                                              
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0     14882      C   ./UnifiedMemoryPerf                           56MiB |
+-----------------------------------------------------------------------------+

```

## 通过Slurm提交GPU任务（利用CUDA官方示例）

在Slurm工作目录中创建2个临时脚本：

- midScript.sh（通过脚本执行官方示例，官方示例c语言编译后的二进制文件，Slurm貌似不能直接执行,就通过这个临时shell脚本来调用）  
- test_cuda_job.sh （Slurm命令写在这，后面直接运行这个脚本来提交JOB）

```shell
vi midScript.sh

#!/bin/bash
/usr/local/cuda-10.1/samples/1_Utilities/UnifiedMemoryPerf/UnifiedMemoryPerf


vi test_cuda_job.sh

#!/bin/bash
sbatch -N 1 -c 2 -A p1 --mem=10 -G 1 -p q1gpu --uid=[系统存在的用户id，默认root] --gid=[系统存在的组id，默认root] -D /etc/job-file/user/547016 /etc/job-file/user/547016/midScript.sh
```

执行提交任务的脚本test_cuda_job.sh，并查看GPU占用情况

```shell
./test_cuda_job.sh

nvidia-smi -l 1
```
