# Spring概述

## 概念

Spring是一个轻量级Java开发框架，最早有Rod Johnson创建，目的是为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。它是一个分层的JavaSE/JavaEE full-stack（一站式）轻量级开源框架，为开发Java应用程序提供全面的基础架构支持。Spring负责基础架构，因此Java开发者可以专注于应用程序的开发

> Spring最根本的使命是**解决企业级应用开发的复杂性，即简化Java开发**

Spring可以做很多事情，它为企业级开发提供给了丰富的功能，但是这些功能的底层都依赖于它的两个核心特性，也就是**依赖注入（dependency injection，DI）和面向切面编程（aspect-oriented programming，AOP）**

为了降低Java开发的复杂性，Spring采取了以下4种关键策略

- 基于POJO的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少样板式代码

## 优缺点

### 优点

- 方便解耦，简化开发：Spring就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给Spring管理
- AOP编程的支持：Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能
- 声明式事务的支持：只需要通过配置就可以完成对事务的管理，而无需手动编程
- 方便程序的测试：Spring对Junit4支持，可以通过注解方便的测试Spring程序
- 方便集成各种优秀框架：Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis等）
- 降低JavaEE API的使用难度：Spring对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低

### 缺点

- Spring依赖反射，反射影响性能
- 使用门槛升高，入门Spring需要较长时间

## Spring的模块

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210610114757.png" alt="在这里插入图片描述" style="zoom:80%;" />

- spring core：提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能。
- spring beans：提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean。
- spring context：构建于 core 封装包基础上的 context 封装包，提供了一种框架式的对象访问方法。
- spring jdbc：提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析， 用于简化JDBC。
- spring aop：提供了面向切面的编程实现，让你可以自定义拦截器、切点等。
- spring Web：提供了针对 Web 开发的集成特性，例如文件上传，利用 servlet listeners 进行 ioc 容器初始化和针对 Web 的 ApplicationContext。
- spring test：主要为测试提供支持的，支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。

## Spring框架中都用到了那些设计模式

- 工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例
- 单例模式：Bean默认为单例模式
- 代理模式：SpringAOP功能用到了JDK的动态代理和CGLIB字节码生成技术
- 模板方法：用来解决代码重复的问题，比如：RestTemplate, JmsTemplate, JpaTemplate、refresh
- 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到被制动更新，如Spring中listener的实现–ApplicationListener

# Spring IOC

## IOC是什么？

IOC（Inversion of Control）即控制反转，简单说就是把原来代码需要实现的对象创建、依赖反转给容器来帮忙实现，需要创建一个容器并且需要一种描述让容器指导要创建对象之间的关系，在Spring中管理对象及其依赖关系是通过Spring的IOC容器实现的

IOC的实现方式有==依赖注入==和==依赖查找==是，由于依赖查找用的很少，因此IOC也叫做依赖注入。依赖注入指对象被动地接受依赖类而不用自己主动去找，对象不是从容器中查找它依赖的类，而是在**容器实例化对象时，主动将它所依赖的类注入给它**

> 依赖注入更加准确的描述了IOC的设计理念，所谓依赖注入（Dependency Injection），即组件之间的依赖关系有容器在应用系统运行期来决定，也就是由容器动态地将某种依赖关系的目标对象实例注入到应用程序中的各个关联的组件之中，组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系

举例：一个Car类需要一个Engine的对象，那么一般需要手动new一个Engine，利用IOC就只需要定义一个私有的Engine类型的成员变量，容器会在运行时创建一个Engine的实例对象并将引用自动注入给成员变量

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/5 16:37
 * @description：车
 */
@Component
@Data
public class Car {
    @Autowired
    public Engine engine;
}

/**
 * @author ：zsy
 * @date ：Created 2021/6/5 16:38
 * @description：引擎
 */
@Component
public class Engine {
    private String name;
}
```

> 控制反转是：关于一个对象如何获取他所依赖的对象的引用，这个责任的反转

## IOC容器初始化的过程

### 基于xml的容器初始化

当创建一个ClassPathXmlApplicationContext时，构造方法做了两件事

1. 调用父容器的构造方法为容器设置好Bean资源资源加载器
2. 调用父类的`setConfigLocations`方法设置Bean信息的定位路径

ClassPathXmlApplicationContext通过调用父类AbstractApplicationContext的`refresh`方法启动整个IOC容器对Bean定义的载入过程，`refresh`是一个模板方法，规定了IOC容器的启动流程，在创建IOC容器前如果已有容器存在，需要把已有的容器销毁，保证在`refresh`方法后使用的是新创建的IOC容器

容器创建后通过loadBeanDefinitions方法加载Bean配置资源，该方法做两件事：

- 调用资源加载器的方法获取需要加载的资源
- 真正执行加载功能，由子类XmlBeanDefinitionReader实现。加载资源时
  - 首先解析配置文件路径，读取配置文件内容，然后通过xml解析器将Bean配置信息转换成文档对象
  - 之后按照Spring Bean定义规则对文档对象进行解析

Spring IOC容器中注解解析的**Bean信息放在一个HashMap集合中，key是字符串，值是BeanDefinition**，注册过程中需要==synchronized==保证线程安全，当配置文件的Bean被解析且被注册到IOC容器中后，初始化就算真正完成了，Bean 定义信息已经可以使用且可被检索。Spring IoC 容器的作用就是对这些注册的 Bean 定义信息进行处理和维护，注册的 Bean 定义信息是控制反转和依赖注入的基础

```xml
<context:component-scan base-package="school.xauat"></context:component-scan>
<bean id="user" class="school.xauat.pojo.User">
    <property name="username" value="zhangsan"></property>
    <property name="age" value="15"></property>
</bean>
```

测试类

```java
@org.junit.Test
public void Test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    Object user = context.getBean("user");
    System.out.println(user);
}
```

### 基于注解的容器初始化

分为两种：

- 直接将注解Bean注解到容器中，可以在初始化容器时注册，也可以在容器创建之后手动注册，然后刷新容器使其对注册的注解Bean进行处理
- 通过扫描指定包及其子包的所有类处理，在初始化注解容器时指定要自动扫描的路径

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/5 16:00
 * @description：配置类
 */
@Configuration
@ComponentScan({"school.xauat"})
public class Config {
    @Bean(name = "car")
    public Car getCar() {
        return new Car().name("凯迪拉克").engine(new Engine("引擎"));
    }
}

@org.junit.Test
public void test() {
    ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
    Car car = context.getBean("car", Car.class);
    System.out.println(car);
}
```

或者

在xml中开启注解装配

```xml
<context:annotation-config></context:annotation-config>
```

然后使用`ClassPathXmlApplicationContext`加载xml配置文件

## 依赖注入的实现方法

### 构造方法注入

IOC Service Provider会检查被注入对象的构造方法取得它所需要的依赖对象列表，进而为其注入相应的对象。这种方法的优点是在对象构造完成后就处于就绪状态，可以马上使用。缺点是当依赖对象较多时，构造方法的参数列表会比较长，构造方法无法被继承，无法设置默认值。对于非必需的依赖处理可能需要引入多个构造方法，参数数量的变动可能会造成维护的困难

```xml
<bean id="user2" class="school.xauat.pojo.User">
    <constructor-arg name="username" value="lisi"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
</bean>
```

### setter方法注入

当前对象要只需要为其依赖对象对应的属性添加setter方法，就可以通过setter方法将依赖对象注入中。setter 方法注入在描述性上要比构造方法注入强，并且可以被继承，允许设置默认值。缺点是无法在对象构造完成后马上进入就绪状态

```xml
<bean id="user1" class="school.xauat.pojo.User">
    <property name="username" value="zhangsan"></property>
    <property name="age" value="15"></property>
</bean>
```

### 构造器依赖注入和Setter方法注入的区别

|         构造函数注入         |         Setter注入         |
| :--------------------------: | :------------------------: |
|         没有部分注入         |         有部分注入         |
|      不会覆盖setter属性      |      会覆盖setter属性      |
| 任意修改都会创建一个新的实例 | 任意修改不会创建一个新实例 |
|      适用于设置很多属性      |      使用设置少量属性      |

两种依赖方式都可以使用，构造器注入和Setter方法注入。最好的解决方案是用构造器参数实现强制依赖，setter方法实现可选依赖

### 接口注入

 必须实现某个接口，接口提供方法来为其注入依赖对象。使用少，因为它强制要求被注入对象实现不必要接口，侵入性强，已在Spring4中废除

## 依赖注入的过程？

- `getBean`方法获取Bean实例，该方法调用`doGetBean`，`doGetBean`真正实现从IOC容器获取Bean的功能，也是触发依赖注入的地方
- 具体创建Bean对象的过程是由`ObjectFactory`的`createBean`完成的，该方法主要通过`createBeanInstance`方法生成Bean包含的Java对象实例和`populateBean`方法对Bean属性的依赖注入进行处理
- 在`populateBean`方法中，注入过程主要分为两种情况：
  - 属性值类型不需要强制转换时，不需要强制类型转换时，不需要解析属性值，直接进行依赖注入
  - 属性类型需要强制转换时
    - 首先解析属性值
    - 然后对解析后的属性值进行依赖注入。
    - 依赖注入的过程就是将Bean对象实例设置到它所依赖的Bean对象的属性上
    - 真正的依赖注入是通过`setPropertyValues`方法实现的，该方法使用了委派模式
- `BeanWrapperImpl`类负责对完成初始化的Bean对象进行依赖注入
  - 对于非集合类型属性，使用JDK反射，通过属性的setter方法为属性设置注入后的值
  - 对于集合类型的属性，将属性值解析为目标类型的集合直接赋值给属性

> 当容器对Bean的定位、载入、解析和依赖注入完成后就不再需要手动创建对象，IOC容器会自动为我们创建对象并且依赖注入

## Bean的生命周期

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210616120413.png" alt="img" style="zoom:80%;" />

- Spring容器读取XML配置文件中bean定义并实例化Bean
- Spring根据Bean的定义设置属性值
- 如果该Bean实现了BeanNameAware接口，Spring将Bean的id传递给setBeanName()方法
- 如果该Bean实现了BeanFactoryAware接口，Spring将BeanFactory传递给setBeanFactory()方法
- 如果bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用传入进来
- 如果bean实现了BeanPostProcessor接口，Spring将调用它们的post-ProcessBeforeInitialization()方法
- 如果bean实现了InitializingBean接口，Spring将调用它们的after-PropertiesSet()方法。类似地，如果bean使用initmethod声明了初始化方法，该方法也会被调用

> 此时Bean就已经准备就绪了，可以被应用程序使用了，他们一直驻留在容器中，直到容器被销毁

- 果bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果bean使用destroy-method声明了销毁方法，该方法也会被调用

> 自己定制初始化和注销方法

XML 方式通过配置 bean 标签中的 init-Method 和 destory-Method 指定自定义初始化和销毁方法。

```xml
<bean id="user" class="school.xauat.pojo.User" init-method="initMethod" destroy-method="destoryMethod">
    <property name="username" value="zhangsan"></property>
    <property name="age" value="15"></property>
</bean>
```

注解方式通过 `@PreConstruct` 和 `@PostConstruct` 注解指定自定义初始化和销毁方法

```java
@PostConstruct
public void destory() {
}
@PreDestroy
public void init() {
}
```

## Bean的作用范围

通过`scope`属性指定bean的作用范围，包括：

- singleton：单例模式，是默认作用域，不管收到多少Bean请求每个容器中只有一个唯一的Bean实例
- prototype：原型模式，和singleton相反，每次Bean请求都会创建一个新的实例，创建后，Spring将不在管理

web项目中才用到的时

- request：每次HTTP请求都会创建一个新的Bean并把它放到request域中，在请求完成之后Bean会失效并被垃圾收集器回收，不同于prototype，创建后，Spring依然在监听
- session：和request类似，确保每个session中有一个Bean实例，session过期后Bean会随之失效
- global session：全局的web域，类似与servlet中的applicaton

## Bean线程安全性

> Spring的单例Bean不是线程安全的

Spring中的Bean默认是单例模式，Spring框架中没有对单例Bean进行多线程的封装处理

实际上大部分时候 spring bean 无状态的（比如 dao 类），所有某种程度上来说 bean 也是安全的，但如果 bean 有状态的话（比如 view、model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了

- 有状态：有数据存储功能
- 无状态：不会保存数据

只有无状态的Bean才会在多线程下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式

### 线程不安全解决方案

> 为了让无状态的海量HTTP请求之间不收影响，我们可以采取以下几种措施

#### 单例变原型

对web项目，可以Controller类上加注解@Scope("prototype")或者@Scope("request")，对非web项目，在Component类上添加注解@Scope("prototype")

优点：实现简单

缺点：很大程度上增大了Bean创建实例化销毁的服务器资源开销

#### 线程隔离类ThreadLocal

ThreadLocal为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据访问的冲突，因为每一个线程都拥有自己的变量副本，从而就没有必要对该变量进行同步，ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal

web服务器默认的请求线程池大小为10，这10个核心线程可以被之后不同的HTTP请求复用是，所以这也是为什么相同线程名的结果不会重复的原因

总结：ThreadLocal的方式可以达到线程线程隔离，但是无法到达并发安全

#### 尽量避免使用成员变量

在业务允许的条件下，能不用成员变量，就尽量避免使用成员变量，将成员变量替换为RequestMapping方法中的局部变量，这种方式是最恰当的

#### 使用并发安全的类

如果必须使用成员变量呢

java作为功能性超强的编程性语言，API丰富，如果非要在单例Bean中使用成员变量，可以考虑使用并发安全的容器，如ConcurrentHashMap、ConcurrentHashSet等，将成员变量（一般可以是当前运行中的任务列表这类变量）封装到这些并发安全的容器中进行管理

#### 分布式或微服务的并发安全

如果需要将进一步考虑到微服务或分布式服务的影响，使用并发安全的类不足以处理了，所以可以借助于共享某些信息的分布式缓存中间件Redis等，这样既可以保证同一种服务的不同服务实例都拥有同一份共享信息（如当前运行中的任务列表等这类变量）

## 扩展问题

 返回 Class 对象表示的类或接口的所有已声明的方法数组，但是不包括从父类继承和接口实现的方法。

### 创建Bean的方式

#### xml

- 默认无参构造方法，只需要指明 bean 标签中的 id 和 class 属性，如果没有无参构造方***报错
- 静态工厂方法，通过 bean 标签中的 class 属性指明静态工厂，factory-method 属性指明静态工厂方法
- 实例工厂方法，通过 ean 标签中的 factory-bean 属性指明实例工厂，factory-method 属性指明实例工厂方法

#### 注解

`@Component` 把当前类对象存入 Spring 容器中，相当于在 xml 中配置一个 bean 标签。value 属性指定 bean 的 id，默认使用当前类的首字母小写的类名

`@Controller`，`@Service`，`@Repository` 三个注解都是 `@Component` 的衍生注解，作用及属性都是一模一样的。只是提供了更加明确语义，`@Controller` 用于表现层，`@Service`用于业务层，`@Repository`用于持久层。如果注解中有且只有一个 value 属性要赋值时可以省略 value

如果想将第三方的类变成组件又没有源代码，也就没办法使用 `@Component` 进行自动配置，这种时候就要使用 `@Bean` 注解。被 `@Bean` 注解的方法返回值是一个对象，将会实例化，配置和初始化一个新对象并返回，这个对象由 Spring 的 IoC 容器管理。name 属性用于给当前 `@Bean` 注解方法创建的对象指定一个名称，即 bean 的 id。当使用注解配置方法时，如果方法有参数，Spring 会去容器查找是否有可用 bean对象，查找方式和 `@Autowired` 一样

### BeanFactory、FactoryBean和ApplicationContext的区别?

#### BeanFactory（”低级容器"）

BeanFactory是一个Bean工厂，使用简单工厂模式，是Spring IOC容器的顶级接口，可以理解为含有Bean集合的工厂，作用是管理Bean，包括实例化、定位、配置对象及建立这些对象间的依赖。BeanFactory 实例化后并不会自动实例化 Bean，只有当 Bean 被使用时才实例化与装配依赖关系，属于==延迟加载==，适合多例模式

#### FactoryBean

FactoryBean是一个工厂Bean，使用了工厂方法模式，作用是生产其他Bean实例，可以通过实现改接口，提供一个工厂方法来自定义实例化Bean的逻辑，FactoryBean接口有BeanFactory中的配置实现，，这些对象本身就是用于创建对象的工厂，如果一个 Bean 实现了这个接口，那么它就是创建对象的工厂 Bean，而不是 Bean 实例本身

#### ApplicationContext（”高级容器“）

ApplicationContext是BeanFactory的子接口，扩展了BeanFactory的功能，提供了支持国际化的文本消息，统一的资源文件读取方式，事件传播以及应用层的特别配置等。该接口定义了一个`refresh`方法，用于刷新整个容器，即重新加载/刷新所有的bean。容器会在初始化时对配置的 Bean 进行预实例化，Bean 的依赖注入在容器初始化时就已经完成，属于==立即加载==，适合单例模式，一般推荐使用

实现类：

- ClassPathApplicationContext：只能读取路径下的配置文件（推荐使用）
- FileSystemXmlApplicationContext：可以读取任意硬盘路径的配置文件，你可以点击根目录下的文件的属性查看他在本地硬盘的位置
- AnnotationConfigApplicationContext：读取注解配置类

# Spring AOP

## AOP是什么？

AOP（Aspect-Oriented Programming）即面向切面编程，简单地说就是将代码中重复的部分抽取出来，在需要执行的时候使用动态代理技术，在不修改源码的基础上对方法进行增强，减少了系统的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性

> 常见的场景包括：权限认证、自动缓存、错误处理、日志、调试和事务

### 动态代理

#### JDK

- Spring根据类是否实现接口来判断动态代理的方式，如果实现接口会使用JDK动态代理，核心是==InvocationHandler==接口和==Proxy==类，JDK动态代理主要通过重组字节码实现，首先获得被代理对象的引用和所有接口，生成新的类必须实现代理类的所有接口，动态生成Java代码后编译生成新的`.class`文件并重新加载到JVM运行。JDK代理直接写Class字节码
- InvocationHandler 的 invoke(Object proxy,Method method,Object[] args)：proxy是最终生成的代理实例; method 是被代理目标实例的某个具体方法; args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。

#### CGLIB

- 如果代理类没有实现InvocationHandler接口，那么Spring AOP会选择使用CGLIB来动态代理目标类
- CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中的特定方法并添加增强代码
- 采用ASM框架写字节码，生成代理类的效率低，但是CGLIB调用方法的效率高，因为JDK使用反射调用方法，
- CGLIB使用FastClass机制为代理类和被代理类各生成一个子类，这个类会为代理类或者被代理类的方法生成一个index，这个index可以作为参数直接定位要调用的方法
- 如果没有实现接口会使用CGLIB是运行时动态生成某个类的子类，如果某个类被标记为final，不能使用CGLIB

## AOP相关术语

`Aspect`：切面是通知和切点的结合。通知和切点共同定义了切点的全部内容，在Spring AOP中，切面可以使用通用类（基于模式的风格）或者在普通类中以`@AspectJ`注解来实现

`Joinpoint`：指连接点，在SpringAOP中，一个连接点总是代表一个方法的执行。应用可能有数以千计应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为

`Advice`：通知，指切面对于某个连接点所产生的动作，包括

- 前置通知（Before）
- 后置通知（After）
- 返回后通知（AfterReturning）
- 异常通知（AfterThrowing）
- 环绕通知（Around）

`Pointcut`：切入点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点

`Proxy`：代理，Spring AOP 中有 JDK 动态代理和 CGLib 代理，目标对象实现了接口时采用 JDK 动态代理，反之采用 CGLib 代理

`Target`：代理的目标对象，指一个或多切面所通知的对象

`Weaving` ：织入，指把增强应用到目标对象来创建代理对象的过程

- 编译期：切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面

## AOP的过程

SpringAOP由BeanPostProcessor后置处理器开始，这个后置处理器是一个监听器，可以监听容器触发的Bean生命周期事件，向容器注册后置处理器以后，容器中管理的Bean就具备了接收IOC容器回调事件的能力。BeanPostProcessor 的调用发生在 Spring IoC 容器完成 Bean 实例对象的创建和属性的依赖注入后，为 Bean 对象添加后置处理器的入口是 `initializeBean` 方法

AOP工作中心在于如何将增强编织到目标对象的连接点上，这里包含两个动作

- 如何通过pointcut和advice定位到特定的joinpoint上
- 如何在advice中编写代码

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210616113847.png" alt="在这里插入图片描述" style="zoom:80%;" />

## 扩展问题

### AOP有哪些实现方式

AOP实现的关键在于代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表是AspectJ，动态代理则以SpringAOP为代表。

### SpringAOP 和 AspectJ AOP有什么区别

- AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ（切面）织入到Java字节码中，运行的时候就是增强的AOP对象
- Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时，在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理

# Spring注解

## 基于Spring注解配置

基于Java的配置，允许在少量Java注解的帮助下，进行大部分Spring的配置而非通过xml文件。

- `@Configuration`：用于指定当前类是一个Spring配置类，标记类可以当做一个Bean的定义，被IOC容器使用
  - value属性用于指定配置类的字节码文件
- `@Bean`：表示此方法要返回一个对象，作为一个Bean注册到Spring应用上下文中
- `@ComponentScan`：用于指定Spring在初始化容器时要扫描的包，basePaceages属性用于指定要扫描的包
- `@PropertySource`：用于加载`.properties` 文件中的配置。value 属性用于指定文件位置，如果是在类路径下需要加上 classpath
- `@Import` 用于导入其他配置类，在引入其他配置类时可以不用再写 `@Configuration` 注解

## 对象注解

`@Component`：将java标记为Bean，他是任何Spring管理组件的通用构造型，Spring的组件扫描机制可以将其拾取并拉入容器中

`@Controller`：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IOC 容器中

`@Service`：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图

`@Repository`：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException

## 依赖注入（DI）的相关注解

`@Required`（已过时）：这个注解表明Bean的属性，必须在配置的时候设置，通过一个Bean定义的显式的属性值或通过自动装配，若@Required注解的bean属性未被设置，容器将抛出BeanInitializationException

`@Autowired`：自动==按类型注入==，如果有多个匹配则按照指定 Bean 的 id 查找，查找不到会报错，默认情况下，==要求对象必须存在==（可以设置required为false），可用于构造函数、成员变量、Setter方法

`@Qualifier`：在自动按照类型注入的基础上==再按照 Bean 的 id 注入==，给变量注入时必须搭配，当创建多个相同类型的Bean，并希望仅使用属性装配其中一个 Bean时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 Bean来消除歧义

`@Resource` ：直接按照 Bean 的 id 注入，当找不到与名称匹配的Bean才会按照类型来装配注入，只能注入 Bean 类型

`@Value` ：用于注入基本数据类型和 String 类型

## @AutoWired注解自动装配的过程

> 使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />

在启动容器时，容器自动装载了一个`AutowiredAnnotationBeanPostProcessor`后置处理器，当容器扫描到@Autowired、@Resource时就会在IOC容器自动查找需要的Bean，并装配给该对象属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

- 如果查询结果刚好为一个，就将该Bean装配给@Autowired指定的数据
- 如果查询的结果不止一个，那么@Autowired会根据名称来查找
- 如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false

## 自动装配有哪些局限性？

自动装配的局限性是：

- **重写**：你仍需用 和 配置来定义依赖，意味着总要重写自动装配。
- **基本数据类型**：你不能自动装配简单的属性，如基本数据类型，String字符串，和类。
- **模糊特性**：自动装配不如显式装配精确，如果有可能，建议使用显式装配。

## AOP的相关注解有哪些？

`@Aspect`：声明被注解的类是一个切面Bean

`@Before`：前置通知，指定某个连接点之前执行的通知

`@After`：后置通知，指定某个连接点退出是执行的通知（不论正常返回还是退出）

`@AfterReturning`：返回后通知，指某连接点正常完成之后的通知，返回值使用returning属性接收

`@AfterThrowing`：异常通知，指方法抛出异常导致退出是执行的通知，和`@AfterReturning`只会有一个执行，异常使用throwing属性接受

# Spring事务

## Spring事务实现的方式

> Spring事务的本质其实就是数据库对事务的支持，，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的

- 声明式事务管理：可以将业务代码和事务管理分离，主需要用注解和XML配置来管理事务
- 编程式事务管理：通过编程的方式管理事务，灵活性高，但是难维护

## Spring事务隔离级别

spring 有五大隔离级别，默认值为 ISOLATION_DEFAULT（使用数据库的设置），其他四个隔离级别和数据库的隔离级别一致

- ISOLATION_DEFAULT：用底层数据库的设置隔离级别，数据库设置的是什么我就用什么
- ISOLATION_READ_UNCOMMITTED：未提交读，最低隔离级别、事务未提交前，可被其他事务读取（会出现幻读、脏读、不可重复读）
- ISOLATION_READ_COMMITTED：提交读，一个事务提交后才能被其他事务读取到（会造成幻读、不可重复读），SQL server 的默认级别
- ISOLATION_REPEATABLE_READ：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），MySQL 的默认级别
- ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读

## Spring事务传播行为

> 当多个事务同时存在的时候，Spring如何处理这些事务的行为称为事务的传播行为

-  PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置
- PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行
- PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常
- PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务
-  PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常
- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行

## 事务管理类型的选择

大多数Spring框架用户选择声明式事务管理，因为它对应用代码影响很小，因此更符合一个无侵入的轻量级容器的思想。。声明式事务管理要优于编程式事务管理，虽然比编程式事务管理（这种方式允许你通过代码控制事务）少了一点灵活性。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别