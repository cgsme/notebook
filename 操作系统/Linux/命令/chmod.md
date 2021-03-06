# chmod 命令

改变文件或目录的权限。有两种方式：

- 使用数字方式修改

```txt
r:4 读
w:2 写
x:1 执行
```

- 使用符号方式修改

```txt
r、w、x
```

## chmod 语法

- 数字方式

```shell
#ddd代表三个数字，如777=rwxrwxrwx 600=rw-------
chmod [-R] ddd 文件或目录
```

- 符号方式

> chmod | u（所有者）/ g（群组）/ o（其他用户）/ a（所有，默认） | +(加入) / -(除去) / =(设置) | r / w / x | 文件或目录 

```shell
#给文件或目录增加读权限
chmod +r 文件或目录

#给文件或目录移除写权限
chmod -w 文件或目录

#给文件或目录移除可执行权限
chmod -x 文件或目录

#假如要个一个文件的权限改为[-rwxr-xr-x]，
#即 user(u): 具有rwx，可读、可写、可执行
#   group和other（g/o）: 具有可读、可执行权限。没有写权限
chmod u=rwx,go=rx /opt/test.txt
```

### 参数说明

```txt
-R: 递归修改，文件夹下的文件和文件夹都会被修改。
```

## 示例

```shell
# 数字方式，修改 /opt/test.txt 文件的权限为：所有者具有读写权限，群组和其他用户任何无权限
chmod 600 /opt/test.txt

# 符号方式，修改 /opt/test.txt 文件权限为：所有者、群组、其他用户都增加写权限
chmod +w /opt/test.txt     # 相当于 chmod a+w /opt/test.txt
```
