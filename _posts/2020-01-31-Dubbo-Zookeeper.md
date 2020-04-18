---
subtitle:   Dubbo知识整合

date:       2020-01-31

author:     JZ

header-img: img/post-bg-swift.jpg

catalog: true

tags:

    - java
    - Dubbo
    - SpringBoot
---

# 安装zookeeper

<https://archive.apache.org/dist/zookeeper/>  路径下选择需要安装的zookeeper版本。 若在windows上使用安装，在zookeeper跟路径下新建data和log两个文件夹， 复制conf文件夹下的zoo_sample.cfg，为zoo.cfg，修改

```
dataDir=../data
dataLogDir=../log
```

分别运行bin 文件夹下的zkServer.cmd与zkCli.cmd

# ZooKeeper概述

ZooKeeper是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于ZooKeeper实现注入数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁和分布式队列等功能。

目前ZooKeeper最常用的使用场景用于**担任生产者和服务消费者的注册中心（提供发布订阅任务）**。服务生产者将自己提供的服务注册到Zookeeper中心，服务的消费者在进行服务调用的时候先到Zookeeper中查找服务，获取到服务生产者的详细信息之后，再去调用服务生产者的内容与数据。

![img](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LhJiyDNbX84FynNFWuE%2F-Lyg3WL3sJKsONJP-0N3%2F-LygAUnMfc7sTbz5oj_R%2Fimage.png?alt=media&token=df547a23-826c-4123-a7d6-7b1d6cf4e6a1)

Zookeeper充当服务注册中心

## 为什么使用奇数台服务器构成集群

所谓Zookeeper容错是指，当宕掉几个Zookeeper服务器之后，剩下的个数必须大于宕掉的服务器的个数这样整个集群才是可用的，那么即可使用的服务器数量必须大于n/2, 而， n与n-1的容忍度是一样的。

## 关于ZooKeeper的概念

### 重要概念总结

- ZooKeeper本身就是一个分布式程序
- 为了保证高可用，最好是以集群的形态部署ZooKeeper，这样只要集群中大部分机器是可用的， 那么ZooKeeper本身仍然可使用
- ZooKeeper将数据保存在内存当中，这也保证了高吞吐量和低延迟（但是内存限制了其存储容量的大小）
- ZooKeeper是高性能的。在“读”多于“写”的应用程序中尤其的高性能，因为“写”会导致所有的服务器同步状态
- ZooKeeper有临时节点的概念，当创建临时节点的客户端一直保持活动，瞬间节点就一直存在。当绘画终结时，瞬间节点被删除。持久节点是指一旦这个ZNode被创建了，除非主动进行ZNode的移除操作。否则这个结点会一直存在。

<https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/ZooKeeper>

# 安装Dubbo‌

下载Dubbo-admin的git仓库[<https://github.com/apache/dubbo-admin>]的(注意切换成master分支)； 下载到本地之后，修改 `dubbo-admin/src/main/resources/application.properties`

中的`dubbo.registry.address` 为自己所使用的注册中心的地址， 修改完之后使用 `mvn clean install` ， 生成对应的jar包， 并且使用 `java  -jar jar's name`  运行jar包。 此时Dubbo的基本运行环境已经搭建完成。

### Dubbo 的三大核心功能

- 面像接口的远程调用
- 智能容错和负载均衡
- 服务的自动注册和发现



## 分布式和微服务

### 集群模式

集群处理到达瓶颈的时候，将单机上的服务复制到其余的服务器上，形成了“集群”， 所有的服务器为一个节点。每个节点都提供相同的服务，系统的处理能力有所增强。

集群要解决的问题为，使用哪个节点来处理，使得每个节点的压力都比较平均。要实现这个功能就要在增加一个调度者的角色，用户的所有请求都先交给他，他根据当前的所有节点的负载情况，决定这个请求交由那个节点处理，该调度者的名字被称为--负载均衡服务器

集群结构的好处就是系统扩展非常容易，添加节点就可以增加处理能力，但是当业务发展到一定的程度的时候，整个的性能随着处理器的增加提升效果并不明显，此时需要使用微服务结构。

### 微服务结构

微服务就是将一个完整的系统，按照业务功能，拆分成一个个独立的子系统，在微服务结构中，每个子系统被称为“服务”。这些子系统独立运行在web容器当中，之间通过RPC（Remote Process Call）远程服务调用。通过RPC方式调用的好处：

1. 降低系统之间的耦合性，系统与系统之间的边界明确，容易排查问题，开发效率高；
2. 系统更容易扩展，可以针对行的扩展某些服务。可以针对性的提升部分业务的服务器数量
3. 服务的复用性更搞，相同业务可以重复使用，无需重复开发。



# Spring Boot 集成Dubbo

## 生产者

- 添加依赖

exit: Ctrl+↩

```xml
<dependency>
    <groupId>com.alibaba.spring.boot</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
```

- 在properties中添加dubbo相关信息

```properties
spring.application.name=dubbo-spring-boot-starter
spring.dubbo.service=true
spring.dubbo.registry=N/A
```

‌

- 在SpringBootApplication上添加 `@EnableDubboConfiguration` , 表示开启dubbo功能，（dubbo provider服务可以不使用web容器）

```java
@SpringBootApplication
@EnableDubboConfiguration
public class ProviderDubbo{
    //
}
```



- 编写dubbo服务， 只需要在要发布的服务上添加`@Service`(com.alibaba.dubbo.config.annotation.service) 注解， 其中interfaceClass是需要发布的服务接口

```java
@Service(interfaceClass=HelloService.class)
@Component
public class HelloServiceImpl implements HelloService{
    //
}
```

## 消费者

- 依赖添加

```xml
<dependency>
    <groupId>com.alibaba.spring.boot</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
```

- 在properties中添加dubbo相关信息

```properties
spring.application.name=dubbo-spring-boot-starter
```

- 开启@EnableDubboConfiguration

```java
@SpringBootApplication
@EnableDubboConfiguration
public class ConsumerDubbo{
    //
}
```

- 通过 `@Reference`  注入需要使用的interface

```java
@Component
public class HelloConsumer{
    @Reference(url="dubbo://127.0.0.1:20880")
    private HelloService helloService;
}
```
<<<<<<< HEAD



## Dubbo框架构成

### 角色组成

- provider: 暴露服务的服务提供方
- consumer：调用远程服务的消费方
- registry：服务注册与发现的注册中心
- monitor：统计服务的调用次数和调用时间的监控中心
- container：服务运行的容器
=======
### 负载均衡策略
在集群负载均衡时， Dubbo提供多种均衡策略，默认为 `Random` 随机调用
#### Random LoadBalance
为每台能提供相同服务的服务器配置权重，根据权重可大约算出请求落在每台服务器上的概率
#### RoundRobin LoadBalance（轮询， 不推荐）
存在慢的提供者累计请求的问题，比如： 第二台服务器正常运转，但是其处理请求速度较慢，久而久之会造成大量的请求在第二台服务器上等待的情况
#### LeastActive LoadBalance
- 最少活跃调用书，相同活跃的随机
- 慢的提供则收到的更少的请求，因为越慢的调用前后计数差会越大
>>>>>>> 2aebe58ab0a6f4a74da5e92ce4fd6feb9aa783ac

