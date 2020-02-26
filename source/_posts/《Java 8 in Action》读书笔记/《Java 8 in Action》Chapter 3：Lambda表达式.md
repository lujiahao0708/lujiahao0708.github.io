---
layout: post
title: 《Java 8 in Action》Chapter 3：Lambda表达式
date: 2019-03-07 22:34:52
categories: Java
tags:
- Java8
- 读书笔记
description: 《Java 8 in Action》Chapter 3：Lambda表达式
---

# 1. Lambda简介
可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。
- 匿名——我们说匿名，是因为它不像普通的方法那样有一个明确的名称:写得少而想得多!
- 函数——我们说它是函数，是因为Lambda函数不像方法那样属于某个特定的类。但和方法一样，Lambda有参数列表、函数主体、返回类型，还可能有可以抛出的异常列表。
- 传递——Lambda表达式可以作为参数传递给方法或存储在变量中。
- 简洁——无需像匿名类那样写很多模板代码。

# 2. Lambda写法
`(parameters) -> expression 或 (parameters) -> { statements; }`
`eg：(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());`
Lambda表达式有三个部分:
- 参数列表——这里它采用了Comparator中compare方法的参数，两个Apple。 
- 箭头——箭头->把参数列表与Lambda主体分隔开。
- Lambda主体——比较两个Apple的重量。表达式就是Lambda的返回值了。

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%203%EF%BC%9ALambda%E8%A1%A8%E8%BE%BE%E5%BC%8F/1.lambda%E7%A4%BA%E4%BE%8B.png)

# 3. 函数式接口和函数描述符
函数式接口就是只定义一个抽象方法的接口。接口上标有`@FunctionalInterface`表示该接口会设计成 一个函数式接口，如果你用`@FunctionalInterface`定义了一个接口，而它却不是函数式接口的话，编译器将返回一个提示原因的错误。接口现在还可以拥有默认方法(即在类没有对方法进行实现时， 其主体为方法提供默认实现的方法)。哪怕有很多默认方法，只要接口只定义了一个抽象方法，它就仍然是一个函数式接口。

函数式接口的抽象方法的签名就是Lambda表达式的签名。我们将这种抽象方法叫作：函数描述符。例如，Runnable接口可以看作一个什么也不接受什么也不返回(void)的函数的签名，因为它只有一个叫作run的抽象方法，这个方法什么也不接受，什么也不返回(void)。

# 4. 三种常用的函数式接口
## 4.1 Predicate
```java
/**
 * Represents a predicate (boolean-valued function) of one argument.
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #test(Object)}.
 * @param <T> the type of the input to the predicate
 * @since 1.8
 */
@FunctionalInterface
public interface Predicate<T> {
    /**
     * Evaluates this predicate on the given argument.
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
```
Predicate的英文示意是：谓词。
Predicate接口定义了一个名叫test的抽象方法，它接受泛型T对象，并返回一个boolean。

## 4.2 Consumer
```java
/**
* Represents an operation that accepts a single input argument and returns no
* result. Unlike most other functional interfaces, {@code Consumer} is expected
* to operate via side-effects.
* <p>This is a <a href="package-summary.html">functional interface</a>
* whose functional method is {@link #accept(Object)}.
* @param <T> the type of the input to the operation
* @since 1.8
*/
@FunctionalInterface
public interface Consumer<T> {
   /**
    * Performs this operation on the given argument.
    * @param t the input argument
    */
   void accept(T t);
}
```
Consumer的英文示意是：消费者。
Consumer接口定义了一个名叫accept的抽象方法，它接受泛型T对象，并没有返回任何值。

## 4.3 Function
```java
/**
* Represents a function that accepts one argument and produces a result.
* <p>This is a <a href="package-summary.html">functional interface</a>
* whose functional method is {@link #apply(Object)}.
* @param <T> the type of the input to the function
* @param <R> the type of the result of the function
* @since 1.8
*/
@FunctionalInterface
public interface Function<T, R> {
   /**
    * Applies this function to the given argument.
    * @param t the function argument
    * @return the function result
    */
   R apply(T t);
}
```
Function的英文示意是：功能。
Function接口定义了一个名叫apply的抽象方法，它接受泛型T对象，并返回一个泛型R的对象。
   
> Java还有一个自动装箱机制来帮助程序员执行这一任务:装箱和拆箱操作是自动完成的。但这在性能方面是要付出代价的。装箱后的值本质上就是把原始类型包裹起来，并保存在堆里。因此，装箱后的值需要更多的内存，并需要额外的内存搜索来获取被包裹的原始值。Java 8为我们前面所说的函数式接口带来了一个专门的版本，以便在输入和输出都是原始类型时避免自动装箱的操作。

## 4.4 Java 8中的常用函数式接口
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%203%EF%BC%9ALambda%E8%A1%A8%E8%BE%BE%E5%BC%8F/2.Java%208%E4%B8%AD%E7%9A%84%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3.png)

# 5. 类型检查、类型推断以及限制
## 5.1 类型检查
Lambda的类型是从使用Lambda的上下文推断出来的。上下文(比如，接受它传递的方法的参数，或接受它的值的局部变量)中Lambda表达式需要的类型称为目标类型。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%203%EF%BC%9ALambda%E8%A1%A8%E8%BE%BE%E5%BC%8F/3.%E7%B1%BB%E5%9E%8B%E6%A3%80%E6%9F%A5.png)
类型检查过程可以分解为如下所示。
- 首先，你要找出filter方法的声明。
- 第二，要求它是Predicate<Apple>(目标类型)对象的第二个正式参数。
- 第三，Predicate<Apple>是一个函数式接口，定义了一个叫作test的抽象方法。
- 第四，test方法描述了一个函数描述符，它可以接受一个Apple，并返回一个boolean.
- 最后，filter的任何实际参数都必须匹配这个要求。

这段代码是有效的，因为我们所传递的Lambda表达式也同样接受Apple为参数，并返回一个 boolean。请注意，如果Lambda表达式抛出一个异常，那么抽象方法所声明的throws语句也必 须与之匹配。有了目标类型的概念，同一个Lambda表达式就可以与不同的函数式接口联系起来，只要它 们的抽象方法签名能够兼容。比如，前面提到的Callable和PrivilegedAction，这两个接口都代表着什么也不接受且返回一个泛型T的函数。 因此，下面两个赋值是有效的:
```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

特殊的void兼容规则
如果一个Lambda的主体是一个语句表达式， 它就和一个返回void的函数描述符兼容(当然需要参数列表也兼容)。
例如，以下两行都是合法的，尽管List的add方法返回了一个 boolean，而不是Consumer上下文(T -> void)所要求的void:
```java
// Predicate返回了一个boolean 
Predicate<String> p = s -> list.add(s); 
// Consumer返回了一个void 
Consumer<String> b = s -> list.add(s);
```

## 5.2 类型推断
Java编译器会从上下文(目标类型)推断出用什么函数式接 口来配合Lambda表达式，这意味着它也可以推断出适合Lambda的签名，因为函数描述符可以通过目标类型来得到。这样做的好处在于，编译器可以了解Lambda表达式的参数类型，这样就可以在Lambda语法中省去标注参数类型。
```java
// 没有类 型推断
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); 
// 有类型推断
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); 
```

## 5.3 使用局部变量
Lambda表达式 也允许使用自由变量(不是参数，而是在外层作用域中定义的变量)，就像匿名类一样。 它们被 称作捕获Lambda。
Lambda捕获的局部变量必须显式声明为final， 或事实上是final。换句话说，Lambda表达式只能捕获指派给它们的局部变量一次。
- 第一，实例变量和局部变量背后的实现有一 个关键不同。实例变量都存储在堆中，而局部变量则保存在栈上。如果Lambda可以直接访问局部变量，而且Lambda是在一个线程中使用的，则使用Lambda的线程，可能会在分配该变量的线程将这个变量收回之后，去访问该变量。因此，Java在访问自由局部变量时，实际上是在访问它的副本，而不是访问原始变量。如果局部变量仅仅赋值一次那就没有什么区别了——因此就有了这个限制。
- 第二，这一限制不鼓励你使用改变外部变量的典型命令式编程模式(我们会在以后的各章中 解释，这种模式会阻碍很容易做到的并行处理)。

# 6. 方法引用
方法引用可以被看作仅仅调用特定方法的Lambda的一种快捷 写法，方法引用看作针对仅仅涉及单一方法的Lambda的语法糖。目标引用放在分隔符::前，方法的名称放在后面。方法引用主要有三类：
- (1) 指向静态方法的方法引用(例如Integer的parseInt方法，写作Integer::parseInt)。
- (2) 指向任意类型实例方法的方法引用(例如String的length方法，写作 String::length)。
- (3) 指向现有对象的实例方法的方法引用(假设你有一个局部变量expensiveTransaction 用于存放Transaction类型的对象，它支持实例方法getValue，那么你就可以写expensive- Transaction::getValue)。

```java
对于一个现有构造函数，你可以利用它的名称和关键字new来创建它的一个引用: ClassName::new。它的功能与指向静态方法的引用类似。
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
     
这就等价于:
Supplier<Apple> c1 = () -> new Apple(); // 利用默认构造函数创建 Apple的Lambda表达式
Apple a1 = c1.get(); // 调用Supplier的get方法 将产生一个新的Apple
```

# 7. 复合Lambda表达式的有用方法
## 7.1 比较器复合
```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
// 逆序 按重量递 减排序
inventory.sort(comparing(Apple::getWeight).reversed());
// 比较器链 按重量递减排序;两个苹果一样重时，进一步按国家排序
inventory.sort(comparing(Apple::getWeight)
         .reversed()
         .thenComparing(Apple::getCountry));
```

## 7.2 谓词复合
```java
// 产生现有Predicate 对象redApple的非
Predicate<Apple> notRedApple = redApple.negate();
// 链接两个谓词来生成另 一个Predicate对象  一个苹果既是红色又比较重
Predicate<Apple> redAndHeavyApple = redApple.and(a -> a.getWeight() > 150);
// 链接Predicate的方法来构造更复杂Predicate对象 表达要么是重(150克以上)的红苹果，要么是绿苹果
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(a -> a.getWeight() > 150) 
                                            .or(a -> "green".equals(a.getColor()));
请注意，and和or方法是按照在表达式链中的位置，从左向右确定优 先级的。因此，a.or(b).and(c)可以看作(a || b) && c。
```

## 7.3 函数复合
```java
andThen方法会返回一个函数，它先对输入应用一个给定函数，再对输出应用另一个函数。 比如，
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g);
int result = h.apply(1);
数学上会写作g(f(x))或(g o f)(x)
这将返回4

compose方法，先把给定的函数用作compose的参数里面给的那个函 数，然后再把函数本身用于结果。
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.compose(g);
int result = h.apply(1);
数学上会写作f(g(x))或(f o g)(x) 
这将返回3
```

# 8. 小结
以下是你应从本章中学到的关键概念。
* Lambda表达式可以理解为一种匿名函数:它没有名称，但有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常的列表。
* Lambda表达式让你可以简洁地传递代码。
* 函数式接口就是仅仅声明了一个抽象方法的接口。
* 只有在接受函数式接口的地方才可以使用Lambda表达式。
* Lambda表达式允许你直接内联，为函数式接口的抽象方法提供实现，并且将整个表达式作为函数式接口的一个实例。
* Java 8自带一些常用的函数式接口，放在java.util.function包里，包括Predicate<T>、Function<T,R>、Supplier<T>、Consumer<T>和BinaryOperator<T>，如表3-2所述。
* 为了避免装箱操作，对Predicate<T>和Function<T, R>等通用函数式接口的原始类型特化:IntPredicate、IntToLongFunction等。
* 环绕执行模式(即在方法所必需的代码中间，你需要执行点儿什么操作，比如资源分配 和清理)可以配合Lambda提高灵活性和可重用性。
* Lambda表达式所需要代表的类型称为目标类型。
* 方法引用让你重复使用现有的方法实现并直接传递它们。
* Comparator、Predicate和Function等函数式接口都有几个可以用来结合Lambda表达式的默认方法。

## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！
