# go mod

## GO111MODULE

> - mode，即无论要构建的源码目录是否在GOPATH路径下，go compiler都会在传统的GOPATH和vendor目录(仅支持在gopath目录下的 package)下搜索目标程序依赖的go package；
> - 当GO111MODULE的值为on时（export GO111MODULE=on），go modules experiment feature始终开启，与off相反，go compiler会始终使用module-aware mode，即无论要构建的源码目录是否在GOPATH路径下，go compiler都不会在传统的GOPATH和vendor目录下搜索目标程序依赖的go package，而是在go mod命令的缓存目录($GOPATH/pkg/mod）下搜索对应版本的依赖package；
> - 当GO111MODULE的值为auto时(不显式设置即为auto)，也就是我们在上面的例子中所展现的那样：使用GOPATH mode还是module-aware mode，取决于要构建的源码目录所在位置以及是否包含go.mod文件。如果要构建的源码目录不在以GOPATH/src为根的目录体系下，且包含go.mod文件(**两个条件缺一不可**)，那么使用module-aware mode；否则使用传统的GOPATH mode

## 参考

- [GO的包管理发展及go mod应用](https://zhuanlan.zhihu.com/p/311969770)
- [初窥Go Module](https://tonybai.com/2018/07/15/hello-go-module/)
