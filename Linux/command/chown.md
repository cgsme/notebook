# chown 命令

修改文件的所有者（change owner）。使用者必须是已经存在系统中的账号，也就是在/etc/passwd 文件中有记录的使用者。

## chown 语法

    # 1、只修改文件或目录所有者
    chown [-R] 用户名称 文件或目录

    # 2、修改文件或目录所有者及群组
    chown [-R] 用户名称:群组名称 文件或目录

### 参数说明

    -R: 递归修改，如果修改的目录下还有文件或目录，则会一起修改。

## 示例

    # 修改 /opt/test.txt 文件的所有者为 user01
    chown user01 /opt/test.txt

    # 将 /opt/test.txt 文件的所有者修改为root用户，以及将群组也修改为root群组
    chown root:root /opt/test.txt

### 查看

    ls -l
