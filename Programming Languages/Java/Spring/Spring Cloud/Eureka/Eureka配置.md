# Eureka 配置详解

## Eureka Server

## Eureka Client

- eureka.client.enabled=true                        |   是否开启client，默认true
- eureak.client.register-with-eureka=true           |   是否将服务注册至注册中心，默认true
- eureka.client.fetch-registry=true                 |   是否从注册中心检索服务，默认true

## Eureka instance

- eureka.instance.prefer-ip-address=false           |   注册服务时是否使用IP注册，默认false
- eureka.instance.ip-address                        |   实例的IP地址
- eureka.instance.hostname                          |   实例的hostname，默认localhost
- eureka.instance.instance-id                       |   实例的ID
