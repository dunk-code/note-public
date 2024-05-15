windows系统下，查看系统进程

```bash
netstat -ano | findstr 8000 
```

根据PID（进程号）寻找进程名

```bash
tasklist | findstr 14688
```

任务管理器杀死进程 

或者

```bash
taskkill /f /t /im javaw.exe；或者taskkill /pid 3672 /F
```

> 微服务架构模式是一种将单个应用程序开发为一组小型服务的方法，每个小型服务都在自己的进程中运行并与轻量级机制（通常是HTTP资源API）进行通信。这些服务围绕业务功能构建，并且可以由全自动部署机制独立部署。这些服务的集中管理几乎没有，它可以用不同的编程语言编写并使用不同的数据存储技术。尽管它们的优势使它们在最近几年变得非常时尚，但它们却带来了分销增加，一致性降低的缺点，并且要求运营管理成熟。 																												-------马丁·福勒

[文章代码](https://github.com/dunk-code/springcloud-learn)

[TOC]

# 文章总览

![](https://gitee.com/zhang-songyao/blog-images/raw/master/20210528104126.png)

# [微服务架构](https://martinfowler.com/articles/microservices.html)

## 单体应用架构

### 什么是单体应用架构

简单的说就是不管啥功能都往一个应用里写，比如电商系统。用户功能、商品功能、订单功能等等，都往一个应用里写

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210523093203.png" alt="img" style="zoom:80%;" />

## 微服务架构

> 微服务架构是一种架构风格，而SpringCloud是实现微服务架构的一系列框架的有序集合

### 微服务

微服务是一个新兴的软件架构，就是把一个大型的单个应用程序和服务拆分为数十个的支持微服务。一个微服务的策略可以让工作变得更为简便，它可扩展单个组件而不是整个的应用程序堆栈，从而满足服务等级协议

简而言之，微服务架构风格，就像是把一个单独的应用程序开发为一套小服务，每个小服务运行在自己的进程中，并使用轻量级机制通信，通常是 HTTP API。这些服务围绕业务能力来构建，并通过完全自动化部署机制来独立部署。这些服务使用不同的编程语言书写，以及不同数据存储技术，并保持最低限度的集中式管理

### 什么是微服务架构

微服务架构是一种架构模式，它提倡将单一应用程序划分成一组小的服务，服务之间互相协调、互相配合，为用户提供最终价值，一个大型复杂软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅专注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。如果在大规模系统协作下，如何保持系统的稳定性、可快速迭代性就变得非常紧迫。微服务架构便是在这样的历史背景下诞生，提供了一套大型项目开发的解决方案。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210523093214.png" alt="img" style="zoom:80%;" />

### 使用微服务架构的优势和劣势

#### 优势

- 服务的独立部署：每个服务都是一个独立的项目，可以独立部署，不依赖与其他服务，耦合性低
- 服务快速启动：拆分之后服务启动速冻必然比拆分之前快很多，因为以来的库少了，代码量也少了
- 更加适合敏捷开发：敏捷开发以用户的需求进化为核心，采用迭代、循序渐进的方式进行。服务拆分可以快速发布新版本，修改那个版本只需要发布对应的服务即可，不用整体发布
- 职责专一：由专门的团队负责专门的服务，业务发展迅速，研发人员也会越来越多，每个团队可以负责对应的业务线，服务拆分有利于团队之间的分工
- 服务可以按需动态扩容：当某个服务的访问量较大时，只需要将这个服务扩容即可
- 代码复用：每个服务都可以提供REST API，所有的基础服务都必须抽出来，很多底层实现都可以以接口方式提供

#### 劣势

- 分布式部署，调用复杂性高：单体应用的时候，所有模块之前调用都是在本地进行的，在微服务中，每个模块独立部署，通过HTTP来进行通信这当中会产生很多问题：网络问题、容错问题、调用关系
- 独立的数据库，分布式事务的挑战：每个微服务都有自己的数据库，这就是所谓的去中心化的数据管理。这种模式的优点在于不同服务，可以选择适合业务自身的数据，比如：订单服务可以使用MySQL、评论服务可以使用MongoDB、商品搜索服务可以用Elasticsearch，缺点就是事务问题了。目前最理想的解决方案就是柔性事务中的最终一致性
- 测试难度提升：服务和服务之间通过接口交互，当接口改变时，对所有的调用方法都有影响，这是自动化测试就非常重要了，否则人工一个一个测试，工作量就太大了
- 运维难度的提升：传统的单体中，我们可能只需要关注一个Tomcat的集群、一个MySQL的集群就可以，但这在微服务架构下是行不通的，当业务增加时，服务也将越来越多，服务的部署、监控变得非常负责，这个时候对于运维的要求就高了

## 微服务架构的四个核心问题

1. 服务很多，客户端怎么访问
2. 这么多服务，服务之间如何通信
3. 这么多服务，如何治理
4. 服务挂了怎么办

解决方案

SpringCloud生态

- SpringCloud NetFlix（一站式解决方案）
- Apache Dubbo Zookeeper（半自动，需要整合）
- Spring Cloud Alibaba（最新的一站式）

## 微服务的技术栈

|               微服务条目               |                          落地技术                           |
| :------------------------------------: | :---------------------------------------------------------: |
|                服务开发                |                SpringBoot、Spring、SpringMVC                |
|             服务配置与管理             |           Netflix公司的Archaius、阿里的Diamind等            |
|             服务注册与发现             |                 Eureka、Consul、Zookeeper等                 |
|                服务调用                |                       Rest、RPC、gRPC                       |
|               服务熔断器               |                      Hystrix、Envoy等                       |
|                负载均衡                |                       Ribbon、Nginx等                       |
| 服务接口调用(客户端调用服务的简化工具) |                           Feign等                           |
|                消息队列                |                 Kafka、RabbitMQ、ActiveMQ等                 |
|            服务配置中心管理            |                 SpringCloud Config、Chef等                  |
|           服务路由(API网关)            |                           Zuul等                            |
|                服务监控                |             Zabbix、Nagios、Metrics、Spectator              |
|               全链路追踪               |                   Zipkin、Brave、Dapper等                   |
|                服务部署                |               Docker、OpernStack、Kubernetes                |
|            数据流操作开发包            | SpringCloud Stream(封装与Redis,Rabbit,Kafaka等发送接收消息) |
|              事件消息总线              |                      Spring Cloud Bus                       |

## 当前各大IT公司用的微服务架构有哪些？

- 阿里：dubbo+HFS
- 京东：JSF
- 新浪：Motan
- 当当网 DubboxX

## 各微服务框架对比

| 功能点/服务框架 |                     Netflix/SpringCloud                      |                            Motan                             |           gRPC            |  Thrift  |            Dubbo/DubboX             |
| --------------- | :----------------------------------------------------------: | :----------------------------------------------------------: | :-----------------------: | :------: | :---------------------------------: |
| 功能定位        |                       完整的微服务框架                       | RPC框架，但整合了ZK或Consul，实现集群环境的基本服务注册/发现 |          RPC框架          | RPC框架  |              服务框架               |
| 支持Rest        |             是，Ribbon支持多种可插拔的序列化选择             |                              否                              |            否             |    否    |                 否                  |
| 支持RPC         |                              否                              |                        是（Hession2）                        |            是             |    是    |                 是                  |
| 支持多语言      |                       是（Rest形式）？                       |                              否                              |            是             |    是    |                 否                  |
| 负载均衡        | 是（服务端zuul+客户端Ribbon），zuul-服务，动态路由，云端负载均衡Eureka（针对中间层服务器） |                         是（客户端）                         |            否             |    否    |            是（客户端）             |
| 配置服务        |     Netfix Archaius，Spring Cloud Config Server集中配置      |                     是（zookeeper提供）                      |            否             |    否    |                 否                  |
| 服务调用链监控  |            是（zuul），zuul提供边缘服务，API网关             |                              否                              |            否             |    否    |                 否                  |
| 高可用/容错     |               是（服务端Hystrix+客户端Ribbon）               |                         是（客户端）                         |            否             |    否    |            是（客户端）             |
| 典型应用案例    |                           Netflix                            |                             Sina                             |          Google           | Facebook |                                     |
| 社区活跃程度    |                              高                              |                             一般                             |            高             |   一般   | 2017年后重新开始维护，之前中断了5年 |
| 学习难度        |                             中等                             |                              低                              |            高             |    高    |                 低                  |
| 文档丰富程度    |                              高                              |                             一般                             |           一般            |   一般   |                 高                  |
| 其他            |      Spring Cloud Bus为我们的应用程序带来了更多管理端点      |                           支持降级                           | Netflix内部在开发集成gRPC | IDL定义  |          实践的公司比较多           |

# SpringCloud概念

## 目前成熟的互联网架构

应用服务化拆分+消息中间件

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210525150147.png" alt="image-20210525150144718" style="zoom:80%;" />

## 为什么需要学习SpringCloud

不论是商业模式和还是用户应用，在业务初期都很简单，我们通常会把它实现为单体结构应用。但是随着业务逐渐发展，产品思想会变得越来越复杂，单体结构的应用会越来越复杂，这就会给应用带来以下问题

- 代码结构混乱：业务复杂，导致代码量大，管理会越来越难。同时，这也会给业务的快速迭代带来巨大挑战
- 开发效率变低：开发人员同时开发一套代码，很难避免代码冲突。开发过程会伴随着不断解决冲突的过程，这会严重影响开发效率
- 排查问题成本高：线上业务发现bug，修复bug的过程可能很简单。但是，由于只有一套代码，需要重新编译、打包、上线、成本很高

由于单体结构的应用随着系统复杂度的增高，会暴露出各种各样的问题。近些年，微服务架构逐渐取代了单体架构，且这种趋势会越来越流行。SpringCloud是目前最常用的微服务开发框架，已经在企业开发中大量应用

## 什么是SpringCloud

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210524162954.svg" style="zoom:80%;" />

SpringCloud是一系列框架的有序集合。它利用SpringBoot的开发便利性巧妙的化简了分布式系统基础设施的开发，如服务发现注册、配置中心、智能路由、消息总线、负载均衡、断路器、数据监控等，都可以使用SpringBoot的开发风格做到一键启动和部署，SpringCloud并没有重复制造轮子，它只是将各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行在封装，屏蔽掉了复杂的配置和实验原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包

## SpringCloud的设计目标

协调各个微服务，简化分布式系统开发

微服务的框架：Dubbo、Kubernetes。

### 特征

- 分布式/版本化配置
- 服务注册与发现
- 路由
- 服务到服务的通话
- 负载均衡
- 断路器
- 分布式锁
- 领头选举和集群状态
- 分布式消息传递

### 优点：

- 产出与Spring家族，Spring在企业级开发框架中比例很大，可以保证后续的更新
- 组件丰富，功能齐全，SpringCloud为微服务架构提供了非常完整的支持，例如：配置管理、服务发现、断路器、微服务网关等
- Spring Cloud 社区活跃度很高，教程很丰富，遇到问题很容易找到解决方案
- 服务拆分粒度更细，耦合度比较低，有利于资源重复利用，有利于提高开发效率
- 可以更精准的制定优化服务方案，提高系统的可维护性
- 减轻团队的成本，可以并行开发，不用关注其他人怎么开发，先关注自己的开发
- 微服务可以是跨平台的，可以用任何一种语言开发
- 适于互联网时代，产品迭代周期更短

### 缺点：

- 微服务过多，治理成本高，不利于维护系统
- 分布式系统开发的成本高（容错，分布式事务等）对团队挑战大

> 总的来说优点大过于缺点，目前看来Spring Cloud是一套非常完善的分布式框架，目前很多企业开始用微服务、Spring Cloud的优势是显而易见的。因此对于想研究微服务架构的同学来说，学习Spring Cloud是一个不错的选择。

## SpringCloud的发展前景

Spring Cloud对于中小型互联网公司来说是一种福音，因为这类公司往往没有实力或者没有足够的资金投入去开发自己的分布式系统基础设施，使用Spring Cloud一站式解决方案能在从容应对业务发展的同时大大减少开发成本。同时，随着近几年微服务架构和Docker容器概念的火爆，也会让Spring Cloud在未来越来越“云”化的软件开发风格中立有一席之地，尤其是在五花八门的分布式解决方案中提供了标准化的、全站式的技术方案，意义可能会堪比当年Servlet规范的诞生，有效推进服务端软件系统技术水平的进步

## 扩展问题

### SpringCloud和Dubbo的区别

|              |     Dubbo     |         Spring Cloud         |
| :----------: | :-----------: | :--------------------------: |
| 服务注册中心 |   Zookeeper   | Spring Cloud Netflix Eureka  |
| 服务调用方式 |      RPC      |           REST API           |
|   服务监控   | Dubbo-monitor |      Spring Boot Admin       |
|    断路器    |    不完善     | Spring Cloud Netflix Hystrix |
|   服务网关   |      无       |  Spring Cloud Netflix Zuul   |
|  分布式配置  |      无       |     Spring Cloud Config      |
|   服务跟踪   |      无       |     Spring Cloud Sleuth      |
|   消息总线   |      无       |       Spring Cloud Bus       |
|    数据流    |      无       |     Spring Cloud Stream      |
|   批量任务   |      无       |      Spring Cloud Task       |

- Dubbo只真针对与服务治理，相当于SpringCloud的一个子集，SpringCloud涉及方方面面，SpringCloud中各种组件应有尽有
- 性能问题，Dubbo优于SpringCloud，Dubbo基于Netty的TCP及二进制数据传输；SpringCloud基于HTTP，HTTP每次都要创立连接，传输的是文本内容，性能损耗大
- SpringCloud支持断路器，与git完美集成配置文件支持版本控制，事物总线实现配置文件的更新与服务自动装配等等一系列的微服务架构要素

- Spring Cloud抛弃了RPC通讯，采用基于HTTP的REST方式。Spring Cloud牺牲了服务调用的性能，但是同时也避免了原生RPC带来的问题。REST比RPC更为灵活，不存在代码级别的强依赖，在强调快速演化的微服务环境下，显然更合适。
- Dubbo像组装机，Spring Cloud像一体机
- 社区的支持与力度：Dubbo曾经停运了5年，虽然重启了，但是对于技术发展的新需求，还是需要开发者自行去拓展，对于中小型公司，显然显得比较费时费力，也不一定有强大的实力去修改源码

### SpringBoot和SpringCloud的区别

- SpringBoot专注于快速方便的开发单个个体微服务
- SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整和并管理起来，为各个微服务之间提供配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务
- SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud不能离开SpringBoot，属于依赖的关系
- SpringBoot专注于快速、方便的开发单体微服务个体；SpringCloud关注全局的服务治理框架

# SpringCloud基础

> SpringCloud的子项目，大致可以分为两类，一类是对现有成熟框架“SpringBoot化”的封装和抽象，也是数量最多的项目，第二类是开发了一部分分布式系统的基础设施，如SpringCloud Stream扮演的就是Kafka，ActiveMQ这样的角色

## SpringCloud Netflix

Netflix OSS 开源组件集成，包括Eureka、Hystrix、Ribbon、Feign、Zuul等核心组件。

- Eureka：服务治理组件，包括服务端的注册中心和客户端的服务发现机制
- Ribbon：负载均衡的服务调用组件，具有多种负载均衡调用策略
- Hystrix：服务容错组件，实现了断路器模式，为依赖服务的出错和延迟提供了容错能力
- Feign：基于Ribbon和Hystrix的声名式服务调用组件
- Zuul：API网关组件，对组件提供路由及过滤功能

### [REST](https://blog.csdn.net/wangjj_016/article/details/3615948?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162192682016780366570393%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162192682016780366570393&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~top_positive~default-1-3615948.nonecase&utm_term=rest&spm=1018.2226.3001.4450)

#### REST是什么

REST的英文是Representational State Transfer，中文翻译”表述性状态转移“，一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件，是所有Web应用都应该遵守的架构设计指导原则。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

##### REST的关键原则

- Resource（资源）：表示为所有事务定义ID
- Hypertext Driver（超文本驱动）：表示所有事务链接在一起
- Uniform Interface（统一接口）：表示使用标准的方法
- Representation（资源的表述）：表示资源多重表述的方法
- State Transfer（状态转移）：表示无状态通信

#### 使用RESTful操作资源 

【GET】 /users # 查询用户信息列表

【GET】 /users/1001 # 查看某个用户信息

【POST】 /users # 新建用户信息

【PUT】 /users/1001 # 更新用户信息(全部字段)

【PATCH】 /users/1001 # 更新用户信息(部分字段)

【DELETE】 /users/1001 # 删除用户信息

#### [API设计风格基本规则](https://blog.csdn.net/qq_27026603/article/details/82012277?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162192756216780265434505%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162192756216780265434505&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-82012277.first_rank_v2_pc_rank_v29&utm_term=restful%E9%A3%8E%E6%A0%BC&spm=1018.2226.3001.4187)

#### RestTemplate

> RestTemplate是Spring提供的用于访问Rest服务的客户端，RestTemplate提供了多重便捷访问远程HTTP服务的方，能够大大提高客户端的编码效率

##### 创建RestTemplate对象

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:27
 * @description：配置RestTemplate
 */
@Configuration
public class ConfigBean {
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

##### getForObject

url：请求的地址，两种方式，其中一种是字符串，另一种是URI

responseType：返回值类型

uriVariables：有两种方式，其中一种是可变长参数，另一种是Map形式

```java
@Override
@Nullable
public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
			new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables);
}
@Override
@Nullable
public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
			new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables);
}
@Override
@Nullable
public <T> T getForObject(URI url, Class<T> responseType) throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
			new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.GET, requestCallback, responseExtractor);
}
```

##### getForEntity

> 不同与getForObject，getForEntity可以获取返回的状态码、请求头信息，通过getBoby获取响应内容，其余的和getForObject一样

```java
@Override
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables)
		throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
	return nonNull(execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables));
}
@Override
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables)
		throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
	return nonNull(execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables));
}
@Override
public <T> ResponseEntity<T> getForEntity(URI url, Class<T> responseType) throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
	return nonNull(execute(url, HttpMethod.GET, requestCallback, responseExtractor));
}
```

##### PostForObject

> 使用post的方式调用接口

```java
@Override
@Nullable
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType,
		Object... uriVariables) throws RestClientException {
	RequestCallback requestCallback = httpEntityCallback(request, responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
			new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
}
@Override
@Nullable
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType,
		Map<String, ?> uriVariables) throws RestClientException {
	RequestCallback requestCallback = httpEntityCallback(request, responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
			new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
}
@Override
@Nullable
public <T> T postForObject(URI url, @Nullable Object request, Class<T> responseType)
		throws RestClientException {
	RequestCallback requestCallback = httpEntityCallback(request, responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
			new HttpMessageConverterExtractor<>(responseType, getMessageConverters());
	return execute(url, HttpMethod.POST, requestCallback, responseExtractor);
}
```

除了get、post的方法外，RestTemplate还提供了put、delete等操作，还有一个比较使用的exchange操作，可以同时执行get、post、put、delete四种请求方式

### [Eureka](https://github.com/Eureka/Zuul/wiki)

#### 概念

> 服务注册与发现

Eureka是Netflix的一个子模块，也是核心模块之一，Eureka是一个基于REST的服务，用于定位服务，以实现云端中间层服务的发现和故障转移。服务注册和发现对于微服务架构来说是非常重要的，有了服务发现与注册，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了，功能类似于Dubbo的注册中心，ZooKeeper

#### 原理

系统中的服务使用Eureka的客户端连接到Eureka Server并维持心跳连接，这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行，SpringCloud的一些其他模块（例如Zuul）可以通过Eureka Server来发现系统中的其他微服务，并执行相关逻辑

Eureka中包含两大组件

- Eureka server：节点启动后，会在Eureka Server中进行注册， 这样Eureka server中的服务注册表中将会村粗所有可用服务节点的信息，节点信息可以在界面中直观的看到
- Eureka client：用户简化Eureka server的交互，客户端也具备一个内置的、轮询负载算法的负载均衡器，在启动后，将会向Eureka server发送心跳（默认周期为30s），如果Eureka server在多个心跳周期内没有收到某节点的心跳，Eureka server将会从服务注册表中将这个节点移除

三大角色：

- Eureka server：提供服务的注册和发现
- Service Provider：服务提供方将自身的服务注册到Eureka server中，从而使服务消费方能够找到
- Service Consumer：服务方从Eureka server中获取注册服务列表，从而能够小消费服务

#### 什么是自我保护模式

默认情况下，如果Eureka server在一定时间没有收到某个微服务实例的心跳，Eureka server将会注销该实例（90s后），但是当网络分区发生故障，微服务和Eureka server之间无法正常通信，以上行为将变得非常危险，因为微服务本身是健康的，此时不该注销该服务。Eureka server通过自我保护模式来解决这个问题，当Eureka server节点在短时间内丢失过多客户端（可能发生了网络分区故障）这个节点将会进入自我保护模式，一旦进入该模式，Eureka server将会保护注册表中的信息，不在删除节点数据，当网络恢复后，Eureka server自动退出自我保护模式

> 设计哲学：宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例，使用自我保护模式，可以让Eureka集群更加健壮和稳定

在SpringCloud中，可以使用`eureka.server.enable-self-preservation = false`禁用自我保护模式【不推荐关闭自我保护机制】

#### DiscoveryClient

> 使用DiscoveryClient可以获取某个服务的响应信息

```java
@GetMapping("/discovery")
public Object discovery() {
    List<String> services = discoveryClient.getServices();
    System.out.println(services.toString());
    List<ServiceInstance> instances = discoveryClient.getInstances("SPRINGCLOUD-PROVIDER-DEPT-8001");
    for (ServiceInstance instance : instances) {
        System.out.println(instance.getHost() + "\n" +
                instance.getPort() + "\n" +
                instance.getUri() + "\n" +
                instance.getServiceId() + "\n" +
                instance.getInstanceId()+ "\n");
    }
    return this.discoveryClient;
}
```

#### 实例

##### Eureka注册中心

###### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

###### Eureka服务启动类

EnableEurekaServer注解表示开启Eureka Server

```java
@SpringBootApplication
@EnableEurekaServer //Eureka启动类
public class EurekaServer_7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer_7001.class, args);
    }
}
```

###### 配置文件

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: localhost # Eureka服务端的名字
  client:
    fetch-registry: false # 由于注册中心的职责就是维护服务实例，它并不需要去检索服务，所以设置为false
    register-with-eureka: false # 由于该应用为注册中心，所有设置为false代表不向注册中心注册自己

    service-url: # 监控页面地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
```

##### 服务提供者

注册中心已经创建并且启动好后，接下来需要实现将一个服务提供者springcloud-provider-dept-8001注册到Eureka中，并提供一个接口给其他服务调用。

###### 引入依赖

```xml
<!--Eureka-->
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
<!--springBoot-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

###### 服务提供者启动类

EnableEurekaClient表示当前服务是一个Eureka客户端

```java
@EnableEurekaClient //开启Eureka注册
@SpringBootApplication
public class Provider8001 {
    public static void main(String[] args) {
        SpringApplication.run(Provider8001.class, args);
    }
}
```

###### 配置文件

```yml
server:
  port: 8001

# Eureka配置 服务注册地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: springcloud-provider # 定义实例ID格式
    ip-address: true # 采用IP注册
```

###### 编写接口提供服务

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/24 22:36
 * @description：提供服务接口
 */

@RestController
public class DeptController {

    @Autowired
    DeptService deptService;

    @PostMapping("/dept/add/")
    public boolean addDept(Dept dept) {
        return deptService.insert(dept.getDname());
    }

    @GetMapping("/dept/get/{deptno}")
    public Dept queryById(@PathVariable long deptno) {
        return deptService.selectById(deptno);
    }

    @GetMapping("/dept/list")
    public List<Dept> list() {
        return deptService.list();
    }

}

```

##### 服务消费者

创建服务消费者springcloud-consumer-dept-80，消费刚刚创建的接口

###### 配置文件

```yml
server:
  port: 80

spring:
  application:
    name: springcloud-consumer-dept-80
```

###### 创建RestTemplate对象

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:27
 * @description：配置RestTemplate
 */
@Configuration
public class ConfigBean {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

###### 编写Controller消费服务

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:26
 * @description：
 */
@RestController
public class DeptController {

    @Autowired
    RestTemplate restTemplate;

    private static final String URL_PREFIX = "http://localhost:8001/dept";

    @RequestMapping("/dept/consumer/add")
    public boolean addDept(Dept dept) {
        return restTemplate.postForObject(URL_PREFIX + "/add", dept, Boolean.class);
    }

    @RequestMapping("/dept/consumer/get/{deptno}")
    public Dept getById(@PathVariable Long deptno) {
        return restTemplate.getForObject(URL_PREFIX + "/get/" + deptno, Dept.class);
    }

    @RequestMapping("/dept/consumer/list")
    public List<Dept> list() {
        return restTemplate.getForObject(URL_PREFIX + "/list", List.class);
    }

}

```

##### 通过Eureka消费接口

###### 引入依赖

```xml
<!--Ribbon-->
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-ribbon -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.8.RELEASE</version>
</dependency>
<!--Eureka-->
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

###### 配置负载均衡

```java
 /**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:27
 * @description：配置RestTemplate
 */
@Configuration
public class ConfigBean {
     //配置负载均衡实现restTemplate
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

###### 配置文件

```yml
server:
  port: 80

spring:
  application:
    name: springcloud-consumer-dept-80

eureka:
  client:
    # 不向注册中心注册自己
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

###### 通过服务名获取服务

```java
private static final String URL_PREFIX = "http://springcloud-provider-dept/dept";
```

#### CAP原则

- C：数据一致性
- A：服务可用性
- P：服务对网络分区故障的容错性

> 这三个特性在任何分布式系统中都不能同时满足，最多同时满足两个

根据CAP将NoSQL数据库分为满足CA、AP、CP原则三大类

- CA：单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大
- CP：满足一致性，分区容忍性的系统，通常性能不是特别高
- AP：满足可用性，分区容忍性的系统，通常对一致性要求低一些

#### ZooKeeper和Eureka的区别

> ZooKeeper保证的是CP原则

当向注册中心查询服务列表时，我们可以容忍服务直接down掉不可用，但是不能接受注册中心返回的是几分钟以前的注册信息。也就是说，服务注册功能对一致性的要求要高于可用性。但是ZooKeeper会出现这样的一种情况，当master节点因网路故障与其他节点失去联系时，剩余的节点会重新进行leader选举。问题在于，选举leader的时间太长，30~120s，且选举期间整个ZooKeeper集群是都是不可用的，这就导致在选举期间注册服务瘫痪，在云部署的环境下，因网络问题使得ZooKeeper集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

> Eureka保证的是AP原则

Eureka看明白了这一点，因此在设计时就优先保证可用性。**Eureka各个节点都是平等的**，几个节点挂掉不影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册时如果发现连接失败，则会自动切换至其他的节点，只要有一台Eureka还在，就能保证注册服务可用（保证可用性），只不过查到的信息可能不是最新的（不保证一致性）。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

1. Eureka不再从注册列表中移除因为长时间没有收到心跳而应该过期的服务
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上（即保证当前节点依然可用）
3. 当前网络稳定时，当前实例新的注册信息会被同步到其它节点中

因此，Eureka可以很好的应对因网络故障导致节点失去联系的情况，而不会像ZooKeeper那样使整个注册服务瘫痪。

#### Eureka集群搭建

> 使用一台机器模拟Eureka集群搭建

修改文件`C:\Windows\System32\drivers\etc`目录下的hosts文件

**注意：**两个Eureka注册中心的hostname不能相同

```bash
127.0.0.1 	eureka7001.com
127.0.0.1	eureka7002.com
```

springcloud-eureka-server-7001/application.yml配置

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: eureka7001.com # Eureka服务端的名字
  client:
    fetch-registry: false # 由于注册中心的职责就是维护服务实例，它并不需要去检索服务，所以设置为false
    register-with-eureka: false # 由于该应用为注册中心，所有设置为false代表不向注册中心注册自己

    service-url: # 监控页面地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/,http://eureka7002.com:7002/eureka/
spring:
  application:
    name: eureka-server-master
```

springcloud-eureka-server-7002/application.yml配置

```yml
server:
  port: 7002
eureka:
  instance:
    hostname: eureka7002.com # Eureka服务端的名字
  client:
    fetch-registry: false # 由于注册中心的职责就是维护服务实例，它并不需要去检索服务，所以设置为false
    register-with-eureka: false # 由于该应用为注册中心，所有设置为false代表不向注册中心注册自己

    service-url: # 监控页面地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/,http://eureka7001.com:7001/eureka/
spring:
  application:
    name: eureka-server-cluster
```

springcloud-provider-dept-8001/application.yml配置

```yml
server:
  port: 8001
# Eureka配置 服务注册地址
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
  instance:
    # 采用IP注册
    prefer-ip-address: true
    # 定义实例ID格式
    instance-id: ${spring.application.name}:${spring.cloud.client.ipaddress}:${server.port}

```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210526145129.png" alt="image-20210526145125616" style="zoom:80%;" />

### [Ribbon](https://github.com/Ribbon/Zuul/wiki)

> 目前主流的负载方案分为两种：一种是集中是负载均衡，在消费者个服务提供方中间使用独立的代理方式进行负载，有硬件的（比如F5），也有软件的（比如Nginx），另一种则是客户端自己做负载均衡，根据自己的请求情况做负载，Ribbon属于客户端自己做负载

Ribbon是Netflix开源的一款用于客户端负载均衡的工具软件

#### 概念

SpringCloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现

简单的说，Ribbon是Netflix发布的开源项目，其主要功能是提供客户端实现负载均衡算法。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，Ribbon是一个客户端负载均衡器，我们可以在配置文件中列出所有Load Balancer后面的所有机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器，我们也很容易使用Ribbon实现自定义的负载均衡算法

#### 作用

- LB：负载均衡（Load Balance），在微服务或者分布式进群中经常使用的一种应用
- **负载均衡简单说就是将用户的请求平均分摊到多个服务器上，从而实现系统的HA（高可用）**
- 常见的负载均衡有软件Nginx，LVS，硬件F5等
- Dubbo和SpringCloud中均给我们提供了负载均衡，**SpringCloud的负载均衡算法可以自定义**
- 负载均衡的简单分类
  - 集中式LB：即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如Nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方
  - 进程式LB：将LB的逻辑集成到消费方，消费方从服务注册中心获取那些地址可用，然后再从这地址中选出一个合适的服务器；Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址

#### [LVS](https://blog.csdn.net/weixin_40470303/article/details/80541639)

##### 简介

LVS(Linux Virtual Server)即Linux虚拟服务器，是由章文嵩博士主导的开源负载均衡项目，目前LVS已经被集成到Linux内核模块中。该项目在Linux内核中实现了基于IP的数据请求负载均衡调度方案，终端互联网用户从外部访问公司的外部负载均衡服务器，终端用户的Web请求会发送给LVS调度器，调度器根据自己预设的算法决定将该请求发送给后端的某台Web服务器，比如，轮询算法可以将外部的请求平均分发给后端的所有服务器，终端用户访问LVS调度器虽然会被转发到后端真实的服务器，但如果真实服务器连接的是相同的存储，提供的服务也是相同的服务，最终用户不管是访问哪台真实服务器，得到的服务内容都是一样的，整个集群对用户而言都是透明的。根据LVS工作模式的不同，真实服务器会选择不同的方式将用户需要的数据发送到终端用户，LVS工作模式分为NAT模式、TUN模式、以及DR模式。
<img src="https://img-blog.csdn.net/201806012200210" alt="img" style="zoom:80%;" />



#### 使用实现负载均衡

创建三个相同的数据库

创建三个服务提供者，**一定要保证三个服务的application.name一致！！**

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210526180209.png" alt="image-20210526180206566" style="zoom:80%;" />

application.yml文件如下：

```yml
server:
  port: 8002


mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml
  # config-location: classpath:mybatis/mybaits-config.xml
  configuration:
    #下划线转为大写
    map-underscore-to-camel-case: true


spring:
  application:
    name: springcloud-provider-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db_02?useSSL=false&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 333


# Eureka配置 服务注册地址
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
  instance:
    # 采用IP注册
    prefer-ip-address: true
    # 定义实例ID格式
    instance-id: ${spring.application.name}:${spring.cloud.client.ipaddress}:${server.port}
```

在消费者模块`springcloud-consumer-dept-80`的配置类ConfigBean中进行配置负载均衡，只需要在RestTemplate注入IOC的方法上添加一个注解`@LoadBalanced`

```java
 /**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:27
 * @description：配置RestTemplate
 */
@Configuration
public class ConfigBean {
     //配置负载均衡实现restTemplate
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

在消费者模块`springcloud-consumer-dept-80`修改Controller的访问路径，路径不能写死，应该通过服务名来访问

```java
private static final String URL_PREFIX = "http://SPRINGCLOUD-PROVIDER-DEPT/dept";
```

开启三个服务提供者、两个Eureka注册中心和consumer消费者	

测试数据，可以看到，Ribbon和Eureka整合以后，**客户端可以直接根据服务的名称来调用，不用关心IP地址和端口号！**

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210526180614.png" alt="image-20210526180610848" style="zoom:80%;" />



#### 自定义负载均衡算法

Ribbon作为一款客户端负载均衡框架，默认的负载均衡策略是轮询，同时也提供了很多其他策略。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210526200627.png" alt="image-20210526200622465" style="zoom:80%;" />

- AvailabilityFilteringRule：过滤掉那些一直连接失败的且被标记为circuit tripped的后端server，并过滤掉那些高并发的Server或者使用一个AvailablilityPredicate来包含过滤Server的逻辑。其实就会说检查Status里面记录的各个Server的运行状态
- RoundRobinRule：轮询（默认的），轮询index，选择index对应的index
- RandomRule：随机
- RetryRule：对选定的负载均衡策略机上重试机制，也就是说当选定了某个策略进行请求负载时在一个配置时间段内，若选择Server不成功，则一直尝试使用subRule的方式选择一个可用的Server
- BestAvailableRule：选择一个最小并发请求的Server，逐个考察所有的Server，如果Server被标记为错误，则跳过，指导选择最小并发的Server
- WeightedResponseTimeRule：根据响应时间分配一个Weight（权重），响应时间越长，Weight越小，被选中的可能性越低，作用同WeightedResponse，ResponseTimeRuleWeighted后改名为WeightedResponseTimeRule

Ribbon实现负载均衡，最核心的一个注解就是`@LoadBalanced`，负载有一个核心的接口 `IRule`

##### 步骤

我们要自定义Ribbon负载均衡算法，在消费者模块`springcloud-consumer-dept-80`中进行自定义，注意，我们自定义的负载均衡算法，不能和主启动类同级，可以主启动类的上一级

写一个配置类AweiRule，添加注解`Configuration`，进行配置，指定自定义的负载均衡算法

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/26 21:03
 * @description：自定义负载均衡策略入口
 */
@Configuration
public class MyRule {

    @Bean
    public IRule myRule() {
        return new MyRandomRule();
    }
}
```

在主启动类的上一级进一个包myrule，新建一个类MyRandomRule，继承`AbstractLoadBalancerRule`，然后写负载均衡算法

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/26 21:05
 * @description：自定义随机负载均衡策略
 */
public class MyRandomRule extends AbstractLoadBalancerRule {

    private int total = 0;
    private int curIndex = 0;

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {

    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers(); //获取档期啊存活的服务
            List<Server> allList = lb.getAllServers();//获取当前所有的服务

            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }

            if (total > 5) {
                total++;
            } else {
                total = 0;
                curIndex++;
                curIndex = curIndex % upList.size();
            }
            server = upList.get(curIndex);

            if (server == null) {
                Thread.yield();
                continue;
            }
            if (server.isAlive()) {
                return (server);
            }
            server = null;
            Thread.yield();
        }

        return server;

    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }
}
```

在主启动类上，添加注解`@RibbonClient(name="SPIRINTCLOUD-PROVIDER-DEPT",configuration = MyRule.class)`，通过name指定服务的名字，通过configuration来指定自己自定义的Ribbon配置类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:34
 * @description：启动类
 */
@EnableEurekaClient
@SpringBootApplication
// 在微服务启动的时候，就能够去去加载我们自定义的Ribbon类
@RibbonClient(name = "SPRINGCLOUD-PROVIDER-DEPT", configuration = MyRule.class)
public class Consumer80 {
    public static void main(String[] args) {
        SpringApplication.run(Consumer80.class, args);
    }
}
```

#### 负载均衡的意义

在计算中，负载均衡可以改善跨计算机、计算机集群、网络连接、中央处理单元或磁盘驱动器等多种计算资源的工作负载分布。负载均衡旨在优化资源的使用，最大化吞吐量，最小化响应时间并避免任何单一资源的过载。使用多个组件进行负载平衡而不是单个组件，可能会通过冗余来提高可靠性和可用性。负载均衡通常涉及专用软件或者硬件，例如多层交换机或域名系统服务器进程

### [Feign](https://github.com/Ribbon/Feign/wiki)

#### 概念

Feign是声明式的Web Service客户端，它让微服务之间调用变得更简单，类似controller调用service，SpringCloud继承了Feign和Eureka，可以在使用Feign提供负载均衡的HTTP客户端

> Feign，主要是社区，大家都习惯面向接口编程，这个是很多开发人员的规范。调用微服务访问两种方法

1. 微服务名字【ribbon】
2. 接口和注解【feign】

#### Feign能干什么？

- Feign旨在使编写Java Http客户端变得更容易。
- 前面在使用Ribbon+RestTemplate时，利用RestTemplate对HTTP请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Mapper接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

#### Feign集成了Ribbon

利用Ribbon维护了MicroServiceCloud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过Feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用

#### 使用步骤

##### 导入依赖

```xml
<dependency>
    <groupId>school.xauat</groupId>
    <artifactId>springcloud-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
<!--Feign-->
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-feign -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
<!--Eureka-->
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>	
```

##### 在springcloud-api中添加依赖

```xml
<!--Feign-->
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-feign -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

##### 在springcloud-api中添加service模块

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/26 21:59
 * @description：Fegin的Service类
 */
@FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT")
@Service
public interface DeptService {


    @GetMapping("/dept/get/{deptno}")
    Dept selectById(Long deptno);

    @PostMapping("/dept/add/")
    boolean insert(String dname);

    @GetMapping("/dept/list")
    List<Dept> list();
}
```

##### 通过注解的方式注入DeptService

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:26
 * @description：
 */
@RestController
public class DeptController {

    @Autowired
    DeptService deptService;


    @RequestMapping("/dept/consumer/add")
    public boolean addDept(Dept dept) {
        return deptService.insert(dept.getDname());
    }

    @RequestMapping("/dept/consumer/get/{deptno}")
    public Dept getById(@PathVariable Long deptno) {
        return deptService.selectById(deptno);
    }

    @RequestMapping("/dept/consumer/list")
    public List<Dept> list() {
        return deptService.list();
    }

}
```

##### 配置启动类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/25 20:34
 * @description：启动类
 */
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients(basePackages = {"school.xauat"})//扫描提供的包
public class FeignConsumer80 {
    public static void main(String[] args) {
        SpringApplication.run(FeignConsumer80.class, args);
    }
}
```

### [Hystrix](https://github.com/Netflix/Hystrix/wiki)

#### 概念

Hystrix是Netflix针对微服务分布式采用的熔断保护中间件，相当于电路中的保险丝。

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，旨在隔离远程系统，服务和第三方库的访问点，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能保证在一个依赖出现问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性，Hystrix通过HystrixCommand对调用进行隔离，这样可以阻止故障的连锁反应，能够让接口调用快速失败并迅速恢复正常，或者回退并优雅降级

##### Hystrix能作什么

- 服务降级
- 服务熔断
- 服务限流
- 接近实时的监控

#### 服务熔断

##### 什么是服务熔断

当下游的服务因为某种原因变得不可用或者响应过慢，上游服务为了保证自己整体服务的可用性，不在继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转情况好转则恢复调用。需要说明的是熔断其实是一个框架级的处理，那么这套熔断机制的设计，基本上业内用的是`断路器模式`，如`Martin Fowler`提供的状态转换图如下所示

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210527173545.jpeg" alt="11415ee375132ec63637807fd473e83d.png" style="zoom:80%;" />

- 最开始处于close状态，一旦检测到错误达到一定的阈值，便转为open状态
- 这时候会有一个reset timeout，到了这个时间了，会转移到half open状态
- 尝试放行一部分请求到后端，一旦检测成功便会回归到closed状态，即恢复服务

业内目前流行的熔断器很多，例如阿里出的Sentinel,以及最多人使用的Hystrix
在Hystrix中，对应配置如下

```yml
//滑动窗口的大小，默认为20
circuitBreaker.requestVolumeThreshold 
//过多长时间，熔断器再次检测是否开启，默认为5000，即5s钟
circuitBreaker.sleepWindowInMilliseconds 
//错误率，默认50%
circuitBreaker.errorThresholdPercentage
```

这些属于框架层级的实现，我们只要实现对应接口就好

#### Hystrix断路器

断路器：本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩

#### 服务雪崩

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败！

当一切正常时，请求看起来是这样的：

![](https://gitee.com/zhang-songyao/blog-images/raw/master/20210526225629.png)

当其中有一个系统有延迟时，它可能阻塞整个用户的请求

![image-20210526225716226](https://gitee.com/zhang-songyao/blog-images/raw/master/20210526225717.png)



多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C有调用其他的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源在几秒钟内饱和，比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障，这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统

#### 服务熔断实例

创建新的客户端`springcloud-provider-dept-hystrix-8001`类似于`springcloud-provider-dept-8001`

##### 引入依赖

客户端导入Hystrix依赖

```xml
<!-- hystric依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    <version>2.2.3.RELEASE</version>
</dependency>
```

##### 备份方法

`HystrixCommand`会在方法抛出异常时，调用`fallbackMethod`中的方法

```java
@GetMapping("/dept/get/{deptno}")
//配置备选方案
@HystrixCommand(fallbackMethod = "hystrixQueryById")
public Dept queryById(@PathVariable Long deptno) {
    Dept dept = deptService.selectById(deptno);
    if (dept == null) {
        throw new RuntimeException("deptno===>" + deptno + "：不存在该用户");
    }
    return dept;
}
public Dept hystrixQueryById(@PathVariable Long deptno) {
    return new Dept().setDeptno(deptno)
            .setDname("ID==>"+deptno+"，没有对应的信息，为null---@Hystrix")
            .setDbSource("no this database in mysql");
}
```

##### 设置启动类

在主启动类添加对熔断的支持`@EnableHystrix`

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/24 23:00
 * @description：
 */
@EnableEurekaClient //开启Eureka注册
@SpringBootApplication
@EnableDiscoveryClient //可以忽略
@EnableHystrix
public class ProviderHystrix8001 {
    public static void main(String[] args) {
        SpringApplication.run(ProviderHystrix8001.class, args);
    }
}
```

#### 服务降级

##### 什么是服务降级

两种场景

- 当下游的服务因为某种原因响应过慢，下游服务主动停掉一些不太重要的业务，释放出服务器资源，增加服务器响应速度
- 当下游的服务因为某些原因不可用，上游主动调用本地的一些降级策略，避免卡顿，迅速返回给用户

服务降级，当服务器压力剧增的情况下，根据当前业务情况及流量对一些方服务和页面有策略的降级，以此释放服务器资源，以保证核心的任务正常执行。比如电商平台，在针对双11、618等高峰情形下采用部分服务不出现或者延时出现的情形

#### 服务降级实例

`springcloud-api`在客户端创建服务失败回调工厂的实现类`DeptServiceFallbackFactory`实现`FallbackFactory`接口

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/27 17:10
 * @description：失败回调工厂
 */
@Component
public class DeptServiceFallbackFactory implements FallbackFactory {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept selectById(Long deptno) {
                return new Dept()
                        .setDeptno(deptno)
                        .setDname("id==>"+deptno+"，没有对应的信息，客户端提供了降级的信息，这个服务现在已经被关闭")
                        .setDbSource("没有数据");
            }

            @Override
            public boolean insert(String dname) {
                return false;
            }

            @Override
            public List<Dept> list() {
                return null;
            }
        };
    }
}
```

客户端`DeptClientService`接口上添加注解`@FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT", fallbackFactory = DeptServiceFallbackFactory.class)`

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/26 21:59
 * @description：Fegin的Service类
 */
@Service
@FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT", fallbackFactory = DeptServiceFallbackFactory.class)
public interface DeptClientService {


    @GetMapping("/dept/get/{deptno}")
    Dept selectById(Long deptno);

    @PostMapping("/dept/add/")
    boolean insert(String dname);

    @GetMapping("/dept/list")
    List<Dept> list();
}
```

`springcloud-consumer-dept-feign`客户端开启服务降级配置

```yml
# 开启服务降级
feign:
  hystrix:
    enabled: true	
```

#### 服务降级和服务熔断的区别

- 服务熔断：在服务端处理，服务熔断属于降级的一种
- 服务降级：在客户端处理，有很多降级方式，如开关降级、限流降级、熔断降级

> 熔断和降级必定是一起出现。因为当发生**下游服务不可用**的情况，这个时候为了对最终用户负责，就需要**进入上游的降级逻辑**了。因此，将熔断降级视为降级方式的一种

#### Hystrix监控

##### 导入依赖

新建一个模块`springcloud-consumer-hystrix-dashboard`，导入依赖

```xml
<dependencies>
        <!--Hystrix dashboard-->
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-hystrix-dashboard -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
        <!--Hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.3.RELEASE</version>
        </dependency>
        <!--Ribbon-->
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-ribbon -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
            <version>2.2.8.RELEASE</version>
        </dependency>
        <!--Eureka-->
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>school.xauat</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
```

##### 配置端口号

```yml
server:
  port: 9001
spring:
  application:
    name: hystrix-dashboard-demo
```

##### 配置启动类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/27 19:24
 * @description：监控页面启动类
 */

@SpringBootApplication
//开启监控页面
@EnableHystrixDashboard
public class DeptConsumerDashboard {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerDashboard.class, args);
    }
}
```

要保证我们的服务者模块都要有`spring-boot-starter-actuator`依赖和`hystrix`依赖，来完成监控

```xml
<!--完善eureka监控信息-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

##### 监控一个服务

在有熔断机制的服务模块下添加

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/24 23:00
 * @description：
 */
@EnableEurekaClient //开启Eureka注册
@SpringBootApplication
@EnableDiscoveryClient //可以忽略
@EnableHystrix
public class ProviderHystrix8001 {
    public static void main(String[] args) {
        SpringApplication.run(ProviderHystrix8001.class, args);
    }

    // 增加一个Servlet
    @Bean
    public ServletRegistrationBean hystrixMetricsStreamServlet(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
            registrationBean.addUrlMappings("/actuator/hystrix.stream");
        return registrationBean;
    }
}
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210527202946.png" alt="img" style="zoom:80%;" />

### [Zuul](https://github.com/Netflix/Zuul/wiki)

> API网关是对外服务的一个入口，其隐藏了内部架构的实现，是微服务机构中必不可少的一个组件，API网关为我们管理大量的API接口，还可以对接用户、适配协议、进行安全认证、转发路由、限制流量、监控日志、防止爬虫、进行灰度发布等

#### 简介

Zuul是Netflix OSS的一员，是一个基于JVM路由和服务端的负载均衡器。提供路由、监控、弹性、安全等方面的服务框架。

Zuul的核心是过滤器，通过这些过滤器我们可以扩展出很多功能

- 动态路由：动态的将客户端的请求路由到后端不同服务，做一些逻辑处理，比如聚合多个服务的数据返回
- 请求监控：可以对整个系统的请求进行监控，记录详细的请求响应日志，可以实时统计当前系统的访问量以及监控状态
- 认证鉴权：对每一个请求做认证，拒绝非法请求，保护后端服务
- 压力测试：通过Zuul可以冬天的将请求转发到后端集群中，还可以识别测试流量和真实流量，从而做一些特殊处理
- 灰度发布：灰度发布可以保证整个系统的稳定性，在初始灰度的时候既可以发现、调整问题，以保证其影响度

> 注意：Zuul服务最终还是会注册进Eureka

#### 实例

创建`springcloud-zuul-9527`

##### 导入依赖

```xml
<dependencies>
    <!--Zuul-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        <version>2.2.3.RELEASE</version>
    </dependency>
    <!--Hystrix dashboard-->
    <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-hystrix-dashboard -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>
    <!--Hystrix-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        <version>2.2.3.RELEASE</version>
    </dependency>
    <!--Ribbon-->
    <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-ribbon -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        <version>2.2.8.RELEASE</version>
    </dependency>
    <!--Eureka-->
    <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>school.xauat</groupId>
        <artifactId>springcloud-api</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
</dependencies>
```

##### 添加配置

```yml
server:
  port: 9527
spring:
  application:
    name: springcloud-zuul-demo
eureka:
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
  instance:
    instance-id: ${spring.application.name}:${server.port}
    # 采用IP注册
    prefer-ip-address: true

zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
  ignored-services: "*" # 不能再使用这个路径访问了  ignored: 忽略，隐藏全部的
  prefix: /dunkcode   # 添加统一的访问前缀
```

##### 配置启动类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/27 21:02
 * @description：Zuul启动类
 */

@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }
}
```

## SpringCloud Config

> 集中配置管理工具，分布式系统中统一的外部配置管理，默认使用Git来存储配置，可以支持客户端配置的刷新及加密、解密操作

### 概述

微服务意味着要将单体应用中的业务拆分成一个个微服务，每个服务的粒度相对较小，因此系统中会出现大量服务。由于每个服务都需要必要的配置信息才能运行，所以一套式的、动态的配置设施是必不可少的。SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带一个`application.yml`

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210528084210.png" alt="image-20200616100811719" style="zoom:80%;" />

SpringCloud Config可以从GitHub上获取配置服务信息，为微服务架构中的微服务提供集中化的外部配置支持,配置服务器为各个不同微服务应用的所有环境提供了一个**中心化的外部配置**

SpringCloud Config分为服务端和客户端两部分

- 服务端：也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。
- 客户端：则是通过指定的配置中心来管理应用资源，以及业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用Git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过Git客户端工具来方便的管理和访问配置内容。

### 分布式配置中心能干嘛

- 集中配置文件
- 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
- 运行期间动态调整配置，不再需要每个服务部署的及其上边写配置文件，服务会向配置中心统一拉取配置自己的信息
- 当配置发生变动时，服务不需要重启，即可感知到服务配置得到变化并热部署新的配置
- 将配置信息以REST接口形式暴露

### 实例

#### 创建远程的Git仓库

> 在GitHub或者Gitee创建一个远程仓库，存放中心配置信息

引入`config-client`客户端配置文件

```yml
spring:
    profiles: 
        active: dev
---
server:
    port: 8201
spring:
  profiles: dev
  application:
    name: spirngcloud-provider-dept
# Eureka 的配置
eureka:
  client:
    service-url:       # 指定注册到哪里
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
---
server:
    port: 8202
spring:
  profiles: test
  application:
    name: spirngcloud-provider-dept
# Eureka 的配置
eureka:
  client:
    service-url:       # 指定注册到哪里
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

#### 配置服务端

##### 创建`springcloud-config-server-1122`服务端项目

##### 引入依赖

```xml
<dependencies>
    <!--SpringCloud Config-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
        <version>2.2.3.RELEASE</version>
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
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
</dependencies>
```

##### 修改配置

```yml
server:
  port: 1122
spring:
  application:
    name: springcloud-config-server
  cloud:
    config:
      server:
        git:
          #远程仓库https地址
          uri: https://gitee.com/zhang-songyao/spring-cloud.git
```

##### 设置启动类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/27 23:37
 * @description：ConfigServer启动类
 */
@SpringBootApplication
@EnableConfigServer
public class ConfigServer1122 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServer1122.class, args);
    }
}
```

#### 配置客户端

##### 创建`springcloud-config-client-3344`项目

##### 引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--starter-config -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
        <version>2.2.3.RELEASE</version>
    </dependency>
</dependencies>
```

##### 修改配置

- bootstrap：系统级别配置（优先级高）
- application：用户级别配置

>bootstrap

```yml
spring:
  cloud:
    config:
      name: config-client   # 需要从git上读取的资源名称，不要后缀
      uri: http://localhost:1122
      profile: dev
      label: master  # 指定分支
```

>application

```yml
spring:
  application:
    name: springcloud-config-client-3344
```

##### 设置启动类

为了测试我们是否通过服务端拿到了远程Git仓库中的配置信息，引入一个Controller接口

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/28 0:33
 * @description：服务接口
 */
@RestController
public class ConfigClientController {
    @Value("${spring.application.name}")
    private String applicationName;
    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServer;
    @Value("${server.port}")
    private String port;
    @RequestMapping("/config")
    public String getConfig(){
        return "applicationNmae：" + applicationName
                +"\teurekaServer："+eurekaServer
                +"\tport"+ port;
    }
}
```

> 启动类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/28 0:33
 * @description：Config客户端启动类
 */
@SpringBootApplication
public class ConfigClient_3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient_3344.class,args);
    }
}
```

### 总结

`springcloud-config-server-1122`相当于是将远程Git仓库中的配置文件加载到本地

`springcloud-config-client-3344`在启动时通过`bootstrap.yml`中的配置获取本地端口（客户端）中指定分支，指定文件下的指定配置，当做自己的配置启动项目

```yml
spring:
  cloud:
    config:
      name: config-client   # 需要从git上读取的资源名称，不要后缀
      uri: http://localhost:1122
      profile: dev
      label: master  # 指定分支
```

> 可以通过修改Git仓库的中相应配置，直接修改项目，不需要重新部署项目

## Spring Cloud Bus

用于传播集群状态变化的消息总线，使用轻量级消息代理链接分布式系统中的节点，可以用来动态刷新集群中的服务配置。

## Spring Cloud Consul

基于Hashicorp Consul的服务治理组件。

## Spring Cloud Security

安全工具包，对Zuul代理中的负载均衡OAuth2客户端及登录认证进行支持。

## Spring Cloud Sleuth

Spring Cloud应用程序的分布式请求链路跟踪，支持使用Zipkin、HTrace和基于日志（例如ELK）的跟踪。

## Spring Cloud Stream

轻量级事件驱动微服务框架，可以使用简单的声明式模型来发送及接收消息，主要实现为Apache Kafka及RabbitMQ。

## Spring Cloud Task

用于快速构建短暂、有限数据处理任务的微服务框架，用于向应用中添加功能性和非功能性的特性。

## Spring Cloud Zookeeper

基于Apache Zookeeper的服务治理组件。

## Spring Cloud Gateway

API网关组件，对请求提供路由及过滤功能。

## Spring Cloud OpenFeign

基于Ribbon和Hystrix的声明式服务调用组件，可以动态创建基于Spring MVC注解的接口实现用于服务调用，在Spring Clou

# [SpringCloud面试题](https://www.cnblogs.com/aishangJava/p/11927311.html)

## 什么是SpringCloud Gateway？

SpringCloud Gateway是SpringCloud官方推出的第二代网关框架，取代Zuul网关。网关作为流量的，在微服务系统中有着非常作用，网关常见的功能有路由转发、权限校验、限流控制等作用

使用了一个RouteLocatorBuilder的bean去创建路由，除了创建路由RouteLocatorBuilder可以让你添加各种predicates和filters，predicates断言的意思，顾名思义就是根据具体的请求的规则，由具体的route去处理，filters是各种过滤器，用来对请求做各种判断和修改。

## SpringCloud断路器的作用是什么？

在分布式架构中，断路器的作用也是类似的，当某个服务单元发生故障（类似类似用电器发生短路）之后，通过短路器的故障监控（类似熔断保险丝），向调用方返回一个错误响应，而不是长时间等待。这样就不会使线程因调用故障被长时间占用不释放是，避免了故障在分布式系统中的蔓延

## Rest和RPC对比

RPC的最主要缺陷就是服务提供方和调用方式之间的依赖太强，我们需要为每一个微服务进行接口定义，并通过持续发布，需要严格的版本控制才不会出现服务提供和调用之间因为版本不同而产生的冲突

REST是轻量级的接口,服务的提供和调用不存在代码之间的耦合,只是通过一个约定进行规范,但也有可能出现文档和接口不一致而导致的服务集成问题,但可以通过swagger工具整合,是代码和文档一体化解决,所以REST在分布式环境下比RPC更加灵活

> 这也是为什么当当网的DubboX在对Dubbo的增强中增加了对REST的支持的原因



参考文档

- 《SpringCloud微服务入门、实战与进阶》 尹吉欢
- [马丁富勒微服务博文](https://martinfowler.com/articles/microservices.html)

- [官方文档](https://www.springcloud.cc/spring-cloud-netflix.html)
- [中文API文档](https://www.springcloud.cc/spring-cloud-dalston.html)
- [springCloud中文网](https://www.springcloud.cc/)
- [LVS负载均衡](https://blog.csdn.net/weixin_40470303/article/details/80541639)
- [阿里巴巴分布式服务框架 Dubbo 团队成员梁飞专访](https://blog.csdn.net/blogdevteam/article/details/8173233?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-0&spm=1001.2101.3001.4242)

