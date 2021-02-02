# 简介

spring cloud 为开发人员提供了快速构建分布式系统的一些工具，包括**配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话**等等。它运行环境简单，可以在开发人员的电脑上跑。

## 服务注册中心(Eureka)

### 一：创建一个maven主工程

这个pom文件作为父pom文件，起到依赖版本控制的作用，其他module工程继承该pom

### 二：创建model工程（Eureka Server,Eureka Client）

其pom.xml继承了父pom文件，并引入spring-cloud-starter-netflix-eureka-server的依赖

```xml
<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
```

!> 在springboot工程的启动application类上加一个注解@EnableEurekaServer来启动一个服务注册中心

在默认情况下erureka server也是一个eureka client ,必须要指定一个 server。

```xml
<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
```

!> 通过注解@EnableEurekaClient 表明自己是一个eurekaclient.还需要在配置文件中注明自己的服务注册中心的地址

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

>  Spring cloud有两种服务调用方式，一种是ribbon+restTemplate，另一种是feign

## 负载均衡客户端（ribbon）

### 建一个服务消费者

```xml
 <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
```

!> 在工程的启动类中,通过@EnableDiscoveryClient向服务中心注册；并且向程序的ioc注入一个bean: restTemplate;并通过@LoadBalanced注解表明这个restRemplate开启负载均衡的功能。

在ribbon中它会根据服务名来选择具体的服务实例，根据服务实例在请求的时候会用具体的url替换掉服务名

```java
 public String hiService(String name) {
        return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
    }
```

## 通过Feign去消费服务


> - Feign 采用的是基于接口的注解
> - Feign 整合了ribbon，具有负载均衡的能力
> - 整合了Hystrix，具有熔断的能力

起步依赖spring-cloud-starter-feign

!> 在程序的启动类Application ，加上@EnableFeignClients注解开启Feign的功能

定义一个feign接口，通过@ FeignClient（“服务名”），来指定调用哪个服务。比如在代码中调用了service-hi服务的“/hi”接口

```java
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

## 断路器简介（Hystrix）

## 在ribbon使用断路器

```xml
<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
```

!> 在程序的启动类ServiceRibbonApplication 加@EnableHystrix注解开启Hystrix,在Service方法上加上@HystrixCommand注解。该注解对该方法创建了熔断器的功能，并指定了fallbackMethod(快速失败方法)熔断方法

## Feign中使用断路器

> 只需要在FeignClient的SchedualServiceHi接口的注解中加上fallback的指定类就行了

```java
@FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

>  SchedualServiceHiHystric需要实现SchedualServiceHi 接口，并注入到Ioc容器中

```java
@Component
public class SchedualServiceHiHystric implements SchedualServiceHi {
    @Override
    public String sayHiFromClientOne(String name) {
        return "sorry "+name;
    }
}
```

## Zuul

> Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如／api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。

```xml
 <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
```

!> 在其入口applicaton类加上注解@EnableZuulProxy，开启zuul的功能

```yaml
zuul:
  routes:
    api-a:
      path: /api-a/**
      serviceId: service-ribbon
    api-b:
      path: /api-b/**
      serviceId: service-feign
```

!> zuul不仅只是路由，并且还能过滤，做一些安全验证。

```java
@Component
public class MyFilter extends ZuulFilter {

    private static Logger log = LoggerFactory.getLogger(MyFilter.class);
    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s >>> %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("token");
        if(accessToken == null) {
            log.warn("token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            try {
                ctx.getResponse().getWriter().write("token is empty");
            }catch (Exception e){}

            return null;
        }
        log.info("ok");
        return null;
    }
}
```

- filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
  - pre：路由之前
  - routing：路由之时
  - post： 路由之后
  - error：发送错误调用
- filterOrder：过滤的顺序
- shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
- run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。

## 配置服务（Spring Cloud Config）

方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。

> 支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。
>
> 组件有两种：一是config server，二是config client

### config server

```xml
<artifactId>spring-cloud-config-server</artifactId>		
```

!> 在程序的入口Application类加上@EnableConfigServer注解开启配置服务器的功能

需要在程序的配置文件application.properties文件配置以下：

```properties
spring.application.name=config-server
server.port=8888

spring.cloud.config.server.git.uri=https://github.com/forezp/SpringcloudConfig/
spring.cloud.config.server.git.searchPaths=respo
spring.cloud.config.label=master
spring.cloud.config.server.git.username=
spring.cloud.config.server.git.password=
```

- spring.cloud.config.server.git.uri：配置git仓库地址
- spring.cloud.config.server.git.searchPaths：配置仓库路径
- spring.cloud.config.label：配置仓库的分支
- spring.cloud.config.server.git.username：访问git仓库的用户名
- spring.cloud.config.server.git.password：访问git仓库的用户密码

配置服务中心可以从远程程序获取配置信息。

```json
{"name":"foo","profiles":["dev"],"label":"master",
"version":"792ffc77c03f4b138d28e89b576900ac5e01a44b","state":null,"propertySources":[]}
```

http请求地址和资源文件映射如下:

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

## config client

```xml
<artifactId>spring-cloud-starter-config</artifactId>
```

配置文件**bootstrap.properties**：

```properties
spring.application.name=config-client
spring.cloud.config.label=master
spring.cloud.config.profile=dev
spring.cloud.config.uri= http://localhost:8888/
server.port=8881
```

- spring.cloud.config.label 指明远程仓库的分支
- spring.cloud.config.profile
  - dev开发环境配置文件
  - test测试环境
  - pro正式环境
- spring.cloud.config.uri= http://localhost:8888/ 指明配置服务中心的网址。