

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

静态内部类可以访问外部类的静态变量和方法；在静态内部类中可以定义静态变量、静态方法、构造反函数等；静态内部类通过==外部类.静态内部类==的方式调用

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