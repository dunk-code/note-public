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

