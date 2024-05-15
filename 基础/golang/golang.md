# 基础知识

## 工作区和GOPATH

不同于其他语言，go中没有项目的说法，**只有包**, 其中有两个重要的路径，GOROOT 和 GOPATH

- GOROOT：GOROOT就是GO的安装目录，类似Java的JDK
- GOPATH：GOPATH就是工作空间，保存go项目代码和第三方依赖包

**`GOPATH`**可以设置多个，其中，第一个将会是默认的包目录，使用 go get 下载的包都会在第一个path中的src目录下，使用 go install时，在哪个**`GOPATH`**中找到了这个包，就会在哪个`GOPATH`下的bin目录生成可执行文件

GOPATH是开发时的工作目录。用于：

- 保存编译后的二进制文件。

- go get和go install命令会下载go代码到GOPATH。
- import包时的搜索路径

使用GOPATH时，GO会在以下目录中搜索包：

- GOROOT/src：该目录保存了Go标准库代码。
- GOPATH/src：该目录保存了应用自身的代码和第三方依赖的代码。

### Go语言源码组织方式

Go语言源码的组织方式就是以环境变量GOPATH、工作区、src目录和代码为主线。一般情况下，Go语言的源码文件都需要被存放在环境变量GOPATH包含的某个工作区（目录）中的src目录下的某个代码包（目录）中

### 代码安装后的结果

```go
go install github.com/labstack/echo
```

归档文件的相对目录与 pkg 目录之间还有一级目录，叫做平台相关目录。平台相关目录的名称是由 build（也称“构建”）的目标操作系统、下划线和目标计算架构的代号组成的。

比如，构建某个代码包时的目标操作系统是 Linux，目标计算架构是 64 位的，那么对应的平台相关目录就是 linux_amd64。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/202301291926104.png)



[go get](https://github.com/hyper0x/go_command_tutorial/blob/master/0.3.md)

Go 语言官方提供的go get命令是比较基础的，其中并没有提供依赖管理的功能。目前 GitHub 上有很多提供这类功能的第三方工具，比如glide、gb以及官方出品的dep、vgo等等，它们在内部大都会直接使用go get。

问题

> Go 语言在多个工作区中查找依赖包的时候是以怎样的顺序进行的？ 

先GOROOT，后GOPATH

> 如果在多个工作区中都存在导入路径相同的代码包会产生冲突吗？

不会

## 基础数据类型

### 数组和切片

它们的共同点是都属于集合类的类型，并且，它们的值也都可以用来存储某一种类型的值（或者说元素）。

不过，它们最重要的不同是：数组类型的值（简称数组）的长度是固定的，而切片类型的值（简称切片）是可变长的。

声明的区别，数组长度在声明的时候必须给定，并且之后不会再改变，可以说，数组的长度是其类型的一部分；而切片的类型字面量中只有元素的类型，而没有长度。切片的长度可以自动地随着其中元素的增长而增长，但不会随着元素数量的减少而减少

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/202301291933680.png)

可以将切片看做是对数组的一层简单封装，因为每个切片底层数据结构中，一定会包含一个数组。数组可以被叫做切片的底层数组，而切片也可以被看做是对数组某个连续片段的引用

>也正因为如此，Go 语言的切片类型属于引用类型，同属引用类型的还有字典类型、通道类型、函数类型等；而 Go 语言的数组类型则属于值类型，同属值类型的有基础数据类型以及结构体类型。
>
>注意，Go 语言里不存在像 Java 等编程语言中令人困惑的“传值或传引用”问题。在 Go 语言中，我们判断所谓的“传值”或者“传引用”只要看被传递的值的类型就好了。 如果传递的值是引用类型的，那么就是“传引用”。
>
>如果传递的值是值类型的，那么就是“传值”。从传递成本的角度讲，引用类型的值往往要比值类型的值低很多。 我们在数组和切片之上都可以应用索引表达式，得到的都会是某个元素。我们在它们之上也都可以应用切片表达式，也都会得到一个新的切片。



问题

> 怎么快速计算截断切片的长度和容量？

举例

```go
s3 := []int{1, 2, 3, 4, 5, 6, 7, 8} 
s4 := s3[3:6]
```

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/202301291947818.png)

s4切片的长度：6 - 3 = 3

s4切片的容量： 8 - 3 = 5

> 怎么估算切片容量的增长？

一旦一个切片无法容纳更多的元素，Go 语言就会想办法扩容。但它并不会改变原来的切片，而是会生成一个容量更大的切片，然后将把原有的元素和新元素一并拷贝到新切片中。在一般的情况下，你可以简单地认为新切片的容量（以下简称新容量）将会是原切片容量（以下简称原容量）的 2 倍。

但是，当原切片的长度（以下简称原长度）大于或等于1024时，Go 语言将会以原容量的1.25倍作为新容量的基准（以下新容量基准）。新容量基准会被调整（不断地与1.25相乘），直到结果不小于原长度与要追加的元素数量之和（以下简称新长度）。最终，新容量往往会比新长度大一些，当然，相等也是可能的。 

另外，如果我们一次追加的元素过多，以至于使新长度比原容量的 2 倍还要大，那么新容量就会以新长度为基准。注意，与前面那种情况一样，最终的新容量在很多时候都要比新容量基准更大一些。更多细节可参见runtime包中 slice.go 文件里的growslice及相关函数的具体实现。

> 切片的底层数组什么时候被替换？

确切的说，一个切片的底层数组永远不会被替换，虽然在扩容的时候Go语言一定会生成新的底层数组，但是它同时也会生成新的切片。它只是把新的切片作为了新底层数组的窗口，而没有对原切片，及其底层数组做变动

请记住，在无需扩容时，append函数返回的是指向原底层数组的原切片，而在需要扩容时，append函数返回的是指向新底层数组的新切片。所以，严格来讲，“扩容”这个词用在这里虽然形象但并不合适。不过鉴于这种称呼已经用得很广泛了，我们也没必要另找新词了。

> 如果多个切片指向了同一个底层数组，那么应该注意什么？



> 怎么沿用“扩容”的思想对切片进行“缩容”？



> 清空slice的两种方式

如果要清空一个slice，那么可以简单的赋值为nil，垃圾回收期会自动回收原有的数据

```go
a := [1,2,3]
a = nil
fmt.Println(a, len(a), cap(a) // [] 0 0
```

nil slice 和普通 slice一样可以使用 cap len 内置函数，以及被 for range 遍历。本质和 empty slice 性质一样，零长度和零容量，当然也可以使用 append 操作。

但是如果还需要使用 slice 底层内存，那么最佳的方式是 **re-slice**

```go
a := [1,2,3]
a = a[:0]
fmt.Println(a, len(a), cap(a) // [] 0 3
fmt.Println(a[:1]) // [1]
```

不过如果序列化成 json 时候，上述二者就不太相同了，nil slice 会编码成 `null`，而 empty slice 会编码成 `[]`。

### container包下的容器

#### heap

堆

#### List

List包含的方法中，用于插入新元素的那些方法只接受`interface{}`类型的值。这些方法在内部会使用`Element`值，包装接受到的新元素。这样做正是为了避免直接使用我们自己生成的元素，主要原因是避免链表的内部关联，遭到外界破坏，这对于链表本身以及我们这些使用者来说都是有益的。

在这些方法中，“给定的元素”都是Element类型的，Element类型是Element类型的指针类型，Element的值就是元素的指针。

```go
func (l *List) MoveBefore(e, mark *Element)
func (l *List) MoveAfter(e, mark *Element)

func (l *List) MoveToFront(e *Element)
func (l *List) MoveToBack(e *Element)
```

具体问题是，如果我们自己生成这样的值，然后把它作为“给定的元素”传给链表的方法，那么会发生什么？链表会接受它吗？ 

这里，给出一个典型回答：不会接受，这些方法将不会对链表做出任何改动。因为我们自己生成的Element值并不在链表中，所以也就谈不上“在链表中移动元素”。更何况链表不允许我们把自己生成的Element值插入其中。

Front和Back方法分别用于获取链表中最前端和最后端的元素

InsertBefore和InsertAfter方法分别用于在指定的元素之前和之后插入新元素，PushFront和PushBack方法则分别用于在链表的最前端和最后端插入新元素。

```go
func (l *List) Front() *Element
func (l *List) Back() *Element

func (l *List) InsertBefore(v interface{}, mark *Element) *Element
func (l *List) InsertAfter(v interface{}, mark *Element) *Element

func (l *List) PushFront(v interface{}) *Element
func (l *List) PushBack(v interface{}) *Element
```

这些方法都会把一个Element值的指针作为结果返回，它们就是链表留给我们的安全“接口”。拿到这些内部元素的指针，我们就可以去调用前面提到的用于移动元素的方法了。

#### Ring

### 字典

### 通道

#### 基本操作

#### 高级玩法

### 函数

### 结构体

struct{}{} 它占用空间字节0字节，整个Go程序中永远只有一份。无数次使用，都用到的是一个值

https://juejin.cn/post/7120219197799563278

#### 结构体互相嵌套

```go
// 结构体互相嵌套

type Address struct {
	country  string
	province string
	// ...
}

// 匿名字段
type Person1 struct {
	name string
	Address
}

// 非匿名字段
type Person2 struct {
	name    string
	address Address
}

func main() {
	person1 := Person1{"zhangsan", Address{"cn", "sx"}}
	fmt.Println(person1.province)

	person2 := Person2{"zhangsan", Address{"cn", "sx"}}
	fmt.Println(person2.address.province)
}
```

#### 接口互相嵌套

Go 语言团队鼓励我们声明体量较小的接口，并建议我们通过这种接口间的组合来扩展程序、增加程序的灵活性。

这是因为相比于包含很多方法的大接口而言，小接口可以更加专注地表达某一种能力或某一类特征，同时也更容易被组合在一起

Go 语言标准库代码包io中的ReadWriteCloser接口和ReadWriter接口就是这样的例子，它们都是由若干个小接口组合而成的。以io.ReadWriteCloser接口为例，它是由io.Reader、io.Writer和io.Closer这三个接口组成的。

它们中的每一个都只代表了一种能力

```go
// 接口互相嵌套

type Name interface {
	WriteName()
}

type Address interface {
	WriteAddress()
}

type Person interface {
	Name
	Address
	Hello()
}

type person struct {
	name    string
	address string
}

func (p person) WriteName() {
	fmt.Println(p.name)
}

func (p person) WriteAddress() {
	fmt.Println(p.address)
}

func (p person) Hello() {
	fmt.Println("hello")
}

func main() {
	p := person{"zs", "cn"}
	p.Hello()
	p.WriteName()
	p.WriteAddress()
}
```

#### 结构体内嵌接口

```go
// 结构体内嵌接口

type HelloWorld interface {
	Hello()
}

// 匿名字段
type Person1 struct {
	name string
	HelloWorld
}

// 非匿名字段
type Person2 struct {
	name       string
	helloWorld HelloWorld
}

type Hello struct {
	name string
}

func (h Hello) Hello() {
	fmt.Println("hello", h.name)
}

func main() {
	person1 := Person1{"zs", Hello{"!"}}
	person1.Hello()

	person2 := Person2{"ls", Hello{"?"}}
	person2.helloWorld.Hello()
}

```

#### 值方法和指针方法

一个自定义数据类型的方法集合中仅会包含它的所有值方法，而该类型的指针类型的方法集合却囊括了前者的所有方法，包括所有值方法和所有指针方法。

严格来讲，我们在这样的基本类型的值上只能调用到它的值方法。但是，Go 语言会适时地为我们进行自动地转译，使得我们在这样的值上也能调用到它的指针方法。

- 指针类型可以调用值方法和指针方法，调用值方法都时候，会进行值复制
- 值类型可以调用值方法和指针方法，调用指针方法的时候， 会默认加上(&cat).指针方法

```go
package main

import (
	"fmt"
)

type Cat struct {
	name           string // 名字。
	scientificName string // 学名。
	category       string // 动物学基本分类。
}

func New(name, scientificName, category string) Cat {
	return Cat{
		name:           name,
		scientificName: scientificName,
		category:       category,
	}
}

func (cat *Cat) SetName(name string) {
	cat.name = name
}

func (cat Cat) SetNameOfCopy(name string) {
	cat.name = name
}

func (cat Cat) Name() string {
	return cat.name
}

func (cat Cat) ScientificName() string {
	return cat.scientificName
}

func (cat Cat) Category() string {
	return cat.category
}

func (cat Cat) String() string {
	return fmt.Sprintf("%s (category: %s, name: %q)",
		cat.scientificName, cat.category, cat.name)
}

func main() {
	c := &Cat{name: "1"}
	c.SetNameOfCopy("a")
	fmt.Println(c)
	cat := New("little pig", "American Shorthair", "cat")
	cat.SetName("monster") // (&cat).SetName("monster")
	fmt.Printf("The cat: %s\n", cat)

	cat.SetNameOfCopy("little pig")
	fmt.Printf("The cat: %s\n", cat)

	type Pet interface {
		SetName(name string)
		Name() string
		Category() string
		ScientificName() string
	}

	_, ok := interface{}(cat).(Pet)
	fmt.Printf("Cat implements interface Pet: %v\n", ok)
	_, ok = interface{}(&cat).(Pet)
	fmt.Printf("*Cat implements interface Pet: %v\n", ok)
}
```

比如，一个指针类型实现了某某接口类型，但它的基本类型却不一定能够作为该接口的实现类型。



> struct{}代表是一个空结构体，不占用内存空间，但是也是一个结构体！

### 接口

怎样判定一个数据类型的某一个方法实现的就是某个接口类型中的某个方法呢？

这有两个充分必要条件，一个是“两个方法的签名需要完全一致”，另一个是“两个方法的名称要一模一样”。显然，这比判断一个函数是否实现了某个函数类型要更加严格一些（只要函数签名一致即可）。

我声明的类型Dog附带了 3 个方法。其中有 2 个值方法，分别是Name和Category，另外还有一个指针方法SetName。

这就意味着，Dog类型本身的方法集合中只包含了 2 个方法，也就是所有的值方法。而它的指针类型*Dog方法集合却包含了 3 个方法。

而这三个指针方法都是Pet接口中某个方法的实现，所以*Dog就成为了Pet接口的实现类型

```go
type Pet interface {
	SetName(name string)
	Name() string
	Category() string
}

type Dog struct {
	name string
}

func (d *Dog) SetName(name string) {
	d.name = name
}

func (d Dog) Name() string {
	return d.name
}

func (d Dog) Category() string {
	return "dog"
}
```

对于一个接口类型的变量来说，例如上面的变量pet，我们赋给它的值可以被叫做它的实际值（也称动态值），而该值的类型可以被叫做这个变量的实际类型（也称动态类型）。

动态类型这个叫法是相对于静态类型而言的。对于变量pet来讲，它的静态类型就是Pet，并且永远是Pet，但是它的动态类型却会随着我们赋给它的动态值而变化。

```go
func main() {
	var pet Pet
	d := &Dog{name: "xq"}
	pet = d
	fmt.Printf("%T, %v", pet, pet)
}
```

Pet 接口中删除SetName方法，Dog也是Pet接口的实现类型

```go
func main() {
	var pet Pet
	d := Dog{name: "xq"}
	pet = d
	d.SetName("wwww")
	fmt.Printf("%T, %v", pet, pet)
}

main.Dog, {xq}
```

修改失败的原因：

接口类型变量是一种特殊的数据接口`iface`，`iface`的实例会包含两个指针，一个是指向类型信息的指针，另一个是指向动态值的指针。这里的类型信息是由另一个专用数据结构的实例承载的，其中包含了动态值的类型，以及使它实现了接口的方法和调用它们的途径，等等

```go
type emptyInterface struct {
	typ  *rtype
	word unsafe.Pointer
}
```

接口变量值什么时候为nil？

```go
func main() {
	var pet Pet
	var dog Dog
	pet = dog
	fmt.Printf("%T, %v", pet, pet)
	if pet == nil {
		fmt.Printf("pet is nil")
	}
}

main.Dog, {}
```

只有当把nil直接赋值给pet时

> 用好小接口和接口组合总是有益的，我们可以以此形成接口矩阵，进而搭起灵活的程序框架。如果在实现接口时再配合运用结构体类型间的嵌入手法，那么接口组合就可以发挥更大的效用。

如果我们把一个值为nil的某个实现类型的变量赋给了接口变量，那么在这个接口变量上仍然可以调用该接口的方法吗？如果可以，有哪些注意事项？如果不可以，原因是什么？

可以，实现类型赋值给接口变量时，会默认为该类型的零值，特别需要注意指针类型（无论实现类型本身，还是实现类型的部分字段），零值为nil，会触发空指针panic

### 指针

`&`寻址符号

寻址规则

- 不可变的值不可寻址。常量、基本类型的值字面量、字符串变量的值、函数以及方法的字面量都是如此。其实这样规定也有安全性方面的考虑。
- 绝大多数被视为临时结果的值都是不可寻址的。算术操作的结果值属于临时结果，针对值字面量的表达式结果值也属于临时结果。但有一个例外，对切片字面量的索引结果值虽然也属于临时结果，但却是可寻址的。
- 若拿到某值的指针可能会破坏程序的一致性，那么就是不安全的，该值就不可寻址。由于字典的内部机制，对字典的索引结果值的取址操作都是不安全的。另外，获取由字面量或标识符代表的函数或方法的地址显然也是不安全的。

```go
package main

type Named interface {
	// Name 用于获取名字。
	Name() string
}

type Dog struct {
	name string
}

func (dog *Dog) SetName(name string) {
	dog.name = name
}

func (dog Dog) Name() string {
	return dog.name
}

func main() {
	// 示例1。
	const num = 123
	//_ = &num // 常量不可寻址。
	//_ = &(123) // 基本类型值的字面量不可寻址。

	var str = "abc"
	_ = str
	//_ = &(str[0]) // 对字符串变量的索引结果值不可寻址。
	//_ = &(str[0:2]) // 对字符串变量的切片结果值不可寻址。
	str2 := str[0]
	_ = &str2 // 但这样的寻址就是合法的。

	//_ = &(123 + 456) // 算术操作的结果值不可寻址。
	num2 := 456
	_ = num2
	//_ = &(num + num2) // 算术操作的结果值不可寻址。

	//_ = &([3]int{1, 2, 3}[0]) // 对数组字面量的索引结果值不可寻址。
	//_ = &([3]int{1, 2, 3}[0:2]) // 对数组字面量的切片结果值不可寻址。
	_ = &([]int{1, 2, 3}[0]) // 对切片字面量的索引结果值却是可寻址的。
	//_ = &([]int{1, 2, 3}[0:2]) // 对切片字面量的切片结果值不可寻址。
	//_ = &(map[int]string{1: "a"}[0]) // 对字典字面量的索引结果值不可寻址。

	var map1 = map[int]string{1: "a", 2: "b", 3: "c"}
	_ = map1
	//_ = &(map1[2]) // 对字典变量的索引结果值不可寻址。

	//_ = &(func(x, y int) int {
	//	return x + y
	//}) // 字面量代表的函数不可寻址。
	//_ = &(fmt.Sprintf) // 标识符代表的函数不可寻址。
	//_ = &(fmt.Sprintln("abc")) // 对函数的调用结果值不可寻址。

	dog := Dog{"little pig"}
	_ = dog
	//_ = &(dog.Name) // 标识符代表的函数不可寻址。
	//_ = &(dog.Name()) // 对方法的调用结果值不可寻址。

	//_ = &(Dog{"little pig"}.name) // 结构体字面量的字段不可寻址。

	//_ = &(interface{}(dog)) // 类型转换表达式的结果值不可寻址。
	dogI := interface{}(dog)
	_ = dogI
	//_ = &(dogI.(Named)) // 类型断言表达式的结果值不可寻址。
	named := dogI.(Named)
	_ = named
	//_ = &(named.(Dog)) // 类型断言表达式的结果值不可寻址。

	var chan1 = make(chan int, 1)
	chan1 <- 1
	//_ = &(<-chan1) // 接收表达式的结果值不可寻址。

}

```

 New 返回的是一个临时结果，setName是一个指针方法，会默认对New的结果进行取址，

```go
type Dog struct {
	name string
}

func (dog *Dog) SetName(name string) {
	dog.name = name
}

func New(name string) Dog {
	return Dog{name: name}
}

func main() {
	New("xq").SetName("monster")
}
```

上边这行链式调用会让编译器报告两个错误，一个是果，即：不能在New("xq")的结果值上调用指针方法。一个是因，即：不能取得New("xq")的地址。

## go语句执行规则

> Don’t communicate by sharing memory; share memory by communicating. 

不要通过共享数据来通讯，恰恰相反，要以通讯的方式共享数据。



GMP



### 流程控制

range表达式只会在for语句开始执行时被求值一次，无论后边会有多少次迭代； 

range表达式的求值结果会被复制，也就是说，被迭代的对象是range表达式结果值的副本而不是原值。

```go
numbers1 := []int{1, 2, 3, 4, 5, 6}
for i := range numbers1 {
  if i == 3 {
    numbers1[i] |= i
  }
}
fmt.Println(numbers1)

[1 2 3 7 5 6]
```



```go
numbers2 := [...]int{1, 2, 3, 4, 5, 6}
maxIndex2 := len(numbers2) - 1
for i, e := range numbers2 {
  if i == maxIndex2 {
    numbers2[0] += e
  } else {
    numbers2[i+1] += e
  }
}
fmt.Println(numbers2)

[7 3 5 7 9 11]
```

## 错误处理

```go
type error interface {
	Error() string
}
```

怎样判断error具体代表哪一类错误？

1. 对于类型在已知范围内的一系列错误值，一般使用类型断言表达式或类型 switch 语句来判断； 
2. 对于已有相应变量且类型相同的一系列错误值，一般直接使用判等操作来判断； 
3. 对于没有相应变量且类型未知的一系列错误值，只能使用其错误信息的字符串表示形式来做判断。

### defer、recover、panic



## go测试

Go 程序编写三类测试，即：功能测试（test）、基准测试（benchmark，也称性能测试），以及示例测试（example）。

> 测试源码文件的主名称应该以被测源码文件的主名称为前导，并且必须以“_test”为后缀

- 对于功能测试函数来说，其名称必须以 Test 为前缀，并且参数列表中只应有一个*testing.T* 类型的参数声明。 
- 对于性能测试函数来说，其名称必须以 Benchmark 为前缀，并且唯一参数的类型必须是*testing.B* 类型的。 
- 对于示例测试函数来说，其名称必须以 Example 为前缀，但对函数的参数列表没有强制规定。

多个测试用例

```go
func expirationFunc(expiration time.Duration) time.Duration {
	if expiration == 0 {
		randNum := randGenerator.Int63()
		return 24*time.Hour + time.Duration(randNum%(int64(12*time.Hour)))
	}

	return expiration
}

func Test_expirationFunc(t *testing.T) {
	type args struct {
		expiration time.Duration
	}
	tests := []struct {
		name string
		args args
	}{
		// TODO: Add test cases.
		{
			name: "test1",
			args: args{},
		},
		{
			name: "test2",
			args: args{},
		},
		{
			name: "test3",
			args: args{},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := expirationFunc(tt.args.expiration); !(24*time.Hour <= got && got <= 36*time.Hour) {
				t.Errorf("expirationFunc() = %v", got)
			}
		})
	}
}
```

### go test命令

> 单元测试的意思就是：对单一的功能模块进行边界清晰的测试，并且不掺杂任何对外部环境的检测。这也是“单元”二字要表达的主要含

https://deepzz.com/post/the-command-flag-of-go-test.html

### 单元测试

```go
功能测试
// 打印日志
go test -v
// flag
go test -args -flagname=val
// 指定函数
go test -v  -run  测试函数名字
```

### 性能测试(基准测试)

> 如果测试函数的执行时间没有超过上限，此上限默认为 1 秒，那么命令就会改大b.N的值，然后再次执行测试函数，如此往复，直到这个时间大于或等于上限为止。

```go
go test -bench=. -run ^$ ./ -v
// 第一个标记及其值为-bench=.，只有有了这个标记，命令才会进行性能测试。该标记的值.表明需要执行任意名称的性能测试函数，当然了，函数名称还是要符合 Go 程序测试的基本规则的
// 第一个标记及其值为-bench=.，只有有了这个标记，命令才会进行性能测试。该标记的值.表明需要执行任意名称的性能测试函数，当然了，函数名称还是要符合 Go 程序测试的基本规则的
// 如果运行go test命令的时候不加-run标记，那么就会使它执行被测代码包中的所有功能测试函数。
// 指定cpu核心数 P （processor）
-cpu
// 重复执行测试函数
-count

// -benchmem标记和-benchtime标记的作用分别是什么？
- -benchmem 输出基准测试的内存分配统计信息。 
- -benchtime 用于指定基准测试的探索式测试执行时间上限
```

BenchmarkGetPrimes-8被称为单个性能测试的名称，它表示命令执行了性能测试函数BenchmarkGetPrimes，并且当时所用的最大 P 数量为8。

最大 P 数量相当于可以同时运行 goroutine 的逻辑 CPU 的最大个数。这里的逻辑 CPU，也可以被称为 CPU 核心，但它并不等同于计算机中真正的 CPU 核心，只是 Go 语言运行时系统内部的一个概念，代表着它同时运行 goroutine 的能力。

可以通过调用 runtime.GOMAXPROCS函数改变最大 P 数量，也可以在运行go test命令时，加入标记-cpu来设置一个最大 P 数量的列表，以供命令在多次测试时使用。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/202302201648780.png)

### 执行次数计算

```go
性能测试函数的执行次数 = `-cpu`标记的值中正整数的个数 x `-count`标记的值 x 探索式执行中测试函数的实际执行次数
功能测试函数的执行次数 = `-cpu`标记的值中正整数的个数 x `-count`标记的值
```



![img](https://gitee.com/zhang-songyao/blog-images/raw/master/202302201710963.png)只有测试源码文件的名称对了，测试函数的名称和签名也对了，当我们运行go test命令的时候，其中的测试代码才有可能被运行。

### 执行步骤

go test命令在开始运行时，会先做一些准备工作，比如，确定内部需要用到的命令，检查我们指定的代码包或源码文件的有效性，以及判断我们给予的标记是否合法，等等。

每个代码包中串行执行，多个代码包并行执行。

性能测试串行执行，打印结果串行打印



### 并发性能测试

我们在运行go test命令的时候，可以追加标记-parallel，该标记的作用是：设置同一个被测代码包中的功能测试函数的最大并发执行数。该标记的默认值是测试运行时的最大 P 数量（这可以通过调用表达式runtime.GOMAXPROCS(0)获得）

-parallel标记对性能测试是无效的。当然了，对于性能测试来说，也是可以并发进行的，不过机制上会有所不同。

如果基准测试需要在并行设置中测试性能，它可以使用 RunParallel 辅助函数；此类基准测试旨在与 go test -cpu 标志一起使用：

```go
func BenchmarkTemplateParallel(b *testing.B) {
    templ := template.Must(template.New("test").Parse("Hello, {{.}}!"))
    b.RunParallel(func(pb *testing.PB) {
        var buf bytes.Buffer
        对于 pb.Next() {
            buf.Reset()
            templ.Execute(&buf, "世界")
        }
    })
}
```

### 性能测试计时器

`testing.B`类型有这么几个指针方法：`StartTimer`、`StopTimer`和`ResetTimer`。这些方法都是用于操作当前的性能测试函数专属的计时器。

所谓的计时器，是一个逻辑上的概念，它其实是`testing.B`类型中一些字段的统称。这些字段用于记录：当前测试函数在当次执行过程中耗费的时间、分配的堆内存的字节数以及分配次数。

实际上，`go test`命令本身就会用到这样的计时器。当准备执行某个性能测试函数的时候，命令会重置并启动改函数专属的计时器，一旦这个函数执行完毕，命令又会立即停止这个计时器。

在性能测试函数中，我们可以通过对`b.StartTimer`和`b.StopTimer`方法的联合运用，再去除掉任何一段代码的执行时间；

相比之下，b.ResetTimer方法的灵活性就要差一些了，它只能用于：去除在调用它之前那些代码的执行时间。不过，无论在调用它的时候，计时器是不是正在运行，它都可以起作用。



## 核心代码包

### sync.Mutex & sync.RWMutex

Go语言宣扬==用通讯的方式共享数据==

竞态条件：一旦数据被多个线程共享，那么很有可能产生争用和冲突的情况，这种情况被称为竞态条件(race condition)，往往会破坏共享数据的一致性。

共享数据的一致性代表着某种约束：多个线程对共享数据的操作总是可以达到他们各自预期的效果，如果一致性得不到保证，那么会影响到一些线程中的代码和流程的正确性。

同步的作用

- 避免多个线程在同一时刻操作同一个数据块
- 协调多个线程，避免同一时刻操作同一个数据块

在 Go 语言中，可供我们选择的同步工具并不少。其中，最重要且最常用的同步工具当属互斥量（mutual exclusion，简称 mutex）。sync包中的Mutex就是与其对应的类型，该类型的值可以被称为互斥量或者互斥锁。



### sync.Cond



## golang反射性能分析

https://colobu.com/2019/01/29/go-reflect-performance/
