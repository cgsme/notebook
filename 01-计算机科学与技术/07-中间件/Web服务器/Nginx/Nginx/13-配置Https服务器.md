# 配置Https服务器

要配置 HTTPS 服务器，必须在 server 块中的侦听套接字上启用 ssl 参数，并且应指定服务器证书和私钥文件的位置：

```conf
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ...
}
```

服务器证书是公共实体。它被发送到连接到服务器的每个客户端。私钥是一个安全实体，应存储在访问受限的文件中，但是它必须可由 nginx 的主进程读取。私钥也可以与证书存储在同一文件中：

```conf
ssl_certificate     www.example.com.cert;
ssl_certificate_key www.example.com.cert;
```

在这种情况下，文件访问权限也应受到限制。尽管证书和密钥存储在一个文件中，但只有证书被发送到客户端。

指令 `ssl_protocols` 和 `ssl_ciphers` 可用于限制连接仅包含 SSL/TLS 的强版本和密码。默认情况下，nginx 使用 `ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3` 和 `ssl_ciphers HIGH:!aNULL:!MD5`，因此通常不需要显式配置它们。注意，这些指令的默认值已更改多次。

## HTTPS服务器优化

SSL 操作会消耗额外的 CPU 资源。在多处理器系统上，应运行多个工作进程，数量不少于可用 CPU 核心的数量。有两种方法可以最大限度地减少每个客户端的这些操作数量：第一个是启用保活连接以通过一个连接发送多个请求，第二个是重用 SSL 会话参数以避免并行和后续连接的 SSL 握手。会话存储在 worker 之间共享的 SSL 会话缓存中，并由 `ssl_session_cache` 指令配置。 1 MB 的缓存包含大约 4000 个会话。默认缓存超时为 5 分钟。可以通过使用 `ssl_session_timeout` 指令来增加它。以下是针对具有 10 MB 共享会话缓存的多核系统进行优化的示例配置：

```conf
worker_processes auto;

http {
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;

    server {
        listen              443 ssl;
        server_name         www.example.com;
        keepalive_timeout   70;

        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        ...
```

## SSL 证书链

某些浏览器可能会抱怨由知名证书颁发机构签署的证书，而其他浏览器可能会毫无问题地接受该证书。发生这种情况的原因是，颁发机构使用中间证书签署了服务器证书，而该中间证书不存在于随特定浏览器分发的众所周知的受信任证书颁发机构的证书库中。在这种情况下，权威机构提供了一系列链式证书，这些证书应连接到签名的服务器证书。服务器证书必须出现在组合文件中的链接证书之前：

```shell
cat www.example.com.crt bundle.crt > www.example.com.chained.crt
```

生成的文件应在 `ssl_certificate` 指令中使用：

```conf
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.chained.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```

如果服务器证书和捆绑包以错误的顺序连接，nginx 将无法启动并显示错误消息：

```text
SSL_CTX_use_PrivateKey_file(" ... /www.example.com.key") failed
   (SSL: error:0B080074:x509 certificate routines:
    X509_check_private_key:key values mismatch)
```

因为 nginx 尝试使用捆绑包的第一个证书的私钥而不是服务器证书。

浏览器通常存储它们收到并由可信机构签名的中间证书，因此，经常使用的浏览器可能已经拥有所需的中间证书，并且可能不会抱怨没有链接捆绑包发送的证书。为了确保服务器发送完整的证书链，可以使用 `openssl` 命令行，例如：

```shell
openssl s_client -connect www.godaddy.com:443
...
Certificate chain
 0 s:/C=US/ST=Arizona/L=Scottsdale/1.3.6.1.4.1.311.60.2.1.3=US
     /1.3.6.1.4.1.311.60.2.1.2=AZ/O=GoDaddy.com, Inc
     /OU=MIS Department/CN=www.GoDaddy.com
     /serialNumber=0796928-7/2.5.4.15=V1.0, Clause 5.(b)
   i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=Go Daddy Secure Certification Authority
     /serialNumber=07969287
 1 s:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=Go Daddy Secure Certification Authority
     /serialNumber=07969287
   i:/C=US/O=The Go Daddy Group, Inc.
     /OU=Go Daddy Class 2 Certification Authority
 2 s:/C=US/O=The Go Daddy Group, Inc.
     /OU=Go Daddy Class 2 Certification Authority
   i:/L=ValiCert Validation Network/O=ValiCert, Inc.
     /OU=ValiCert Class 2 Policy Validation Authority
     /CN=http://www.valicert.com//emailAddress=info@valicert.com
...
```

> 使用 SNI 测试配置时，指定 -servername 选项非常重要，因为 openssl 默认情况下不使用 SNI。

在此示例中，www.GoDaddy.com 服务器证书 #0 的主题 (“s”) 由颁发者 (“i”) 签名，该颁发者本身就是证书 #1 的主题，该证书由颁发者签名，该颁发者本身就是证书 #2 的主题，该证书由知名颁发者 ValiCert, Inc. 签名。其证书存储在浏览器的内置证书库中（位于杰克建造的房子里）。

如果尚未添加证书捆绑包，则仅显示服务器证书#0。

## 单个 HTTP/HTTPS 服务器

可以配置一个服务器来处理 HTTP 和 HTTPS 请求：

```conf
server {
    listen              80;
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```

> 在 0.7.14 之前，无法有选择地为各个侦听套接字启用 SSL，如上所示。使用 ssl 指令只能为整个服务器启用 SSL，因此无法设置单个 HTTP/HTTPS 服务器。添加 `listen` 指令的 ssl 参数来解决这个问题。因此，不鼓励在现代版本中使用 ssl 指令。

## 基于名称的 HTTPS 服务器

配置两个或多个 HTTPS 服务器侦听单个 IP 地址时会出现一个常见问题：

```conf
server {
    listen          443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```

通过此配置，浏览器将接收默认服务器的证书，即 `www.example.com`，无论请求的服务器名称如何。这是由 SSL 协议行为引起的。 SSL 连接是在浏览器发送 HTTP 请求之前建立的，并且 nginx 不知道请求的服务器的名称。因此，它可能只提供默认服务器的证书。

解决该问题的最古老、最可靠的方法是为每个 HTTPS 服务器分配一个单独的 IP 地址：

```conf
server {
    listen          192.168.1.1:443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          192.168.1.2:443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```

## 具有多个名称的 SSL 证书

有其他方法允许在多个 HTTPS 服务器之间共享单个 IP 地址。然而，它们都有各自的缺点。一种方法是在 SubjectAltName 证书字段中使用具有多个名称的证书，例如 `www.example.com` 和 `www.example.org`。然而，SubjectAltName 字段的长度是有限的。

另一种方法是使用具有通配符名称的证书，例如，`*.example.org`。通配符证书确保指定域的所有子域的安全，但仅在一个级别上。此证书匹配 `www.example.org`，但不匹配 `example.org` 和 `www.sub.example.org`。这两种方法也可以结合使用。证书可能在 SubjectAltName 字段中包含确切和通配符名称，例如，`example.org` 和 `*.example.org`。

最好将具有多个名称的证书文件及其私钥文件放置在 `http` 级别，以便在所有服务器中继承其单个内存副本：

```conf
ssl_certificate     common.crt;
ssl_certificate_key common.key;

server {
    listen          443 ssl;
    server_name     www.example.com;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ...
}
```

## 服务器名称指示 (Server Name Indication)

在单个 IP 地址上运行多个 HTTPS 服务器的更通用解决方案是 [TLS Server Name Indication extension](http://en.wikipedia.org/wiki/Server_Name_Indication)（SNI，RFC 6066），它允许浏览器在 SSL 握手期间传递请求的服务器名称，因此，服务器将知道应为连接使用哪个证书。SNI 目前被大多数现代浏览器支持，但某些旧版或特殊客户端可能不使用。

> 在 SNI 中只能传递域名，然而，如果请求包含字面值的 IP 地址，某些浏览器可能会错误地将服务器的 IP 地址作为其名称传递。不应依赖于此。

为了在 Nginx 中使用 SNI，它必须在构建 Nginx 二进制文件的 OpenSSL 库以及运行时动态链接的库中都得到支持。自 0.9.8f 版本以来，如果使用 `--enable-tlsext` 配置选项构建 OpenSSL，则 OpenSSL 支持 SNI。自 OpenSSL 0.9.8j 起，此选项默认启用。如果 Nginx 是在支持 SNI 的情况下构建的，那么在使用 `-V` 开关运行时，Nginx 将会输出提示：

```shell
$ nginx -V
...
TLS SNI support enabled
...
```

然而，如果启用了 SNI 的 Nginx 动态链接到不支持 SNI 的 OpenSSL 库，Nginx 会显示警告：

```text
nginx was built with SNI support, however, now it is linked
dynamically to an OpenSSL library which has no tlsext support,
therefore SNI is not available
```

## 兼容性

- 自 0.8.21 和 0.7.62 以来，“-V” 开关已显示 SNI 支持状态。
- 自 0.7.14 以来，listen 指令的 ssl 参数已受支持。在 0.8.21 之前，它只能与 default 参数一起指定。
- 自 0.5.23 以来，SNI 已受支持。
- 自 0.5.6 以来，共享 SSL 会话缓存已受支持。
- 1.23.4 及更高版本：默认 SSL 协议为 TLSv1、TLSv1.1、TLSv1.2 和 TLSv1.3（如果 OpenSSL 库支持）。
- 1.9.1 及更高版本：默认 SSL 协议为 TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 库支持）。
- 0.7.65、0.8.19 及更高版本：默认 SSL 协议为 SSLv3、TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 库支持）。
- 0.7.64、0.8.18 及更早版本：默认 SSL 协议为 SSLv2、SSLv3 和 TLSv1。
- 1.0.5 及更高版本：默认 SSL 密码为 “HIGH:!aNULL:!MD5” 。
- 0.7.65、0.8.20 及更高版本：默认 SSL 密码为 “HIGH:!ADH:!MD5” 。
- 0.8.19 版本：默认 SSL 密码为 “ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM” 。
- 0.7.64、0.8.18 及更早版本：默认 SSL 密码为 “ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP” 。
