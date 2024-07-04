# 学会控制Nginx

nginx可以被 signal 控制。默认情况下，`master`进程的进程ID会写入 `/usr/local/nginx/logs/nginx.pid` 文件中。pid 文件可以在配置时被改变，或者在 `nginx.conf` 文件中通过 pid 指令配置。`master`进程支持以下 signal：

    TERM, INT   快速停止
    QUIT        优雅停止
    HUP         更改配置、跟上更改的时区（仅适用于 FreeBSD 和 Linux）、使用新配置启动新worker进程、正常关闭旧worker进程
    USR1        重新打开日志文件
    USR2        更新一个可执行文件
    WINCH       优雅停止worker进程

个别`worker`进程也可以被singal控制，支持的signal如下：

    TERM, INT   快速停止
    QUIT        优雅停止
    USR1        重新打开日志文件
    WINCH       调试异常终止（需要启用 debug_points）

## 轮换日志（滚动日志）

为了滚动日志文件，需要首先重命名日志文件。然后 `USR1` 信号应该发送到 `master` 进程。然后 `master` 进程将重新打开当前所有已打开的日志文件，并为它们分配一个非特权用户，  `worker` 进程在该用户下作为所有者运行。成功重新打开日志文件后，`master` 进程关闭所有打开的文件，并向 `worker` 进程发送消息，要求它们重新打开文件。`worker` 进程打开新文件并立即关闭旧文件。因此，旧文件可以被立即用于后期处理，例如压缩。

## 即时升级可执行文件

为了升级服务器的可执行文件，应首先将新的可执行文件替换旧文件。之后 `USR2` 信号应发送到 `master` 进程。`master` 进程首先将其带有进程 ID 的文件重命名为带有 `.oldbin` 后缀的新文件，例如 `/usr/local/nginx/logs/nginx.pid.oldbin`，然后启动一个新的可执行文件，该文件又启动新的工作进程：

      PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
    33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
    33135 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
    33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
    36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)

之后，所有 `worker` 进程（旧的和新的）继续接受请求。如果 `WINCH` 信号发送到第一个 `master` 进程，它将向其 `worker` 进程发送消息，请求它们正常关闭，然后它们将开始退出：

      PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
    33135 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
    36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)

一段时间后，只有新的工作进程才会处理请求：

    PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)

注意，旧的 `master` 进程不会关闭其监听的socket，如果有需要，可以重新启动其 `worker` 进程。如果由于某种原因新的可执行文件无法正常工作，可以执行以下操作之一：

- 向旧的 `master` 进程发送 `HUP` 信号。旧的 `master` 进程将启动新的 `worker` 进程，而无需重新读取配置。之后，通过向新的 `master` 进程发送 `QUIT` 信号，可以正常关闭所有新进程。
- 向新的 `master` 进程发送 `TERM` 信号。然后它会向其 `worker` 进程发送一条消息，要求它们立即退出，它们基本上会立即退出。（如果新进程由于某种原因没有退出，则应向它们发送 `KILL` 信号以强制它们退出。）当新的 `master` 进程退出时，旧的 `master` 进程会自动启动新的 `worker` 进程。

如果新的 `master` 进程退出，则旧的 `master` 进程将丢弃带有进程 ID 的文件名中的 .oldbin 后缀。

如果可执行文件升级成功，则应将 `QUIT` 信号发送到旧的 `master` 进程，并且只留下新进程：

    PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    36264     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
