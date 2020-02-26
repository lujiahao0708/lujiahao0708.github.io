---
layout: post
title: 《Java 8 in Action》Chapter 5：使用流
date: 2019-03-10 11:34:52
categories: Java
tags:
- Java8
- 读书笔记
description: 《Java 8 in Action》Chapter 5：使用流
---

> 流让你从外部迭代转向内部迭代，for循环显示迭代不用再写了，流内部管理对集合数据的迭代。这种处理数据的方式很有用，因为你让Stream API管理如何处理数据。这样Stream API就可以在背后进行多种优化。此外，使用内部迭代的话，Stream API可以决定并行运行你的代码。这要是用外部迭代的话就办不到了，因为你只能用单一线程挨个迭代。

# 1. 筛选和切片
## 1.1 用谓词筛选
该操作会接受一个谓词(一个返回 boolean的函数)作为参数，并返回一个包括所有符合谓词的元素的流。筛选出所有素菜
```java
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarian)
                                .collect(toList());
```

## 1.2 筛选各异的元素
返回一个元素各异(根据流所生成元素的 hashCode和equals方法实现)的流。筛选出列表中所有的偶数，并确保没有重复。
```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct()
        .forEach(System.out::println);
```

## 1.3 截短流
流支持limit(n)方法，该方法会返回一个不超过给定长度的流。所需的长度作为参数传递给limit。如果流是有序的，则最多会返回前n个元素。选出热量超过300卡路里的头三道菜
```java
List<Dish> dishes = menu.stream()
                        .filter(d -> d.getCalories() > 300)
                        .limit(3)
                        .collect(toList());
```
limit也可以用在无序流上，比如源是一个Set。这种情况下，limit的结果不会以 任何顺序排列。

## 1.4 跳过元素
流还支持skip(n)方法，返回一个扔掉了前n个元素的流。如果流中元素不足n个，则返回一个空流。跳过超过300卡路里的头两道菜，并返回剩下的。
```java
List<Dish> dishes = menu.stream()
                        .filter(d -> d.getCalories() > 300)
                        .skip(2)
                        .collect(toList());
```

# 2. 映射
## 2.1 对流中每一个元素应用函数
流支持map方法，它会接受一个函数作为参数。这个函数会被应用到每个元素上，并将其映射成一个新的元素。提取流中菜肴的名称：
```java
List<String> dishNames = menu.stream()
                             .map(Dish::getName)
                             .collect(toList());
```

## 2.2 流的扁平化
flatmap方法让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流。单个流都被合并起来，即扁平化为一个流。例如，给定单词列表 ["Hello","World"]，你想要返回列表["H","e","l", "o","W","r","d”]。
```java
List<String> words = Arrays.asList("Java 8", "Lambdas", "In", "Action”);
List<String> uniqueCharacters = words.stream()
         .map(w -> w.split(""))
         .flatMap(Arrays::stream)
         .distinct()
         .collect(Collectors.toList());
```
# 3. 查找和匹配
## 3.1 检查谓词是否至少匹配一个元素
anyMatch方法可以回答“流中是否有一个元素能匹配给定的谓词”。anyMatch方法返回一个boolean，因此是一个终端操作。比如，你可以用它来看看菜单里面是否有素食可选择:
```java
if(menu.stream().anyMatch(Dish::isVegetarian)){
System.out.println("The menu is (somewhat) vegetarian friendly!!");
}
```

## 3.2 检查谓词是否匹配所有元素
allMatch方法的工作原理和anyMatch类似，但它会看看流中的元素是否都能匹配给定的谓词。比如，你可以用它来看看菜品是否有利健康(即所有菜的热量都低于1000卡路里):
```java
boolean isHealthy = menu.stream().allMatch(d -> d.getCalories() < 1000);
```
和allMatch相对的是noneMatch。它可以确保流中没有任何元素与给定的谓词匹配。比如， 你可以用noneMatch重写前面的例子:
```java
boolean isHealthy = menu.stream().noneMatch(d -> d.getCalories() >= 1000);
```
anyMatch、allMatch和noneMatch这三个操作都用到了我们所谓的短路，这就是大家熟悉 的Java中&&和||运算符短路在流中的版本。

## 3.3 查找元素
findAny方法将返回当前流中的任意元素。它可以与其他流操作结合使用。比如，你可能想找到一道素食菜肴。你可以结合使用filter和findAny方法来实现这个查询:
```java
Optional<Dish> dish =menu.stream()
            .filter(Dish::isVegetarian)
            .findAny();
```
Optional<T>类(java.util.Optional)是一个容器类，代表一个值存在或不存在。Optional里面几种可以迫使你显式地检查值是否存在或处理值不存在的情形的方法也不错。
- isPresent()将在Optional包含值的时候返回true, 否则返回false。
- ifPresent(Consumer<T> block)会在值存在的时候执行给定的代码块。
- T get()会在值存在时返回值，否则抛出一个NoSuchElement异常。
- T orElse(T other)会在值存在时返回值，否则返回一个默认值。

## 3.4 查找第一个元素 
为此有一个findFirst 方法，它的工作方式类似于findany。 例如，给定一个数字列表，下面的代码能找出第一个平方 能被3整除的数: 
```java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
                                                           .map(x -> x * x)
                                                           .filter(x -> x % 3 == 0)
                                                           .findFirst(); // 9
```

# 4. 归约 
归约操作 (将流归约成一个值)。用函数式编程语言的术语来说，这称为折叠(fold)，因为你可以将这个操作看成把一张长长的纸(你的流)反复折叠成一个小方块，而这就是折叠操作的结果。
## 4.1 元素求和 
reduce操作是如何作用于一个流的:Lambda反复结合每个元素，直到流被归约成一个值。 
reduce接受两个参数:
- 一个初始值，这里是0;
- 一个BinaryOperator<T>来将两个元素结合起来产生一个新值，这里我们用的是 lambda (a, b) -> a + b。 

```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

reduce还有一个重载的变体，它不接受初始值，但是会返回一个Optional对象:
```java
Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));
```

## 4.2 最大值和最小值
使用reduce来计算流中的最大值
```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
```
要计算最小值，你需要把Integer.min传给reduce来替换Integer.max:
```java
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

## 4.3 小结
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/05.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%205%EF%BC%9A%E4%BD%BF%E7%94%A8%E6%B5%81/%E4%B8%AD%E9%97%B4%E6%93%8D%E4%BD%9C%E5%92%8C%E7%BB%88%E7%AB%AF%E6%93%8D%E4%BD%9C.png)

# 5. 实战
```java
public class ExecDemo {
    public static void main(String[] args) {
        Trader raoul = new Trader("Raoul", "Cambridge");
        Trader mario = new Trader("Mario","Milan");
        Trader alan = new Trader("Alan","Cambridge");
        Trader brian = new Trader("Brian","Cambridge");
        List<Transaction> transactions = Arrays.asList(
                new Transaction(brian, 2011, 300),
                new Transaction(raoul, 2012, 1000),
                new Transaction(raoul, 2011, 400),
                new Transaction(mario, 2012, 710),
                new Transaction(mario, 2012, 700),
                new Transaction(alan, 2012, 950)
        );

        System.out.println("(1) 找出2011年发生的所有交易，并按交易额排序(从低到高)。");
        List<Transaction> collect = transactions.stream()
                .filter(t -> t.getYear() == 2011)
                .sorted(Comparator.comparing(Transaction::getValue))
                .collect(Collectors.toList());
        System.out.println(collect);

        System.out.println("\n(2) 交易员都在哪些不同的城市工作过?");
        List<String> collect1 = transactions.stream()
                .map(transaction -> transaction.getTrader().getCity())
                .distinct()
                .collect(Collectors.toList());
        System.out.println(collect1);
        // [Cambridge, Milan]

        System.out.println("\n(3) 查找所有来自于剑桥的交易员，并按姓名排序。");
        List<Trader> collect2 = transactions.stream()
                .map(Transaction::getTrader)
                .filter(trader -> trader.getCity().equals("Cambridge"))
                .distinct()
                .sorted(Comparator.comparing(Trader::getName))
                .collect(Collectors.toList());
        System.out.println(collect2);

        System.out.println("\n(4) 返回所有交易员的姓名字符串，按字母顺序排序。");
        String reduce = transactions.stream()
                .map(transaction -> transaction.getTrader().getName())
                .distinct()
                .sorted()
                .reduce("", (n1, n2) -> n1 + n2);
        System.out.println(reduce);

        System.out.println("\n(5) 有没有交易员是在米兰工作的?");
        boolean b = transactions.stream()
                .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"));
        System.out.println(b);

        System.out.println("\n(6) 打印生活在剑桥的交易员的所有交易额。");
        transactions.stream()
                .filter(transaction -> transaction.getTrader().getCity().equals("Cambridge"))
                .map(Transaction::getValue)
                .forEach(System.out::println);

        System.out.println("\n(7) 所有交易中，最高的交易额是多少?");
        transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::max)
                .ifPresent(System.out::println);

        System.out.println("\n(8) 找到交易额最小的交易。");
        transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::min)
                .ifPresent(System.out::println);
    }
```

# 6. 数值流
## 6.1 原始类型流特化
Java 8引入了三个原始类型特化流接口来解决这个问题:IntStream、DoubleStream和 LongStream，分别将流中的元素特化为int、long和double，从而避免了暗含的装箱成本。
### 6.1.1 映射到数值流
将流转换为特化版本的常用方法是mapToInt、mapToDouble和mapToLong。这些方法和前 面说的map方法的工作方式一样，只是它们返回的是一个特化流，而不是Stream<T>。例如，你 可以像下面这样用mapToInt对menu中的卡路里求和:
```java
int calories = menu.stream()                        // 返回一个 Stream<Dish>
                    .mapToInt(Dish::getCalories)    // 返回一个 IntStream
                    .sum();
```
请注意，如果流是空的，sum默认返回0。IntStream还支持其他的方便方法，如max、min、average等。

### 6.1.2 转换回对象流
要把原始流转换成一般流(每个int都会装箱成一个 Integer)，可以使用boxed方法，如下所示:
```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); //将Stream转换为数值流
Stream<Integer> stream = intStream.boxed(); // 将数值流转换为Stream
```

### 6.1.3 默认值OptionalInt
对于三种原始流特化，也分别有一个Optional原始类 型特化版本:OptionalInt、OptionalDouble和OptionalLong。例如，要找到IntStream中的最大元素，可以调用max方法，它会返回一个OptionalInt:
```java
OptionalInt maxCalories = menu.stream().mapToInt(Dish::getCalories).max();
```

## 6.2 数值范围
Java 8引入了两个可以用于IntStream和LongStream的静态方法，帮助生成这种范围: range和rangeClosed。这两个方法都是第一个参数接受起始值，第二个参数接受结束值。但range是不包含结束值的，而rangeClosed则包含结束值。
```java
IntStream evenNumbers = IntStream.range(1, 100) .filter(n -> n % 2 == 0); // 一个从1到100的偶数流   表示范围[1, 100)
IntStream evenNumbers = IntStream.rangeClosed(1, 100) .filter(n -> n % 2 == 0); // 一个从1到100的偶数流   表示范围[1, 100]
```

# 7. 构建流
## 7.1 由值创建流
你可以使用静态方法Stream.of，通过显式值创建一个流。它可以接受任意数量的参数。例如，以下代码直接使用Stream.of创建了一个字符串流。然后，你可以将字符串转换为大写，再一个个打印出来:
```java
Stream<String> stream = Stream.of("Java 8 ", "Lambdas ", "In ", "Action"); 
stream.map(String::toUpperCase).forEach(System.out::println);
```
你可以使用empty得到一个空流，如下所示: 
```java
Stream<String> emptyStream = Stream.empty();
```

## 7.2 由数组创建流
你可以使用静态方法Arrays.stream从数组创建一个流。它接受一个数组作为参数。例如，你可以将一个原始类型int的数组转换成一个IntStream，如下所示:
```java
int[] numbers = {2, 3, 5, 7, 11, 13}; 
int sum = Arrays.stream(numbers).sum();
```

## 7.3 由文件生成流 
Java中用于处理文件等I/O操作的NIO API(非阻塞 I/O)已更新，以便利用Stream API。 java.nio.file.Files中的很多静态方法都会返回一个流。例如，一个很有用的方法是 Files.lines，它会返回一个由指定文件中的各行构成的字符串流。

## 7.4 由函数生成流:创建无限流
Stream API提供了两个静态方法来从函数生成流:Stream.iterate和Stream.generate。 这两个操作可以创建所谓的无限流:不像从固定集合创建的流那样有固定大小的流。由iterate 2 和generate产生的流会用给定的函数按需创建值，因此可以无穷无尽地计算下去!一般来说， 应该使用limit(n)来对这种流加以限制，以避免打印无穷多个值。

### 7.4.1 迭代 
```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```
此操作将生成一个无限流——这个流没有结尾，因为值是按需计算的，可以永远计算下去。我们说这个流是无界的。正如我们前面所讨论的，这是流和集合之间的一个关键区别。我们使用limit方法来显式限制流的大小。这里只选择了前10个偶数。然后可以调用forEach终端操作来消费流，并分别打印每个元素。

### 7.4.2 生成
与iterate方法类似，generate方法也可让你按需生成一个无限流。但generate不是依次 对每个新生成的值应用函数的。它接受一个Supplier<T>类型的Lambda提供新的值。我们先来看一个简单的用法:这段代码将生成一个流，其中有五个0到1之间的随机双精度数。 
```java
Stream.generate(Math::random)
          .limit(5)
          .forEach(System.out::println);
```

# 8. 小结
这一章很长，但是很有收获!现在你可以更高效地处理集合了。事实上，流让你可以简洁地表达复杂的数据处理查询。此外，流可以透明地并行化。以下是你应从本章中学到的关键概念。
- Streams API可以表达复杂的数据处理查询。常用的流操作总结在表5-1中。
- 你可以使用filter、distinct、skip和limit对流做筛选和切片。
- 你可以使用map和flatMap提取或转换流中的元素。 
- 你可以使用findFirst和findAny方法查找流中的元素。你可以用allMatch、noneMatch和anyMatch方法让流匹配给定的谓词。
- 这些方法都利用了短路:找到结果就立即停止计算;没有必要处理整个流。
- 你可以利用reduce方法将流中所有的元素迭代合并成一个结果，例如求和或查找最大元素。
- filter和map等操作是无状态的，它们并不存储任何状态。reduce等操作要存储状态才能计算出一个值。sorted和distinct等操作也要存储状态，因为它们需要把流中的所有元素缓存起来才能返回一个新的流。这种操作称为有状态操作。
- 流有三种基本的原始类型特化:IntStream、DoubleStream和LongStream。它们的操作也有相应的特化。
- 流不仅可以从集合创建，也可从值、数组、文件以及iterate与generate等特定方法创建。
- 无限流是没有固定大小的流。

## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！
