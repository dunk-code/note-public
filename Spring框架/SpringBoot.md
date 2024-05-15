# SpringBoot概述

## 什么是SpringBoot

SpringBoot是Spring开源组织下的子项目，是Spring组件一站式解决方案，主要是简化了使用Spring的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手

## SpringBoot优点

- 创立独立的Spring应用
- 内嵌Web服务器
- 自动starter依赖，简化构建配置
- 自动配置Spring以及第三方功能
- 提供生产级别的监控、健康检查及外部化配置
- 无代码生成、无序编写XML

## Spring存在的问题

- 大量的xml文件，配置繁琐
- 整合第三方框架复杂
- 低效的开发效率和部署效率等问题
- 依赖尾部的web服务器
- 日志管理需要依赖
- pom中需要配置非常多的依赖

## SpringBoot怎么解决

- 创建独立的Spring应用程序
- 直接嵌入Tomcat、Jetty等，无需部署war文件（jar 即可）
- 提供starter启动机，简化配置依赖
- 自动配置Spring和第三方库
- 提供生产准备功能，度量、运行状况和外部化配置
- 没有代码生成，不需要xml配置

# SpringBoot自动配置的原理是什么？

## SpringBoot特点

### 依赖管理

- 父项目做依赖管理：几乎声明了所有开发中常用的依赖版本号，自动版本仲裁
- 开发导入starter场景启动器
  - 无需关注版本号，自动版本仲裁
    - 引入依赖默认都可以不写版本号
    - 引入非版本仲裁的jar，需要些版本号
  - 可以修改默认版本号

### 自动配置

- 自动配置好Tomcat
  - 引入Tomcat依赖
  - 配置Tomcat
- 自动配好SpringMVC
  - 引入SpringMVC全套组件
  - 自动配好SpringMVC常用组件
- 自动配好Web常见功能，如：字符编码问题
  - SpringBoot帮我们配置好了所有web开发的常见场景
- 默认的包结构
  - 主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来
  - 无需以前的包扫描配置
  - 想要改变扫描路径，`@SpringBootApplication(scanBasePackages="school.*")`
  - 或者@ComponentScan 指定扫描路径

```java
@SpringBootApplication
等同于
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("school.xauat.boot")
```

### 自动配置原理

注解`@EnableAutoConfiguration`，`@Configuration`，`@ConditionalOnClass`就是自动配置的核心，`@EnableAutoConfiguration`给容器导入META/spring.factories里定义的自动配置类。筛选有效的自动配置类

每一个自动配置类结合对应的**Properties.java读取配置文件进行自动配置功能

## 容器功能











# SpringBoot配置加载顺序

在SpringBoot中，可以使用以下几种方式来加载配置

- properties文件
- YAML文件
- 系统环境变量
- 命令行参数

# Swagger

## Swagger2简介

[swagger官网](https://swagger.io/) 对 swagger 的描述如下：

> Swagger takes the manual work out of API documentation, with a range of solutions for generating, visualizing, and maintaining API docs.
>
> Simplify API development for users, teams, and enterprises with the Swagger open source and professional toolset.

Swagger提供了用于生成，可视化和维护API文档的一系列解决方案，从而使API文档不再需要人工操作。

借助Swagger开源和专业工具集，为用户，团队和企业简化API开发。

## 使用Swagger解决的问题

现在大部分公司都采用前后端分离开发的模式，前端和后端工程师各司其职。这就要求有一份及时更新且完整的Rest API 文档来提高工作效率。Swagger 解决的问题主要有以下三点：

保证文档的时效性：只需要少量的注解，Swagger 就可以根据代码自动生成 API 文档，代码变文档跟着变
接口请求参数和返回结果不明确的问题
在线测试接口

## SpringBoot集成Swagger2

导入依赖swagger2和swagger-ui

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

创建swagger的配置类，并启动项目

```java
/**
 * @author ：zsy
 * @date ：Created 2021/12/14 0:20
 * @description：
 */
@Configuration
@EnableSwagger2
public class Swagger2Config {

}
```

发现报错 解决问题：SpringBoot(2.6.1)的版本较高，需要切换至SpringBoot(2.5.7)版本可以正常启动

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211214002638.png" alt="image-20211214002633698" style="zoom:80%;" />

启动后的界面

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211214002836.png" alt="image-20211214002833036" style="zoom:80%;" />

### 配置Swagger的信息

#### ApiInfo

```java
// 配置swagger的apiInfo
private ApiInfo getApiInfo() {
    // 配置作者信息
    Contact DEFAULT_CONTACT = new Contact("dunkcode", "http://www.dunkcode.cn", "zsy3020000629@163.com");
        return new ApiInfo("程序dunk的api日记", "Api Documentation", "1.0", "http://www.dunkcode.cn",
            DEFAULT_CONTACT, "Apache 2.0", "http://www.apache.org/licenses/LICENSE-2.0", new ArrayList<VendorExtension>());
}
```

#### 配置扫描接口

- RequestHandlerSelectors：配置扫描接口的方式
  - basePackage()：扫描指定的包
  - any()：扫描全部
  - none()：全部不扫描
  - withClassAnnotation()：扫描类上的注解，参数是一个注解的反射对象
  - withMethodAnnotation()：扫描方法上的注解
- PathSelectors：配置过滤路径的方式
  - ant()：指定路径
  - any()：扫描全部
  - none()：全部不扫描

```java
@Bean
// 配置swagger的Docket的bean实例
public Docket docket() {
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(getApiInfo())
            // 配置扫描的接口
            .select()
            .apis(RequestHandlerSelectors.basePackage("school.xauat.swagger.controller"))
            .paths(PathSelectors.any())
            .build();
}
```

配置是否启动Swagger`.enable(false)`		

> 如何使Swagger在生产环境中使用，而在发布环境中不使用？

两种解决方案

- 通过properties获取配置

```java
/**
 * @author ：zsy
 * @date ：Created 2021/12/14 12:03
 * @description：
 */
public class Config {
    static Properties properties;

    static {
        try (InputStream in = Config.class.getResourceAsStream("/application.properties")) {
            properties = new Properties();
            properties.load(in);
        } catch (IOException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    public static String getProfiles() {
        return properties.getProperty("spring.profiles.active");
    }
}
```

- 通过Environment

```java
Profiles profiles = Profiles.of("dev");
boolean isDev = environment.acceptsProfiles(profiles);
```

#### 配置文档分组

```java
@Bean
public Docket docket1() {
    return new Docket(DocumentationType.SWAGGER_2).groupName("A");
}
@Bean
public Docket docket2() {
    return new Docket(DocumentationType.SWAGGER_2).groupName("B");
}
```

#### 接口注释信息

```java
@Api(tags = {"启动Controller"})
@RestController
public class HelloController {
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211214151632.png" alt="image-20211214151627967" style="zoom: 67%;" />

Model中配置实体类信息

```java
/**
 * @author ：zsy
 * @date ：Created 2021/12/14 14:40
 * @description：
 */
@ApiModel("用户类")
public class User {
    @ApiModelProperty(value = "用户名")
    public String username;
    @ApiModelProperty(value = "密码")
    public String password;
}
```

同时需要在controller中返回User

```java
@ApiOperation("请求用户")
@GetMapping(value = "/user")
public User user(@ApiParam("用户名") String username) {
    return new User();
}
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211214150003.png" alt="image-20211214150000278" style="zoom:80%;" />

### [Swagger相关接口功能](https://blog.csdn.net/ThinkWon/article/details/107477801?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163946564916780274154059%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=163946564916780274154059&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-107477801.pc_v2_rank_blog_default&utm_term=swagger2&spm=1018.2226.3001.4450)































