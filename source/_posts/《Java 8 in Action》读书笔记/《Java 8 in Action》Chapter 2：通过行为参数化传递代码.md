---
layout: post
title: 《Java 8 in Action》Chapter 2：通过行为参数化传递代码
date: 2019-03-03 10:52:12
categories: Java
tags:
- Java8
- 读书笔记
description: 《Java 8 in Action》Chapter 2：通过行为参数化传递代码
---

你将了解行为参数化，这是Java 8非常依赖的一种软件开发模式，也是引入 Lambda表达式的主要原因。行为参数化就是可以帮助你处理频繁变更的需求的一种软件开发模式。一言以蔽之，它意味 着拿出一个代码块，把它准备好却不去执行它。这个代码块以后可以被你程序的其他部分调用。本章通过筛选苹果这个实际需求来一步步引出Lambda表达式，同时我也会把代码贴出来，读完你会看到代码是如何一步一步的向Lambda转化。多代码来袭，保护我方ADC！！

## 代码演化
### 1.实习生版本
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.ArrayList;
import java.util.List;

/**
 * 筛选绿色苹果
 * @author lujiahao
 * @date 2019-02-19 18:28
 */
public class FilterAppleV0 {
    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        List<Apple> greenAppleList = new ArrayList<>();
        
        for (Apple apple : appleList) {
            if ("green".equals(apple.getColor())) {
                greenAppleList.add(apple);
            }
        }

        System.out.println("原集合:" + appleList);
        System.out.println("绿苹果集合:" + greenAppleList);
    }
}
```

这种之所以称之为实习生版本，是因为此种写法比较初级，所有代码在一个方法中实现。没有进行方法的抽取，不符合面向对象的理念，希望大家在编码工作时避免这种写法。

### 2.方法抽取版本
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.ArrayList;
import java.util.List;

/**
 * 筛选绿色苹果
 * @author lujiahao
 * @date 2019-02-19 18:30
 */
public class FilterAppleV1 {

    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        System.out.println("原集合:" + appleList);

        List<Apple> filterGreenApples = filterGreenApples(appleList);
        System.out.println("绿苹果集合:" + filterGreenApples);
    }

    /**
     * 筛选绿色苹果
     * @param appleList
     * @return
     */
    public static List<Apple> filterGreenApples(List<Apple> appleList) {
        List<Apple> resultList = new ArrayList<>();
        for (Apple apple : appleList) {
            if ("green".equals(apple.getColor())) {
                resultList.add(apple);
            }
        }
        return resultList;
    }
}
```

此版本对筛选绿色苹果的方法进行了简单的抽取，相较于上个版本有了很大的提升。然而，如果需求方改变想法，想筛选红色的苹果。复制filterGreenApples() 方法并将其中的绿色筛选条件改为红色，确实可以实现。但是，这样有太多重复的模板代码，不是良好的编码规范。因此，我们将筛选条件颜色进一步抽象化。

### 3.筛选条件作为参数传入
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.ArrayList;
import java.util.List;

/**
 * 需要判断的属性作为参数传入
 * @author lujiahao
 * @date 2019-02-19 18:30
 */
public class FilterAppleV2 {

    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        System.out.println("原集合:" + appleList);

        List<Apple> filterGreenApples = filterApples(appleList, "green");
        System.out.println("筛选绿色苹果:" + filterGreenApples);

        List<Apple> filterRedApples = filterApples(appleList, "red");
        System.out.println("筛选红色苹果:" + filterRedApples);
    }

    /**
     * 筛选特定颜色苹果
     * @param appleList
     * @return
     */
    public static List<Apple> filterApples(List<Apple> appleList, String color) {
        List<Apple> resultList = new ArrayList<>();
        for (Apple apple : appleList) {
            if (color.equals(apple.getColor())) {
                resultList.add(apple);
            }
        }
        return resultList;
    }
}
```

满足了颜色的筛选条件，然而需求方又灵光一闪，筛选大于150克的苹果。无论是复制filterApples() 方法，还是增加重量作为参数传入，都是不推荐的编码习惯。第一种方法复制了大部分的代码来实现遍历，它打破了DRY（Don’t Repeat Yourself）的软件工程原则；第二种方法并不能考虑到所有情况，并且每次修改都对原有代码产生了影响，无法做到修改对外封闭的原则。

### 4.行为参数化
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.ArrayList;
import java.util.List;

/**
 * 行为参数化
 * @author lujiahao
 * @date 2019-02-19 18:30
 */
public class FilterAppleV3 {
    
    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        System.out.println("原集合:" + appleList);

        List<Apple> filterGreenApples = filterApples(appleList, new AppleGreenColorPredicate());
        System.out.println("筛选绿色苹果:" + filterGreenApples);

        List<Apple> filterHeavyApples = filterApples(appleList, new AppleHeavyWeightPredicate());
        System.out.println("筛选重量大于150苹果:" + filterHeavyApples);
    }

    /**
     * 筛选绿色苹果
     * @param appleList
     * @return
     */
    public static List<Apple> filterApples(List<Apple> appleList, ApplePredicate predicate) {
        List<Apple> resultList = new ArrayList<>();
        for (Apple apple : appleList) {
            // 谓词对象封装了条件
            if (predicate.filter(apple)) {
                resultList.add(apple);
            }
        }
        return resultList;
    }

    public interface ApplePredicate {
        boolean filter(Apple apple);
    }

    public static class AppleHeavyWeightPredicate implements ApplePredicate {
        @Override
        public boolean filter(Apple apple) {
            return apple.getWeight() > 150;
        }
    }

    public static class AppleGreenColorPredicate implements ApplePredicate {
        @Override
        public boolean filter(Apple apple) {
            return "green".equals(apple.getColor());
        }
    }
}
```

我们对苹果的所有属性进行更高一个层次的抽象建模，通过定义ApplePredicate 接口，AppleHeavyWeightPredicate 和 AppleGreenColorPredicate 分别实现该接口来达到进行不同的筛选功能。客户端调用中创建不同的实现类，对于filterApple() 方法而言，是传入了不同的行为，即行为参数化。行为参数化：让方法接受多种行为(或战略)作为参数，并在内部使用，来完成不同的行为。
其原理如下图所示：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/01.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%202%EF%BC%9A%E9%80%9A%E8%BF%87%E8%A1%8C%E4%B8%BA%E5%8F%82%E6%95%B0%E5%8C%96%E4%BC%A0%E9%80%92%E4%BB%A3%E7%A0%81/%E8%A1%8C%E4%B8%BA%E5%8F%82%E6%95%B0%E5%8C%96.png)

### 5.匿名内部类
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.ArrayList;
import java.util.List;

/**
 * 使用匿名类
 * @author lujiahao
 * @date 2019-02-19 18:30
 */
public class FilterAppleV4 {
    
    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        System.out.println("原集合:" + appleList);

        List<Apple> filterGreenApples = filterApples(appleList, new ApplePredicate() {
            @Override
            public boolean filter(Apple apple) {
                return "green".equals(apple.getColor());
            }
        });
        System.out.println("筛选绿色苹果:" + filterGreenApples);

        List<Apple> filterHeavyApples = filterApples(appleList, new ApplePredicate() {
            @Override
            public boolean filter(Apple apple) {
                return apple.getWeight() > 150;
            }
        });
        System.out.println("筛选重量大于150苹果:" + filterHeavyApples);
    }

    /**
     * 筛选绿色苹果
     * @param appleList
     * @return
     */
    public static List<Apple> filterApples(List<Apple> appleList, ApplePredicate predicate) {
        List<Apple> resultList = new ArrayList<>();
        for (Apple apple : appleList) {
            // 谓词对象封装了条件
            if (predicate.filter(apple)) {
                resultList.add(apple);
            }
        }
        return resultList;
    }

    public interface ApplePredicate {
        boolean filter(Apple apple);
    }
}
```

当每次有新的查询需求提出，都要新建一个实现类，随着条件越来越多，实现类的数量也在急剧上升。此时，通过使用匿名内部类的方式，来减少实现类过多的模板代码。然而，匿名内部类并非完美，第一，它往往很笨重，因为它占用了很多空间；第二，很多程序员觉得它用起来很让人费解。

### 6.使用 Lambda 表达式
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.ArrayList;
import java.util.List;

/**
 * 使用Lambda表达式
 * @author lujiahao
 * @date 2019-02-19 18:30
 */
public class FilterAppleV5 {
    
    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        System.out.println("原集合:" + appleList);

        List<Apple> filterGreenApples = filterApples(appleList, (Apple apple) -> "green".equals(apple.getColor()));
        System.out.println("筛选绿色苹果:" + filterGreenApples);

        List<Apple> filterHeavyApples = filterApples(appleList, (Apple apple) -> apple.getWeight() > 150);
        System.out.println("筛选重量大于150苹果:" + filterHeavyApples);
    }

    /**
     * 筛选绿色苹果
     * @param appleList
     * @return
     */
    public static List<Apple> filterApples(List<Apple> appleList, ApplePredicate predicate) {
        List<Apple> resultList = new ArrayList<>();
        for (Apple apple : appleList) {
            // 谓词对象封装了条件
            if (predicate.filter(apple)) {
                resultList.add(apple);
            }
        }
        return resultList;
    }

    public interface ApplePredicate {
        boolean filter(Apple apple);
    }
}
```

不得不承认这代码看上去比先前干净很多，而且它看起来更像是在陈述问题本身，更加通俗易懂。

### 7.List 类型抽象化
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.*;

/**
 * List类型抽象话
 *
 * @author lujiahao
 * @date 2019-02-19 18:30
 */
public class FilterAppleV6 {

    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        System.out.println("原集合:" + appleList);

        List<Apple> filterGreenApples = filter(appleList, (Apple apple) -> "green".equals(apple.getColor()));
        System.out.println("筛选绿色苹果:" + filterGreenApples);

        System.out.println("=============================================");

        List<Integer> numberList = Arrays.asList(1, 2, 3);
        System.out.println("原集合:" + numberList);
        
        List<Integer> numbers = filter(numberList, (Integer i) -> i % 2 == 0);
        System.out.println("能被2整除的数:" + numbers);
    }

    /**
     * 筛选绿色苹果
     */
    public static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> resultList = new ArrayList<>();
        for (T t : list) {
            // 谓词对象封装了条件
            if (predicate.filter(t)) {
                resultList.add(t);
            }
        }
        return resultList;
    }

    public interface Predicate<T> {
        boolean filter(T t);
    }
}
```

在通往抽象的路上，我们还可以更进一步。目前，filterApples方法还只适用于Apple。还可以将List类型抽象化，从而支持所有类型。

### 8.演化小结
这一路演化中我们可以看出代码是如何一步一步转化的更加简洁更加优雅，对此我们进行总结：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/01.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%202%EF%BC%9A%E9%80%9A%E8%BF%87%E8%A1%8C%E4%B8%BA%E5%8F%82%E6%95%B0%E5%8C%96%E4%BC%A0%E9%80%92%E4%BB%A3%E7%A0%81/%E6%BC%94%E5%8C%96%E5%B0%8F%E7%BB%93.png)
## 实例
### 1.用 Comparator 排序
```java
package com.lujiahao.learnjava8.chapter2;

import java.util.Comparator;
import java.util.List;

/**
 * 用 Comparator 排序
 * @author lujiahao
 * @date 2019-03-02 18:34
 */
public class ComparatorDemo {
    public static void main(String[] args) {
        List<Apple> appleList = DataUtil.generateApples();
        System.out.println("原集合:" + appleList);

        appleList.sort(new Comparator<Apple>() {
            @Override
            public int compare(Apple o1, Apple o2) {
                return o1.getWeight().compareTo(o2.getWeight());
            }
        });
        System.out.println("按重量升序:" + appleList);

        appleList.sort((Apple a1, Apple a2) -> a1.getColor().compareTo(a2.getColor()));
        System.out.println("按颜色字典排序:" + appleList);
    }
}
```

### 2.用 Runnable 执行代码块
```java
package com.lujiahao.learnjava8.chapter2;

/**
 * 用 Runnable 执行代码块
 * @author lujiahao
 * @date 2019-03-02 18:42
 */
public class RunnableDemo {
    public static void main(String[] args) {
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello Java 8!");
            }
        });
        t.start();

        Thread t1 = new Thread(() -> System.out.println("Hello Lambda!"));
        t1.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 3.GUI 事件处理
```java
Button button = new Button(“Send”);
button.setOnAction(new EventHandler<ActionEvent>() {
    public void handle(ActionEvent event) {
        lable.setText(“Send!!”);
    }
}

button.setOnAction((ActionEvent event) -> lable.setText(“Send!!”));
```

小猿之前搞安卓开发的，各种控件的监听都是这个样子，想想以前各种代码啊啊啊~~~

## 总结
以下是你应从本章中学到的关键概念。
* 行为参数化，就是一个方法接受多个不同的行为作为参数，并在内部使用它们，完成不同行为的能力。
* 行为参数化可让代码更好地适应不断变化的要求，减轻未来的工作量。
* 传递代码，就是将新行为作为参数传递给方法。但在Java 8之前这实现起来很啰嗦。为接口声明许多只用一次的实体类而造成的啰嗦代码，在Java 8之前可以用匿名类来减少。
* Java API包含很多可以用不同行为进行参数化的方法，包括排序、线程和GUI处理。

## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！！
