
# 基本语法

## 命名约定

类：首字母大写

方法（method）、变量：首字母小写

## 文件结构

源代码的文件名必须与公共类的名字相同（扩展名：.java）。

### 包

包用于将类组织到一个集合中，以逆序域名命名，并放置源文件于对应的基于类路径的相对子目录，如：./com/mw2566/handsome/(*.java|.class)

`jar`文件就是`class`文件的容器。为方便解决`jar`之间的依赖关系，模块（`.jmod`）内含有相关的依赖关系，并方便部署至JRE上。

###  注释与javadoc

#### 自由格式文本

支持html标签。

支持无须转义的块。{@code … }

#### 注释

类注释放在import 语句后，类定义之前。字段、方法类似。

方法注释含：

`@param` variable description

`@return` description

`@throw` error comment

通用注释含：

`@author` `@version`   `@see` `@link`

#### 包注释

需要在包目录添加文件：`package-info.java`，内含`/** comment */`后接package语句。

## 数据类型

### char

character涵盖Unicode字符，char类型描述UTF-16编码中的一个代码单元。因此建议不要在程序中使用char类型。

### String

==S==tring是一个预定义类，允许使用`+`表示拼接。与字符数组不同的是，该类没有提供修改特定字符的方法，而应该：`str = str.substring(0,3) + "sb";`

`==`表示两个字符串是否指向同一位置，**而非其内容相等**，请使用equals方法。

串值为null代表未关联串对象，不代表空串。

未初始化地声明对象数组时，元素均对应nul。

### boolean

请注意，整数值和布尔值之间不允许进行类型转换。

### 声明

请注意，仅对于基本类型（数字类不是对象），变量名指向其本身；而对于对象类型，变量名视作**对象指针**，只是其的一个“管理者”，不直接控制对象的状态变化。

Java中不区分变量的声明和定义。

可以用`var`从方法中的局部变量初始化值推断类型，效果似C++的`auto`。

## 操作符

### 运算符

`>>>`： 逻辑右移

`>>`：算术右移

### 常用类

#### Math

`floormod`：对于除数为正者，总是获取正模值

#### 输入输出

```java
import java.util.*;

Scanner in = new Scanner(system.in);
//Scanner in = new Scanner(Path.of("src.txt"),StandardCharsets=UTF_8);
String str = in.next();
int i = in.nextInt();
system.out.print("No endl" + "this way");
```

### 标签控制流

`break`函数可将函数执行流跳转至带标签语句块**末尾**；

# 面向对象

## 类 

### 实例

final字段必须在构造器执行时初始化，并且以后不能再修改。

### 方法

更改器方法：调用后，调用者对象状态发生改变；

访问器方法：只访问对象，不修改对象（对应C++带有const后缀的方法）

Java中的方法参数均为call by *value*，不能修改参数变量的内容。可以调用方法以修改内容。**传入对象时，方法得到的是对象引用的副本，原来的对象引用和这个副本都引用同一个对象。** 即，可以改变对象参数的状态，不能让一个对象参数引用一个新的对象。

> 如果需要返回一个可变对象的引用，首先应该对它进行克隆（clone)。

重载（overload）：同方法名，不同的方法参数。

#### 静态字段与静态方法

`static`定义的字段，说明每个类只有一个这样的字段，其下所有实例共享有之。

静态方法不需要在对象上执行，调用时是否加入对象均可。

#### 构造器

this指向所构造的对象，也可在一个构造器中，调用类的另一个构造器。

### 初始化块

```java
public class aClass{
    private static int cnt;
    private String name = "";
    static{//initialization block（shared）
        var generator = new Random();
        cnt = generator.nextInt(10000);
    }
    {//Object initialization block
        cnt++;
    }
    //Constuctor...    
}
```

## 类间关系

- 依赖（uses a）通常以方法参数、局部变量的、静态方法调用的形式出现
- 聚合（has a）
- 继承（is a）箭头指向父类

![](http://img.070077.xyz/202205071305277.png)

关联关系是指一种拥有的关系，一个类知道另一个类的属性和方法，两个类处于同一个层次上。代码体现：成员变量。

聚合是一种弱的‘拥有’关系，体现的是 A 对象可以包含 B 对象，但 B 对象不是 A 对象的一部分，处于不平等的层次上。

访问权限：
- （`+`）**public**：可以被所有其他类所访问。 
- （`-`）**private**：只能被自己访问和修改。 
- （`#`）**protected**：自身，子类及同一个包中类可以访问。

|修饰符|当前类|同一包|子类|外部包|
|---|---|---|---|---|
|public|✓|✓|✓|✓|
|protected|✓|✓|✓|✗|
|default|✓|✓|✗|✗|
|private|✓|✗|✗|✗|

- 除了在编译期，运行时，JVM 也会执行访问控制检查。
- 当代码尝试访问一个字段、方法或类时，JVM 会检查调用者是否有合适的权限。
- 如果没有，JVM 会抛出一个 `IllegalAccessException`。
- 这是通过检查调用者的类加载器和被调用的目标的类加载器以及它们的运行时包信息来完成的，确保它们满足访问修饰符的规定。


### 继承

继承（inheritance）类不能直接访问私有类的字段，需要使用**超类的**公共接口，如`super.get()`。

其中，`super`关键字指示“超类方法”。类似于`this`，其也可对应一个构造器`super(args)`，调用超类对应的构造器。

#### 多态

超类对象的任何地方都可以用子类对象替换。通常有两种实现方法：子类继承父类（extends）、类实现接口（implements）

对象变量是多态（polymorphic）的，也可以引用其对应的子类的对象。

对于声明的类型转换，**依赖关系的验证的方式是** `sub instanceof sup`

```java
if (val instanceof Var){  
    in.add((Var)val);  
}
```

#### 阻止继承

`final`修饰符表示类中的特定方法，不允许子类覆盖。

控制修饰符`protected`表示对本包和所有子类可见。

如果子类的方法重写了父类的方法，那么**子类中该方法的访问级别不允许低于父类的访问级别。** 这是为了确保可以使用父类实例的地方都可以使用子类实例去代替，也就是确保满足里氏替换原则。

#### 抽象类聚合

占位类，只可继承，不可构造，表示子类的共有属性，关键字：`abstract`。其内的抽象方法是未实现的声明，需要由类覆盖实现；也可以有具体方法，但不建议。

#### 所有对象的超类——Object

内定义以下方法，可重写覆盖原样式。

- `boolean equals(Object obj)` 检测（引用）相等。约定：重写该方法时需实现成等价关系。
- `String toString(Object obj)` 得到字符串，并直接可用`+`拼接，如`“Hello” + obj`；默认为*类名@散码*；重写可包含`super`方法，以用于日志或调试。
- `Class getClass()` 返回含对象信息的类对象。
- `protected（Object） clone()` 用于创建并返回当前对象的一份拷贝，默认操作是浅拷贝。由于是protected方法，代码不能直接调用之，对象可对于未重写的clone方法调用发出CloneNotSupportedException异常，即便是默认操作也需要使`clone`声明为`public`后，调用`super.clone()`。

> 反射库(reflection library)是运行时分析类的程序，由于不属于应用范畴，在此略过。

>  枚举(类)：各元素对应为一个实例，因此也可有私有的构造器操作。

## 泛型容器

泛型(generic)，将数据类型视作参数，类似于C++的template。

> 变参（varargs）方法：方法参数：(double… args)；args为对应的数组；用Object可不指定类型

### 数组列表/ArrayList

`var vec = new ArrayList<objE>()`，类似于C++ STL的序列式容器，详见4.1。

据经验，尽量使用接口访问。objE - 对象类型。基本类型需使用包装器(wrapper)，与之带来的是较大的性能开销。若`objE`写作`T`，视为`Object`。

对于基本类型，需转换为`Integer` `Character ` `Byte`等类的 final 对象，这些类称作包装器。

数字包装类默认创建了对应于[-128，127]缓存地址（`Character` 为 [0,127]），如果在该范围内，调用`ValueOf`不会创建新的实例。

基本类型作为方法参数和返回值时，会自动包装和解包装。

### “擦拭法”

擦拭法是指，虚拟机对泛型其实一无所知，所有的工作都是编译器做的。因此`objT`：

- 不能实例化`T`类型，例如：`new T()`；不能获取带泛型类型的`Class`
- 不能判断带泛型类型的类型，例如：`x instanceof Pair<String>`；

### 通配符

#### extend

上界通配（Upper Bounds Wildcards）：使用`Pair<? extends Number>`使得方法接收所有泛型类型为`Number`或`Number`子类的`Pair`类型。于是就可以传入`Number`的子类`Double`等。

但是要记得，泛型的实现是“擦拭法”，编译器确定了选择范围中的一个，确定后不再接受不同的类型。

该实现会影响`set`型方法，只允许传入引用`null`，即可读不可写。

#### super

`Pair<? super Integer>`表示，方法参数接受的泛型类型范围为`Integer`及其超类。

只可以`Object`接收`get`方法返回的引用，也允许`set`传入引用。除特例，可以认为，内部代码对于参数可写不可读。

>PECS原则：Producer Extends Consumer Super。
>
>即：如果需要返回`T`，它是生产者（Producer），要使用`extends`通配符；如果需要写入`T`，它是消费者（Consumer），要使用`super`通配符。

#### 无限定通配符

`<?>`是所有`<T>`的超类，只允许使用上面两个特例，可认为不可读不可写。很少使用。

>一些泛型使用上的局限：
>
>- 如果在方法内部创建了泛型数组，最好不要将它返回给外部使用。
>- 同时使用泛型和可变参数时需要特别小心。
>- 可以声明带泛型的数组，但不能直接创建带泛型的数组，必须强制转型。

## 接口

接口(interface)，即需求一个类实现(implement)的功能。

接口不是类，没有实例，包含需求的方法。接口可以有一个默认实现（会被具体实现覆盖），参下：

```java
public interface Comparable<T>{
    default int compareTo(T t){ return 0; }
    //int compareTo(T t);
}
//e.g.
public interface List<T> {
    int size(); 
    T get(int index); 
    void add(T t);
    void remove(T t);
}
//建议：Comparable接口的实现，">" "<" "=="（0）均需对应其相应的返回值。
```

> 面向接口的设计模式：先定义接口，再实现类，适合团队交接

## 浅复制和深复制

Java 的变量模型简单来说，可以把它理解成 python 那样的模型：
```
var a = {1, 2}
var b = a.remove(1);  
System.out.println(a); //{2}
```

## 内部类

内部类（inner class）是定义于另一个类里的类，达到对同包的类隐藏、访问外部类字段的目的。其可以是匿名的。具体实现略。

如果不需要访问外围类对象，可声明为`static`

# 异常

## 异常捕获

### 一些事实

- 异常的发生`!=`程序终止![image-20220205221749465](http://img.070077.xyz/image-20220205221749465.png)

### 受检查型异常

这是异常的层次结构图：

![exception-architechture-java](http://img.070077.xyz/exception-architechture-java.png)

除`Error`和`RuntimeException`类派生出来的异常外，其他异常均称为检查型(checked)异常。

- 需在首部通过`throws`声明，说明可能抛出的异常
- 现有异常可派生，创建自建异常类，包含默认构造器、*描述*构造器（超类`Throwable`定义有`toString`方法）
- 参上，进行异常传播时，可把原始异常实例作为构造抛出异常的参数，以保留案发现场。

#### 争议

`checked exception`机制需要花费大量的精力预测程序中的错误行为，并且大部分异常本身难以被处理，在函数式编程的时代理念下，这一机制被称作“最大的错误”。毕竟，`throw`关键字将强制开发人员调用此方法来捕获异常。

> pokemon catch : 捕获泛型异常是否可取的问题时，出现的一个黑话。
>
> ```java
> try { // doSomething }
> catch (Exception) {// Do nothing}
> ```

### `try`语句块

```java
    try {
        readAFile();
    } catch (UnsupportedEncodingException | EOFException e) {//子类需先于超类
        throw new IllegalArgumentException("File Error");
    } catch (IOException e) { // 可无需特别地执行语句块，只需在方法声明，省略此分支
        e.printStackTrace();
    } finally {//不论是否出现异常，均会执行该语句块
        ClearSomething();//一般仅为打扫战场，不建议进行控制流语句操作
    }
```

对于使用了带`AutoCloseable`接口的资源，有`try-with-resources`语句：

```java
try( var in = new Scanner(new FileInputStream("words",...))){//resources ";"
    while(in.hasNext){...}//work woth resources
}//不论该块如何退出，都会AutoClose对应资源（关闭in）
```

其内部也可以有`Catch`等语句，`finally`内的语句会在关闭资源后执行。

**除内存外的资源，均不能依赖垃圾回收自动关闭，因此，对于文件、管道（Scanner…）、数据库连接、`Socket`等不会被自动垃圾回收的资源，推荐使用`try-with-resources`语句块。**

### 异常分析工具

`Throwable`类倾情提供以下方法，支援我们与bug的战斗。

- `printStackTrace` 堆栈轨迹文本描述信息

- `getSuppressed` 得到异常的“被阻塞”异常，一般是close方法的异常

`java.lang.StackWalker.Stackframe`帮助我们通过调用实例分析异常的来源。

- `StackWalker getInstance()`得到实例流

## 断言

语法格式：`assert condition : expression`

类加载器提供断言的开关，启用断言：`-ea(-enableassertions)`，关闭：`-da`

> 你可能需要注意一些异常信息的输出流，可能位于stderr，使用`2>`进行重定向

##  日志

对于程序的行为，JAVA自带的logging API分为七个级别：

- SEVERE  -  WARNING 
- INFO - CONFIG
- FINE - FINER - FINEST

### 第三方日志

准备工作：将第三方库`jar`文件与Java源码`Main.java`置于同一路径或指定的`classpath`。

有关的操作略。

# 集合（容器）

集合（colection），由若干元素构成的整体。对标的是C++的STL。

## List

List，有序列表，行为基本与数组相同，包含`ArrayList`和`LinkedList`两种。我们使用`Iterator`来高效地访问内部的元素，参考代码：

```java
import java.util.List;
public class Main {
    public static void main(String[] args) {
        List<Integer> list = List.of(12, 34, 56);
        Integer[] array = list.toArray(new Integer[0]);//3 | list.size()
        //Integer[] array = list.toArray(Integer[]::new);//方法引用
        for (Integer n : array) {
            System.out.println(n);
        }
    }
}
```

## Map

类似于`unordered_map`.对标有序的`map`，java中`Treemap`类实现了`SortedMap`接口，以保证遍历时以Key的顺序来进行排序（放入的Key必须实现`Comparable`接口）。

应用广泛。`HashMap`通过对key计算`hashCode()`，通过空间换时间的方式，直接定位到value所在的内部数组的索引，具有较高的查找效率非常高。

如果实例有限，作为key的对象是常量`enum`类型，可以使用Java集合库提供的一种`EnumMap`，在内部以一个*紧凑* 的数组存储value，并且根据`enum`类型的key直接定位到内部数组的索引，建少了`hashCode()`的计算步骤，效率更高，且没有额外的空间浪费。

## Set

`Set`实际上相当于只存储key的`Map`（无序，有序：`TreeSet`），去重利器。

```java
public static <T> Set<T> removeDuplicateBySet(List<T> data) {
    if (CollectionUtils.isEmpty(data)) {
        return new HashSet<>();
    }
    return new HashSet<>(data);//注意相关null的处理
}
```

## 杂项

### 队列

- |                    | Queue                    | Deque                             |
  | ------------------ | ------------------------ | --------------------------------- |
  | 添加元素到队尾     | `add(E e)` / offer(E e)  | `addLast(E e)` / offerLast(E e)   |
  | 取队首元素并删除   | `E remove()` / E poll()  | `E removeFirst()` / E pollFirst() |
  | 取队首元素但不删除 | `E element()` / E peek() | `E getFirst()` / E peekFirst()    |
  | 添加元素到队首     | 无                       | `addFirst(E e)` / offerFirst(E e) |
  | 取队尾元素并删除   | 无                       | `E removeLast() `/ E pollLast()   |
  | 取队尾元素但不删除 | 无                       | `E getLast()` / E peekLast()      |

- PriorityQueue 类似Queue，可通过`Comparator`自定义优先级排序算法（元素可不实现`Comparable`接口）

- Stack：作为遗留物，已不推荐使用。提供`pop` `push` `peek`（取栈顶元素但不弹出）方法。

### Collections共有方法

`Collections.sort(list, /*Comparator */)` 排升序。

 `Collection.removeIf(/* Condition */)`删除满足特定条件的元素

`shuffle(list)` 洗牌（打乱顺序）

`List<T> immutable = Collections.unmodifiableList(mutable)`封装成不可变对应类集合

`int binarySearch(List list, Object key)` 对*有序的*List进行二分查找，返回索引

### Bitset

位集，比`boolean`的`List`省空间。具有按位操作的方法。

# 函数式编程

## Lambda表达式

lambda表达式是一个可传递代码块，使Java方便地用于函数式编程。

语法： `(Args) -> {code}`（很像匿名函数对不对）

其可转换为接口。如：

```java
Arrays.sort(strarray, (a,b)->a.compareToIgnoreCase(b));//参数类型可推断，省略；代码块只有一句，省略大括号；省略return；
Arrays.sort(array, String::compareTo);//对于只含调用的操作，lambda表达式和方法引用相当
```

## 流

流是一个比集合更高概念上获取和转换数据的操作。其特点是：访问时再计算出值。

### 流的创建

`Collection`接口的`stream`方法可将集合转换为流，譬如：

`Stream<String> ss = List.of("G", "Od").stream()`

 ```java
 public class Main {
     public static void main(String[] args) {
         Stream<Integer> natual = Stream.generate(new NatualSupplier());
         natual.limit(14).forEach(System.out::println);
         // 流可创建无限序列，因为实质是算法，访问时计算；无限序列访问须先变成有限序列
     }
 }
 class NatualSupplier implements Supplier<Integer> {//通过supplier接口提供算法
     int n = 0;
     public Integer get() {//流通过调用Supplier.get()算法来不断构造下一个元素
         return n++;
     }
 }
 ```

### 流的转换

`map`操作，顾名思义，是一个一一映射。`Stream<Integer> spow = s.map(n -> n * n)`

`filter`操作，筛选出输入流中的元素，测试为`true`的元素们组成新的`Stream`。

其中的方法参数很像lambda表达式对不对，其实就是的，因此可以写成方法引用。

`distinct`操作，返回去重后的流。

### `Optional`类型

`Optional<T>`是一个包装器对象（容器），使用圣经有：

- 在值不存在的情况下产生相应的替代物（防止NullPointerException）

  `T orElse(T other)`当`Optional`为空时，产生`other`

- 仅在值存在时使用该值

  `ifPresent(Consumer<？ super T> action)`若值非空，传给`action`（Lambda | 方法引用 | …）

```java
if (o.isPresent()) {  
    var val = o.get();  
    //...
}
```

#### 管道化

可使用`map` `filter`等方法转换`Optional`内部的值，对应地有空值的处理方案。例如

```java
Optional.ofNullable(zoo).map(o -> o.getDog()).map(d ->d.getAge()) .ifPresent(age ->System.out.println(age));
//如此，若中间存在Null,当 ofNullable 等方法返回EMPTY而不会抛出NPE异常
```

#### 与流的关系

`Stream`类通过`findFirst`等方式产生`Optional`，作为有可能返回null值的很好的返回容器。

假设目前对于`String`类`ids`有返回`Optional`的查找方法`lookup`。其中，方法 `flatmap`会拼接满足条件的流，语句

`Stream<User> users = idstrings.map(Users::lookup).flatmap(Optional::stream)`

可丢弃不存在的用户。

### 收集

流通过`forEach`方法，可将某个函数应用于流中的各个元素。

`Collectors`收集器可收集流中的元素，产生对象，比如对于`Stream<Person>`：`stream.collect(Collectors.toMap(Person::getId,aFunction.identity()))`

`reduce`方法接收二元函数，持续地对流中的元素相继应用：

`Optional<Integer> sum = values.stream().reduce(Integer::sum)`

流库中提供基本类型流，以减少包装的开销，在此略。

# API（应用编程接口）

## IO

Java中的IO流机制较多，基础地基于`byte`的`InputStream`和`OutputStream`，派生出处理`Unicode`文本的`Reader`和`Writer`类、附着于文件的`FileInputStream` 和 `FileOutputStream`类等，并提供相应方法、`Closeable`接口（用于`try-with-resources`）。

所以Java对于不用的读取需求，派生出了多个读写流。基于此，`FilterInputStream` 和 `FilterOutputStream`赋予处理流的功能，再基于此，对于文本输出，可使用扩展于`Writer`的`PrintWriter`进行打印。

`DataInput`和`DataOutput`接口定义了一些二进制格式处理方法，如`DataInputStream`类就实现之；还有对于`zip`和随机访问文件的功能，略。

## 对象序列化

对象序列化(object serialization)机制，可将任何对象写出，并读回。这个机制是

- 每个对象都关联一个序列号（根据数据域类型和方法签名的SHA等）
- 对于已保存过的对象，标记“与序列号为 x 的对象相同”
- 读回对象时，第一次遇到的序号则对应进行构建初始化
- 遇到“与序列号为 x 的对象相同”标记，获取相关对象引用

## 锁机制

`FileChannel`类的`lock`和`unlock`方法

## Date-Time API

略

# JAVA I/O 模型

## NIO

Java NIO 中最核心的就是 *selector*, 每当连接事件、接收连接事件、读事件和写事件中的一种**事件就绪时，相关的事件处理器就会执行对应的逻辑，这种基于事件驱动的模式叫作 Reactor 模式**。
以下是 Reactor模式的五种重要角色：

-   Handle（Linux-fd）：它是**资源在操作系统层面上的一种抽象**，表示一种由操作系统提供的资源，比如 socket 或者文件描述符。该资源**与事件绑定在一起，也可用于表示一个个事件**。
-   Synchronous Event Demultiplexer（同步事件分离器）：Handle 代表的事件会被注册到同步事件分离器上，**常用的如 I/O 多路复用。当事件就绪时，同步事件分离器会分发和处理这些事件，本质是一个系统调用，用于等待事件的发生。** 调用方在调用它的时候会被阻塞，一直到同步事件分离器上有时间就绪为止。在 Java NIO 领域中，同步事件分离器对应的组件就是 Selector，对应的阻塞方法就是 select 方法。
-   Event Handler（事件处理器）：它由多个回调方法构成，这些回调方法就是**对某个事件的逻辑反馈，事件处理器一般都是抽象接口。** 比如当 Channel 被注册到 Selector 时的回调方法、写事件发生时的回调方法等。在 Java NIO 中，并没有提供事件处理器的抽象供我们使用。
-   Concrete Event Handler（具体事件处理器)：它是**事件处理器（抽象接口）的实现**。它本身实现了事件处理器所提供的各种回调方法，从而实现了特定的业务逻辑。比如针对连接事件需要打印一条日志，就可以在连接事件的回调方法里实现打印日志的逻辑。
-   Initiation Dispatcher（初始分发器）：整个事件处理器的核心，可以把它看作 Reactor， 它**规定了事件的调度策略，并且用于管理事件处理器，提供了事件处理器的注册、删除等方法**，事件处理器需要注册到 Initiation Dispatcher 上才能生效。Initiation Dispatcher 会通过 Synchronous Event Demultiplexer 来等待事件的发生。一旦事件发生，Initiation Dispatcher 首先会分离出每一个事件，然后找到相应的事件处理器，最后调用相关的回调方法处理这些事件。

## Reactor 的三种模型

1. 单 Reactor 单线程模型
   只有一个 Reactor，并且无论与 IO 相关的读/写，还 是与 IO 无关的编/解码或计算，都在一个 Handler 线程上完成。
2. 单 Reactor 多线程模型
   **除了 IO 操作以外的逻辑通过多线程来执行**，而IO 的读/写和 Reactor 的处理还是由一个线程执行。

> 将业务逻辑交给线程池来处理，可以充分利用多核 CPU 的处理能力。

3. 主从 Reactor 多线程模型
   当客户端连接数很多，IO 操作频繁时，单 Reactor 就会暴露问题，因为连接事件往往没有读/写事件频繁，读写事件会影响新客户端建立连接的请求，导致连接超时等情况。
   **处理连接事件的线程与处理读/写事件的线程分离**，设置多个 Reactor, Main Reactor 一般只有一个，它负责监听和处理连接请求，而 Sub Reactor 可以有多个， 用线程池进行管理，主要负责监听和处理读/写事件等。当然也可以将 Main Reactor 改为多个，通过线程池管理，数量其取决于客户端连接是否频繁，并不是越多越好。

## AIO
异步非阻塞的 IO 操作方式，有两种使用方式：一种是简单的将来式，另一种是回调式。
-   将来式：用 Future 类实现将来式，将执行任务交给线程池执行后，执行任务的线程并不会阻塞，它会返回一个 Future 对象，在执行结束后，Future 对象的状态会变成完成。在 Future 对象中可以获得对应的返回值，但是需要调用 get 方法来获取结果。如果结果还没返回，则调用 get 方法的线程就会被阻塞，这无疑跟同步调用相差无几。
-   回调式：Java 提供了 CompletionHandler 作为回调接口，在调用 read, write 等方法时，可以传入 CompletionHandler 的实现作为事件完成的回调接口，这种方式需要用户自行编写回调后的业务逻辑。

> 在 Java 8 中提供了 CompletableFuture，它既支持原来的 Future 的功能，也支持回调式。除此之外，CompletableFuture 还支持不同 CompletableFuture 间的相互协调或组合，方便了异步 I/O 的开发。

---
参考：
《Java核心技术》