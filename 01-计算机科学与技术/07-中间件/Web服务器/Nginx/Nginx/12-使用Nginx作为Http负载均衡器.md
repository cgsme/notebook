# 使用Nginx作为Http负载均衡器

## 介绍

跨多个应用程序实例的负载均衡是优化资源利用率、最大化吞吐量、减少延迟和确保容错配置的常用技术。

可以使用 nginx 作为非常高效的 HTTP 负载均衡器，将流量分配到多个应用程序服务器，并使用 nginx 提高 Web 应用程序的性能、可扩展性和可靠性。

## 负载均衡方法

nginx 支持以下负载均衡机制（或​​方法）：

- round-robin —— 轮询，对应用程序服务器的请求以循环方式分发，
- least-connected —— 最少连接，下一个请求将分配给活动连接数最少的服务器，
- ip-hash —— 哈希函数用于确定应为下一个请求选择哪个服务器（基于客户端的 IP 地址）。

## 默认负载均衡配置（轮询）

使用 nginx 进行负载平衡的最简单配置如下所示：

```conf
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
```

在上面的示例中，同一应用程序的 3 个实例在 srv1-srv3 上运行。当没有特别配置负载均衡方法时，默认为**轮询**。所有请求都代理到服务器组 myapp1，nginx应用HTTP负载均衡来分发请求。

nginx 中的反向代理实现包括 HTTP、HTTPS、FastCGI、uwsgi、SCGI、memcached 和 gRPC 的负载平衡。

要为 HTTPS 而不是 HTTP 配置负载平衡，只需使用 https 作为协议。

为 FastCGI、uwsgi、SCGI、memcached 或 gRPC 设置负载平衡时，分别使用 `fastcgi_pass`、`uwsgi_pass`、`scgi_pass`、`memcached_pa​​ss` 和 `grpc_pass` 指令。

## 最少连接负载均衡

另一个负载平衡规则是最少连接。当某些请求需要更长的时间才能完成时，最少连接允许更公平地控制应用程序实例上的负载。

通过最少连接的负载均衡，nginx 会尽量不让繁忙的应用服务器因过多的请求而超载，而是将新请求分发到不太繁忙的服务器。

当least_conn指令用作服务器组配置的一部分时，nginx中的最少连接负载均衡被激活：

```conf
    upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
```

## 会话保持（ip hash）

请注意，通过**轮询**或**最少连接**负载均衡，每个客户端的请求可能会分发到不同的服务器。无法保证同一客户端始终定向到同一服务器。

如果需要将客户端绑定到特定的应用程序服务器（换句话说，使客户端的会话“粘性”或“持久”，总是尝试选择特定的服务器），则可以使用 ip-hash 负载平衡机制用过的。

使用 ip-hash，客户端的 IP 地址用作哈希密钥，以确定应为客户端的请求选择服务器组中的哪台服务器。此方法确保来自同一客户端的请求始终会定向到同一服务器，除非该服务器不可用。

要配置 ip-hash 负载均衡，只需将 ip_hash 指令添加到服务器（upstream）组配置中即可：

```conf
upstream myapp1 {
    ip_hash;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```

## 加权负载均衡

还可以通过使用服务器权重进一步影响 nginx 负载均衡算法。

在上面的示例中，未配置服务器权重，这意味着所有指定的服务器都被视为同等适合特定均衡方法。

特别是对于轮询，它还意味着在服务器之间或多或少平等地分配请求 - 前提是有足够的请求，并且请求以统一的方式处理并足够快地完成。

当为服务器指定权重参数时，权重将被视为负载均衡决策的一部分。

```conf
upstream myapp1 {
    server srv1.example.com weight=3;
    server srv2.example.com;
    server srv3.example.com;
}
```

通过此配置，每 5 个新请求将在应用程序实例之间分配，如：3 个请求将定向到 srv1，一个请求将定向到 srv2，另一个请求将定向到 srv3。

在最新版本的 nginx 中，同样可以使用具有最少连接的权重和 ip-hash 负载均衡。

## 健康检查

nginx 中的反向代理实现包括in-band (或 passive)服务器健康检查。如果某个特定服务器的响应因错误而失败，nginx 会将该服务器标记为失败，并在一段时间内尝试避免选择该服务器来处理后续的入站请求。

`max_fails` 指令设置在 `fail_timeout` 期间连续尝试与服务器通信失败的次数。默认情况下， `max_fails` 设置为 1。设置为 0 时，将禁用该服务器的运行状况检查。 `fail_timeout` 参数还定义服务器将被标记为失败的时间长度。服务器故障后经过 `fail_timeout` 间隔后，nginx将开始用实时客户端的请求优雅地探测服务器。如果探测成功，服务器将被标记为活动服务器。

## 更多内容

此外，nginx 中还有更多控制服务器负载平衡的指令和参数，例如 `proxy_next_upstream` 、 `backup` 、 `down` 和 `keepalive` 。[参考文档](https://nginx.org/en/docs/)。
