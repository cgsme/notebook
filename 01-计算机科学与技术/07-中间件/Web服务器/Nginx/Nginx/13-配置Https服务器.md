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
