# SpringBootService 微服务

  - Maven创建SpringBoot项目
  - 配置SpringBoot项目pom.xml文件
  - 配置SpringBoot项目properties文件
  - 编写启动文件
  
运行Debug，执行启动文件
使用Maven Project打包程序，创建bat文件启动服务

# SpringBootEureka 注册中心

  - Maven创建SpringBoot项目
  - 配置SpringBoot项目pom.xml文件
  - 配置SpringBoot项目properties文件
  - 编写启动文件
  
运行Debug，执行启动文件
修改SpringBootService，让Eureka发现此服务。
  - pom.xml添加spring-cloud-starter-netflix-eureka-client
  - properties中添加eureka.client.service-url.defaultZone
  - 启动文件中添加@EnableDiscoveryClient注解

# SpringBootConfig 配置中心

配置中心其实就是读取SVN上的文件后，发送给其他服务，让他们读取。
使用SVN或其他，在目录下创建springbootService-release.properties文件，将SpringBootService的application.properties的内容复制进去

  - Maven创建SpringBoot项目
  - 配置SpringBoot项目pom.xml文件
  - 配置SpringBoot项目application.yml文件
  - 编写启动文件
  
运行Debug，执行启动文件
修改SpringBootService，让它能读取配置中心的配置文件。
  
  - 在resources目录下创建bootstrap.yml，用来配置读取Config配置中心下的哪个文件
  - pom.xml文件添加spring-cloud-starter-config和spring-boot-starter-actuator
  - 启动文件中添加@RefreshScope

# SpringBootConsumer 客户端

## Feign 

Feign客户端是一个web声明式http远程调用工具，提供了接口和注解方式进行调用，类似controller调用service。
Feign的一个关键机制就是使用了动态代理。
  - 某个接口定义了@FeignClient注解，Feign就会针对这个接口创建一个动态代理
  - 接着你要是调用那个接口，本质就是会调用 Feign创建的动态代理，这是核心中的核心
  - 根据你在接口上的@RequestMapping等注解，来动态构造出你要请求的服务的地址
  - 最后针对这个地址，发起请求、解析响应
  
## Hystrix 断路器

在springcloud中断路器组件就是Hystrix。Hystrix也是Netflix套件的一部分。他的功能是，当对某个服务的调用在一定的时间内（默认10s，由metrics.rollingStats.timeInMilliseconds配置），有超过一定次数（默认20次，由circuitBreaker.requestVolumeThreshold参数配置）并且失败率超过一定值（默认50%，由circuitBreaker.errorThresholdPercentage配置），该服务的断路器会打开。返回一个由开发者设定的fallback

## Ribbon 负载均衡

每次请求时选择一台机器，均匀的把请求分发到各个机器上。Ribbon的负载均衡默认使用的最经典的Round Robin轮询算法。

## SpringBootConsumerFeign项目

直接从SpringBootService复制一份，命名为springbootConsumerFeign，修改配置里面的名字
  - 在pom中添加Feign
  - 修改application.properties文件
  - 启动文件中添加@EnableFeignClients注解
  - 新建client目录并添加ServiceFeignClient.java文件来声明接口指向SpringBootService的接口
  - 添加ServiceFallback.java文件，用于处理熔断发生事件
  - 修改ServiceController.java，调用ServiceFeignClient的方法

启动SpringBootConsumer项目测试接口
测试Hystrix是否能正常使用

## SpringBootConsumerRibbon项目

使用Ribbbon+RestTemplate调用各个微服务。
直接从SpringBootService复制一份，命名为springbootConsumerRibbon，修改配置里面的名字
  - 在pom中添加Ribbon
  - 修改application.properties文件
  - 启动文件中添加@EnableHystrix注解，配置RestTemplate
  - ServiceController.java中引入RestTemplate，使用RestTemplate调用数据接口

启动SpringBootConsumer项目测试接口
测试Hystrix是否能正常使用


# SpringBootZuul 路由控制
