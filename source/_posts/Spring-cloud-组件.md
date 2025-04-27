---
title: Spring cloud 组件
excerpt: "Spring cloud 组件"
date: 2025-04-27 22:20:57
tag: [Spring cloud,微服务]
categories: [Spring cloud]
---

## maven依赖相关

spring cloud 与spring cloud alibaba 均依赖于spring boot

所以要先制定springboot的版本，再指定spring cloud 与spring cloud alibaba 的版本：
例如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.4</version>
    </parent>

    <packaging>pom</packaging>

    <properties>
        <spring-cloud.version>2023.0.3</spring-cloud.version>
        <spring-cloud-alibaba.version>2023.0.3.2</spring-cloud-alibaba.version>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>

```



## nacos

官方地址：https://nacos.io/

> 服务的 **注册中心** 与 **配置中心**


### 注册中心


windows 单机启动：
> startup.cmd -m standalone


需要引入服务发现的包：
```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```

所需要的配置：
```yaml
spring:
  application:
    name: demo-service
  cloud:
    nacos:
      server-addr: localhost:8848  #nacos  地址
```

再启动类上添加启用服务发现的注解：
> @EnableDiscoveryClient

即可使用如下代码获取其他的服务地址：
```java

    @Autowired
    DiscoveryClient discoveryClient; //spring 规范

    @Autowired
    NacosServiceDiscovery nacosServiceDiscovery; // nacos提供
    
    //获取服务的ip与端口，直接使用restTeplate调用即可
```


以上方式每次只能获取固定的服务IP与Port端口，不能做到负载，可以引入负载均衡的包：
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
```

可以使用负载均衡的方式获取多实例服务的其中某个实例：
```java

    @Autowired
    LoadBalancerClient loadBalancerClient;
    
    ......
    
    ServiceInstance choose = loadBalancerClient.choose("product-service");
    
    //服务的接口地址
    String url = "http://" + choose.getHost() + ":" + choose.getPort() + "/product/" + id;
```

但是,这样还是很麻烦，虽然获取了服务的ip与端口，但是还是要拼写，于是我们可以用@LoadBalanced注解的方式:

```java
    //给注册restTemplate实例的上面加上负载均衡的注解
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
    
    ......
    
    //后面使用时即可用这种方式，直接发起服务调用  product-service 即为服务注册的名称
    
    String url = "http://product-service/product/" + id;

    ResponseEntity<Product> productResponseEntity = restTemplate.getForEntity(url, Entity.class);
```


### 配置中心

项目的每个服务需要维护很多个yaml配置，可以将这些配置放在nacos中，并且可以根据环境和服务分开隔离存储，以便启动时按需加载和动态更新。

引入包：

```xml

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>

```

配置如下：

```yaml

spring:
  application:
    name: demo-service
  cloud:
    nacos:
      server-addr: localhost:8848
      config:
        namespace: 9bced401-649a-4e77-9526-89a27d8b3210   #命名空间的id 不是名称
  config:
    import:   #那个 组（服务） 的 哪个配置
      - nacos:common.yml?group=order
      - nacos:datasource.yml?group=order
```

注意：如果本地与nacos存在相同的配置，则nacos的会覆盖本地的。


配置的动态刷新：

1. 使用 @ConfigurationProperties(prefix = "licon")  将配置映射到实体类上，实现无感刷新
2. 使用 @Value("${xx}") + @RefreshScope标注在使用的地方上，即可实现配置的动态刷新

给配置变化添加一个监听器，做一些操作,可以使用 NacosConfigManager，例如：

```java
    @Bean
    ApplicationRunner applicationRunner(NacosConfigManager nacosConfigManager){
        return args -> {
            ConfigService configService = nacosConfigManager.getConfigService();
            configService.addListener("order-service.yml", "DEFAULT_GROUP", new Listener() {
                @Override
                public Executor getExecutor() {
                    return Executors.newFixedThreadPool(4);
                }

                @Override
                public void receiveConfigInfo(String s) {
                    System.out.println("配置变化："+s);
                }
            });
        };
    }
```


## openfeign 远程服务调用组件

上面提到过使用 restTemplate来进行服务的代用，但是每次都需要写地址，这样很麻烦，我们可以使用 openfeign 来简化这个过程

引入依赖：
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

配置：

开启日志，先要添加一个对象，再在yaml中配置日志
```java
    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
```

```yaml
logging:
  level:
    com:
      licon:
        feign: debug  #开启日志
```


编写一个远程服务调用的接口，然后再类上打上注解，@FeignClient ,例如：
```java
@FeignClient(value = "product-service",fallback = ProductFeignFallback.class)
public interface ProductFeignClient {

    @GetMapping("/product/{id}")
    Product getProductById(@PathVariable("id") Long id);
}
```

其中 value是服务的名称

fallback 是一个兜底机制，用于指定一个降级处理类（即一个实现了接口的类），当 Feign 客户端调用失败时，自动回退到这个类中定义的处理逻辑

fallbackFactory：提供一个降级工厂类，能够创建包含更多上下文信息的降级处理实例，适合更复杂的降级逻辑。
```java

@Component
public class MyServiceFallbackFactory implements FallbackFactory<MyServiceClient> {
    @Override
    public MyServiceClient create(Throwable cause) {
        return new MyServiceClient() {
            @Override
            public String getData() {
                // 根据异常信息返回不同的降级响应
                if (cause instanceof FeignException.FeignClientException) {
                    return "Fallback response: Client error occurred.";
                } else {
                    return "Fallback response: Service is currently unavailable.";
                }
            }
        };
    }
}
```

配置好就可以直接使用ProductFeignClient 调用方法即可。


请求拦截器：RequestInterceptor

响应拦截器：ResponseInterceptor

## sentinel



