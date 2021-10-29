# Ansible

Ansible是新出现的基于Python开发的自动化运维工具，集合了众多运维工具（puppet、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。  

Ansible是基于 paramiko 开发的,并且基于模块化工作，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。ansible不需要在远程主机上安装client/agents，因为它们是基于ssh来和远程主机通讯的。  

ansible目前已经已经被红帽官方收购。

## ansible 程序结构

安装目录如下(yum安装)：  
　　配置文件目录：/etc/ansible/  
　　执行文件目录：/usr/bin/  
　　Lib库依赖目录：/usr/lib/pythonX.X/site-packages/ansible/  
　　Help文档目录：/usr/share/doc/ansible-X.X.X/  
　　Man文档目录：/usr/share/man/man1/

## ansible配置文件查找顺序

- 检查环境变量ANSIBLE_CONFIG指向的路径文件(export ANSIBLE_CONFIG=/etc/ansible.cfg)；
- ~/.ansible.cfg，检查home目录下的ansible.cfg配置文件；
- /etc/ansible.cfg检查etc目录的配置文件。

## ansible配置文件

ansible 的配置文件为/etc/ansible/ansible.cfg，一些常见的参数：

    inventory = /etc/ansible/hosts      # 这个参数表示资源清单inventory文件的位置
    library = /usr/share/ansible        # 指向存放Ansible模块的目录，支持多个目录方式，只要用冒号（：）隔开就可以
    forks = 5                           # 并发连接数，默认为5
    sudo_user = root                    # 设置默认执行命令的用户
    remote_port = 22                    # 指定连接被管节点的管理端口，默认为22端口，建议修改，能够更加安全
    host_key_checking = False           # 设置是否检查SSH主机的密钥，值为True/False。关闭后第一次连接不会提示配置实例
    timeout = 60                        # 设置SSH连接的超时时间，单位为秒
    log_path = /var/log/ansible.log     # 指定一个存储ansible日志的文件（默认不记录日志）

## ansible 常用命令

    /usr/bin/ansible　　            # Ansibe AD-Hoc 临时命令执行工具，常用于临时命令的执行
    /usr/bin/ansible-doc 　　       # Ansible 模块功能查看工具
    /usr/bin/ansible-galaxy　　     # 下载/上传优秀代码或Roles模块 的官网平台，基于网络的
    /usr/bin/ansible-playbook　     # Ansible 定制自动化的任务集编排工具
    /usr/bin/ansible-pull　　       # Ansible远程执行命令的工具，拉取配置而非推送配置（使用较少，海量机器时使用，对运维的架构能力要求较高）
    /usr/bin/ansible-vault　　      # Ansible 文件加密工具
    /usr/bin/ansible-console　　    # Ansible基于Linux Consoble界面可与用户交互的命令执行工具
