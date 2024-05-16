# scp 命令

## scp介绍

scp是secure copy的缩写，是linux下基于ssh登陆进行安全的远程文件拷贝命令。scp是加密的，rcp是不加密的。scp是rcp的加强版。

由于scp是加密的，可能速度会有点影响。scp占用系统资源少，对系统负荷低。

## 语法

    scp [-1246BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file] [-l limit] [-o ssh_option] [-P port] 
    [-S program] [[user@]host1:]file1 [...] [[user@]host2:]file2

简易写法：

    scp [可选参数] file_source file_target

### 参数说明

    -1： 强制scp命令使用协议ssh1
    -2： 强制scp命令使用协议ssh2
    -4： 强制scp命令只使用IPv4寻址
    -6： 强制scp命令只使用IPv6寻址
    -B： 使用批处理模式（传输过程中不询问传输口令或短语）
    -C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
    -p： 保留原文件的修改时间，访问时间和访问权限。
    -q： 不显示传输进度条。
    -r： 递归复制整个目录。
    -v： 详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
    -c cipher：        以cipher将数据传输进行加密，这个选项将直接传递给ssh。
    -F ssh_config：    指定一个替代的ssh配置文件，此参数直接传递给ssh。
    -i identity_file： 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
    -l limit：         限定用户所能使用的带宽，以Kbit/s为单位。
    -o ssh_option：    如果习惯于使用ssh_config(5)中的参数传递方式
    -P port：          注意是大写的P, port是指定数据传输用到的端口号
    -S program：       指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项

## 示例

两台主机，主机A：192.168.0.10，主机B：192.168.0.20

- 下载文件：当前在主机A，将主机B的download.txt文件下载到本机的/usr/local/目录下

    #-r为递归复制，单个文件不需要
    scp -r root@192.168.0.20:/usr/local/download.txt /usr/local/
    #可能要输入密码

- 上传文件：当前在主机A，将主机A的/usr/local/upload.txt文件上传到主机B的/usr/local/目录中

    #-r为递归复制，单个文件不需要
    scp -r /usr/local/upload.txt root@192.168.0.20:/usr/local/

## 参考

- [Linux scp命令](https://zhuanlan.zhihu.com/p/201086165)
