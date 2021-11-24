# Helm

## 概念

chart代表着Heml包，它包含了在Kubernetes集群内部运行的应用程序、工具或服务所需的所有**资源定义**。可以理解为yum的rpm包。

Release是运行在Kubernetes集群中的chart的实例。一个chart通常可以同一个集群中安装多次，每一次安装都会创建一个新的release。

Repository（仓库）是用来存放和共享charts的地方。类似yum的rpm包仓库。

即：**Helm 将 charts 安装到 Kubernetes 集群中，每次安装都会创建一个新的实例（release）**

## 安装

- [安装 Helm](https://helm.sh/zh/docs/intro/install/)

## 语法

### search 搜索

```bash
helm search repo/hub <chart_name>
```

通过search查看chart包，可以在 hub 或者 repo 中查找。  

repo： 一般是自己在本地通过`helm repo add`方式添加的仓库。

hub：是从 [Artifact Hub](https://artifacthub.io/) 中查找并列出 helm charts。Artifact Hub中存放了大量不同的仓库。

### install 安装chart包

```bash
# 用法
$ helm install <release_name> <chart_name>

# 如：安装wordpress
# happy-panda 为 release 的名称
# bitnami/wordpress 为 chart名称
# 如果想让Helm生成一个名称，使用标志：--generate-name。
$ helm install happy-panda bitnami/wordpress [--generate-name]
```

安装过程中，helm会打印一些有用的信息，其中包括：哪些资源已经被创建，release当前的状态，以及是否还需要执行额外的配置操作。

#### 更多安装方法

helm install 支持从多个来源进行安装：

- chart仓库（如上所述）
- 本地chart压缩包：`helm install foo foo-1.0.0.tgz`
- 解压后的chart目录：`helm install foo path/to/foo`
- 完整的URL：`helm install foo https://example.com/charts/foo-1.2.3.tgz`

- 参考[Helm使用](https://helm.sh/zh/docs/intro/using_helm/)

### status 查看状态、读取配置

Helm不会等到所有资源都运行结束才退出。一些chart需要比较大（大于600M）的Docker镜像，可能需要很长的时间才会安装到集群中。
此时可以通过 `helm status` 来追踪 release 的状态，或者重新读取配置信息。

```bash
helm status <release_name>
```

### show values 查看可配置选项

```bash
helm show values <bitnami/chart_name>
```

然后可通过`yaml`格式的配置文件覆盖上面输出的任意配置项，并在安装过程中使用该文件。

```bash
echo '{port: 8181}' > values.yaml
helm install -f values.yaml bitnami/nginx --generate-name
```

上面的命令举例说明修改了nginx的端口，chart中的其他配置保持不变。

安装过程中有两种方式传递配置信息：

- `--values`（或者 -f）：使用yaml文件覆盖配置。**可以指定多次，优先使用最右边的配置文件**。
- `--set`：通过命令行的方式对指定配置项进行覆盖。
  
如果同时使用两种方式，则 --set 中的值会被合并到 --values 中，但是 --set 中的值优先级更高。通过 --set 指定的配置会被保存到ConfigMap中。可以通过 `helm get values <release_name>` 来查看指定 release 中通过 --set 设置的值。也可以通过运行 `helm upgrade` 并指定 `--reset-values` 字段来清除 --set 中设置的值。

#### `-- set` 的格式和限制

`-- set` 选项使用0个或多个 name/value 对。最简单的用法类似于：`--set name=value`。等价于 yaml 格式:`name: value`。多个值使用逗号分隔。`--set a=b,c=d`。支持更复杂的表达式，参考[这里](https://helm.sh/zh/docs/intro/using_helm/)

### upgrade 升级或更新release

通过upgrade可以将chart升级到新的版本，或是修改release的配置。

```bash
# 对release进行升级，使用新的配置文件 new.yaml 和 旧的 chart
helm upgrade -f new.yaml <release_name> <bitnami/old_chart_name> 
```

一次升级操作会使用已有的 release 并根据提供的信息对其进行升级。由于Kubernetes的chart可能会很大而且很复杂，Helm会尝试执行最小侵入式升级。即只会更新自上次发布以来发生了改变的内容。

### get values 查看release配置

```bash
$ helm get values <release_name>
xxport:
  xx:
    xxx: 8181
```

### rollback 回滚

通过rollback可以回滚release到之前的版本

```bash
helm rollback <release_name>  <revision>

# 如：将my-nginx回滚到最初的版本。
# release版本其实是一个增量修订（revision）
# 每当发生了一次安装、升级或者回滚操作，revision的值都会加 1，第一次revision的值永远是 1
# 可以通过 helm history [release_name] 查看一个特定release的修订版本号
helm rollback my-nginx 1
```

### install、upgrade、rollback 有用选项

- `--timeout`: 等待kubernetes命令完成时间，默认值为5m0s
- `--wait`: 表示必须要等到所有的pod都处于ready状态，pvc都被绑定，deployment都至少拥有最小ready状态pods个数（Desired减去 maxUnavailable），并且service都具有ip地址（如果是LoadBlancer，则为ingress），才会标记该release为成功。最长等待时间由 --timeout决定。如果达到超时时间，release将被标记为FAILED。当deployment的replicas被设置为1，但其滚动升级策略中的maxUnavailable 没有被设置为0时，--wait将返回就绪，因为已经满足了最小ready pod数。
- `--no-hooks`: 不运行当前的命令钩子。
- `recrate-pods`: （仅适用于upgrade和rollback），这个参数会导致重建所有pod（deployment的pod除外）。（helm3中废弃）

### uninstall 卸载release

uninstall 命令可以从集群中卸载一个 release

```bash
helm uninstall <release_name>
```

在helm之前，删除一个release之后会保留一条删除记录。而helm3中，删除release也会移除release的记录（**此时无法回滚删除的release**）。如果想保留删除记录，使用: `helm uninstall --keep-history`。使用 `helm list --uninstalled` 只会展示在删除时使用了 `--keep-history` 标记的release。

`helm list --all` 会展示Helm保留的所有release记录，包括失败或删除的条目（指定了 `--keep-history`）。

```bash
$  helm list --all
NAME            VERSION UPDATED                         STATUS          CHART
happy-panda     2       Wed Sep 28 12:47:54 2016        UNINSTALLED     wordpress-10.4.5.6.0
inky-cat        1       Wed Sep 28 12:59:46 2016        DEPLOYED        alpine-0.1.0
kindred-angelf  2       Tue Sep 27 16:16:10 2016        UNINSTALLED     alpine-0.1.0
```

### list 查看所有当前部署的release

```bash
$helm list
NAME            VERSION UPDATED                         STATUS          CHART
my-cat          1       Wed Sep 28 12:59:46 2077        DEPLOYED        alpine-0.1.0
```

### 仓库 (repo)

```shell
helm repo [command]
```

其中可用的command包括：

- add：添加新的chart仓库
- index：生成一个包含chart包的目录的索引文件
- list：列出已经按装的chart实例
- remove：移除一个或多个chart仓库
- update：更新仓库中可用的charts信息到本地

```shell
# 添加仓库
$ helm repo add repo_name https://repo.com
# 更新仓库
$ helm repo update

# 查看仓库列表
$ helm repo list
repo_name    https://repo.com
```
