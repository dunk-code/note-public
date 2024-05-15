> Mybatis查询Mysql中的时间datetime类型,相差8小时的解决方案
>
> URL后面添加：serverTimezone=Asia/Shangha

# Mybatis-Plus

[官方链接](https://baomidou.com)：https://baomidou.com

## 简介

### 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 快速入门

### 安装

SpringBoot依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3.4</version>
</dependency>
```

Spring依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <version>3.4.3.4</version>
</dependency>

```

>注意
>
>引入 `MyBatis-Plus` 之后请不要再次引入 `MyBatis` 以及 `MyBatis-Spring`，以避免因版本差异导致的问题

其他需要的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!--<version>5.1.49</version>-->
</dependency>
```

### 配置

> `application.properties`中配置数据源

```yaml
#数据库连接配置
spring.datasource.username=root
spring.datasource.password=root
#mysql5~8 驱动不同driver-class-name     8需要增加时区的配置serverTimezone=UTC
#useSSL=false 安全连接
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?	serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

SpringBoot启动类上配置包扫描

```java
@SpringBootApplication
@MapperScan("school.xauat.mybatisplus.mapper")
public class MybatisPlusLearnApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusLearnApplication.class, args);
    }

}
```

#### 配置日志

> 我们所有的sql是不可见的，我们希望知道他们是怎么执行的，所以要配置日志知道

```yml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 核心功能

### CRUD接口

> 具体的看mybatis-plus的文档，后面有代码生成，基本不需要自己手写

#### Service CRUD 接口

说明:

- 通用 Service CRUD 封装[IService (opens new window)](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java)接口，进一步封装 CRUD 采用 `get 查询单行` `remove 删除` `list 查询集合` `page 分页` 前缀命名方式区分 `Mapper` 层避免混淆，
- 泛型 `T` 为任意实体对象
- 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类
- 对象 `Wrapper` 为 [条件构造器](https://baomidou.com/01.指南/02.核心功能/wrapper.html)

```java
public interface UserService extends IService<User> {}
```

####  Mapper CRUD 接口

说明:

- 通用 CRUD 封装[BaseMapper (opens new window)](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-core/src/main/java/com/baomidou/mybatisplus/core/mapper/BaseMapper.java)接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器
- 泛型 `T` 为任意实体对象
- 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
- 对象 `Wrapper` 为 [条件构造器](https://baomidou.com/01.指南/02.核心功能/wrapper.html)

### 主键生成策略

> 主键生成策略

TableId

- 描述：主键注解
- 使用位置：实体类主键字段

| 属性  | 类型   | 必须指定 | 默认值      | 描述         |
| :---- | :----- | :------- | :---------- | :----------- |
| value | String | 否       | ""          | 主键字段名   |
| type  | Enum   | 否       | IdType.NONE | 指定主键类型 |

IdType

| 值            | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| AUTO          | 数据库 ID 自增                                               |
| NONE          | 无状态，该类型为未设置主键类型（注解里等于跟随全局，全局里约等于 INPUT） |
| INPUT         | insert 前自行 set 主键值                                     |
| ASSIGN_ID     | 分配 ID(主键类型为 Number(Long 和 Integer)或 String)(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextId`(默认实现类为`DefaultIdentifierGenerator`雪花算法) |
| ASSIGN_UUID   | 分配 UUID,主键类型为 String(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextUUID`(默认 default 方法) |
| ID_WORKER     | 分布式全局唯一 ID 长整型类型(please use `ASSIGN_ID`)         |
| UUID          | 32 位 UUID 字符串(please use `ASSIGN_UUID`)                  |
| ID_WORKER_STR | 分布式全局唯一 ID 字符串类型(please use `ASSIGN_ID`)         |

> 默认 : ID_WORKER 全局唯一Id

分布式系统唯一Id生成：https://www.cnblogs.com/haoxinyue/p/5208136.html

- **Twitter的snowflake算法**

snowflake是Twitter开源的分布式ID生成算法，结果是一个long型的ID。其核心思想是：使用41bit作为毫秒数，10bit作为机器的ID（5个bit是数据中心（北京、香港···），5个bit的机器ID），12bit作为毫秒内的流水号（意味着每个节点在每毫秒可以产生 4096 个 ID），最后还有一个符号位，永远是0。

- 具体实现的代码可以参看https://github.com/twitter/snowflake。

### 条件构造器

说明:

- 以下出现的第一个入参`boolean condition`表示该条件**是否**加入最后生成的sql中，例如：query.like(StringUtils.isNotBlank(name), Entity::getName, name) .eq(age!=null && age >= 0, Entity::getAge, age)
- 以下代码块内的多个方法均为从上往下补全个别`boolean`类型的入参,默认为`true`
- 以下出现的泛型`Param`均为`Wrapper`的子类实例(均具有`AbstractWrapper`的所有方法)
- 以下方法在入参中出现的`R`为泛型,在普通wrapper中是`String`,在LambdaWrapper中是**函数**(例:`Entity::getId`,`Entity`为实体类,`getId`为字段`id`的**getMethod**)
- 以下方法入参中的`R column`均表示数据库字段,当`R`具体类型为`String`时则为数据库字段名(**字段名是数据库关键字的自己用转义符包裹!**)!而不是实体类数据字段名!!!,另当`R`具体类型为`SFunction`时项目runtime不支持eclipse自家的编译器!!!
- 以下举例均为使用普通wrapper,入参为`Map`和`List`的均以`json`形式表现!
- 使用中如果入参的`Map`或者`List`为**空**,则不会加入最后生成的sql中!!!
- 有任何疑问就点开源码看,看不懂**函数**的[点击我学习新知识](https://www.jianshu.com/p/613a6118e2e0)

警告:

不支持以及不赞成在 RPC 调用中把 Wrapper 进行传输

1. wrapper 很重
2. 传输 wrapper 可以类比为你的 controller 用 map 接收值(开发一时爽,维护火葬场)
3. 正确的 RPC 调用姿势是写一个 DTO 进行传输,被调用方再根据 DTO 执行相应的操作
4. 我们拒绝接受任何关于 RPC 传输 Wrapper 报错相关的 issue 甚至 pr

具体操作看[官网](https://baomidou.com/pages/10c804/#in)：https://baomidou.com/pages/10c804/#in

> AbstractWrapper

说明:

QueryWrapper(LambdaQueryWrapper) 和 UpdateWrapper(LambdaUpdateWrapper) 的父类
用于生成 sql 的 where 条件, entity 属性也用于生成 sql 的 where 条件
注意: entity 生成的 where 条件与 使用各个 api 生成的 where 条件**没有任何关联行为**

### 代码生成器（旧）

> 注意

适用版本：mybatis-plus-generator 3.5.1以下版本，3.5.1以上的请参考 [代码生成器新](https://baomidou.com/pages/779a6e/)

AutoGenerator 是 MyBatis-Plus 的代码生成器，通过 AutoGenerator 可以快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码，极大的提升了开发效率。

> 模板代码

```java
/**
 * @author ：zsy
 * @date ：Created 2021/12/23 17:36
 * @description：
 */
public class CodeGenerator {
    public static void main(String[] args) {
        // 构建一个代码生成器对象
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        // 获取当前目录
        String projectPath = System.getProperty("user.dir");
        // 输出目录
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("zsy");
        gc.setOpen(false);
        // 是否覆盖
        gc.setFileOverride(false);
        // 去Service的I前缀
        gc.setServiceName("%sService");
        // 设置主键策略
        gc.setIdType(IdType.ASSIGN_ID);
        // 数据库时间类型 到 实体类时间类型 对应策略
        gc.setDateType(DateType.ONLY_DATE);
        gc.setSwagger2(true);
        mpg.setGlobalConfig(gc);

        // 设置数据源
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/offershow?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=Asia/Shanghai");
        dsc.setUsername("root");
        dsc.setPassword("333");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setDbType(DbType.MYSQL);
        mpg.setDataSource(dsc);

        // 包的配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName("offershow");
        pc.setParent("school.xauat");
        pc.setEntity("pojo");
        pc.setController("controller");
        pc.setService("service");
        pc.setMapper("mapper");
        mpg.setPackageInfo(pc);

        // 策略配置
        StrategyConfig sc = new StrategyConfig();
        sc.setInclude("t_entry", "t_business", "t_comment", "t_education");
        // 设置前缀
        sc.setTablePrefix("t_");
        sc.setColumnNaming(NamingStrategy.underline_to_camel);
        sc.setEntityLombokModel(true);//是否使用lombok开启注解


        // 设置逻辑删除
        // sc.setLogicDeleteFieldName("deleted");

        //自动填充配置
        /*TableFill createTime = new TableFill("create_time", FieldFill.INSERT);
        TableFill updateTime = new TableFill("update_time", FieldFill.INSERT_UPDATE);
        ArrayList<TableFill> tableFills = new ArrayList<>();
        tableFills.add(createTime);
        tableFills.add(updateTime);
        sc.setTableFillList(tableFills);*/

        //乐观锁配置
        // sc.setVersionFieldName("version");

        // 开启驼峰命名
        sc.setRestControllerStyle(true);
        sc.setControllerMappingHyphenStyle(true);

        mpg.setStrategy(sc);
        mpg.execute();
    }

}
```

### 代码生成器（新）

> 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.1</version>
</dependency>
```

> 快速生成

```java
FastAutoGenerator.create(DATA_SOURCE_CONFIG)
    // 全局配置
    .globalConfig((scanner, builder) -> builder.author(scanner.apply("请输入作者名称？")).fileOverride())
    // 包配置
    .packageConfig((scanner, builder) -> builder.parent(scanner.apply("请输入包名？")))
    // 策略配置
    .strategyConfig((scanner, builder) -> builder.addInclude(getTables(scanner.apply("请输入表名，多个英文逗号分隔？所有输入 all")))
                        .controllerBuilder().enableRestStyle().enableHyphenStyle()
                        .entityBuilder().enableLombok().addTableFills(
                                new Column("create_time", FieldFill.INSERT)
                        ).build())
    /*
        模板引擎配置，默认 Velocity 可选模板引擎 Beetl 或 Freemarker
       .templateEngine(new BeetlTemplateEngine())
       .templateEngine(new FreemarkerTemplateEngine())
     */
    .execute();


// 处理 all 情况
protected static List<String> getTables(String tables) {
    return "all".equals(tables) ? Collections.emptyList() : Arrays.asList(tables.split(","));
}

```

## 扩展

### 逻辑删除

> 说明

只对自动注入的 sql 起效:

- 插入: 不作限制
- 查找: 追加 where 条件过滤掉已删除数据,且使用 wrapper.entity 生成的 where 条件会忽略该字段
- 更新: 追加 where 条件防止更新到已删除数据,且使用 wrapper.entity 生成的 where 条件会忽略该字段
- 删除: 转变为 更新

例如:

- 删除: `update user set deleted=1 where id = 1 and deleted=0`
- 查找: `select id,name,deleted from user where deleted=0`

字段类型支持说明:

- 支持所有数据类型(推荐使用 `Integer`,`Boolean`,`LocalDateTime`)
- 如果数据库字段使用`datetime`,逻辑未删除值和已删除值支持配置为字符串`null`,另一个值支持配置为函数来获取值如`now()`

附录:

- 逻辑删除是为了方便数据恢复和保护数据本身价值等等的一种方案，但实际就是删除。
- 如果你需要频繁查出来看就不应使用逻辑删除，而是以一个状态去表示。

> 配置

```yml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

> 实体类上添加字段

```java
@TableLogic
private Integer deleted;
```

> 常见问题

如何 insert ?

1. 字段在数据库定义默认值(推荐)
2. insert 前自己 set 值
3. 使用 [自动填充功能](https://baomidou.com/pages/4c6bcf/)

删除接口自动填充功能失效

1. 使用 `update` 方法并: `UpdateWrapper.set(column, value)`(推荐)
2. 使用 `update` 方法并: `UpdateWrapper.setSql("column=value")`
3. 使用 [Sql 注入器](https://baomidou.com/pages/42ea4a/) 注入 `com.baomidou.mybatisplus.extension.injector.methods.LogicDeleteByIdWithFill` 并使用(推荐)

> 测试	

```java
@Test
public void testLogicDeleted() {
    System.out.println(userMapper.selectById(1));
    userMapper.deleteById(1);
    System.out.println(userMapper.selectById(1));
}
```

> 观察查询语句和delete语句

```sql
SELECT id,name,age,email,create_time,update_time,version,deleted FROM t_user WHERE id=? AND deleted=0
UPDATE t_user SET deleted=1 WHERE id=? AND deleted=0
```

### 自动填充功能

创建时间、更改时间！ 这些操作一般都是自动化完成，我们不希望手动更新

阿里巴巴开发手册︰几乎所有的表都要配置 gmt_create、gmt_modified ！而且需要自动化

> 方式一：数据库级别（工作中不允许修改数据库级别）

```java
private Date createTime;//驼峰命名
private Date updateTime;
```

> 方式二：代码级别

```java
//字段  字段添加填充内容
@TableField(fill = FieldFill.INSERT)//value = ("create_time"),
private Date createTime;
@TableField(fill = FieldFill.INSERT_UPDATE)
private Date updateTime;
```

> TableField

- 描述：字段注解（非主键）

|       属性       |             类型             | 必须指定 |          默认值          | 描述                                                         |
| :--------------: | :--------------------------: | :------: | :----------------------: | :----------------------------------------------------------- |
|      value       |            String            |    否    |            ""            | 数据库字段名                                                 |
|      exist       |           boolean            |    否    |           true           | 是否为数据库表字段                                           |
|    condition     |            String            |    否    |            ""            | 字段 `where` 实体查询比较条件，有值设置则按设置的值为准，没有则为默认全局的 `%s=#{%s}`，[参考(opens new window)](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlCondition.java) |
|      update      |            String            |    否    |            ""            | 字段 `update set` 部分注入，例如：`update="%s+1"` 表示更新时会 `set version=version+1` （该属性优先级高于 `el` 属性） |
|  insertStrategy  |             Enum             |    否    |  FieldStrategy.DEFAULT   | 举例：NOT_NULL `insert into table_a(<if test="columnProperty != null">column</if>) values (<if test="columnProperty != null">#{columnProperty}</if>)` |
|  updateStrategy  |             Enum             |    否    |  FieldStrategy.DEFAULT   | 举例：IGNORED `update table_a set column=#{columnProperty}`  |
|  whereStrategy   |             Enum             |    否    |  FieldStrategy.DEFAULT   | 举例：NOT_EMPTY `where <if test="columnProperty != null and columnProperty!=''">column=#{columnProperty}</if>` |
|       fill       |             Enum             |    否    |    FieldFill.DEFAULT     | 字段自动填充策略                                             |
|      select      |           boolean            |    否    |           true           | 是否进行 select 查询                                         |
| keepGlobalFormat |           boolean            |    否    |          false           | 是否保持使用全局的 format 进行处理                           |
|     jdbcType     |           JdbcType           |    否    |    JdbcType.UNDEFINED    | JDBC 类型 (该默认值不代表会按照该值生效)                     |
|   typeHandler    | Class<? extends TypeHandler> |    否    | UnknownTypeHandler.class | 类型处理器 (该默认值不代表会按照该值生效)                    |
|   numericScale   |            String            |    否    |            ""            | 指定小数点后保留的位数                                       |

关于`jdbcType`和`typeHandler`以及`numericScale`的说明:

`numericScale`只生效于 update 的 sql. `jdbcType`和`typeHandler`如果不配合`@TableName#autoResultMap = true`一起使用,也只生效于 update 的 sql. 对于`typeHandler`如果你的字段类型和 set 进去的类型为`equals`关系,则只需要让你的`typeHandler`让 Mybatis 加载到即可,不需要使用注解

> FieldStrategy

|    值     |                            描述                             |
| :-------: | :---------------------------------------------------------: |
|  IGNORED  |                          忽略判断                           |
| NOT_NULL  |                        非 NULL 判断                         |
| NOT_EMPTY | 非空判断(只对字符串类型字段,其他类型字段依然为非 NULL 判断) |
|  DEFAULT  |                        追随全局配置                         |

> FieldFill

|      值       |         描述         |
| :-----------: | :------------------: |
|    DEFAULT    |      默认不处理      |
|    INSERT     |    插入时填充字段    |
|    UPDATE     |    更新时填充字段    |
| INSERT_UPDATE | 插入和更新时填充字段 |

现在pojo对象属性上声明注解

```java
@Data
@TableName("t_user")
public class User {
    @TableId
    private Long id;
    private String name;
    private Integer age;
    private String email;
    //字段  字段添加填充内容
    @TableField(fill = FieldFill.INSERT)//value = ("create_time"),
    private Date createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
}
```

添加自定义处理类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/12/22 17:24
 * @description：
 */
@Slf4j
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("->{}", metaObject);
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("->{}", metaObject.getOriginalObject());
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
```

## 插件

### 乐观锁

当要更新一条记录的时候，希望这条记录没有被别人更新

乐观锁实现方式：

- 取出记录时,获取当前version
- 更新时,带上这个version
- 执行更新时,set version = newVersion where version = oldVersion
- 如果version不对,就更新失败

配置插件

```java
/**
 * @author ：zsy
 * @date ：Created 2021/12/23 14:39
 * @description：
 */
@MapperScan("school.xauat.mybatisplus.mapper")
@Configuration
@EnableTransactionManagement
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```

t_user表中添加字段version

pojo中添加`@Vsersion`注解

```java
@Version
private Integer version;
```

> 成功案例和失败案例

```java
@Test
public void testOptimisticLockerSuccess() {
    User user = userMapper.selectById(1);
    user.setName("王五");
    user.setEmail("834187150@gmail.com");
    userMapper.updateById(user);
}

@Test
public void testOptimisticLockerFail() {
    // 线程1
    User user1 = userMapper.selectById(2L);
    user1.setName("赵四1");
    user1.setEmail("834187150@gmail.com");

    // 线程2
    User user2 = userMapper.selectById(2L);
    user2.setName("赵四2");
    user2.setEmail("834187150@gmail.com");
    userMapper.updateById(user2);

    userMapper.updateById(user1);
}
```

可以观察执行的sql语句

```sql
// 会使用version去进行判断
Preparing: UPDATE t_user SET name=?, age=?, email=?, update_time=?, version=? WHERE id=? AND version=?
```

### 分页插件

PaginationInnerInterceptor

支持的数据库

- mysql，oracle，db2，h2，hsql，sqlite，postgresql，sqlserver，Phoenix，Gauss ，clickhouse，Sybase，OceanBase，Firebird，cubrid，goldilocks，csiidb
- 达梦数据库，虚谷数据库，人大金仓数据库，南大通用(华库)数据库，南大通用数据库，神通数据库，瀚高数据库

> 属性介绍

|  属性名  |   类型   | 默认值 |                             描述                             |
| :------: | :------: | :----: | :----------------------------------------------------------: |
| overflow | boolean  | false  | 溢出总页数后是否进行处理(默认不处理,参见 `插件#continuePage` 方法) |
| maxLimit |   Long   |        |  单页分页条数限制(默认无限制,参见 `插件#handlerLimit` 方法)  |
|  dbType  |  DbType  |        | 数据库类型(根据类型获取应使用的分页方言,参见 `插件#findIDialect` 方法) |
| dialect  | IDialect |        |          方言实现类(参见 `插件#findIDialect` 方法)           |

> 建议单一数据库类型的均设置 dbType

配置插件

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
    // 分页插件
    mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
    return mybatisPlusInterceptor;
}
```

> 测试

```java
@Test
public void testPage() {
    Page<User> page = new Page<>(1, 3);
    Page<User> userPage = userMapper.selectPage(page, null);
    List<User> users = userPage.getRecords();
    users.forEach(System.out::println);
}
```

### 性能分析插件

> 性能分析拦截器，用于分析每条SQL语句及其执行时间

已经在3.2.0版本后移除

```java
 //性能分析插件
@Bean
@Profile({"dev","test"})//设置dev开发、test测试 环境开启  保证我们的效率
public PerformanceInterceptor performanceInterceptor(){
    PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
    performanceInterceptor.setMaxTime(100);//设置sql最大执行时间*ms，如果超过了则不执行
    performanceInterceptor.setFormat(true);//开启sql格式化
    return performanceInterceptor;
}
```

官网文档提供的

```java
// SQL性能分析插件
mybatisPlusInterceptor.addInnerInterceptor(new IllegalSQLInnerInterceptor());
```

