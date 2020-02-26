---
layout: post
title: 《Java 8 in Action》Chapter 10：用Optional取代null
date: 2019-03-22 11:34:52
categories: Java
tags:
- Java8
- 读书笔记
description: 《Java 8 in Action》Chapter 10：用Optional取代null
---

> 1965年，英国一位名为Tony Hoare的计算机科学家在设计ALGOL W语言时提出了null引用的想法。ALGOL W是第一批在堆上分配记录的类型语言之一。Hoare选择null引用这种方式，“只是因为这种方法实现起来非常容易”。虽然他的设计初衷就是要“通过编译器的自动检测机制，确保所有使用引用的地方都是绝对安全的”，他还是决定为null引用开个绿灯，因为他认为这是为“不存在的值”建模最容易的方式。很多年后，他开始为自己曾经做过这样的决定而后悔不已，把它称为“我价值百万的重大事物”。实际上，Hoare的这段话低估了过去五十年来数百万程序员为修复空引用所耗费的代价。近十年出现的大多数现代程序设计语言1，包括Java，都采用了同样的设计方式，其原因是为了与更老的语言保持兼容，或者就像Hoare曾经陈述的那样，“仅仅是因为这样实现起来更加容易”。

# 1. 如何为确实的值建模
```java
public class Person {
        private Car car;
        public Car getCar() { return car; }
    }
public class Car {
        private Insurance insurance;
        public Insurance getInsurance() { return insurance; }
}
public class Insurance {
    private String name;
    public String getName() { return name; }
}
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```
上面这段代码的问题就在于，如果person没有车，就会造成空指针异常。

## 1.1 采用防御式检查减少NullPointerException
### 1.1.1 深层质疑
简单来说就是在需要的地方添加null检查
```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
            return insurance.getName();
        }
    }
    return "Unknown";
}
```
上述代码不具备扩展性，同时还牺牲了代码的可读性。

### 1.1.2 过多的退出语句
```java
public String getCarInsuranceName(Person person) {
    if (person == null) {
        return "Unknown";
    }
    Car car = person.getCar();
    if (car == null) {
        return "Unknown";
    }
    Insurance insurance = car.getInsurance();
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
}
```
这种模式中方法的退出点有四处，使得代码的维护异常艰难。

## 1.2 null带来的种种问题
* 它是错误之源。 NullPointerException是目前Java程序开发中最典型的异常。它会使你的代码膨胀。
* 它让你的代码充斥着深度嵌套的null检查，代码的可读性糟糕透顶。
* 它自身是毫无意义的。 null自身没有任何的语义，尤其是是它代表的是在静态类型语言中以一种错误的方式对缺失变量值的建模。
* 它破坏了Java的哲学。 Java一直试图避免让程序员意识到指针的存在，唯一的例外是:null指针。
* 它在Java的类型系统上开了个口子。 null并不属于任何类型，这意味着它可以被赋值给任意引用类型的变量。这会导致问题， 原因是当这个变量被传递到系统中的另一个部分后，你将无法获知这个null变量最初赋值到底是什么类型。

## 1.3 其他语言中null的替代品
- Groovy中的安全导航操作符
- Haskell中的Maybe类型
- Scala中的Option[T]

# 2. Optional类入门
变量存在时，Optional类只是对类简单封装。变量不存在时，缺失的值会被建模成一个“空”的Optional对象，由方法Optional.empty()返回。Optional.empty()方法是一个静态工厂方法，它返回Optional类的特定单一实例。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/10.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2010%EF%BC%9A%E7%94%A8Optional%E5%8F%96%E4%BB%A3null/1.png)

引入Optional类的意图并非要消除每一个null引用，相反的是，它的目标是帮助开发者更好地设计出普适的API。

# 3. 应用Optional的几种模式

## 3.1 创建Optional对象
### 3.1.1 声明一个空的Optional
正如前文已经提到，你可以通过静态工厂方法Optional.empty，创建一个空的Optional对象:
```java
Optional<Car> optCar = Optional.empty();
```
### 3.1.2 依据一个非空值创建Optional
你还可以使用静态工厂方法Optional.of，依据一个非空值创建一个Optional对象:
```java
Optional<Car> optCar = Optional.of(car);
```
如果car是一个null，这段代码会立即抛出一个NullPointerException，而不是等到你试图访问car的属性值时才返回一个错误。

### 3.2.3 可接受null的Optional
最后，使用静态工厂方法Optional.ofNullable，你可以创建一个允许null值的Optional对象:
```java
Optional<Car> optCar = Optional.ofNullable(car);
```
如果car是null，那么得到的Optional对象就是个空对象。

## 3.2 使用map从Optional对象中提取和转换值
从对象中提取信息是一种比较常见的模式。
```java
String name = null;
    if(insurance != null){
        name = insurance.getName();
    }
为了支持这种模式，Optional提供了一个map方法。
Optional<Insurance> optInsurance = Optional.ofNullable(insurance); 
Optional<String> name = optInsurance.map(Insurance::getName);
```
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/10.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2010%EF%BC%9A%E7%94%A8Optional%E5%8F%96%E4%BB%A3null/2.png)

## 3.3 使用flatMap链接Optional对象
使用流时，flatMap方法接受一个函数作为参数，这个函数的返回值是另一个流。 这个方法会应用到流中的每一个元素，最终形成一个新的流的流。但是flagMap会用流的内容替换每个新生成的流。换句话说，由方法生成的各个流会被合并或者扁平化为一个单一的流。
```java
public String getCarInsuranceName(Optional<Person> person) { return person.flatMap(Person::getCar)
                     .flatMap(Car::getInsurance)
                     .map(Insurance::getName)
                     .orElse("Unknown");
}
```
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/10.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2010%EF%BC%9A%E7%94%A8Optional%E5%8F%96%E4%BB%A3null/3.png)

## 3.4 默认行为及解引用Optional对象
1. get()是这些方法中最简单但又最不安全的方法。如果变量存在，它直接返回封装的变量值，否则就抛出一个NoSuchElementException异常。所以，除非你非常确定Optional变量一定包含值，否则使用这个方法是个相当糟糕的主意。此外，这种方式即便相对于嵌套式的null检查，也并未体现出多大的改进。
2. orElse(T other)是我们在代码清单10-5中使用的方法，正如之前提到的，它允许你在 Optional对象不包含值时提供一个默认值。
3. orElseGet(Supplier<? extends T> other)是orElse方法的延迟调用版，Supplier 方法只有在Optional对象不含值时才执行调用。如果创建默认值是件耗时费力的工作，你应该考虑采用这种方式(借此提升程序的性能)，或者你需要非常确定某个方法仅在 Optional为空时才进行调用，也可以考虑该方式(这种情况有严格的限制条件)。
4. orElseThrow(Supplier<? extends X> exceptionSupplier)和get方法非常类似，它们遭遇Optional对象为空时都会抛出一个异常，但是使用orElseThrow你可以定制􏳵希望抛出的异常类型。
5. ifPresent(Consumer<? super T>)让你能在变量值存在时执行一个作为参数传入的方法，否则就不进行任何操作。

## 3.5 两个Optional对象的组合
```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (person.isPresent() && car.isPresent()) {
        return Optional.of(findCheapestInsurance(person.get(), car.get())); 
    } else {
        return Optional.empty();
    }
}
```

## 3.6 使用filter剔除特定的值
filter方法接受一个谓词作为参数。如果Optional对象的值存在，并且它符合谓词的条件， filter方法就返回其值;否则它就返回一个空的Optional对象。
```java
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
       System.out.println("ok”);
}
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance ->
                        "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```

Optional类中的方法进行了分类和概括:
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/10.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2010%EF%BC%9A%E7%94%A8Optional%E5%8F%96%E4%BB%A3null/4.png)
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/10.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2010%EF%BC%9A%E7%94%A8Optional%E5%8F%96%E4%BB%A3null/5.png)


# 4. 使用Optional的实战示例
## 4.1 用Optional封装可能为null的值
```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```
每次你希望安全地对潜在为null的对象进行转换，将其替换为Optional对象时，都可以考虑使用这种方法。

## 4.2 异常与Optional的对比
```java
public static Optional<Integer> stringToInt(String s) {
try {
return Optional.of(Integer.parseInt(s));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```
我们的建议是，你可以将多个类似的方法封装到一个工具类中，让我们称之为OptionalUtility。通过这种方式，你以后就能直接调用OptionalUtility.stringToInt方法，将String转换为一个Optional<Integer>对象，而不再需要记得你在其中封装了笨拙的 try/catch的逻辑了。

## 4.3 把所有内容结合起来
```java
public int readDuration(Properties props, String name) {
    String value = props.getProperty(name);
    if (value != null) {
        try {
            int i = Integer.parseInt(value);
            if (i > 0) {
                return i;
            }
        } catch (NumberFormatException nfe) { }
    }
    return 0; 
}
// 优化版本
public int readDuration(Properties props, String name) {
        return Optional.ofNullable(props.getProperty(name))
                        .flatMap(OptionalUtility::stringToInt)
                        .filter(i -> i > 0)
                        .orElse(0);
}
```

# 5. 小结
这一章中，你学到了以下的内容。
1. null引用在上被引入到程序设计语言中，目的是为了表示变量值的。
2. Java 8中引入了一个新的类java.util.Optional<T>，对存在或缺失的变量值进行建模。
3. 你可以使用静态工厂方法Optional.empty、Optional.of以及Optional.ofNullable创建Optional对象。
4. Optional类支持多种方法，比如map、flatMap、filter，它们在概念上与Stream类中对应的方法十分相似。
5. 使用Optional会使你更积极地解引用Optional对象，以应对变量值缺失的问题，最终，你能更有效地止代码中出现不而至的空指针异常。
6. 使用Optional能帮助你设计更好的API，用户只需要阅读方法签名，就能了解该方法是否接受一个Optional类型的值。


## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！！
