# history命令

## 语法

    # 展示所有历史命令
    [root@localhost ~]# histroy

    # 执行历史命令
    # history会列出bash保存的所有历史命令，并且给它们编了号，我们可以使用“叹号接编号”的方式运行特定的历史
    !+字母：调出最近使用一次以此字母开头的命令
    !!: 表示最近使用的一次操作的命令
    !+数字：表示调出历史记录的几条命令
    !-数字：调出最近的几条命令

## 常用参数

    c：清空当前历史命令；
    a：将历史命令缓冲区中命令写入历史命令文件【/root/.bash_history】
    r：将历史命令文件中的命令读入当前历史命令缓冲区
    w：将当前历史命令缓冲区命令写入历史命令文件中【/root/.bash_history】
    n：打印最近的n条历史命令

## 格式化history命令的输出格式

- 显示执行时间【默认只显示序号和命令】

    #临时生效
    [root@localhost ~]# export HISTTIMEFORMAT='%F %T'
    [root@localhost ~]# echo $HISTTIMEFORMAT
    [root@localhost ~]# history 2

    #永久生效
    #修改/root/.bash_history  添加export HISTTIMEFORMAT='%F %T'
    [root@localhost ~]# vim /root/.bash_history
    [root@localhost ~]# source /root/.bashrc    # 命令生效

## 示例

举例：

- 查看history的缓存数量

        #直接查看变量
        [root@localhost ~]# echo $HISTSIZ
        #查看文件内容。注意：HISTSIZE=0，表示禁用history
        [root@localhost ~]# cat /etc/profile | grep 'HISTSIZE'

        #查看最近10条命令
        [root@localhost ~]# history 10

- 使用最近一次使用的命令

        [root@localhost ~]# !!

- 使用第Num条命令

        [root@localhost ~]# !994

- 显示最近使用的某个特定命令

        [root@localhost ~]# !echo

- 获得上一条命令的参数

        [root@localhost ~]# ls sun.txt
        #这里的!$就是上条命令的参数
        [root@localhost ~]# cat !$

- 清空历史命令

        #清空本次会话的历史命令
        [root@localhost ~]# history -c
        #清空所有的历史命令（覆盖）
        [root@localhost ~]# history -w
