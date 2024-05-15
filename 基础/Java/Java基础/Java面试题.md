

# 面向对象

## 面向对象和面向过程的区别

- 面向过程：具体化的，流程化的，解决一个问题，你需要一步一步分析，一步一步实现
  - 优点：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源；比如单片机、嵌入式开发、Linux/Unix等一般采用面向过程开发，性能是最重要的因素
  - 缺点：没有面向对象易维护、易复用、易扩展
- 面向对象：模型化的，你只需要抽出一个类，这是一个封闭的盒子，在这里你拥有数据也拥有解决问题的方法。需要什么功能直接使用就可以了，不必去一步一步的实现，至于这个功能是如何实现的
  - 优点：：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统更加灵活、更加易于维护
  - 缺点：性能比面向过程低

## 三大特性

抽象：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面，抽象只关注对象有哪些属性和行为，并不关注这些行为的细节

### 封装

是对象功能内聚的表现形式，在抽象基础上决定信息是否公开及公开等级，核心问题是以什么方式暴漏哪些信息。主要任务是对属性、数据、敏感行为实现隐藏，对属性的访问和修改必须通过公共接口实现。封装使对象关系变得简单，降低了代码耦合度，方便维护

迪米特原则就是对封装的要求，即 A 模块使用 B 模块的某接口行为，对 B 模块中除此行为外的其他信息知道得应尽可能少。不直接对 public 属性进行读取和修改而使用 getter/setter 方法是因为假设想在修改属性时进行权限控制、日志记录等操作，在直接访问属性的情况下无法实现。如果将 public 的属性和行为修改为 private 一般依赖模块都会报错，因此不知道使用哪种权限时应优先使用 private

### 继承

用来扩展一个类，子类可继承父类的部分属性和行为使模块具有服用性，继承是“is-a”的关系，可使用里氏替换原则判断是够满足“is-a”的关系，即任何父类出现的地方子类都可以出现。。如果父类引用直接使用子类引用来代替且可以正确编译并执行，输出结果符合子类场景预期，那么说明两个类符合里氏替换原则

### 多态

以封装和继承为基础，根据运行时对象实际类型不同的行为具有不同表现形式，多态指在编译层面无法确定最终调用的方法体，在运行期由JVM动态绑定，适合合适的重写方法，由于重载属于静态绑定，本质上重载结果是完全不同的方法，因此多态一般专指重写

## 重载与重写

### 重载

**重载**指方法名称相同，但参数类型个数不同，是行为水平方向不同实现。对编译器来说，方法名称和参数列表组成了一个唯一键，称为方法签名，JVM 通过方法签名决定调用哪种重载方法。不管继承关系如何复杂，重载在编译时可以根据规则知道调用哪种目标方法，因此属于静态绑定

JVM 在重载方法中选择合适方法的顺序：① 精确匹配。② 基本数据类型自动转换成更大表示范围。③ 自动拆箱与装箱。④ 子类向上转型。⑤ 可变参数

### 重写

**重写**指子类实现接口或继承父类时，保持方法签名完全相同，实现不同方法体，是行为垂直方向不同实现

元空间有一个方法表保存方法信息，如果子类重写了父类的方法，则方法表中的方法引用会指向子类实现。父类引用执行子类方法时无法调用子类存在而父类不存在的方法。

重写方法访问权限不能变小，返回类型和抛出的异常类型不能变大，必须加 `@Override`

## 类之间的关系

| 类关系 |              描述              | 权力强侧 |               举例               |
| :----: | :----------------------------: | :------: | :------------------------------: |
|  继承  |     父子类之间的关系：is-a     |   父类   |          小狗继承于动物          |
|  实现  | 接口和实现类之间的关系：can-do |   接口   |        小狗实现了狗叫接口        |
|  组合  |  比聚合更强的关系：contains-a  |   整体   |         头是身体的一部分         |
|  聚合  |     暂时组装的关系：has-a      |  组装方  |    小狗和绳子是暂时的聚合关系    |
|  依赖  |  一个类用到另一个：depends-a   | 被依赖方 |      人养小狗，人依赖于小狗      |
|  关联  |    平等的使用关系：links-a     |   平等   | 人使用卡消费，卡可以提取人的信息 |

## 访问权限控制符

| 访问权限控制符 | 本类 | 包内 | 包外子类 | 任何地方 |
| :------------: | :--: | :--: | :------: | :------: |
|     public     |  √   |  √   |    √     |    √     |
|   protected    |  √   |  √   |    √     |    ×     |
|       无       |  √   |  √   |    ×     |    ×     |
|    private     |  √   |  ×   |    ×     |    ×     |

# 语言特性

## Java语言的优点

- 平台无关性，摆脱硬件束缚，一次编写，到处运行
- 相对安全的内存管理和访问机制，避免大部分内存泄漏和指针越界
- 热点代码检测和运行时编译及优化，使程序随时间运行增长获得更高的性能
- 完善的应用程序接口，支持第三方类库

## Java如何实现平台无关？

JVM：java编译器可生成与计算机体系结构无关的字节码指令，字节码指令不仅可以轻易的在任何机器上解释执行，还可以动态地转换为本地机器代码，转换是由JVM实现的，JVM是平台相关的，屏蔽了不同操作系统的差异

语言规范：基本数据类型有明确的规定，例如int永远为32位，而C/C++中可能是16位、32位，也可能是编译器开发商指定的其他大小，Java中数值类型有固定的字节数，二进制格式存储和传输，字符串采用标准的Unicode格式存储

## JDK和JRE的区别

JDK：Java Development Kit，开发工具包。提供了编译运行Java程序的各种工具，包括编译器（javac.exe）、JRE以及打包工具（jar.exe），是Java的核心

JRE：Java Runtime Environment，运行时环境，运行Java程序所必须的环境，包括JVM、核心类库、核心配置工具

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210612162823.png" alt="img" style="zoom:80%;" />

### Java是按值调用还是引用调用

- 按值传递：是指接受调用者提供的值
- 按引用调用：是指方法接受调用者提供的变量地址

Java总是按值调用，方法得到的所有参数的副本，传递对象时实际上方法接受的是对象引用的副本。方法不能修改基本数据类型的参数，如果传递了一个int值，改变值不会影响实参，因为改变的是值的一个副本

可以改变对象参数的状态，但不能让对象参数引用一个新的对象，如果传递了一个int数组，改变数组的内容会影响实参，而改变这个参数的引用并不会让实参引用新的数组对象

## 浅拷贝和深拷贝的区别

- 浅拷贝：只复制当前对象的基本数据类型即引用变量，没有复制引用变量指向的实际对象。修改克隆对象可能影响原对象，不安全
- 深拷贝：完全拷贝基本数据类型和应用数据类型，安全

# 序列化

Java 对象 JVM 退出时会全部销毁，如果需要将对象及状态持久化，就要通过序列化实现，将内存中的对象保存在二进制流中，需要时再将二进制流反序列化为对象。==对象序列化保存的是对象的状态，因此属于类属性的静态变量不会被序列化==

## 常见的序列化方式

### Java 原生序列化

实现 `Serializabale` 标记接口，Java 序列化保留了对象类的元数据（如类、成员变量、继承类信息）以及对象数据，兼容性最好，但不支持跨语言，性能一般。序列化和反序列化必须保持序列化 ID 的一致，一般使用 `private static final long serialVersionUID` 定义序列化 ID，如果不设置编译器会根据类的内部实现自动生成该值。如果是兼容升级不应该修改序列化 ID，防止出错，如果是不兼容升级则需要修改

实现`Externalizable`接口，在`WriteExternal()`方法里定义了那些属性可以序列化，那些不可以序列化；在`readExternal()`方法，根据序列对象挨个读取进行反序列化，并且自动封装成对象返回

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/5 16:37
 * @description：车
 */
@Component
@Data
public class Car implements Externalizable {
    @Autowired
    public Engine engine;

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(engine);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        Engine engine = (Engine) in.readObject();
    }
}

```

### Hessian 序列化

Hessian 序列化是一种支持动态类型、跨语言、基于对象传输的网络协议。Java 对象序列化的二进制流可以被其它语言反序列化。Hessian 协议的特性：① 自描述序列化类型，不依赖外部描述文件，用一个字节表示常用基础类型，极大缩短二进制流。② 语言无关，支持脚本语言。③ 协议简单，比 Java 原生序列化高效。Hessian 会把复杂对象所有属性存储在一个 Map 中序列化，当父类和子类存在同名成员变量时会先序列化子类再序列化父类，因此子类值会被父类覆盖

### JSON 序列化

JSON 序列化就是将数据对象转换为 JSON 字符串，在序列化过程中抛弃了类型信息，所以反序列化时只有提供类型信息才能准确进行。相比前两种方式可读性更好，方便调试

>序列化通常会使用网络传输对象，而对象中往往有敏感数据，容易遭受攻击，Jackson 和 fastjson 等都出现过反序列化漏洞，因此不需要进行序列化的敏感属性传输时应加上 transient 关键字。transient 的作用就是把变量生命周期仅限于内存而不会写到磁盘里持久化，变量会被设为对应数据类型的零值

# 常问方法和类

## Object 类有哪些方法？

**equals：**检测对象是否相等，默认使用 `==` 比较对象引用，可以重写 equals 方法自定义比较规则。equals 方法规范：自反性、对称性、传递性、一致性、对于任何非空引用 x，`x.equals(null)` 返回 false

**hashCode：**散列码是由对象导出的一个整型值，没有规律，每个对象都有默认散列码，值由对象存储地址得出。字符串散列码由内容导出，值可能相同。为了在集合中正确使用，一般需要同时重写 equals 和 hashCode，要求 equals 相同 hashCode 必须相同，hashCode 相同 equals 未必相同，因此 hashCode 是对象相等的必要不充分条件

**toString**：打印对象时默认的方法，如果没有重写打印的是表示对象值的一个字符串

**clone：**clone 方法声明为 protected，类只能通过该方法克隆它自己的对象，如果希望其他类也能调用该方法必须定义该方法为 public。如果一个对象的类没有实现 Cloneable 接口，该对象调用 clone 方法抛出一个 CloneNotSupport 异常。默认的 clone 方法是浅拷贝，一般重写 clone 方法需要实现 Cloneable 接口并指定访问修饰符为 public

**finalize：**确定一个对象死亡至少要经过两次标记，如果对象在可达性分析后发现没有与 GC Roots 连接的引用链会被第一次标记，随后进行一次筛选，条件是对象是否有必要执行 finalize 方法。假如对象没有重写该方法或方法已被虚拟机调用，都视为没有必要执行。如果有必要执行，对象会被放置在 F-Queue 队列，由一条低调度优先级的 Finalizer 线程去执行。虚拟机会触发该方法但不保证会结束，这是为了防止某个对象的 finalize 方法执行缓慢或发生死循环。只要对象在 finalize 方法中重新与引用链上的对象建立关联就会在第二次标记时被移出回收集合。由于运行代价高昂且无法保证调用顺序，在 JDK 9 被标记为过时方法，并不适合释放资源

**getClass：**返回包含对象信息的类对象。

**wait / notify / notifyAll：**阻塞或唤醒持有该对象锁的线程

## hashCode与equals的

HashSet如何检查重复，两个对象的hashCode()相同，则equals()也一定为true

### hashCode()介绍

hashCode()的作用是获取哈希吗，也称为散列码；它实际上是返回一个int整数，这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode()定义在JDK的Object类中，这就意味着Java中任何类都包含hashCode()函数

散列表存储的是键值对（key-value），它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）

### 为什么要有hashCode()

**我们以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode**：

当你把对象加入HashSet时，HashSet会先计算对象的hashCode值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashCode值作比较，如果没有相符的hashCode，HashSet会假设对象没有重复出现。但是如果发现有相同 hashCode值的对象，这时会调用 equals()方法来检查 hashCode相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置

### hashCode()与equals()的相关规定

如果两个对象相等，则hashcode一定相等

两个对象相等，对两个对象分别调用equals返回都是true

两个对象有相同的hashcode值，但是他们不一定相等

> 因此，equals()方法被覆盖，则hashCode()值也必须被覆盖

hashCode()默认行为是对堆上的对象产生独特的值，如果没有重写hashCode()，则该class两个对象，无论如何都不会相等（即使这两个对象指向相同的数据）

#### 对象的相等与指向他们的引用相等，两者有什么不同？

对象的相等 比的是内存中存放的内容是否相等而 引用相等 比较的是他们指向的内存地址是否相等

### ==和equals？

- **==**：它的作用是判断两个对象的地址是不是相等的，即，判断两个对象是不是同一个对象（基本的数据类型**==**比较的是值，引用数据类型**==**比较的是内存地址）
- **equals()**：它的作用也是判断两个对象是否相等。但它一般有两种使用情况
  - 类没有覆盖equals()方法，则通过equals()比较该类的两个对象时，等价通过**==**比较这两个对象
  - 类覆盖了**equals()**方法，一般，我们都覆盖**equals()**放方法来判断两个对象内容是否相等，若他们的内容相等，则返回true

注意：String中的equals方法是被重写过的，因为Object的equals方法比较的对象的内存地址，而String的equals方法比较的是对象的值

当创建String类型的对象时，虚拟机会在常量池中查找有么有已经存在的值和要创建的值相同的对象，如果有就把它赋值给当前引用，如果没有就在常量池中创建一个String对象

## String a = "abc"  和 String b = new String("abc")有什么区别?

> String a = "abc"

首先会在栈中创建一个对String类对象的引用变量a，然后去查字符串常量池是否含有"abc"。如果有，会把a指向这个对象的地址。如果字符串常量池中没有"abc"，则在栈中创建是三个char型的值‘a’、‘b’、‘c’，然后在堆中创建一个String对象object，它的值是刚才在栈中创建的三个char型值组成的数组{'a','b','c'}，接着这个String对象object会被放入字符串常量池中，最后将a指向这个对象的地址

> String b = new String("abc");

两步

- 第一步同上
- 第二步由于"abc"已经被创建并保存在字符串常量池中，因此JVM会在堆中创建一个String对象，它的值共享栈中已有的三个char型值

# 数据类型

## java的基本数据类型

| 数据类型 |            内存大小            |  默认值  |          取值范围           |
| :------: | :----------------------------: | :------: | :-------------------------: |
|   byte   |               1B               | (byte)0  |         -128 ~ 127          |
|  short   |               2B               | (short)0 |      -2^15^ ~ 2^15^-1       |
|   int    |               4B               |    0     |      -2^31^ ~ 2^31^-1       |
|   long   |               8B               |    0L    |      -2^63^ ~ 2^63^-1       |
|  float   |               4B               |   0.0F   | ±3.4E+38（有效位数 6~7 位） |
|  double  |               8B               |   0.0D   | ±1.7E+308（有效位数 15 位） |
|   char   | 英文1B，中文UTF-8占3B，GBK占2B | '\u0000' |     '\u0000' ~ '\uFFFF'     |
| boolean  |       单个变量4B/数组1B        |  false   |         true、false         |

> JVM没有boolean赋值的专用字节码指令，`boolean f = false` 就是使用 ICONST_0 即常数 0 赋值。单个 boolean 变量用 int 代替，boolean 数组会编码成 byte 数组

## 包装类

每个基本数据类型都对应一个包装类，除了int和char对应Integer和Character外，其余基本数据类型的包装类都是首字母大写

- 自动装箱：将基本类型用它们对应的引用类型包装起来

- 自动拆箱：将包装类型转换为基本数据类型

### int和Integer区别

Java是一个近乎纯洁的面向对象的编程语言，但是为了编程方便还是引入了基本数据类型，但是为了能够将这些基本数据类型当做对象操作，Java为每一个基本数据类型都引入了对应的包装类型（wrapper class）），int 的包装类就是 Integer，从 Java 5 开始引入了自动装箱/拆箱机制，使得二者可以相互转换

### Java 为每个原始类型提供了包装类型：

原始类型: boolean，char，byte，short，int，long，float，double

包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double

### Integer a = 127和Integer b = 127相等吗

对于对象引用类型：==比较的是对象的内存地址。
对于基本数据类型：==比较的是值

如果整型字面量的值在**-128到127之间**，那么自动装箱时不会new新的Integer对象，而是直接引用常量池中的Integer对象，超过范围 a1==b1的结果是false

```java
public static void main(String[] args) {
    Integer a = new Integer(3);
    Integer b = 3;  // 将3自动装箱成Integer类型
    int c = 3;
    System.out.println(a == b); // false 两个引用没有引用同一对象
    System.out.println(a == c); // true a自动拆箱成int类型再和c比较
    System.out.println(b == c); // true

    Integer a1 = 128;
    Integer b1 = 128;
    System.out.println(a1 == b1); // false

    Integer a2 = 127;
    Integer b2 = 127;
    System.out.println(a2 == b2); // true
}
```

## String

### 字符串拼接的方式有哪些

- 直接用 `+` ，底层用 StringBuilder 实现。只适用小数量，如果在循环中使用 `+` 拼接，相当于不断创建新的 StringBuilder 对象再转换成 String 对象，效率极差
- 使用 String 的 concat 方法，该方法中使用 `Arrays.copyOf` 创建一个新的字符数组 buf 并将当前字符串 value 数组的值拷贝到 buf 中，buf 长度 = 当前字符串长度 + 拼接字符串长度。之后调用 `getChars` 方法使用 `System.arraycopy` 将拼接字符串的值也拷贝到 buf 数组，最后用 buf 作为构造参数 new 一个新的 String 对象返回。效率稍高于直接使用 `+`
- 使用 StringBuilder 或 StringBuffer，两者的 `append` 方法都继承自 AbstractStringBuilder，该方法首先使用 `Arrays.copyOf` 确定新的字符数组容量，再调用 `getChars` 方法使用 `System.arraycopy` 将新的值追加到数组中。StringBuilder 是 JDK5 引入的，效率高但线程不安全。StringBuffer 使用 synchronized 保证线程安全

### String a = "a" + new String("b")创建了几个对象？

常量和常量拼接仍是常量，结果在常量池，只要有变量参与拼接结果就是变量，存在堆

使用字面量时只创建一个常量池中的常量，使用 new 时如果常量池中没有该值就会在常量池中新创建，再在堆中创建一个对象引用常量池中常量。因此 `String a = "a" + new String("b")` 会创建四个对-象，常量池中的 a 和 b，堆中的 b 和堆中的 ab

### intern()

```java
String s3 = new String("a")+new String("a");
s3.intern();
String s4 = "aa";
System.out.println(s3==s4);
```

打印结果：

- jdk6：false
- jdk7：true

intern()方法的不同，主要是因为常量池在jdk1.6与jdk1.7的位置不同引起的

- jdk1.6中的常量池是放在perm区，perm区与堆区是分开的，jdk1.6中的intern方法，会把字符串复制到常量池中，并且返回引用
- jdk1.7当中已经从perm当中移动到了堆区（jdk1.8已经完全取消了perm区，已有一个新的元空间替代），jdk1.7会把相同字符串实例的引用添加到常量池中，并没有复制，然后返回该引用（注意==返回的是首次创建时候的引用地址==）

> 一个是复制实例，一个是复制该实例的引用

## StringBuffer和StringBuilder的区别？

### 可变性

- String类中使用字符数组保存字符，`private final char value[]`，所以String对象是不可变的。

- StringBuilder与StringBuffer都继承自`AbstractStringBuilder`类，在`AbstractStringBuilder`中也是使用字符数组保存字符串

### 线程安全性

- String中的对象是不可变的，也就可以理解常量，线程安全。
- `AbstractStringBuilder`是StringBuilder与StringBuffer的公共父类，StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder并没有对方法进行加同步锁，所以是非线程安全的

### 性能

- 每次对String类型进行改变的时候，都会生成一个新的String对象，然后将指向新的String对象。
- StringBuffer每次都会对StringBuffer进行操作，而不是生成新的对象并改变对象的引用，相同情况下使用StringBuilder相比使用StringBuffer仅能获得10%~15% 左右的性能提升，但却要冒多线程不安全的风险

### 总结

- 如果操作少量的数据使用String
- 单线程操作大量数据使用StringBuilder
- 多线程操作大量数据使用StringBuffer

## switch 是否能作用在 byte 上，是否能作用在 long 上，是否能作用在 String 上？

在Java5以前，switch(expr)中，expr只能是byte、short、char、int，从java5开始，java中引入了枚举类型，expr也可以是enum类型，从Java7开始，expr还可以是字符串(String)，但是==long不支持==

# final关键字

- 修饰变量

凡是对成员变量或者局部变量（在方法中的或者代码块中的变量称为本地变量）声明为final的都叫做final变量，final变量经常和static关键字一起使用，作为常量

final修饰的基本数据的变量时，必须赋予初始值且不能变修饰引用变量时，该引用变量不能在指向其他对象

当final修饰基本数据类型变量时不赋予初始值以及引用变量指向其他对象时就会报错

当final修饰基本数据类型变量被改变时，就会报错

- 修饰方法

final也可以声明方法，方法上面加上final关键字，代表这个方法不可以被子类方法重写，如果你认为一个方法的功能足够完整了，子类不需要改变的话，你也可以声明此方法为final，final方法比非final方法要快，因为在编译的时候已经静态绑定了，不需要在运行时在进行绑定

- 修饰类

使用final修饰的类叫做final类，final类通常功能是完整的，他们不能被继承，Java中有许多类是final的，如String，Integer以及其他包装类

### final、finally、finalize 有什么区别？

final可以修饰类、变量、方法，修饰类表示该类不能被继承、修饰方法表示该方法不能被重写、修饰变量表示该变量是一个常量不能被重新赋值。
finally一般作用在try-catch代码块中，在处理异常的时候，通常我们将一定要执行的代码方法finally代码块中，表示不管是否出现异常，该代码块都会执行，一般用来存放一些关闭资源的代码。
finalize是一个方法，属于Object类的一个方法，而Object类是所有类的父类，Java 中允许使用 finalize()方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。



# 注解

## 注解的概念

注解（Annotation）是Java提供的设置程序中元素的关联信息和元数据（MetaData）的方法，他是一个接口，程序可以通过反射获取指定程序中元素的注解对象，然后通过该注解对象获取注解中的元数据信息

## 什么是注解？什么是元注解？

- 注解：是一种标识，使类或借口附加额外的信息，帮助编译器和JVM完成一些特定的功能，例如`@Override`标识一个方法是重写方法。
- 元注解：元注解是自定义注解的注解：
  - `@Target`：约束作用位置，值是ElementType枚举常量，包括 METHOD 方法、VARIABLE 变量、TYPE 类/接口、PARAMETER 方法参数、CONSTRUCTORS 构造方法和 LOACL_VARIABLE 局部变量等
  - `@Retention`：约束生命周期
    - SOURCE：在源文件中有效，即源文件中被保留
    - CLASS：在Class文件中有效，即在Class文件中保留
    - RUNTIME：在运行时有效，即在运行时保留
  - `@Documented`：表明这个注解应该被javadoc工具记录，因此可以为javadoc类的工具文档化
  - `@Inherited`：是一个标记注解，表明某个被标注的类型是被继承的，如果有一个使用了`@Inherited`修饰的Annotation被用于一个Class，则这个注解将被用于该Class类

### 注解处理器

注解用于描述元数据的信息，使用的重点在于对注解处理器的定义，JavaSE5扩展了反射机制的API，以帮助程序员快速构造自定义注解处理器，对注解的使用一般包含定义及使用注解接口，我们一般通过封装统一的注解工具来使用注解

#### 定义注解

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 14:48
 * @description：
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitProvider {
    //供应商编号
    public  int id() default -1;
    //供应商名称
    public String name() default "";
    //供应商地址
    public  String address() default "";
}
```

#### 使用注解接口

定义一个Apple类，并通过注解方式定义了一个属性

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 14:51
 * @description：苹果
 */
@Setter
@Getter
public class Apple {
    @FruitProvider(id = 1, name = "陕西商洛", address = "陕西省西安市")
    private String appleProvider;
}
```

#### 定义注解处理器

通过反射信息获取注解数据

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 14:53
 * @description：注解处理器
 */
public class FruitInfoUtil {

    public static void getFruitInfo(Class<?> clazz) {
        System.out.println("供应商信息");
        Field[] declaredFields = clazz.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            FruitProvider annotation = declaredField.getAnnotation(FruitProvider.class);
            System.out.println("供应商编号：" + annotation.id());
            System.out.println("供应商姓名：" + annotation.name());
            System.out.println("供应商地址：" + annotation.address());
            System.out.println(annotation);
        }
    }
}
```

#### 测试

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 14:54
 * @description：
 */
public class Test {
    public static void main(String[] args) {
        FruitInfoUtil.getFruitInfo(Apple.class);
    }
}

供应商信息
供应商编号：1
供应商姓名：陕西商洛
供应商地址：陕西省西安市
@注解.FruitProvider(name=陕西商洛, address=陕西省西安市, id=1)
```

# 类

## 内部类

定义在类内部的类被称为内部类，内部类根据不同的定义方式，可以分为静态内部类、成员内部类、局部内部类和匿名内部类

### 静态内部类

> 定义在类内部的静态类，就是静态内部类

静态内部类可以访问外部类的静态变量和方法；在静态内部类中可以定义静态变量、静态方法、构造函数等；静态内部类通过==外部类.静态内部类==的方式调用

静态内部类可以访问外部类所有的静态变量，而不可访问外部类的非静态变量；静态内部类的创建方式，==new 外部类.静态内部类()==，如下

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 15:07
 * @description： 静态内部类
 */
public class OuterClass {
    private static String className = "StaticInnerClass";

    public static class StaticInnerClass {
        public void getClassName() {
            System.out.println("className：" + className);
        }
    }

    //调用静态内部类
    public static void main(String[] args) {
        StaticInnerClass staticInnerClass = new StaticInnerClass();
        staticInnerClass.getClassName();
    }
}
```

Java集合类中HashMap在内部维护了一个静态内部类Node数组用于存放元素，Node元素对使用者是透明的，像这种和外部类关系密切且不依赖外部类实例得嘞，可以使用静态内部类

### 成员内部类

> 定义在类内部，成员位置上的非静态类，就是成员内部类。

成员内部内部类不能定义静态方法和变量（final修饰除外），因为成员内部类是非静态的，而在Java的非静态代码块中不能定义静态方法和静态变量

成员内部类可以访问外部类所有的变量和方法，包括静态和非静态，私有和公有。成员内部类依赖于外部类的实例，它的创建方式==外部类实例.new 内部类()==

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 15:16
 * @description：成员内部类
 */
public class OutClass {
    private static int a = 12;
    private int b = 2;

    public class MemberInnerClass {
        public void print() {
            System.out.println(a);
            System.out.println(b);
        }
    }

    public static void main(String[] args) {
        OutClass outClass = new OutClass();
        MemberInnerClass memberInnerClass = outClass.new MemberInnerClass();
        memberInnerClass.print();
    }

}

```

### 局部内部类

> 定义在方法中的内部类，就是局部内部类

当一个类只需要在某个方法中使用特定的类时，可以通过局部内部类来优雅的实现

局部内部类的创建方式，在对应方法内，==new 内部类()==

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 15:26
 * @description：局部内部类
 */
public class OutClass {
    private  static int a;
    private int b;

    public void printClassTest(final int c) {
        final int d = 1;
        class PastClass {
            public void print() {
                System.out.println(c);
            }
        }
        PastClass pastClass = new PastClass();
        pastClass.print();
    }

    public static void main(String[] args) {
        OutClass outClass = new OutClass();
        outClass.printClassTest(10);
    }

}
```

### 匿名内部类

> 匿名内部类就是没有名字的内部类，日常开发中使用的比较多。

匿名内部了通过继承一个父类或者实现一个接口的方式直接定义并使用的类，匿名内部类没有class关键字，这是应为匿名内部类直接使用new生成一个对象引用

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 15:38
 * @description：匿名内部类
 */
public class Worker {
    public void work() {
        // lamda ((Service) () -> System.out.println("工人正在工作....")).doSome();
        new Service() {
            @Override
            public void doSome() {
                System.out.println("工人正在工作....");
            }
        }.doSome();
    }

    public static void main(String[] args) {
        Worker worker = new Worker();
        worker.work();
    }
}

interface Service {
    void doSome();
}
```

#### 匿名内部类的特点

- 匿名内部类必须继承一个抽象类或者实现一个接口
- 匿名内部类不能定义任何静态成员和静态方法
- ==当所在的方法的形参需要被匿名内部类使用时，必须声明为 final==
- 匿名内部类不能是抽象的，它必须要实现继承的类或者实现的接口的==所有抽象方法==

### 总结

#### 内部类的优点

- 一个内部类对象可以访问创建它的外部对象的内容，包括私有数据
- 内部类不为同一个包的其他类所见，具有很好的封装性
- 内部类有效实现了多重继承，优化了java单继承的缺陷
- 匿名内部类可以很方便的定义回调

#### 应用场景

1. 一些多算法场合
2. 解决一些非面向对象的语句块
3. 适当使用内部类，使得代码更加灵活和富有扩展性
4. 当某个类除了它的外部类，==不再被其他的类使用时==

#### 局部内部类和匿名内部类访问局部变量的时候，为什么变量必须要加上final？

因为生命周期不一样，局部变量直接存储在栈中，当方法执行结束后，非final的局部变量就被销毁了。而局部内部类对局部变量的引用依然存在，如果局部内部类要调用局部变量时就会出错，加了final，可以确保局部内部类使用的变量与外层的局部变量区分开，解决了这个问题

```java
/**
 * @author ：zsy
 * @date ：Created 2021/6/12 15:50
 * @description：测试final
 */
public class OuterClass {
    private int age = 12;

    class Inner {
        private int age = 13;

        public void print() {
            int age = 14;
            System.out.println("局部变量：" + age);
            System.out.println("内部类变量：" + this.age);
            System.out.println("外部类变量：" + OuterClass.this.age);
        }
    }

    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        outerClass.new Inner().print();
    }
}	
```



## 抽象类和接口的区别？

> 接口和抽象类对实体类进行更高层次的抽象，仅定义公共行为和特征

| 语法维度 | 抽象类                                             | 接口                                                         |
| -------- | -------------------------------------------------- | ------------------------------------------------------------ |
| 成员变量 | 无特殊要求                                         | 默认public static final常量                                  |
| 构造方法 | 有构造方法，不能实例化                             | 没有构造方法，不能实例化                                     |
| 方法     | 抽象类可以没有抽象方法，但有抽象方法的一定是抽象类 | 默认public abstract，JDK8支持默认/静态方法，JDK9支持私有方法 |
| 继承     | 单继承                                             | 多继承                                                       |

## 抽象类和接口怎么选择？

- 抽象类体现is-a关系，接口体现can-do关系。与接口相比，抽象类是对同类事务相对具体的抽象
- 抽象类是模板设计，包含一组具体特征，例如某汽车，底盘、控制电路等是抽象出来的共同特征，但内饰、显示屏、座椅材质可以根据不同级别配置存在不同实现
- 接口是契约式设计，是开放的，定义了方法名、参数、返回值、抛出的异常类型，谁都可以实现它，但必须遵守接口的约定。例如所有车辆都必须实现刹车这种强制规范
- 接口是顶级类，抽象类在接口下面的第二层，对接口进行了组合，然后实现了部分接口，当纠结定义接口和抽象类时，推荐定义为接口，遵循接口隔离原则，按维度划分成多个接口，再利用抽象类去实现这些，方便后续的扩展和重构

# 泛型

## 什么是泛型，有什么用？

泛型的本质是参数化类型，解决不确定对象具体类型的问题。泛型在定义处只具备执行Object方法的能力

泛型的好处：

- 类型安全，放置什么出来就是什么，不存在 ClassCastException
- 提升可读性，编码阶段就显示知道泛型集合、泛型方法等处理的对象类型
- 代码重用，合并了同类型的处理代码

## 泛型标记和泛型限定

## 泛

| 泛型标记  |                  说明                  |
| :-------: | :------------------------------------: |
| E-Element |   在集合中使用，表示集合中存放的元素   |
|  T-Type   | 表示Java类，包括基本的类和我们定义的类 |
|   K-Key   |          表示键，比如Map的key          |
|  V-Value  |                 表示值                 |
| N-Number  |              表示数据类型              |
|     ?     |             表示不确定类型             |

> 泛型限定

- 对泛型上限的限定`<? extends T>`：表示该通配符所代表的类型是T类型的子类或者几口T的子接口
- 对翻新下限限定`<? super T>`：表示该通配符所代表的类型是T类型的父类或者父接口

## 泛型擦除是什么？

泛型用于编译阶段，编译后字节码文件不包括泛型类型信息，因为虚拟机没有泛型类型对象，所以编译时，类型参数会被编译器时去掉，这个过程称为类型擦除

例如：编码时定义 `List<Object>` 或 `List<String>`，在编译后都会变成 `List` ，泛型附加的类型信息对JVM来说是不可见的

类型擦除的过程，首先查找用来替换类型参数的具体类（一般为Object），如果指定了类型参数的上界，则以上界作为替换时的具体类，然后把代码中的类型参数都替换为具体的类型

例如：`<T extends A>`会使用A类型替换T

如果有多个限定类型就会替换为第一个限定类型

例如： `<T extends A & B>` 会使用 A 类型替换 T

# Java集合容器面试题

## 常用的集合类

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210604170635.png" alt="image-20210604170631750" style="zoom:80%;" />

Collection接口的子类包括：Set接口、Queue接口和List接口

- Set接口（无序：元素存入和取出的顺序可能不一致）的实现类主要有：HashSet、TreeSet、LinkedHashSet
- Queue接口包含Deque和BlockingQueue
- List接口（有序：元素存入集合的顺序和取出顺序一致）的实现类：ArrayList、LinkedList、Stack以及Vector

![image-20210604170955453](https://gitee.com/zhang-songyao/blog-images/raw/master/20210604170957.png)

Map接口：键值对集合，存储键、值之间的映射。Key无序，唯一；value，不要求有序，允许重复

实现类：HashMap、TreeMap、HashTable、ConcurrentHashMap以及Properties

## List接口


### 谈谈ArrayList

ArrayList是容量可变的非线程安全列表，使用数组实现，集合扩容时会创建更大的数组，把原有的数组复制到新的数组中。支持对元素的快速随机访问，但插入与删除的速度很慢。ArrayList实现了RandomAcess标记接口，如果一个类实现了这个接口，证明使用索引遍历比迭代器更快

ArrayList主要底层实现是Object[]elementData，被transient修饰，重写了writeObject()方法，序列化时会调用 writeObject()写入流，反序列化时调用 readObject()重新赋值到新对象的 elementData

> writeObject()

先调用 defaultWriteObject() 方法序列化 ArrayList 中的非 transient 元素，然后遍历 elementData，只序列化已存入的元素，这样既加快了序列化的速度，又减小了序列化之后的文件大小

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();
    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);
    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

size是实际的大小，elementData大小等于size

**modCount **记录了 ArrayList 结构性变化的次数，继承自 AbstractList。所有涉及结构变化的方法都会增加该值。expectedModCount 是迭代器初始化时记录的 modCount 值，每次访问新元素时都会检查 modCount 和 expectedModCount 是否相等，不相等就会抛出异常。这种机制叫做 fail-fast，所有集合类都有这种机制

#### 初始容量

> 初始容量为10

JDK1.7时，相当于饿汉式，第一次创建无参构造器时，就创建一个初始容量为10的数组

JDK1.8时，相当于懒汉式，只有当整整add时才会分配默认的初始容量

#### 扩容

> ArrayList的扩容阈值为1.5

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

扩容过程：

- 假设有一个长度为10的数组，此时新增加一个元素，发现ArrayList已经满了，需要扩容
- 重新定义一个长度为10 * 1.5的数组
- 将原数组的数据原封不动的复制到新的数组
- 将ArrayList的地址指向新数组

#### 如何实现数组和List之间的转换

数组转List：使用Arrays.asList()进行转换

List转数组：使用List自带的toArray()方法

### 谈谈LinkedList

LinkedList的本质是双向链表，AbstractList 外还实现了 Deque 接口，这个接口具有队列和栈的性质，成员变量被 transient 修饰，原理和 ArrayList 类似

LinkedList 包含三个重要的成员：size、first 和 last。size 是双向链表中节点的个数，first 和 last 分别指向首尾节点的引用

LinkedList的优点在于可以将零碎的内存空间通过附加引用的方式关联起来，形成链路顺序查找的线性结构，内存利用率高

### ArrayList和LinkedList的区别

- 数据结构实现：ArrayList是动态数组的数据结构实现，而LinkedList是双向链表的数据结构实现

- ArrayList的查询和访问速度较快，但是新增、删除的速度较慢，LinkedList的查找和访问元素到的速度较慢，但是它的新增，删除速度较快
- ArrayList需要一份连续的内存空间，LinkedList不需要连续的内存空间（特别地，当创建一个ArrayList集合的时候，连续的内存空间必须要大于等于创建的容量）
- 两者都是线程不安全的

> 综合来说，在需要频繁读取集合中的元素时，更推荐使用 ArrayList，而在插入和删除操作较多时，更推荐使用 LinkedList

### ArrayList和Vector的区别

| 区别     | ArrayList        | Vector        |
| -------- | ---------------- | ------------- |
| 线程安全 | 非线程安全       | 线程安全      |
| 性能     | 优               | 差            |
| 扩容     | ArrayList扩容1.5 | Vector扩容2倍 |

Vector类的所有方法都是同步的。可以由两个线程安全地访问一个Vector对象、但是一个线程访问Vector的话代码要在同步操作上耗费大量的时间

Arraylist不是同步的，所以在不需要保证线程安全时时建议使用Arraylist

### 快速失败（fail-fast）

是Java集合的一种错误检测机制，当多个线程对集合进行结构上的改变操作时，有可能会产生fail-fast机制

迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历

### 安全失败（fail-safe）

采用安全失败机制的集合容器，在比那里时不是直接在集合内容上访问的，而是先复制原有的集合内容，在拷贝的集合上进行遍历

原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException

缺点：基于拷贝内容的优点是避免了ConcurrentModificationException，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的

> 场景：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。

### 怎么确保一个集合不被修改

可以使用 Collections. unmodifiableCollection(Collection c) 方法来创建一个只读集合，这样改变集合的任何操作都会抛出 Java. lang. UnsupportedOperationException 异常

## 迭代器

### Iterator

Iterator接口提供遍历任何Collection的接口。我们可以从一个 Collection 中使用迭代器方法来获取迭代器实例。迭代器取代了 Java 集合框架中的 Enumeration，迭代器允许调用者在迭代过程中移除元素

迭代器的特点：只能单向遍历，但是更加安全，因为它可以担保，在当前遍历的集合元素被更改的时候，就会抛出ConcurrentModificationException 异常

如何边遍历边删除Collection中的元素

```java
Iterator<Integer> it = list.iterator();
while(it.hasNext()){
   *// do something*
   it.remove();
}
错误写法：list.remove(i)
```

### Iterator 和 ListIterator 有什么区别

Iterator 可以遍历 Set 和 List 集合，而 ListIterator 只能遍历 List
Iterator 只能单向遍历，而 ListIterator 可以双向遍历（向前/后遍历）
ListIterator 实现 Iterator 接口，然后添加了一些额外的功能，比如添加一个元素、替换一个元素、获取前面或后面元素的索引位置

## Set接口

### 谈谈HashSet

HashSet是基于HashMap实现的，HashSet的值存放与HashMap的key上，HashMap的value统一为PRESENT，基本上都是直接调用底层的HashMap相关方法来完成的

```java
private static final Object PRESENT = new Object();
```

HashMap先比较hashCode，再比较equals，所以key是不会重复的

```java
public boolean add(E e) {
	return map.put(e, PRESENT)==null;
}
```

### TreeSet同TreeMap

## Map接口

### 谈谈TreeMap

TreeMap基于红黑树实现，增删改查的平均和最差时间复杂度均为O(logn)，最大的特点是Key有序，Key必须实现Comparable接口或者提供Comparator比较器，所以Key不能为null

HashMap依靠hashCode和equals去重，而TreeMap依靠Comparable或者Comparator。TreeMap 排序时，如果比较器不为空就会优先使用比较器的 compare 方法，否则使用 Key 实现的 Comparable 的 compareTo 方法，两者都不满足会抛出异常

> 红黑树的操作，HashMap中会有介绍

### Comparable和Comparator接口的区别

Comparable一般都是通过类去实现接口，在类内部去实现comparaTo方法，所以一般人也称为内部比较器。实现了Comparable接口的类有一个共同特点，就是这些类可以和自己比较，至于具体和另一个实现了Comparable接口的类如何比较，则依赖compareTo方法的实现

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

Comparator一般都是写一个类去实现Comparator接口，让这个类作为专用的比较器，在需要比较器的地方当做参数传进去，外比较器。 一个对象实现了Comparable接口，但是开发者认为compareTo方法中的比较方式并不是自己想要的那种比较方式

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

#### 升序降序记法

这里o1表示位于前面的对象，o2表示后面的对象

返回-1（或负数），表示不需要交换01和02的位置，o1排在o2前面，asc 返回1（或正数），表示需要交换01和02的位置，o1排在o2后面，desc

#### 总结

1、如果实现类没有实现Comparable接口，又想对两个类进行比较（或者实现类实现了Comparable接口，但是对compareTo方法内的比较算法不满意），那么可以实现Comparator接口，自定义一个比较器，写比较算法

2、实现Comparable接口的方式比实现Comparator接口的耦合性  要强一些，如果要修改比较算法，要修改Comparable接口的实现类，而实现Comparator的类是在外部进行比较的，不需要对实现类有任何修  改。从这个角度说，其实有些不太好，尤其在我们将实现类的.class文件打成一个.jar文件提供给开发者使用的时候。实际上实现Comparator 接口的方式后面会写到就是一种典型的策略模式。

### HashMap

[HashMap、Hashtable、ConcurrentHashMap（1.7、1.8）源码分析 + 红黑树](https://blog.csdn.net/qq_45796208/article/details/115984778?spm=1001.2014.3001.5501)

## Queue

队列的特点

队列是一种比较特殊的线性结构。它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作。进行插入操作的端称为队尾，进行删除操作的端称为队头。
队列中最先插入的元素也将最先被删除，对应的最后插入的元素将最后被删除。因此队列又称为“先进先出”（FIFO—first in first out）的线性表，与栈(FILO-first in last out)刚好相反

### BlockingQueue

四组API

|      方式      | 抛出异常 | 不抛出异常，有返回值 | 阻塞一直等待 |            超时等待            |
| :------------: | :------: | :------------------: | :----------: | :----------------------------: |
|      添加      |   add    |        offer         |     put      | offer("c",2, TimeUnit.SECONDS) |
|      删除      |  remove  |         poll         |     take     |    poll(2,TimeUnit.SECONDS)    |
| 判断对列首元素 | element  |         peek         |      -       |               -                |



[IO与NIO](https://www.jianshu.com/p/5bb812ca5f8e)

# JavaIO流

## 阻塞（Block）与非阻塞（Non-Block）

> 阻塞和非阻塞是进程在访问数据的时候，数据是否准备就绪的一种处理方式。

当数据没有准备的时候

**阻塞**：往往需要等待缓冲区中的数据准备好之后才处理其他的事情，否则一直等待。

**非阻塞**：当我们的进程访问我们的数据缓冲区的时候，如果数据没有准备好，则直接返回，不会等待，如果数据已经准备好，也直接返回

### 阻塞IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507174328.jpeg)

### 非阻塞IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507174351.jpeg)

## 同步（Synchronous）与异步（Asynchronous）

同步和异步都是基于应用程序和操作系统处理IO时间所采用的方式

**同步**：是应用程序要直接参与IO读写操作

**异步**：所有的IO读写都交给操作系统去处理，应用程序只需要等待通知。

同步方式在处理 IO 事件的时候，必须阻塞在某个方法上面等待我们的 IO 事件完成（阻塞 IO 事件或者通过轮询 IO事件的方式），对于异步来说，所有的 IO 读写都交给了操作系统。这个时候，我们可以去做其他的事情，并不需要去完成真正的 IO 操作，当操作完成 IO 后，会给我们的应用程序一个通知。

> 举例

同步阻塞：普通水壶烧水，站在旁边，主动看水开了没有

同步非阻塞：去干点别的事，每过一段时间去看看水开了没有，水没开就走人

异步阻塞：站在旁边，不会每过一段时间主动看水开了没有。如果水开了，水壶自动通知他。 

异步非阻塞：去干点别的事，如果水开了，水壶自动通知他。异步非阻塞

> 所以异步相比于同步带来的好处就是在我们处理IO数据的时候，异步的方式我们可以把这部分等待所消耗的资源用于处理其他事务，提升我们服务自身的性能

### 同步IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507175052.jpeg)

### 异步IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507175103.jpeg)

## 总结

同步和异步是通信机制，阻塞和非阻塞是调用状态

- 同步IO是用户线程发起IO请求后需要等待或轮询内核IO操作完成后才能继续执行

- 异步IO是用户线程发起IO请求后可以继续执行，当内核IO操作完成后会通知用户线程，或调用用户线程注册的回调函数
- 阻塞IO是IO操作需要彻底完成后才能返回用户空间
- 非阻塞IO是IO操作调用后立即返回一个状态值，无需等待IO操作彻底完成

## NIO模型

### NIO（Non-blocking / New I/O）

NIO是一种同步非阻塞的IO模型，于 Java 1.4 中引入，对应 `java.nio` 包，提供了Channel、Selector、Buffer等抽象。NIO中的N可以理解为Non-blocking，不单纯是 New。它支持面向缓冲的，基于通道的 I/O 操作方法。NIO提供了与传统BIO模型中的`Socket`和`ServerSocket`相对应的`SocketChannel`和`ServerSocketChannel`两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发



NIO的非阻塞模式，是一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用，就可以什么都不获取，而不是保持线程阻塞，所以直到数据变得可以读取之前，该线程可以继续做其他的事情，非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 **线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道**

### NIO的特点

- 一个线程可以处理多个通道，减少线程数创建的数量
- 读写非阻塞，节约资源，没有可写\可读数据时，不会发生阻塞导致线程资源的浪费

### 核心组件

#### Channel（通道）

Channel是一个双向通道，与传统IO操作只允许单向的读写不同的是，NIO的Channel允许在一个通道上进行读和写的操作，替换了传统BIO中的Stream，不能直接访问数据，要通过Buffer来读取数据，也可和其他Channel交互

> 主要实现

`FileChannel`、`SocketChannel`、`ServerSocketChannel`、`DatagramChannel`

#### Buffer（缓冲区）

Buffer顾名思义，他是一个缓冲区，实际上是一个容器，一个连续数组，Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507200113.webp)

Buffer缓冲区本质是一块可以写入数据，然后可以从中读取数据的内存这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该模块内存。为了理解Buffer的工作原理，需要熟悉它的三个属性：

- position：下次读写数据的位置
- limit：本次读写的极限位置
- capacity：最大容量

使用步骤：向 Buffer 写数据，调用 flip 方法转为读模式，从 Buffer 中读数据，调用 clear 或 compact 方法清空缓冲区

- `flip` 将写转为读，底层实现原理把 position 置 0，并把 limit 设为当前的 position 值
- `clear` 将读转为写模式（用于读完全部数据的情况，把 position 置 0，limit 设为 capacity）
- `compact` 将读转为写模式（用于存在未读数据的情况，让 position 指向未读数据的下一个）

通道方向和 Buffer 方向相反，读数据相当于向 Buffer 写，写数据相当于从 Buffer 读

#### Selector（多路复用器）

轮询检查多个Channel的状态，判断注册事件是否发生，即判断Channel是否处于可读或可写状态，适用前需要将Channel注册到Selector中，注册后会得到一个`SelectionKey`，通过`SelectionKey`获取Channel和Selector的相关信息

## BIO模型

### BIO（传统IO）

- BIO是一个同步并阻塞的IO模式，传统的 `java.io` 包，它基于流模型实现，提供了我们最熟知的一些 IO 功能，比如**File抽象、输入输出流**等。**交互方式是同步、阻塞的方式**
- 在读取输入流或者写入输出流时，在读、写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序
- 服务器实现模式为一个连接请求对应一个线程，服务器需要为每一个客户端请求创建一个线程，如果这个连接不做任何事会造成不必要的线程开销。可以通过线程池改善，这种IO称为伪异步IO，适用连接数目少且服务器资源多的场景

## Java BIO与NIO比较

| IO模型 |  BIO   |   NIO    |
| :----: | :----: | :------: |
|  通信  | 面向流 | 面向缓冲 |
|  处理  | 阻塞IO | 非阻塞IO |
|  触发  |   无   |  选择器  |

## AIO概念

AIO是JDK7引入的异步非阻塞IO。服务器实现模式为一个有效的请求对应一个线程，客户端的IO请求都是由操作系统先完成IO操作后，再通知服务器应用来直接使用准备好的数据，适用连接数目多且连接时间长的场景

实现方式包括通过 Future 的 `get` 方法进行阻塞式调用以及实现 CompletionHandler 接口，重写请求成功的回调方法 `completed` 和请求失败回调方法 `failed`

## JavaIO流分为哪几种

- 按照流的流向分，可以分为输入流和输出流
- 按照操作单元划分，可以分为字节流和字符流
- 按照流的角色划分为节点流和处理流

> 字符流一般用于文本文件，字节流一般用于图像或者其他文件

字符流包括了字符输入流Reader和字符输出流Writer，字节流包括字节输入流`inputStream`和字节输出流`OutputStream`。

字符流和字节流都有对应的缓冲流，字节流也可以分为字符流，缓冲流带有一个8KB的缓冲数组，可以提高流的读写效率。

除了缓冲流外还有过滤流`FilterReader`、字符数组流`CharArrayReader`、字节数组流`ByteArrayInputStream`、文件流`FileInputStream`

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210612210945.jpeg" alt="IO-操作对象分类" style="zoom:80%;" />



# JDK8新特性

## Lambda表达式

> 允许把函数作为参数传递到方法，简化匿名内部类代码

- 为什么要使用Lambda表达式

  - 避免匿名内部类定义过多
  - 可以代码看起来很简洁

- 函数式接口的定义

  - 任何接口，如果只包含唯一一个抽象方法，那么他就是一个函数式接口

  ```java
  public interface Runnable {
      public abstract void run();
  }
  ```

  - 对于函数式接口，我们可以通过Lambda表达式来创建它的接口对象

  ```java
  interface Love {
      void lambda(int a);
  }
  
  Love love = (int a) -> {...}
  ```

- Lambda表达式简化

  - 去掉参数类型

  ```java
  love = (a) -> {...}
  ```

  - 去掉括号

  ```java
  love = a -> {...}
  ```

  - 去掉花括号

  ```java
  love = a -> ...
  ```

  - 多个参数情况

  ```java
  love = (a, b, c) -> ...
  ```


## 函数式接口

> 输入T类型返回R类型

```java
public interface Function<T, R> {
    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
```

实例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 19:26
 * @description：函数式接口
 */
public class Demo1 {
    public static void main(String[] args) {
        Function<String, String> function = str -> {return str;};

        System.out.println(function.apply("zhangsna"));
    }
}
```

### 断定性接口

> 输入T类型返回boolean类型

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
```

实例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 19:29
 * @description：断定性接口
 */
public class Demo2 {
    public static void main(String[] args) {
        Predicate<String> predicate = str -> {return str.isEmpty();};

        predicate.test("zhangsan");

    }
}
```

### 消费型接口

> 只要输入类型，没有返回类型

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
```

### 供给性接口

> 只有返回类型，没有输入类型

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

实例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 19:37
 * @description：消费型接口和供给型接口
 */
public class Demo3 {

    public static void main(String[] args) {
        Consumer<String> consumer = o -> System.out.println(o);

        Supplier<Integer> supplier = () -> 1024;
        
    }
}
```

## 方法引用

可以引用已有类或对象的方法和构造方法，进一步简化 lambda 表达式

## 接口

接口可以定义 `default` 修饰的默认方法，降低了接口升级的复杂性，还可以定义静态方法。

## 注解

引入重复注解机制，相同注解在同地方可以声明多次。注解作用范围也进行了扩展，可作用于局部变量、泛型、方法异常等。

## 类型推测

加强了类型推测机制，使代码更加简洁。

## Optional 类

处理空指针异常，提高代码可读性。

## Stream 类

引入函数式编程风格，提供了很多功能，使代码更加简洁。方法包括 `forEach` 遍历、`count` 统计个数、`filter` 按条件过滤、`limit` 取前 n 个元素、`skip` 跳过前 n 个元素、`map` 映射加工、`concat` 合并 stream 流等。

## 日期

增强了日期和时间 API，新的 java.time 包主要包含了处理日期、时间、日期/时间、时区、时刻和时钟等操作。

## JavaScript

提供了一个新的 JavaScript引擎，允许在 JVM上运行特定JavaScript 应用。

