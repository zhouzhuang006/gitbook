# Dubbo 从入门到放弃

### 背景

换了个工作，新公司使用的是Dubbo框架， 以前用的一直是Spring Cloud。对于Dubbo  只会拼写和发音， 决定简单了解一下入个门~



# 介绍

### 官网地址

http://dubbo.apache.org/en-us/

Apache Dubbo 是Apache 的顶级项目，什么是Dubbo 呢？ 且看官网介绍

```
Apache Dubbo™ is a high-performance, java based open source RPC framework.
```

原来如此 Apache Dubbo 是一个高可用的框架， 基于java的开源RPC框架， 那么问题来了， 什么是RPC?

```
RPC是远程过程调用（Remote Procedure Call）的缩写

简单来说就是两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。
```

对于RPC不理解的小伙伴， 请看 [什么是rpc框架](https://www.jianshu.com/p/3d2c2515f385)

### Dubbo定位

```
A high performance Java RPC framework

Apache Dubbo |ˈdʌbəʊ| is a high-performance, light weight, java based RPC framework. Dubbo offers three key functionalities, which include interface based remote call, fault tolerance & load balancing, and automatic service registration & discovery.
```

原来Dubbo 是一个高性能的Java RPC 框架

Dubbo 提供三个关键功能：远程调用、容错性、负载均衡，还有服务自动自注册发现

### 原理图

<img src="architecture.png" alt="architecture" style="zoom: 50%;" />

####  节点角色规范

| 节点        | 角色规格                       |
| ----------- | ------------------------------ |
| `Provider`  | 提供者公开远程服务             |
| `Consumer`  | 消费者致电远程服务             |
| `Registry`  | 注册表负责服务发现和配置       |
| `Monitor`   | 监视器计算服务调用的数量和耗时 |
| `Container` | 容器管理服务的生命周期         |

#### 服务关系

1. `Container`负责启动，加载和运行服务`Provider`。
2. `Provider``Register`在启动时向其注册服务。
3. `Consumer`从`Register`启动时开始订阅所需的服务。
4. `Register`将`Provider`s列表返回到`Consumer`，更改时，`Register`将`Consumer`通过长连接将更改后的数据推送到。
5. `Consumer``Provider`根据软负载平衡算法选择s 之一并执行调用，如果失败，它将选择另一个`Provider`。
6. 两者`Consumer`和`Provider`都会计算内存中调用服务的次数和耗时，并将统计信息发送到`Monitor`每分钟。

### 特性功能

Dubbo具有以下功能：连接性，鲁棒性，可伸缩性和可升级性。

#### 连接性

- `Register`负责注册和搜索服务地址（例如目录服务），`Provider`并且`Consumer`仅在启动期间与注册表交互，并且注册表不会转发请求，因此压力较小
- “监视器”负责计算服务调用的数量和耗时，统计信息将首先在`Provider`和`Consumer`的内存中汇总，然后发送到`Monitor`
- “提供商”将服务注册到“注册”，并将耗时的统计信息（不包括网络开销）报告给“监控器”
- “消费者”从中获取服务提供商的地址列表，`Registry`根据LB算法直接致电提供商，向上报耗时的统计信息`Monitor`，其中包括网络开销
- 之间的连接`Register`，`Provider`并且`Consumer`是长连接，`Moniter`是一个例外
- `Register``Provider`通过长连接意识到存在，当断开连接时`Provider`，`Register`会将事件推送到`Consumer`
- 它不影响已经运行的实例`Provider`和`Consumer`甚至所有`Register`和`Monitor`趴下，因为`Consumer`得到的缓存`Provider`上榜
- `Register`并且`Monitor`是可选的，`Consumer`可以`Provider`直接连接

#### 坚固性

- `Monitor`的停机时间不会影响使用情况，只会丢失一些采样数据
- 当数据库服务器关闭时，`Register`可以通过检查其缓存将服务`Provider`列表返回到`Consumer`，但是新服务器`Provider`无法注册任何服务
- `Register` 是一个对等集群，当任何实例出现故障时，它将自动切换到另一个集群
- 即使所有`Register`实例都发生故障，`Provider`并且`Consumer`仍然可以通过检查其本地缓存来进行通信
- 服务`Provider`是无状态的，一个实例的停机时间不会影响使用情况
- `Provider`一个服务的所有s故障后，`Consumer`无法使用该服务，并无限地重新连接以等待服务`Provider`恢复

#### 可扩展性

- `Register` 是一个可以动态增加其实例的对等群集，所有客户端将自动发现新实例。
- `Provider`是无状态的，它可以动态地增加部署实例，并且注册表会将新的服务提供者信息推送到`Consumer`。

#### 可升级性

当服务集群进一步扩展并且IT治理结构进一步升级时，需要动态部署，并且当前的分布式服务体系结构不会带来阻力。这是未来可能的架构：

![达博建筑的未来](dubbo-architecture-future.jpg)

#### 节点角色规范

| 节点         | 角色规格                                       |
| ------------ | ---------------------------------------------- |
| `Deployer`   | 用于自动服务部署的本地代理                     |
| `Repository` | 该存储库用于存储应用程序包                     |
| `Scheduler`  | 调度程序会根据访问压力自动增加或减少服务提供商 |
| `Admin`      | 统一管理控制台                                 |
| `Registry`   | 注册表负责服务发现和配置                       |
| `Monitor`    | 监控器计算服务呼叫时间和时间                   |

### 功能列表

#### 基于透明接口的RPC


Dubbo提供了基于接口的高性能RPC，对用户是透明的。

#### 智能负载平衡


Dubbo支持多种现成的负载平衡策略，可以感知下游服务状态，从而减少总体延迟并提高系统吞吐量。

#### 自动服务注册和发现


Dubbo支持多个服务注册中心，可以立即在线/离线检测服务。

#### 高扩展性


Dubbo的微内核和插件设计确保了它可以通过第三方实现轻松地跨核心功能（如协议、传输和序列化）进行扩展。

#### 运行时流量路由


Dubbo可以在运行时进行配置，以便流量可以根据不同的规则进行路由，这使得它很容易支持蓝绿色部署、数据中心感知路由等功能。

#### 可视化服务治理


Dubbo为服务治理和维护提供了丰富的工具，例如查询服务元数据、运行状况和统计信息。



# 快速开始

## 用法示例

### Spring本地服务配置

local.xml：

```xml
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” />
<bean id=“xxxAction” class=“com.xxx.XxxAction”>
    <property name=“xxxService” ref=“xxxService” />
</bean>
```

### 远程服务的春季配置

可以根据本地配置进行很少的更改即可完成远程配置：

- 分为`local.xml`两部分，将服务定义部分放入`remote-privider.xml`（存在于提供者节点中），同时将参考部分放入`remote-consumer.xml`（存在于使用者节点中）。
- 添加``到提供者的配置，以及``使用者的配置。

remote-provider.xml：

```xml
<!-- define remote service bean the same way as local service bean -->
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” /> 
<!-- expose the remote service -->
<dubbo:service interface=“com.xxx.XxxService” ref=“xxxService” /> 
```

remote-consumer.xml：

```xml
<!-- reference the remote service -->
<dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” />
<!-- use remote service the same say as local service -->
<bean id=“xxxAction” class=“com.xxx.XxxAction”> 
    <property name=“xxxService” ref=“xxxService” />
</bean>
```



## 快速开始

使用Dubbo的最常见方法是在Spring框架中运行它。以下内容将指导您使用Spring框架的[XML配置](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/xsd-configuration.html)开发Dubbo应用程序。

如果您不想依赖Spring，可以尝试使用[API配置](http://dubbo.apache.org/en-us/docs/user/configuration/api.html)。

首先，我们创建一个名为dubbo-demo的根目录：

```
mkdir dubbo-demo
cd dubbo-demo
```

接下来，我们将在根目录下创建3个子目录：

- dubbo-demo-api：通用服务api
- dubbo-demo-provider：演示提供者代码
- dubbo-demo-consumer：演示消费者代码

## 服务提供者

### 定义服务接口

DemoService.java [[1\]](http://dubbo.apache.org/en-us/docs/user/quick-start.html#fn1)：

```java
package org.apache.dubbo.demo;

public interface DemoService {
    String sayHello(String name);

}
```

项目结构应如下所示：

```
.
├── dubbo-demo-api
│   ├── pom.xml
│   └── src
│       └── main
│           └── java
│               └── org
│                   └── apache
│                       └── dubbo
│                           └── demo
│                               └── DemoService.java
```

### 在服务提供商中实现接口

DemoServiceImpl.java [[2\]](http://dubbo.apache.org/en-us/docs/user/quick-start.html#fn2)：

```java
package org.apache.dubbo.demo.provider;
import org.apache.dubbo.demo.DemoService;

public class DemoServiceImpl implements DemoService {
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```

### 使用Spring配置公开服务

provider.xml：

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- provider's application name, used for tracing dependency relationship -->
    <dubbo:application name="demo-provider"/>
    <!-- use multicast registry center to export service -->
    <dubbo:registry address="multicast://224.5.6.7:1234"/>
    <!-- use dubbo protocol to export service on port 20880 -->
    <dubbo:protocol name="dubbo" port="20880"/>
    <!-- service implementation, as same as regular local bean -->
    <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl"/>
    <!-- declare the service interface to be exported -->
    <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService"/>
</beans>
```

该演示使用多播作为注册表，因为它很简单并且不需要额外的安装。如果您喜欢像Zookeeper这样的机构，请[在此处](https://github.com/dubbo/dubbo-samples)查看示例。

### 配置日志记录系统

默认情况下，Dubbo使用log4j作为日志记录系统，它还支持slf4j，Apache Commons Logging和JUL日志记录。

以下是一个示例配置：

log4j.properties

```
###set log levels###
log4j.rootLogger=info, stdout
###output to the console###
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%d{dd/MM/yy hh:mm:ss:sss z}] %t %5p %c{2}: %m%n
```

### 引导服务提供商

Provider.java

```java
package org.apache.dubbo.demo.provider;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Provider {

    public static void main(String[] args) throws Exception {
        System.setProperty("java.net.preferIPv4Stack", "true");
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/dubbo-demo-provider.xml"});
        context.start();
        System.out.println("Provider started.");
        System.in.read(); // press any key to exit
    }
}
```

最后，项目结构应如下所示：

```
├── dubbo-demo-provider
│   ├── pom.xml
│   └── src
│       └── main
│           ├── java
│           │   └── org
│           │       └── apache
│           │           └── dubbo
│           │               └── demo
│           │                   └── provider
│           │                       ├── DemoServiceImpl.java
│           │                       └── Provider.java
│           └── resources
│               ├── META-INF
│               │   └── spring
│               │       └── dubbo-demo-provider.xml
│               └── log4j.properties
```

## 服务消费者

完成安装步骤，请参阅：[消费者演示安装](http://dubbo.apache.org/en-us/docs/admin/install/consumer-demo.html)

### 使用Spring配置引用远程服务

Consumer.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- consumer's application name, used for tracing dependency relationship (not a matching criterion),
    don't set it same as provider -->
    <dubbo:application name="demo-consumer"/>
    <!-- use multicast registry center to discover service -->
    <dubbo:registry address="multicast://224.5.6.7:1234"/>
    <!-- generate proxy for the remote service, then demoService can be used in the same way as the
    local regular interface -->
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.demo.DemoService"/>
</beans>
```

### 引导消费者

Consumer.java [[3\]](http://dubbo.apache.org/en-us/docs/user/quick-start.html#fn3)：

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.apache.dubbo.demo.DemoService;
 
public class Consumer {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"META-INF/spring/dubbo-demo-consumer.xml"});
        context.start();
        // Obtaining a remote service proxy
        DemoService demoService = (DemoService)context.getBean("demoService");
        // Executing remote methods
        String hello = demoService.sayHello("world");
        // Display the call result
        System.out.println(hello);
    }
}
```

### 配置日志记录系统

这与在提供商端进行配置的方式相同。

最后，项目结构应如下所示：

```
├── dubbo-demo-consumer
│   ├── pom.xml
│   └── src
│       └── main
│           ├── java
│           │   └── org
│           │       └── apache
│           │           └── dubbo
│           │               └── demo
│           │                   └── consumer
│           │                       └── Consumer.java
│           └── resources
│               ├── META-INF
│               │   └── spring
│               │       └── dubbo-demo-consumer.xml
│               └── log4j.properties
```

## 开始演示

### 启动服务提供商

运行`org.apache.dubbo.demo.provider.Provider`该类以启动提供程序。

### 开始服务消费者

运行`org.apache.dubbo.demo.provider.Consumer`该类以启动使用者，您应该能够看到以下结果：

```
Hello world
```

## 完整的例子

您可以在Github存储库中找到完整的示例代码。

- [提供者演示](http://dubbo.apache.org/en-us/docs/admin/install/provider-demo.html)
- [消费者演示](http://dubbo.apache.org/en-us/docs/admin/install/consumer-demo.html)





参考文档：

http://dubbo.apache.org/en-us/docs/user/quick-start.html