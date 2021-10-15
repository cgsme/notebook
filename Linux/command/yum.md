# yum

yellowdog update modified。python写的，基于rpm的软件包管理工具

## yum仓库

### yum仓库组成

- rpm软件包
- 相关文件（repodata）
  - 依赖信息数据库
  - 软件包列表文件
  - 包组列表文件

### 仓库配置介绍

yum仓库位于 `/etc/yum.repos.d` 目录下。可以在改目录中创建自定义的repo文件，repo文件配置项包括：

- \[local]  #仓库名，标识符
- name: this is local repo  #仓库描述，说明仓库用途
- baseurl: URL  #仓库位置。如：file://xxx。yum源提供的方式，包括FTP\HTTP\FILE
- gpgcheck: 0/1  #是否做证书检查
- gpkey: URL #证书路径
- enabled: 0/1 #是否激活次仓库，默认开启

### 常用命令

> 添加仓库

    yum-config-manager --add-repo http://www.example.com/example.repo

> 指定软件安装路径

    yum --installroot=/xxx/xxx --releaserve=/ install vim
  