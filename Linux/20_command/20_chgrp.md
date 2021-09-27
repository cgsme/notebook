# chgrp 命令

修改文件的群组。（change group）

## chgrp 语法

    chgrp [-R] groupname dirname/filename

### 参数说明

    -R: 进行递归修改。便利目录下的所有文件、目录，都更新群主。

## 示例

    # 将/opt/test.txt 文件的所属组改为 usersgroup。  若不存在usergroup群组，则会报错。
    chgrp usersgroup /opt/test.txt   

### 查看

    ls -l
