# 分布式服务框架Dubbo

dubbo是一个RPC(Remote Procedure Call 远程过程调用)框架，SOA(Service Oriented Architecture面向服务框架)框架。当服务越来越多，服务url配置管理变得非常困难，服务之间依赖越来越复杂，服务调用量越来越大，就需要用分布式服务来解决问题。dubbo就是分布式服务的一种。

## Dubbo技术架构

  - Provider：服务提供方
  - Consumer：服务消费方
  - Register：服务注册中心
  - Monitor：服务监控中心
  - Container：服务运行容器

Dubbo框架很像生产者-消费者模型，在这种模型基础上加上了注册中心和监控中心用来管理和监控服务。

Dubbo不需要web容器，直接打成jar运行即可
Dubbo推荐使用Hessiian序列化
Dubbo默认使用Netty通信框架
Dubbo有6种集群容错方案，默认失败自动切换
Dubbo有4种负载均衡策略，默认随机，按权重设置随机概率


发布-订阅的过程

  - 启动容器，加载，运行服务提供者
  - 服务提供方在启动时，自动检查依赖服务是否可用，在注册中心发布自己提供的服务
  - 服务消费方在启动时，在注册中心订阅自己所需的服务
 
失败或变更时
  
  - 注册中心，返回服务提供方地址列表给消费方，如果变更，注册中心将基于长连接推送变更数据给消费方
  - 服务消费方，从提供方地址列表中，基于负载均衡算法，选择一个提供方进行服务调用，如果失败，选择其他
  - 服务消费方和提供方，在内存中累计调用次数和调用时间定时发送发到监控中心
