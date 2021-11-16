# os-release.md

## 文件说明

    /etc/os-release
    /usr/lib/os-release

这两个文件包含了操作系统的身份数据，用来标识操作系统的版本和名称。

## 查看操作系统名称和版本

有以下方式查看当前操作系统名称和版本号：

    cat /etc/os-release
    lsb_release -a
    hostnamectl

## 查看系统内核版本

    uname -r

## 参考

官方文档：

- [os-release](https://www.linux.org/docs/man5/os-release.html)
