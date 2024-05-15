![image-20210425195014249](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429235421.png)

# 基础部分

修改数据库密码

```sql
alter user root@localhost  IDENTIFIED BY '123456';
```

跳过密码验证，在my.cnf中添加skip-grant-tables

## 数据库三大范式

- 第一范式：每个列不可再拆分
- 第二范式：在第一范式的基础上，非主键完全依赖于主键，而不是依赖于主键的一部分
- 第三范式：在第二范式的基础上，非主键只依赖于主键，不依赖于其他非主键

> 在设计数据库结构的时候，要尽量遵守三范式，如果不遵守，必须有足够的理由，比如性能，事实上，我们经常会为了性能而妥协数据库的设计。

## 数据类型

![image-20210429232319687](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429232321.png)

> 使用策略：

- 对于经常变更的数据来说，CHAR比VARCHAR好，因为CHAR不容易产生碎片
- 对于非常短的列，CHAR比VARCHAR在存储空间上更有效率。使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存。
- 尽量避免使用TEXT/BLOB类型，查询时会使用临时表，导致严重的性能开销。

## SQL语句

数据定义语言DDL（Data Ddefinition Language）：CREATE，DROP，ALTER。主要为以上操作 即对逻辑结构等有操作的，其中包括表结构，视图和索引

数据查询语言DQL（Data Query Language）SELECT。这个较为好理解 即查询操作，以select关键字。各种简单查询，连接查询等 都属于DQL

数据操纵语言DML（Data Manipulation Language）INSERT，UPDATE，DELETE。主要为以上操作 即对数据进行操作的，对应上面所说的查询操作 DQL与DML共同构建了多数初级程序员常用的增删改查操作。而查询是较为特殊的一种 被划分到DQL中。

数据控制功能DCL（Data Control Language）GRANT，REVOKE，COMMIT，ROLLBACK。主要为以上操作 即对数据库安全性完整性等有操作的，可以简单的理解为权限控制等。

## 键

- 超键：在关系中能唯一标识元组的属性集称为关系模式的超键。一个属性可以为作为一个超键，多个属性组合在一起也可以作为一个超键。超键包含候选键和主键。
- 候选键：是最小超键，即没有冗余元素的超键。
- 主键：数据库表中对存储数据对象予以唯一和完整标识的数据列或数据属性的组合。一个数据列只能有一个主键，且主键的取值不能缺失，既不能为null
- 外键：在一个表中存在的另一个表的主键称为此表的外键

## SQL约束

- NOT NULL: 用于控制字段的内容一定不能为空（NULL）。
- UNIQUE: 控件字段内容不能重复，一个表允许有多个 Unique 约束。
- PRIMARY KEY: 也是用于控件字段内容不能重复，但它在一个表只允许出现一个。
- FOREIGN KEY: 用于预防破坏表之间连接的动作，也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。
- CHECK: 用于控制字段的值范围。

## 关联查询

- 交叉连接（CROSS JOIN）
- 内连接（INNER JOIN）
- 外连接（LEFT JOIN / RIGHT JOIN）
- 联合查询（UNION / UNION ALL）
- 全连接 FULL JOIN

内连接分为三类

- 等值连接：ON A.id=B.id
- 不等值连接：ON A.id > B.id
- 自连接：SELECT * FROM A T1 INNER JOIN A T2 ON T1.id=T2.pid

外连接（LEFT JOIN/RIGHT JOIN）

- 左外连接：LEFT OUTER JOIN, 以左表为主，先查询出左表，按照ON后的关联条件匹配右表，没有匹配到的用NULL填充，可以简写成LEFT JOIN
- 右外连接：RIGHT OUTER JOIN, 以右表为主，先查询出右表，按照ON后的关联条件匹配左表，没有匹配到的用NULL填充，可以简写成RIGHT JOIN

>笛卡尔积现象：当两张表进行连接查询的时候，没有任何条件进行限制，最终的查询结果条数是两张表记录条数的乘积。

联合查询

- 就是把多个结果集集中在一起，UNION前的结果为基准，需要注意的是联合查询的列数要相等，相同的记录行会合并
- 如果使用UNION ALL，不会合并重复的记录行
- 效率 UNION 高于 UNION ALL

全连接（FULL JOIN）

- MySQL不支持全连接
- 可以使用LEFT JOIN 和UNION和RIGHT JOIN联合使用

## 事务

### 什么是数据库事务

事务是一个不可分割的数据库操作序列，也是数据库并发控制的基本单位，其执行的结果必须使数据库从一种一致性状态转变到另一种一致性状态，事务是逻辑上的一组操作，要么都执行，要么都不执行。

### 事务的四大特性（ACID）

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210603215151.png" alt="image-20210603215147242" style="zoom:80%;" />



- 原子性（Atomicity）：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用
- 一致性（Consistency）：一致性是对数据可见性的约束，保证在一个事务中的多次操作的数据中间状态对其他事务不可见的。 因为这些中间状态，是一个过渡状态，与事务的开始状态和事务的结束状态是不一致的；一致性是指在**事务开始之前和事务结束以后**，**数据库的完整性约束没有被破坏**。这是说数据库事务不能破坏**关系数据的完整性**以及**业务逻辑上的一致性**，如A给B转账，不论转账的事务操作是否成功，其两者的存款总额不变
- 隔离性（Isolation）：并发访问数据库，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的
- 持久性（Durability）：一个事务提交后，它对数据库中数据的改变是持久的，即使数据库发生故障也不应该有其他任何影响

### 脏读、幻读、不可重复读

- 脏读（Dirty Read）：某个事务更新一份数据，另一个事务读取到了同一份事务，前一个事务回滚了操作，则后一个事务读取的数据就会不正确
- 不可重复读（Non-repeatable read）：一个事务的两次查询之中的结果不一致，可能两次查询中间插入了一个事务**更新**的原有数据
- 幻读（(Phantom Read)）：在一个事务两次查询中数据列数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有**几列数据是它先前所没有的**。

> 幻读和不可重复读的区别

不可重复读的重点是修改:

同样的条件, 你读取过的数据, 再次读取出来发现值不一样了

幻读的重点在于新增或者删除

同样的条件, 第1次和第2次读出来的记录数不一样

### 事务的隔离级别

为了达到事务的四大特性，数据库定义了4种不同的事务隔离级别，由低到高依次为Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。

|     隔离级别     | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| READ-UNCOMMITTED |  √   |     √      |  √   |
|  READ-COMMITTED  |  ×   |     √      |  √   |
| REPEATABLE-READ  |  ×   |     ×      |  √   |
|   SERIALIZABLE   |  ×   |     ×      |  ×   |

> MySQL默认采用的是REPEATABLE-READ隔离级别，Oracle默认采用的是READ-COMMITTED隔离级别

## 子查询

> 什么是子查询

- 条件：一条SQL语句的查询结果作为另一条查询语句的条件或查询结果
- 嵌套：多条SQL语句嵌套使用，内部的SQL查询语句称为子查询

## 扩展问题

### in和exists的区别

MySQL的in语句是吧外表和内表做hash连接，而exists语句是对外表做loop循环，每次loop循环再对内表进行查询，一直大家都认为exists比in语句的效率要高，这种说法其实是不准确的。这个是要区分环境的。

- 如果查询的两个表大小相当，那么用in和exists差别不大。
- 如果两个表中一个较小，一个是大表，则子查询表大的用exists，子查询表小的用in。
- not in 和not exists：如果查询语句使用了not in，那么内外表都进行全表扫描，**没有用到索引；而not exists的子查询依然能用到表上的索引**。所以无论那个表大，用not exists都比not in要快。

### varchar与char的区别

#### char的特点

- char表示定长字符串，长度固定的
- 如果插入数据的长度小于char的长度，则用空格填充
- 因为长度固定，所以存取速度要比varchar快很多，甚至能快50%，但正是因为其长度固定，所以会占用多余空间，是空间换时间的做法
- 对于char来说，最多能存放255个字符，和编码无关

#### varchar的特点

- varchar表示可变长字符串，长度是可变的
- 插入的数据是多长，就按照多长来存储；
- varchar在存取方面与char相反，它存取慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；
- 对于varchar来说，最多能存放的字符个数为65532

>总之，结合性能角度（char更快）和节省磁盘空间角度（varchar更小），具体情况还需具体来设计数据库才是妥当的做法。

varchar(50)中50的含义

最多存放50个字符，varchar(50)和(200)存储hello所占的空间一样，但后者在排序时消耗更多的内存，应为order by col采用fixed_length计算col长度(Memory引擎也是)在早期MySQL版本中，50代表字节数，现在代表字符数。

### int(20)中20的含义

是指显示字符的长度，20表示最大显示宽度为20，但仍占4个字节存储，存储范围不变，不影响内部存储，只是影响带zerofill定义的int，前面补多少个0，易于显示。

> 设计的意义：只是规定一些工具来显示字符的个数，int(1)和int(20)存储和计算是一样的

### float和double的区别

- FLOAT类型数据可以存储至多8位十进制数，并在内存中占4字节。
- DOUBLE类型数据可以存储至多18位十进制数，并在内存中占8字节。

### drop、delete和truncate的区别

|          | Delete                                   | Truncate                       | Drop                                               |
| -------- | ---------------------------------------- | ------------------------------ | -------------------------------------------------- |
| 类型     | 属于DML                                  | 属于DDL                        | 属于DDL                                            |
| 回滚     | 可回滚                                   | 不可回滚                       | 不可回滚                                           |
| 删除内容 | 表结构还在，删除表的全部或者一部分数据行 | 表结构还在，删除表中的所有数据 | 从数据库中删除表，所有的数据行，索引和权限也会删除 |
| 删除速度 | 删除速度慢，需要逐行删除                 | 删除速度快                     | 删除速度最快                                       |

>因此，在不再需要一张表的时候，用drop；在想删除部分数据行时候，用delete；在保留表而删除所有数据的时候用truncate。

### Union和Union All的区别

- 如果使用UNION ALL，不会合并重复的记录行
- 效率 UNION 高于 UNION ALL

# 进阶部分

## 引擎

> 存储引擎（Storage engine）：MySQL中的数据、索引以及其他对象是如何存储的，是一套文件系统的实现，存储引擎是基于表的，不是基于库的

查看MySQL数据库默认的存储引擎

show engines 查询当前数据库支持的存储引擎

```sql
showvariableslike'%storage_engine%'；
```

### 存储引擎

- InnoDB引擎：InnoDB引擎提供了对数据库ACID事务的支持，并且还提供了行级锁和外键约束。它设计的目标就是处理大数据容器的数据库系统
- MyISAM引擎（原本MySQL的默认引擎）：不提供事务支持，也不支持行级锁和外键
- MEMORY引擎：所有数据都在内存中，数据的处理速度快，但是安全性不高

### InnoDB引擎

#### 支持事务

```sql
start transaction;

commit

rollback
```

#### 外键约束

在创建外键的时候，要求必须有对应的索引，字表在创建的时候，也会自动创建对应的索引

```sql
create table country_innodb( 
    country_id int NOT NULL AUTO_INCREMENT, 
    country_name varchar(100) NOT NULL, 
    primary key(country_id) 
)ENGINE=InnoDB DEFAULT CHARSET=utf8; 

create table city_innodb( 
    city_id int NOT NULL AUTO_INCREMENT, 
    city_name varchar(50) NOT NULL, 
    country_id int NOT NULL, 
    primary key(city_id), key idx_fk_country_id(country_id),
    CONSTRAINT `fk_city_country` FOREIGN KEY(country_id) REFERENCES country_innodb(country_id) ON DELETE RESTRICT ON UPDATE CASCADE 
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

创建索引的时，可以指定在删除、更新父表时，对子表进行相应的操作，包括RESTRICT、CASCADE、SET NULL、NOT ACTION

- RESTRICT、NOT ACTION相同，是指限制在子表有关联记录的情况下，父表不能更新
- CASCADE表示父表在更新或者删除时更新或者删除子表对应的记录
- SET NULL表示父表在更新或者删除时，子表的对应字段被SET NULL

子表的外键指定是ON DELETE RESTRICT ON UPDATE CASCADE 方式的， 那么在主表删除记录的时候， 如果子表有对应记录， 则不允许删除， 主表在更新记录的时候， 如果子表有对应记录， 则子表对应更新 。

#### 存储方式

InnoDB存储表和索引有以下两种方式：

1. 使用共享空间存储，这种方式创建的表的表结构保存在*.frm文件中，数据和索引保存在innodb_data_home_dir 和 innodb_data_fifile_path定义的表空间中，可以是多个文件。
2. 使用多表空间存储，这种方式创建的表的表结构仍然存在*.frm文件中，但是每个数据和索引单独保存在*.ibd中

### MyISAM和InnoDB区别

|                                                              |                            MyISAM                            |                            InnoDB                            |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|                           存储结构                           | frm-表格定义、MYD（MYData）-数据文件、MYI（MYINDEX）-索引文件 | 所有的表都保存在同一个数据文件中（也可能是多个文件，或是独立的表空间文件）表大小受限于操作系统文件的大小，一般为2GB |
|                           存储空间                           |                     可被压缩，存储空间小                     | 表需要更多的存储空间，他会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引 |
|                     可移植性、备份及恢复                     | 由于MyISAM的数据是以文件的形式存储，所以在跨平台的数据转移中会很方便。在备份和恢复时可单独针对某个表进行操作 | 免费的方案可以是拷贝数据文件、备份 binlog，或者用 mysqldump，在数据量达到几十G的时候就相对痛苦了 |
|                           文件格式                           |                        *.MYI、  *.MYD                        |      表结构存在*.frm文件，数据和索引都是集中存储的*.ibd      |
|                             外键                             |                            不支持                            |                           **支持**                           |
|                             事务                             |                            不支持                            |                           **支持**                           |
| 锁支持（锁是避免资源争用的一个机制，MySQL锁对用户几乎是透明的） |                           表级锁定                           |       **行级锁定**、表级锁定，锁定力度小**并发能力高**       |
|                            SELECT                            |                          MyISAM更优                          |                                                              |
|                    INSERT、UPDATE、DELETE                    |                                                              |                          InnoDB更优                          |
|                        索引的实现方式                        |                      B+树索引，MyISAM表                      |                 B+树索引、InnoDB是索引组织表                 |
|                           哈希索引                           |                            不支持                            |                             支持                             |
|                           全文索引                           |                            不支持                            |                             支持                             |
|                         记录存储顺序                         |                      按记录插入顺序保存                      |                      按主键大小有序插入                      |
|                       select count(*)                        |  MyISAM更快，因为MyISAM内部维护了一个计数器，可以直接调取。  |                                                              |

### MyISAM索引与InnoDB索引的区别？

- InnoDB索引是聚簇索引，MyISAM是非聚簇索引
- InnoDB主键索引的叶子节点存储着行数据，因此主键索引效率非常高
- MyISAM索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据
- InnoDB非主键索引的叶子节点存储的是主键和其他索引的列数据，因此查询时做到覆盖索引会非常高效

### InnoDB引擎的4大特性

- 插入缓冲（insert buffer)
- 二次写(double write)
- 自适应哈希索引(ahi)
- 预读(read ahead)

### 存储引擎选择

如果没有特别的需求，使用默认的`Innodb`即可。

MyISAM：以读写插入为主的应用程序，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不高。比如博客系统、新闻门户网站。允许少量的数据丢失

Innodb：用于事务处理应用程序，支持外键，日过要保证数据的完整性；在并发条件下要求数据的一致性，数据操作除了插入和查询外，还包很多更新、删除操作，那么InnoDB存储引擎是比较合适的，InnoDB存储引擎降低由于删除和更新导致的锁定，还可以确保事务的完整提交和回滚，对于类似于计费系统或者财务系统等对数据准确性要求比较高的系统，InnoDB是最合适的选择

MEMORY：将所有数据保存在RAM中，在需要快速定位记录和其他类似数据环境下，可以提供几块的访问。MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，其次是要确保表的数据可以恢复，数据库异常终止后表中的数据是可以恢复的。MEMORY表通常用于更新不太频繁的小表，用以快速得到访问结果。

MERGE：用于将一系列等同的MyISAM表以逻辑方式组合在一起，并作为一个对象引用他们。MERGE表的优点在于可以突破对单个MyISAM表的大小限制，并且通过将不同的表分布在多个磁盘上，可以有效的改善MERGE表的访问效率。这对于存储诸如数据仓储等VLDB环境十分合适。

## 索引

### 索引概述

MySQL官方对索引的定义为：索引（indexes）是帮助MySQL高级获取数据的数据结构（有序）。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这个数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引

一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往存储在磁盘上的文件中。（可能存储在单独的索引文件中，也可能和数据一起存储在数据文件中）

我们通常所说的索引，包括聚集索引、覆盖索引、组合索引、前缀索引、唯一索引等，没有特定说明，默认使用的是B+树结构组织的索引

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210426094645.png" alt="image-20210426094644479" style="zoom:67%;" />

为了加快Col2的查询，可以维护一个右边的二叉搜索树，每个节点分别包含一索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉搜索树获取到响应数据。

> 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上

### 索引的优势劣势

#### 优势

- 提高了数据检索的效率，降低了数据库IO成本
- 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗
  - 被索引的列会自动进行排序，包括【单例索引】和【组合索引】，只是组合索引的排序要复杂
  - 如果按照索引列的顺序进行排序，对应的order by语句来说，效率会提高很多

#### 劣势

- 空间方面：索引也是一张表，该表中保存了主键与索引字段，并指向实体类的记录，所以索引也是要占用空间的
- 时间方面：创建索引和维护索引要耗费时间，虽然索引大大提高了查询效率，同时却降低了更新表的速率。如对表INSERT、UPDATE、DELETE。因为更新时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息

### 索引的结构

> 参考：[一文搞懂MySQL索引所有知识点（建议收藏）](https://blog.csdn.net/qq_35190492/article/details/109257302?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161943785316780357275236%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=161943785316780357275236&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-109257302.nonecase&utm_term=mysql)

#### 分析AVL树的问题

> 为什么不使用平衡二叉树

1. 时间复杂度和树高相关，树有多高就需要检索多少次，每个节点的读取，都对应一次磁盘IO操作。树的高度就等于每次查询数据是磁盘IO操作的次数。在数据量很大时，查询性能就会很差
2. 平衡二叉树不支持范围查询快速查找，范围查询时需要从根节点多次遍历，查询效率不高。

> 索引是MySQL的存储引擎层中实现的，而不是在服务器层实现的，所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型的

- BTREE索引：最常见的索引类型，大部分索引都支持B树索引

- HASH索引：只有Memory引擎支持，使用场景简单，类似于数据结构中简单实现的HASH表（散列表）一样，当我们在mysql中用哈希索引时，主要就是通过Hash算法（常见的Hash算法有**直接定址法、平方取中法、折叠法、除数取余法、随机数法**），将数据库字段数据转换成定长的Hash值，与这条数据的行指针一并存入Hash表的对应位置；如果发生Hash碰撞（两个不同关键字的Hash值相同），则在对应Hash键下以链表形式存储。当然这只是简略模拟图。

  value可以存储行记录或者行磁盘地址，Hash表在等值查询而定时候效率很高O(1)；但是不支持范围快速查找，范围查找只能通过扫描全表方式。

  <img src="C:/Users/dell/AppData/Roaming/Typora/typora-user-images/image-20210603215223953.png" alt="image-20210603215223953" style="zoom:80%;" />

- R-tree索引（空间索引）：空间索引是MyISAM引擎的一个特殊索引类型，主要用于物理空间数据类型，通常使用较少

- Full-test（全文索引）：全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从MySQL5.6版本后开始支持全文索引

|   索引    |    InnoDB     | MyISAM | Memory |
| :-------: | :-----------: | :----: | :----: |
|   BTREE   |     支持      |  支持  |  支持  |
|   HASH    |    不支持     | 不支持 |  支持  |
|  R-tree   |    不支持     |  支持  | 不支持 |
| Full-test | 5.6版本后支持 |  支持  | 不支持 |

> 聚集索引、复合索引、前缀索引、唯一索引默认都是使用B+tree树（多路搜索树，不一定是二叉的）索引，统称索引

#### 分析MySQL的读取过程

> 分析MySQL的存储情况

MySQL的数据是存储在磁盘文件中的，查询处理数据时，需要需要先把磁盘中的数据加载到内存中，磁盘IO 操作非常耗时，所以我们优化的重点就是尽量减少磁盘 IO 操作。访问二叉树的每个节点就会发生一次IO，如果想要减少磁盘IO操作，就需要尽量降低树的高度。那如何降低树的高度呢？

假设key为bigint = 8byte，每个节点有两个指针，每个指针为4byte，一个节点占用的空间16byte（8 + 4 * 2 = 16）。

因为在MySQL的InnoDB存储引擎一次IO会读取的一页（默认一页16K）的数据量，而二叉树一次IO有效数据量只有16字节，空间利用率极低。为了最大化利用一次IO空间，一个简单的想法是在每个节点存储多个元素，在每个节点尽可能多的存储数据。

每个节点可以存储1000个索引（16k/16=1000），这样就将二叉树改造成了多叉树，通过增加树的叉树，将树从高瘦变为矮胖。构建1百万条数据，树的高度只需要2层就可以（1000*1000=1百万），也就是说只需要2次磁盘IO就可以查询到数据。磁盘IO次数变少了，查询数据的效率也就提高了。

#### BTREE结构

BTREE又叫多路平衡搜索树，一颗m叉的BTree特性如下：

- 树中每个节点最多包含m个孩子
- 除根节点与叶子节点外，每个节点至少有[ceil(m / 2)]个孩子
- 若根节点不是叶子节点，则至少有两个孩子
- 所有叶子节点在同一层，叶节点具有相同的深度，叶子节点之间没有指针相连
- 节点中的元素包含键值和数据，节点中的键值从大到小排列。也就是说，在所有的节点都储存数据。
- 每个非叶子节点有n个key与n + 1个指针组成，其中[ceil(m / 2)] <= n <= m - 1

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426202259.png)

##### 插入案例

以5叉BTree为例，key的数量：2 <= n <= 4，当n > 4时，中间节点分裂到父节点，两边节点分裂。

插入 C N G A H E K Q M F W L T Z D P R X Y S

插入 C N G A 不需要分裂

![image-20210426160340400](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426160341.png)

插入H ，G向上分裂

![1](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426160848.gif)

插入E K Q不需要分裂

![image-20210426161046745](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426161048.png)

插入M，M向上分裂

![2](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426161208.gif)

插入F W L T 不需要分裂

![image-20210426161357023](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426161357.png)

插入 Z，T向上分裂

![3](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426161539.gif)

插入D P R X Y不需要分裂

![image-20210426161650682](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426161651.png)

插入 S M向上分裂

![4](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426161810.gif)

最终插入完成后

![image-20210426162325722](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426162326.png)

BTREE树和二叉树相比，查询数据的效率更高，因为对于相同的数据量来说，BTREE的层级结构比二叉树小，搜索速度快

#### 分析B树存在的问题

- B树不支持范围查询的快速查找，假如要查询一个范围的数据，需要回到根节点重新遍历查找，需要从根节点进行多次遍历，查询效率有待提高。
- 如果data存储的是行记录，行的大小会随着列数的增多，所占空间就会变大，这是，一个页中可存储的数据量就会变少，树相应就会变高，磁盘IO次数就会变大。

#### B+TREE结构

> B+Tree性质

1. n叉B+Tree最多含有n个key，不是用来保存数据而是保存数据的索引
2. B+Tree的叶子节点保存所有key的信息，及指向含这些关键字记录的指针，且叶子节点本身依key的大小排序
3. 所有的非叶子节点都可以看做key的索引部分，节点中仅含其子树中最大或最小的关键字
4. B+树中，数据对象的插入和删除仅在叶子节点上进行
5. B+树有两个头指针，一个是树的根节点，一个是最小key的叶子节点

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426203245.png)

##### B+树和B树的区别

> B+树和B树最主要的区别在于**非叶子节点是否存储数据**的问题

- B树：非叶子节点和叶子节点都会存储数据
- B+树：只有叶子节点才会存储数据，非叶子节点只存储键值，叶子检点之间使用双向指针连接，最底层的叶子节点形成了一个双向有序链表。

##### B+树查询过程

B+树的最底层叶子节点包含了所有的索引项，B+树在查找的时候，由于数据都存放在最底层的叶子节点，所以每次查询都需要检索到叶子节点才能查询到数据，所以在需要查询数据的情况下每次的磁盘的IO跟树高有直接的关系，但是从另一方面来说，由于数据都被放到了叶子节点，所以放索引的磁盘块锁存放的索引数量是会跟这增加的，所以相对于B树来说，**B+树的树高理论上情况下是比B树要矮的**。也存在索引覆盖查询的情况，在索引中数据满足了当前查询语句所需要的全部数据，此时只需要找到索引即可立刻返回，不需要检索到最底层的叶子节点。（这里需要区分的是在InnoDB中Data存储的为行数据，而MyIsam中存储的是磁盘地址。）

> **可以看到B+树可以保证等值和范围查询的快速查找，MySQL的索引就采用了B+树的数据结构。**

### 索引分类

- 主键索引：数据列不允许重复，不允许为NULL，一个表只能有一个主键。

- 唯一索引：数据列不允许重复，允许为NULL值，一个表允许多个列创建唯一索引。

- 普通索引：基本的索引类型，没有唯一性的限制，允许为NULL值。

- 全文索引：即一个索引包含多个列，是目前搜索引擎使用的一种关键技术。

- 空间索引：MySQL在5.7之后的版本支持了空间索引，而且支持OpenGIS几何数据模型。MySQL在空间索引这方面遵循OpenGIS几何数据模型规则。

- 前缀索引：在文本类型如CHAR,VARCHAR,TEXT类列上创建索引时，可以指定索引列的长度，但是数值类型不能指定。

- 按照索引数量来分类

  - 单例索引

  - 组合索引

    组合索引的使用，需要遵循最左前缀匹配原则（最左匹配原则）。一般情况下在条件允许的情况下使用组合索引代替多个单例索引使用

### MySQL的索引实现

#### MyIsam索引

##### 主键索引

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426205639.png)

表的索引存储在索引文件*.MYI中，数据文件存储在数据文件*.MYD中。

###### 等值查询

```sql
select * from user where id = 28;
```

1. 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路。（1次磁盘IO）
2. 将左子树节点加载到内存中，比较16<28<47，向下检索。（1次磁盘IO）
3. 检索到叶节点，将节点加载到内存中遍历，比较16<28，18<28，28=28。查找到值等于30的索引项。（1次磁盘IO）
4. 从索引项中获取磁盘地址，然后到数据文件user.MYD中获取对应整行记录。（1次磁盘IO）
5. 将记录返给客户端。

**磁盘IO次数：3次索引检索+记录数据检索。**

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426205949.png)

###### 范围查询

```sql
select * from user where id between 28 and 47;
```

1. 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路。（1次磁盘IO）
2. 将左子树节点加载到内存中，比较16<28<47，向下检索。（1次磁盘IO）
3. 检索到叶节点，将节点加载到内存中遍历比较16<28，18<28，28=28<47。查找到值等于28的索引项。
4. 根据磁盘地址从数据文件中获取行记录缓存到结果集中。（1次磁盘IO）
5. 我们的查询语句时范围查找，需要向后遍历底层叶子链表，直至到达最后一个不满足筛选条件。
6. 向后遍历底层叶子链表，将下一个节点加载到内存中，遍历比较，28<47=47，根据磁盘地址从数据文件中获取行记录缓存到结果集中。（1次磁盘IO）
7. 最后得到两条符合筛选条件，将查询结果集返给客户端。

**磁盘IO次数：4次索引检索+记录数据检索。**

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426210052.png)

> 在MyISAM查询时，会将索引节点缓存在MySQL缓存中，而数据依赖于操作系统自身的缓存，所以并不是每次都是走的磁盘

##### 辅助索引

在MyISAM中，辅助索引和主键索引的结构是一样的，没有任何区别，叶子节点的数据存储的都是行记录的磁盘地址。只是主键索引的键值是唯一的，而辅助索引的键值可以重复。

查询数据时，由于辅助索引的键值不唯一，可能存在多个拥有相同的记录，所以即使是等值查询，也需要按照范围查询的方式在辅助索引树中检索数据。

#### InnoDB索引

##### 主键索引（聚簇索引）

每个InnoDB表都有一个聚簇索引，聚簇索引使用B+树构建，叶子节点存储的数据是整行记录。一般情况下，聚簇索引等同于主键索引，当一个表没有创建主键索引时，InnoDB会自动创建一个ROWID字段来构建聚簇索引。具体规则

>1. 在表上定义主键，InnoDB将主键索引引用做聚簇索引
>2. 如果表没有定义主键，InnoDB会选择第一个不为NULL的唯一索引列作聚簇索引
>3. 如果以上都没有，InnoDB会使用一个6byte长整型的隐式字段ROWID子段来构建聚簇索引，该ROWID字段会在插入新行时自动递增。

**除了聚簇索引之外的所有索引都称为辅助索引，在InnoDB，辅助索引中的叶子节点存储的数据是该行的主键值，在检索时，InnoDB使用此主键值在聚簇索引中搜索记录。**

InnoDB的数据和索引存储在一个文件t_user_innodb.ibd中。InnoDB的数据组织方式，是聚簇索引。

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426213438.png)

```sql
select * from user_innodb where id = 28;
```

1. 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路。（1次磁盘IO）
2. 将左子树节点加载到内存中，比较16<28<47，向下检索。（1次磁盘IO）
3. 检索到叶节点，将节点加载到内存中遍历，比较16<28，18<28，28=28。查找到值等于28的索引项，直接可以获取整行数据。将改记录返回给客户端。（1次磁盘IO）

磁盘IO数量：3次。

##### 辅助索引

https://www.cnblogs.com/lzghyh/p/12910814.html

> 在非聚簇索引中，同一个非叶子节点下的叶子节点中的主键值不一定按照升序排列。非聚簇索引的排序取决于建索引时的顺序以及数据库系统如何处理索引数据。

除聚簇索引之外的所有索引都称为辅助索引，InnoDB的辅助索引只会存储主键而非磁盘地址

```sql
select * from t_user_innodb where age=19;
```

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426213816.png)

根据在辅助索引树中获取的主键id，到主键索引树检索数据的过程称为**回表**查询。

**磁盘IO数：辅助索引3次+获取记录回表3次**

##### 组合查询

![在这里插入图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210426214010.png)

最左匹配原则：

最左前缀匹配原则和联合索引的索引存储结构和检索方式是有关系的。

在组合索引树中，最底层的叶子节点按照第一列a列从左到右递增排列，但是b列和c列是无序的，b列只有在a列值相等的情况下小范围内递增有序，而c列只能在a，b两列相等的情况下小范围内递增有序。

就像上面的查询，B+树会先比较a列来确定下一步应该搜索的方向，往左还是往右。如果a列相同再比较b列。但是如果查询条件没有a列，B+树就不知道第一步应该从哪个节点查起。

可以说创建的idx_abc(a,b,c)索引，相当于创建了(a)、（a,b）（a,b,c）三个索引。

组合索引的最左前缀匹配原则：使用组合索引查询时，mysql会一直向右匹配直至遇到范围查询(>、<、between、like)就停止匹配。

### 覆盖索引

覆盖索引并不是说是索引结构，覆盖索引是一种很常用的优化手段。因为在使用辅助索引的时候，我们只可以拿到主键值，相当于获取数据还需要再根据主键查询主键索引再获取到数据。但是试想下这么一种情况，在上面abc_innodb表中的组合索引查询时，如果我只需要abc字段的，那是不是意味着我们查询到组合索引的叶子节点就可以直接返回了，而不需要回表。这种情况就是覆盖索引。

### 索引语法

#### 创建索引

- 在CREATE TABLE时创建索引

```sql
CREATE TABLE user_index2 (
	id INT auto_increment PRIMARY KEY,
	first_name VARCHAR (16),
	last_name VARCHAR (16),
	id_card VARCHAR (18),
	information text,
	KEY name (first_name, last_name),
	FULLTEXT KEY (information),
	UNIQUE KEY (id_card)
);

```

- 使用ALTER TABLE命令去增加索引

```sql
添加主键索引 ，索引值必须唯一，为空
ALTER TABLE table_name ADD primary key index_name (column_list);
添加普通索引
ALTER TABLE table_name ADD INDEX index_name (column_list);
ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3);创建组合索引
添加唯一索引，索引值必须是唯一的（除了NULL，ULL可能出现多次）
ALTER TABLE table_name ADD unique index_name (column_list);
ALTER TABLE table_name ADD UNIQUE (column1,column2);创建唯一组合索引
添加fulltext复合索引，用于全文索引
ALTER TABLE table_name ADD fulltext index_name (column_list);创建全文索引
```

- 使用CREATE INDEX命令创建索引

```sql
CREATE INDEX index_name ON table_name (column_list);
```

#### 查看索引	

```sql
SHOW INDEX FROM table_name/G;
SHOW INDEX FROM table_name/G; 
```

#### 删除索引

根据索引名删除普通索引、唯一索引、全文索引：`alter table 表名 drop KEY 索引名`

```sql
ALTER TABLE user_index DROP KEY name;
```

```sql
DROP INDEX index_name ON table_name;
```

删除主键索引：`alter table 表名 drop primary key`（因为主键只有一个）。这里值得注意的是，如果主键自增长，那么不能直接执行此操作（自增长依赖于主键索引）：

```sql
ALTER TABLE user_index drop primary key
```

需要取消自增长再行删除：

```sql
ALTER TABLE user_index
-- 重新定义字段
MODIFY id int,
DROP PRIMARY KEY
```

但通常不会删除主键，因为设计主键一定与业务逻辑无关。

### 索引设计的原则

索引的设计遵循一些已有的原则，创建索引的时候，尽量符合这些原则，便于提升索引的使用效率，更高效的使用索引。

- 对于查询频次较高，且数据量比较大的表简历索引
- 索引字段的选择，最佳候选列应当从where字句而定条件中提取，如where字句中的组合比较多，那么应当挑选最常用的，过滤效果好的列的组合
- 使用唯一索引，区分度越高，索引的效率越高
- 索引的数量不应该太高，索引的数量越多，维护索引的代价自然也就增加，对于DML操作比价频繁的表，索引不应该太多。另外索引过多的话，提高了MySQL在索引的选择中的代价
- 使用短索引，索引创建后也是使用硬盘存储的，因此提升索引的访问IO效率，也可以提升总体的访问效率。
- 最左前缀匹配原则，组合索引非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整
- 尽量扩展索引不要创建索引，比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可
- 定义外键的数据列一定要建立索引
- 对于定义为text、image和bit的数据类型的列不要建立索引

### 扩展问题

#### 创建索引需要注意什么

- 非空字段：应该指定列为NOT NULL，除非你想存储NULL，在MySQL中，含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值

- 取值离散大的字段：（变量各个取值之间的差异程度）的列放到联合索引的前面，可以通过count()函数查看字段的差异值，返回值越大说明字段的唯一值越多字段的离散程度高；
- 索引字段越小越好：数据库的数据存储以页为单位一页存储的数据越多一次IO操作获取的数据越大效率越高。

#### 聚簇索引和非聚簇索引

聚簇索引：将数据存储与索引放到了一块，找到索引也就找到了数据
非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，myisam通过key_buffer把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在key buffer命中时，速度慢的原因

>澄清一个概念：innodb中，在聚簇索引之上创建的索引称之为辅助索引，辅助索引访问数据总是需要二次查找，非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引，辅助索引叶子节点存储的不再是行的物理位置，而是主键值

#### 前缀索引

语法：index(field(10))，使用字段值的前10个字符建立索引，默认是使用字段的全部内容建立索引。

前提：前缀的标识度高。比如密码就适合建立前缀索引，因为密码几乎各不相同。

实操的难度：在于前缀截取的长度。

我们可以利用select count(*)/count(distinct left(password,prefixLen));，通过从调整prefixLen的值（从1自增）查看不同前缀长度的一个平均匹配度，接近1时就可以了（表示一个密码的前prefixLen个字符几乎能确定唯一一条记录）

#### 使用索引查询一定能提高性能吗

通常，通过索引查询数据比全表扫描要快，但是维护索引也需要付出代价，执行DML语句的时候每条语句将为此付出4,5次的磁盘IO。因为索引需要额外的存储空间和处理，那些不必要的索引反而会使查询时间变慢，使用查询不一定能提高查询性能，索引范围查询使用于两种情况：

- 基于一个范围的检索，一般查询返回结果集小于表中记录数的30%
- 基于非唯一性索引的检索

#### 百万级别或以上的数据如何删除

关于索引：由于索引需要额外的维护成本，因为索引文件是单独存在的文件,所以当我们对数据的增加,修改,删除,都会产生额外的对索引文件的操作,这些操作需要消耗额外的IO,会降低增/改/删的执行效率。所以，在我们删除数据库百万级别数据的时候，查询MySQL官方手册得知删除数据的速度和创建的索引数量是成正比的

1. 想要删除百万数据的时候，可以先删除索引（大概耗时3mins）
2. 然后删除其中无用的数据文件（2mins）
3. 删除完成后重新创建索引（此时数据较少）创建索引也很快（10mins）
4. 与之前直接删除，绝对是要快速很多，更别说万一删除中断,一切删除会回滚。

#### 最左前缀法则

- 顾名思义，就是最左优先，在创建多列索引时，要根据业务需求，where子句中使用最频繁的一列放在最左边。
- 最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
- =和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式

#### B树和B+树的区别

B树中，可以将键和值存放在节点内部和叶子节点；但在B+树中，内部节点都是键，没有值，叶子节点同时存放键和值。

B+树的叶子节点有一条链相连，而B树的叶子节点各自独立。

#### B树、B+的优势

B树可以在内部节点同时存储键和值，因此，把频繁访问的数据放在靠近根节点的地方将会大大提高热点数据的查询效率。这种特性使得B树在特定数据重复多次查询的场景中更加高效。

由于B+树的内部节点只存放键，不存放值，因此，一次读取，可以在内存页中获取更多的键，有利于更快地缩小查找范围。 B+树的叶节点由一条链相连，因此，当需要进行一次全数据遍历的时候，B+树只需要使用O(logN)时间找到最小的一个节点，然后通过链进行O(N)的顺序遍历即可。而B树则需要对树的每一层进行遍历，这会需要更多的内存置换次数，因此也就需要花费更多的时间

#### Hash索引和B+树索引有什么区别

> Hash索引的原理

Hash索引底层就是hash表，进行查找的时候，调用一次hash函数可以获取到相应的键值，之后进行回表查询获得实际数据。

> B+树索引原理

多路平衡查找树，对于每次查询都是从根节点出发，查找到叶子节点方可以获得所查的键值，然后根据查询判断是否需要回表查询数据。

> 区别

- hash索引进行等值查询更快(一般情况下)，但是却无法进行范围查询

因为在hash索引中经过hash函数建立索引之后，索引的顺序与原顺序无法保持一致，不能支持范围查询。而B+树的的所有节点皆遵循(左节点小于父节点，右节点大于父节点，多叉树也类似)，天然支持范围。

- hash索引不支持使用索引进行排序，原理同上
- hash索引不支持模糊查询以及多列索引的最左前缀匹配。原理也是因为hash函数的不可预测。AAAA和AAAAB的索引没有相关性。
- hash索引任何时候都避免不了回表查询数据，而B+树在符合某些条件(聚簇索引，覆盖索引等)的时候可以只通过索引完成查询。
- hash索引虽然在等值查询上较快，但是不稳定。性能不可预测，当某个键值存在大量重复的时候，发生hash碰撞，此时效率可能极差。而B+树的查询效率比较稳定，对于所有的查询都是从根节点到叶子节点，且树的高度较低。

在数据量很大的情况下，索引能够很大的提高查询的效率

### 创建表和索引

![image-20210429210115309](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429210116.png)

```sql
create index idx_seller_name_sta_addr on tb_seller(name,status,address);
```

创建name、status、address的联合索引

### 索引失效案例

- 全值匹配，对联合索引中的所有值指定具体值

该情况下索引生效，执行效率提高

![image-20210429210650675](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429210651.png)

- 最左前缀法则

> 最左前缀法则类似于爬楼梯，越右边的索引，楼层越高，例如name表示1层，status表示2层，address表示3层

下面执行语句只使用到了name索引，没有使用到address索引因为你没有办法不通过第二层直接到达第三层

![image-20210429210919027](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429210919.png)

如果违背最左前缀法则则索引失效，如果符合最左侧，但是跳出某一列，只有最左侧生效	

- 范围查询右边的列，不能使用索引

下面执行语句只用到了name和status索引,最后一个address没有使用到索引

![image-20210429211458505](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429211500.png)

- 运算操作的列，索引失效

![image-20210429212203151](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429212203.png)

- 字符串类型列不添加单引号，索引失效

status是varchar类型，匹配的时候没有使用单引号导致其以及其后的索引失效

底层匹配的时候，先进行类型转换，相当于运算，导致索引失效

![image-20210429212230304](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429235422.png)

- 尽量使用覆盖索引，避免select * 

![image-20210429213523311](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429213524.png)

>TIP :
>
>using index ：使用覆盖索引的时候就会出现 
>
>using where：在查找使用索引的情况下，需要回表去查询所需的数据 
>
>using index condition：查找使用了索引，但是需要回表查询数据 
>
>using index ; using where：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据 

- 用or分割的条件，如果or前的条件中列有索引，而后面的列中没有索引，那么涉及的索引都不被使用

![image-20210429213549109](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429213550.png)

- 以%开头的模糊查询，索引失效

![image-20210429213628332](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429213629.png)

> 解决办法使用覆盖索引

![image-20210429213728086](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429213729.png)

- 如果MySQL评估使用索引比全表查询更慢，则不适用索引

创建一个address的索引，在查询北京市的时候没有使用索引，因为表中的北京市数据相比西安市数量多，所以全表查询速度大于索引查询

![image-20210429213841472](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429213843.png)

- is NULL ，is NOT NULL 有时索引失效

与address类似，null的数量多则is null索引失效，is not null 生效

- in走索引、not in不走索引

![image-20210429214419047](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429214420.png)

尽量使用in而不是用not in

- 单例索引和复合索引的选择

尽量使用复合索引，而减少使用单例索引

```sql
create index idx_name_sta_address on tb_seller(name, status, address); 
就相当于创建了三个索引 ： 
name 
name + status 
name + status + address
```

> 创建单例索引，数据库会选择最优的索引（辨识度最高的索引）来使用，并不会使用全部索引

创建name、status、address索引，进行以下查询，使用的是address索引，因为address索引的辨识度高

![image-20210429214800215](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429214801.png)

### 查看索引使用情况

```sql
show status like 'Handler_read%'; 
show global status like 'Handler_read%';
```

## 视图

### 视图概述

视图（View）是一种虚拟存在的表，视图并不在数据库中真实存在，行和列数据来自定义视图的查询中使用的表，并且在使用视图时动态生成的。通俗来讲，视图是一条SELECT语句执行后返回的结果集，所以创建视图的时候，主要工作就落在创建这条SQL语句上，视图相对于普通的表的优势：

- 简单：使用视图的用户完全不需要关心后面对应的表结构、关联条件和筛选条件，对用户来说已经是过滤好的结果集
- 安全：使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行或者某个列，但是通过视图可以简单的实现
- 数据独立：一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，原表增加对视图没有影响；原表修改列名，则可以通过修改视图来解决，不会对访问者造成影响

### 创建视图和修改视图

创建视图

```sql
CREATE VIEW view_name AS SELECT 
```

查看创建视图的语句

```sql
SHOW CREATE VIEW view_name
```

删除视图

```sql
DROP VIEW view_name
```

修改视图

```sql
ALTER VIEW view_name AS SELECT
```

更新视图时，可以先CREATE后CREATE也可以使用REPLACE更新视图，如果更新的视图不存在，则创建视图

```sql
CREATE OR REPLACE VIEW 
```

> 更新视图

通常视图是可以更新的。更新一个视图将更新其基表（视图本身没有数据），如果对视图进行增加或者删除行，实际上是对基表增加或者删除行。

如果MySQL不能正确地确定被更新的基数据，则不允许更新（包括插入和删除），如果视图中有以下操作，则不能更新视图：

1. 分组（使用GROUP BY 和HAVING）
2. 联结
3. 子查询
4. 并
5. 聚集函数（MIN()、COUNT()、SUM()等）
6. 使用DISTINCT关键字的视图

#### 小结

视图是虚拟的表，他们包含的不是数据而是根据需要检索数据的查询。视图提供了一种MySQL的SELECT语句层次的封装，可以简化数据处理以及重新格式化基础数据或保护基础数据。

### 扩展问题

#### 为什么使用视图？什么是视图

为了提高负责SQL语句的复用性和表操作的安全性，所谓视图，本质是一种虚拟的表，物理上是不存在的，其内容与真实表相似，包含一系列带有名称的列和行数据。但是是不并不在数据库中以存储的数据值形式存在，行和列来自定义视图的查询所用的基本表，并且在具体引用视图是动态生成。

#### 视图有哪些特点

- 视图的列可以来自不同的表，是表的抽象和在逻辑意义上建立的新关系。
- 视图是由基本表(实表)产生的表(虚表)。
- 视图的建立和删除不影响基本表。
- 对视图内容的更新(添加，删除和修改)直接影响基本表。
- 当视图来自多个基本表时，不允许添加和删除数据。

#### 视图的使用场景

> 视图的根本用途：简化SQL查询，提高开发效率，如果说还有另外一个用途就是兼容老的表结构

- 重用SQL语句
- 简化复杂的SQL操作，在编写查询后，可以方便的重用它而不必知道它的基本查询细节
- 使用表的组成部分不是整个表
- 保护数据可以给用户授予表的特定部分的访问权限而不是整个表的访问权限
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据

#### 视图的缺点

- 性能：数据库必须把视图的查询转化成对基表的查询，如果这个视图是由一个负责的多表查询所定义的，那么，即使是视图的一个简单查询，数据库也把它变成一个复杂的结合体，需要花费一定的时间。
- 修改线程，当用户视图修改视图的某些行时，受到了基本表的影响，对于复杂的视图是不可修改的

## 存储过程

> 什么是存储过程

存储过程简单来说，存储过程是一个预编译的SQL语句，优点是允许模块化设计，创建一次，调用多次，可以将其视为批文件，虽然他们的作用不仅限于批处理。

### 为什么需要存储过程？

#### 存储过程的优势

- 通过把处理封装在容易使用的单元中，简化复杂的操作
- 由于不要求反复建立一系列处理步骤，保证了数据的完整性，如果所有的开发人员和应用程序都是用同一存储过程，则所使用的代码都是相同的。这一点的延伸防止了错误保证数据的一致性
- 简化对变动的管理，如果表名、列名或者业务逻辑改变，主需要改变存储过程中的代码，使用它的人员甚至不需要知道变化
- 提高性能，因为使用存储过程比使用单独的SQL语句要快，存储过程都是预编译过得，执行效率高
- 存储过程可以重复使用，减少数据库开发人员工作量
- 安全性高，执行存储过程需要一定的权限
- 存储过程的代码直接存放在数据库中，通过存储过程调用，减少网路通讯

#### 存储过程的缺陷

- 存储过程的编写比基本SQL语句复杂，编写存储过程需要更高的技能，更丰富的经验
- 移植性问题，数据库端代码当然是与数据库相关的。但是如果是做工程型项目，基本不存在移植问题。
- 如果在一个程序系统中大量的使用存储过程，到程序交付使用的时候随着用户需求的增加会导致数据结构的变化，接着就是系统的相关问题了，最后如果用户想维护该系统可以说是很难很难、而且代价是空前的，维护起来更麻烦。

### 存储过程的使用

执行存储过程

MySQL称存储过程的执行为调用，因此MySQL执行存储过程的语句是CALL，CALL接受存储过程的名字以及需要传递给他的任意参数

```sql
CALL product_name (@var1,@var2)
```

> MySQL的所有边变量都必须以@开始

创建存储过程

```sql
CREATE PROCEDURE procedure_name ([OUT,IN]var)
BEGIN
	SELECT MIN(price) 
	INTO var
	FROM table_name;
	.....
END;
```

> 命令行界面

```sql
DELIMITER $
CREATE PROCEDURE procedure_name ([OUT,IN]var)
BEGIN
	SELECT MIN(price) 
	INTO var
	FROM table_name;
	.....
END $
DELIMITER;
```

OUT：从存储过程传出

IN：传递给存储过程

INOUT：对存储过程传入和传出

INTO：保存到响应的变量

删除存储过程

```sql
DROP PROCEDURE procedure_name
```

查询存储过程

```sql
select name from mysql.proc where db='db_name';
```

```sql
show procedure status
```

### 语法

#### 变量

- DECLARE

通过DECLARE可以定义一个局部变量，该变量的作用范围只能在BEGIN...END块中

```sql
DECLARE var_name type [DEFALUt value]
```

- SET

```sql
SET var_name = expr
```

- SELECT INTO

#### 游标

为什么需要游标？

使用简单的select语句，没有办法得到第一行、下一行或前10行，也不存在每次一行的处理所有行的简单方法（相对于成批的处理它们）

有时需要在检索出来的行中前进或者后退一行或者多行，这就是使用游标的原因，**游标是一个存储在MYSQL服务器上的数据库查询**，他不是一条SELECT语句，而是被该句检索出来的结果集，存储游标之后，应用程序可以根据需要滚动或者浏览其中的数据

> MySQL只能用于存储过程（和函数）

> 游标是用来存储查询结果集的数据类型，在存储过程和函数中可以使用光标对结果集进行循环处理，光标的使用包括光标的声明、OPEN、FETCH、CLOSE，其语法如下

声明游标

```sql
DECLARE cursor_name CURSOR FOR select_statement;
```

OPEN光标

```sql
OPEN cursor_name;
```

FETCH光标

```sql
FETCH cursor_name;
```

CLOSE光标

```sql
CLOSE cursor_name;
```

使用游标抓取数据

```sql
DELIMITER $
CREATE PROCEDURE getAll() 
BEGIN
--定义三个接收变量
DECLARE e_id int(11);
DECLARE c_name varchar(25);
DECLARE t_name varchar(25);
--定义循环终止变量
DECLARE num int(11) default 0;
--定义游标
DECLARE result CURSOR FOR select * from city_country;
--循环变量赋值
select count(*) into num from city_country;
select concat('num=', num);
--开启游标
OPEN result;
--开启循环
repeat
--抓取游标
FETCH result into e_id,c_name, t_name;
--输出数据
select concat('id=', e_id, ', cityName=', c_name, ', countryName=', t_name);
--循环变量自减
set num = num - 1;
--循环终止条件
until num = 0
end repeat;
--关闭游标
CLOSE result;
END $
```

> 通过句柄机制

>DECLARE语句的次序 DECLARE语句发布存在特定的次序，用DECLARE语句定义的局部变量必须在定义任意游标或句柄之前定义，而句柄必须在游标之后定义

```sql
DELIMITER $
CREATE PROCEDURE getAll() 
BEGIN

DECLARE e_id int(11);
DECLARE c_name varchar(25);
DECLARE t_name varchar(25);
DECLARE has_data int(11) default 1;

DECLARE result CURSOR FOR select * from city_country;
--句柄事件
DECLARE EXIT HANDLER FOR NOT FOUND set has_data = 0;


OPEN result;
repeat
FETCH result into e_id,c_name, t_name;
select concat('id=', e_id, ', cityName=', c_name, ', countryName=', t_name);
select concat('e_id=' , e_id ,' e_name=',e_name,'e_age=',e_age,'e_sal=',e_sal);
until has_data= 0
end repeat;
CLOSE result;
END $
```

#### 总结

什么是游标

游标是系统为用户开设的一个数据缓冲区，存放SQL语句的执行结果，每个游标区都有一个名字。用户可以通过游标逐一获取记录并赋给主变量，交由主语言进一步处理。

### 存储函数

> 存储函数是一个有返回值的存储过程

```sql
CREATE FUNCTION function_name()
RETURNS type
START
....
END
```

实例

```sql
CREATE FUNCTION fun(countryId int)
RETURNS int 
BEGIN
DECLARE num int;
SELECT count(*) into num from city where country_id = countryId; 
return num;
END $

select fun(1)
```

## 触发器

### 介绍

触发器是与表有关的数据库对象，指在insert/update/delete之前或者之后，触发并自动执行的一条SQL语句。

> 触发器这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作

触发器只支持行级触发，不支持语句级触发

| 触发器类型     | NEW和OLD的使用                                     |
| -------------- | -------------------------------------------------- |
| INSERT型触发器 | NEW表示将要或者已经新增的数据                      |
| UPDATE型触发器 | OLD表示修改之前的数据，NEW表示将要或已经修改的数据 |
| DELETE型触发器 | OLD表示将要或者已经删除的数据                      |

### 语法

创建触发器

```sql
create trigger trigger_name
before/after insert/update/delete
on tbl_name
[ for each row ] -- 行级触发器
begin
trigger_stmt ;
end;
```

删除触发器

```sql
drop trigger trigger_name
```

查看触发器

```sql
show triggers;
```

### 实例

```sql
DELIMITER $
create trigger emp_logs_insert_trigger
after insert
on emp
for each row
begin
insert into emp_logs (id,operation,operate_time,operate_id,operate_params)values(null,'insert',now(),new.id,concat('插入后(id:',new.id,', name:',new.name,',age:',new.age,', salary:',new.salary,')'));
end $
DELIMITER ;

DELIMITER $
create trigger emp_logs_update_trigger
after update 
on emp
for each row
begin
insert into emp_logs (id,operation,operate_time,operate_id,operate_params)values(null,'update',now(),new.id,concat('修改前(id:',old.id,', name:',old.name,',age:',old.age,', salary:',old.salary,') , 修改后(id',new.id, 'name:',new.name,',age:',new.age,', salary:',new.salary,')'));
end $
DELIMITER ;

DELIMITER $
create trigger emp_logs_delete_trigger
after delete
on emp
for each row
begin
insert into emp_logs (id,operation,operate_time,operate_id,operate_params)values(null,'delete',now(),old.id,concat('删除前(id:',old.id,', name:',old.name,',age:',old.age,', salary:',old.salary,')'));
end $
DELIMITER ;
```

### 使用场景

- 可以通过数据库中的相关表实现级联更改
- 实时监控某张表中的某个字段的更改而需要作出相应的处理
- 可以生产某些业务编号
- 滥用的话，会造成数据库及应用程序的维护困难

### MySQL中都有哪些触发器？

- Before Insert
- After Insert
- Before Update
- After Update
- Before Delete
- After Delete

## 体系结构

![MySQL体系架构](https://gitee.com/zhang-songyao/blog-images/raw/master/20210428162313.jpg)

MySQL最上层是连接组件，下面服务器由连接池、管理工具和服务、SQL接口、解析器、优化器、缓存、存储引擎、文件系统组成。

- Connection Pool : 连接池组件
- Management Services & Utilities : 管理服务和工具组件
- SQL Interface : SQL接口组件
- Parser : 查询分析器组件
- Optimizer : 优化器组件
- Caches & Buffers : 缓冲池组件
- Pluggable Storage Engines : 存储引擎
- File System : 文件系统

> 连接层

最上层是一些客户端和链接服务，包含本地sock 通信和大多数基于客户端/服务端工具实现的类似于 TCP/IP的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

> 服务层

第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定表的查询的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

> 引擎层

存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。

> 存储层

数据存储层，主要是将数据存储在文件系统之上，并完成与存储引擎的交互。

>和其他数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎上，插件式的存储引擎架构，将查询处理和其他的系统任务以及数据的存储提取分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

## SQL优化

> 为什么需要SQL优化

在应用的的开发过程中，由于初期数据量小，开发人员写 SQL 语句时更重视功能上的实现，但是当应用系统正式上线后，随着生产数据量的急剧增长，很多 SQL 语句开始逐渐显露出性能问题，对生产的影响也越来越大，此时这些有问题的 SQL 语句就成为整个系统性能的瓶颈，因此我们必须要对它们进行优化。

### 查看SQL执行频率

show [session|global] status 可以根据需要加上参数“session”或者“global”来显示 session 级（当前连接）的计结果和global 级（自数据库上次启动至今）的统计结果。如果不写，默认使用参数是“session”。

```sql
show status like 'Com_______';
```

```sql
show status like 'Innodb_row_%'
```

Com_xxx 表示每个 xxx 语句执行的次数，我们通常比较关心的是以下几个统计参数。

![image-20210428211637194](https://gitee.com/zhang-songyao/blog-images/raw/master/20210428211638.png)

Com_*** : 这些参数对于所有存储引擎的表操作都会进行累计。

Innodb_*** : 这几个参数只是针对InnoDB 存储引擎的，累加的算法也略有不同。

### 定位低效率执行SQL

可以通过以下两种方式定位执行效率较低的 SQL 语句。

- 慢查询日志 : 通过慢查询日志定位那些执行效率较低的 SQL 语句，用--log-slow-queries[=fifile_name]选项启动时，mysqld 写一个包含所有执行时间超过long_query_time 秒的 SQL 语句的日志文件。
- show processlist : 慢查询日志在查询结束以后才纪录，所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用show processlist命令查看当前MySQL在进行的线程，包括线程的状态、是否锁表等，可以实时地查看 SQL 的执行情况，同时对一些锁表操作进行优化。

1） id列，用户登录mysql时，系统分配的"connection_id"，可以使用函数connection_id()查看 

2） user列，显示当前用户。如果不是root，这个命令就只显示用户权限范围的sql语句 

3） host列，显示这个语句是从哪个ip的哪个端口上发的，可以用来跟踪出现问题语句的用户 

4） db列，显示这个进程目前连接的是哪个数据库 

5） command列，显示当前连接的执行的命令，一般取值为休眠（sleep），查询（query），连接 （connect）等 

6） time列，显示这个状态持续的时间，单位是秒 

7） state列，显示使用当前连接的sql语句的状态，很重要的列。state描述的是语句执行中的某一个状态。一 个sql语句，以查询为例，可能需要经过copying to tmp table、sorting result、sending data等状态 才可以完成 

8） info列，显示这个sql语句，是判断问题语句的一个重要依据 

### explain分析执行计划

通过以上步骤查询到效率低的SQL语句之后，可以通过EXPLAIN或者DESC命令获取MySQL如何执行SELECT语句的信息，包括在SELECT语句执行过程中如何连接和连接顺序

```sql
EXPLAIN SELECT *
DESC SELECT *
```

![image-20210429174238429](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429174241.png)

|     字段      |                             含义                             |
| :-----------: | :----------------------------------------------------------: |
|      id       | select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。 |
|  select_type  | 表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（子查询中的第一个 SELECT）等 |
|     table     |                        输出结果集的表                        |
|     type      | 表示表的连接类型，性能由好到差的连接类型为( system ---> const -----> eq_ref ------> ref-------> ref_or_null----> index_merge ---> index_subquery -----> range -----> index ------>all ) |
| possible_keys |                  表示查询时，可能使用的索引                  |
|      key      |                      表示实际使用的索引                      |
|    key_len    |                        索引字段的长度                        |
|     rows      |                扫描行的数量，并不是一个准确值                |
|     extra     |                     执行情况的说明和描述                     |

#### id

> id字段是select查询的序列号，十一组数字，表示的是查询中执行select字句或者操作表的顺序

- id相同表示表的顺序是从上到下
- id不同，id值越大优先级越高，越先被执行
- id 有相同，也有不同，同时存在。id相同的可以认为是一组，从上往下顺序执行；在所有的组中，id的值越大，优先级越高，越先执行。

#### select_type

| **select_type** |              **description**              |
| :-------------: | :---------------------------------------: |
|     SIMPLE      |       不包含任何子查询或union等查询       |
|     PRIMARY     |   包含子查询最外层查询就显示为 PRIMARY    |
|    SUBQUERY     |     在select或 where字句中包含的查询      |
|     DERIVED     |           from字句中包含的查询            |
|      UNION      |         出现在union后的查询语句中         |
|  UNION RESULT   | 从UNION中获取结果集，例如上文的第三个例子 |

#### type

|      type       |                             含义                             |
| :-------------: | :----------------------------------------------------------: |
|      NULL       |            MySQL不访问任何表，索引，直接返回结果             |
|     system      | 表只有一行记录（等于系统表），这是const类型的特例，一般不会出现 |
|      const      | 通过索引一次就找到了。const用于比较primary key或者unique索引，因为只匹配一行数据，所以很快 |
|     eq_ref      | 在join查询中使用PRIMARY KEY or UNIQUE NOT NULL索引关联。关联的记录只有一条 |
|       ref       |     使用非唯一索引查找数据，返回所有匹配某个单独值的多行     |
|    fulltext     |                         使用全文索引                         |
|   ref_or_null   |                  对Null进行索引的优化的 ref                  |
|   index_merge   |                       合并查询使用索引                       |
| unique_subquery |                    在子查询中使用 eq_ref                     |
| index_subquery  |                      在子查询中使用ref                       |
|      range      | 只检索给定返回的行，使用一个索引来选择行，where之后出现between、<、>、in等操作 |
|      index      |                        遍历整个索引树                        |
|       ALL       |                     遍历全表以找到匹配行                     |

> 一般来说我们需要保证查询至少达到range级别，最好是ref

##### 执行效率

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL 

#### extra

|      extra      |                             含义                             |
| :-------------: | :----------------------------------------------------------: |
|   Using index   |                         使用覆盖索引                         |
|   Using where   |                使用了用where子句来过滤结果集                 |
| Using filesort  | 使用文件排序，使用非索引列进行排序时出现，非常消耗性能，尽量优化 |
| Using temporary | 使用了临时表保存了中间结果，MySQL在对查询结果排序使用了临时表，常见于order by 和group by。 sql优化的目标可以参考阿里开发手册 |

### show profile

Mysql从5.0.37版本开始增加了对 show profiles 和 show profile 语句的支持。show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。

通过 have_profiling 参数，能够看到当前MySQL是否支持profile： 

查询是否支持profile

```sql
select @@have_profiling;
```

查询是否开启profiling

```sql
select @@profiling;
```

开启profiling

```sql
set profiling = 1
```

执行完上述命令之后，再执行show profiles 指令， 来查看SQL语句执行的耗时：

```sql
show profiles;
```

通过show profile for query query_id 语句可以查看到该SQL执行过程中每个线程的状态和消耗的时间

```sql
show profile for query query_id
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210429180038.png" alt="image-20210429180037097" style="zoom: 67%;" />

![image-20210429180055845](https://gitee.com/zhang-songyao/blog-images/raw/master/20210429180056.png)

>TIP ：
>
>Sending data 状态表示MySQL线程开始访问数据行并把结果返回给客户端，而不仅仅是返回个客户端。由于 
>
>在Sending data状态下，MySQL线程往往需要做大量的磁盘读取操作，所以经常是整各查询中耗时最长的状态。

### 扩展问题

#### 如何定位及优化SQL语句的性能问题

对于低性能的SQL语句的定位，最重要也是最有效的方法就是使用执行计划，MySQL提供了explain命令来查看语句的执行计划，不管是哪种数据库，或者说是哪种数据库引擎，在对一条SQL语句进行执行的过程都会做很多相关的优化，对于查询语句，最重要的优化方式就是使用索引。而**执行计划，就是显示数据库引擎对于SQL语句的执行的详细情况，其中包含了是否使用索引，使用什么索引，使用的索引的相关信息等**。

### 大批量插入数据

对于InnoDB类型的表来说。有以下几种方式可以提高导入的效率

当使用load命令导入数据的时候，适当的设置可以提高导入效率

```sql
load data local infile 'filepath' into table table_name fields terminated by ',' lines terminated by '\n';
```

#### 主键顺序插入

因为InnoDB类型是按照主键顺序保存的，索引导入数据按照主键的顺序排序，可以有效的提高导入数据的效率。如果InnoDB表没有主键，那么系统会自动默认创建一个内部列作为主键，所以如果可以给表创建一个主键，将可以利用这点，来提高导入数据的效率。

#### 关闭唯一性校验

在导入数据前执行 SET UNIQUE_CHECKS=0，关闭唯一性校验，在导入结束后执行SET UNIQUE_CHECKS=1，恢复唯一性校验，可以提高导入的效率。

```sql
SET UNIQUE_CHECKS=0
SET UNIQUE_CHECKS=1
```

#### 手动提交数据

如果应用使用自动提交的方式，建议在导入前执行 SET AUTOCOMMIT=0，关闭自动提交，导入结束后再执行 SET AUTOCOMMIT=1，打开自动提交，也可以提高导入的效率。

```sql
 SET AUTOCOMMIT=0
 SET AUTOCOMMIT=1
```

### 优化insert语句

#### 使用多个值表的insert

如果需要同时对一张表插入很多行数据时，应该尽量使用多个值表的insert语句，这种方式将大大的缩减客户端与数据库之间的连接、关闭等消耗。使得效率比分开执行的单个insert语句快。

```sql
insert into tb_test values(1,'Tom'); 
insert into tb_test values(2,'Cat'); 
insert into tb_test values(3,'Jerry');
```

> 优化后

```sql
insert into tb_test values(1,'Tom'),(2,'Cat')，(3,'Jerry'); 
```

#### 在事务中进行数据的插入

```sql
start transaction;
insert....
insert....
commit
```

#### 数据有序插入

### 优化order by语句

执行原理：将查询结果按照某个字段排序时，如果该字段没有建立索引，那么执行计划会将查询出的所有数据使用外部排序（将数据从硬盘分批读取到内存使用内部排序，最后合并排序结果），这个操作是很影响性能的，因为需要将查询涉及到的所有数据从磁盘中读到内存（如果单条数据过大或者数据量过多都会降低效率），更无论读到内存之后的排序了。
如果我们对该字段建立索引alter table 表名 add index(字段名)，那么由于索引本身是有序的，因此直接按照索引的顺序和映射关系逐条取出数据即可。而且如果分页的，那么只用取出索引表某个范围内的索引对应的数据，而不用像上述那取出所有数据进行排序再返回某个范围内的数据。（从磁盘取数据是最影响性能的）

#### 环境准备

创建一张员工表

```sql
CREATE TABLE `emp` ( 
    `id` int(11) NOT NULL AUTO_INCREMENT, 
    `name` varchar(100) NOT NULL, 
    `age` int(3) NOT NULL, 
    `salary` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210430155454.png" alt="image-20210430155453080" style="zoom:67%;" />

#### 	创建name 和 salary的联合索引

```sql
create index idx_emp_name_sal on emp(name,salary);
```

#### 两种排序方式

##### filesort排序

> 通过对返回数据进行排序，不通过索引直接返回排序结果的排序

##### use index排序

> 通过有序索引顺序扫描直接排序，不需要额外排序，操作效率较高

单字段查询的时候，只有覆盖索引的时候才是use index

![image-20210430160754621](https://gitee.com/zhang-songyao/blog-images/raw/master/20210430160757.png)

多字段查询的时候，只有覆盖索引的时候才会使用索引

![image-20210430161136216](https://gitee.com/zhang-songyao/blog-images/raw/master/20210430161137.png)

了解了MySQL的排序方式，优化目标就清晰了，尽量减少额外的排序，通多索引直接返回有序数据，where 条件和Order by 使用相同的索引，并且Order By 的顺序和索引顺序相同， 并且Order by 的字段都是升序，或者都是降序。否则肯定需要额外的操作，这样就会出现FileSort。

##### filesort优化

通过创建合适的索引，能够减少 Filesort 的出现，但是在某些情况下，条件限制不能让Filesort消失，那就需要加快 Filesort的排序操作。对于Filesort ， MySQL 有两种排序算法：

1. 两次扫描算法：4.1版本之前，使用该排序方式，首先根据条件取出排序字段和指针信息，然后在排序区sort buffer中排序，如果sort buffer不够，则在临时表temporary table中存储结果。完成排序后，在根据行指针回表读取数据，该操作导致大量的IO操作
2. 一次扫描算法：一次性的取出满足条件的所有字段，然后在排序区sort buffer中排序后直接输出结果集，排序时内存开销较大，但是排序效率比两次扫描高

> 通过比较系统变量` max_length_for_sort_data `的大小和Query语句取出的字段总大小， 来判定是否那种排序算法，如果`max_length_for_sort_data` 更大，那么使用第二种优化之后的算法；否则使用第一种。

可以适当的提高`max_length_for_sort_data `和`sort_buffer_size`系统变量，来增大排序区的大小，提高排序的效率

```sql
show variables like 'max_length_for_sort_data'; --1k
show variables like 'sort_buffer_size';--256k
```

### 优化group by语句

由于GROUP BY 实际上也同样会进行排序操作，而且与ORDER BY 相比，GROUP BY 主要只是多了排序之后的分组操作。当然，如果在分组的时候还使用了其他的一些聚合函数，那么还需要一些聚合函数的计算。所以，在GROUP BY 的实现过程中，与 ORDER BY 一样也可以利用到索引。

#### 可以通过order by null 来禁止排序

首先需要删除需要分组字段的索引，要不然仍然会走索引

![image-20210430164025239](https://gitee.com/zhang-songyao/blog-images/raw/master/20210430164027.png)

从上面的例子可以看出，第一个SQL语句需要进行"filesort"，而第二个SQL由于order by null 不需要进行"filesort"， 而上文提过Filesort往往非常耗费时间。

### 优化嵌套查询

Mysql4.1版本之后，开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，有些情况下，子查询是可以被更高效的连接（JOIN）替代。

连接(Join)查询之所以更有效率一些 ，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上需要两个步骤的查询工作。

### 优化OR条件

建议使用union替换or

UNION 语句的 type 值为 ref，OR 语句的 type 值为 range，可以看到这是一个很明显的差距

UNION 语句的 ref 值为 const，OR 语句的 type 值为 null，const 表示是常量值引用，非常快

这两项的差距就说明了 UNION 要优于 OR 。 

![image-20210430165346359](https://gitee.com/zhang-songyao/blog-images/raw/master/20210430165347.png)

### 优化分页查询

一般分页查询时，通过创建覆盖索引能够比较好地提高性能。一个常见又非常头疼的问题就是 limit 2000000,10 ，此时需要MySQL排序前2000010 记录，仅仅返回2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大 

#### 优化思路1

先在索引上完成查询操作，最后根据主键关联回表查询所有需要的数据

```sql
select * from table_name t, (select id from table_name limit 10000000,10) a where a.id = t.id;
```

#### 优化思路二

该方法适用于主键自增的表，而且主键不能出现断层。

```sql
select * from table_name where id > 1000000 limit 10;
```

>核心思想都一样,就是减少load的数据.

解决超大分页,其实主要是靠缓存,可预测性的提前查到内容,缓存至redis等k-V数据库中,直接返回即可.

阿里巴巴Java开发手册

```
【推荐】利用延迟关联或者子查询优化超多分页场景。 

说明：MySQL并不是跳过offset行，而是取offset+N行，然后返回放弃前offset行，返回N行，那当offset特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行SQL改写。 

正例：先快速定位需要获取的id段，然后再关联： 

SELECT a.* FROM 表1 a, (select id from 表1 where 条件 LIMIT 100000,20 ) b where a.id=b.id
```

### 使用SQL提示

SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

#### USE INDEX

在查询语句中表名的后面，添加 use index 来提供希望MySQL去参考的索引列表，就可以让MySQL不再考虑其他可用的索引。

#### IGNORE INDEX

如果用户只是单纯的想让MySQL忽略一个或者多个索引，则可以使用 ignore index 作为 hint （暗示）

#### FORCE INDEX

为强制MySQL使用一个特定的索引，可在查询中使用 force index 作为hint 。 

## 应用优化

### 使用连接池

对于访问数据库来说，建立连接的代价是比较昂贵的，因为我们频繁的创建关闭连接，是比较耗费资源的，我们有必要建立 数据库连接池，以提高访问的性能。

### 减少对MySQL的访问

#### 减少对数据进行重复检索

在编写应用代码时，需要能够理清对数据库的访问逻辑。能够一次连接就获取到结果的，就不用两次连接，这样可以大大减少对数据库无用的重复请求。

#### 增加cache层

在应用中，我们可以在应用中增加缓存层来达到减轻数据库负担的目的。缓存层有很多种，也有很多实现方式，只要能达到降低数据库的负担又能满足应用需求就可以。

因此可以部分数据从数据库中抽取出来放到应用端以文本方式存储， 或者使用框架(Mybatis, Hibernate)提供的一级缓存/二级缓存，或者使用redis数据库来缓存数据 。

### **非关系数据库**

也可以考虑将非核心（重要）数据，存在 MongoDB 中，这样可以提高插入以及查询的效率。

### **全文检索**

如果业务系统中的数据量比较大（达到千万级别），这个时候，如果再对数据库进行查询，特别是进行分页查询，速度将变得很慢（因为在分页时首先需要count求合计数），为了提高访问效率，这个时候，可以考虑加入Solr 或 者 ElasticSearch全文检索服务，来提高访问效率。

### 负载均衡

#### 利用MySQL复制分流查询

负载均衡是应用中使用非常普遍的一种优化方法，它的机制就是利用某种均衡算法，将固定的负载量分布到不同的服务器上， 以此来降低单台服务器的负载，达到优化的效果。

![image-20210501101649810](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501101650.png)

#### 采用分布式数据库结构

分布式数据库架构适合大数据量、负载高的情况，它有良好的拓展性和高可用性。通过在多台服务器之间分布数据，可以实现在多台服务器之间的负载均衡，提高访问效率。

## 查询缓存优化

### SQL的生命周期

开启Mysql的查询缓存，当执行完全相同的SQL语句的时候，服务器就会直接从缓存中读取结果，当数据被修改，之前的缓存会失效，修改比较频繁的表不适合做查询缓存。

​	![image-20210501104108352](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501104109.png)

1. 客户端发送一条查询给服务器
2. 应用服务器和数据库服务器建立一个连接
3. 服务器先会检查查询缓存，如果命中了缓存，则立即返回存储在缓存中的结果。否则进入下一阶段
4. 服务器端进行SQL解析、预处理，再由优化器生成对应的执行计划
5. MySQL根据优化器生成的执行计划，调用存储引擎的API来执行查询
6. 读取数据到内存并进行逻辑处理
7. 通过步骤2的连接，将结果返回给客户端。
8. 关闭连接，释放资源

### 查询缓存配置

查看前MySQL数据库是否支持查询缓存

```sql
SHOW VARIABLES LIKE 'hava_query_cache';
```

查看当前MySQL是否开启了查询缓存

```sql
SHOW VARIABLES LIKe 'query_cache_type';
```

查看查询缓存占用大小

```sql
SHOW VARIABLES LIKE 'query_cache_size';
```

查看查询缓存的状态变量

```sql
SHOW STATUS LIKE 'Qcache%';
```

![image-20210501141902845](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501141903.png)

|          参数           |                             含义                             |
| :---------------------: | :----------------------------------------------------------: |
|   Qcache_free_blocks    |                   查询缓存中的可用内存块数                   |
|   Qcache_free_memory    |                     查询缓存的可用内存量                     |
|       Qcache_hits       |                        查询缓存命中数                        |
|     Qcache_inserts      |                    添加到查询缓存的查询数                    |
|  Qcache_lowmen_prunes   |            由于内存不足而从查询缓存中删除的查询数            |
|    Qcache_not_cached    | 非缓存查询的数量（由于 query_cache_type 设置而无法缓存或未缓存） |
| Qcache_queries_in_cache |                    查询缓存中注册的查询数                    |
|   Qcache_total_blocks   |                      查询缓存中的块总数                      |

### 开启查询缓存

MySQL的查询缓存默认是关闭的，需要手动配置参数 query_cache_type ， 来开启查询缓存，query_cache_type该参数的可取值有三个 

|    值    |                             含义                             |
| :------: | :----------------------------------------------------------: |
| OFF 或 0 |                       查询缓存功能关闭                       |
| ON 或 1  | 查询缓存功能打开，SELECT的结果符合缓存条件即会缓存，否则，不予缓存，显式指定SQL_NO_CACHE，不予缓存 |
|  DEMAND  | 查询缓存功能按需进行，显式指定 SQL_CACHE 的SELECT语句才会缓存；其它均不予缓存 |

在my.cnf文件中配置，配置后重启服务

```
query_cache_type=1
```

查询缓存SELECT选项

SQL_CACHE : 如果查询结果是可缓存的，并且 query_cache_type 系统变量的值为ON或 DEMAND ，则缓存查询结果 。

SQL_NO_CACHE : 服务器不使用查询缓存。它既不检查查询缓存，也不检查结果是否已缓存，也不缓存查询结果。

```sql
SELECT SQL_CACHE id, name FROM customer; SELECT SQL_NO_CACHE id, name FROM customer; 
```

### 查询缓存失效的情况

1. SQL语句不一致，要命中查询缓存，查询的SQL语句必须完全一致（包括大小写）
2. 当查询语句中有一些不确定时，不会缓存， now() , current_date() , curdate() , curtime() , rand() ,uuid() , user() , database() 。 
3. 不使用表的查询
4. 查询MySQL系统表的时候，不会缓存mysql， information_schema或 performance_schema 
5. 存储函数、触发器或事件的主体内执行的查询
6. 如果表更改则使用该表的所有高速缓存查询都将变为无效并从高速缓存中删除。这包括使用 MERGE 映射到已更改表的表的查询。一个表可以被许多类型的语句，如被改变 INSERT， UPDATE， DELETE， TRUNCATE，TABLE， ALTER TABLE， DROP TABLE，或 DROP DATABASE 。

## 内存管理优化

### 内存优化的原则

1. 将尽量多的内存分配给MySQL系统，但要给操作系统和其他程序预留足够空间
2. MyISAM 存储引擎的数据文件读取依赖于操作系统自身的IO缓存，因此，如果有MyISAM表，就要预留更多的内存给操作系统做IO缓存
3. 排序区、连接区等缓存是分配给每个数据库会话（session）专用的，其默认值的设置要根据最大连接数合理分配，如果设置太大，不但浪费资源，而且在并发连接较高时会导致物理内存耗尽。

## MySQL并发参数调整

从实现上来说，MySQL Server是多线程的结构，包括后台线程和客户服务线程，多线程可以效利用服务器资源，提高数据库的并发性能。在Mysql中，控制并发连接和线程的主要参数包括 max_connections、back_log、thread_cache_size、table_open_cahce。

###  **max_connections**

采用max_connections 控制允许连接到MySQL数据库的最大数量，默认值是 151。如果状态变量connection_errors_max_connections 不为零，并且一直增长，则说明不断有连接请求因数据库连接数已达到允许最大值而失败，这是可以考虑增大max_connections 的值。

Mysql 最大可支持的连接数，取决于很多因素，包括给定操作系统平台的线程库的质量、内存大小、每个连接的负荷、CPU的处理速度，期望的响应时间等。在Linux 平台下，性能好的服务器，支持 500-1000 个连接不是难事，需要根据服务器性能进行评估设定。

### **back_log**

back_log 参数控制MySQL监听TCP端口时设置的积压请求栈大小。如果MySql的连接数达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源，将会报错。5.6.6 版本之前默认值为 50 ， 之后的版本默认为 50 +（max_connections / 5）， 但最大不超过900。

如果需要数据库在较短的时间内处理大量连接请求， 可以考虑适当增大back_log 的值。

### **table_open_cache**

该参数用来控制所有SQL语句执行线程可打开表缓存的数量， 而在执行SQL语句时，每一个SQL执行线程至少要打开 1 个表缓存。该参数的值应该根据设置的最大连接数 max_connections 以及每个连接执行关联查询中涉及的表

的最大数量来设定 ：

max_connections x N 

### **thread_cache_size**

为了加快连接数据库的速度，MySQL 会缓存一定数量的客户服务线程以备重用，通过参数 thread_cache_size 可控制 MySQL 缓存客户服务线程的数量。

###  **innodb_lock_wait_timeout**

该参数是用来设置InnoDB 事务等待行锁的时间，默认值是50ms ， 可以根据需要进行动态设置。对于需要快速反馈的业务系统来说，可以将行锁的等待时间调小，以避免事务长时间挂起； 对于后台运行的批量处理程序来说，可以将行锁的等待时间调大， 以避免发生大的回滚操作。

## 锁

> 什么是锁

锁是计算机协调多个进程或线程并发访问某一资源的机制（避免争抢）。在数据库中，除传统的计算资源（如 CPU、RAM、I/O 等）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的

一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

### 锁分类

从对数据的的粒度分：

- 表锁：操作时，会锁定整个表
- 行锁：操作时，会锁定当前操作行

从对数据操作的类型分：

- 读锁（共享锁）：针对同一份数据，共享锁可以同时加上多个，多个读操作可以同时进行而不会互相影响
- 写锁（排它锁）：当前操作没有完之前，排他锁只可以加一个，它会阻断其他写锁和读锁，共享锁都相斥

相对其他数据库而言，MySQL的锁机制比较简单，其最显著的特点是不同的存储引擎支持不同的锁机制。下表中罗列出了各存储引擎对锁的支持情况：

| 存储引擎 | 表级锁 | 行级锁 | 页面锁 |
| -------- | ------ | ------ | ------ |
| MyISAM   | 支持   | 不支持 | 不支持 |
| InnoDB   | 支持   | 支持   | 不支持 |
| MEMORY   | 支持   | 不支持 | 不支持 |
| BDB      | 支持   | 不支持 | 支持   |

表级锁：偏向MyISAM存储引擎，**开销小，加锁快，不会出现死锁；锁的粒度大，发生冲突的概率最高，并发度最低**

行级锁：偏向InnoDB存储引擎，**开销大，加锁慢，会出现死锁；锁的粒度小，发生锁冲突的概率最低，并发度也高**

页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，表级锁速度快，但冲突多，行级冲突少，但速度慢。所以取了折衷的页级，一次锁定相邻的一组记录，并发度一般。

### MyISAM表锁

MyISAM在执行SELECT前，会自动给涉及的所有表加锁，在执行更新操作（DML）前，会自动给涉及的表加写锁，这个过程不需要用户干预，因此，用户一般不需要直接用Lock table 命令给MyISAM表加显示锁。

```sql
--加读锁 
lock table table_name read;
--加写锁
lock table table_name write;
```



- 对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求
- 对MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作

> MyISAM的读写锁调度是写优先，这也是MyISAM不适合做写为主的表的存储引擎的原因，因为写锁后，其他线程不能做任何操作，大量的更新会是查询很难得到锁，从而造成永远的阻塞。

### 隔离级别和锁的关系

- Read Uncommitted级别下，读取数据不需要加锁，这样就不会跟被修改的数据上的排他锁冲突
- Read Committed级别下，读操作需要加共享锁，但是在语句执行完后释放锁
- Repeatable Read级别下，读操作需要加共享锁，但是在事务提交之前并不释放共享锁，必须等待事务执行完毕（解决了不可重复读的）
- SERIALIZABLE 是限制性最强的隔离级别，因为该级别**锁定整个范围的键**，并一直持有锁，直到事务完成。

### InnoDB行锁

> 行锁的特点

偏向InnoDB存储引擎，开销大，加锁慢，会出现死锁；锁的粒度小，发生锁冲突的概率最低，并发度也高

MySQL默认的隔离级别是Repeatable read

```sql
show variavles like 'tx_isolation'
```

- 共享锁（S）：又称为读锁，简称S锁，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
- 排他锁（X）：又称为写锁，简称X锁，排他锁就是不能与其他锁并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据就行读取和修改。

> 对于DML语句，InnoDB会自动给涉及数据集加排他锁，对于普通的SELECT语句，InnoDB不会加任何锁

### 无索引行锁升级为表锁（行锁的实现）

如果不通过索引条件（包括索引失效）检索数据，那么InnoDB将对表中的所有记录加锁，实际效果跟表锁一样。

![image-20210501203451448](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501203452.png)

由于执行更新时 ， name字段本来为varchar类型， 我们是作为数组类型使用，存在类型转换，索引失效，最终行锁变为表锁 

### 间隙锁

https://www.jianshu.com/p/478bc84a7721

间隙锁是MySQL在**RR可重复读隔离级别下**用来修复幻读才引入的一种锁，间隙锁也只有在RR可重复读隔离级别下才会存在，如果是在RC读已提交隔离级别下，是没有间隙锁的存在的。

当我们用范围条件，而不是使用相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据进行加锁； 对于键值在条件范围内但并不存在的记录，叫做 "间隙（GAP）" ， InnoDB也会对这个 "间隙" 加锁，这种锁机制就是所谓的 间隙锁（Next-Key锁）

![image-20210501203550068](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501203552.png)

### InnoDB存储引擎的锁的算法

- Record Lock：单个行记录上的锁
- Gap Lock：间隙锁，锁定一个范围，不包括记录本身
- Next-key lock：record+gap ：锁定一个范围，包含记录本身

### 什么是死锁

死锁是指两个或者多个事务在同一个资源上相互占用，并请求对方的资源，从而导致恶行循环的现象。

常件解决死锁的方法：

- 如果不同程序会并发的存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁的机会
- 在同一个事务中，尽可能做到一次锁定所有需要的资源，减少死锁产生的概率
- 对非常容易产生死锁的业务部分，可以尝试使用升级锁定的颗粒度，通过表级锁来减少死锁产生的概率

> 如果业务处理不好可以用分布式事务锁或者使用乐观锁

### 乐观锁和悲观锁

数据库系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和一致性以及数据库的统一性。乐观并发控制（乐观锁）和悲观并发控制（悲观锁）是并发控制主要采用的技术手段。

- 悲观锁：假定冲突会发生，屏蔽一切可能违反数据完整性的操作。在查询完数据的时候就把事务锁起来，直到提交事务。实现方式：使用数据库中的锁机制
- 乐观锁：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。在修改数据的时候把事务锁起来，通过version的方式来进行锁定。实现方式：乐一般会使用版本号机制或CAS算法实现。

> 使用场景

**乐观锁适用于写比较少的情况下（多读场景）**，即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量

但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以**一般多写的场景下用悲观锁就比较合适。**

### 行锁争用情况

```sql
show status like 'innodb_row_lock%'
```

![image-20210501203716423](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501203717.png)

- Innodb_row_lock_current_waits: 当前正在等待锁定的数量 
- Innodb_row_lock_time: 从系统启动到现在锁定总时间长度 
- Innodb_row_lock_time_avg:每次等待所花平均时长 
- Innodb_row_lock_time_max:从系统启动到现在等待最长的一次所花的时间 
- Innodb_row_lock_waits: 系统启动后到现在总共等待的次数 

当等待的次数很高，而且每次等待的时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手制定优化计划。 

### 总结

InnoDB存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面带来了性能损耗可能会比表锁高一些，但是在整体并发处理能力方面要远远高于MyISAM的表锁，当系统并发量较高的时候，InnoDB的整体性能和MyISAM相比就会有显著的优势。

优化：

- 尽可能让所有的数据检索都通过索引来完成，避免无索引行锁升级为表锁
- 合理设计索引，尽量缩小锁的范围
- 尽可能减少索引条件，及索引范围，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度
- 尽可使用低级别事务隔离（但是需要业务层面满足需求）

## 常用SQL技巧

### SQL的执行顺序

编写顺序

```sql
SELECT DISTINCT 
	<select list> 
FROM
	<left_table> <join_type> 
JOIN<right_table> ON <join_condition>
WHERE
	<where_condition>
GROUP BY
	<group_by_list> 
HAVING
	<having_condition>
ORDER BY 
	<order_by_condition> 
LIMIT
	<limit_params>
```

执行顺序

```sql
FROM		<left_table> 
ON			<join_condition> 
<join_type> 	JOIN 	<right_table>
WHERE 		<where_condition> 
GROUP BY 	<group_by_list>
HAVING 		<having_condition> 
SELECT DISTINCT <select list> 
ORDER BY 	<order_by_condition> 
LIMIT 		<limit_params>
```

### 正则表达式

> 正则表达式（Regular Expression）是指一个用来描述或者匹配一系列符合某个句法规则的字符串的单个字符串。

| **符号** |           **含义**            |
| :------: | :---------------------------: |
|    ^     |    在字符串开始处进行匹配     |
|    $     |    在字符串开始处进行匹配     |
|    .     | 匹配任意单个字符, 包括换行符  |
|  [...]   |    匹配出括号内的任意字符     |
|  [^..]   |   匹配不出括号内的任意字符    |
|    a*    |  匹配零个或者多个a(包括空串)  |
|    a+    | 匹配一个或者多个a(不包括空串) |
|    a?    |       匹配零个或者一个a       |
|  a1\|a2  |          匹配a1或a2           |
|   a(m)   |           匹配m个a            |
|  a(m,)   |         至少匹配m个a          |
|  a(m,n)  |       匹配m个a 到 n个a        |
|  a(,n)   |          匹配0到n个a          |
|  (...)   |    将模式元素组成单一元素     |

```sql
select * from emp where name regexp '^T'; 
select * from emp where name regexp '2$'; 
select * from emp where name regexp '[uvw]'; 
```

### 常用函数

#### 数字函数

![image-20210501211215084](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501211215.png)

#### 字符串函数

![image-20210501211233610](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501211234.png)

日期函数

![image-20210501211301439](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501211302.png)

#### 聚合函数

![image-20210501211323794](https://gitee.com/zhang-songyao/blog-images/raw/master/20210501211324.png)

## MySQL常用工具

### mysql

```sql
mysql [options] [database]
```

#### 连接选项

参数 ：

- -u, --user=name 指定用户名 

- -p, --password[=name] 指定密码 
- -h, --host=name 指定服务器IP或域名 
- -P, --port=# 指定连接端口 

示例 ：

- mysql -h 127.0.0.1 -P 3306 -u root -p 

- mysql -h127.0.0.1 -P3306 -uroot -p2143 


#### 执行选项

-e, --execute=name 执行SQL语句并退出 

此选项可以在Mysql客户端执行SQL语句，而不用连接到MySQL数据库再执行，对于一些批处理脚本，这种方式尤其方便。

```sql
mysql -uroot -p*** db -e 'select ....';
```

### mysqladmin

mysqladmin是一个执行管理操作的客户端程序，可以用它来检查服务器的配置和当前状态、创建并删除数据库等

通过mysqladmin --help查看帮助文档

示例

```
mysqladmin -uroot -p2143 create 'test01'; mysqladmin -uroot -p2143 drop 'test01'; mysqladmin -uroot -p2143 version;
```

### mysqlbinlog

由于服务器生成的是二进制日志文件以二进制格式保存，所以如果想要查询这些文本格式，就会使用到mysqlbinlog

```sql
mysqlbinlog [options] log-files1 log-files2 ... 
选项：
-d, --database=name : 指定数据库名称，只列出指定的数据库相关操作。 
-o, --offset=# : 忽略掉日志中的前n行命令。
-r,--result-file=name : 将输出的文本格式日志输出到指定文件。 
-s, --short-form : 显示简单格式， 省略掉一些信息。 
--start-datatime=date1 --stop-datetime=date2 : 指定日期间隔内的所有日志。 
--start-position=pos1 --stop-position=pos2 : 指定位置间隔内的所有日志。
```

### mysqldump

mysqldump 客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的SQL语句。

```sql
mysqldump [options] db_name [tables] 
mysqldump [options] --database/-B db1 [db2 db3...] 
mysqldump [options] --all-databases/-A 
```

#### 输出内容选项

```sql
参数：
--add-drop-database 在每个数据库创建语句前加上 Drop database 语句 
--add-drop-table 在每个表创建语句前加上 Drop table 语句 , 默认开启 ; 不开启 (-- skip-add-drop-table) 
-n, --no-create-db 不包含数据库的创建语句 
-t, --no-create-info 不包含数据表的创建语句 
-d --no-data 不包含数据 
-T, --tab=name 自动生成两个文件：一个.sql文件，创建表结构的语句； 一个.txt文件，数据文件，相当于select into outfile
```

示例

```sql
mysqldump -uroot -p db table_name --add-drop-database --add-drop-table > 外部文件目录
mysqldump -uroot -p2143 -T /tmp test city 
```

### mysqlimport/source

mysqlimport 是客户端数据导入工具，用来导入mysqldump 加 -T 参数后导出的文本文件。

```
mysqlimport [options] db_name textfile1 [textfile2...]
```

导入sql文件

```
source file/*.sql
```

### mysqlshow

mysqlshow 客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。

```
mysqlshow [options] [db_name [table_name [col_name]]]
```

参数

```
--count 显示数据库及表的统计信息（数据库，表 均可以不指定） 
-i 显示指定数据库或者指定表的状态信息 
```

![image-20210502165429868](https://gitee.com/zhang-songyao/blog-images/raw/master/20210502165431.png)

```
#查询每个数据库的表的数量及表中记录的数量 
mysqlshow -uroot -p2143 --count 
#查询test库中每个表中的字段书，及行数 
mysqlshow -uroot -p2143 test --count 
#查询test库中book表的详细情况 
mysqlshow -uroot -p2143 test book --count
```

## MySQL日志

在任何一种数据库中，都会有各种各样的日志，记录着数据库工作的方方面面，以帮助数据库管理员追踪数据库曾经发生过的各种事件。MySQL 也不例外，在 MySQL 中，有 4 种不同的日志，分别是错误日志、二进制日志（BINLOG 日志）、查询日志和慢查询日志，这些日志记录着数据库在不同方面的踪迹。

### 错误日志

> 错误日志是MySQL中最重要的日志之一，他记录了当MySQL启动和停止时，以及服务器运行过程中发生任何严重错误时的相关信息。当数据库出现任何故障无法使用的时候，可以首先查看此日志

该日志是默认开启的 ， 默认存放目录为 mysql 的数据目录（var/lib/mysql）, 默认的日志文件名为hostname.err（hostname是主机名）。

```sql
#配置开启binlog日志， 日志的文件前缀为 mysqlbin -----> 生成的文件名如 : mysqlbin.000001,mysqlbin.000002 
log_bin=mysqlbin 
#配置二进制日志的格式 
binlog_format=STATEMENT
```

查看日志位置指令

```sql
show variables like 'log_error%';
```

```sql
tail -f /文件目录
```

### 二进制日志

> 二进制日志（BINLOG）记录了**所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句**，但是不包括数据查询语句。此日志对于灾难时的数据恢复起着极其重要的作用，MySQL的主从复制， 就是通过该binlog实现的。

二进制日志，默认情况下是没有开启的，需要到MySQL的配置文件中去开启，并配置MySQL日志的格式。

日志存放位置 : 配置时，给定了文件名但是没有指定路径，日志默认写入Mysql的数据目录。

```sql
#配置开启binlog日志， 日志的文件前缀为 mysqlbin -----> 生成的文件名如 :
mysqlbin.000001,mysqlbin.000002 log_bin=mysqlbin
#配置二进制日志的格式 
binlog_format=STATEMENT
```

#### 日志格式

- **STATEMENT**：该日志格式在日志文件中记录的都是SQL语句（statement），每一条对数据进行修改的SQL都会记录在日志文件中，通过Mysql提供的mysqlbinlog工具，可以清晰的查看到每条语句的文本。主从复制的时候，从库（slave）会将日志解析为原文本，并在从库重新执行一次。
- **ROW**：该日志格式在日志文件中记录的是每一行的数据变更，而不是记录SQL语句。比如，执行SQL语句 ： update tb_book set status='1' , 如果是STATEMENT 日志格式，在日志中会记录一行SQL文件； 如果是ROW，由于是对全表进行更新，也就是每一行记录都会发生变更，ROW 格式的日志中会记录每一行的数据变更。
- **MIXED**：这是目前MySQL默认的日志格式，即混合了STATEMENT 和 ROW两种格式。默认情况下采用STATEMENT，但是在一些特殊情况下采用ROW来进行记录。MIXED 格式能尽量利用两种模式的优点，而避开他们的缺点。

#### 日志读取

```sql
mysqlbinlog log-file
```

#### 日志删除

对于比较繁忙的系统，由于每天生成日志量大 ，这些日志如果长时间不清楚，将会占用大量的磁盘空间。

##### 方式一

```sql
Reset Master
```

通过 Reset Master 指令删除全部 binlog 日志，删除之后，日志编号，将从 xxxx.000001重新开始 。

##### 方式二

执行指令 purge master logs to 'mysqlbin.*' ，该命令将删除 * 编号之前的所有日志。

##### 方式三

执行指令 purge master logs before 'yyyy-mm-dd hh24:mi:ss' ，该命令将删除日志为 "yyyy-mm-dd hh24:mi:ss" 之前产生的所有日志 。

##### 方式四

设置参数 --expire_logs_days=# ，此参数的含义是设置日志的过期天数， 过了指定的天数后日志将会被自动删除，这样将有利于减少DBA 管理日志的工作量。

### 查询日志		

查询日志中记录了客户端的所有操作语句，而二进制日志不包含查询数据的SQL语句。

默认情况下， 查询日志是未开启的。如果需要开启查询日志，可以设置以下配置 ： 

```
#该选项用来开启查询日志 ， 可选值 ： 0 或者 1 ； 0 代表关闭， 1 代表开启 
general_log=1 

#设置日志的文件名 ， 如果没有指定， 默认的文件名为 host_name.log general_log_file=file_name 
```

### 慢查询日志

慢查询日志记录了所有执行时间超过参数 long_query_time 设置值并且扫描记录数不小于**min_examined_row_limit** 的所有的SQL语句的日志。**long_query_time** 默认为 10 秒，最小为 0， 精度可以到微秒。

```
# 该参数用来控制慢查询日志是否开启， 可取值： 1 和 0 ， 1 代表开启， 0 代表关闭 
slow_query_log=1 

# 该参数用来指定慢查询日志的文件名
slow_query_log_file=slow_query.log 

# 该选项用来配置查询的时间限制， 超过这个时间将认为值慢查询， 将需要进行日志记录， 默认10s 
long_query_time=10 
```

开启慢查询

```sql
set global slow_query_log = ON;
```

查询long_query_time的值

```sql
show variables like 'log%';
```

直接通过cat 指令查询该日志文件 ：

![image-20210502162219348](https://gitee.com/zhang-songyao/blog-images/raw/master/20210502162221.png)

如果慢查询日志内容很多， 直接查看文件，比较麻烦， 这个时候可以借助于mysql自带的 mysqldumpslow 工具， 来对慢查询日志进行分类汇总。

![image-20210502162245673](https://gitee.com/zhang-songyao/blog-images/raw/master/20210502162246.png)

## MySQL复制

> 复制概述

复制是指将主数据库的DDL和DML操作通过二进制文件日志传到从库服务器，然后从库对这些日志重新执行（也叫重做），从而使得从库和主库的数据保持一致。

MySQL支持一台主库同时向多台从库进行复制， 从库同时也可以作为其他从服务器的主库，实现链状复制。

> 复制原理

![image-20210504151343689](https://gitee.com/zhang-songyao/blog-images/raw/master/20210504151344.png)				

分为三步：

- Master主库在事务提交时，会把数据变更作为时间Events记录在二进制文件Binlog中
- 主库推送二进制日志文件Binlog中的日志事件到从库的中继日志（Relay Log）
- slave重做中继日志中的事件，将改变反映它自己的数据

主：binlog线程——记录下所有改变了数据库数据的语句，放进master上的binlog中；

从：io线程——在使用start slave 之后，负责从master上拉取 binlog 内容，放进自己的relay log中；

从：sql执行线程——执行relay log中的语句；


> 复制的优势

MySQL 复制的有点主要包含以下三个方面：

- 主库出现问题，可以快速切换到从库提供服务。
- 可以在从库上执行查询操作，从主库中更新，实现读写分离，降低主库的访问压力。
- 可以在从库中执行备份，以避免备份期间影响主库的服务。

> MySQL主从复制解决的问题

- 数据分布：随意开始或停止复制，并在不同地理位置分布数据备份
- 负载均衡：降低单个服务器的压力
- 高可用和故障切换：帮助应用程序避免单点失败
- 升级测试：可以用高版本的MySQL作为从库

> MySQL主从复制的工作原理

- 在主库上将数据更改记录到二进制文件
- 从库将主库日志复制到自己的中继日志
- 从库读取中继日志的事件，将其重新放回从库数据中

### 搭建步骤

#### master

在master的配置文件（my.cnf）中配置如下

```
#mysql 服务ID,保证整个集群环境中唯一 
server-id=1 

#mysql binlog 日志的存储路径和文件名 
log-bin=/var/lib/mysql/mysqlbin 

#错误日志,默认已经开启 
#log-err 

#mysql的安装目录
#basedir 

#mysql的临时目录 
#tmpdir 

#mysql的数据存放目录
#datadir 

#是否只读,1 代表只读, 0 代表读写
read-only=0 

#忽略的数据, 指不需要同步的数据库 
binlog-ignore-db=mysql 

#指定同步的数据库 
#binlog-do-db=db01
```

配置完毕，重启MySQL服务器

```
sevice mysql restart
docker restart mysql
```

创建同步数据的账户，并进行授权操作

```
grant replication slave on *.* to 'itcast'@'192.168.192.131' identified by 'itcast'; 
flush privileges; 123
```

查看master状态

```
show master status
```

![image-20210503162729147](https://gitee.com/zhang-songyao/blog-images/raw/master/20210503162730.png)

File : 从哪个日志文件开始推送日志文件 

Position ： 从哪个位置开始推送日志 

Binlog_Ignore_DB : 指定不需要同步的数据库

#### slave

在slave配置文件中配置

```
#mysql服务端ID,唯一 
server-id=2 

#指定binlog日志 
log-bin=/var/lib/mysql/mysqlbin
```

配置完毕，重启服务器

执行如下指令

```
change master to master_host= '192.168.192.130', master_user='itcast', master_password='itcast', 
master_log_file='mysqlbin.000001', master_log_pos=413;
```

指定当前主库对应的主库ip地址，用户名，密码，从哪个日志文件开始的那个位置开始同步推送日志。

开启同步操作

```
start slave;
show slave status;
```

![image-20210503163226368](https://gitee.com/zhang-songyao/blog-images/raw/master/20210503163227.png)

停止同步操作

```
stop slave;
```

#### 验证同步操作

在主库中创建数据库，创建表，并插入数据，可以在从库中看到。

### 读写分离有哪些解决方案

读写分离是依赖于主从复制，而主从复制又是为读写分离服务的。因为主从复制要求`slave`不能写只能读（如果对`slave`执行写操作，那么`show slave status`将会呈现`Slave_SQL_Running=NO`，此时你需要按照前面提到的手动同步一下`slave`）。

#### 方案一

使用mysql-proxy代理

优点：直接实现读写分离和负载均衡，不用修改代码，master和slave用一样的帐号，mysql官方不建议实际生产中使用

缺点：降低性能， 不支持事务

#### 方案二

使用AbstractRoutingDataSource+aop+annotation在dao层决定数据库。

如果采用了mybatis， 可以将读写分离放在ORM层，比如mybatis可以通过mybatis plugin拦截sql语句，所有的insert/update/delete都访问master库，所有的select 都访问salve库，这样对于dao层都是透明。 plugin实现时可以通过注解或者分析语句是读写方法来选定主从库。不过这样依然有一个问题， 也就是不支持事务， 所以我们还需要重写一下DataSourceTransactionManager， 将read-only的事务扔进读库， 其余的有读有写的扔进写库。

#### 方案三

使用AbstractRoutingDataSource+aop+annotation在service层决定数据源，可以支持事务.

缺点：类内部方法通过this.xx()方式相互调用时，aop不会进行拦截，需进行特殊处理。

## 综合案例

基于方案三的主从分离

[gitee](https://gitee.com/zhang-songyao/MySQL_Cluster)

[github](https://github.com/dunk-code/MySQL_Cluster)

执行原理

![image-20210504164111410](https://gitee.com/zhang-songyao/blog-images/raw/master/20210504164112.png)

# [MySQL面试题](https://thinkwon.blog.csdn.net/article/details/104778621)

> [https://blog.csdn.net/qq_40884473/article/details/105213408?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162814757816780265461914%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162814757816780265461914&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-105213408.first_rank_v2_pc_rank_v29&utm_term=mysql%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0acid&spm=1018.2226.3001.4187](https://blog.csdn.net/qq_40884473/article/details/105213408?ops_request_misc=%7B%22request%5Fid%22%3A%22162814757816780265461914%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=162814757816780265461914&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-105213408.first_rank_v2_pc_rank_v29&utm_term=mysql怎么实现acid&spm=1018.2226.3001.4187)

原子性：是通过undolog来实现的

持久性：通过redolog来实现的

隔离性：是通过读写锁 + MVCC 来实现的

一致性：是通过原子性、持久性、隔离性来实现的

## 原子性的实现：

回滚就是当发生错误异常或者现实的执行rollback语句的时候，需要把数据还原到原先的模样。所以这时候也就需要用到undo log进行回滚

- 每条数据变更操作都会伴随一条undo log的生成，并且回滚日志必须先于数据持久化到磁盘
- 所谓的回滚就是根据回滚日志做逆向操作，比如 delete 的逆向操作为 insert ，insert的逆向操作为delete，update的逆向为update等

> undo log记录了数据被修改前的信息以及新增和被删除的数据信息，根据undo log生成回滚语句，

## 持久性的实现

MySQL的数据存储机制，MySQL的表数据是存放在磁盘上得，因此想要存取的时都要经历磁盘IO，然而即使是使用SSD磁盘IO也是非常消耗性能的。为此，为了提升性能InnoDB提供了缓冲池（Buffer Pool），Buffer Pool中包含了磁盘数据页的映射，可以当做缓存来使用

读数据：会首先从缓冲池中读取，如果缓冲池中没有，则从磁盘读取在放入缓冲池；

写数据：会首先写入缓冲池，缓冲池中的数据会定期同步到磁盘中；

上面这种缓冲池的措施虽然在性能方面带来了质的飞跃，但是它也带来了新的问题，当MySQL系统宕机，断电的时候可能会丢数据！！！

因为我们的数据已经提交了，但此时是在缓冲池里头，还没来得及在磁盘持久化，所以我们急需一种机制需要存一下已提交事务的数据，为恢复数据使用。

于是 redo log就派上用场了。

既然redo log也需要存储，也涉及磁盘IO为啥还用它？

（1）redo log 的存储是顺序存储，而缓存同步是随机操作。

（2）缓存同步是以数据页为单位的，每次传输的数据大小大于redo log。


## 隔离性的实现

隔离性是事务ACID特性里最复杂的一个。在SQL标准里定义了四种隔离级别，每一种级别都规定一个事务中的修改，哪些是事务之间可见的，哪些是不可见的。

级别越低的隔离级别可以执行越高的并发，但同时实现复杂度以及开销也越大。

Mysql 隔离级别有以下四种（级别由低到高）：

READUNCOMMITED(未提交读)
READCOMMITED(提交读)
REPEATABLEREAD(可重复读)
SERIALIZABLE (可重复读)
只要彻底理解了隔离级别以及他的实现原理就相当于理解了ACID里的隔离型。前面说过原子性，隔离性，持久性的目的都是为了要做到一致性，但隔离型跟其他两个有所区别，原子性和持久性是为了要实现数据的可性保障靠，比如要做到宕机后的恢复，以及错误后的回滚。

那么隔离性是要做到什么呢？ 隔离性是要管理多个并发读写请求的访问顺序。 这种顺序包括串行或者是并行

说明一点，写请求不仅仅是指insert操作，又包括update操作。

总之，从隔离性的实现可以看出这是一场数据的可靠性与性能之间的权衡。

## 一致性的实现

数据库总是从一个一致性的状态转移到另一个一致性的状态。

举个例：小盟从银行卡转 400 到 基金账户

‍1.假如执行完 update bank set balance = balance - 400;之发生异常了，银行卡的钱也不能平白无辜的减少，而是回滚到最初状态。

2.又或者事务提交之后，缓冲池还没同步到磁盘的时候宕机了，这也是不能接受的，应该在重启的时候恢复并持久化。

3.假如有并发事务请求的时候也应该做好事务之间的可见性问题，避免造成脏读，不可重复读，幻读等。在涉及并发的情况下往往在性能和一致性之间做平衡，做一定的取舍，所以隔离性也是对一致性的一种破坏。

总结

实现事务采取了哪些技术以及思想？

★ 原子性：使用 undo log ，从而达到回滚

★ 持久性：使用 redo log，从而达到故障后恢复

★ 隔离性：使用锁以及MVCC,运用的优化思想有读写分离，读读并行，读写并行

★ 一致性：通过回滚，以及恢复，和在并发环境下的隔离做到一致性。

## 大表数据查询，怎么优化

1. 优化方案、sql语句+索引
2. 加缓存，记忆快取（memcached），redis
3. 主从复制，读写分离
4. 垂直拆分，根据模块的耦合度，将一个大的系统分为多个小的系统，也就是分布式系统
5. 水平分割针对数据量大的表，这一步最麻烦，最能考验技术水平，要选择一个合理的sharding key, 为了有好的查询效率，表结构也要改动，做一定的冗余，应用也要改，sql中尽量带sharding key，将数据定位到限定的表上去查，而不是扫描全部的表

## 关心过业务系统里面的sql耗时吗？统计过慢查询吗？对慢查询都怎么优化过？

在业务系统中，除了使用主键进行的查询，其他的我都会在测试库上测试其耗时，慢查询的统计主要由运维在做，会定期将业务中的慢查询反馈给我们。

慢查询的优化首先要搞明白慢的原因是什么？ 是查询条件没有命中索引？是load了不需要的数据列？还是数据量太大？

所以优化也是针对这三个方向来的，

- 首先分析语句，看看是否load了额外的数据，可能是查询了多余的行并且抛弃掉了，可能是加载了许多结果中并不需要的列，对语句进行分析以及重写。
- 分析语句的执行计划，然后获得其使用索引的情况，之后修改语句或者修改索引，使得语句可以尽可能的命中索引。
- 如果对语句的优化已经无法进行，可以考虑表中的数据量是否太大，如果是的话可以进行横向或者纵向的分表。

## 主键使用自增ID还是UUID？

推荐使用自增ID，不要使用UUID。

因为在InnoDB存储引擎中，主键索引是作为聚簇索引存在的，也就是说，主键索引的B+树叶子节点上存储了主键索引以及全部的数据(按照顺序)，如果主键索引是自增ID，那么只需要不断向后排列即可，**如果是UUID，由于到来的ID与原来的大小不确定，会造成非常多的数据插入，数据移动，然后导致产生很多的内存碎片，进而造成插入性能的下降。**

总之，在数据量大一些的情况下，用自增主键性能会好一些。

关于主键是聚簇索引，如果没有主键，InnoDB会选择一个唯一键来作为聚簇索引，如果没有唯一键，会生成一个隐式的主键。

## 字段为什么要求定义为not null？

null的值会占用更多的字节，并且会在程序中造成很多与预期不符的情况。

## 如果要存储用户的密码散列，应该使用什么字段进行存储？

密码散列，盐，用户身份证号等固定长度的字符串应该使用char而不是varchar来存储，这样可以节省空间且提高检索效率。

## 数据表损坏的修复方式有哪些？

使用 myisamchk 来修复，具体步骤：

1）修复前将mysql服务停止。
2）打开命令行方式，然后进入到mysql的/bin目录。
3）执行myisamchk –recover 数据库所在路径/*.MYI

使用repair table 或者 OPTIMIZE table命令来修复，REPAIR TABLE table_name 修复表 OPTIMIZE TABLE table_name 优化表 REPAIR TABLE 用于修复被破坏的表。 OPTIMIZE TABLE 用于回收闲置的数据库空间，当表上的数据行被删除时，所占据的磁盘空间并没有立即被回收，使用了OPTIMIZE TABLE命令后这些空间将被回收，并且对磁盘上的数据行进行重排（注意：是磁盘上，而非数据库）

## 大表怎么优化

> 某个表有近千万数据，CRUD比较慢，如何优化？分库分表了是怎么做的？分表分库了有什么问题？有用到中间件么？他们的原理知道么？

当MySQL单表记录数过大时，数据库的CRUD性能会明显下降，优化措施：

- **限定数据的范围**：务必禁止不带任何限制数据范围条件的查询语句。比如：我们当用户在查询订单历史的时候，我们可以控制在一个月的范围内
- **读写分离**：经典的数据库拆分方案，主库负责写，从库负责读
- **缓存**：使用MySQL的缓存，另外对重量级、更新少的数据可以考虑使用应用级别的缓存
- 分库分表的方式进行优化

### 垂直分区

根据数据库里面而的数据表的相关性进行拆分，例如，用户表中既有用户的登录信息又有用户的基本信息，可以将用户表拆分成两个单独的表，甚至放到单独的库做分库。

简单来说，垂直拆分是指数据表列的拆分，把一张类列比较多的表拆分为多张表。

![image-20210504154022883](https://gitee.com/zhang-songyao/blog-images/raw/master/20210504154023.png)

> 优点

可以使得行数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简化表的结构，易于维护。

> 缺点

主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；

垂直分表

![image-20210504154112086](https://gitee.com/zhang-songyao/blog-images/raw/master/20210504154116.png)

> 适用场景

- 如果一个表中某些列常用，另外一些列不常用
- 可以使数据行变小，一个数据页能存储更多数据，查询时减少I/O次数

> 缺点

- 有些分表的策略基于应用层的逻辑算法，一旦逻辑算法改变，整个分表逻辑都会改变，扩展性较差
- 对于应用层来说，逻辑算法增加开发成本
- 管理冗余列，查询所有数据需要join操作

### 水平分区

保持数据表结构不变，通过某种策略存储数据分片。这样每一片数据分散到不同的表或者库中，达到了分布式的目的。 水平拆分可以支撑非常大的数据量。

水平拆分是指数据表行的拆分，表的行数超过200万行时，就会变慢，这时可以把一张的表的数据拆成多张表来存放。举个例子：我们可以将用户信息表拆分成多个用户信息表，这样就可以避免单一表数据量过大对性能造成影响。


![image-20210504154251960](https://gitee.com/zhang-songyao/blog-images/raw/master/20210504154252.png)

需要注意的一点是:分表仅仅是解决了单一表数据过大的问题，但由于表的数据还是在同一台机器上，其实对于提升MySQL并发能力没有什么意义，所以 **水平拆分最好分库** 。

水平拆分能够 **支持非常大的数据量存储，应用端改造也少**，但 **分片事务难以解决** ，跨界点Join性能较差，逻辑复杂。

《Java工程师修炼之道》的作者推荐 **尽量不要对数据进行分片，因为拆分会带来逻辑、部署、运维的各种复杂度** ，一般的数据表在优化得当的情况下支撑千万以下的数据量是没有太大问题的。如果实在要分片，尽量选择客户端分片架构，这样可以减少一次和中间件的网络I/O。

水平分表

![image-20210504154348484](https://gitee.com/zhang-songyao/blog-images/raw/master/20210504154349.png)

> 适用场景

- 表中的数据本身就有独立性，例如表中分表记录各个地区的数据或者不同时期的数据，特别是有些数据常用，有些不常用。
- 需要把数据存放在多个介质上。

> 缺点

- 给应用增加复杂度，通常查询时需要多个表名，查询所有数据都需UNION操作`（同步es，查询）`
- 在许多数据库应用中，这种复杂度会超过它带来的优点，查询时会增加读一个索引层的磁盘次数

**数据库分片的两种常见方案：**

- 客户端代理： 分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。 当当网的 Sharding-JDBC 、阿里的TDDL是两种比较常用的实现。
- 中间件代理： 在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中。 我们现在谈的 Mycat 、360的Atlas、网易的DDB等等都是这种架构的实现。

## 连接相关

数据库里面，长连接是指连接成功后，如果客户端持续有请求，则一直使用同一个连接。短连接则是指每次执行完很少的几次查询就断开连接，下次查询再重新建立一个。

建立连接的过程通常是比较复杂的，所以我建议你在使用中要尽量减少建立连接的动作，也就是尽量使用长连接。

但是全部使用长连接后，你可能会发现，有些时候 MySQL 占用内存涨得特别快，这是因为 MySQL 在执行过程中临时使用的内存是管理在连接对象里面的。这些资源会在连接断开的时候才释放。所以如果长连接累积下来，可能导致内存占用太大，被系统强行杀掉（OOM），从现象看就是 MySQL 异常重启了。

> 怎么解决这个问题呢？

你可以考虑以下两种方案。

- 定期断开长连接。使用一段时间，或者程序里面判断执行过一个占用内存的大查询后，断开连接，之后要查询再重连。
- 如果你用的是 MySQL 5.7 或更新版本，可以在每次执行一个比较大的操作后，通过执行 mysql_reset_connection 来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。

### 归档

- log data
- history data

https://maxwell.gitbook.io/way-to-architect/shu-ju-ku/mysql/mysqlzhong-de-sharding/fen-ku-fen-biao-hou-de-ping-hua-kuo-rong





# MySQL监控指标

## Table_open_cache

"Table_open_cache"是MySQL中一个系统变量，它指定了MySQL服务器缓存打开表的数量。每个打开的表都需要占用内存，因此，将Table_open_cache设置得太小可能会导致MySQL服务器不得不频繁地关闭和打开表，从而降低MySQL的性能。

Table Definition Cache（表定义缓存）是用来缓存数据表定义的一种缓存机制。当MySQL需要执行一个查询语句时，它首先需要检查查询中使用的表是否在Table Definition Cache中已经存在，如果存在，则可以直接使用缓存中的表定义，否则就需要从磁盘上读取表定义，这会导致一定的IO开销。

在MySQL的性能监控中，我们可以使用以下状态变量来监控Table Definition Cache的使用情况：

1. Table_open_cache_hits：表示从Table Definition Cache中找到表定义的次数。
2. Table_open_cache_misses：表示从磁盘上读取表定义的次数。
3. Table_open_cache_overflows：表示由于Table Definition Cache已经满了而被强制移除的表定义的数量。

如果Table_open_cache_hits的值较低，同时Table_open_cache_misses的值较高，说明Table Definition Cache的大小可能需要适当调整。如果Table_open_cache_overflows的值较高，说明Table Definition Cache的大小可能设置得过小，需要增加其大小来减少缓存失效率。因此，Table Definition Cache相关的状态变量可以帮助我们诊断和优化MySQL的性能问题。

## Network Traffic

MySQL的Network Traffic（网络流量）是指MySQL服务器与客户端之间在一定时间内传输的数据量，它是MySQL性能监控中的一个重要指标。

MySQL的Network Traffic涉及到两个方面：

1. 接收数据量：MySQL服务器从客户端接收的数据量，包括客户端发送的SQL语句和其他命令等。
2. 发送数据量：MySQL服务器向客户端发送的数据量，包括查询结果、错误信息、警告信息等。

在MySQL性能监控中，我们可以通过以下状态变量来监控MySQL的Network Traffic：

1. Bytes_received：表示MySQL服务器接收到的字节数。
2. Bytes_sent：表示MySQL服务器发送的字节数。
3. Bytes_total：表示MySQL服务器接收和发送的总字节数。