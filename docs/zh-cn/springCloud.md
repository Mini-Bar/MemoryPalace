# 简介



spring cloud 为开发人员提供了快速构建分布式系统的一些工具，包括**配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话**等等。它运行环境简单，可以在开发人员的电脑上跑。

[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)



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

## Spring Cloud Bus

将分布式的节点用轻量的消息代理连接起来。它可以用于广播配置文件的更改或者服务之间的通讯，也可以用于监控。以下是用Spring Cloud Bus实现通知微服务架构的配置文件的更改。

[安装RabbitMq](zh-cn/rabbitMq)

在pom文件加上起步依赖spring-cloud-starter-bus-amqp

```xml
<artifactId>spring-cloud-starter-bus-amqp</artifactId>
```

在配置文件application.properties中加上RabbitMq的配置，包括RabbitMq的地址、端口，用户名、密码。并需要加上spring.cloud.bus的三个配置

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

spring.cloud.bus.enabled=true
spring.cloud.bus.trace.enabled=true
management.endpoints.web.exposure.include=bus-refresh
```

如果是传统的做法，需要重启服务，才能达到配置文件的更新。此时，我们只需要发送post请求：http://localhost:8881/actuator/bus-refresh 你会发现config-client会重新读取配置文件.

/actuator/bus-refresh接口可以指定服务，即使用”destination”参数，比如 “/actuator/bus-refresh?destination=customers:” 即刷新服务名为customers的所有服务。

## Spring Cloud Sleuth(集成服务追踪组件zipkin)

### 一：术语

- **Span**：基本工作单元，例如，在一个新建的span中发送一个RPC等同于发送一个回应请求给RPC，span通过一个64位ID唯一标识，trace以另一个64位ID表示，span还有其他数据信息，比如摘要、时间戳事件、关键值注释(tags)、span的ID、以及进度ID(通常是IP地址) span在不断的启动和停止，同时记录了时间信息，当你创建了一个span，你必须在未来的某个时刻停止它。

- **Trace**：一系列spans组成的一个树状结构，例如，如果你正在跑一个分布式大数据工程，你可能需要创建一个trace。

- **Annotation**：用来及时记录一个事件的存在，一些核心annotations用来定义一个请求的开始和结束

  - `cs` - Client Sent -客户端发起一个请求，这个annotion描述了这个span的开始
  - `sr` - Server Received -服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络延迟
  - `ss` - Server Sent -注解表明请求处理的完成(当请求返回客户端)，如果ss减去sr时间戳便可得到服务端需要的处理请求时间
  - `cr` - Client Received -表明span的结束，客户端成功接收到服务端的回复，如果cr减去cs时间戳便可得到客户端从服务端获取回复的所有所需时间 将Span和Trace在一个系统中使用Zipkin注解的过程图形化：

> 在spring Cloud为F版本的时候，已经不需要自己构建Zipkin Server了，只需要下载jar即可

下载完成jar 包之后，需要运行jar

> java -jar zipkin-server-2.10.1-exec.jar

访问浏览器localhost:9494

创建service-hi,在其pom引入起步依赖spring-cloud-starter-zipkin

```xml
<artifactId>spring-cloud-starter-zipkin</artifactId>
```

在其配置文件application.yml指定zipkin server的地址，头通过配置“spring.zipkin.base-url”指定：

```properties
server.port=8988
spring.zipkin.base-url=http://localhost:9411
spring.application.name=service-hi
```

!> 其实通过引入spring-cloud-starter-zipkin依赖和设置spring.zipkin.base-url就可以了。对外暴露接口!

# 高可用的服务注册中心

Eureka通过运行多个实例，使其更具有高可用性。事实上，这是它默认的熟性，你需要做的就是给对等的实例一个合法的关联serviceurl。

在eureka-server工程中resources文件夹下，创建配置文件application-peer1.yml:

```yaml
server:
  port: 8761

spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2:8769/eureka/
```

并且创建另外一个配置文件application-peer2.yml：

```yaml
server:
  port: 8769

spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
```

按照官方文档的指示，需要改变etc/hosts，windows电脑，在c:/windows/systems/drivers/etc/hosts 修改。linux系统通过vim /etc/hosts ,加上：

> ```xml
> 127.0.0.1 peer1
> 127.0.0.1 peer2
> ```

启动eureka-server：

> ```shell
> java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer1
>java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer2
> ```

启动service-hi:

> ``` shell
> java -jar service-hi-0.0.1-SNAPSHOT.jar - -
> ```

你会发现注册了service-hi，并且有个peer2节点，同理访问localhost:8769你会发现有个peer1节点。

client只向8761注册，但是你打开8769，你也会发现，8769也有 client的注册信息。

Eureka-eserver peer1 8761,Eureka-eserver peer2 8769相互感应，当有服务注册时，两个Eureka-eserver是对等的，它们都存有相同的信息，这就是通过服务器的冗余来增加可靠性，当有一台服务器宕机了，服务并不会终止，因为另一台服务存有相同的数据。

## Hystrix Dashboard（断路器模型）

在pom的工程文件引入相应的依赖：

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        
    </dependencies>
```

!> 在程序的入口ServiceHiApplication类，加上@EnableHystrix注解开启断路器，这个是必须的，并且需要在程序中声明断路点HystrixCommand(方法上面)；加上@EnableHystrixDashboard注解（类上），开启HystrixDashboard

### 图形展示

打开http://localhost:8762/actuator/hystrix.stream，可以看到一些具体的数据

打开locahost:8762/hystrix 可以看见界面

## Hystrix Turbine

Hystrix Turbine将每个服务Hystrix Dashboard数据进行了整合。

!> Hystrix Turbine的使用非常简单，只需要引入相应的依赖和加上注解和配置就可以了。

#### service-turbine

引入相应的依赖：

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
        </dependency>

    </dependencies>
```

在其入口类ServiceTurbineApplication加上注解@EnableTurbine，开启turbine，@EnableTurbine注解包含了@EnableDiscoveryClient注解，即开启了注册服务。

配置文件application.yml：

```yaml
server:
  port: 8764

spring:
  application:
    name: service-turbine

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
management:
  endpoints:
    web:
      exposure:
        include: "*"
      cors:
        allowed-origins: "*"
        allowed-methods: "*"

turbine:
  app-config: service-hi,service-lucy
  aggregator:
    clusterConfig: default
  clusterNameExpression: new String("default")
  combine-host: true
  instanceUrlSuffix:
    default: actuator/hystrix.stream     
```

## Nacos

Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

### 使用Nacos作为服务注册中心注册

**Nacos 的关键特性包括:**

- 服务发现和服务健康监测
- 动态配置服务，带管理界面，支持丰富的配置维度。
- 动态 DNS 服务
- 服务及其元数据管理

Nacos依赖于Java环境，所以必须安装Java环境。然后从官网下载Nacos的解压包，安装稳定版的

下载完成后，解压，在解压后的文件的/bin目录下，windows系统点击startup.cmd就可以启动nacos。linux或mac执行以下命令启动nacos

> ```shell
> sh startup.sh -m standalone
> ```

启动成功，在浏览器上访问：http://localhost:8848/nacos 会跳转到登陆界面，默认的登陆用户名为nacos，密码也为nacos。



!> 在pom文件引入nacos的Spring Cloud起步依赖

!> 在工程的配置文件application.yml做相关的配置

!> 然后在Spring Boot的启动文件`NacosProviderApplication`加上`@EnableDiscoveryClient`注解.

nacos作为服务注册和发现组件时，在进行服务消费，可以选择`RestTemplate`和`Feign`等方式。这和使用Eureka和Consul作为服务注册和发现的组件是一样的，没有什么区别。这是因为spring-cloud-starter-alibaba-nacos-discovery依赖实现了Spring Cloud服务注册和发现的相关接口，可以和其他服务注册发现组件无缝切换。

### 使用Nacos作为配置中心

引入nacos-config的Spring cloud依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-alibaba-nacos-config</artifactId>
	<version>0.9.0.RELEASE</version>
</dependency>
```

在bootstrap.yml(一定是bootstrap.yml文件，不是application.yml文件)文件配置以下内容：

```yaml
spring:
  application:
    name: nacos-provider
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml
        prefix: nacos-provider
  profiles:
    active: dev
```













[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)[✨](https://www.emojiall.com/zh-hans/emoji/✨)

以上学习自[方志朋的博客](https://www.fangzhipeng.com/)

我也是看到最后一篇才知道他是《深入理解Spring Cloud与微服务构建》的作者！