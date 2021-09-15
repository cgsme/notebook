# ssh-keygen 免密登录

当前三台主机，主机A：192.168.0.10，主机B192.168.0.11，主机C：192.168.0.12。

想要从主机A免密登录至主机B、主机C：

## 公钥方式

执行以下命令：

    #这条命令会在`·`~/.ssh/`目录下生成私钥和公钥 `id_rsa`、`id_rsa.pub`
    ssh-keygen -t rsa   #一直回车就行

    #将公钥发送到需要免密登录的主机的`~/.ssh/`中
    ssh-copy-id -i ~/.ssh/id.rsa.pub root@192.169.0.11
    ssh-copy-id -i ~/.ssh/id.rsa.pub root@192.169.0.12

## 参考

- [Linux下实现免密码登录(超详细)](https://www.cnblogs.com/yixue2017/p/7559970.html)
