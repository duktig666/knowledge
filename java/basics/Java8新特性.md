

# Java8新特性概述

Java 8 (又称为jdk 1.8) 是Java 语言开发的一个主要版本。
Java 8 是oracle公司于2014年3月发布，可以看成是自Java 5 以来最具革命性的版本。Java 8为Java语言、编译器、类库、开发工具与JVM带来了大量新特性。

## Java8的优势

- 速度更快
- 代码更少(增加了新的语法：**Lambda 表达式**)
- **强大的Stream API**
- 便于并行
- 最大化减少空指针异常：Optional
- Nashorn引擎，允许在JVM上运行JS应用

*其中最为核心的为Lambda 表达式与Stream API*



## 图示

![image-20200726214114740](https://gitee.com/koala010/typora/raw/master/img/20200726214114.png)



# Lambda表达式

## 为什么使用Lambda 表达式？

Lambda 是一个**匿名函数**，我们可以把Lambda 表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。使用它可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升。



## Lambda表达式语法

*注：示例中有些使用了函数式接口，可参考下文*

### 基础语法

Java8中引入了一个新的操作符 "->" 该操作符称为箭头操作符或 Lambda 操作符箭头操作符将 Lambda 表达式拆分成两部分：
 * 左侧：Lambda 表达式的参数列表
* 右侧：Lambda 表达式中所需执行的功能， 即Lambda 体

### 语法格式一：无参数，无返回值

```java
() -> System.out.println("Hello Lambda!");
```

#### 示例

```java
@Test
public void test1(){
    //java8之前的写法
    //jdk 1.7 前，内部类引用外部的局部变量必须是 final
    String str = "java8之前的接口内部实现类写法";
    Runnable r = new Runnable() {
        @Override
        public void run () {
            System.out.println(str);
        }
    };
    r.run();

    //java8之后的写法
    String str1 = "测试“无参数，无返回值”的Lambda表达式，实现接口内部实现类写法";
    Runnable r1 = () -> System.out.println(str1);
    r1.run();
}
```

结果：

```java
java8之前的接口内部实现类写法
测试“无参数，无返回值”的Lambda表达式，实现接口内部实现类写法
```



### 语法格式二：有一个参数，并且无返回值

```java
(x) -> System.out.println(x)
```

#### 示例

```java
@Test
public void test2(){
    Consumer<String> consumer = (x) -> System.out.println(x);
    consumer.accept("测试“有一个参数，并且无返回值”的lambda表达式");
}
```

结果：

```
测试“有一个参数，并且无返回值”的lambda表达式
```



### 语法格式三：若只有一个参数，小括号可以省略不写

```java
x -> System.out.println(x)
```

#### 示例

对于上边的示例可以省略括号，如下：

```java
@Test
public void test2(){
    Consumer<String> consumer = x -> System.out.println(x);
    consumer.accept("测试“若只有一个参数，小括号可以省略不写”的lambda表达式");
}
```



### 语法格式四：有两个以上的参数，有返回值，并且 Lambda 体中有多条语句

```java
(x, y) -> {
    //执行语句
    return [返回值];
};
```

#### 示例

```java
@Test
public void test3(){
    BiFunction<Integer,Integer,Integer> biFunction = (x,y) -> {
        System.out.println("测试“有两个以上的参数，有返回值，并且 Lambda 体中有多条语句”");
        return x+y;
    };
    Integer sum = biFunction.apply(1, 2);
    System.out.println(sum);
}
```

结果:

```
测试“有两个以上的参数，有返回值，并且 Lambda 体中有多条语句”
3
```



### 语法格式五：若 Lambda 体中只有一条语句， return 和 大括号都可以省略不写

```java
Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
```

#### 示例

改写语法格式四的示例

```java
@Test
public void test4 () {
    BiFunction<Integer, Integer, Integer> biFunction = ( x, y ) -> x + y;
    Integer sum = biFunction.apply(1, 2);
    System.out.println(sum);
}
```



### 语法格式六：Lambda 表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推断出，数据类型，即“类型推断”		

```java
(Integer x, Integer y) -> Integer.compare(x, y);
```

可以省略为

```java
(x , y) -> Integer.compare(x, y);
```

#### 示例：

改写语法格式二的示例

两种写法效果相同

```
Consumer<String> consumer = (String x ) -> System.out.println(x);
consumer.accept("测试“有一个参数，并且无返回值”的lambda表达式");
```

```java
Consumer<String> consumer = x -> System.out.println(x);
consumer.accept("测试“有一个参数，并且无返回值”的lambda表达式");
```



### 类型推断

上述Lambda 表达式中的参数类型都是由编译器推断得出的。Lambda 表达式中无需指定类型，程序依然可以编译，这是因为javac根据程序的上下文，在后台推断出了参数的类型。Lambda 表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的“类型推断”。



类型推断不只是在`Lambda表达式`中运用，在很多地方也有使用到，比如泛型的类型推断：

```
Map<String,string> map = new HashMap<>();
List<Integer> list = new ArrayList<>();
```

"="号后边的泛型可以省略，用的也是类型推断（在Java7中则不支持，必须进行显示声明）。



小结：

上联：左右遇一括号省
下联：左侧推断类型省
横批：能省则省



### 语法支持

**Lambda 表达式需要“函数式接口”的支持**

> 函数式接口：接口中只有一个抽象方法的接口，称为函数式接口。 可以使用注解 @FunctionalInterface 修饰可以检查是否是函数式接口



## 函数式接口

### 什么是函数式(Functional)接口？

- 只包含一个抽象方法的接口，称为函数式接口。
- 你可以通过Lambda 表达式来创建该接口的对象。（若Lambda 表达式抛出一个受检异常(即：非运行时异常)，那么该异常需要在目标接口的抽象方法上进行声明）。
- 我们可以在一个接口上使用`@FunctionalInterface 注解`，这样做可以检查它是否是一个函数式接口。同时javadoc 也会包含一条声明，说明这个接口是一个函数式接口。
- 在`java.util.function包`下定义了Java 8 的丰富的函数式接口



#### 举例

![image-20200620184440255](https://gitee.com/koala010/typora/raw/master/img/20200726214127.png)



### 如何理解函数式接口?

- Java从诞生日起就是一直倡导“一切皆对象”，在Java里面面向对象(OOP)编程是一切。但是随着python、scala等语言的兴起和新技术的挑战，Java不得不做出调整以便支持更加广泛的技术要求，也即**java不但可以支持OOP还可以支持OOF（面向函数编程）**
- 在函数式编程语言当中，函数被当做一等公民对待。在将函数作为一等公民的编程语言中，Lambda表达式的类型是函数。但是在Java8中，有所不同。**在Java8中，Lambda表达式是对象，而不是函数，它们必须依附于一类特别的对象类型——函数式接口。**
- 简单的说，**在Java8中，Lambda表达式就是一个函数式接口的实例。**这就是Lambda表达式和函数式接口的关系。也就是说，**只要一个对象是函数式接口的实例，那么该对象就可以用Lambda表达式来表示。**
- 所以以前用匿名实现类表示的现在都可以用Lambda表达式来写



### 自定义函数式接口

#### 示例

```java
/**
 * 自定义函数式接口
 *  不带泛型
 */
@FunctionalInterface
interface MyFunctional{

    /**
     * 处理整数方法
     * @param num 整数
     * @return 处理后的整数
     */
    Integer handleNumber(Integer num);
}

/**
 * 自定义函数式接口
 *  带泛型
 */
@FunctionalInterface
interface MyFunc<T>{

    /**
     * 处理泛型数据
     * @param t 目标泛型数据
     * @return 处理后的泛型数据
     */
    T handle(T t);
}
```

#### 使用自定义的函数式接口

```java
@Test
public void test6 () {
    MyFunctional myFunctional = x -> x * x;
    Integer n = myFunctional.handleNumber(4);
    System.out.println("测试不带泛型的自定义函数式接口，结果：" + n);

    MyFunc<String> myFunc = x -> {
        if (x.length() > 5) {
            x=x.substring(0, 5);
        }
        return x;
    };
    String s = myFunc.handle("测试带泛型的自定义函数式接口");
    System.out.println(s);
}
```

结果：

```java
测试不带泛型的自定义函数式接口，结果：16
测试带泛型
```



### 作为参数传递Lambda 表达式

实例（稍后写）

**为了将Lambda 表达式作为参数传递，接收Lambda表达式的参数类型必须是与该Lambda 表达式兼容的函数式接口的类型。**



### Java 内置四大核心函数式接口

| 类型       | 函数式接口     | 参数类型 | 返回类型 | 包含方法          | 用途                                                   |
| ---------- | -------------- | -------- | -------- | ----------------- | ------------------------------------------------------ |
| 消费型接口 | Consumer<T>    | T        | void     | void accept(T t)  | 对类型为T的对象应用操作                                |
| 供给型接口 | Supplier<T>    | 无       | T        | T get()           | 返回类型为T的对象                                      |
| 函数型接口 | Function<T, R> | T        | R        | R apply(T t)      | 对类型为T的对象应用操作，<br>返回R类型的对象。         |
| 断定型接口 | Predicate<T>   | T        | boolean  | boolean test(T t) | 确定类型为T的对象是否满足某约束，<br/>并返回boolean 值 |



#### 使用实例

```java
/**
 * 测试四大内置函数式接口
 */
@Test
public void test7 () {
    handleStr("aaBb", x -> {
        System.out.println("---测试消费型函数式接口，处理字符串---");
        x = x.toUpperCase();
        System.out.println(x);
    });

    System.out.println("---利用供给型接口：创建指定数量的随机数，存储到List并返回---");
    List<Integer> list = handleStr(5, () -> (int) (Math.random() * 100));
    for (Integer num : list) {
        System.out.println(num);
    }

    System.out.println("---利用函数型接口接口，处理字符串类型数据（一个参数，有返回值）;改写消费型接口的示例---");
    String s = handleStr2("aaBb", x -> x = x.toUpperCase());
    System.out.println(s);

    System.out.println("---利用断言型接口，判断字符串是否符合条件---");
    handleStr3("利用断言型接口，判断字符串是否符合条件", x -> x.length() > 3);
}

/**
 * 利用消费型函数式接口，处理字符串类型数据（一个参数，无返回值）
 */
private void handleStr ( String str, Consumer<String> consumer ) {
    consumer.accept(str);
}

/**
 * 利用供给型接口：创建指定数量的随机数，存储到List并返回
 */
private List<Integer> handleStr ( int num, Supplier<Integer> sup ) {
    List<Integer> list = new ArrayList<>();
    for (int i = 0; i < num; i++) {
        Integer n = sup.get();
        list.add(n);
    }
    return list;
}

/**
 * 利用函数型接口接口，处理字符串类型数据（一个参数，有返回值）
 * 改写消费型接口的示例
 */
private String handleStr2 ( String str, Function<String, String> function ) {
    return function.apply(str);
}

/**
 * 利用断言型接口，判断字符串是否符合条件
 */
private void handleStr3 ( String str, Predicate<String> pre ) {
    if(pre.test(str)){
        System.out.println("符合条件");
    }
}
```

结果：

```java
---测试消费型函数式接口，处理字符串---
AABB
---利用供给型接口：创建指定数量的随机数，存储到List并返回---
80
15
55
83
66
---利用函数型接口接口，处理字符串类型数据（一个参数，有返回值）;改写消费型接口的示例---
AABB
---利用断言型接口，判断字符串是否符合条件---
符合条件
```



### 全部函数式接口API

这些API实在四大函数式接口的基础上派生出来的，使用方法如出一辙。

| 序号 | 接口 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **BiConsumer<T,U>** 代表了一个接受两个输入参数的操作，并且不返回任何结果 |
| 2    | **BiFunction<T,U,R>** 代表了一个接受两个输入参数的方法，并且返回一个结果 |
| 3    | **BinaryOperator<T>** 代表了一个作用于于两个同类型操作符的操作，并且返回了操作符同类型的结果 |
| 4    | **BiPredicate<T,U>** 代表了一个两个参数的boolean值方法        |
| 5    | **BooleanSupplier** 代表了boolean值结果的提供方               |
| 6    | **Consumer<T>** 代表了接受一个输入参数并且无返回的操作        |
| 7    | **DoubleBinaryOperator**代表了作用于两个double值操作符的操作，并且返回了一个double值的结果。 |
| 8    | **DoubleConsumer**代表一个接受double值参数的操作，并且不返回结果。 |
| 9    | **DoubleFunction<R>** 代表接受一个double值参数的方法，并且返回结果 |
| 10   | **DoublePredicate**代表一个拥有double值参数的boolean值方法   |
| 11   | **DoubleSupplier**代表一个double值结构的提供方               |
| 12   | **DoubleToIntFunction**接受一个double类型输入，返回一个int类型结果。 |
| 13   | **DoubleToLongFunction**接受一个double类型输入，返回一个long类型结果 |
| 14   | **DoubleUnaryOperator**接受一个参数同为类型double,返回值类型也为double 。 |
| 15   | **Function<T,R>** 接受一个输入参数，返回一个结果。            |
| 16   | **IntBinaryOperator**接受两个参数同为类型int,返回值类型也为int 。 |
| 17   | **IntConsumer**接受一个int类型的输入参数，无返回值 。        |
| 18   | **IntFunction<R>** 接受一个int类型输入参数，返回一个结果 。   |
| 19   | **IntPredicate**接受一个int输入参数，返回一个布尔值的结果。 |
| 20   | **IntSupplier**无参数，返回一个int类型结果。                 |
| 21   | **IntToDoubleFunction**接受一个int类型输入，返回一个double类型结果 。 |
| 22   | **IntToLongFunction**接受一个int类型输入，返回一个long类型结果。 |
| 23   | **IntUnaryOperator**接受一个参数同为类型int,返回值类型也为int 。 |
| 24   | **LongBinaryOperator**接受两个参数同为类型long,返回值类型也为long。 |
| 25   | **LongConsumer**接受一个long类型的输入参数，无返回值。       |
| 26   | **LongFunction<R>** 接受一个long类型输入参数，返回一个结果。  |
| 27   | **LongPredicate**R接受一个long输入参数，返回一个布尔值类型结果。 |
| 28   | **LongSupplier**无参数，返回一个结果long类型的值。           |
| 29   | **LongToDoubleFunction**接受一个long类型输入，返回一个double类型结果。 |
| 30   | **LongToIntFunction**接受一个long类型输入，返回一个int类型结果。 |
| 31   | **LongUnaryOperator**接受一个参数同为类型long,返回值类型也为long。 |
| 32   | **ObjDoubleConsumer<T>** 接受一个object类型和一个double类型的输入参数，无返回值。 |
| 33   | **ObjIntConsumer<T>** 接受一个object类型和一个int类型的输入参数，无返回值。 |
| 34   | **ObjLongConsumer<T>** 接受一个object类型和一个long类型的输入参数，无返回值。 |
| 35   | **Predicate<T>** 接受一个输入参数，返回一个布尔值结果。       |
| 36   | **Supplier<T>** 无参数，返回一个结果。                        |
| 37   | **ToDoubleBiFunction<T,U>** 接受两个输入参数，返回一个double类型结果 |
| 38   | **ToDoubleFunction<T>** 接受一个输入参数，返回一个double类型结果 |
| 39   | **ToIntBiFunction<T,U>** 接受两个输入参数，返回一个int类型结果。 |
| 40   | **ToIntFunction<T>** 接受一个输入参数，返回一个int类型结果。  |
| 41   | **ToLongBiFunction<T,U>** 接受两个输入参数，返回一个long类型结果。 |
| 42   | **ToLongFunction<T>** 接受一个输入参数，返回一个long类型结果。 |
| 43   | **UnaryOperator<T>** 接受一个参数为类型T,返回值类型也为T。    |



## 方法引用与构造器引用

### 方法引用

当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！（实现抽象方法的参数列表，必须与方法引用方法的参数列表保持一致！）
方法引用：使用操作符“::”将方法名和对象或类的名字分隔开来。

#### 使用情况

如下三种主要使用情况：

- 对象::实例方法
- 类::静态方法
- 类::实例方法



#### 注意事项

  	 ①方法引用所引用的方法的参数列表与返回值类型，需要与函数式接口中抽象方法的参数列表和返回值类型保持一致！
  	 ②若Lambda 的参数列表的第一个参数，是实例方法的调用者，第二个参数(或无参)是实例方法的参数时，格式： `ClassName::MethodName`



#### 使用示例

```java
@Test
public void test8 () {
    System.out.println("---方法引用  对象的引用 :: 实例方法名---");
    Student student = new Student("小明", 22, Student.Gender.MAN);
    Supplier<String> supplier = () -> student.getName();
    //使用方法引用，可以进行如下改写
    Supplier<String> supplier2 = student::getName;
    String s = supplier.get();
    String s2 = supplier2.get();
    System.out.println(s + " " + s2 + " " + "是否相同：" + s.equals(s2));

    System.out.println("---方法引用  类名 :: 静态方法名---");
    Comparator<Integer> com = ( x, y ) -> Integer.compare(x, y);
    //改写
    Comparator<Integer> com2 = Integer::compare;
    com2.compare(1, 2);

    System.out.println("---方法引用 类名 :: 实例方法名 ---");
    BiPredicate<String, String> bp = (x, y) -> x.equals(y);
    System.out.println(bp.test("abcde", "abcde"));
    //改写
    BiPredicate<String, String> bp2 = String::equals;
    System.out.println(bp2.test("abc", "abc"));
}
```



### 构造器引用

格式：`ClassName::new`
与函数式接口相结合，自动与函数式接口中方法兼容。

注意事项：

可以把构造器引用赋值给定义的方法，与构造器参数列表要与接口中抽象方法的参数列表一致！

#### 示例

```java
@Test
public void test9 () {
    System.out.println("---测试构造器引用---");
    Function<String, String> fun = String::new;
    String s = fun.apply("测试");
    System.out.println(s);

    //使用无参构造器引用，创建对象
    Supplier<Student> student = Student::new;
}
```



### 数组引用

格式：`type[] :: new`

#### 示例

```java
@Test
public void test10() {
    System.out.println("---测试数组引用---");
    Function<Integer, String[]> fun = String[]::new;
    String[] strs = fun.apply(10);
    System.out.println(strs.length);
}
```



# Stream API

## Stream简介

Stream 是Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。使用Stream API 对集合数据进行操作，就类似于使用SQL 执行的数据库查询。也可以使用Stream API 来并行执行操作。简而言之，Stream API 提供了一种高效且易于使用的处理数据的方式。



## 什么是Stream？

> Stream是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。*“集合讲的是数据，流讲的是计算！”*



## 注意事项

- Stream 自己不会存储元素。
- Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
- Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。



## Stream的使用

### Stream操作的三个步骤

1. 创建Stream
   一个数据源（如：集合、数组），获取一个流
2. 中间操作
   一个中间操作链，对数据源的数据进行处理
3. 终止操作(终端操作)
   一个终止操作，执行中间操作链，并产生结果

#### 图解Stream的操作步骤

![image-20200620202143071](https://gitee.com/koala010/typora/raw/master/img/20200726214137.png)



#### 创建Stream

##### 1.通过集合创建Stream

Java8 中的Collection 接口被扩展，提供了两个获取流的方法：

- `default Stream<E> stream()` : 返回一个顺序流
- `default Stream<E> parallelStream()` : 返回一个并行流



##### 2.通过数组创建Stream

Java8 中的Arrays 的静态方法stream() 可以获取数组流：

- `static <T> Stream<T> stream(T[] array)`: 返回一个流

重载形式，能够处理对应基本类型的数组：

- `public static IntStream stream(int[] array)`
- `public static LongStream stream(long[] array)`
- `public static DoubleStream stream(double[] array)`



##### 3.通过Stream的静态方法of()创建Stream

可以调用Stream类静态方法of(), 通过显示值创建一个流。它可以接收任意数量的参数。

- `public static<T> Stream<T> of(T... values) `: 返回一个流



##### 4.创建无限流

可以使用静态方法Stream.iterate() 和Stream.generate(),创建无限流。

- 迭代
  `public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)`
- 生成
  `public static<T> Stream<T> generate(Supplier<T> s)`

##### 示例

```java
@Test
public void test1 () {
    //1. Collection 提供了两个方法  stream() 与 parallelStream()
    List<String> list = new ArrayList<>();
    //获取一个顺序流
    Stream<String> stream = list.stream();
    //获取一个并行流
    Stream<String> parallelStream = list.parallelStream();

    //2. 通过 Arrays 中的 stream() 获取一个数组流
    Integer[] nums = new Integer[10];
    Stream<Integer> stream1 = Arrays.stream(nums);

    //3. 通过 Stream 类中静态方法 of()
    Stream<Integer> stream2 = Stream.of(1,2,3,4,5,6);

    //4. 创建无限流
    //迭代
    Stream<Integer> stream3 = Stream.iterate(0, (x) -> x + 2).limit(10);
    stream3.forEach(System.out::println);

    //生成
    Stream<Double> stream4 = Stream.generate(Math::random).limit(2);
    stream4.forEach(System.out::println);
}
```



#### Stream的中间操作

**多个中间操作**可以连接起来形成一个**流水线**，除非流水线上触发终止操作，否则**中间操作不会执行任何的处理！**
**而在终止操作时一次性全部处理，称为“惰性求值”。**



新建一个学生类，方便测试。

```java
public class Student {

    private String name;
    private int age;
    private Gender gender;

    public Student () {
    }

    public Student ( String name, int age, Gender gender ) {
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    public String getName () {
        return name;
    }

    public void setName ( String name ) {
        this.name = name;
    }

    public int getAge () {
        return age;
    }

    public void setAge ( int age ) {
        this.age = age;
    }

    public Gender getGender () {
        return gender;
    }

    public void setGender ( Gender gender ) {
        this.gender = gender;
    }

    @Override
    public boolean equals ( Object o ) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Student student = (Student) o;
        return age == student.age &&
                Objects.equals(name, student.name) &&
                gender == student.gender;
    }

    @Override
    public int hashCode () {
        return Objects.hash(name, age, gender);
    }

    @Override
    public String toString () {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", gender=" + gender +
                '}';
    }

    /** 性别枚举 */
    public enum Gender{
        /** 男 */
        MAN,
        /** 女 */
        WOMAN;
    }

}
```

创建一个集合方便测试

```java
List<Student> studentList = Arrays.asList(
        new Student("李四", 59, Student.Gender.MAN),
        new Student("张三", 18, Student.Gender.MAN),
        new Student("王五", 28, Student.Gender.MAN),
        new Student("赵小苗", 8, Student.Gender.WOMAN),
        new Student("赵小苗", 8, Student.Gender.WOMAN),
        new Student("李芳", 8, Student.Gender.WOMAN)
);
```



##### 1.筛选与切片

| 方法                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| filter(Predicate p) | 接收Lambda ，从流中排除某些元素。                            |
| distinct()          | 筛选，通过流所生成元素的hashCode() 和equals() 去除重复元素   |
| limit(long maxSize) | 截断流，使其元素不超过给定数量。                             |
| skip(long n)        | 跳过元素，返回一个扔掉了前n 个元素的流。若流中元素不足n 个，则返回一个空流。与limit(n) 互补 |



###### 使用示例

```java
@Test
public void test2 () {
    System.out.println("---测试filter，过滤信息---");
    Stream<Student> studentStream = studentList.stream()
            .filter(s -> s.getAge() > 20);
    studentStream.forEach(System.out::println);

    System.out.println("---测试limit,取前1个元素---");
    studentList.stream()
            .filter((e) -> e.getAge() >= 20)
            .limit(1)
            .forEach(System.out::println);

    System.out.println("---测试skip，跳过1个元素---");
    studentList.stream()
            .filter((e) -> e.getAge() >= 20)
            .skip(1)
            .forEach(System.out::println);

    System.out.println("---测试distinct，去重---");
    studentList.stream()
            .distinct()
            .forEach(System.out::println);
}
```

结果：

```java
---测试filter，过滤信息---
Student{name='李四', age=59, gender=MAN}
Student{name='王五', age=28, gender=MAN}
---测试limit,取前1个元素---
Student{name='李四', age=59, gender=MAN}
---测试skip，跳过1个元素---
Student{name='王五', age=28, gender=MAN}
---测试distinct，去重---
Student{name='李四', age=59, gender=MAN}
Student{name='张三', age=18, gender=MAN}
Student{name='王五', age=28, gender=MAN}
Student{name='赵小苗', age=8, gender=WOMAN}
```



##### 2.映射

| 方法                            | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| map(Function f)                 | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的素。 |
| mapToDouble(ToDoubleFunction f) | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新DoubleStream。 |
| mapToInt(ToIntFunction f)       | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的IntStream。 |
| mapToLong(ToLongFunction f)     | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的LongStream。 |
| flatMap(Function f)             | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |



###### 使用示例

```java
public class StreamTest {

    @Test
    public void test3 () {
        Stream<String> stringStream = studentList.stream()
                .map(Student::getName);
        stringStream.forEach(System.out::println);

        //map与flatMap的区别    与list的add、addAll方法区别类似
        List<String> strList = Arrays.asList("aaa", "bbb", "ccc", "ddd", "eee");

        Stream<String> stream = strList.stream()
                .map(String::toUpperCase);

        stream.forEach(System.out::println);

        Stream<Stream<Character>> stream2 = strList.stream()
                .map(StreamTest::filterCharacter);

        stream2.forEach((sm) -> {
            sm.forEach(System.out::println);
        });

        System.out.println("---------------------------------------------");
        Stream<Character> stream3 = strList.stream()
                .flatMap(StreamTest::filterCharacter);
        
        stream3.forEach(System.out::println);
    }

    private static Stream<Character> filterCharacter(String str){
        List<Character> list = new ArrayList<>();

        for (Character ch : str.toCharArray()) {
            list.add(ch);
        }
        return list.stream();
    }

}
```



##### 3.排序

| 方法                   | 描述                               |
| ---------------------- | ---------------------------------- |
| sorted()               | 产生一个新流，其中按自然顺序排序   |
| sorted(Comparatorcomp) | 产生一个新流，其中按比较器顺序排序 |

###### 使用示例

```java
@Test
public void test4 () {
    System.out.println("---自然排序---");
    studentList.stream()
            .map(Student::getName)
            .sorted()
            .forEach(System.out::println);

    System.out.println("---定制排序---");
    studentList.stream()
            .sorted((x,y)->{
                if(x.getAge() == y.getAge()){
                    return x.getName().compareTo(y.getName());
                }else{
                    return Integer.compare(x.getAge(), y.getAge());
                }
            })
            .forEach(System.out::println);

}
```

结果：

```java
---自然排序---
张三
李四
李芳
王五
赵小苗
赵小苗
---定制排序---
Student{name='李芳', age=8, gender=WOMAN}
Student{name='赵小苗', age=8, gender=WOMAN}
Student{name='赵小苗', age=8, gender=WOMAN}
Student{name='张三', age=18, gender=MAN}
Student{name='王五', age=28, gender=MAN}
Student{name='李四', age=59, gender=MAN}
```



#### 终止操作

流进行了终止操作后，不能再次使用

##### 1.查找与匹配

| 方法                  | 描述                     |
| --------------------- | ------------------------ |
| allMatch(Predicate p) | 检查是否匹配所有元素     |
| anyMatch(Predicate p) | 检查是否至少匹配一个元素 |
| noneMatch(Predicatep) | 检查是否没有匹配所有元素 |
| findFirst()           | 返回第一个元素           |
| findAny()             | 返回当前流中的任意元素   |
| count()               | 返回流中元素总数         |
| max(Comparatorc)      | 返回流中最大值           |
| min(Comparatorc)      | 返回流中最小值           |
| forEach(Consumerc)    | 内部迭代                 |

*注：forEach(Consumerc)，内部迭代(使用Collection 接口需要用户去做迭代，称为外部迭代。相反，Stream API 使用内部迭代——它帮你把迭代做了)。*

#### 示例

```java
@Test
public void test5 () {
    boolean b1 = studentList.stream()
            .allMatch(e -> e.getGender().equals(Student.Gender.WOMAN));
    System.out.println("是否有女学生：" + b1);

    Optional<Student> first = studentList.stream()
            .sorted(( x, y ) -> {
                return x.getAge() - y.getAge();
            })
            .findFirst();
    System.out.println("得到年龄最小的学生：\n" + first.get());

    long count = studentList.stream()
            .filter(e->e.getGender().equals(Student.Gender.WOMAN))
            .count();
    System.out.println("女生数量：" + count);
}
```

结果：

```java
是否有女学生：false
得到年龄最小的学生：
Student{name='赵小苗', age=8, gender=WOMAN}
女生数量：3
```



##### 2.规约

| 方法                             | 描述                                                    |
| -------------------------------- | ------------------------------------------------------- |
| reduce(T iden, BinaryOperator b) | 可以将流中元素反复结合起来，得到一个值。返回T           |
| reduce(BinaryOperator b)         | 可以将流中元素反复结合起来，得到一个值。返回Optional<T> |

*注：map 和reduce 的连接通常称为map-reduce 模式，因Google 用它来进行网络搜索而出名。*

```java
@Test
public void test6 () {
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

    Integer sum = list.stream()
            .reduce(0, ( x, y ) -> x + y);

    System.out.println(sum);

    System.out.println("----------------------------------------");

    Optional<Integer> op = studentList.stream()
            .map(Student::getAge)
            .reduce(Integer::sum);

    System.out.println("所有学生年龄之和" + op.get());
}
```

结果：

```
55
----------------------------------------
所有学生年龄之和129
```



##### 3.收集

| 方法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| collect(Collector c) | 将流转换为其他形式。接收一个Collector接口的实现，用于给Stream中元素做汇总的方法 |

Collector 接口中方法的实现决定了如何对流执行收集操作(如收集到List、Set、Map)。但是Collectors 类提供了很多静态方法，可以方便地创建常见收集器实例，具体方法与实例如下表：

| 序号 | 方法              | 返回类型             | 描述                                                         |
| ---- | ----------------- | -------------------- | ------------------------------------------------------------ |
| 1    | toList            | List<T>              | 把流中元素收集到List                                         |
| 2    | toSet             | Set<T>               | 把流中元素收集到Set                                          |
| 3    | toCollection      | Collection<T>        | 把流中元素收集到创建的集合                                   |
| 4    | counting          | Long                 | 计算流中元素的个数                                           |
| 5    | summingInt        | Integer              | 对流中元素的整数属性求和                                     |
| 6    | averagingInt      | Double               | 计算流中元素Integer属性的平均值                              |
| 7    | summarizingInt    | IntSummaryStatistics | 收集流中Integer属性的统计值。如：平均值                      |
| 8    | joining           | String               | 连接流中每个字符串                                           |
| 9    | maxBy             | Optional<T>          | 根据比较器选择最大值                                         |
| 10   | minBy             | Optional<T>          | 根据比较器选择最小值                                         |
| 11   | reducing          | 归约产生的类型       | 从一个作为累加器的初始值开始，利用BinaryOperator与<br/>流中元素逐个结合，从而归约成单个值 |
| 12   | collectingAndThen | 转换函数返回的类型   | 包裹另一个收集器，对其结果转换函数                           |
| 13   | groupingBy        | Map<K,List<T>>       | 根据某属性值对流分组，属性为K，结果为V                       |
| 14   | partitioningBy    | Map<Boolean,List<T>> | 根据true或false进行分区                                      |

###### 使用示例：

```java
@Test
public void test7 () {
    //测试收集collect
    List<String> list = studentList.stream()
            .map(Student::getName)
            .collect(Collectors.toList());
    System.out.println("学生姓名转集合：" + list);

    Set<String> set = studentList.stream()
            .map(Student::getName)
            .collect(Collectors.toSet());
    System.out.println("学生姓名转Set，去重：" + set);

    //求最小的学生年龄
    Optional<Integer> minAge = studentList.stream()
            .map(Student::getAge)
            .collect(Collectors.minBy(Integer::compare));
    System.out.println("求最小的学生年龄：" + minAge.get());

    System.out.println("---根据性别分组---");
    Map<Student.Gender, List<Student>> map = studentList.stream()
            .collect(Collectors.groupingBy(Student::getGender));
    System.out.println(map);

    System.out.println("---根据年龄和性别多级分组---");
    Map<Student.Gender, Map<String, List<Student>>> map2=studentList.stream()
            .collect(Collectors.groupingBy(Student::getGender,Collectors.groupingBy(e->{
                if(e.getAge() >= 60) {
                    return "老年";
                } else if(e.getAge() >= 35) {
                    return "中年";
                } else {
                    return "成年";
                }
            })));
    System.out.println(map2);

    System.out.println("---根据年龄分区---");
    Map<Boolean, List<Student>> map3 = studentList.stream()
            .collect(Collectors.partitioningBy((e) -> e.getAge() >= 30));
    System.out.println(map3);

    System.out.println("连接流的字符串");
    String str = studentList.stream()
            .map(Student::getName)
            .collect(Collectors.joining("," , "----", "----"));
    System.out.println(str);

    System.out.println("规约收集，求所有学生年龄的和");
    Optional<Integer> sum = studentList.stream()
            .map(Student::getAge)
            .collect(Collectors.reducing(Integer::sum));
    System.out.println(sum.get());
}
```

结果：

```java
学生姓名转集合：[李四, 张三, 王五, 赵小苗, 赵小苗, 李芳]
学生姓名转Set，去重：[李四, 张三, 赵小苗, 王五, 李芳]
求最小的学生年龄：8
---根据性别分组---
{WOMAN=[Student{name='赵小苗', age=8, gender=WOMAN}, Student{name='赵小苗', age=8, gender=WOMAN}, Student{name='李芳', age=8, gender=WOMAN}], MAN=[Student{name='李四', age=59, gender=MAN}, Student{name='张三', age=18, gender=MAN}, Student{name='王五', age=28, gender=MAN}]}
---根据年龄和性别多级分组---
{WOMAN={成年=[Student{name='赵小苗', age=8, gender=WOMAN}, Student{name='赵小苗', age=8, gender=WOMAN}, Student{name='李芳', age=8, gender=WOMAN}]}, MAN={成年=[Student{name='张三', age=18, gender=MAN}, Student{name='王五', age=28, gender=MAN}], 中年=[Student{name='李四', age=59, gender=MAN}]}}
---根据年龄分区---
{false=[Student{name='张三', age=18, gender=MAN}, Student{name='王五', age=28, gender=MAN}, Student{name='赵小苗', age=8, gender=WOMAN}, Student{name='赵小苗', age=8, gender=WOMAN}, Student{name='李芳', age=8, gender=WOMAN}], true=[Student{name='李四', age=59, gender=MAN}]}
连接流的字符串
----李四,张三,王五,赵小苗,赵小苗,李芳----
规约收集，求所有学生年龄的和
129
```



## 并行流与串行流

> 并行流就是把一个内容分成多个数据块，并用不同的线程分别处理每个数据块的流。

Java 8 中将并行进行了优化，我们可以很容易的对数据进行并行操作。Stream API 可以声明性地通过`parallel() `与`sequential() `在并行流与顺序流之间进行切换。



### 了解Fork/Join 框架

> Fork/Join 框架：就是在必要的情况下，将一个大任务，进行拆分(fork)成若干个小任务（拆到不可再拆时），再将一个个的小任务运算的结果进行join 汇总.

#### 图解

![image-20200620212853966](https://gitee.com/koala010/typora/raw/master/img/20200726214151.png)



### Fork/Join 框架与传统线程池的区别

采用“工作窃取”模式（work-stealing）：

当执行新的任务时它可以将其拆分分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。

相对于一般的线程池实现,fork/join框架的优势体现在对其中包含的任务处理方式上。

在一般的线程池中,如果一个线程正在执行的任务由于某些原因无法继续运行,那么该线程会处于等待状态.

而在fork/join框架实现中,如果某个子问题由于等待另外一个子问题的完成而无法继续运行。那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行。

这种方式减少了线程的等待时间,提高了性能。



# Optional类

## Optional类的由来？解决空指针问题

到目前为止，臭名昭著的空指针异常是导致Java应用程序失败的最常见原因。

以前，为了解决空指针异常，Google公司著名的Guava项目引入了Optional类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。

受到Google Guava的启发，Optional类已经成为Java 8类库的一部分。



## Optional类简介

> Optional<T> 类(java.util.Optional) 是一个容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null，表示这个值不存在。原来用null 表示一个值不存在，现在Optional 可以更好的表达这个概念。并且可以避免空指针异常。

Optional类的Javadoc描述如下：

这是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional提供很多有用的方法，这样我们就不用显式进行空值检测。



## Optional类的使用

### 创建Optional类对象

- `Optional.of(T t) `: 创建一个Optional 实例，**t必须非空**
- `Optional.empty()` : 创建一个空的Optional 实例
- `Optional.ofNullable(T t)`：若 t 不为 null,创建 Optional 实例,否则创建空实例（**t可以为null**）

### 判断Optional容器中是否包含对象

- `boolean isPresent()` : 判断是否包含对象
- `void ifPresent(Consumer<? super T> consumer) `：如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它。

### 获取Optional容器的对象

- `T get()`: 如果调用对象包含值，返回该值，否则抛异常
- `T orElse(T other)` ：如果有值则将其返回，否则返回指定的other对象。
- `T orElseGet(Supplier<? extends T> other)` ：如果有值则将其返回，否则返回由Supplier接口实现提供的对象。
- `T orElseThrow(Supplier<? extends X> exceptionSupplier)` ：如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。

### 其他

- `map(Function f)`: 如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty()

 * 	`flatMap(Function mapper)`:与 map 类似，要求返回值必须是Optional



### 示例

```java
@Test
public void test1 () {
    //测试创建Optional
    Optional<Student> op = Optional.of(new Student( "张三", 18, Student.Gender.MAN));

    Optional<String> s = op.map(Student::getName);
    String name = s.get();
    System.out.println(name);

    Optional<Student> op1 = Optional.ofNullable(null);
    if (op1.isPresent()){
        System.out.println(op1.get());
    }else{
        System.out.println("此对象为空");
    }
    System.out.println(op1.orElse(new Student( "默认", 18, Student.Gender.MAN)));
    
}
```

结果：

```java
张三
此对象为空
Student{name='默认', age=18, gender=MAN}
```



# 新时间日期API

传统时间API存在线程安全问题，多下程下会出现问题。

## LocalDate、LocalTime、LocalDateTime 

`LocalDate`、`LocalTime`、`LocalDateTime` 类的实例是不可变的对象，分别表示使用ISO-8601日历系统的日期、时间、日期和时间。它们提供了简单的日期或时间，并不包含当前的时间信息。也不包含与时区相关的信息。

*注：ISO-8601日历系统是国际标准化组织制定的现代公民的日期和时间的表示法*



### 使用

`LocalDate`、`LocalTime`、`LocalDateTime`的使用方法基本一致，只是代表的是系统的日期、时间、日期和时间。



```java
@Test
public void test(){
    //获取系统的时间
    LocalDateTime ldt = LocalDateTime.now();
    System.out.println(ldt);
    //根据指定日期/时间创建
    LocalDateTime ldt2 = LocalDateTime.of(2016, 12, 25, 12, 12, 12);
    System.out.println(ldt2);
    //向当前对象添加天、周、月.....   plusXxx()
    LocalDateTime ldt3 = ldt2.plusDays(3).plusMonths(2);
    System.out.println(ldt3);
    //向当前对象减去天、周、月....   minusXxx()
    LocalDateTime ldt4 = ldt2.minusYears(1);
    System.out.println(ldt4);
    //添加一个Duration或者Period   plus()/minus()

    //修改对象指定的月、天、周、年.....  withXxx()
    LocalDateTime ldt5 = ldt2.withDayOfMonth(1);
    System.out.println(ldt5);

    //获取指定时间/日期对象的年月日....  get()
    int year = ldt.getYear();
    System.out.println(year);
}
```

结果：

```
2020-06-23T21:05:40.207
2016-12-25T12:12:12
2017-02-28T12:12:12
2015-12-25T12:12:12
2016-12-01T12:12:12
2020
```



## Instant 时间戳

用于“时间戳”的运算。它是以Unix元年(传统的设定为UTC时区1970年1月1日午夜时分)开始所经历的描述进行运算。



### 使用

```java
@Test
public void test2 () {
    //获取当前系统时间戳   2020-06-23T12:34:41.555Z(还是时间的格式,默认是UTC时区)
    Instant instant = Instant.now();
    System.out.println(instant);
    //利用偏移量，设置为东八区
    OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
    System.out.println(offsetDateTime);
    //获取毫秒形式的时间戳值
    System.out.println("秒数" + instant.getEpochSecond());
    System.out.println("毫秒数" + instant.toEpochMilli());
    //指定秒数获取instant实例
    Instant instant1 = Instant.ofEpochSecond(instant.getEpochSecond());
    System.out.println(instant1);
}
```

结果：

```
2020-06-23T13:05:27.201Z
2020-06-23T21:05:27.201+08:00
秒数1592917527
毫秒数1592917527201
2020-06-23T13:05:27Z
```



## Duration 和Period

`Duration`:用于计算两个“时间”间隔
`Period`:用于计算两个“日期”间隔



### 使用

```java
@Test
public void test3 () {
    Instant ins1 = Instant.now();
    System.out.println("--------------------");
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    Instant ins2 = Instant.now();
    System.out.println("所耗费时间为：" + Duration.between(ins1, ins2));

    System.out.println("----------------------------------");
    LocalDate ld1 = LocalDate.now();
    LocalDate ld2 = LocalDate.of(2011, 1, 1);
    Period pe = Period.between(ld2, ld1);
    System.out.println(pe.getYears());
    System.out.println(pe.getMonths());
    System.out.println(pe.getDays());
}
```

结果：

```
--------------------
所耗费时间为：PT1.001S
----------------------------------
9
5
22
```



## TemporalAdjuster(时间校正器)

`TemporalAdjuster` : 时间校正器。有时我们可能需要获取例如：将日期调整到“下个周日”等操作。
`TemporalAdjusters` : 该类通过静态方法提供了大量的常用`TemporalAdjuster `的实现。



### 使用

```java
@Test
public void test4 () {
    LocalDateTime ldt = LocalDateTime.now();
    LocalDateTime ldt1 = ldt.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));
    System.out.println(ldt + "\n" + ldt1);

    //自定义：下一个工作日
    LocalDateTime ldt3 = ldt.with((l) -> {
        LocalDateTime ldt2 = (LocalDateTime) l;
        DayOfWeek dow = ldt2.getDayOfWeek();
        if(dow.equals(DayOfWeek.FRIDAY)){
            return ldt2.plusDays(3);
        }else if(dow.equals(DayOfWeek.SATURDAY)){
            return ldt2.plusDays(2);
        }else{
            return ldt2.plusDays(1);
        }
    });
    System.out.println(ldt3);
}
```

结果：

```
2020-06-23T21:06:10.163
2020-06-28T21:06:10.163
2020-06-24T21:06:10.163
```



## DateTimeFormatter(解析和格式化日期或时间)

`java.time.format.DateTimeFormatter `类：该类提供了三种格式化方法：

- 预定义的标准格式
- 语言环境相关的格式
- 自定义的格式

### 使用

```java
@Test
public void test5 () {
    //DateTimeFormatter dtf = DateTimeFormatter.ISO_LOCAL_DATE;
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss E");
    LocalDateTime ldt = LocalDateTime.now();
    String strDate = ldt.format(dtf);
    System.out.println(strDate);

    LocalDateTime newLdt = LocalDateTime.parse("2020-02-03 15:56:23", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    System.out.println(newLdt);
}
```

结果

```
2020年06月23日 21:02:28 星期二
2020-02-03T15:56:23
```



## ZonedDate、ZonedTime、ZonedDateTime时区处理

Java8 中加入了对时区的支持，带时区的时间为分别为：`ZonedDate`、`ZonedTime`、`ZonedDateTime`

其中每个时区都对应着ID，地区ID都为“{区域}/{城市}”的格式
例如：`Asia/Shanghai `等

`ZoneId`：该类中包含了所有的时区信息
`getAvailableZoneIds()`: 可以获取所有时区时区信息
`of(id) `: 用指定的时区信息获取ZoneId 对象



### 使用：

```java
@Test
public void test6 () {
    //获取所有失去的set集合
    Set<String> set = ZoneId.getAvailableZoneIds();
    //ZonedDate、ZonedTime、ZonedDateTime ： 带时区的时间或日期
    LocalDateTime ldt = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));
    System.out.println(ldt);

    ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("US/Pacific"));
    System.out.println(zdt);

}
```

结果：

```
2020-06-23T21:11:03.112
2020-06-23T06:11:03.114-07:00[US/Pacific]
```



## Clock

Clock 时钟类用于获取当时的时间戳，或当前时区下的日期时间信息。以前用到System.currentTimeInMillis() 和 TimeZone.getDefault() 的地方都可用Clock替换。



`instant()`和`millis()`返回的并不是当前系统的时间营，应该是UTC的时间。但可以获取当前系统的默认时区。

### 使用

```java
@Test
public void test7 () {
    //获取系统默认时区 (当前瞬时时间 )
    Clock clock1 = Clock.systemDefaultZone();
    System.out.println("系统时区" + clock1);
    System.out.println("系统时间日期：" + clock1.instant());
    System.out.println("时间毫秒：" + clock1.millis());
    System.out.println(clock1.getZone());
    System.out.println("-------------------");
    //获取系统时钟，并将其转换成使用UTC时区的日期和时间
    Clock clock = Clock.systemUTC();
    System.out.println("时间日期：" + clock.instant());
    System.out.println("时间毫秒值：" + clock.millis());
    System.out.println("-------------");
    Clock clock2 = Clock.system(ZoneId.of("Asia/Shanghai"));
    System.out.println(clock2);
}
```

结果

```
系统时区SystemClock[Asia/Shanghai]
系统时间日期：2020-06-23T13:34:22.764Z
时间毫秒：1592919262792
Asia/Shanghai
-------------------
时间日期：2020-06-23T13:34:22.792Z
时间毫秒值：1592919262792
-------------
SystemClock[Asia/Shanghai]
```



# 接口中的默认方法与静态方法

Java 8中允许接口中包含具有具体实现的方法，该方法称为“默认方法”，默认方法使用`default `关键字修饰。



## 接口默认方法的”类优先”原则

- 若一个接口中定义了一个默认方法，而另外一个父类或接口中又定义了一个同名的方法时选择父类中的方法。如果一个父类提供了具体的实现，那么接口中具有相同名称和参数的默认方法会被忽略。

- 接口冲突。如果一个父接口提供一个默认方法，而另一个接口也提供了一个具有相同名称和参数列表的方法（不管方法是否是默认方法），那么必须覆盖该方法来解决冲突



# 重复注解与类型注解

Java 8对注解处理提供了两点改进：可重复的注解及可用于类型的注解。



## 使用

```java
@Repeatable(MyAnnotations.class)
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE,TYPE_PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String value();
}
```

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE,TYPE_PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotations {
    MyAnnotation[] value();
}
```

```java
@MyAnnotation("哈哈")
@MyAnnotation("嘿嘿")
public void show(){
   System.out.println("测试重复注解");
}
```

```java
public class MyAnnationTest {

    @MyAnnotation("哈哈")
    @MyAnnotation("嘿嘿")
    public void show () {
        System.out.println("测试重复注解");
    }

    @Test
    public void test () throws Exception {
        Class<MyAnnationTest> clazz = MyAnnationTest.class;
        Method method = clazz.getDeclaredMethod("show");
        MyAnnotation[] annotations = method.getAnnotationsByType(MyAnnotation.class);
        System.out.println("---获取注解信息---");
        for (MyAnnotation annotation : annotations) {
            System.out.println(annotation.value());
        }
    }

}
```

结果：

```
---获取注解信息---
哈哈
嘿嘿
```

