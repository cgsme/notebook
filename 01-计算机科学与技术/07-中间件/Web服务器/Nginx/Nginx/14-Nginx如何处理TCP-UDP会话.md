# nginx 如何处理 TCP/UDP 会话

来自客户端的 TCP/UDP 会话在被称为 phases 的连续步骤中进行处理：

`Post-accept`

在接受客户端连接后的第一阶段。[ngx_stream_realip_module](https://nginx.org/en/docs/stream/ngx_stream_realip_module.html) 模块在这个阶段被调用。

`Pre-access`

初步的访问检查。在此阶段会调用 [ngx_stream_limit_conn_module](https://nginx.org/en/docs/stream/ngx_stream_limit_conn_module.html) 和 [ngx_stream_set_module](https://nginx.org/en/docs/stream/ngx_stream_set_module.html) 模块。

`Access`

在实际数据处理之前的客户端访问限制。在此阶段，调用 [ngx_stream_access_module](https://nginx.org/en/docs/stream/ngx_stream_access_module.html) 模块，对于 [njs](https://nginx.org/en/docs/njs/index.html)，调用[js_access](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_access) 指令。

`SSL`

TLS/SSL 终止。[ngx_stream_ssl_module](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html) 模块在这个阶段被调用。

`Preread`

将初始数据字节读入 [预读缓冲区](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#preread_buffer_size) ，以便诸如 [ngx_stream_ssl_preread_module](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html) 之类的模块在处理数据之前对其进行分析。对于 [njs](https://nginx.org/en/docs/njs/index.html)，调用[js_access](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_access) 指令。

`Content`

强制阶段，在此阶段数据实际被处理，通常被代理到 upstream 服务器，或者指定的值被返回给客户端。对于 [njs](https://nginx.org/en/docs/njs/index.html)，调用[js_filter](https://nginx.org/en/docs/stream/ngx_stream_js_module.html#js_filter) 指令。

`Log`

客户端会话处理结果被记录的最后阶段。在此阶段会调用 [ngx_stream_log_module](https://nginx.org/en/docs/stream/ngx_stream_log_module.html) 模块。
