# Ingress

Service的统一网关入口

## 工作原理（nginx为例）

1、用户编写Ingress规则，说明哪个域名对应集群中的哪个service
2、Ingress控制器动态感知Ingress服务规则的变化，然后生成一段对应的Nignx反向代理配置
3、Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中，并不断更新
4、到此为止，其实真正在工作的就是一个Nignx，内部配置了用户定义的请求转发规则
