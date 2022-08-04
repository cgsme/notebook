# Cobra 使用指南

> [官方指南](https://github.com/spf13/cobra/blob/master/user_guide.md)

虽然允许根据自己需求调整结构，但通常基于 Cobra 的应用程序将遵循以下组织结构：

    ▾ appName/
    ▾ cmd/
        add.go
        your.go
        commands.go
        here.go
      main.go

在一个Cobra应用中，通常 `main.go` 文件是非常空的。 它有一个目的：初始化 Cobra。

    package main

    import (
        "{pathToYourApp}/cmd"
    )

    func main() {
        cmd.Execute()
    }

## 使用Cobra生成器

Cobra 提供了自己的程序，可以创建您的应用程序并添加您想要的任何命令。这是将 Cobra 整合到应用中的最简单方法。

[参考](https://github.com/spf13/cobra/blob/master/cobra/README.md)更多信息。

## 使用Cobra库

为了手动实现Cobra，需要创建一个空的`main.go`文件和一个`rootCmd`文件。可以选择性地提供额外的合适的命令。

### 创建 rootCmd

Cobra不需要任何特殊的构造器。可以简单地创建命令。

理想情况下，app/cmd/root.go 中代码如下：

    var rootCmd = &cobra.Command{
        Use:   "hugo",
        Short: "Hugo is a very fast static site generator",
        Long: `A Fast and Flexible Static Site Generator built with
                        love by spf13 and friends in Go.
                        Complete documentation is available at http://hugo.spf13.com`,
        Run: func(cmd *cobra.Command, args []string) {
            // Do Stuff Here
        },
    }

    func Execute() {
    if err := rootCmd.Execute(); err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}

另外可以在`init()`函数中定义`标志`和处理配置。

例如，cmd/root.go：

    package cmd

    import (
        "fmt"
        "os"

        "github.com/spf13/cobra"
        "github.com/spf13/viper"
    )

    var (
        // Used for flags.
        cfgFile     string
        userLicense string

        rootCmd = &cobra.Command{
            Use:   "cobra",
            Short: "A generator for Cobra based Applications",
            Long: `Cobra is a CLI library for Go that empowers applications.
    This application is a tool to generate the needed files
    to quickly create a Cobra application.`,
        }
    )

    // Execute executes the root command.
    func Execute() error {
        return rootCmd.Execute()
    }

    func init() {
        cobra.OnInitialize(initConfig)

        rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra.yaml)")
        rootCmd.PersistentFlags().StringP("author", "a", "YOUR NAME", "author name for copyright attribution")
        rootCmd.PersistentFlags().StringVarP(&userLicense, "license", "l", "", "name of license for the project")
        rootCmd.PersistentFlags().Bool("viper", true, "use Viper for configuration")
        viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
        viper.BindPFlag("useViper", rootCmd.PersistentFlags().Lookup("viper"))
        viper.SetDefault("author", "NAME HERE <EMAIL ADDRESS>")
        viper.SetDefault("license", "apache")

        rootCmd.AddCommand(addCmd)
        rootCmd.AddCommand(initCmd)
    }

    func initConfig() {
        if cfgFile != "" {
            // Use config file from the flag.
            viper.SetConfigFile(cfgFile)
        } else {
            // Find home directory.
            home, err := os.UserHomeDir()
            cobra.CheckErr(err)

            // Search config in home directory with name ".cobra" (without extension).
            viper.AddConfigPath(home)
            viper.SetConfigType("yaml")
            viper.SetConfigName(".cobra")
        }

        viper.AutomaticEnv()

        if err := viper.ReadInConfig(); err == nil {
            fmt.Println("Using config file:", viper.ConfigFileUsed())
        }
    }

### 创建 main.go

有了rootCmd，还需要mian函数去执行它的`Excute`函数。为清楚起见，Execute函数写在rootCmd上比较合适，虽然它可以在任何命令上调用。

在Cobra应用中，通常main.go文件都是很空的，只有一个目的：初始化Cobra。

    package main

    import (
        "{pathToYourApp}/cmd"
    )

    func main() {
         cmd.Execute()
    }

### 创建额外的命令

也可以创建新的命令，通常在`cmd/`目录下为每个新命令提供单独的源文件。

比如，若想创建一个version命令，可以创建cmd/version.go文件，内容如下：

    package cmd

    import (
        "fmt"

        "github.com/spf13/cobra"
    )

    func init() {
        rootCmd.AddCommand(versionCmd)
    }

    var versionCmd = &cobra.Command{
        Use:   "version",
        Short: "Print the version number of Hugo",
        Long:  `All software has versions. This is Hugo's`,
        Run: func(cmd *cobra.Command, args []string) {
            fmt.Println("Hugo Static Site Generator v0.9 -- HEAD")
        },
    }

### 返回并处理错误

可以使用`RunE`来给使用命令的用户返回错误：

    package cmd

    import (
        "fmt"

        "github.com/spf13/cobra"
    )

    func init() {
        rootCmd.AddCommand(tryCmd)
    }

    var tryCmd = &cobra.Command{
        Use:   "try",
        Short: "Try and possibly fail at something",
        RunE: func(cmd *cobra.Command, args []string) error {
            if err := someFunc(); err != nil {
                return err
            }
            return nil
        },
    }

通过`RunE`，错误就能在运行时被捕获。

### 使用标志（Flags）

标志给命令执行提供修饰符。

#### 给命令定义标志

由于标志是在不同位置定义和使用的，我们需要在外部定义一个具有正确范围的变量来分配要使用的标志。

    var Verbose bool
    var Source string

有两种不同的方式来定义标志。

#### Persistent Flags（持久标志）

标志可以是`persistent（持续）`的，这意味着该标志将可用于**分配给它的命令**以及**该命令下的每个命令**。对于全局标志，可以在根命令上分配一个标志作为持久标志。

    rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")

#### Local Flags（本地标志）

也可以在本地分配标志，该标志仅适用于该特定命令。

    localCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")

#### 父命令上的Local Flags

默认情况下，Cobra只在目标命令上解析本地标志， 并且任何本地标志在父命令中都将被忽略。开启`Command.TraverseChildren`，Cobra将会在执行目标命令之前在每一个命令上解析本地标志。

    command := cobra.Command{
        Use: "print [OPTIONS] [COMMANDS]",
        TraverseChildren: true,
    }

#### 通过配置绑定标志（Bind Flags with Config）

可以通过[viper](https://github.com/spf13/viper)绑定标志：

    var author string

    func init() {
        rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
        viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
    }

在这个例子中，持久标志`author`和`viper`绑定了。注意：当用户未提供 `--author` 标志时，变量 `author` 将不会设置为配置中的值。

[viper文档](https://github.com/spf13/viper#working-with-flags)

#### Required flags（必要标志）

默认情况下，标志是可选的。如果希望当标志没有设置的时候报告一个错误，可以将标志标记为**必须**：

    rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
    rootCmd.MarkFlagRequired("region")

或者，对于持久标志：

    rootCmd.PersistentFlags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
    rootCmd.MarkPersistentFlagRequired("region")

### 位置和自定义参数

可以使用 Command 的 Args 字段指定位置参数的验证。

内置了以下验证器：

- `NoArgs` - 如果有任何位置参数，该命令将报告错误。
- `ArbitraryArgs` - 命令可以接收任意参数。
- `OnlyValidArgs` - 命令只允许Command中的`ValidArgs`字段中的参数，否则将会报错。
- `MinimumNArgs(int)` - 命令最少需要N个参数，否则报错。
- `MaximumNArgs(int)` - 命令最多允许N个参数，否则报错。
- `ExactArgs(int)` - 命令必须有N个参数，否则报错。
- `ExactValidArgs(int)` - 命令必须有N个参数或者参数必须在`ValidArgs`字段中声明。
- `RangeArgs(min, max)` - 命令参数个数必须在min-max范围内。

设置自定义验证器的例子：

    var cmd = &cobra.Command{
        Short: "hello",
        Args: func(cmd *cobra.Command, args []string) error {
            if len(args) < 1 {
                return errors.New("requires a color argument")
            }
            if myapp.IsValidColor(args[0]) {
                return nil
            }
            return fmt.Errorf("invalid color specified: %s", args[0])
        },
        Run: func(cmd *cobra.Command, args []string) {
            fmt.Println("Hello, World!")
        },
    }

### 案例

下面的例子中，定义了三个命令。两个是顶层命令，另一个是其中一个顶层命令的子命令。  
在这个案例中，root是不可执行的（不给`rootCmd`命令提供`Run`），意味着必须有一个子命令。

我们只给每个命令定义一个标志（flag）。

更多标志相关的文档，参考[pflag](https://github.com/spf13/pflag)

    package main

    import (
    "fmt"
    "strings"

    "github.com/spf13/cobra"
    )

    func main() {
    var echoTimes int

    var cmdPrint = &cobra.Command{
        Use:   "print [string to print]",
        Short: "Print anything to the screen",
        Long: `print is for printing anything back to the screen.
    For many years people have printed back to the screen.`,
        Args: cobra.MinimumNArgs(1),
        Run: func(cmd *cobra.Command, args []string) {
        fmt.Println("Print: " + strings.Join(args, " "))
        },
    }

    var cmdEcho = &cobra.Command{
        Use:   "echo [string to echo]",
        Short: "Echo anything to the screen",
        Long: `echo is for echoing anything back.
    Echo works a lot like print, except it has a child command.`,
        Args: cobra.MinimumNArgs(1),
        Run: func(cmd *cobra.Command, args []string) {
        fmt.Println("Echo: " + strings.Join(args, " "))
        },
    }

    var cmdTimes = &cobra.Command{
        Use:   "times [string to echo]",
        Short: "Echo anything to the screen more times",
        Long: `echo things multiple times back to the user by providing
    a count and a string.`,
        Args: cobra.MinimumNArgs(1),
        Run: func(cmd *cobra.Command, args []string) {
        for i := 0; i < echoTimes; i++ {
            fmt.Println("Echo: " + strings.Join(args, " "))
        }
        },
    }

    cmdTimes.Flags().IntVarP(&echoTimes, "times", "t", 1, "times to echo the input")

    var rootCmd = &cobra.Command{Use: "app"}
    rootCmd.AddCommand(cmdPrint, cmdEcho)
    cmdEcho.AddCommand(cmdTimes)
    rootCmd.Execute()
    }

更多完整的大型应用案例，参考[Hugo](http://gohugo.io/)

### 帮助命令（Help Command）

当有子命令的时候，Cobra会为应用自动添加帮助命令。当用户执行`app help`时会调用help命令。另外，帮助命令支持所有的命令作为输入。比如说，有个一个名为`create`的没有额外配置的命令，当执行`app help create`时，Cobra将会开始工作。每个命令都会自动被添加`--help`标志。

#### 帮助命令案例

下面的输出时Cobra自动生成的。除了命令和标志定义之外，什么都不需要。

    $ cobra help

    Cobra is a CLI library for Go that empowers applications.
    This application is a tool to generate the needed files
    to quickly create a Cobra application.

    Usage:
    cobra [command]

    Available Commands:
    add         Add a command to a Cobra Application
    help        Help about any command
    init        Initialize a Cobra Application

    Flags:
    -a, --author string    author name for copyright attribution (default "YOUR NAME")
        --config string    config file (default is $HOME/.cobra.yaml)
    -h, --help             help for cobra
    -l, --license string   name of license for the project
        --viper            use Viper for configuration (default true)

    Use "cobra [command] --help" for more information about a command.

`Help`也是一个普通的命令，没有特殊的逻辑。实际上也可以创建自己的Help命令。

#### 定义自己的help命令

可以通过下面的函数给默认的命令提供自己的帮助命令或者模板：

    cmd.SetHelpCommand(cmd *Command)
    cmd.SetHelpFunc(f func(*Command, []string))
    cmd.SetHelpTemplate(s string)

通过后面两个函数也会应用于任何子命令。

### 用法信息（Usage Message）

当用户提供了不合法的标志或命令，Cobra会给用户展示用法信息。

#### 例子

可以从前面的帮助中认识到这点。因为默认的帮助会嵌入到用法的一部分中。

    ```shell
    $ cobra --invalid
    Error: unknown flag: --invalid
    Usage:
    cobra [command]

    Available Commands:
    add         Add a command to a Cobra Application
    help        Help about any command
    init        Initialize a Cobra Application

    Flags:
    -a, --author string    author name for copyright attribution (default "YOUR NAME")
        --config string    config file (default is $HOME/.cobra.yaml)
    -h, --help             help for cobra
    -l, --license string   name of license for the project
        --viper            use Viper for configuration (default true)

    Use "cobra [command] --help" for more information about a command.
    ```

#### 定义自己的用法信息

可以提供自己的用法处理函数或者模板给Cobra使用。比如帮助，通过公共方法可以覆盖用法函数和模板：

    cmd.SetUsageFunc(f func(*Command) error)
    cmd.SetUsageTemplate(s string)

### 版本标记

如果Version字段在rootCmd中设置了，那么Cobra会添加一个顶层的`--version`标志。运行应用时使用`--version`标志将会使用版本模板打印版本信息到标准输出。版本模板可以通过`cmd.SetVersionTemplate(s string)`函数自定义。

### 执行前和执行后的钩子

可以在mian函数执行前或执行后运行自定义的函数。`PersistentPreRun`和`PreRun`函数在`Run`之前将会被执行。`PersistentPostRun`和`PostRun`在`Run`之后将会被执行。`Persistent*Run`将会被子命令继承（如果子命令定义了自己的`Persistent*Run`，则将覆盖父命令的。）。这些函数的运行顺序如下：

- `PersistentPreRun`
- `PreRun`
- `Run`
- `PostRun`
- `PersistentPostRun`

下面是一个拥有所有这些功能的两个命令的例子。当子命令被执行，将会执行root命令的`PersistentPreRun`函数，而不是root命令的`PersistentPostRun`函数：

    package main

    import (
        "fmt"

        "github.com/spf13/cobra"
    )

    func main() {

        var rootCmd = &cobra.Command{
            Use:   "root [sub]",
            Short: "My root command",
            PersistentPreRun: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside rootCmd PersistentPreRun with args: %v\n", args)
            },
            PreRun: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside rootCmd PreRun with args: %v\n", args)
            },
            Run: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside rootCmd Run with args: %v\n", args)
            },
            PostRun: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside rootCmd PostRun with args: %v\n", args)
            },
            PersistentPostRun: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside rootCmd PersistentPostRun with args: %v\n", args)
            },
        }

        var subCmd = &cobra.Command{
            Use:   "sub [no options!]",
            Short: "My subcommand",
            PreRun: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside subCmd PreRun with args: %v\n", args)
            },
            Run: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside subCmd Run with args: %v\n", args)
            },
            PostRun: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside subCmd PostRun with args: %v\n", args)
            },
            PersistentPostRun: func(cmd *cobra.Command, args []string) {
                fmt.Printf("Inside subCmd PersistentPostRun with args: %v\n", args)
            },
        }

        rootCmd.AddCommand(subCmd)

        rootCmd.SetArgs([]string{""})
        rootCmd.Execute()
        fmt.Println()
        rootCmd.SetArgs([]string{"sub", "arg1", "arg2"})
        rootCmd.Execute()
    }

输出：

    Inside rootCmd PersistentPreRun with args: []
    Inside rootCmd PreRun with args: []
    Inside rootCmd Run with args: []
    Inside rootCmd PostRun with args: []
    Inside rootCmd PersistentPostRun with args: []

    Inside rootCmd PersistentPreRun with args: [arg1 arg2]
    Inside subCmd PreRun with args: [arg1 arg2]
    Inside subCmd Run with args: [arg1 arg2]
    Inside subCmd PostRun with args: [arg1 arg2]
    Inside subCmd PersistentPostRun with args: [arg1 arg2]

### 当出现`unkonwn command`时的建议

当“unkonwn command”错误出现时，Cobra会自动打印建议。当打字出现错误时，Cobra会有和`git`类似的行为，比如：

    $ hugo srever
    Error: unknown command "srever" for "hugo"

    Did you mean this?
            server

    Run 'hugo --help' for usage.

建议是基于每个注册的子命令自动提出的，并使用[Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance)(字符串相似度算法)的实现。每个匹配最小距离 2（忽略大小写）的已注册命令都将显示为建议。

如果想要禁用建议或者改变字符距离配置，可以使用：

    // 禁用
    command.DisableSuggestions = true

或者

    // 改变距离
    command.SuggestionsMinimumDistance = 1

还可以使用`SuggestFor`属性显式地为需要建议的命令设置名称。这允许为在字符串距离方面不接近的字符串提供建议，但在您的命令集和一些您不想要别名的命令中有意义。

比如：

    $ kubectl remove
    Error: unknown command "remove" for "kubectl"

    Did you mean this?
            delete

    Run 'kubectl help' for usage.

### 为命令生成文档

Cobra可以基于子命令、标志等生成文档。更多关于文档生成的信息，参阅[docs generation documentation](https://github.com/spf13/cobra/blob/master/doc/README.md)。

### shell自动补全

Cobra可以给以下shell生成自动补全：bash，zsh，fish，PowerShell。如果给命令添加更多的信息，自动补全将会非常强大和灵活。更多关于自动补全，参考[Shell Completions](https://github.com/spf13/cobra/blob/master/shell_completions.md)。
