![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210512212620.png)

# Mybatis基础

## 概念

> Mybatis是什么

Mybatis是一款优秀的持久层框架，一个半ORM（对象关系映射）框架，它支持定制化SQL、存储过程以及高级映射。Mybatis避免了所有JDBC代码和手动设置参数以及获取结果集。Mybatis可以使用简单的XML或注解来配置和映射原生类型、接口和Java的POJO（普通老式Java对象）为数据库中的记录

> ORM是什么

ORM（Object Relational Mapping）对象关系映射，是一种为了解决关系型数据库数据与简单Java对象（POJO）的映射关系的技术。简单来说，ORM是通过使用描述对象和数据库之间的映射关系的元数据，将程序中的对象自动持久化到关系型数据库中

> 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？

Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以他是全自动的

而Mybatis在查询关联对象或者关联集合对象时，需要手动编写SQL来完成，所以称之为半自动ORM映射工具

> 传统JDBC开发存在的问题

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/16 20:42
 * @description：JDBC
 */
public class JDBCTest {
    public static void main(String[] args) {
        ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
        String driver = bundle.getString("driver");
        String url = bundle.getString("url");
        String user = bundle.getString("user");
        String password = bundle.getString("password");

        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        PreparedStatement preparedStatement = null;

        try {
            //注册驱动
            Class.forName(driver);
            //获取连接
            conn = DriverManager.getConnection(url, user, password);
            //获取数据库操作对象
            //stmt = conn.createStatement();
            //执行sql语句
            String sql = "select empno,ename,sal from emp where sal > ?";
            //int count=executUpdate(insert/delete/update)
            //ResultSet rs=executeQuery(select)
            preparedStatement = conn.prepareStatement(sql);
            preparedStatement.setInt(1, 3000);
            rs = preparedStatement.executeQuery();
            //处理数据查询集
            //boolean flag1=rs.next();
            while (rs.next()) {
                String empno = rs.getString("empno");//JDBC中所有下标从1开始。不是从0开始。
                String ename = rs.getString("ename");
                String sal = rs.getString("sal");
                System.out.println(empno + "\t" + ename + "\t" + sal);
            }
			/*while(rs.next()){
				String empno=rs.getString(1);//JDBC中所有下标从1开始。不是从0开始。
				String ename=rs.getString(2);
				String sal=rs.getString(3);
				System.out.println(empno+"\t"+ename+"\t"+sal);
			}*/
            //释放资源
        } catch (SQLException | ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            try {
                if (rs != null) {
                    rs.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (preparedStatement != null) {
                    preparedStatement.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (stmt != null) {
                    stmt.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (conn != null) {
                    conn.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

    }
}

```

1. 频繁创建数据库连接对象、释放，容易造成系统资源浪费，影响系统性能。可以使用连接池解决这个问题。但是使用jdbc需要自己实现连接池
2. sql语句定义、参数设置、结果集处理存在硬编码。实际项目中sql语句变化的可能性较大，一旦发生变化，需要修改java代码，系统需要重新编译，重新发布。不好维护
3. 使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不一定，可能多也可能少，修改sql还要修改代码，系统不易维护
4. 结果集处理存在重复代码，处理麻烦。如果可以映射成Java对象会比较方便

> 针对JDBC编程的不足，Mybatis是如何解决这些问题

1. Mybatis-config.xml中配置数据连接池，使用连接池管理数据库连接

   - POOLED：由Mybatis创建传统的javax.sql.DataSource连接池用于数据库操作，操作完成后Mybatis会将连接返回给连接池，此配置常见于开发或测试环境中。

   * UNPOOLED：由Mybatis为每一次数据库操作创建一个新的连接，并在操作完成后关闭连接，此配置未践行池化思想且仅适用于规模较小的并发应用程序中。
   * JNDI：采用服务器提供的JNDI技术获取DataSource对象，不同服务器中获取的DataSource对象不一致，例如在Tomcat服务器中采用DBCP连接池，此配置不适用于非Web或Maven的war工程。

2. 将Sql语句配置在XXXXmapper.xml文件中与java代码分离。

3. Mybatis自动将java对象映射至sql语句

4. Mybatis自动将sql执行结果映射至java对象

> Mybatis优缺点

优点

与传统的数据库访问技术相比，ORM有以下优点：

- 基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在XML里，解除sql与程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可重用
- 与JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接
- 很好的与各种数据库兼容（因为MyBatis使用JDBC来连接数据库，所以只要JDBC支持的数据库MyBatis都支持）
- 提供映射标签，支持对象与数据库的ORM字段关系映射；提供对象关系映射标签，支持对象关系组件维护
- 能够与Spring很好的集成

缺点

- SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求
- SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库

> Hibernate和Mybatis的区别

**相同点**

都是对jdbc的封装，都是持久层的框架，都用于dao层的开发。

映射关系

- MyBatis 是一个半自动映射的框架，配置Java对象与sql语句执行结果的对应关系，多表关联关系配置简单
- Hibernate 是一个全表映射的框架，配置Java对象与数据库表的对应关系，多表关联关系配置复杂

SQL优化和移植性

- Hibernate 对SQL语句封装，提供了日志、缓存、级联（级联比 MyBatis 强大）等特性，此外还提供 HQL（Hibernate Query Language）操作数据库，数据库无关性支持好，但会多消耗性能。如果项目需要支持多种数据库，代码开发量少，但SQL语句优化困难。
- MyBatis 需要手动编写 SQL，支持动态 SQL、处理列表、动态生成表名、支持存储过程。开发工作量相对大些。直接使用SQL语句操作数据库，不支持数据库无关性，但sql语句优化容易。

开发难易程度和学习成本

- Hibernate 是重量级框架，学习使用门槛高，适合于需求相对稳定，中小型的项目，比如：办公自动化系统
- MyBatis 是轻量级框架，学习使用门槛低，适合于需求变化频繁，大型的项目，比如：互联网电子商务系统

**总结**

MyBatis 是一个小巧、方便、高效、简单、直接、半自动化的持久层框架，

Hibernate 是一个强大、方便、高效、复杂、间接、全自动化的持久层框架。

## 缓存

### 简介

> Mybatis的一级、二级缓存

一级缓存：基于PerpetualCache 的HashMap本地缓存，其存储作用域为Session，当Session flush或close之后，该Session中的所有Cache就将清空，默认打开一级缓存

二级缓存与一级缓存其机制相同，默认也是采用PerpetualCache，HashMap存储，不同在于其存储作用域为Mapper(Namespace)，并且可自定义存储源，如Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类要实现Serializable序列化接口（可用来保存对象的状态），可在它的映射文件中配置<cache/>

对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear

### 一级缓存

一级缓存也叫本地缓存：SQLSession

基于PerpetualCache的HashMap本地缓存，其存储作用域为Session，当Session flush或者close之后，该Session中的所有Cache都会被清空，默认打开一级缓存

### 二级缓存

- 二级缓存也叫全局缓存，一级缓存的作用域太低了，所以诞生了二级缓存
- 默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache
- 工作机制
  - 一个会话查询一条数据，这个数据会被放在当前会话的一级缓存中
  - 如果当前会话关闭了，这分会话对应的一级缓存就没有了，但是我们想要的是会话关闭了，一级缓存中数据被保存到二级缓存中
  - 新的会话查询信息，就会直接从二级缓存中获取数据
  - 不同的mapper查出的数据会放在自己对应的缓存中

> 对于缓存数据的更新机制，当某一个作用域（一级缓存或者二级缓存namespaces）进行了CUD操作后，默认该作用域下的所有select中的缓存将被clear

开启二级缓存的步骤

全局配置参数

```xml
<setting name="cacheEnabled" value="true"/>
```

开启二级缓存

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>

```

- 按照先进先出的淘汰策略缓存项
- 缓存容量为512个对象引用
- 缓存每隔60s刷新一次
- 缓存返回的对象是写安全的，即在外部修改对象不会影响到缓存内部存储对象

测试二级缓存

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/13 22:36
 * @description：测试缓存
 */
public class Test {

    @org.junit.Test
    public void test() {
        SqlSession sqlSession1 = MybatisUtil.getSqlSession();
        SqlSession sqlSession2 = MybatisUtil.getSqlSession();
        EmpMapper empMapper1 = sqlSession1.getMapper(EmpMapper.class);
        EmpMapper empMapper2 = sqlSession2.getMapper(EmpMapper.class);
        Employee user1 = empMapper1.select(1);


        Employee user2 = empMapper2.select(1);
        sqlSession1.close();

        sqlSession2.close();
        System.out.println(user1 == user2);


    }
}
```

## 高级查询

### 建表

根据数据库设计的第三范式，设计两张表，t_class（班级表）和t_stu（学生表），学生与班级之间是一对一的关系，班级与学生之间是多对一的关系（加外键），具体实现：

> 班级表

```sql
create table t_class(
id int(10) primary key auto_increment,
cname varchar(20) not null
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

> 学生表

```sql
create table t_stu (
id int(10) primary key auto_increment,
name varchar(20) not null,
cid int(10) not null,
constraint fk_stu_class foreign key(cid) references t_class(id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

> 插入数据

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210522105003.png" alt="image-20210522104958868" style="zoom:80%;" />

> 创建实体类

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/22 10:51
 * @description：班级类
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class Clazz {
    private int id;
    private String cname;
    private List<Student> students;
}

/**
 * @author ：zsy
 * @date ：Created 2021/5/22 10:50
 * @description：学生类
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class Student {
    private int id;
    private String cname;
    private Clazz clazz;
}
```

### 一对一查询

Mybatis实现一对一查询的方式：通过在<resultMap>里面配置association节点完成一对一类的查询

- 联合查询：几个表联合查询，只查询一次
- 嵌套查询：先查一个表，根据这个表里面的结果的外键id，去再另外一个表里面查询数据

#### 联合查询

两种写法	

```xml
<mapper namespace="school.xauat.mapper.StudentMapper">

    <resultMap id="studentResultMap" type="student">
        <id property="id" column="id>"></id>
        <result property="name" column="name"></result>
        <association property="clazz" column="cid" javaType="clazz">
            <id property="id" column="id"></id>
            <result property="cname" column="cname"></result>
        </association>
    </resultMap>


    <select id="selectById" resultMap="studentResultMap">
        select s.id, s.name, c.id as cid, c.cname from t_stu s left join t_class c on s.cid = c.id where s.id = #{id}
    </select>
</mapper>
```

> 区别于上面，class对象需要关联

```xml
<mapper namespace="school.xauat.mapper.StudentMapper">

    <resultMap id="studentResultMap" type="student">
        <id property="id" column="id>"></id>
        <result property="name" column="name"></result>
        <association property="clazz" column="cid" resultMap="class">
        </association>
    </resultMap>
    
    <resultMap id="class" type="clazz">
        <id property="id" column="id"></id>
        <result property="cname" column="cname"></result>
    </resultMap>

    
    <select id="selectById" resultMap="studentResultMap">
        select s.id, s.name, c.id as cid, c.cname from t_stu s left join t_class c on s.cid = c.id where s.id = #{id}
    </select>
</mapper>
```

#### 嵌套查询

```xml
<mapper namespace="school.xauat.mapper.StudentMapper">

    <resultMap id="studentResultMap" type="student">
        <id property="id" column="id>"></id>
        <result property="name" column="name"></result>
        <association property="clazz" column="cid" javaType="clazz" select="selectClass">
            <id property="id" column="id"></id>
            <result property="name" column="cname"></result>
        </association>
    </resultMap>

    <select id="selectById" resultMap="studentResultMap">
        select * from t_stu where id = #{id}
    </select>
    
    <select id="selectClass" resultType="clazz">
        select * from t_class where id = #{id}
    </select>
</mapper>
```

### 一对多查询

```xml
<mapper namespace="school.xauat.mapper.ClassMapper">
    
    <resultMap id="ClassResultMap" type="clazz">
        <id property="id" column="classId"></id>
        <result property="cname" column="cname"></result>
        <collection property="students" column="classId" javaType="ArrayList" select="getStudentById"></collection>
    </resultMap>
    
    <resultMap id="studentResultMap" type="student">
        <id property="id" column="id"></id>
        <result property="name" column="name"></result>
    </resultMap>


    <select id="selectById" resultMap="ClassResultMap">
        select id as classId, cname from t_class where id = #{id}
    </select>

    <select id="getStudentById" resultMap="studentResultMap">
        select id, name from t_stu where cid = #{classId}
    </select>
</mapper>
```

### 面试题

#### Mybatis实现一对一，一对多有几种方式，怎么操作？

- 有联合查询和嵌套查询，联合查询是几个表联合查询，只查询一次，通过在resultMap里面的association，collection节点配置一对一，一对多的类就可以完成
- 嵌套查询是先查一个表，根据这个表里面的结果的外键id，去再另外一个表里面查询数据，也是通过配置association，collection，但另外一个表的查询通过select节点配置。

#### Mybatis是否可以映射Enum枚举类

Mybatis可以映射枚举类，不单可以映射枚举类，Mybatis可以映射任何对象到表的一列上，映射方式为自定义一个TypeHandler，实现TypeHandler的setParameter和getResult()接口方法。

TypeHandler有两个作用，一是完成从javaType至jdbcType的转换，二是完成jdbcType至javaType的转换，体现为setParameter()和getResult()两个方法，分别代表设置sql问号占位符参数和获取列查询结果

## [扩展问题](https://blog.csdn.net/u011277123/article/details/87934463?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162126865316780262550872%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162126865316780262550872&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduend~default-1-87934463.nonecase&utm_term=MyBatis%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8&spm=1018.2226.3001.4450)

### #{}和${}的区别

#{}是占位符，预编译处理，${}是字符串替换，没有预编译处理

Mybatis在处理#{}时，会将sql中的#{}替换为?，**调用preparedStatement的set方法来赋值**，并对变量自动加上单引号

Mybatis在执行${}时，会把${}替换为变量的值，相当于JDBC的Statement编译，不会加单引号

**变量替换后，#{} 对应的变量自动加上单引号 ‘’；变量替换后，${} 对应的变量不会加上单引号**

> 使用#{}可以有效的防止SQL注入，提高系统的安全性

#{} 的变量替换是在DBMS 中；${} 的变量替换是在 DBMS 外

```sql 
select ${param} from table_name where id = #{}
```

### 什么是Sql注入

SQL注入即是指web应用程序对用户输入数据的合法性没有判断或过滤不严，攻击者可以在web应用程序中事先定义好的查询语句的结尾上添加额外的SQL语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。

### 为什么需要预编译

> 定义

SQL 预编译指的是数据库驱动在发送 SQL 语句和参数给 DBMS 之前对 SQL 语句进行编译，这样 DBMS 执行 SQL 时，就不需要重新编译

> 为什么需要预编译

JDBC 中使用对象 PreparedStatement 来抽象预编译语句，使用预编译。预编译阶段可以优化 SQL 的执行。预编译之后的 SQL 多数情况下可以直接执行，DBMS 不需要再次编译，越复杂的SQL，编译的复杂度将越大，预编译阶段可以合并多次操作为一个操作。同时预编译语句对象可以重复利用。把一个 SQL 预编译后产生的 PreparedStatement 对象缓存下来，下次对于同一个SQL，可以直接使用这个缓存的 PreparedState 对象。Mybatis默认情况下，将对所有的 SQL 进行预编译

### 通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？	

Dao接口（Mapper接口）；接口的权限名，就是映射文件中的namespace的值；接口的方法，就是映射文件**MappedStatement的id值；接口方法内的参数，就是传递给sql的参数。**Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement，举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面id = findStudentById的MappedStatement。在Mybatis中，每一个<select>、<insert>、<update>、<delete>标签，都会被解析为一个MappedStatement对象。参数在SQL执行的时候，需要通过反射拿到接口中的参数，给SQL赋值，执行查询

Dao接口中的方法都是不可以重载的，因为全限名 + 方法名的保存和寻找策略

Dao接口的工作原理就是JDK的动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回

# MyBatis源码

## Mybatis的编程步骤

1. 创建SqlSessionFactory
2. 通过SqlSessionFactory创建SqlSession
3. 通过SqlSession执行数据库操作
4. 调用session.commit()提交事务
5. 调用session.close()关闭会话

## Mybatis的工作原理

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210518001321.png" alt="MyBatis工作原理" style="zoom:80%;" />

- 读取Mybatis配置文件：mybatis-config.xml为Mybatis的全局配置文件，配置了Mybatis的运行环境等信息，例如数据库连接信息
- 加载映射文件：映射文件即SQL映射文件，该文件中配置了操作数据库的SQL语句，需要在Mybatis配置文件mybatis-config.xml中加载。mybatis-config.xml文件中可以加载多个映射文件，每个文件对应数据库中的一张表
- 构建会话工厂：通过Mybatis的环境等配置信息构建会话工厂SqlSessionFatory
- 创建会话对象：有工厂创建SqlSession对象，该对象包含了执行的SQL语句的所有方法
- Executor执行器：Mybatis底层定义了一个Executor接口来操作数据库，它将根据SqlSession传递的参数动态地生成需要执行的SQL语句，同时负责查询缓存的维护
- MappedStatement对象：在Executor接口的执行方法中有一个MappedStatement类型的参数，该参数是对映射信息的封装，用于存储要映射的SQL语句的id、参数信息
- 输入参数映射：输入参数类型可以是Map、List等集合类型，也可以是基本数据类型和POJO类型，输入参数映射过程类似于JDBC对preparedStatement对象设置参数的过程
- 输出结果映射：输出结果类型可以是 Map、 List 等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程

## Mybatis的功能架构

Mybatis的功能架构分为三层：

- API接口层：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接到调用请球就会调用数据处理层来完成具体的数据处理
- 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果的映射处理等。它主要的目的是根据调用的请求完成一次数据库操作
- 基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是公用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210519223858.webp" alt="Mybatis框架架构" style="zoom:80%;" />

这张图从上往下看。MyBatis的初始化，会从mybatis-config.xml配置文件，解析构造成Configuration这个类，就是图中的红框。

(1)加载配置：配置来源于两个地方，一处是配置文件，一处是Java代码的注解，将SQL的配置信息加载成为一个个MappedStatement对象（包括了传入参数映射配置、执行的SQL语句、结果映射配置），存储在内存中。

(2)SQL解析：当API接口层接收到调用请求时，会接收到传入SQL的ID和传入对象（可以是Map、JavaBean或者基本数据类型），Mybatis会根据SQL的ID找到对应的MappedStatement，然后根据传入参数对象对MappedStatement进行解析，解析后可以得到最终要执行的SQL语句和参数。

(3)SQL执行：将最终得到的SQL和参数拿到数据库进行执行，得到操作数据库的结果。

(4)结果映射：将操作数据库的结果按照映射的配置进行转换，可以转换成HashMap、JavaBean或者基本数据类型，并将最终结果返回。

## [配置文件解析过程](https://thinkwon.blog.csdn.net/article/details/114808962)

### 配置文件解析入口

> 单独使用Mybatis时，第一步需要根据配置文件创建SqlSessionFactory对象

```java
String resource = "Mybatis-config.xml";
InputStream inputStream = null;
try {
    inputStream = Resources.getResourceAsStream(resource);
} catch (IOException e) {
    e.printStackTrace();
}
SqlSessionFatory sqlSessionFatory = new SqlSessionFactoryBuilder().build(inputStream);
```

通过resource加载配置文件，得到一个输入流然后再通过 SqlSessionFactoryBuilder 对象的`build`方法构建 SqlSessionFactory 对象

>build：构建SqlSessionFactory对象

```java
public SqlSessionFactory build(InputStream inputStream) {
    //方法重载
  return build(inputStream, null, null);
}

public SqlSessionFactory build(InputStream inputStream, String environment) {
  return build(inputStream, environment, null);
}

public SqlSessionFactory build(InputStream inputStream, Properties properties) {
  return build(inputStream, null, properties);
}

public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
  try {
      //创建配置文件解析器对象
    XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      //返回解析后的SqlSessionFatory对象
      //parser.parse()读取mybatis-config.xml配置文件，生成configuration对象
    return build(parser.parse());
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error building SqlSession.", e);
  } finally {
    ErrorContext.instance().reset();
    try {
      inputStream.close();
    } catch (IOException e) {
      // Intentionally ignore. Prefer previous error.
    }
  }
}

public SqlSessionFactory build(Configuration config) {
    // 创建 DefaultSqlSessionFactory
  return new DefaultSqlSessionFactory(config);
}   
```

> parse：通过创建的XMLConfigBuilder对象解析配置文件

```java
public Configuration parse() {
  if (parsed) {
    throw new BuilderException("Each XMLConfigBuilder can only be used once.");
  }
  parsed = true;
    //解析mybatis-config中的<configuration>标签
  parseConfiguration(parser.evalNode("/configuration"));
  return configuration;
}
```

>parseConfiguration：/configuration对应配置文件中的<configuration>标签，进一步解析该标签下的具体配置

```java
private void parseConfiguration(XNode root) {
    try {
        // 解析 properties 配置
        propertiesElement(root.evalNode("properties"));

        // 解析 settings 配置，并将其转换为 Properties 对象
        Properties settings = settingsAsProperties(root.evalNode("settings"));

        // 加载 vfs
        loadCustomVfs(settings);

        // 解析 typeAliases 配置
        typeAliasesElement(root.evalNode("typeAliases"));

        // 解析 plugins 配置
        pluginElement(root.evalNode("plugins"));

        // 解析 objectFactory 配置
        objectFactoryElement(root.evalNode("objectFactory"));

        // 解析 objectWrapperFactory 配置
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));

        // 解析 reflectorFactory 配置
        reflectorFactoryElement(root.evalNode("reflectorFactory"));

        // settings 中的信息设置到 Configuration 对象中
        settingsElement(settings);

        // 解析 environments 配置
        environmentsElement(root.evalNode("environments"));

        // 解析 databaseIdProvider，获取并设置 databaseId 到 Configuration 对象
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));

        // 解析 typeHandlers 配置
        typeHandlerElement(root.evalNode("typeHandlers"));

        // 解析 mappers 配置
        mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```

到此，将一个Mybatis的主配置文件已经加载完毕

### 解析 environments 配置

> Mybatis的数据源和事务管理器配置在environments下

```java
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
        </dataSource>
    </environment>
</environments>
```

```java
private String environment;

private void environmentsElement(XNode context) throws Exception {
    if (context != null) {
        if (environment == null) {
            // 获取 default 属性
            environment = context.getStringAttribute("default");
        }
        for (XNode child : context.getChildren()) {
            // 获取 id 属性
            String id = child.getStringAttribute("id");
            /*
             * 检测当前 environment 节点的 id 与其父节点 environments 的属性 default 
             * 内容是否一致，一致则返回 true，否则返回 false
             */
            if (isSpecifiedEnvironment(id)) {
                // 解析 transactionManager 节点，逻辑和插件的解析逻辑很相似，不在赘述
                TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
                // 解析 dataSource 节点，逻辑和插件的解析逻辑很相似，不在赘述
                DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
                // 创建 DataSource 对象
                DataSource dataSource = dsFactory.getDataSource();
                Environment.Builder environmentBuilder = new Environment.Builder(id)
                    .transactionFactory(txFactory)
                    .dataSource(dataSource);
                // 构建 Environment 对象，并设置到 configuration 中
                configuration.setEnvironment(environmentBuilder.build());
            }
        }
    }
}
```

## [映射文件解析过程](https://thinkwon.blog.csdn.net/article/details/115423167)

>mapperElement：解析Mapper映射文件

```java
private void mapperElement(XNode parent) throws Exception {
  if (parent != null) {
      //循环遍历所有<Mappers>标签下的子标签	
    for (XNode child : parent.getChildren()) {
        //如果子标签中包含<package>标签  	<package name="***"/>
      if ("package".equals(child.getName())) {	
          //获取package节点中的name属性	
        String mapperPackage = child.getStringAttribute("name");	
          //从指定包中查找mapper接口，根据接口mapper解析映射文件
        configuration.addMappers(mapperPackage);
      } else {	
          //获取mapper标签下的三个属性
        String resource = child.getStringAttribute("resource");
        String url = child.getStringAttribute("url");
        String mapperClass = child.getStringAttribute("class");
          //<mapper resource="***"/> 类路径下的Mapper.xml
        if (resource != null && url == null && mapperClass == null) {
          ErrorContext.instance().resource(resource);
          InputStream inputStream = Resources.getResourceAsStream(resource);
          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
          mapperParser.parse();
            //<mapper url="***"/> 远程的Mapper.xml
        } else if (resource == null && url != null && mapperClass == null) {
          ErrorContext.instance().resource(url);
          InputStream inputStream = Resources.getUrlAsStream(url);
          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
          mapperParser.parse();
            //<mapper mapperClass="***"/> 类似于package
        } else if (resource == null && url == null && mapperClass != null) {
          Class<?> mapperInterface = Resources.classForName(mapperClass);
          configuration.addMapper(mapperInterface);
        } else {
            //以上三个参数，只能出现一个，出现多个，将抛出异常
          throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
        }
      }
    }
  }
}
```

以上代码的代码逻辑：根据节点的属性值加载映射文件

- 从文件系统中加载映射文件
- 通过URL的方式加载和解析映射文件
- 通过mapper接口加载映射信息，映射信息可以配置在注解中，也可以配置在配置文件中（即<package>配置）
- 通过包扫描的方式获取到某个包下的所有类，使用第三种凡是为每个类解析映射文件

在 MyBatis 中，通过注解配置映射信息的方式是有一定局限性的

> 因为最初设计时，MyBatis 是一个 XML 驱动的框架。配置信息是基于 XML 的，而且映射语句也是定义在 XML 中的。而到了 MyBatis 3，就有新选择了。MyBatis 3 构建在全面且强大的基于 Java 语言的配置 API 之上。这个配置 API 是基于 XML 的 MyBatis 配置的基础，也是新的基于注解配置的基础。注解提供了一种简单的方式来实现**简单映射语句**，而不会引入大量的开销。

> **注意：** 不幸的是，**Java 注解的的表达力和灵活性十分有限**。尽管很多时间都花在调查、设计和试验上，**最强大的 MyBatis 映射并不能用注解来构建**——并不是在开玩笑，的确是这样。

### 映射文件的解析入口

```java
// -☆- XMLMapperBuilder
public void parse() {
    // 检测映射文件是否已经被解析过
    if (!configuration.isResourceLoaded(resource)) {
        // 解析 mapper 节点
        configurationElement(parser.evalNode("/mapper"));
        // 添加资源路径到“已解析资源集合”中
        configuration.addLoadedResource(resource);
        // 通过命名空间绑定 Mapper 接口
        bindMapperForNamespace();
    }

    // 处理未完成解析的节点
    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
}

```

### 解析映射文件

> 解析的整个过程，可以根据流程图读源码

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210518210929.png" alt="image-20210518210928011" style="zoom:80%;" />

> 以下是一个mapper.xml映射文件

```xml
<mapper namespace="school.xauat.mapper.EmpMapper">

    <cache
            eviction="FIFO"
            flushInterval="60000"
            size="512"
            readOnly="true"/>


    <resultMap id="empResult" type="employee">
        <id property="id" column="id"></id>
        <result property="last_name" column="last_name"></result>
        <result property="gender" column="gender"></result>
        <result property="email" column="email"></result>
        <!--...-->
    </resultMap>

    <sql id="table">
        tbl_employee
    </sql>

    <select id="selectById" resultMap="empResult">
        SELECT
            id, last_name, gender, email
        FROM
            <include refid="table"></include>
        WHERE
            id = #{id}
    </select>
</mapper>
```

解析映射文件包含了解析`<cache>`，`<resultMap>`，`<sql>` 以及 `<select | insert | update | delete>` 等，主要考虑解析SQL语句，Mybatis将解析后的SQL语句封装成一个个MappedStatement对象，存放在MappedStatements集合中

#### MappedStatements

MappedStatement是对SQL语句的封装

```java
public final class MappedStatement {

  private String resource;
  private Configuration configuration;
  private String id;
  private Integer fetchSize;
  private Integer timeout;
  private StatementType statementType;
  private ResultSetType resultSetType;
  private SqlSource sqlSource;
  private Cache cache;
  private ParameterMap parameterMap;
  private List<ResultMap> resultMaps;
  private boolean flushCacheRequired;
  private boolean useCache;
  private boolean resultOrdered;
  private SqlCommandType sqlCommandType;
  private KeyGenerator keyGenerator;
  private String[] keyProperties;
  private String[] keyColumns;
  private boolean hasNestedResultMaps;
  private String databaseId;
  private Log statementLog;
  private LanguageDriver lang;
  private String[] resultSets;

  MappedStatement() {
    // constructor disabled
  }
}
```

> MappedStatements是一个继承了HashMap的哈希表，重写了put方法，在put的时候，如果key存在，则抛出异常

```java
protected static class StrictMap<V> extends HashMap<String, V>
```

```java
//重写了put方法，不能添加重复的key，否则报错
public V put(String key, V value) {
    //如果key存在，则抛出异常（key是当前MappedStatement对象的id）
    //***Mapper.id -> Mapper
  if (containsKey(key)) {
    throw new IllegalArgumentException(name + " already contains value for " + key
        + (conflictMessageProducer == null ? "" : conflictMessageProducer.apply(super.get(key), value)));
  }
  if (key.contains(".")) {
    final String shortKey = getShortName(key);
    if (super.get(shortKey) == null) {
      super.put(shortKey, value);
    } else {
      super.put(shortKey, (V) new Ambiguity(shortKey));
    }
  }
  return super.put(key, value);
}
```

#### 解析SQL

>addMappers

```java
public void addMappers(String packageName, Class<?> superType) {
  ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
  resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    //得到packageName包下所有类
  Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
    //遍历所有的类
  for (Class<?> mapperClass : mapperSet) {
    addMapper(mapperClass);
  }
}
```

>addMapper：对应page包下的Mapper接口寻找类路径下的Mapper.xml文件

```java
public <T> void addMapper(Class<T> type) {
    //如果类的类型是接口才会遍历
  if (type.isInterface()) {
      //如果configuration中已经包含该类型，则抛出异常
    if (hasMapper(type)) {
      throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
    }
    boolean loadCompleted = false;
    try {
      knownMappers.put(type, new MapperProxyFactory<>(type));
      // It's important that the type is added before the parser is run
      // otherwise the binding may automatically be attempted by the
      // mapper parser. If the type is already known, it won't try.
        //注解解析器，解析@Select、@Update等注解
      MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
        //解析Mapper接口为Mapper.xml	
      parser.parse();
      loadCompleted = true;
    } finally {
      if (!loadCompleted) {
        knownMappers.remove(type);
      }
    }
  }
}
```

>parse：解析package下的Mapper方法和xml文件

```java
public void parse() {
  String resource = type.toString();
  if (!configuration.isResourceLoaded(resource)) {
      //将xml文件中的sql加载到configuration中
    loadXmlResource();
      //标志位，告诉configuration已经加载过了resource
    configuration.addLoadedResource(resource);
    assistant.setCurrentNamespace(type.getName());
    parseCache();
    parseCacheRef();
      //加载完成xml文件后，加载Mapper中的方法，主要解析Mapper方法上的注解
    Method[] methods = type.getMethods();
    for (Method method : methods) {
      try {
        // issue #237
        if (!method.isBridge()) {
          parseStatement(method);
        }
      } catch (IncompleteElementException e) {
        configuration.addIncompleteMethod(new MethodResolver(this, method));
      }
    }
  }
  parsePendingMethods();
}
```

>loadXmlResource：将xml文件中的sql加载到configuration中

```java
private void loadXmlResource() {
    //判断当前xmlMapper是否已经加载
    //防止加载两次	
  if (!configuration.isResourceLoaded("namespace:" + type.getName())) {
      //xmlResource为当前的Mapper.xml文件
    String xmlResource = type.getName().replace('.', '/') + ".xml";
    // #1347
    InputStream inputStream = type.getResourceAsStream("/" + xmlResource);
    if (inputStream == null) {
      try {
          //尝试获得Mapper.xml文件
        inputStream = Resources.getResourceAsStream(type.getClassLoader(), xmlResource);
      } catch (IOException e2) { 
          //忽视，用户可能不需要xml文件，以注解的形式存在
      }
    }
      //如果xml文件存在，解析xml文件，加入到configuration中
    if (inputStream != null) {
      XMLMapperBuilder xmlParser = new XMLMapperBuilder(inputStream, assistant.getConfiguration(), xmlResource, configuration.getSqlFragments(), type.getName());
        //解析xml文件中的sql加入到configuration中
      xmlParser.parse();
    }
  }
}
```

> parseStatement：将Mapper类中方法上的注解转化为一个MappedStatement对象

```java
void parseStatement(Method method) {
    ....
        //省去部分代码，主要内容通过反射，拿到方法上的注解
    ....
      //创建一个MappedStatement对象，将该对象加入到configuration中
    assistant.addMappedStatement(
        mappedStatementId,
        sqlSource,
        statementType,
        sqlCommandType,
        fetchSize,
        timeout,
        // ParameterMapID
        null,
        parameterTypeClass,
        resultMapId,
        getReturnType(method),
        resultSetType,
        flushCache,
        useCache,
        // TODO gcode issue #577
        false,
        keyGenerator,
        keyProperty,
        keyColumn,
        // DatabaseID
        null,
        languageDriver,
        // ResultSets
        options != null ? nullOrEmpty(options.resultSets()) : null);
  }
}
```

>addMappedStatement：将MappedStatement存入StrictMap

```java
public void addMappedStatement(MappedStatement ms) {
    //mappedStatements是一个StrictMap，继承了HashMap重写了put方法
  mappedStatements.put(ms.getId(), ms);
}
```

> parse：解析xml文件中内容

```java
public void parse() {
  if (!configuration.isResourceLoaded(resource)) {
    configurationElement(parser.evalNode("/mapper"));
    configuration.addLoadedResource(resource);
    bindMap	perForNamespace();
  }
  parsePendingResultMaps();
  parsePendingCacheRefs();
  parsePendingStatements();
}

private void configurationElement(XNode context) {
    try {
        // 获取 mapper 命名空间
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.equals("")) {
            throw new BuilderException("Mapper's namespace cannot be empty");
        }

        // 设置命名空间到 builderAssistant 中
        builderAssistant.setCurrentNamespace(namespace);

        // 解析 <cache-ref> 节点
        cacheRefElement(context.evalNode("cache-ref"));

        // 解析 <cache> 节点
        cacheElement(context.evalNode("cache"));

        // 已废弃配置，这里不做分析
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));

        // 解析 <resultMap> 节点
        resultMapElements(context.evalNodes("/mapper/resultMap"));

        // 解析 <sql> 节点
        sqlElement(context.evalNodes("/mapper/sql"));

        // 解析 <select>、...、<delete> 等节点
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
    }
}

```

解析SQL语句主要做的事情

1. 解析<include>节点
2. 解析<selectKey>节点
3. 解析SQL，获取SqlSource
4. 构建MappedStatement对象

> sqlSource分为动态SQL(${})和静态SQL(#{})，对于静态SQL会将sql中的#{}解析为?、动态SQL不会解析

## 创建会话SqlSession对象

> 具体就不细究了，太多了

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
      //获取configuration中的环境
    final Environment environment = configuration.getEnvironment();
      //创建事务工厂
        final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      //SQL语句执行器
    final Executor executor = configuration.newExecutor(tx, execType);
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    closeTransaction(tx); // may have fetched a connection so lets call close()
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

## [SQL执行过程](https://thinkwon.blog.csdn.net/article/details/115603376)

> SqlSession 是通过 JDK 动态代理的方式为接口生成代理对象的。在调用接口方法时，方法调用会被代理逻辑拦截。在代理逻辑中可根据方法名及方法归属接口获取到当前方法对应的 SQL 以及其他一些信息，拿到这些信息即可进行数据库操作

### 执行代理逻辑

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  return mapperRegistry.getMapper(type, sqlSession);
}
```

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    //根据传入的Mapper.class找到mapper代理工程对象ProxyFactory代理工厂对象
  final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
  if (mapperProxyFactory == null) {
    throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
  }
  try {
      //返回Mapper接口的代理对象
    return mapperProxyFactory.newInstance(sqlSession);
  } catch (Exception e) {
    throw new BindingException("Error getting mapper instance. Cause: " + e, e);
  }
}
```

> 使用动态代理执行

```java
protected T newInstance(MapperProxy<T> mapperProxy) {
    //mapperProxy实现了InvocationHandler接口，调用了invoke方法
  return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
}
```

>invoke

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
      //如果方法是Object类的方法例如toString()等，不需要走代理，直接执行Object方法
    if (Object.class.equals(method.getDeclaringClass())) {
      return method.invoke(this, args);
    } else if (isDefaultMethod(method)) {//判断方法是否是默认方法，java8新特性，是默认方法，直接执行
      return invokeDefaultMethod(proxy, method, args);
    }
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
    //获取mapperMethod对象（包含当前执行方法的MapperStatement对象（SQL语句）、返回值）
  final MapperMethod mapperMethod = cachedMapperMethod(method);
    //执行该方法
  return mapperMethod.execute(sqlSession, args);
}
```

代理逻辑会首先检测被拦截的方法是不是定义在 Object 中的，比如 equals、hashCode 方法等。对于这类方法，直接执行即可。除此之外，MyBatis 从 3.4.2 版本开始，对 JDK 1.8 接口的默认方法提供了支持，具体就不分析了。完成相关检测后，紧接着从缓存中获取或者创建 MapperMethod 对象，然后通过该对象中的 execute 方法执行 SQL。在分析 execute 方法之前，我们先来看一下 MapperMethod 对象的创建过程。MapperMethod 的创建过程看似普通，但却包含了一些重要的逻辑，所以不能忽视

### 创建MapperMethod对象

```java
public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
    //通过mapperInterface、method 在config中获取当前执行方法的MappedStatement
  this.command = new SqlCommand(config, mapperInterface, method);
    //当前方法的标志（返回值、参数列表）
  this.method = new MethodSignature(config, mapperInterface, method);
}
```

#### **SqlCommand 对象**

```java
public static class SqlCommand {

    private final String name;
    private final SqlCommandType type;

    public SqlCommand(Configuration configuration, Class<?> mapperInterface, Method method) {
        final String methodName = method.getName();
        final Class<?> declaringClass = method.getDeclaringClass();
        // 解析 MappedStatement
        MappedStatement ms = resolveMappedStatement(mapperInterface, methodName, declaringClass, configuration);
        
        // 检测当前方法是否有对应的 MappedStatement
        if (ms == null) {
            // 检测当前方法是否有 @Flush 注解
            if (method.getAnnotation(Flush.class) != null) {
                // 设置 name 和 type 遍历
                name = null;
                type = SqlCommandType.FLUSH;
            } else {
                /*
                 * 若 ms == null 且方法无 @Flush 注解，此时抛出异常。
                 * 这个异常比较常见，大家应该眼熟吧
                 */ 
                throw new BindingException("Invalid bound statement (not found): "
                    + mapperInterface.getName() + "." + methodName);
            }
        } else {
            // 设置 name 和 type 变量
            name = ms.getId();
            type = ms.getSqlCommandType();
            if (type == SqlCommandType.UNKNOWN) {
                throw new BindingException("Unknown execution method for: " + name);
            }
        }
    }
}

```

SqlCommand获取当前方法的MappedStatement对象

#### **创建 MethodSignature 对象**

```java
public static class MethodSignature {

    private final boolean returnsMany;
    private final boolean returnsMap;
    private final boolean returnsVoid;
    private final boolean returnsCursor;
    private final Class<?> returnType;
    private final String mapKey;
    private final Integer resultHandlerIndex;
    private final Integer rowBoundsIndex;
    private final ParamNameResolver paramNameResolver;

    public MethodSignature(Configuration configuration, Class<?> mapperInterface, Method method) {

        // 通过反射解析方法返回类型
        Type resolvedReturnType = TypeParameterResolver.resolveReturnType(method, mapperInterface);
        if (resolvedReturnType instanceof Class<?>) {
            this.returnType = (Class<?>) resolvedReturnType;
        } else if (resolvedReturnType instanceof ParameterizedType) {
            this.returnType = (Class<?>) ((ParameterizedType) resolvedReturnType).getRawType();
        } else {
            this.returnType = method.getReturnType();
        }
        
        // 检测返回值类型是否是 void、集合或数组、Cursor、Map 等
        this.returnsVoid = void.class.equals(this.returnType);
        this.returnsMany = configuration.getObjectFactory().isCollection(this.returnType) || this.returnType.isArray();
        this.returnsCursor = Cursor.class.equals(this.returnType);
        // 解析 @MapKey 注解，获取注解内容
        this.mapKey = getMapKey(method);
        this.returnsMap = this.mapKey != null;
        /*
         * 获取 RowBounds 参数在参数列表中的位置，如果参数列表中
         * 包含多个 RowBounds 参数，此方法会抛出异常
         */ 
        this.rowBoundsIndex = getUniqueParamIndex(method, RowBounds.class);
        // 获取 ResultHandler 参数在参数列表中的位置
        this.resultHandlerIndex = getUniqueParamIndex(method, ResultHandler.class);
        // 解析参数列表
        this.paramNameResolver = new ParamNameResolver(configuration, method);
    }
}

```

MethodSignature 即方法签名，顾名思义，该类保存了一些和目标方法相关的信息。比如目标方法的返回类型，目标方法的参数列表信息等

### 执行execute方法

>execute：方法执行

```java
public Object execute(SqlSession sqlSession, Object[] args) {
  Object result;
    //根据SQL类型执行，响应的数据库操作
  switch (command.getType()) {
    case INSERT: {
        //对用户传入的参数进行转换
      Object param = method.convertArgsToSqlCommandParam(args);
        //执行插入操作，rowCountResult 方法用于处理返回值
      result = rowCountResult(sqlSession.insert(command.getName(), param));
      break;
    }
    case UPDATE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
      break;
    }
    case DELETE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
      break;
    }
    case SELECT:
          //根据目标方法的返回值类型进行查询操作
      if (method.returnsVoid() && method.hasResultHandler()) {
          /*
                 * 如果方法返回值为 void，但参数列表中包含 ResultHandler，表明使用者
                 * 想通过 ResultHandler 的方式获取查询结果，而非通过返回值获取结果
                 */
        executeWithResultHandler(sqlSession, args);
        result = null;
      } else if (method.returnsMany()) {
          //执行查询，返回多个结果
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) {
          // 执行查询操作，并将结果封装在 Map 中返回
        result = executeForMap(sqlSession, args);
      } else if (method.returnsCursor()) {
          // 执行查询操作，并返回一个 Cursor 对象
        result = executeForCursor(sqlSession, args);
      } else {
        Object param = method.convertArgsToSqlCommandParam(args);
          // 执行查询操作，并返回一个结果
        result = sqlSession.selectOne(command.getName(), param);
        if (method.returnsOptional()
            && (result == null || !method.getReturnType().equals(result.getClass()))) {
          result = Optional.ofNullable(result);
        }
      }
      break;
    case FLUSH:
          // 执行刷新操作
      result = sqlSession.flushStatements();
      break;
    default:
      throw new BindingException("Unknown execution method for: " + command.getName());
  }
    // 如果方法的返回值为基本类型，而返回值却为 null，此种情况下应抛出异常
  if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
    throw new BindingException("Mapper method '" + command.getName()
        + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
  }
  return result;
}
```

>convertArgsToSqlCommandParam：解析获取Mapper接口中的参数映射

```java
public Object convertArgsToSqlCommandParam(Object[] args) {
  return paramNameResolver.getNamedParams(args);
}

public Object getNamedParams(Object[] args) {
  final int paramCount = names.size();
  if (args == null || paramCount == 0) {
    return null;
      //如果参数的值是一个并且没有注解，直接返回参数的值
  } else if (!hasParamAnnotation && paramCount == 1) {
    return args[names.firstKey()];
  } else {
    final Map<String, Object> param = new ParamMap<>();
    int i = 0;
    for (Map.Entry<Integer, String> entry : names.entrySet()) {
      param.put(entry.getValue(), args[entry.getKey()]);
      // add generic param names (param1, param2, ...)
      final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
      // ensure not to overwrite parameter named with @Param
      if (!names.containsValue(genericParamName)) {
        param.put(genericParamName, args[entry.getKey()]);
      }
      i++;
    }
    return param;
  }
}
```

getNamedParams方法执行的三种情况

- 如果参数只有一个，并且没有注解，直接返回参数值
- 如果多个参数
  - 参数有注解，key为注解的值
  - 参数无注解，jdk1.7或1.8，key为arg0，arg1..
  - 参数无注解，jdk1.8配置了-parameters，key为反射的参数名
- 最后处理，key为param0、param1

{

​	name:zhangsan

​	age:12

​	param1:zhangsan

​	param2:12

}

> Mybatis这样做，极大的方便了Mybatis的扩展性，方便集成于别的框架	

### SQL语句的执行分析

Mybatis对以下指令提供了支持

- 查询语句：SELECT
- 更新语句：INSERT/UPDATE/DELETE
- 存储过程：CALL

#### 查询语句的执行过程

- executeWithResultHandler
- executeForMany
- executeForMap
- executeForCursor

这些方法内部都调用了 SqlSession 中的一些 select* 方法，比如 selectList、selectMap、selectCursor 等，针对不同的返回值，需要有专门的处理方法

> 下面主要分析SelectOne的执行过程

```java
public <T> T selectOne(String statement, Object parameter) {
  // Popular vote was to return null on 0 results and throw exception on too many.
    //调用selectList获取结果
  List<T> list = this.selectList(statement, parameter);
  if (list.size() == 1) {
      //返回结果
    return list.get(0);
      //如果查询结果大于1，抛出异常
  } else if (list.size() > 1) {
    throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
  } else {
    return null;
  }
}
```

由以上代码可以得到，selectOne方法调用selectList方法，并返回第一个元素

```java
public <E> List<E> selectList(String statement, Object parameter) {
  return this.selectList(statement, parameter, RowBounds.DEFAULT);
}	

public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
  try {
      //获取MappedStatement对象
    MappedStatement ms = configuration.getMappedStatement(statement);
      //调用query方法
    return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

#### Executor类

Executor是一个接口它的实现类如下

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210520171302.png" alt="image-20210520171259562" style="zoom:80%;" />

默认情况下，executor 的类型为 CachingExecutor，该类是一个装饰器类，用于给目标 Executor 增加二级缓存功能。那目标 Executor默认情况下是 SimpleExecutor

```java
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    //获取BoundSql
  BoundSql boundSql = ms.getBoundSql(parameterObject);
    //创建CacheKey
  CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
    //重载
  return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

##### Mybatis的基本执行器

- Simpleexecutor：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象
- ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象
- BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同

> Executor的这些特点，都严格限制在SqlSession生命周期范围内

##### Mybatis如何指定使用哪一种执行器

```java
SqlSession sqlSession = new SqlSessionFactoryBuilder().build(inputStream).openSession(ExecutorType.REUSE)
```

在Mybatis配置文件中，在设置（settings）可以指定默认的ExecutorType执行器类型，也可以手动给DefaultSqlSessionFactory的创建SqlSession的方法传递ExecutorType类型参数，如SqlSession openSession(ExecutorType execType)

配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新


### 总结

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210520173059.jpeg" alt="img" style="zoom:80%;" />

在Mybatis中，SQL执行过程的实现代码是由层次的，每层都有相应的功能，比如，SqlSession 是对外接口的接口，因此它提供了各种语义清晰的方法，供使用者调用。Executor 层做的事情较多，比如一二级缓存功能就是嵌入在该层内的。StatementHandler 层主要是与 JDBC 层面的接口打交道。至于 ParameterHandler 和 ResultSetHandler，一个负责向 SQL 中设置运行时参数，另一个负责处理 SQL 执行结果，它们俩可以看做是 StatementHandler 辅助类。最后看一下右边横跨数层的类，Configuration 是一个全局配置类，很多地方都依赖它。MappedStatement 对应 SQL 配置，包含了 SQL 配置的相关信息。BoundSql 中包含了已完成解析的 SQL 语句，以及运行时参数等。

## 分析一个SQL语句的执行过程

1. 创建SqlSessionFactory对象、解析Mybatis的主配置文件，主要解析Mapper映射文件（<Mapper>标签）`将每一个sql语句解析成一个与之对应的mappedStatement对象，存放在Configuration对象的mappedStatements属性中，key是命名空间+id`
   1. 如果是配置的是package，解析package包下的所有接口
      1. 类路径下寻找所有接口对应的xml文件，解析xml
      2. 解析完xml后，解析所有的方法，主要解析所有方法内的注解
   2. 如果是配置的是mapper，解析mapper标签
      1. resource配置
      2. url配置
      3. class配置
2. 创建SqlSession对象，基于configuration中的Environment（数据源、连接池）创建事务工厂，准备一个执行器
3. 通过SqlSession对象获取Mapper映射接口对象，从MapperProxyFactoryMapper映射工厂中拿到对应的Mapper接口的代理对象
4. 执行接口对应的方法，其实是调用该Mapper接口代理对象的invoke方法，invoke方法的主要步骤
   1. 对于Object类和default方法，直接放行
   2. 对于增删改查方法，创建一个cachedMapperMethod对象，里面包含该方法对应的SQL语句（MappedStatement对象）和返回值类型
   3. 使用执行器执行SQL语句查询结果
      1. 反射获取方法对应的参数值的映射
      2. 静态SQL：调用preparedStatement预编译SQL（BoundSql）
      3. 动态SQL：将${}换为对应的值
   4. 使用typeHandler将查询到的结果转换为想要的返回类型

> Mybatis中Sql的状态

SqlSource：在解析配置文件的时候生成的Sql，对于静态Sql，将#{}换为?；对于动态Sql不做任何改变

BoundSql：具体在执行的时候生成的Sql，对于动态Sql，将${}替换为对应参数的值；对于动态Sql，调用preparedStatement编译Sql，为?赋值。

# 造轮子

## 模拟MyBatis解析SQL的过程

将sqlSource解析为BoundSql

```java
/**
 * @author ：zsy
 * @date ：Created 2021/5/15 20:32
 * @description：模拟MyBatis执行过程
 */
interface UserMapper {
    @Select("select * from t_user where id = #{id} AND name = #{name}")
    List<User> selectAll(Integer id, String name);
}
public class Application {
    public static void main(String[] args) {
        //JDK动态代理
        UserMapper userMapper = (UserMapper) Proxy.newProxyInstance(Application.class.getClassLoader(),
                new Class<?>[]{UserMapper.class}, (proxy, method, args1) -> {
            Select annotation = method.getAnnotation(Select.class);
            String[] sql = annotation.value();
            Map<String, Object> nameArgMap = builderMethodMap(method, args1);
            if(annotation != null) {
                String parseSql = parseSql(sql[0], nameArgMap);
                System.out.println(parseSql);

            }
            return null;
        });
        userMapper.selectAll(1, "zhangsan");
    }

    /**
     * 解析SQL，JDBC中给占位符传参
     * @param sql
     * @param nameArsMap
     * @return
     */
    public static String parseSql(String sql, Map<String, Object>nameArsMap) {
        StringBuilder parseSQL = new StringBuilder();
        int len = sql.length();
        for(int i = 0; i < len; i++) {
            char c = sql.charAt(i);
            if(c == '#') {
                int nextIndex = i + 1;
                if(nextIndex >= len)
                    throw new RuntimeException(String.format("无法解析#\nsql:%s\nindex:%d", parseSQL.toString(),nextIndex));
                char nextChar = sql.charAt(nextIndex);
                if(nextChar != '{')
                    throw new RuntimeException(String.format("这里应该是#{\nsql:%s\nindex:%d", parseSQL.toString(),nextIndex));
                StringBuilder argSB = new StringBuilder();
                i = parseSQLArg(argSB, sql, nextIndex);

                String argName = argSB.toString();
                Object argValue = nameArsMap.get(argName);
                if(argValue == null) {
                    throw new RuntimeException(String.format("找不到参数：%s", argName));
                }
                parseSQL.append(argValue.toString());
                continue;
            }
            if(c != '#') {
                parseSQL.append(c);
            }
        }
        return parseSQL.toString();
    }

    /**
     * 解析获得参数类型
     * @param argSB
     * @param sql
     * @param nextIndex
     * @return
     */
    private static int parseSQLArg(StringBuilder argSB, String sql, int nextIndex) {
        nextIndex++;
        for(; nextIndex < sql.length(); nextIndex++) {
            char c = sql.charAt(nextIndex);
            if (c != '}') {
                argSB.append(c);
            } else if (c == '}') {
                return nextIndex;
            }
        }
        throw new RuntimeException(String.format("缺少右括号\nsql:%s\nindex:%d", argSB.toString(),nextIndex));
    }

    /**
     * 获取method参数名和参数值的映射关系
     * @param method
     * @param args
     * @return
     */
    public static Map<String, Object> builderMethodMap(Method method, Object[]args) {
        Parameter[] parameters = method.getParameters();
        Map<String, Object> nameArgMap = new HashMap<>();
        int index[] = {0};
        //匿名类对外界属性默认为final，不能修改，使用数组跳过JVM检测
        Arrays.asList(parameters).forEach(parameter -> {
            String name = parameter.getName();
            nameArgMap.put(name, args[index[0]++]);
        });
        return nameArgMap;
    }
}

select * from t_user where id = 1 AND name = zhangsan	
```

> 存在问题

由于IDE缘故，无法获取参数名称，只能获取arg0、arg1

解决办法 pom中配置<compilerArgs>，配置完成，先clean 再complie

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <compilerArgs>
                        <arg>-parameters</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

# [Mybatis面试题](https://thinkwon.blog.csdn.net/article/details/101292950)

> 部分自己补充

## Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？

不同的xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；毕竟namespace不是必须的，只是最佳实践而已。

原因就是namespace+id是作为Map<String, MappedStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。有了namespace，自然id就可以重复，namespace不同，namespace+id自然也就不同。

## 简述Mybatis的Xml映射文件和Mybatis内部数据结构之间的映射关系？

Mybatis会将所有的xml配置信息都封装到All-in-One重量级对象Configuration对象中，在xml映射中，<parameterMap>标签会被解析为ParameterMap对象，其每个子元素被解析为ParameterMapping对象，<resultMap>

标签会被解析为ResultMap对象，其每个子元素会被解析为ResultMapping对象。每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签均会被解析为MappedStatement对象，标签内的sql会被解析为BoundSql对象。

## Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

- 第一种是使用<resultMap>标签，逐一定义列名和对象属性名之间的映射关系。
- 第二种是使用sql列的别名功能，将列别名书写为对象属性名，比如T_NAME AS NAME，对象属性名一般是name，小写，但是列名不区分大小写，Mybatis会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成T_NAME AS NaMe，Mybatis一样可以正常工作。

> 有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

## Mybatis是如何进行分页的？分页插件的原理是什么？

- 逻辑分页：Mybatis使用RowBounds对象进行分页，它是一次性查询很多数据，然后在数据中再进行检索
- 物理分页：自己手写SQL分页或者使用PageHelper，去数据库查询指定条数的分页数据形式（[PageHelper的最佳实践](https://github1s.com/dunk-code/ssm-ssm_crud/blob/HEAD/src/main/java/school/xauat/controller/EmpController.java)）

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，添加对应的物理分页语句和物理分页参数

举例：select * from student

拦截 sql 后重写为：select t.* from （select * from student）t  limit 0，10

> RowBounds并不是一次查询出所有的结果，RowBounds 表面是在“所有”数据中检索数据，其实并非是一次性查询出所有数据，因为Mybatis是对JDBC的封装，在JDBC驱动中有一个Fetch Size的配置，它规定了每次最多从数据库查询出来多少条数据，如果你要查询更多数据，它会在执行next()的时候，去查询更多的数据

## Mybatis如何根据映射器(mapper.xml文件)生成sql语句？

- XMLConfigBuilder解析映射xml文件时，会将每一个sql语句和其配置的内容保存起来
- Mybatis中的一条Sql与它相关的配置信息是由MappedStatement、SqlSource和BoundSql三部分组成的
  - MappedStatement的作用是保存一个映射节点（select|insert|delete|update）的内容，他是一个类，包括许多我们配置的Sql、Sql的id、ResultMap等重要配置内容，同时还有一个重要的属性：SqlSource。Mybatis通过读取MappedStatement来获得某条SQL配置的所有信息
  - SqlSource是提供BoundSql对象的地方，它是一个接口，使用它可以获得一个BoundSql对象
  - BoundSql是一个结果对象，是建立Sql和参数的地方

## Mybatis一级缓存和二级缓存的区别

- 一级缓存：SqlSession范围的缓存，默认开启，在同一个SqlSession中，执行相同的Sql查询时，第一次会去数据库查询，并写入缓存中，第二次会直接从缓存中取，Mybatis的内部缓存使用一个HashMap，key为hashcode + statemenId + sql语句，value为查询出来的结果集映射成的Java对象，**两次查询Sql中间如果有增删改操作，会清空缓存**
- 二级缓存：Mapper级别的缓存，跨SqlSession，默认没有开启，SqlSession1第一次调用Mapper下的SQL进行查询后会将结果存放在Mapper对应的二级缓存区域，SqlSession2再调用Mapper中相同的SQL查询时，会去对应的二级缓存内取结果。**如果SqlSession3执行commit提交，将会清空该Mapper映射下的二级缓存区域的数据。**

## **MyBatis** 是否支持延迟加载？延迟加载的原理是什么？

Mybatis支持延迟加载，设置LazyLoadingEnabled = true即可

基本原理：使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，延迟加载的原理是调用的时候触发加载，而不是初始化的时候就加载信息，比如调用a.getB().getName()，这个时候发现a.getB()的值为null，此时会单独触发事先先保存好的关联B对象的SQL，先查询出来B，再调用s.setB(b)，而这个时候再调用a.getB().getName()就有值了

## **简述 Mybatis 的插件运行原理，以及如何编写一个插件？**

- Mybatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、
  Executor 这 4 种接口的插件，Mybatis 通过动态代理，为需要拦截的接口生成代理对象以实
  现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是
  InvocationHandler 的 invoke()方法，当然，只会拦截那些你指定需要拦截的方法。
- 实现 Mybatis 的 Interceptor 接口并复写 intercept()方法，然后在给插件编写注解，指定
  要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。

