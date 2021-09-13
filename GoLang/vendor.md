# vendor

## 使用vendor进行编译

`
    在编译时，可以使用 -mod=vendor 标记，使用代码主目录文件夹下vendor目录满足依赖获取，go build -mod=vendor。此时，go build 忽略go.mod 中的依赖，（这里仅使用代码root目录下的vendor其他地方的将忽略）

    GOFLAGS=-mod=vendor 设置顶级vendor作为依赖： 
    开启 go env -w GOFLAGS="-mod=vendor" 
    取消 go env -w GOFLAGS="-mod="
`
