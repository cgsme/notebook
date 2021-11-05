# flag

golang 的 flag 包实现了对命令行 flag 的解析。

## 用法

使用 `flag.String()`, `Bool()`, `Int()` 等方式来定义 flags。

比如定义一个整型的 falg， flag 名称为 `-n`，保存在指针变量 `nFlag` 中，变量类型为：`*int`

```go
import "flag"

var nFlag = flag.Int("n", 1233, "-n 的帮助信息")
```

也可以通过 `Var` 函数来定义 falg 到某个变量：

```go
var flagvar int

func init() {
    flag.IntVar(&flagvar, "flag名称", 1234, "flag 的帮助信息")
}
```

或者可以创建满足 `Value` 接口（使用指针接收器）的自定义 flags，并通过以下方式将它们组合起来 ：

```go
// 对于此类 flag， 默认值就是变量的初始值。
flag.Var(&flagVal, "flag_name", "help message for flagname")
```

当所有的 flags 定义完成之后，调用 `flag.Parse()` 将命令行解析为定义的 flag。

这样 flag 就可以直接使用了。如果直接使用这些 flags ，那么它们都是指针，如果将它们绑定给变量，则它们是值。

```go
fmt.Println("ip has value: ", *ip)   // 通过 * 取值
fmt.Println("flagvar has value", flagvar)
```

通过解析之后，跟随在 flags 后面的参数可以通过 `flag.Args()` 来获取，其返回一个切片 slice。或者通过 `flag.Arg(i)` 单独获取。  
索引 `i` 的范围从` 0 - flag.NArg() - 1 `。

## 命令行 flag 语法

支持以下几种语法：

```bash
-flag
-falg=x
-flag x     // 只支持非 boolean 类型的 flag
```

可以使用一个或者两个横杠 `-`，两种方式是等价的。  
最后一种格式不支持 boolean 类型的 flags，因为命令 `cmd -x *` 中的 `*` 是一个Unix Shell 通配符，如果有文件就会改变为 0、false 等。必须使用 `-flag=false`  的形式才可以。

flag 只会在第一个非 flag 参数之前（"-"是一个非 flag 参数）或者终结者 "--" 之后停止。

整型 flags 接受 1234、0665、0x1234 或者负数。

布尔类型的 flags 可以是：1，0，f，t，T，F，true，false，TRUE，FALSE，True，False。

时间类型 flags 可以是任何合法的 `time.ParseDuration` 输入。

默认的命令行标志集由顶级函数控制。 FlagSet 类型允许定义独立的标志集，例如在命令行界面中实现子命令。 FlagSet 的方法类似于命令行标志集的顶级函数。

## Example

> 官方 <https://github.com/golang/go/blob/master/src/flag/example_test.go>

```go

// These examples demonstrate more intricate uses of the flag package.
package flag_test

import (
    "errors"
    "flag"
    "fmt"
    "strings"
    "time"
)

// Example 1: A single string flag called "species" with default value "gopher".
var species = flag.String("species", "gopher", "the species we are studying")

// Example 2: Two flags sharing a variable, so we can have a shorthand.
// The order of initialization is undefined, so make sure both use the
// same default value. They must be set up with an init function.
var gopherType string

func init() {
    const (
        defaultGopher = "pocket"
        usage         = "the variety of gopher"
    )
    flag.StringVar(&gopherType, "gopher_type", defaultGopher, usage)
    flag.StringVar(&gopherType, "g", defaultGopher, usage+" (shorthand)")
}

// Example 3: A user-defined flag type, a slice of durations.
type interval []time.Duration

// String is the method to format the flag's value, part of the flag.Value interface.
// The String method's output will be used in diagnostics.
func (i *interval) String() string {
    return fmt.Sprint(*i)
}

// Set is the method to set the flag value, part of the flag.Value interface.
// Set's argument is a string to be parsed to set the flag.
// It's a comma-separated list, so we split it.
func (i *interval) Set(value string) error {
    // If we wanted to allow the flag to be set multiple times,
    // accumulating values, we would delete this if statement.
    // That would permit usages such as
    //  -deltaT 10s -deltaT 15s
    // and other combinations.
    if len(*i) > 0 {
        return errors.New("interval flag already set")
    }
    for _, dt := range strings.Split(value, ",") {
        duration, err := time.ParseDuration(dt)
        if err != nil {
            return err
        }
        *i = append(*i, duration)
    }
    return nil
}

// Define a flag to accumulate durations. Because it has a special type,
// we need to use the Var function and therefore create the flag during
// init.

var intervalFlag interval

func init() {
    // Tie the command-line flag to the intervalFlag variable and
    // set a usage message.
    flag.Var(&intervalFlag, "deltaT", "comma-separated list of intervals to use between events")
}

func Example() {
    // All the interesting pieces are with the variables declared above, but
    // to enable the flag package to see the flags defined there, one must
    // execute, typically at the start of main (not init!):
    flag.Parse()
    // We don't run it here because this is not a main function and
    // the testing suite has already parsed the flags.
}

```

## 参考

- [golang flag 源码](https://github.com/golang/go/blob/master/src/flag/flag.go)
