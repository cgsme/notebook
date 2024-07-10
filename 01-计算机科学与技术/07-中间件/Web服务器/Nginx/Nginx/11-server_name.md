# 服务器名称（server_name）

## server_name

服务器名称由 `server_name` 指令指定，并确定哪个 `server` 块用于处理给定的请求。`server_name` 可以使用精确名称、通配符名称或正则表达式来定义：

```conf
server {
    listen       80;
    server_name  example.org  www.example.org;
    ...
}

server {
    listen       80;
    server_name  *.example.org;
    ...
}

server {
    listen       80;
    server_name  mail.*;
    ...
}

server {
    listen       80;
    server_name  ~^(?<user>.+)\.example\.net$;
    ...
}
```

按名称搜索虚拟服务器时，如果名称与多个指定变体匹配，例如通配符名称和正则表达式都匹配，将按照以下优先顺序选择第一个匹配的变体：

1. 具体的服务器名称。
2. 以星号开头的最长通配符名称，例如 `*.example.org`
3. 以星号结尾的最长通配符名称，例如 `mail.*`
4. 第一个匹配的正则表达式（按照配置文件中出现的顺序）

## 通配符名称

通配符名称只能在名称的开头或结尾以及点边框上包含星号。名称 `www.*.example.org` 和 `w*.example.org` 无效。但是可以使用正则表达式指定这些名称，例如 `~^www\..+\.example\.org$` 和`~^w.*\.example\.org$`。一个星号可以匹配多个名称部分。名称 `*.example.org` 不仅匹配 `www.example.org`，还匹配 `www.sub.example.org`。

`.example.org` 形式的特殊通配符名称可用于匹配确切名称 `example.org` 和通配符名称 `*.example.org`。

## 正则表达式名称

nginx 使用的正则表达式与 Perl 编程语言 (PCRE) 使用的正则表达式兼容。要使用正则表达式，服务器名称必须以**波浪号**字符开头，否则，它将被视为精确名称，或者如果表达式包含星号，则被视为通配符名称（并且很可能被视为无效名称）：

```conf
server_name  ~^www\d+\.example\.net$;   # 正则表达式 服务器名称必须以**波浪号**字符开头
```

不要忘记设置 `^` 和 `$` 锚点。它们不是语法上需要的，而是逻辑上需要的。另请注意，域名中的点应使用反斜杠进行转义。包含字符 `{` 和 `}` 的正则表达式应加引号：

```conf
server_name  "~^(?<name>\w\d{1,3}+)\.example\.net$";
```

否则nginx将无法启动并显示错误信息：

```text
directive "server_name" is not terminated by ";" in ...
```

命名的正则表达式捕获稍后可以用作变量：

```conf
server {
    server_name   ~^(www\.)?(?<domain>.+)$;     # 捕获 domain 

    location / {
        root   /sites/$domain;                  # 将正在表达式中捕获的 domain 当作变量
    }
}
```

PCRE 库支持使用以下语法的命名捕获：

`?<name>`   Perl 5.10 兼容语法，自 PCRE-7.0 起支持
`?'name'`   Perl 5.10 兼容语法，自 PCRE-7.0 起支持
`?P<name>`  Python 兼容语法，自 PCRE-4.0 起支持

如果nginx启动失败并显示错误信息：

```text
pcre_compile() failed: unrecognized character after (?< in ...
```

上面的报错意味着 PCRE 库很旧，应该尝试使用语法 `?P<name>`。捕获也可以以数字形式使用：

```conf
server {
    server_name   ~^(www\.)?(.+)$;

    location / {
        root   /sites/$2;   
    }
}
```

然而，这种用法应仅限于简单的情况（如上面的情况），因为数字引用很容易被覆盖。

## 各种各样的名称

有一些 server_name 经过特殊处理。

如果需要在 `server` 块中处理没有 `Host` 头字段的请求（这不是默认的），则应指定一个空名称：

```conf
server {
    listen       80;
    server_name  example.org  www.example.org  "";
    ...
}
```

如果 `server` 块中没有定义 `server_name`，则 nginx 使用空名称作为服务器名称。(在这种情况下，0.8.48 之前的 nginx 版本使用计算机的主机名作为服务器名称。)。

如果 `server_name` 定义为`$hostname` (0.9.4)，则使用计算机的主机名。

如果有人使用 IP 地址而不是服务器名称发出请求，则 `Host` 请求头字段将包含 IP 地址，并且可以使用 IP 地址作为服务器名称来处理请求：

```conf
server {
    listen       80;
    server_name  example.org
                 www.example.org
                 ""
                 192.168.1.1
                 ;
    ...
}
```

在包罗万象的服务器示例中，可以看到奇怪的名称“_”：

```conf
server {
    listen       80  default_server;
    server_name  _;
    return       444;
}
```

上面这个 `server_name` 没有什么特别的，它只是无数与任何真实名称不相交的无效域名之一。其他无效名称，例如 `--` 和 `!@#`。

nginx 0.6.25 之前的版本支持特殊名称 `*`，该名称被错误地解释为包罗万象的名称。它从来没有充当包罗万象或通配符服务器名称。相反，它提供了现在由 `server_name_in_redirect` 指令提供的功能。特殊名称 `*` 现已弃用，应使用 `server_name_in_redirect` 指令。请注意，无法使用 `server_name` 指令指定包罗万象的名称或默认服务器。这是 `listen` 指令的属性，而不是 `server_name` 指令的属性。可以定义监听 `*:80` 和 `*:8080` 的服务器，并指定其中一个服务器作为端口 `*:8080` 的默认服务器，而另一个作为端口 `*:80` 的默认服务器：

```conf
server {
    listen       80;
    listen       8080  default_server;      # 指定为 8080 端口的默认服务器
    server_name  example.net;
    ...
}

server {
    listen       80  default_server;        # 指定为 80 端口的默认服务器
    listen       8080;
    server_name  example.org;
    ...
}
```

## 国际化名称

应在 `server_name` 指令中使用 ASCII (Punycode) 表示形式指定国际化域名 (IDN)：

```conf
server {
    listen       80;
    server_name  xn--e1afmkfd.xn--80akhbyknj4f;  # пример.испытание
    ...
}
```

## 虚拟服务器选择

首先，在默认服务器上下文中创建连接。然后，可以在以下请求处理阶段确定服务器名称，每个阶段都涉及服务器配置选择：

- 在 SSL 握手期间，之前。
- 处理完请求行后。
- 处理Host请求头之后。
- 如果在处理请求行之后或从 Host 请求头字段中未确定服务器名称，nginx 将使用空名称作为服务器名称。

在每个阶段，可以应用不同的服务器配置。因此，应谨慎指定某些指令：

- 在使用 [ssl_protocols](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) 指令的情况下，协议列表由 OpenSSL 库设置，然后才能根据通过 SNI 请求的名称应用服务器配置，因此，应仅为默认服务器指定协议；
- `client_header_buffer_size` 和 `merge_slashes` 指令在读取请求行之前执行，因此，这些指令使用默认服务器配置或 SNI 选择的服务器配置；
- 如果处理请求头字段涉及 `ignore_invalid_headers` `、large_client_header_buffers` 和 `underscores_in_headers` 指令，它还取决于服务器配置是否根据 请求行 或 Host 请求头字段进行更新；
- 错误响应将使用当前满足请求的服务器中的 `error_page` 指令进行处理。

## 优化

精确名称、以星号开头的通配符名称和以星号结尾的通配符名称存储在绑定到侦听端口的三个哈希表中。哈希表的大小在配置阶段进行优化，以便可以以最少的 CPU 缓存未命中找到名称。设置哈希表的详细信息在单独的[文档](https://nginx.org/en/docs/hash.html)中提供。

首先搜索确切的名称哈希表。如果未找到名称，则搜索具有以星号开头的通配符名称的哈希表。如果还是找不到该名称，则搜索具有以星号结尾的通配符名称的哈希表。

搜索通配符名称哈希表比搜索精确名称哈希表慢，因为名称是按域名部分搜索的。请注意，特殊通配符形式`.example.org`存储在通配符名称哈希表中，而不是存储在精确名称哈希表中。

正则表达式是按顺序测试的，因此是最慢的方法并且不可扩展。

由于这些原因，最好尽可能使用准确的名称。例如，如果最常请求的服务器名称是 `example.org` 和 `www.example.org`，则显式定义它们会更有效：

```conf
server {
    listen       80;
    server_name  example.org  www.example.org  *.example.org;
    ...
}
```

而不是使用简化形式：

```conf
server {
    listen       80;
    server_name  .example.org;
    ...
}
```

如果定义了大量服务器名称，或者定义了异常长的服务器名称，则可能需要在 `http` 级别调整 `server_names_hash_max_size` 和 `server_names_hash_bucket_size` 指令。 `server_names_hash_bucket_size` 指令的默认值可能等于 32、64 或其他值，具体取决于 CPU 缓存行大小。如果默认值为 32 并且服务器名称定义为 `too.long.server.name.example.org` ，则 nginx 将无法启动并将谁输出错误消息：

```text
could not build the server_names_hash,
you should increase server_names_hash_bucket_size: 32
```

在这种情况下，指令值应增加到 2 的下一个次幂：

```conf
http {
    server_names_hash_bucket_size  64;
    ...
```

如果定义了大量服务器名称，则会出现另一条错误消息：

```text
could not build the server_names_hash,
you should increase either server_names_hash_max_size: 512
or server_names_hash_bucket_size: 32
```

在这种情况下，首先尝试将 `server_names_hash_max_size` 设置为接近服务器名称数量的数字。只有当这没有效果，或者 nginx 的启动时间太长时，才尝试增加 `server_names_hash_bucket_size`。

如果服务器是侦听端口的唯一服务器，则 nginx 根本不会测试服务器名称（并且不会为侦听端口构建哈希表）。然而，有一个例外。如果服务器名称是带有捕获的正则表达式，则 nginx 必须执行该表达式来获取捕获。

## 兼容性

- 从 0.9.4 开始支持特殊服务器名称 "$hostname"。
- 从 0.8.48 开始，默认服务器名称值为空名称""。
- 自 0.8.25 起支持命名正则表达式服务器名称捕获。
- 从 0.7.40 开始支持正则表达式服务器名称捕获。
- 从 0.7.12 开始支持空服务器名称 ""。
- 自 0.6.25 起，支持将通配符服务器名称或正则表达式用作第一个服务器名称。
- 从 0.6.7 开始支持正则表达式服务器名称。
- 从 0.6.0 开始支持通配符形式 example.*。
- 自 0.3.18 起支持特殊形式 .example.org。
- 自 0.1.13 起支持通配符形式 *.example.org。
