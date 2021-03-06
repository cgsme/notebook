# Cobra

## 概述

Cobra 既是一个用于创建强大的现代 CLI 应用程序的库，也是一个生成应用程序和命令文件的程序。

Cobra提供：

- 简单的基于子命令的 CLI: `app server`, `app fetch`等等
- 支持完全符合 POSIX（可移植操作系统接口） 的标志（包括短版和长版）
- 全局、本地和级联标志
- 通过 `cobra init appname` 和 `cobra add cmdname` 可以简单地生成应用和命令
- 智能建议（`app srver ...` 是否为 `app server` ?）
- 自动生成命令和标志的 `help`
- 为你的应用程序自动生成 shell 自动完成功能（bash、zsh、fish、powershell）
- 为应用自动生产使用手册
- 命令别名，这样就可以在不破坏它们的情况下更改内容
- 灵活性定义自己的帮助、用法等
- 可选地与[viper](https://github.com/spf13/viper)紧密集成，用于 12 要素应用程序。

### 安装

Cobra 非常易用，首先使用 go get 命令安装最新版本。此命令将安装 cobra generator 的可执行文件及其依赖项：

    go get -u github.com/spf13/cobra/cobra

## 相关概念

Cobra 建立在命令、参数和标志的结构上。

`Commands`代表动作，`Args`是事物，`Flags`是这些动作的修饰符。

最好的应用程序在使用时读起来就像句子，因此，用户直观地知道如何与它们交互。

要遵循的模式是：  
`APPNAME VERB NOUN --ADJECTIVE` 或者 `APPNAME COMMAND ARG --FLAG`  
VERB: 动词  
NOUN：名词  
ADJECTIVE：形容词  
ARG: 参数

在下面的例子中， `server`是一个command，`port`是一个flag:

    hugo server --port=8080

下面这条命令通过git下载bare地址的代码：

    git clone URL --bare

### Commands（命令）

命令是一个应用程序的中心点。每一个应用支持的交互都被包含在一个命令中。一个命令可以有子命令，并可以选择运行一个动作。

### Flags（标志）

标志是一种可以改变命令行为的方式。Cobra支持完全符合 POSIX 的标志，和go flag包一样。Cobra 命令可以定义持续到子命令的标志和仅可用于该命令的标志。

前面的例子中，`port`就是标志。

## 使用Cobra的项目

[使用Cobra的项目](https://github.com/spf13/cobra/blob/master/projects_using_cobra.md)
