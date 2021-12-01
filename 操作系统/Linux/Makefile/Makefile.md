# Makefile

Makefile即Make file，就是“制作文件”的意思。Makefile本身是一个文件，文件中包含一些列的操作规则，旨在通过一系列的shell命令来“制作”想要的文件。

即：通过Makefile文件来Make file。

## Makefile 文件格式

Makefile文件中包含一系列的规则（rules）。每条规则的格式如下：

```bash
<target>: <prerequisites>
[tab] <commands>
```

### target 目标

目标。通常就是要“制作”的文件的名称。可以是一个或者多个文件名，通过逗号分隔。

目标也可以是一个操作的名称，称为“伪目标”（phony target）。但是，如果当前操作的目录中存在一个与伪目标的名称相同的文件，那么这个伪目标对应的命令将不会执行。因为make命令发现目标文件已经存在，就不需要重新创建了，也就不会执行伪目标对应的命令了。

但是可以通过`.PHONY`来声明伪目标，从而解决伪目标与文件同名的情况：

```bash
# 声明了 show 为伪目标之后，make命令就不会去检查当前目录是否存在一个名为 show 的文件了。 
.PHONY show

show:
    ls -l
```

### prerequisites 前置条件

前置条件是用来“制造”文件的前提条件。通常是一系列的文件名，若这些文件不存在则需要先制造这些文件，存在则执行制造目标文件的命令。若前置条件未发生变化，则不执行当前操作。

```bash
bbb.txt: aaa.txt
    cp aaa.txt bbb.txt
```

上面这个例子中，若aaa.txt文件存在，则`make bbb.txt`命令可正常执行。若aaa.txt不存在，则不会执行这条命令，需要额外的规则先创建aaa.txt。

```bash
# 这里没有依赖任何文件，只要aaa.txt不存在就可以正常执行
aaa.txt:
    echo "this is file aaa" aaa.txt
```

此时，执行`make bbb.txt`，会先创建 bbb.txt 文件，再创建aaa.txt。若再次执行一次相同的命令`make bbb.txt`

### command 命令

每行命令在一个单独的shell中执行。这些Shell之间没有继承关系。若命令需要跨行的话可以再行尾使用反斜杠 `\`，行尾未使用反斜杠的命令将被视作多条命令分别单独执行。

```bash
var-lost:
    export foo=bar
    echo "foo=[$$foo]"   # 无法取到foo的值，因为两行命令不存在上下文关系

var-kept:
    export foo=bar; echo "foo=[$$foo]"  # 可取到foo的值，通过分号相隔

var-kept:
    export foo=bar; \
    echo "foo=[$$foo]"  # 可取到foo的值，因为行尾使用了反斜杠

    
.ONESHELL:
var-kept:
    export foo=bar; 
    echo "foo=[$$foo]"   # 可取到foo的值，通过 .ONESHELL: 标记

```

## Makefile 语法

### 注释

 `#`:表示注释

### 回声（echoing）

正常make会打印每条命令，然后再执行。就是回声。

在命令的前面加上@，就可以关闭回声。

```bash
test:
    @# 测试关闭回声
```

在make执行过程中，我要知道当前在执行哪条命令，所以通常只在注释和纯显示的echo命令前面加上@。

```bash
test:
    @# 这是测试
    @echo TODO
```

### 通配符

主要有星号（`*`）、问号（`？`）和 `[...]` 。比如， `*.o` 表示所有后缀名为o的文件。

### 模式匹配

匹配符% : 进行类似正则表达式运算的匹配。比如，当前目录下有 f1.c 和 f2.c 两个源码文件，将它们编译为对应的对象文件。

```bash
%.o: %.c
```

等同于下面的写法。

```bash
f1.o: f1.c

f2.o: f2.c
```

使用匹配符%，可以将大量同类型的文件，只用一条规则就完成构建。

### 变量和赋值符

使用等号自定义变量。

```bash
txt = Hello World
test:
    @echo $(txt)
```

上面声明了变量txt，使用时需要放在`$()`中。

调用Shell变量，需要在美元符号前，再加一个美元符号，这是因为Make命令会对美元符号转义。

```bash
test:
    @echo $$HOME
```

## 参考

- [Make 命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)