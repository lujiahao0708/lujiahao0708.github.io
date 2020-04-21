---
layout: post
title: 《Java 8 in Action》Chapter 7：并行数据处理与性能
categories: Java
tags:
  - Java8
  - 读书笔记
description: 《Java 8 in Action》Chapter 7：并行数据处理与性能
abbrlink: e2be667d
date: 2019-03-17 11:34:52
---

> 在Java 7之前，并行处理数据集合非常麻烦。第一，你得明确地把包含数据的数据结构分成若干子部分。第二，你要给每个子部分分配一个独立的线程。第三，你需要在恰当的时候对它们进行同步来避免不希望出现的竞争条件，等待所有线程完成，最后把这些部分结果合并起来。Java 7引入了一个叫作分支/合并的框架，让这些操作更稳定、更不易出错。
Stream接口让你不用太费力气就能对数据集执行并行操作。它允许你声明性地将顺序流变为并行流。此外，你将看到Java是如何变戏法的，或者更实际地来说， 流是如何在幕后应用Java 7引入的分支/合并框架的。

# 1. 并行流
并行流就是一个把内容分成多个数据块，并用不同的线程分别处理每个数据块的流。
```java
public static long sequentialSum(long n) {
             return Stream.iterate(1L, i -> i + 1)
                          .limit(n)
                          .reduce(0L, Long::sum);
}
传统写法：
public static long iterativeSum(long n) {
        long result = 0;
        for (long i = 1L; i <= n; i++) {
            result += i;
        }
        return result;
}
```

## 1.1 将顺序流转换为并行流
可以把流转换成并行流，从而让前面的函数归约过程(也就是求和)并行运行——对顺序流调用parallel方法:
```java
public static long parallelSum(long n) {
        return Stream.iterate(1L, i -> i + 1)
                     .limit(n)
                     .parallel()
                     .reduce(0L, Long::sum);
}
```
在现实中，对顺序流调用parallel方法并不意味着流本身有任何实际的变化。它在内部实际上就是设了一个boolean标志，表示你想让调用parallel之后进行的所有操作都并行执行。类似地，你只需要对并行流调用sequential方法就可以把它变成顺序流。请注意，你可能以为把这两个方法结合起来，就可以更细化地控制在遍历流时哪些操作要并行执行，哪些要顺序执行。

> 配置并行流使用的线程池
> 看看流的parallel方法，你可能会想，并行流用的线程是从哪来的?有多少个?怎么自定义这个过程呢?
> 并行流内部使用了默认的ForkJoinPool，它默认的线程数量就是你的处理器数量，这个值是由Runtime.getRuntime().available- Processors()得到的。
> 但是你可以通过系统属性 java.util.concurrent.ForkJoinPool.common.parallelism来改变线程池大小，如下所示:
> System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","12");
> 这是一个全局设置，因此它将影响代码中所有的并行流。反过来说，目前还无法专为某个并行流指定这个值。一般而言，让ForkJoinPool的大小等于处理器数量是个不错的默认值，
> 除非你有很好的理由，否则我们强烈建议你不要修改它。

## 1.2 测量流性能
并行编程可能很复杂，有时候甚至有点违反直觉。如果用得不对(比如采用了一 个不易并行化的操作，如iterate)，它甚至可能让程序的整体性能更差，所以在调用那个看似神奇的parallel操作时，了解背后到底发生了什么是很有必要的。
并行化并不是没有代价的。并行化过程本身需要对流做递归划分，把每个子流的归纳操作分配到不同的线程，然后把这些操作的结果合并成一个值。但在多个内核之间移动数据的代价也可能比你想的要大，所以很重要的一点是要保证在内核中并行执行工作的时间比在内核之间传输数据的时间长。总而言之，很多情况下不可能或不方便并行化。然而，在使用 并行Stream加速代码之前，你必须确保用得对;如果结果错了，算得快就毫无意义了。 

## 1.3 正确使用并行流
错用并行流而产生错误的首要原因，就是使用的算法改变了某些共享状态。下面是另一种实现对前n个自然数求和的方法，但这会改变一个共享累加器:
```java
public static long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add)
    return accumulator.total;
}
public class Accumulator {
    public long total = 0;
    public void add(long value) { total += value; }
}
```
这段代码本身上就是顺序的，因为每次访问total都会出现数据竞争。接下来将这段代码改为并行：
```java
public static long sideEffectParallelSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
    return accumulator.total;}
System.out.println("SideEffect parallel sum done in: " + measurePerf(ParallelStreams::sideEffectParallelSum, 10_000_000L) +" msecs" );
Result: 5959989000692
Result: 7425264100768
Result: 6827235020033
Result: 7192970417739
Result: 6714157975331
Result: 7715125932481
SideEffect parallel sum done in: 49 msecs
```
这回方法的性能无关紧要了，唯一要紧的是每次执行都会返回不同的结果，都离正确值50000005000000差很远。这是由于多个线程在同时访问累加器，执行total += value，而这一句􏱵然看似简单，却不是一个原子操作。问题的根源在于，forEach中调用的方法有副作用，它会改变多个线程共享的对象的可变状态。要是你想用并行Stream又不想引发类似的意外，就必须避免这种情况。现在你知道了，共享可变状态会影响并行流以及并行计算。

## 1.4 高效使用并行流
- 如果有疑问，测量。把顺序流转成并行流轻而易举，但却不一定是好事。我们在本节中已经指出，并行流并不总是比顺序流快。此外，并行流有时候会和你的直觉不一致，所以在考虑选择顺序流还是并行流时，第一个也是最重要的建议就是用适当的基准来检查其性能。
- 留意装箱。自动装箱和拆箱操作会大大降低性能。Java 8中有原始类型流(IntStream、 LongStream、DoubleStream)来避免这种操作，但凡有可能都应该用这些流。
- 有些操作本身在并行流上的性能就比顺序流差。特别是limit和findFirst等依赖于元素顺序的操作，它们在并行流上执行的代价非常大。例如，findAny会比findFirst性能好，因为它不一定要按顺序来执行。你总是可以调用unordered方法来把有序流变成无序流。那么，如果你需要流中的n个元素而不是专门要前n个的话，对无序并行流调用 limit可能会比单个有序流(比如数据源是一个List)更高效。
- 还要考虑流的操作流水线的总计算成本。设N是要处理的元素的总数，Q是一个元素通过 流水线的大致处理成本，则N*Q就是这个对成本的一个粗略的定性估计。Q值较高就意味着使用并行流时性能好的可能性比较大。
- 对于较小的数据量，选择并行流几乎从来都不是一个好的决定。并行处理少数几个元素的好处还抵不上并行化造成的额外开销。
- 要考虑流背后的数据结构是否易于分解。例如，ArrayList的拆分效􏶲比LinkedList 高得多，因为前者用不着遍历就可以平均拆分，而后者则必须遍历。另外，用range工厂方法创建的原始类型流也可以快速分解。最后，你将在7.3节中学到，你可以自己实现Spliterator来完全掌握分解过程。
- 流自身的特点，以及流水线中的中间操作修改流的方式，都可能会改变分解过程的性能。例如，一个SIZED流可以分成大小相等的两部分，这样每个部分都可以比较高效地并行处理，但筛选操作可能丢弃的元素个数却无法预测，导致流本身的大小未知。
- 还要考虑终􏲧操作中合并步骤的代价是大是小(例如Collector中的combiner方法)。 如果这一步代价很大，那么组合每个子流产生的部分结果所付出的代价就可能会超出通过并行流得到的性能提升。

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/07.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%207%EF%BC%9A%E5%B9%B6%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B8%8E%E6%80%A7%E8%83%BD/1.png)

> 并行流背后使用的基础架构是Java 7中引入的分支/合并框架。

# 2. 分支/合并框架
分支/合并框架的目的是以递归方式将可以并行的任务拆分成更小的任务，然后将每个子任务的结果合并起来生成整体结果。它是ExecutorService接口的一个实现，它把子任务分配给线程池(称为ForkJoinPool)中的工作线程。

## 2.1 使用RecursiveTask
要把任务提交到这个池，必须创建RecursiveTask<R>的一个子类，其中R是并行化任务(以 及所有子任务)产生的结果类型，或者如果任务不返回结果，则是RecursiveAction类型(当然它可能会更新其他非局部机构)。要定义RecursiveTask，只需实现它唯一的抽象方法 compute:
```java
protected abstract R compute();
```
这个方法同时定义了将任务拆分成子任务的逻辑，以及无法再拆分或不方便再拆分时，生成单个子任务结果的逻辑。下图表示了递归任务的拆分过程：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/07.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%207%EF%BC%9A%E5%B9%B6%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B8%8E%E6%80%A7%E8%83%BD/2.png)

让我们试着用这个框架为一个数字范围(这里用一个 long[]数组表示)求和。如前所述，你需要先为RecursiveTask类做一个实现，就是下面代码清单中的ForkJoinSumCalculator。
```java
public class ForkJoinSumCalculator extends RecursiveTask<Long> {
    private final long[] numbers;
    private final int start;
    private final int end;

    public static final long THRESHOLD = 10_000;

    public ForkJoinSumCalculator(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    public ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            return computeSequentially();
        }
        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
        leftTask.fork();

        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length / 2, end);
        Long rightResult = rightTask.compute();
        Long leftResult = leftTask.join();
        return leftResult + rightResult;
    }

    private long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }
}
```
这里用了一个LongStream来生成包含前n个自然数的数组，然后创建一个ForkJoinTask (RecursiveTask的父类)，并把数组传递给代码清单7-2所示ForkJoinSumCalculator的公共构造函数。最后，你创建了一个新的ForkJoinPool，并把任务传给它的调用方法 。在ForkJoinPool中执行时，最后一个方法返回的值就是ForkJoinSumCalculator类定义的任务结果。
请注意在实际应用时，使用多个ForkJoinPool是没有什么意义的。正是出于这个原因，一般来说把它实例化一次，然后把实例保存在静态字段中，使之成为单例，这样就可以在软件中任何部分方便地重用了。这里创建时用了其默认的无参数构造函数，这意味着想让线程池使用JVM能够使用的所有处理器。更确切地说，该构造函数将使用Runtime.availableProcessors的返回值来决定线程􏶈使用的线程数。请注意availableProcessors方法虽然看起来是处理器， 但它实际上返回的是可用内核的数量，包括超线程生成的虚拟内核。
当把ForkJoinSumCalculator任务传给ForkJoinPool时，这个任务就由􏶈中的一个线程 执行，这个线程会调用任务的compute方法。该方法会检查任务是否小到足以顺序执行，如果不够小则会把要求和的数组分成两半，分给两个新的ForkJoinSumCalculator，而它们也由ForkJoinPool安排执行。因此，这一过程可以递归重复，把原任务分为更小的任务，直到满足不方便或不可能再进一步拆分的条件(本例中是求和的项目数小于等于10000)。这时会顺序计算每个任务的结果，然后由分支过程创建的(隐含的)任务二叉树遍历回到它的根。接下来会合并每个子任务的部分结果，从而得到总任务的结果。这一过程如下图所示。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/07.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%207%EF%BC%9A%E5%B9%B6%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B8%8E%E6%80%A7%E8%83%BD/3.png)

## 2.2 使用分支/合并框架的最佳做法
- 对一个任务调用join方法会阻塞调用方，直到该任务做出结果。因此，有必要在两个子任务的计算都开始之后再调用它。否则，你得到的版本会比原始的顺序算法更慢更复杂，因为每个子任务都必须等待另一个子任务完成才能启动。
- 不应该在RecursiveTask内部使用ForkJoinPool的invoke方法。相反，你应该始终直接调用compute或fork方法，只有顺序代码才应该用invoke来启动并行计算。
- 对子任务调用fork方法可以把它排进ForkJoinPool。同时对左边和右边的子任务调用它似乎很自然，但这样做的效􏶲要比直接对其中一个调用compute低。这样做你可以为其中一个子任务重用同一线程，从而避免在线程池中多分配一个任务造成的开销。
- 调试使用分支/合并框架的并行计算可能有点棘手。特别是你平常都在你喜欢的IDE里面看栈跟踪(stack trace)来找问题，但放在分支-合并并计算上就不行了，因为调用compute的线程并不是概念上的调用方，后者是调用fork的那个。
- 和并行流一样，你不应理所当然地认为在多核处理器上使用分支/合并框架就比顺序计算快。我们已经说过，一个任务可以分解成多个独立的子任务，才能让性能在并行化时有所提升。所有这些子任务的运行时间都应该比分出新任务所花的时间长;一个惯用方法是把输入/输出放在一个子任务里，计算放在另一个里，这样计算就可以和输入/输出同时进行。此外，在比较同一算法的顺序和并行版本的性能时还有别的因素要考虑。就像任何其他Java代码一样，分支/合并框架需要“预热”或者说要执行几遍才会被JIT编译器优化。这就是为什么在测量性能之前跑几遍程序很重要，我们的测试框架就是这么做的。同时还要知道，编译器内置的优化可能会为顺序版本带来一些优􏲵(例如执行死码分析——删去从未被使用的计算)。

## 2.3 工作窃取
实际中，每个子任务所花的时间可能天差地别，要么是因为划分策略效率低，要么是有不可预知的原因，比如磁盘访问慢，或是需要和外部任务协调执行。分支/合并框架工程用一种称为工作窃取(work stealing)的技术来解决这个问题。
在实际应用中，这意味着这些任务差不多被平均分配到ForkJoinPool中的所有线程上。每个线程都为分配给它的任务保存一个双向链式队列，每完成一个任务，就会从队列头上取出下一个任务开始执行。基于前面所述的原因，某个线程可能早早完成了分配给它的所有任务，也就是它的队列已经空了，而其他的线程还很忙。这时，这个线程并没有闲下来，而是随机选了一个别的线程，从队列的尾巴上“偷走”一个任务。这个过程一直继续下去，直到所有的任务都执行完毕，所有的队列都清空。这就是为什么要划成许多小任务而不是少数几个大任务，这有助于更好地在工作线程之间平衡负载。一般来说，这种工作窃取算法用于在池中的工作线程之间重新分配和平衡任务。 
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/07.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%207%EF%BC%9A%E5%B9%B6%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B8%8E%E6%80%A7%E8%83%BD/4.png)

# 3. Spliterator
Spliterator是Java 8中加入的另一个新接口;这个名字代表“可分迭代器”(splitable iterator)。和Iterator一样，Spliterator也用于遍历数据源中的元素，但它是为了并行执行而设计的。
```java
public interface Spliterator<T> {
        boolean tryAdvance(Consumer<? super T> action);
        Spliterator<T> trySplit();
        long estimateSize();
        int characteristics();
}
```
与往常一样，T是Spliterator遍历的元素的类型。tryAdvance方法的行为类似于普通的 Iterator，因为它会按顺序一个一个使用Spliterator中的元素，并且如果还有其他元素要遍历就返回true。但trySplit是专为Spliterator接口设计的，因为它可以把一些元素划出去分给第二个Spliterator(由该方法返回)，让它们两个并行处理。Spliterator还可通过 estimateSize方法估计还剩下多少元素要遍历，因为即使不那么确切，能快速算出来是一个值也有助于让拆分均匀一点。

## 3.1 拆分过程
将Stream拆分成多个部分的算法是一个递􏰒过程，如图7-6所示。第一步是对第一个 Spliterator调用trySplit，生成第二个Spliterator。第二步对这两个Spliterator调用 trysplit，这样总共就有了四个Spliterator。这个框架不断对Spliterator调用trySplit直到它返回null，表明它处理的数据结构不能再分割，如第三步所示。最后，这个递归拆分过程到第四步就终止了，这时所有的Spliterator在调用trySplit时都返回了null。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/07.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%207%EF%BC%9A%E5%B9%B6%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B8%8E%E6%80%A7%E8%83%BD/5.png)

    Spliterator的特性
    Spliterator接口声明的最后一个抽象方法是characteristics，它将返回一个int，代 表Spliterator本身特性集的编码。
    使用Spliterator的客户可以用这些特性来更好地控制和优化它的使用。
    表7-2总结了这些特性。(不幸的是，虽然它们在概念上与收集器的特性有重叠，编码却不一样。)
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/07.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%207%EF%BC%9A%E5%B9%B6%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B8%8E%E6%80%A7%E8%83%BD/6.png)

## 3.2 实现自定义Spliterator
略

# 4. 小结
在本章中，你了解了以下内容。
* 内部迭代让你可以并行处理一个流，而无需在代码中显式使用和􏷡调不同的线程。
* 虽然并行处理一个流很容易，却不能保证程序在所有情况下都运行得更快。并行软件的行为和性能有时是违反直觉的，因此一定要测量，确保你并没有把程序拖得更慢。
* 像并行流那样对一个数据集并行执行操作可以提升性能，特别是要处理的元素数量庞大，或处理单个元素特别耗时的时候。
* 从性能角度来看，使用正确的数据结构，如尽可能利用原始流而不是一般化的流，几乎总是比尝试并行化某些操作更为重要。
* 分支/合并框架让你得以用递归方式将可以并行的任务拆分成更小的任务，在不同的线程上执行，然后将各个子任务的结果合并起来生成整体结果。
* Spliterator定义了并行流如何拆分它要遍历的数据。


## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！
