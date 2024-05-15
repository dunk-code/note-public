[文档](https://studygolang.com/pkgdoc)

[官网](https://pkg.go.dev/)

[菜鸟](https://www.runoob.com/go/go-map.html)

[地鼠文档](https://www.topgoer.cn/)

[学习资料大全](https://topgoer.com/go%E5%9F%BA%E7%A1%80/)



gvm：golang切换版本https://www.hi-linux.com/posts/20165.html

# 基础了解

## Google为什么要创造Go语言

- 计算机硬件技术更新频繁，性能提高很快。目前主流的编程语言发展明显落后于硬件，不能合理利用多核多CPU的优势提升软件系统性能。
- 软件系统复杂度越来越高，维护成本越来越高，目前缺乏一个足够简洁高效的编程语言。【现有的编程语言: 1.风格不统一 2.计算能力不够 3.处理大并发不够好】
- 企业运行维护很多c/c++的项目，c/c++程序运行速度虽然很快，但是编译速度确很慢，同时还存在内存泄漏的一系列的困扰需要解决。

## 特点

> 简介

Go 语言保证了既能到达静态编译语言的安全和性能，又达到了动态语言开发维护的高效率，使用一个表达式来形容Go语言：Go=C＋Python，说明Go语言既有C静态语言程序的运行速度，又能达到Python动态语言的快速开发。

- 从C语言中继承了很多理念，包括表达式语法，控制结构，基础数据类型，调用参数传值，指针等等，也保留了和C语言一样的编译执行方式及弱化的指针

```go
//go 语言的指针的使用特点(体验)
func testPtr(num *int) {
	*num = 20
}
```

- 引入包的概念，用于组织程序结构，Go语言的一个文件都要归属一个包，而不能单独存在
- 垃圾回收机制，内存自动回收，不需开发人员管理
- 天然并发（重要特点）
  - 从语言层面支持并发，实现简单2
  - goroutine，轻量级线程，可实现大并发处理，高效利用多核。
  - 基于CPS并发模型(Communicating Sequential Processes)实现
- 吸收了管道通信机制，形成Go语言特有的管道channel。通过管道channel，可以实现不同的goroute之间的相互通信。
- 函数可以返回多个值。

```go
//写一个函数，实现同时返回 和，差
//go 函数支持返回多个值
func getSumAndSub(n1 int, n2 int)(int,int){
	sum := n1 + n2 //go 语句后面不要带分号.
	sub := n1 - n2
	return sum , sub
}
```

- 新的创新：比如切片slice、延时执行defer



## 匿名导包和别名导包

如果在没有使用到某个包中的函数的情况下，想要调要该包下的init()函数，那么需要匿名导包

```go
import _	***
```

如果导包的时候，想要给包赋予别名

```go
import mypkg ***
// 如果调包中函数不想带包名
import . ***
```

## defer关键字

defer语句，压栈执行，先写的，先入栈，后执行。后写的，后入栈，后执行

defer在return之后执行

## slice

![img](https://www.topgoer.cn/uploads/golang/images/m_b3ed4a54677e9c668021c12d6cac6258_r.png)

> 动态数组

```go
// %v打印任何类型的详细信息
fmt.Printf("nums: %v\n", nums)
```

## sort比较器

### 基本int型

```go
type IntSlice []int

func (p IntSlice) Len() int           { return len(p) }
func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p IntSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// Sort is a convenience method.
func (p IntSlice) Sort() { Sort(p) }
```



### 自定义类型

想要让自定义类型的集合能够排序，需要该集合实现该Interface保证类型一致，该接口有以下三个方法

```go
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
```

举例：

根据用户年龄进行排序

用户类

```go
type User struct {
	name string
	age  int
}
```

实现接口

```go
type Users []User

func (s Users) Len() int { //返回集合的长度
	return len(s)
}

func (s Users) Less(i, j int) bool { //用来决定是升序还是降序
	// 降序
	// return s[i].age > s[j].age
	// 生序
	return s[i].age < s[j].age
}

func (s Users) Swap(i, j int) { //改变数据在集合中的位置
	s[i], s[j] = s[j], s[i]
}
```

验证

```golang
func main() {
	var users = make(Users, 0)
	for i := 0; i < 10; i++ {
		j := rand.Intn(100)
		user := User{
			name: strconv.Itoa(j) + "~~",
			age:  j,
		}
		users = append(users, user)
	}
	for _, item := range users {
		fmt.Println(item.name, "--", item.age)
	}
	sort.Sort(users)
	fmt.Println("is sorted? ", sort.IsSorted(users)) //可以检测是否已经排序
	for _, item := range users {
		fmt.Println(item.name, "--", item.age)
	}
}
```

### Sort.Slice

```go
func main() {
	var users = make(Users, 0)
	for i := 0; i < 10; i++ {
		j := rand.Intn(100)
		user := User{
			name: strconv.Itoa(j) + "~~",
			age:  j,
		}
		users = append(users, user)
	}
	sort.Slice(users, func(i, j int) bool {
		return users[i].age > users[j].age
	})
	for _, item := range users {
		fmt.Println(item.name, "--", item.age)
	}
}
```



## 反射

#### 变量

<img src="/Users/songyao.zhang/Library/Application Support/typora-user-images/image-20220509215057780.png" alt="image-20220509215057780" style="zoom:80%;" />

```go
type Writer interface {
	write()
}

type Reader interface {
	read()
}

type Book struct {
}

func (b *Book) read() {
	fmt.Println("book read...")
}

func (b *Book) write() {
	fmt.Println("book write...")
}

func main() {
	// b:pair<type:Book, value:Book{}地址>
	b := &Book{}
	var r Reader
	// r:pair<type:Book, value:Book{}地址>
	r = b
	r.read()
	// w:pair<type:Book, value:Book{}地址>
	w := r.(Writer)
	w.write()
}
```

## goroutine

#### java中的线程模型

> https://zhong0316.github.io/2018/01/26/java%E7%BA%BF%E7%A8%8B%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F/

#### 使用内核线程（Kernel-Level Thread，KLT）实现

![使用内核线程实现](https://zhong0316.github.io/images/posts/java/java%E7%BA%BF%E7%A8%8B%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F1.png)

#### 使用用户线程实现

![使用用户线程实现](https://zhong0316.github.io/images/posts/java/java%E7%BA%BF%E7%A8%8B%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F2.png)

#### 使用用户线程加轻量级进程混合实现

![混合实现](https://zhong0316.github.io/images/posts/java/java%E7%BA%BF%E7%A8%8B%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F3.png)

对于Sun JDK来说，它的Windows版与Linux版都是使用一对一的线程模型实现的，一条Java线程就映射到一条轻量级进程之中，因为Windows和Linux系统提供的线程模型就是一对一的。而在Solari平台中，由于操作系统的线程特性可以同时支持一对一（通过Bound Threads或Alternate Libthread实现）及一对多（通过LWP/Thread Based Synchronization实现）的线程模型，因此在Solaris版的JDK中也对应提供了两个平台专有的虚拟机参数：-XX:+UseLWPSynchronization（默认值）和-XX:+UseBoundThreads来明确指定虚拟机使用哪种线程模型。

#### golang中的

![image-20220524220634337](../../../Library/Application%20Support/typora-user-images/image-20220524220634337.png)



> 优化协程调度器

![image-20220522192203159](../../../Library/Application%20Support/typora-user-images/image-20220522192203159.png)

#### GMP

GPM是Go语言运行时（runtime）层面的实现，是go语言自己实现的一套调度系统。区别于操作系统调度OS线程。

- 1.G很好理解，就是个goroutine的，里面除了存放本goroutine信息外 还有与所在P的绑定等信息。
- 2.P管理着一组goroutine队列，P里面会存储当前goroutine运行的上下文环境（函数指针，堆栈地址及地址边界），P会对自己管理的goroutine队列做一些调度（比如把占用CPU时间较长的goroutine暂停、运行后续的goroutine等等）当自己的队列消费完了就去全局队列里取，如果全局队列里也消费完了会去其他P的队列里抢任务。
- 3.M（machine）是Go运行时（runtime）对操作系统内核线程的虚拟， M与内核线程一般是一一映射的关系， 一个groutine最终是要放到M上执行的；

P与M一般也是一一对应的。他们关系是： P管理着一组G挂载在M上运行。当一个G长久阻塞在一个M上时，runtime会新建一个M，阻塞G所在的P会把其他的G 挂载在新建的M上。当旧的G阻塞完成或者认为其已经死掉时 回收旧的M。

P的个数是通过runtime.GOMAXPROCS设定（最大256），Go1.5版本之后默认为物理线程数。 在并发量大的时候会增加一些P和M，但不会太多，切换太频繁的话得不偿失。

单从线程调度讲，Go语言相比起其他语言的优势在于OS线程是由OS内核来调度的，goroutine则是由Go运行时（runtime）自己的调度器调度的，这个调度器使用一个称为m:n调度的技术（复用/调度m个goroutine到n个OS线程）。 其一大特点是goroutine的调度是在用户态下完成的， 不涉及内核态与用户态之间的频繁切换，包括内存的分配与释放，都是在用户态维护着一块大的内存池， 不直接调用系统的malloc函数（除非内存池需要改变），成本比调度OS线程低很多。 另一方面充分利用了多核的硬件资源，近似的把若干goroutine均分在物理线程上， 再加上本身goroutine的超轻量，以上种种保证了go调度方面的性能。

## SingleFlight

https://www.cyningsun.com/01-11-2021/golang-concurrency-singleflight.html

```go
// Do():  相同的 key，fn 同时只会执行一次，返回执行的结果给fn执行期间，所有使用该 key 的调用
// v: fn 返回的数据
// err: fn 返回的err
// shared: 表示返回数据是调用 fn 得到的还是其他相同 key 调用返回的
func (g *Group) Do(key string, fn func() (interface{}, error)) (v interface{}, err error, shared bool) {
// DoChan(): 类似Do方（），以 chan 返回结果
func (g *Group) DoChan(key string, fn func() (interface{}, error)) <-chan Result {
// Forget(): 失效 key，后续对此 key 的调用将执行 fn，而不是等待前面的调用完成
func (g *Group) Forget(key string)
```



## channel

buffered channel

unbuffered channel



> channel被关闭时，可以一直被读取，读取到空

### Select





- 性能分析 （profiling）
- 基准测试（benchmark）
- 重构（re）

> 性能分析

pprof

flame graph

> 基准测试

![image-20220913160237255](https://gitee.com/zhang-songyao/blog-images/raw/master/202209131602839.png)











# Go操作MySQL

[github源码](https://github.com/go-sql-driver/mysql)

## 注册驱动

终端执行命令

```bash
go get -u github.com/go-sql-driver/mysql
```

发现报错

```
go get -u github.com/go-sql-driver/mysql

go get github.com/go-sql-driver/mysql: module github.com/go-sql-driver/mysql: Get "https://proxy.golang.org/github.com/go-sql-driver/mysql/@v/list": dial tcp
 172.217.163.49:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established conne
ction failed because connected host has failed to respond.
```

墙的问题，配置下代理就行，可以用七牛的代理，配置如下

```bash
go env -w GOPROXY=https://goproxy.cn
```

设置完成之后可以查看 是否设置成功

```bash
go env
```

驱动下载成功

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220315203829.png" alt="image-20220315203825507" style="zoom:80%;" />



测试是否连接成功

```go
import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
	"log"
)

func main() {
	db, err := sql.Open("mysql", "root:333@tcp(127.0.0.1:3306)/bjpowernode")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()
}
```

说明

```go
func Open(driverName, dataSourceName string) (*DB, error)
```

1. `sql.Open`的第一个参数是，驱动程序名称。这是驱动程序用来在`database/sql`中进行自身注册的字符串，通常与包名称相同，以避免混淆。例如，`mysql` 用于 [github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql) 。某些驱动程序不遵循约定，而是使用数据库名称，例如，用于 [github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3) 的`sqlite3` 和用于 [github.com/lib/pq](https://github.com/lib/pq) 的 `postgres`。
2. 第二个参数是，特定于驱动程序的语法，它告诉驱动程序如何访问基础数据存储。在此示例中，我们将连接到本地 MySQL 服务器实例内的「hello」数据库。
3. 您(几乎)应该始终检查并处理从所有 `database/sql` 操作返回的错误。我们将在以后讨论一些特殊情况，以解决此问题。
4. 如果 `sql.DB` 的生存期不应超出该功能的范围，则推迟 `db.Close()` 是习惯的做法。

> `sql.Open()` **不会建立与数据库的任何连接**，也不会验证驱动程序的连接参数。而是它只准备数据库抽象以备后用。与基础数据存储区的第一个实际连接将延迟到第一次需要时建立。如果要立即检查数据库是否可用和可访问(例如，检查是否可以建立网络连接并登录)，请使用 `db.Ping()` 进行操作，并记录检查错误

```go
err4ping := db.Ping()
if err4ping != nil {
    fmt.Println(err4ping)
}
```

虽然在完成后对 `Close()` 数据库很习惯，**但是 `sql.DB` 对象被设计为长期存在**。不要经常使用 `Open()` 和 `Close()`。而是为每个需要访问的不同数据存储创建 **一个** `sql.DB` 对象，并保留该对象直到程序完成对该数据存储的访问为止。根据需要传递它，或以某种方式使其在全局范围内可用，但保持打开状态。不要通过短期函数来 `Open()` 和 `Close()`。而是将 `sql.DB` 传递给该短期函数作为参数。

如果不将 `sql.DB` 视为长期对象，则可能会遇到诸如重用和连接共享不良，可用网络资源用尽或由于很多原因而导致的偶发性故障等问题 `TIME_WAIT` 状态中剩余的 TCP 连接数。这些问题表明您没有使用设计的 `database/sql`。

## 处理结果集

### 查询

有几种常用的操作可以从数据库中检索结果。

1. 执行返回一条数据的查询操作。
2. 准备要重复使用的语句，在多次执行该语句后进行销毁。
3. 一次性的执行语句，并且不打算将其重复使用。
4. 执行返回一条数据的查询，这种特殊情况有一个快捷方式。

Go 的 `database/sql` 包中函数名称很重要。**如果一个函数名字包含 `Query`, 那么该函数旨在向数据库发出查询问题, 并且即使它为空，也将返回一组行**。不返回行的语句不应该使用 `Query` 函数; 而应使用 `Exec()`

```go
func query() {
	var (
		username string
		password string
	)
	// Query()将请求发送给mysql查询数据
	rows, err := db.Query("select username, password from t_user where id in (?,?) ", 1, 5)
	if err != nil {
		log.Fatal(err)
	}
	// 关闭资源
	defer rows.Close()
	// 遍历rows结果集
	for rows.Next() {
		// Scan()遍历并赋值给变量
		err := rows.Scan(&username, &password)
		if err != nil {
			log.Fatal(err)
		}
		log.Println(username, password)
	}
	// 遍历后，检查是否出错
	err = rows.Err()
	if err != nil {
		log.Fatal(err)
	}
}
```

> 注意

- 始终需要检查`for rows.Next()`循环末尾是否有错，如果循环中存在错误，需要知道它
- 要存在打开的结果集(由 `rows` 表示)，底层连接就很忙，不能用于任何其他查询。这意味着它在连接池中不可用。如果使用 `rows.Next()` 迭代所有行，最终将读取最后一行，并且`rows.Next()` 将遇到内部 EOF 错误并为您调用 `rows.Close()`。但是，如果出于某种原因您退出该循环——提前返回，等等情况——那么 `rows` 不会关闭，连接保持打开。(不过，如果`rows.Next()` 由于错误返回 false，那么它会自动关闭)。这是耗尽资源的一种简单方式
- **始终** `defer rows.Close()`，循环内，不要使用`defer`，延迟语句不会在循环中执行，会导致内存溢出，如果在循环中重复使用查询和使用结果集，应该在每个处理完后显式调用`row.Close()`

> Query()函数分析

`Query()` 将返回一个 `sql.Rows`，它将保留数据库连接，直到 `sql.Rows` 关闭。由于可能存在未读数据 (例如，更多的数据行)，因此无法使用该连接。

如果不关闭`sql.Rows`，连接将 *永远* 不会被释放。垃圾收集器最终将为您关闭底层 `net.Conn`，但这可能需要很长时间。此外，`database/sql` 包会在连接池中继续跟踪连接，希望您在某个时候释放它，以便可以再次使用该连接。因此，此反模式是耗尽资源 (例如，连接过多) 的好方法

> Scan()怎么工作

当遍历行并将其扫描到目标变量中时，Go 会在后台执行数据类型转换。它基于目标变量的类型。

### 预处理查询

通常，您应该始终对要多次使用的查询执行预处理。预处理查询的结果是一个预处理语句，该语句可以具有占位符(也称为绑定值)，用于执行该语句时将提供的参数。出于所有常见原因(例如，避免 SQL 注入攻击)，这比串联字符串好得多

```go
func prepareQuery() {
	stmt, err := db.Prepare("select username, password from t_user where id = ? ")
	if err != nil {
		log.Fatal(nil)
	}
	defer stmt.Close()
	rows, err := stmt.Query(11)
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()
	for rows.Next() {
		//..
	}
	if err = rows.Err(); err != nil {
		log.Fatal(err)
	}
}
```

### 更新

使用 `Exec()`，最好是使用预编译语句，来完成 `INSERT`，`UPDATE`，`DELETE` 或其他不返回行的语句。以下示例显示如何插入行并检查有关该操作的元数据：

> 错误处理函数

```go
func logErr(err error) {
	if err != nil {
		log.Fatal(err)
	}
}
```

修改操作：

```go
func insert() {
	stmt, err := db.Prepare("insert into t_user(username, password) VALUES (?, ?)")
	logErr(err)
	result, err := stmt.Exec("zhangsan", "77625")
	logErr(err)
	lastId, err := result.LastInsertId()
	logErr(err)
	rowCnt, err := result.RowsAffected()
	logErr(err)
	log.Printf("ID = %d, affected = %d\n", lastId, rowCnt)
}
```

> 执行该语句将产生一个 `sql.Result`，它提供对语句元数据的访问：最后插入的 ID 和受影响的行数

删除操作

```go
func delete() {
	stmt, err := db.Prepare("delete FROM t_user where id = ?")
	logErr(err)
	result, err := stmt.Exec(19)
	logErr(err)
	lastId, err := result.LastInsertId()
	logErr(err)
	rowCnt, err := result.RowsAffected()
	logErr(err)
	log.Printf("ID = %d, affected = %d\n", lastId, rowCnt)
}
```

## 事务

Go中，事务本质上是一个保留与数据存储区连接的对象。它可以让您执行到目前为止所看到的所有操作，但可以保证它们将在同一连接上执行

可以通过调用 `db.Begin()` 开启事务，然后使用该函数生成的 `Tx` 对象上的 `Commit()` 或 `Rollback()` 方法来结束事务。在后台，`Tx` 从池中获得连接，并将其保留，以仅用于该事务。`Tx` 上的方法一对一映射到您可以在数据库本身上调用的方法，例如 `Query()` 等

```go
func testTx() {
	tx, err := db.Begin()
	if err != nil {
		panic(err)
	}
	// 如果执行过程中都没有 commit和rollback那就由这个来收尾
	defer clearTransaction(tx)
	stmt, err := tx.Prepare("insert into t_user(username, password) VALUES (?, ?)")
	if err != nil {
		tx.Rollback()
		return
	}
    defer stmt.Close()
	result, err := stmt.Exec("王五", "123")
	if err != nil {
		tx.Rollback()
		return
	}
	affected, err := result.RowsAffected()
	if err != nil {
		tx.Rollback()
		return
	}
	fmt.Println(affected)
	tx.Commit()
}

func clearTransaction(tx *sql.Tx) {
	err := tx.Rollback()
	if err != nil {
		return
	} else {
		fmt.Println(err)
	}
}
```

在事务内部进行操作时，应注意==不要调用 `db` 变量==。进行所有对您使用 `db.Begin()` 创建的 `Tx` 变量的调用。`db` 不在事务中，只有 `Tx` 对象在事务中。如果您进一步调用 `db.Exec()` 或类似方法，则这些调用将在事务范围之外发生在其他连接上

如果需要使用多个修改连接状态的语句，即使您本身不需要事务，也需要 `Tx`。例如：

- 创建临时表，仅对一个连接可见。
- 设置变量，例如 MySQL 的 `SET @var:= somevalue` 语法。
- 更改连接选项，例如字符集或超时。

如果您需要执行上述任何操作，则需要将活动绑定到单个连接，而 Go 中唯一的方法是使用 `Tx`。

## 预处理

### MySQL预处理

> 普通SQL语句执行过程

1. 客户端对SQL语句进行占位符替换得到完成的SQL语句
2. 客户端发送完整SQL语句到MySQL服务端
3. MySQL服务端执行完整的SQL语句并进行返回客户端

> 预处理执行过程

1. 把SQL语句分为两部分，命令部分与数据部分
2. 先把命令部分发送给MySQL服务端，MySQL服务端进行SQL预处理
3. 然后把数据部分发送给MySQL服务端，MySQL服务端对SQL进行占位符替换
4. 执行完整SQL并将结果即返回

> 为什么要预处理？

1. 优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，==一次编译多次执行==，节省后续的编译成本
2. 避免SQL注入的问题

### Go语言实现MySQL预处理

事务那里已经展示过了

## sqlx

> 注册驱动

```bash
go get github.com/jmoiron/sqlx
```

> sqlx带来的便利

通过反射获取struct的数据类型指针进行赋值，不需要逐个的传递参数

```go
err := rows.Scan(&username, &password)
```

```go
import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
	"log"
)

var db *sqlx.DB

type User struct {
	// 注意这里的属性必须大写
	Id       int
	Username string
	Password string
}

func initDB() (err error) {
	db, err = sqlx.Connect("mysql", "root:333@tcp(127.0.0.1:3306)/spring5")
	if err != nil {
		return
	}
	// 最大连接数
	db.SetMaxOpenConns(10)
	//最大空闲连接数
	db.SetMaxIdleConns(5)
	return
}
func main() {
	err := initDB()
	if err != nil {
		log.Fatalln(err)
	}
	var user User
	err = db.Get(&user, "select id, username, password from t_user where id = 1")
	if err != nil {
		log.Fatalln(err)
	}
	fmt.Println(user)
}
```

查询多个结果

```go
var users []User
err = db.Select(&users, "select id, username, password from t_user where id > ?", 5)
if err != nil {
    log.Fatalln(err)
}
for _, user := range users {
    fmt.Println(user)
}
```

# gin

[官方文档](https://gin-gonic.com/zh-cn/docs/introduction/)

## 简介

Gin是一个用Go(Golang)编写的HTTP Web框架，封装比较优雅，API友好，源码注释比较明确，具有快速灵活，容错方便等特点

> 特性

- 快速
  - 基于 Radix 树的路由，小内存占用。没有反射。可预测的 API 性能
- 支持中间件
  - 传入的 HTTP 请求可以由一系列中间件和最终操作来处理。 例如：Logger，Authorization，GZIP，最终操作 DB
- Crash处理
  - Gin 可以 catch 一个发生在 HTTP 请求中的 panic 并 recover 它。这样，你的服务器将始终可用。例如，你可以向 Sentry 报告这个 panic
- JSON验证
  - Gin 可以解析并验证请求的 JSON，例如检查所需值的存在
- 路由组
  - 更好地组织路由。是否需要授权，不同的 API 版本…… 此外，这些组可以无限制地嵌套而不会降低性能
- 错误管理
  - Gin 提供了一种方便的方法来收集 HTTP 请求期间发生的所有错误。最终，中间件可以将它们写入日志文件，数据库并通过网络发送
- 内容渲染
  - Gin 为 JSON，XML 和 HTML 渲染提供了易于使用的 API
- 可扩展性
  - 新建一个中间件非常简单

## 快速入门

> 要求Go1.13版本以上

### 安装

下载并安装Gin

```bash
go get -u github.com/gin-gonic/gin
```

将Gin引入代码中

```go
import "github.com/gin-gonic/gin"
```

如果使用诸如 `http.StatusOK` 之类的常量，则需要引入 `net/http` 包

```go
import "net/http"
```

### HelloWorld

```go
import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	// 创建路由
	r := gin.Default()
	// 绑定路由规则，执行函数
	// gin.Context，封装了request和response
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "HelloWord",
		})
	})
	// 监听端口， 默认在8080
	r.Run(":8000")
}
```

请求`localhost:8000/ping`，获取结果

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220318170548.png" alt="image-20220318170538332" style="zoom:80%;" />

## Gin路由

### 基本路由

Gin框架采用的路由库是基于[httprouter](https://github.com/julienschmidt/httprouter)做的

### RestFul风格

- gin支持Restful风格的API
- 即Representational State Transfer的缩写。直接翻译的意思是"表现层状态转化"，是一种互联网应用程序的API设计理念：URL定位资源，用HTTP描述操作

1. 获取文章 /blog/getXxx ==Get blog/Xxx==
2. 添加 /blog/addXxx ==POST blog/Xxx==
3. 修改 /blog/updateXxx ==PUT blog/Xxx==
4. 删除 /blog/delXxxx ==DELETE blog/Xxx==

### API参数

可以通过Context的Param方法来获取API参数

```go
func main() {
	r := gin.Default()
	// * 通配符
	r.GET("/user/:name/*password", func(c *gin.Context) {
		name := c.Param("name")
		password := c.Param("password")
		// 截取字符串
		password = strings.Trim(password, "/")
		c.String(http.StatusOK, name+" "+password)
	})
	r.Run(":8000")
}
```

### URL参数

- URL参数可以通过DefaultQuery()或Query()方法获取
- DefaultQuery()若参数不存在则，返回默认值，Query()若不存在，返回空串
- API ? name=zs

```go
func main() {
	r := gin.Default()
	r.GET("/user", func(c *gin.Context) {
		// 指定name的默认值，不存在则赋值张三
		name := c.DefaultQuery("name", "张三")
		c.String(http.StatusOK, fmt.Sprintf("hello %s", name))
	})
	r.Run(":8000")
}
```

### 表单参数

- 表单传输为post请求，http常见的传输格式为四种：
  - application/json
  - application/x-www-form-urlencoded
  - application/xml
  - multipart/form-data
- 表单参数可以通过PostForm()方法获取，该方法默认解析的是`x-www-form-urlencoded`或`from-data`格式的参数

创建表单

```html
<div align="center">
    <form action="http://localhost:8000/form" method="post" enctype="application/x-www-form-urlencoded">
        用户名：<input type="text" name="username" placeholder="请输入你的用户名"> <br>
        密&nbsp;&nbsp;&nbsp;码：<input type="password" name="userpassword" placeholder="请输入你的密码"> <br>
        <input type="submit" value="提交">
    </form>
</div>
```

接受表单数据

```go
func main() {
	r := gin.Default()
	r.POST("/form", func(c *gin.Context) {
		types := c.DefaultPostForm("type", "post")
		username := c.PostForm("username")
		password := c.PostForm("userpassword")
		c.String(http.StatusOK, fmt.Sprintf("username:%s, password:%s, types:%s", username, password, types))
	})
	r.Run(":8000")
}
```

### 上传单个文件

- multipart/form-data格式用于文件上传
- gin文件上传与原生的net/http方法类似，不同在于gin把原生的request封装到c.Request中

前端页面

```html
<div>
    <form action="http://localhost:8000/upload" method="post" enctype="multipart/form-data">
        上传文件:<input type="file" name="file" >
        <input type="submit" value="提交">
    </form>
</div>
```

上传单个文件

```go
func main() {
	r := gin.Default()
	// 设置上传最大尺寸
	r.MaxMultipartMemory = 8 << 20
	r.POST("/upload", func(c *gin.Context) {
		file, err := c.FormFile("file")
		if err != nil {
			c.String(500, "上传文件失败")
		}
		c.SaveUploadedFile(file, file.Filename)
		c.String(http.StatusOK, file.Filename)
	})
	r.Run(":8000")
}
```

上传指定大小、格式的图片

```go
func main() {
	r := gin.Default()
	// 设置上传最大尺寸
	r.POST("/upload", func(c *gin.Context) {
		_, headers, err := c.Request.FormFile("file")
		if err != nil {
			log.Printf("Error when try to get file: %v", err)
		}
		// 获取文件大小
		if headers.Size > 1024*1024*2 {
			c.String(500, "文件过大")
			return
		}
		// 获取文件上传类型
		if headers.Header.Get("Content-Type") != "image/png" {
			c.String(500, "只允许上传png格式的图片")
			return
		}
		c.SaveUploadedFile(headers, "./src/image/"+headers.Filename)
		c.String(http.StatusOK, headers.Filename)
	})
	r.Run(":8000")
}
```

### 上传多个文件

前端页面

```html
<div>
  <form action="http://localhost:8000/upload" method="post" enctype="multipart/form-data">
    上传文件:<input type="file" name="files" multiple>
    <input type="submit" value="提交">
  </form>
</div>
```

后端代码

```go
func main() {
	r := gin.Default()
	r.POST("/upload", func(c *gin.Context) {
		form, err := c.MultipartForm()
		if err != nil {
			c.String(http.StatusBadRequest, fmt.Sprintf("get err %s", err.Error()))
			return
		}
		files := form.File["files"]
		for _, file := range files {
			err := c.SaveUploadedFile(file, "./src/files/"+file.Filename)
			if err != nil {
				c.String(http.StatusBadRequest, fmt.Sprintf("upload err %s", err.Error()))
				return
			}
		}
		c.String(200, fmt.Sprintf("upload ok %d files", len(files)))
	})
	r.Run(":8000")
}
```

### routes group

- routes group是为了管理一些相同的URL

```go
func main() {
	r := gin.Default()
	// 路由组1
	admin := r.Group("/admin")
	// {}：书写规范
	{
		admin.GET("/login", login)
		admin.GET("/submit", submit)
	}
	user := r.Group("/user")
	{
		user.GET("/login", func(c *gin.Context) {
			c.String(http.StatusOK, "用户你好")
		})
		user.GET("/submit", submit)
	}
	r.Run(":8000")
}

func submit(c *gin.Context) {
	c.String(http.StatusOK, "提交成功")
}

func login(c *gin.Context) {
	name := c.DefaultQuery("name", "admin")
	c.String(http.StatusOK, "管理员："+name+" 您好！")
}
```

### 路由拆分与注册

#### 基本的路由注册

下面最基础的gin路由注册方式，适用于路由条目比较少的简单项目或者项目demo

```golang
import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func helloHandler(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"message": "Hello",
	})
}

func main() {
	r := gin.Default()
	r.GET("/hello", helloHandler)
	if err := r.Run(); err != nil {
		fmt.Println("startup service failed, err:%v\n", err)
	}
}
```

#### 路由拆分成单独的文件包

当项目的规模增大后就不太适合继续在项目的main.go文件中去实现路由注册相关逻辑了，我们会倾向于把路由部分的代码都拆分出来，形成一个单独的文件或包：

在`routers.go`文件中定义并注册路由信息：

```golang
import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func helloHandler(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"message": "Hello",
	})
}

func setupRouter() *gin.Engine {
	r := gin.Default()
	r.GET("/hello", helloHandler)
	return r
}
```

此时`main.go`中调用上面定义好的`setupRouter`函数：

```golang
func main() {
	r := routers.SetupRouter()
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup service failed, err:%v\n\n", err)
	}
}
```

此时的目录结构:

```
gin_demo
├── go.mod
├── go.sum
├── main.go
└── routers.go
```

把路由部分的代码单独拆分成包的话也是可以的，拆分后的目录结构如下：

```
gin_demo
├── go.mod
├── go.sum
├── main.go
└── routers
    └── routers.go
```

`routers/routers.go`需要注意此时`setupRouter`需要改成首字母大写：

```golang
func SetupRouter() *gin.Engine {
	r := gin.Default()
	r.GET("/hello", helloHandler)
	return r
}
```

#### 路由拆分为个文件

当业务规模继续膨胀，单独的一个routers文件或包已经满足不了我们的需求了，因为我们把所有的路由注册都写在一个`SetupRouter`函数中的话就会太复杂了。

我们可以分开定义多个路由文件，例如：

```
gin_demo
├── go.mod
├── go.sum
├── main.go
└── routers
    ├── blog.go
    └── user.go
```

`routers/blog.go`中添加一个`LoadBlog`的函数，将`blog`相关的路由注册到指定的路由器：

```golang
func LoadBlog(e *gin.Engine) {
    e.GET("/post", postHandler)
  e.GET("/comment", commentHandler)
  ...
}
```

`main.go`函数中最后实现逻辑

```golang
func main() {
    r := gin.Default()
    routers.LoadBlog(r)
    routers.LoadUser(r)
    if err := r.Run(); err != nil {
        fmt.Println("startup service failed, err:%v\n", err)
    }
}
```

#### 路由拆分不同APP

有时候项目规模实在太大，那么我们就更倾向于把业务拆分的更详细一些，例如把不同的业务代码拆分成不同的APP。

因此我们在项目目录下单独定义一个`app`目录，用来存放我们不同业务线的代码文件，这样就很容易进行横向扩展。大致目录结构如下：

```
gin_demo
├── app
│   ├── blog
│   │   ├── handler.go
│   │   └── router.go
│   └── shop
│       ├── handler.go
│       └── router.go
├── go.mod
├── go.sum
├── main.go
└── routers
    └── routers.go
```

其中`app/blog/router.go`用来定义blog相关路由信息，具体内容如下：

```golang
func Routers(r *gin.Engine) {
	r.GET("/get", search)
	r.GET("/del", delete)
}
```

同理：`app/shop/router.go`用来定义shop相关路由信息

`routers/routers.go`中根据需要定义`Include`函数用来注册子`app`中定义的路由，`Init`函数用来进行路由的初始化操作：

```golang
type Option func(*gin.Engine)

var options = []Option{}

// Include 注册app的路由配置
func Include(opts ...Option) {
	options = append(options, opts...)
}

// Init 初始化
func Init() *gin.Engine {
	r := gin.Default()
	for _, opt := range options {
		opt(r)
	}
	return r
}
```

`main.go`中按如下方式先注册子`app`中的路由，然后再进行路由的初始化：

```go
func main() {
	// 加载多个路由配置
	routers.Include(shop.Routers, blog.Routers)
	// 初始化路由
	r := routers.Init()
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup service failed, err:%v\n\n", err)
	}
}
```

## Gin数据解析和绑定

### Json数据解析和绑定

- 客户端传参，后端接收并解析到结构体

```golang
// User 定义接收数据的结构体
type User struct {
	// binding:"required"修饰的字段，若接收为空值，则报错，是必须字段
	Username string `form:"username" json:"username" uri:"username" xml:"username" binding:"required"`
	Password string `form:"password" json:"password" uri:"password" xml:"password" binding:"required"`
}

func login(c *gin.Context) {
	var json User
	// 将request的body中的数据，自动按照json格式解析到结构体
	if err := c.ShouldBindJSON(&json); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"err": err.Error(),
		})
		return
	}
	if json.Username != "root" && json.Password != "root" {
		// gin.H封装了生成json数据的工具
		c.JSON(http.StatusOK, gin.H{
			"message": "用户名或密码错误",
		})
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"message": "登录成功",
	})
}

func main() {
	r := gin.Default()
	r.POST("/jsonLogin", login)
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server failed err : %s\n", err)
	}
}
```

发送请求

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220319180029.png" alt="image-20220319180026371" style="zoom:80%;" />

### 表单数据解析和绑定

前端页面

```html
<div align="center">
    <form action="http://localhost:8000/formLogin" method="post">
        用户名：<input name="username" type="text"><br/>
        密 码：<input type="password" name="password"><br/>
        <input type="submit" value="提交">
    </form>
</div>
```

使用`Bind()`进行数据解析和绑定

```golang
func login(c *gin.Context) {
	var form User
	// 默认解析并绑定form格式
	// 根据请求头中content-type自动推断
	if err := c.Bind(&form); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"err": err.Error(),
		})
	}
	if form.Username != "root" && form.Password != "root" {
		// gin.H封装了生成json数据的工具
		c.JSON(http.StatusOK, gin.H{
			"message": "用户名或密码错误",
		})
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"message": "登录成功",
	})
}

func main() {
	r := gin.Default()
	r.POST("/formLogin", login)
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server failed err : %s\n", err.Error())
	}
}
```

效果展示

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220321164413.png" alt="image-20220321164410597" style="zoom:80%;" />

### URI数据解析和绑定

后端代码

```golang
func login(c *gin.Context) {
	var uri User
	if err := c.ShouldBindUri(&uri); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"err": err.Error(),
		})
	}
	if uri.Username != "root" && uri.Password != "root" {
		// gin.H封装了生成json数据的工具
		c.JSON(http.StatusOK, gin.H{
			"message": "用户名或密码错误",
		})
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"message": "登录成功",
	})
}

func main() {
	r := gin.Default()
	r.GET("uriLogin/:username/:password", login)
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server failed err : %s\n", err)
	}
}
```

效果展示

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220321164912.png" alt="image-20220321164909157" style="zoom:80%;" />

## Gin渲染

### 各种数据格式的响应

- `json`、结构体、`XML`、`YAML`类似于java的`properties`、`ProtoBuf`

```golang
func main() {
	r := gin.Default()
	// json
	r.GET("/json", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "json",
		})
	})
	// 结构体
	r.GET("/struct", func(c *gin.Context) {
		var user struct {
			id       int
			username string
			password string
		}
		user.id = 1
		user.username = "张三"
		user.password = "123"
		c.JSON(200, user)
	})
	// XML
	r.GET("/xml", func(c *gin.Context) {
		c.XML(200, gin.H{"message": "xml"})
	})
	// YAML
	r.GET("/yaml", func(c *gin.Context) {
		c.YAML(200, gin.H{"message": "yaml"})
	})
	// protobuf格式，谷歌开发的高效存储读取工具
	r.GET("/protoBuf", func(c *gin.Context) {
		reps := []int64{int64(1), int64(2)}
		// 定义数据
		label := "label"
		// 传protobuf格式数据
		data := &protoexample.Test{
			Label: &label,
			Reps:  reps,
		}
		c.ProtoBuf(200, data)
	})
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server failed err : %s\n", err)
	}
}
```

### HTML模板渲染

- gin支持加载HTML模板, 然后根据模板参数进行配置并返回相应的数据，本质上就是字符串替换
- `LoadHTMLGlob()`方法可以加载模板文件

> 注意修改goland的工作路径

使用`goland`时，默认的工作路径为项目根目录，如果代码和模板文件都位于`src`目录下，则会找不到

报错：

```
panic: html/template: pattern matches no files: `tem/**/*`
```

修改路径：

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220321180953.png" alt="image-20220321180950050" style="zoom:80%;" />

目录结构

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220321181044.png" alt="image-20220321181041495" style="zoom:80%;" />

前端

```html
<head>
    <meta charset="UTF-8">
    <title>{{.title}}</title>
</head>
<body>
<div align="center">
    <h1>{{.message}}</h1>
</div>
</body>
</html>
```

后端

```golang
func main() {
	r := gin.Default()
	r.LoadHTMLGlob("./01test/tem/*")
	r.GET("/index", func(c *gin.Context) {
		c.HTML(200, "index.html", gin.H{
			"title":   "html模板渲染",
			"message": "Hello Gin",
		})
	})
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup serve failed err : %s\n", err.Error())
	}
}
```

- 如果你想进行头尾分离就是下面这种写法了：

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220321200522.png" alt="image-20220321200519373" style="zoom:80%;" />

```golang
func main() {
	r := gin.Default()
	r.LoadHTMLGlob("./02test/tem/**/*")
	r.GET("/index", func(c *gin.Context) {
		c.HTML(200, "user/index.html", gin.H{
			"title": "头尾分离模板渲染",
			"msg":   "Hello World",
		})
	})
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server err : %s\n", err)
	}
}
```

`admin/index.html`文件代码：

```
{{ define "user/index.html" }}
{{template "public/header" .}}
{{.msg}}
{{template "public/footer" .}}
{{ end }}
```

`public/header.html`文件代码：

```html
{{define "public/header"}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{.title}}</title>
</head>
<body>

{{end}}
```

`public/footer.html`文件代码：

```html
{{define "public/footer"}}
</body>
</html>
{{ end }}
```

- 如果你需要引入静态文件需要定义一个静态文件目录

```golang
r.Static("/assets", "./assets")
```

### 重定向

```golang
func main() {
	r := gin.Default()
	r.GET("/", func(c *gin.Context) {
		c.Redirect(http.StatusMovedPermanently, "https:www.baidu.com")
	})
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server failed err : %s\n", err)
	}
}
```

## Gin中间件

### 全局中间件

>所有请求都经过此中间件

```golang
// MiddleWare 定义中间件
func MiddleWare() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		log.Println("中间件开始执行了")
		// 设置变量到Context的key中，通过Get()获取
		c.Set("request", "中间件")
		status := c.Writer.Status()
		log.Println("中间件执行结束", status)
		t2 := time.Since(t)
		log.Println("中间件执行耗时", t2)
	}
}

func main() {
	r := gin.Default()
	// 注册中间件
	r.Use(MiddleWare())
	// {}代码规范
	{
		r.GET("/globalMiddle", func(c *gin.Context) {
			rq, _ := c.Get("request")
			fmt.Println("request:", rq)
			c.JSON(200, gin.H{
				"request": rq,
			})
		})
	}
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server err : %s\n", err.Error())
	}
}
```

输出结果：

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220321210627.png" alt="image-20220321210623054" style="zoom:80%;" />

### 局部中间件

```golang
// MiddleWare 定义中间件
func MiddleWare() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		log.Println("局部中间件开始执行了")
		// 设置变量到Context的key中，通过Get()获取
		c.Set("request", "局部中间件")
		status := c.Writer.Status()
		log.Println("局部中间件执行结束", status)
		t2 := time.Since(t)
		log.Println("局部中间件执行耗时", t2)
	}
}

func main() {
	r := gin.Default()
	// {}代码规范
	{
		r.GET("/localMiddle", MiddleWare(), func(c *gin.Context) {
			rq, _ := c.Get("request")
			fmt.Println("request:", rq)
			c.JSON(200, gin.H{
				"request": rq,
			})
		})
	}
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server err : %s\n", err.Error())
	}
}
```

### Next()

> 执行函数

- 定义程序计时中间件，然后定义2个路由，执行函数后应该打印统计的执行时间

```golang
func MiddleWare() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		log.Println("全局中间件开始执行")
		c.Next()
		t2 := time.Since(t)
		log.Println("全局中间件执行耗时：", t2)
	}
}

// 使用全局中间件记录请求的耗时
func main() {
	r := gin.Default()
	r.Use(MiddleWare())
	group := r.Group("/blog")
	{
		group.GET("/list", listHandler)
		group.GET("delete", deleteHandler)
	}
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server err : %s\n", err.Error())
	}
}

func deleteHandler(c *gin.Context) {
	time.Sleep(5 * time.Second)
}

func listHandler(c *gin.Context) {
	time.Sleep(2 * time.Second)
}
```

执行结果

```
[GIN-debug] Listening and serving HTTP on :8000
2022/03/22 17:37:52 全局中间件开始执行
2022/03/22 17:37:54 全局中间件执行耗时： 2.0369246s
[GIN] 2022/03/22 - 17:37:54 | 200 |    2.0369246s |             ::1 | GET      "/blog/list"
2022/03/22 17:38:02 全局中间件开始执行
2022/03/22 17:38:07 全局中间件执行耗时： 5.000646s
[GIN] 2022/03/22 - 17:38:07 | 200 |     5.000646s |             ::1 | GET      "/blog/delete"
```

### 中间件推荐

- [RestGate](https://github.com/pjebs/restgate) - REST API端点的安全身份验证
- [staticbin](https://github.com/olebedev/staticbin) - 用于从二进制数据提供静态文件的中间件/处理程序
- [gin-cors](https://github.com/gin-contrib/cors) - CORS杜松子酒的官方中间件
- [gin-csrf](https://github.com/utrack/gin-csrf) - CSRF保护
- [gin-health](https://github.com/utrack/gin-health) - 通过[gocraft/health](https://github.com/gocraft/health)报告的中间件
- [gin-merry](https://github.com/utrack/gin-merry) - 带有上下文的漂亮 [打印](https://github.com/ansel1/merry) 错误的中间件
- [gin-revision](https://github.com/appleboy/gin-revision-middleware) - 用于Gin框架的修订中间件
- [gin-jwt](https://github.com/appleboy/gin-jwt) - 用于Gin框架的JWT中间件
- [gin-sessions](https://github.com/kimiazhu/ginweb-contrib/tree/master/sessions) - 基于mongodb和mysql的会话中间件
- [gin-location](https://github.com/drone/gin-location) - 用于公开服务器的主机名和方案的中间件
- [gin-nice-recovery](https://github.com/ekyoung/gin-nice-recovery) - 紧急恢复中间件，可让您构建更好的用户体验
- [gin-limit](https://github.com/aviddiviner/gin-limit) - 限制同时请求；可以帮助增加交通流量
- [gin-limit-by-key](https://github.com/yangxikun/gin-limit-by-key) - 一种内存中的中间件，用于通过自定义键和速率限制访问速率。
- [ez-gin-template](https://github.com/michelloworld/ez-gin-template) - gin简单模板包装
- [gin-hydra](https://github.com/janekolszak/gin-hydra) - gin中间件[Hydra](https://github.com/ory-am/hydra)
- [gin-glog](https://github.com/zalando/gin-glog) - 旨在替代Gin的默认日志
- [gin-gomonitor](https://github.com/zalando/gin-gomonitor) - 用于通过Go-Monitor公开指标
- [gin-oauth2](https://github.com/zalando/gin-oauth2) - 用于OAuth2
- [static](https://github.com/hyperboloide/static) gin框架的替代静态资产处理程序。
- [xss-mw](https://github.com/dvwright/xss-mw) - XssMw是一种中间件，旨在从用户提交的输入中“自动删除XSS”
- [gin-helmet](https://github.com/danielkov/gin-helmet) - 简单的安全中间件集合。
- [gin-jwt-session](https://github.com/ScottHuangZL/gin-jwt-session) - 提供JWT / Session / Flash的中间件，易于使用，同时还提供必要的调整选项。也提供样品。
- [gin-template](https://github.com/foolin/gin-template) - 用于gin框架的html / template易于使用。
- [gin-redis-ip-limiter](https://github.com/Salvatore-Giordano/gin-redis-ip-limiter) - 基于IP地址的请求限制器。它可以与redis和滑动窗口机制一起使用。
- [gin-method-override](https://github.com/bu/gin-method-override) - _method受Ruby的同名机架启发而被POST形式参数覆盖的方法
- [gin-access-limit](https://github.com/bu/gin-access-limit) - limit-通过指定允许的源CIDR表示法的访问控制中间件。
- [gin-session](https://github.com/go-session/gin-session) - 用于Gin的Session中间件
- [gin-stats](https://github.com/semihalev/gin-stats) - 轻量级和有用的请求指标中间件
- [gin-statsd](https://github.com/amalfra/gin-statsd) - 向statsd守护进程报告的Gin中间件
- [gin-health-check](https://github.com/RaMin0/gin-health-check) - check-用于Gin的健康检查中间件
- [gin-session-middleware](https://github.com/go-session/gin-session) - 一个有效，安全且易于使用的Go Session库。
- [ginception](https://github.com/kubastick/ginception) - 漂亮的例外页面
- [gin-inspector](https://github.com/fatihkahveci/gin-inspector) - 用于调查http请求的Gin中间件。
- [gin-dump](https://github.com/tpkeeper/gin-dump) - Gin中间件/处理程序，用于转储请求和响应的标头/正文。对调试应用程序非常有帮助。
- [go-gin-prometheus](https://github.com/zsais/go-gin-prometheus) - Gin Prometheus metrics exporter
- [ginprom](https://github.com/chenjiandongx/ginprom) - Gin的Prometheus指标导出器
- [gin-go-metrics](https://github.com/bmc-toolbox/gin-go-metrics) - Gin middleware to gather and store metrics using [rcrowley/go-metrics](https://github.com/rcrowley/go-metrics)
- [ginrpc](https://github.com/xxjwxc/ginrpc) - Gin 中间件/处理器自动绑定工具。通过像beego这样的注释路线来支持对象注册

## 会话控制

### Cookie介绍

- HTTP是无状态协议，服务器不能记录浏览器的访问状态，也就是说服务器不能区分两次请求是否由同一个客户端发出
- Cookie就是解决HTTP协议无状态的方案之一，中文是小甜饼的意思
- Cookie实际上就是服务器保存在浏览器上的一段信息。浏览器有了Cookie之后，每次向服务器发送请求时都会同时将该信息发送给服务器，服务器收到请求后，就可以根据该信息处理请求
- Cookie由服务器创建，并发送给浏览器，最终由浏览器保存

>测试服务端发送cookie给客户端，客户端请求时携带cookie

### Cookie使用

- 模拟权限验证中间件
  - 有2个路由，login和home
  - login用于设置cookie
  - home是访问查看信息的请求
  - 在请求home之前，先跑中间件代码，检验是否存在cookie
- 访问home，会显示错误，因为权限校验未通过

```golang
// User 定义接收数据的结构体
type User struct {
	// binding:"required"修饰的字段，若接收为空值，则报错，是必须字段
	Username string `form:"username" json:"username" uri:"username" xml:"username" binding:"required"`
	Password string `form:"password" json:"password" uri:"password" xml:"password" binding:"required"`
}

func AuthMiddleWare() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 获取Cookie进行验证
		if cookie, err := c.Cookie("user"); err == nil {
			if cookie == "zhangsan" {
				c.Next()
				return
			}
		}
		// 返回错误
		c.JSON(http.StatusBadRequest, gin.H{
			"err": "请先登录",
		})
		// 验证不成功, 不用再调用后续函数
		c.Abort()
		return
	}
}

func main() {
	r := gin.Default()
	r.POST("/login", func(c *gin.Context) {
		var user User
		if err := c.ShouldBindJSON(&user); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"err": err.Error(),
			})
			return
		}
		if user.Username == "zhangsan" && user.Password == "123" {
			// 给客户端设置cookie
			//  maxAge int, 单位为秒
			// path,cookie所在目录
			// domain string,域名
			// secure 是否智能通过https访问
			// httpOnly bool  是否允许别人通过js获取自己的cookie
			c.SetCookie("user", user.Username,
				60, "/", "localhost",
				false, true)
			c.JSON(http.StatusOK, gin.H{
				"msg": "登录成功",
			})
			return
		}
		c.JSON(http.StatusBadRequest, gin.H{
			"msg": "用户名或者密码错误",
		})
	})
	r.GET("/home", AuthMiddleWare(), func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"msg": "data",
		})
	})
	if err := r.Run(":8000"); err != nil {
		fmt.Printf("startup server failed err : %s\n", err.Error())
	}
}
```

> Cookie缺点

- 不安全，明文
- 增加带宽消耗
- 可以被禁用
- cookie有上限

### Session

gorilla/sessions为自定义session后端提供cookie和文件系统session以及基础结构。

主要功能是：

- 简单的API：将其用作设置签名（以及可选的加密）cookie的简便方法。
- 内置的后端可将session存储在cookie或文件系统中。
- Flash消息：一直持续读取的session值。
- 切换session持久性（又称“记住我”）和设置其他属性的便捷方法。
- 旋转身份验证和加密密钥的机制。
- 每个请求有多个session，即使使用不同的后端也是如此。
- 自定义session后端的接口和基础结构：可以使用通用API检索并批量保存来自不同商店的session。

> 获取依赖

```bash
go get -u github.com/gorilla/sessions
```



```go
// 初始化一个cookie存储对象
// something-very-secret应该是一个你自己的密匙，只要不被别人知道就行
var store = sessions.NewCookieStore([]byte("something-very-secret"))

func main() {
	http.HandleFunc("/save", SaveSession)
	http.HandleFunc("/get", GetSession)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("HTTP server failed,err:", err)
		return
	}
}

func SaveSession(w http.ResponseWriter, r *http.Request) {
	// Get a session. We're ignoring the error resulted from decoding an
	// existing session: Get() always returns a session, even if empty.

	//　获取一个session对象，session-name是session的名字
	session, err := store.Get(r, "session-name")
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	// 在session中存储值
	session.Values["foo"] = "bar"
	session.Values[42] = 43
	// 保存更改
	session.Save(r, w)
}
func GetSession(w http.ResponseWriter, r *http.Request) {
	session, err := store.Get(r, "session-name")
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	foo := session.Values["foo"]
	fmt.Println(foo)
}
```

## 参数验证

### 结构体验证

用gin框架的数据验证，可以不用解析数据，减少if else，会简洁许多。

```go
type Person struct {
	//不能为空并且大于10
	Age      int       `form:"age" binding:"required,gt=10"`
	Name     string    `form:"name" binding:"required"`
	Birthday time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
}

func main() {
	r := gin.Default()
	r.GET("/person", func(c *gin.Context) {
		var person Person
		if err := c.ShouldBind(&person); err != nil {
			c.String(500, fmt.Sprint(err))
			return
		}
		c.String(200, fmt.Sprintf("%#v", person))
	})
	r.Run()
}
```

### 自定义验证

```golang
/*
    对绑定解析到结构体上的参数，自定义验证功能
    比如我们要对 name 字段做校验，要不能为空，并且不等于 admin ，类似这种需求，就无法 binding 现成的方法
    需要我们自己验证方法才能实现 官网示例（https://godoc.org/gopkg.in/go-playground/validator.v8#hdr-Custom_Functions）
    这里需要下载引入下 gopkg.in/go-playground/validator.v8
*/
type Person struct {
    Age int `form:"age" binding:"required,gt=10"`
    // 2、在参数 binding 上使用自定义的校验方法函数注册时候的名称
    Name    string `form:"name" binding:"NotNullAndAdmin"`
    Address string `form:"address" binding:"required"`
}

// 1、自定义的校验方法
func nameNotNullAndAdmin(v *validator.Validate, topStruct reflect.Value, currentStructOrField reflect.Value, field reflect.Value, fieldType reflect.Type, fieldKind reflect.Kind, param string) bool {

    if value, ok := field.Interface().(string); ok {
        // 字段不能为空，并且不等于  admin
        return value != "" && !("5lmh" == value)
    }

    return true
}

func main() {
    r := gin.Default()

    // 3、将我们自定义的校验方法注册到 validator中
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        // 这里的 key 和 fn 可以不一样最终在 struct 使用的是 key
        v.RegisterValidation("NotNullAndAdmin", nameNotNullAndAdmin)
    }

    /*
        curl -X GET "http://127.0.0.1:8080/testing?name=&age=12&address=beijing"
        curl -X GET "http://127.0.0.1:8080/testing?name=lmh&age=12&address=beijing"
        curl -X GET "http://127.0.0.1:8080/testing?name=adz&age=12&address=beijing"
    */
    r.GET("/5lmh", func(c *gin.Context) {
        var person Person
        if e := c.ShouldBind(&person); e == nil {
            c.String(http.StatusOK, "%v", person)
        } else {
            c.String(http.StatusOK, "person bind err:%v", e.Error())
        }
    })
    r.Run()
}
```

# GORM

## 简介

> ORM

**对象关系映射**（英语：**Object Relational Mapping**，简称**ORM**，或**O/RM**，或**O/R mapping**），是一种程序设计技术，用于实现[面向对象](https://baike.baidu.com/item/面向对象)编程语言里不同[类型系统](https://baike.baidu.com/item/类型系统/4273825)的[数据](https://baike.baidu.com/item/数据/5947370)之间的转换。从效果上说，它其实是创建了一个可在[编程语言](https://baike.baidu.com/item/编程语言/9845131)里使用的“虚拟对象数据库”。

GORM 是优秀的 Golang ORM 类库

### 概览

- 全特性 ORM (几乎包含所有特性)
- 模型关联 (一对一， 一对多，一对多（反向）， 多对多， 多态关联)
- 钩子 (Before/After Create/Save/Update/Delete/Find)
- 预加载
- 事务
- 复合主键
- SQL 构造器
- 自动迁移
- 日志
- 基于GORM回调编写可扩展插件
- 全特性测试覆盖

## 快速开始

> 安装

```bash
go get -u github.com/jinzhu/gorm
```



