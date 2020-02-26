---
layout: post
title: 《Java 8 in Action》Chapter 11：CompletableFuture：组合式异步编程
date: 2019-03-24 11:34:52
categories: Java
tags:
- Java8
- 读书笔记
description: 《Java 8 in Action》Chapter 11：CompletableFuture：组合式异步编程
---

某个网站的数据来自Facebook、Twitter和Google，这就需要网站与互联网上的多个Web服务通信。可是，你并不希望因为等待某些服务的响应，阻塞应用程序的运行，浪费数十亿宝贵的CPU时钟周期。比如，不要因为等待Facebook的数据，暂停对来自Twitter的数据处理。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/%E5%85%B8%E5%9E%8B%E6%B7%B7%E8%81%9A%E5%BA%94%E7%94%A8.png)

第7章中介绍的分支/合并框架以及并行流是实现并行处理的宝贵工具;它们将一个操作切分为多个子操作，在多个不同的核、CPU甚至是机器上并行地执行这些子操作。与此相反，如果你的意图是实现并发，而非并行，或者你的主要目标是在同一个CPU上执行几个松耦合的任务，充分利用CPU的核，让其足够忙碌，从而最大化程序的吞吐量，那么你其实真正想做的是避免因为等待远程服务的返回，或者对数据库的查询，而阻塞线程的执行，浪费宝贵的计算资源，因为这种等待的时间很可能相当长。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/%E5%B9%B6%E5%8F%91%E5%92%8C%E5%B9%B6%E8%A1%8C.png)

# 1. Future接口
Future接口在Java 5中被引入，设计初衷是对将来某个时刻会发生的结果进行建模。它建模了一种异步计算，返回一个执行运算结果的引用，当运算结束后，这个引用被返回给调用方。在Future中触发那些潜在耗时的操作把调用线程解放出来，让它能继续执行其他有价值的工作，不再需要等待耗时的操作完成。Future的另一个优点是它比更底层的Thread更易用。要使用Future，通常你只需要将耗时的操作封装在一个Callable对象中，再将它提交给ExecutorService。使用Future以异步的方式执行一个耗时的操作:
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/%E4%BD%BF%E7%94%A8Future%E4%BB%A5%E5%BC%82%E6%AD%A5%E7%9A%84%E6%96%B9%E5%BC%8F%E6%89%A7%E8%A1%8C%E4%B8%80%E4%B8%AA%E8%80%97%E6%97%B6%E7%9A%84%E6%93%8D%E4%BD%9C.png)

线程可以在ExecutorService以并发方式调用另一个线程执行耗时操作的同时，去执行一些其他的任务。接着，如果你已经运行到没有异步操作的结果就无法继续任何有意义的工作时，可以调用它的get方法去获取操作的结果。如果操作已经完成，该方法会立刻返回操作的结果，否则它会阻塞你的线程，直到操作完成，返回相应的结果。如果该长时间运行的操作永远不返回了会怎样?Future提供了一个无需任何参数的get方法，推荐使用重载版本的get方法，它接受一个超时的参数，可以定义线程等待Future结果的最长时间，避免无休止的等待。下图是Future异步执行线程原理图。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/Future%E5%BC%82%E6%AD%A5%E6%89%A7%E8%A1%8C%E7%BA%BF%E7%A8%8B%E5%8E%9F%E7%90%86%E5%9B%BE.png)

# 2. 使用CompletableFuture构建异步应用
Future接口有一定的局限性，比如，我们很难表述Future结果之间的依赖性。因此我们引入了CompletableFuture。接下来通过一个“最佳价格查询器“的应用，它会查询多个在线商店，依据给定的产品或服务找出最低的价格，来展现CompletableFuture实现异步应用。通过此例你能学到这些：
* 如何编写异步API
* 如何让使用同步API的代码变为非阻塞代码
* 如何使用流水线将两个接续的异步操作合并为一个异步计算操作
* 如何以响应式的方式处理异步操作的完成事件


同步API和异步API：
- 同步API其实只是对传统方法调用的另一种称呼:你调用了某个方法，调用方在被调用方运行的过程中会等待，被调用方运行结束返回，调用方取得被调用方的返回值并继续运行。即使调用方和被调用方在不同的线程中运行，调用方还是需要等待被调用方结束运行，这就是阻塞式调用这个名词的由来。
- 异步API会直接返回，或者至少在被调用方计算完成之前，将它剩余的计算任务交给另一个线程去做，该线程和调用方是异步的——这就是非阻塞式调用的由来。执行剩余计算任务的线程会将它的计算结果返回给调用方。返回的方式要么是通过回调函数，要么是由调用方再次执行一个“等待，直到计算完成”的方法调用。

## 2.1 实战：实现异步API
### 2.1.1 同步方法
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/%E5%90%8C%E6%AD%A5%E6%96%B9%E6%B3%95.png)

> 同步操作中会为等待同步事件完成而等待1s，这种是无法接受的，对于程序体验来说是非常不好的。

### 2.1.2 将同步方法转换为异步方法
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/%E5%BC%82%E6%AD%A5%E6%96%B9%E6%B3%95.png)

> Java 5引入了java.util.concurrent.Future接口表示一个异步计算(即调用线程可以继续运行，不会因为调用方法而阻塞)的结果。这意味着Future是一个暂时还不可知值的处理器，这个值在计算完成后，可以通过调用它的get方法取得。这种方式下，在进行价格查询的同时，还能执行一些其他的任务，比如查询其他商店中商品的价格，不会阻塞在那里等待第一家商店返回请求的结果。最后，如果所有有意义的工作都已经完成，所有要执行的工作都依赖于商品价格时，再调用Future的get方法。执行了这个操作后，要么获得Future中封装的值(如果异步任务已经完成)，要么发生阻塞，直到该异步任务完成，期望的值能够访问。同时，如果某个商品价格计算发生异常，会将当前线程杀死，从而导致等待get方法返回结果的客户端永久地被阻塞。客户端可以使用重载版本的get方法，设置超时参数来避免。为了让客户端能了解无法提供请求商品价格的原因，你需要使用CompletableFuture的completeExceptionally方法将导致CompletableFuture内发生问题的异常抛出。

## 2.1.3 使用工厂方法supplyAsync创建CompletableFuture对象
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/%E5%BC%82%E6%AD%A5%E6%96%B9%E6%B3%95(%E4%BD%BF%E7%94%A8%E5%B7%A5%E5%8E%82%E6%96%B9%E6%B3%95).png)

> supplyAsync方法接受一个生产者(Supplier)作为参数，返回一个CompletableFuture对象，该对象完成异步执行后会读取调用生产者方法的返回值。生产者方法会交由ForkJoinPool池中的某个执行线程(Executor)运行，但是你也可以使用supplyAsync方法的重载版本，传递第二个参数指定不同的执行线程执行生产者方法。

# 3. 消除代码阻塞问题
## 3.1 顺序同步请求
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.3/%E9%A1%BA%E5%BA%8F%E5%90%8C%E6%AD%A5%E8%AF%B7%E6%B1%82.png)

## 3.2 使用并行流对请求进行并行操作
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.3/%E4%BD%BF%E7%94%A8%E5%B9%B6%E8%A1%8C%E6%B5%81%E5%AF%B9%E8%AF%B7%E6%B1%82%E8%BF%9B%E8%A1%8C%E5%B9%B6%E8%A1%8C%E6%93%8D%E4%BD%9C.png)

## 3.3 使用CompletableFuture发起异步请求
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.3/%E4%BD%BF%E7%94%A8CompletableFuture%E5%8F%91%E8%B5%B7%E5%BC%82%E6%AD%A5%E8%AF%B7%E6%B1%82.png)

CompletableFuture版本的程序似乎比并行流版本的程序还快那么一点儿。但是最后这个版本也不太令人满意。它们看起来不相伯仲，究其原因都一样:它们内部采用的是同样的通用线程池，默认都使用固定数目的线程，具体线程数取决于Runtime.getRuntime().availableProcessors()的返回值。然而，CompletableFuture具有一定的优势，因为它允许你对执行器(Executor)进行配置，尤其是线程池的大小，让它以更适合应用需求的方式进行配置，满足程序的要求，而这是并行流API无法提供的。
顺序执行和并行执行的原理对比：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.3/%E9%A1%BA%E5%BA%8F%E6%89%A7%E8%A1%8C%E5%92%8C%E5%B9%B6%E8%A1%8C%E6%89%A7%E8%A1%8C%E7%9A%84%E5%8E%9F%E7%90%86%E5%AF%B9%E6%AF%94.png)

图11-4的上半部分展示了使用单一流水线处理流的过程，我们看到，执行的流程(以虚线标识)是顺序的。事实上，新的CompletableFuture对象只有在前一个操作完全结束之后，才能创建。与此相反，图的下半部分展示了如何先将CompletableFutures对象聚集到一个列表中(即图中以椭圆表示的部分)，让对象们可以在等待其他对象完成操作之前就能启动。

## 3.4 使用CompletableFuture发起异步请求WithExecutor
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.3/%E4%BD%BF%E7%94%A8CompletableFuture%E5%8F%91%E8%B5%B7%E5%BC%82%E6%AD%A5%E8%AF%B7%E6%B1%82WithExecutor.png)

## 3.5 调用结果:
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.3/%E8%B0%83%E7%94%A8%E7%BB%93%E6%9E%9C.png)

## 3.6 并行——使用流还是CompletableFutures
目前为止，你已经知道对集合进行并行计算有两种方式:要么将其转化为并行流，利用map这样的操作开展工作，要么枚举出集合中的每一个元素，创建新的线程，在CompletableFuture内对其进行操作。后者提供了更多的灵活性，你可以调整线程池的大小，而这能帮助你确保整体的计算不会因为线程都在等待I/O而发生阻塞。
- 如果你进行的是计算密集型的操作，并且没有I/O，那么推荐使用Stream接口，因为实现简单，同时效率也可能是最高的(如果所有的线程都是计算密集型的，那就没有必要创建比处理器核数更多的线程)。 
- 如果你并行的工作单元还涉及等待I/O的操作(包括网络连接等待)，那么使用CompletableFuture灵活性更好，你可以像前文讨论的那样，依据等待/计算，或者 W/C的比率设定需要使用的线程数。这种情况不使用并行流的另一个原因是，处理流的流水线中如果发生I/O等待，流的延迟性会让我们很难判断到底什么时候触发了等待。


# 4. 对多个异步任务进行流水线操作
## 4.1 案例
通过在shop构成的流上采用流水线方式执行三次map操作，我们得到了结果。
* 第一个操作将每个shop对象转换成了一个字符串，该字符串包含了该 shop中指定商品的价格和折扣代码。
* 第二个操作对这些字符串进行了解析，在Quote对象中对它们进行转换。
* 第三个map会操作联系远程的Discount服务，计算出最终的折扣价格，并返回该价格及提供该价格商品的shop。

代码如图:
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.4/Discount.png)
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.4/Quote.png)
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.4/Shop11_4.png)
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.4/Test.png)

原理图：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.4/Discount原理图.png)

Java 8的CompletableFuture API提供了名为thenCompose的方法，它就是专门为这一目的而设计的，thenCompose方法允许你对两个异步操作进行流水线，第一个操作完成时，将其结果作为参数传递给第二个操作。换句话说，你可以创建两个CompletableFutures对象，对第一个CompletableFuture对象调用thenCompose，并向其传递一个函数。当第一个 CompletableFuture执行完毕后，它的结果将作为该函数的参数，这个函数的返回值是以第一 个CompletableFuture的返回做输入计算出的第二个CompletableFuture对象。thenCompose方法像CompletableFuture类中的其他方法一样，也提供了一个以Async后缀结尾的版本thenComposeAsync。通常而言，名称中不带Async的方法和它的前一个任务一样，在同一个线程中运行;而名称以Async结尾的方法会将后续的任务提交到一个线程池，所以每个任务是由不同的线程处理的。

## 4.2 thenCombine方法
将两个CompletableFuture对象结合起来，无论他们是否存在依赖。thenCombine方法，它接收名为BiFunction的第二参数，这个参数 定义了当两个CompletableFuture对象完成计算后，结果如何合并。同thenCompose方法一样， thenCombine方法也提供有一个Async的版本。这里，如果使用thenCombineAsync会导致BiFunction中定义的合并操作被提交到线程池中，由另一个任务以异步的方式执行。

代码图：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.5/thenCombine.png)

原理图：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.5/thenCombine%E5%8E%9F%E7%90%86%E5%9B%BE.png)

## 4.3 响应CompletableFuture的completion事件
Java 8的CompletableFuture通过thenAccept方法提供了这一功能，它接收 CompletableFuture执行完毕后的返回值做参数。thenAccept方法也提供 了一个异步版本，名为thenAcceptAsync。异步版本的方法会对处理结果的消费者进行调度， 从线程池中选择一个新的线程继续执行，不再由同一个线程完成CompletableFuture的所有任 务。因为你想要避免不必要的上下文切换，更重要的是你希望避免在等待线程上浪费时间，尽快响应CompletableFuture的completion事件，所以这里没有采用异步版本。

## 4.3.1 实战
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/11.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%2011%EF%BC%9ACompletableFuture%EF%BC%9A%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/11.5/thenAccept%E5%AE%9E%E6%88%98.png)

# 5. 小结

- 执行比较操作时,尤其是那些依赖一个或多个远程服务的操作，使用异步任务可以改善程序的性能，加快程序的响应速度。
- 你应该尽可能地为客户提供异步API。使用CompletableFuture类提供的特性，你能够轻松地实现这一目标。
- CompletableFuture类还提供了异常管理的机制，让你有机会抛出/管理异步任务执行中发生的异常。
- 将同步API的调用封装到一个CompletableFuture中，你能够以异步的方式使用其结果。
- 如果异步任务之间相互独立，或者它们之间某一些的结果是另一些的输入，你可以将这些异步任务构造或者合并成一个。
- 你可以为CompletableFuture注册一个回调函数，在Future执行完毕或者它们计算的结果可用时，针对性地执行一些程序。
- 你可以决定在什么时候结束程序的运行，是等待由CompletableFuture对象构成的列表中所有的对象都执行完毕，还是只要其中任何一个首先完成就中止程序的运行。


## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！！
