---
title: Java函数式编程
date: 2019-10-08 20:01:56
tags:
  - lambda
  - functional
categories:
  - functional
---

## 函数式编程
> 与面向对象、过程式编程并列的编程范式

- 函数是一等公民
- 强调将计算过程分解成可复用的函数，典型例子：`map`和`reduce`组合的`MapReduce`算法
- 函数是一种数学运算，要求必须是纯粹的，

函数式编程的起源是一门叫做范畴论（**Category Theory**）的数学分支

<!-- more -->

什么是范畴呢？
>In mathematics, a category is an algebraic structure that comprises "objects" that are linked by "arrows".

{% asset_img category.png %}

箭头表示范畴成员之间的关系，正式名称为“态射”（*morphism*）范畴论认为，同一个范畴的所有成员，就是不同状态的“变形”（*transformation*）。通过“态射”，一个成员可以变形成另一个成员。

范畴的数学模型
- 所有成员是一个集合
- 变形关系是函数
也就是说，范畴论是集合论上的抽象，简单的理解就是“集合+函数”
理论上通过函数，范畴中的成员可以算出其他所有成员。

范畴与容器
- 值（`value`）
- 值的形变关系（`function`）

1. 范畴论与函数式编程的关系
2. 范畴论使用函数表达范畴之间的关系。
3. 伴随范畴论的发展，就发展出一整套函数的运算方法。这套方法起初只用于数学运算，后来有人将它在计算机上实现了，就成了“函数式编程”
4. 本质上函数式编程只是范畴论的运算方法，跟数理逻辑、微积分、行列式是同一类东西，都是数学方法，只是能用来写程序

函数的合成（`compose`）
> 函数合同满足结合律

```
// 参数个数相同
compose(f, compose(g, h))
// 等同于
componse(compose(f, g), h)
// 等同于
componse(f, g, h)
```

函数的柯里化
> 把一个多参数的函数，转化为单参数函数

```
// 柯里化之前
function add(x, y) {
    return x + y;
}

add(1, 2)

// 柯里化之后
function addX(y) {
    return function(x){
        return x + y;
    }
}
addX(2)(1)
```

函数常规方法
```
@FunctionalInterface
public interface Func{
    Object exc(Object obj)
}

public class IO{
    private Object val;

    private IO(Object val){
        this.val = val;
    }

    public static IO of(Object val){
        return new IO(val);
    }

    public Object join(){
        return val;
    }

    public IO apply(IO io){
        return of(((Func)val).exc(io.val));
    }

    public IO map(Func func){
        return of(func.exc(val));
    }

    public IO flatMap(Func func){
        return (IO) map(func).join();
    }

    public static final void main(String[] args){
        Func add = x -> of("hello".equals(x) ? "world" : x);
        Func print = x -> {System.out.println(x); return of(x)}

        of(x -> "hello").apply(of("world"));

        of("hello").flatMap(print);
        of("hello").flatMap(add).flatMap(print);
    }
}
```

## Java Lambda
简述`Java Se 8`中新引入的Lambda语言特性以及这些特性背后的设计思想
- `Lambda`表达式
- 方法引用 & 构造器引用
- 拓展的目标类型 & 类型推导
- 接口中的默认、静态方法

### Backgroud
面向对象编程使用带有方法的对象封装行为，函数式编程使用函数封装行为。
*Java*的对象比较重量级，实例化一个类型会涉及不同的类型，并需要初始化字段和方法。
*Java*有时会封装单函数对象，例如`Runnable`、`ActionListener`等
```
public interface Runnable{
    void run();
}
```
此处并不需要专门定义一个类来实现`Runnable`，一般会使用匿名类型内联
```
new Thread(new Runnbal(){
    public void run(){
        //TODO
    }
});
```
并行领域是个值得研究的领域，因为**摩尔定律**在此得到了重生，尽管没有更快的CPU，但是有更多的CPU，串行API就只能使用有限的计算能力

随着函数式编程风格的流行，Java需要提供一种尽可能轻量级的封装方式（**model code as data**），匿名内部类并不是一个好的选择
1. 语法过于冗余
2. 内部类的`this`和外部容易使人误解
3. 类型载入和实例创建语义不够灵活
4. 无法捕获非`final`局部变量
5. 无法对控制流程进行抽象

以上多数问题均在`Java Se 8`中得到了解决
- 通过提供更简洁的语法和局部作用域规则，解决了问题1 & 2
- 通过提供更加灵活便于优惠的表达式语义，绕开了问题3
- 通过允许编译器推断变量的“常量性”，减轻了问题4

### 函数式接口
> `@FunctionalInterface`

每一个函数对象对应一个接口类型，便于和Java类型系统紧密结合。
- 接口是Java类型系统的一部分
- 接口天然就拥有运行时表示（*Runtime representation*）
- 接口可以通过Javadoc注释来表达一些非正式的协定

编译器会通过`@FunctionalInterface`注解来判断且检查是否满足函数式接口的要求
Java并没有引入了一个全新的结构化函数类型，而是选择了“使用已知类型”，因为现有的类库大量使用了函数式接口，我们可以用`lambda`表达式直接实现现有类库。
`Java Se 7`中已存在的函数式接口：
- `java.lang.Runnable`
- `java.util.comcurrent.Callable`
- `java.security.PrivilegedAction`
- `java.util.Comparator`
- `java.io.FileFilter`
- `java.beans.PropertyChangeListener`

`Java Se 8` 新增加了一个包，`java.util.function`包含了常用的函数式接口：
- `Predicate<T>`，接收`T`返回`boolean`
- `Consumer<T>`，接收`T`，无返回
- `Function<T,R>`，接收`T`，返回`R`
- `Supplier<T>, IntSupplier<T>`，提供`T`（工厂），无参
- `UnaryOperator<T>`，接收`T`，返回`T`
- `BinaryOperator<T>, LongBinaryOperator<T>`，接受两个`T`，返回`T`
- `BiFunction<T,U,R>`，接收`T, U`，返回`R`

### Lambda Expressions
> 匿名类型最大的问题就其冗余的语法，lambda表达式提供了轻量的语法，解决了“高度问题”。

```
(parameters) -> expression / statement || (parameters) -> {statements}
```

### Target typing
> 函数式接口的名称并不是Lambda表达式的一部分，Lambda表达式的类型由上下文推导得出。

相同的Lambda表达式在不同的上下文中拥有不同的类型
```
Callable<String> c = () -> "hello world";
PrivilegedAction<String> p = () -> "hello world";
```
编译器利用上下文*所期待的类型*推导Lambda表达式的类型，*所期待的类型*就是目标类型，Lambda表达式只能出现在目标类型为函数式接口的上下文中。

Lambda表达式对目标类型也是有要求的，编译器会检查Lambda表达式的类型和目标类型的方法签名是否一致，当且仅当以下所有条件均满足时，Lambda表达式才可以被赋值给目标类型`F`
- `F`是一个函数式接口
- Lambda表达式的参数和`T`的方法参数在数量和类型上对应
- Lambda表达式的返回值和`T`的方法返回值兼容
- Lambda表达式所抛出的异常和`T`的方法`throws`类型兼容

Lambda表达式的参数类型可以从目标类型中推导出, 当只有一个参数时括号可省略
```
Comparator<String> c = (x, y) -> x.compareTo(y);
FileFilter f = x -> x.getName().endsWith("xml");
```
泛型方法调用和`<>`构造器调用也通过目标类型进行类型推导

```
List<String> list = Collections.emptyList();
```

### 目标类型的上下文
带有目标类型的上下文如下：
- 变量声明
- 赋值
- 返回语句
- 数组初始化
- 方法和构造器参数
- Lambda表达式的函数体
- 三元运算符
- 转型表达式

```
Comparator<String> c = (x, y) -> x.compareTo(y);
Runnable r = () -> {};
```
方法参数的类型推导会涉及：重载解析 & 参数类型推导。
当Lambda表达式作为方法参数时，重载解析会影响到Lambda表达式的目标类型。当Lambda表达式显式指定参数类型，编译器可以直接使用返回类型，当Lambda表达式的参数类型推导得出，重载解析会忽略函数体而只依赖表达式参数数量。

如果解析方法声明时存在二义性，则需要使用转型或显式Lambda表达式来提供更多的类型信息。
```
Stream<String> s = list.stream().map(x -> x.getName());
```
- 显式Lambda表达式参数类型
- Lambda表达式转型为`Function<T,R>`
- 为泛型参数提供一个实际类型，`.<String>map(x -> x.getName()`

Lambda表达式可以通过外部的目标类型推导出内部的返回类型，可以方便的编写返回函数的函数
```
Supplier<Callable<String>> s = () -> () -> "hello world";
Object o = (Callable) () -> "hello world";
```
目标类型不仅适用于Lambda表达式，泛型也收益。
```
List<Integer> list = Collections.checkedList(new ArrayList<>(), Integer.class)
```

### 词法作用域
内部类中使用变量容易出错，继承的成员可能会把外部类的成员掩盖，未限定的`this`引用会指向内部类
相对于内部类，`Lambda`表达式的语义就十分简单，它不会继承`super`，也不会引入新的作用域。Lambda表达式基于词法作用域，其内外具有相同语义（如`for`循环一致）

### 变量捕获
在`Java Se 7`中，编译器对内部类中引用的外部变量（即捕获的变量）要求必须声明为`final`，在`Java Se 8`中放宽了限制，对于`Lambda`表达式和内部类，允许在捕获有效只读的变量（本质上仍然是`final`，只是不用显式声明）

对`this`的引用，以及通过`this`对未限定字段、方法的引用在本质上都是调用`final`局部变量，当引用`this`时相当于捕获了`this`实例，其他情况下，不会保留任何对`this`的引用。
内部类实例会一直保留一个对外部类实例的强引用，而没有捕获`this`的Lambda表达式则不会保留其引用，避免造成内存泄漏。

Lambda表达式不支持修改捕获变量的另一个原因是可以使用**规约**来实现同样的效果。`java.util.stream`包提供了多种规约操作（如`sum`、`min`、`max`等）
```
int sum = list.stream().mapToInt(x -> x.size()).sum();

// sun() 等价于
int sum = list.stream().mapToInt(x -> x.size()).reduce(0, (x, y) -> x + y);
```

### 方法引用
> 隐式`Lambda`表达式

`Lambda`表达式允许定义一个匿名方法以及以函数式接口的方式使用，希望能够在已有的方法上实现同样的特性。
方法引用与`Lambda`表达式具有相同的特性（目标类型 & 函数式接口），可直接通过方法名称引用已有的方法
```
Comparator<String> c = Comparator.comparing(String::toString);
```
`String::toString`可被看成*Lambda表达式*的简写,方法引用不会将语法变得更紧凑，但拥有更明确的语义（可通过方法名直接调用）
```
Consumer<Integer> c1 = System::exit // void exit(int status)
Consumer<String[]> c2 = Arrays::sort; // void sort(Object[] o)
Callable<List<String>> c = Collections::emptyList;
```

方法引用种类
- 静态方法引用`ClassName::method`
- 实例上的实例方法`ref::method`
- 超类上的实例方法`super::method`
- 类型上的实例方法`ClassName::method`
- 构造方法`ClassName:method`
- 数组构造方法`TypeName::new`

```
Predicate<String> p = list::contains;

Callable<Path> c = () -> path;
Privileged<Path> p = c::call;

Function<String, String> f = String::toUpperCase;
```
一般不需要指定方法引用的参数类型，编译器可以推导出结果，但如果需要可以在`::`之前显式提供参数类型。
```
IntFunction<int[]> f = int[]::new;
int[] a = f.apply(3);	// int[3]
```
### 默认方法 & 静态方法
> `Lambda`表达式和方法引用提升了Java语言的表达能力，为了把代码即数据(`code as data`)变的更加容易，需要将这些特性融入到已有的库中。

默认方法（被称为虚拟拓展方法或守护方法）的目标式解决在已有的类库增加功能，使得接口在发布之后仍能被逐步演化。
默认方法利用面向对象的方式向接口增加新的行为，接口方法可以是抽象的或默认的。默认方法拥有其默认实现，实现接口的类型通过继承得到该默认实现（如果类型没有覆盖该默认实现）。默认方法不是抽象方法，所以可以向函数式接口中增加默认方法。

如下展示如何向`Iterator`接口中增加默认方法`skip`
```
interface Iterator<E>{
    boolean hasNaxt();
    E next();
    void remove();

    default void skip(int i){
        for(; i > 0 && hasNext(); i--) {
            next();
         }
    }
}
```
以上`Iterator`定义，所有实现`Iterator`的类型都会继承`skip`方法，子类可以通过覆盖`skip`来提供更好的实现（移动游标或提供原子性操作等）
当接口继承其他接口时，既可以为所继承的抽象方法提供一个默认实现，也可为继承而来的默认方法提供一个新实现或把继承来的默认方法抽象化。

除了默认方法，`Java Se 8`还允许在接口中定义静态方法，这使得我们可以从接口直接调用和它相关的辅助方法，而不是从其它的类中调用（之前此种类对应的接口的复数命名，如`Collections`）
```
public static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<T, U> f){
    return (x, y) -> f.apply(x).compareTo(f.apply(y));
}
```

继承默认方法
和其他方法一样，默认方法也可以被继承，大多数情况下继承行为和期待的一致。不过，当类型或接口的超类拥有多个具有相同签名的方法时，我们就需要一套规则来解决此冲突。
- 类的方法声明优先于接口默认方法
- 被其他类型所覆盖的方法会被忽略（适用于`super`共享一个`method`的情况）

为了演示第二条规则，假设`Collection`和`List`接口均提供了`removeAll`的默认实现，然后`Queue`继承并覆盖了`Collection`中的默认方法，在下面的`implements`从句中，`List`中方法声明会优先于`Queue`中的方法声明。
```
class LinkedList<E> implements List<E>, Queue<E> {...}
```
当独立的默认方法相冲突或默认方法和抽象方法相冲突时会产生编译错误。此时需要显式覆盖父类方法。一般会定义一个默认方法来显式选择父类方法。
```
interface X implements Y, Z {
    default void draw {
        Y.super.draw();
    }
}
```

在设计Lambda时的一个重要目标就时新增的语言特性和库特性能够无缝结合。
```
Collections.sort(list, new Comparator<User>(){
    public int compare(User x, User y){
        return x.getName().compareTo(y.getName());
    }
})

// 有了Lambda表达式，可以去掉冗余的匿名内部类
Collections.sort(list, (x, y) -> x.getName().compareTo(y.getName()));

// 借助Comparator里的comparing方法来实现
Collections.sort(list, Comparator.comparing((User x) -> x.getName()));

// 在类型推导 & 静态导入的帮助下，可以进一步简化
Collections.sort(list, comparing(x -> x.getName()));

// 上面的Lambda表达式实际上是`getName`的代理，可以使用方法引用替代
Collections.sort(list, comparing(User::getName));

// `Collections.sort`并不是一个好的方式，可以在`List`中添加`sort`默认方法来提供更好的实现
list.sort(comparing(User::getName));

// 此外，如果在`Comparator`中增加一个默认方法`reversed`，便可以容易的实现降序操作
list.sort(comparing(User::getName).reversed());
```
### Stream
> A sequence of elments supporting sequential and parallel aggregate operations.

映射方法

- `filter`
- `map`
- `limit`
- `distinct`（依赖`equals`方法）

归纳方法
- `count`
- `collect`

```
list.stream().filter(x -> x.getAge() > 3).map(x -> x.getName()).collect(Collectors.toList());
```

串行流 & 并行流
```
// 串行流
List<User> users = list.stream().collect(Collectors.toList());

// 并行流
List<User> users = list.stream().parallel().collect(Collectors.toList());
```
使用并行只需要`parallel()`即可
并行内部将数据分成多段，每段一个线程并行处理，然后join一起输出

`distinct`，对`stream`中包含的元素去重（依赖`equals`）

{% asset_img distinct.jpg %}

`filter`，对`stream`中包含的元素使用给定的过滤函数进行过滤
{% asset_img filter.jpg %}

`map`，对stream中包含的元素使用给定的转换函数进行转换操作，jdk有三个对于原始类型的变种方法，分别是：`mapToInt`，`mapToLong`，`mapToDouble`，可以避免自动装、拆箱的消耗。
{% asset_img map.bmp %}

`flatMap`，与`map`类似，不同的是每个元素转换得到的是`stream`对象，会把子`stream`中的元素聚合到外部父集合中
{% asset_img flatMap.jpg %}

`peek`，生成一个包含原`stream`的所有元素的新`stream`，同时会提供一个消费函数`Consumer`实例，新`stream`元素被消费的时都会调用给定的`Consumer`函数
{% asset_img peek.jpg %}

`limit`，对于一个`stream`进行截断操作，获取其前N个元素，如果元素个数不足N，返回所有元素
{% asset_img limit.jpg %}

`skip`，返回一个丢弃N个元素后的新`stream`，如果元素个数不足N，返回空`stream`
{% asset_img skip.jpg %}

stream的操作都是`lazy`的，内部会依赖转换函数生成新的函数（不执行），在`reduce`时遍历`stream`，每个元素都执行新的函数。

#### Reduce
> 可以简单的理解为归纳结果方法

reduce可以分为两种
- 可变归纳，将元素归纳到可变容器中，如`Collection`或`StringBuilder`
- 其他归纳，除去可变都时其他，一般都是通过前一次归纳的结果当作下一次的入参，如`reduce`、`count`、`allMatch`

##### 可变归纳
只有一个方法`collect`
```
// supplier是一个工厂函数，生成容器
// accumulator将元素添加到容器中
// combiner将多个结果合并到一个容器中（主要用于并发中）
<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);

List<String> s = Lists.newArrayList("hel","xx",null,"world");
List<String> withoutNull = s.stream().filter(x -> null != x).collect(ArrayList::new, List::add, List::addAll);
```
以下提供`collect`另外一个重写版本（依赖`Collector`）
```
<R, A> R collect(Collector<? super T, A, R) collector);
```

`Java Se 8`提供了工具类`Collectors`
```
List<String> l = s.stream().filter(x -> null != x).collect(Collectors.toList());
```
##### 其他归纳
reduce方法非常通用，简单介绍其最常用的两种重写方式
```
Option<T> reduce(BinaryOperator<T> accumulator);

int sum = list.stream().reduce((x, y) -> x + y).get();
```

`reduce`有一个常用的变种
```
T reduce(T identity, BinaryOperator<T> accumulator);

int sum = list.stream().reduce(0, (x, y) -> x + y);

int sum = list.stream().count();
```

查询相关
- `allMatch`，是否所有元素都满足
- `anyMatch`，任意元素是否满足
- `findFirst`，返回首个元素
- `noneMatch`，是否所有元素均不满足
- `max` & `min`，使用给定的`Operator`计算值
