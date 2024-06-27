# alias

别名。用于设置命令的别名。

用户可以利用 `alias`，自定义指令的别名。若只输入 `alias`，则可以列出目前所有的别名设置。`alias` 的作用仅限于当前登录的会话。若要每次登录都生效，可以在 `.prifile` 或者 `.cshrc` 中设置命令的别名。

## 语法

    alias[别名]=[命令名称]

## 示例

    $ alias lx=ls
    $ lx

    /root /home ...
