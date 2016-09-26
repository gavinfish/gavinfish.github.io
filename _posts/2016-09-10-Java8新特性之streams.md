---
layout: post
title:  "Java8新特性之streams"
date:   2016-09-10
categories: Java
tags: Java API stream

---

在[第二章](./02-lambdas.md)中，我们通过学习lambda表达式，了解了如何能够在不创建额外类的情况下传递行为来帮助我们编写出简洁精练的代码。lambda表达式是一种通过使用函数式接口让开发者能够快速表达他们的想法的语言概念。设计API的时候将lambda，也就是那些使用了函数式接口的流畅的API（我们在[lambdas章节](./02-lambdas.md#lambda表达式是如何在Java8中工作的)中讨论过它们）记在脑子中，我们才能真正体验到lambda的强大，。

在Java8中引进的Stream API是使用lambda的API之一。就像SQL如何帮助你在数据库中形象地查询数据，Stream在Java集合计算上提供了一个形象的声明式的高层抽象来表示计算。形象的意思是指开发者只要写他们想写的，而不是关注他们该如何来写。在这一章中，我们将讨论对一个新的数据处理API的需求、`Collection`和`Stream`的区别，和如何在你的应用中使用Stream API。

> 这一节的代码在[ch03](https://github.com/shekhargulati/java8-the-missing-tutorial/tree/master/code/src/main/java/com/shekhargulati/java8_tutorial/ch03)包中

## 为什么我们需要一个新的数据处理抽象

在我的观点中，主要有两个原因：

1. `Collection` API没有提供高层的概念来查询数据，所以开发者被迫为琐碎的工作编写很多重复的代码。
2. 对`Collection`的并行操作在语言支持方面受到了限制。只能让开发者用Java的并发机制来让数据并行处理快速而有效。

## 在Java8之前的数据处理

看下面的代码并试图说出它的作用。

```java
public class Example1_Java7 {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();

        List<Task> readingTasks = new ArrayList<>();
        for (Task task : tasks) {
            if (task.getType() == TaskType.READING) {
                readingTasks.add(task);
            }
        }
        Collections.sort(readingTasks, new Comparator<Task>() {
            @Override
            public int compare(Task t1, Task t2) {
                return t1.getTitle().length() - t2.getTitle().length();
            }
        });
        for (Task readingTask : readingTasks) {
            System.out.println(readingTask.getTitle());
        }
    }
}
```

上面的代码将阅读任务按照它们标题的长度进行排序后输出。Java7的开发者成天要写这类的代码。为了写一个这样简单的程序，我们编写了15行代码。上面提到的代码的最大问题不是开发者要编写的代码的数量，而是它丢失了开发者的意图，也就是过滤阅读任务，根据标题长度排序，和转换成字符串列表。

## Java8中的数据处理

如下所示，上述的代码可以通过Java8的`Stream` API简化。

```java
public class Example1_Stream {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();

        List<String> readingTasks = tasks.stream()
                .filter(task -> task.getType() == TaskType.READING)
                .sorted((t1, t2) -> t1.getTitle().length() - t2.getTitle().length())
                .map(Task::getTitle)
                .collect(Collectors.toList());

        readingTasks.forEach(System.out::println);
    }
}
```

上面的代码构建了一个由许多流式操作组成的管道流，下面对其一一讲解。

* **stream()**: 通过在一个原始的集合上调用`stream`方法来创建一个流式管道流，而`tasks`就是`List<Task>`类型的。
* **filter(Predicate<T>)**: 这个操作从流中抽取符合断言的判定条件的元素。一旦你有了一个数据流，你可以在其上不调用或者多次调用中间操作。lambda表达式`task -> task.getType() == TaskType.READING`定义了一个断言来过滤所有的阅读任务。该lambda表达式的类型为`java.util.function.Predicate<Task>`。
* **sorted(Comparator<T>)**:这个操作返回一个根据由lambda表达式定义的比较器进行排序后的元素组成的数据流。在上面的例子中，这个比较器是`(t1, t2) -> t1.getTitle().length() - t2.getTitle().length()` 。
* **map(Function<T,R>)**: 这个操作对数据流中的元素都进行`Fuction<T,R>`的操作，并返回新的数据流。
* **collect(toList())**:这个操作将经过各种操作处理后的数据流中的元素收集到一个列表中。

### 为什么Java8编写的代码更好

我认为Java8的代码更好的理由如下：

1. Java8的代码清晰地展现出开发者的意图，如过滤、排序等。
2. 开发者通过Stream API的形式能够在一个高层的抽象上来表现出他们想要做什么，而不是他们如何来作。
3. Stream API为数据处理提供了一个统一的语言。现在当程序员讨论到数据处理时，它们将会有共同的词汇。当两个开发者谈论到`filter`方法时，你可以肯定他们都在使用一个数据过滤操作。
4. 处理数据时不需要重复的代码。用户不需要写专门的`for`循环，也不用创建临时集合来存储数据。所有的工作都可以通过Stream API来完成。
5. Stream不会修改你原来的集合——它们是免于变化的。

## 什么是Stream？

Stream是一些数据上的抽象视图。举例来说，Stream可以是列表、文件中的每行数据，或者其他任意元素的序列的一个视图。Stream API提供了可以连续执行，或者并行执行的操作集合。***开发者需要记住的是Stream是一个高层的抽象，而不是一个数据结构。Stream不会存储你的数据。*** Stream本身是**懒惰**的，只有使用到它们时才会开始计算。这使我们能够产生无数的数据流。在Java8中，你能够像下面一样编写一个Stream来产生无数的唯一标识码。

```
public static void main(String[] args) {
    Stream<String> uuidStream = Stream.generate(() -> UUID.randomUUID().toString());
}
```

在Stream的接口中，有很多像`of`、`generate`和`iterate`一样的静态工厂方法，它们可以用来创建Stream的实例。上面展示的`generate`方法以`Supplier`为参数。`Supplier`是一个函数式接口，用来描述一个不需要参数并返回一个值的函数。我们传递给`generate`方法一个供应者，那么当调用的时候，就能产生一个唯一标识码。

```java
Supplier<String> uuids = () -> UUID.randomUUID().toString()
```

如果我们运行上面的代码，那么什么都不会发生，因为Stream是懒惰的，它没有被使用之前，什么计算都不会进行。如果我们将代码更新成下面这样，我们将会看到在控制台上输出UUID。该程序将会不断运行。

```java
public static void main(String[] args) {
    Stream<String> uuidStream = Stream.generate(() -> UUID.randomUUID().toString());
    uuidStream.forEach(System.out::println);
}
```

Java8允许你在集合对象上调用`stream`方法来创建一个Stream。Stream支持数据处理操作，所以开发者可以用高层数据处理结构来表示计算过程。

## Collection vs Stream

下面的表格解释了Collection和Stream之间的不同。

![Collection vs Stream](https://whyjava.files.wordpress.com/2015/10/collection_vs_stream.png)

让我们来详细讨论一下外部迭代和内部迭代，以及延迟求值。

### 外部迭代 vs 内部迭代

上面所示的代码中，Java8中的Stream API和原来的Collection API的不同在于谁控制了迭代——是迭代器还是使用了迭代器的用户。Stream API的用户只是提供了他们想要使用的操作，然后迭代器将这些操作施加在内部集合中的每一个元素上。当迭代内部集合时，这个过程是由迭代器自己处理的，这被叫做**内部迭代**。而在Collection API中使用`for-each`结构是一个**外部迭代**的例子。

有些人可能会争论，在Collection API中我们不需要使用其中的迭代器，因为`for-each`结构将会处理它，但`for-each`只是使用了迭代器API的人工迭代的语法糖而已。`for-each`结构虽然很简单，却有一些缺点——1）他是内在连续的，2）它导致命令式代码，3）它很难并行化。

### 延迟计算

Stream不会进行计算直到一个最终的命令来调用它。在Stream API中的大多数操作返回一个Stream。这些操作不会被执行——它们只是建立起管道流。让我们看一下下面的代码，并预测它的结果。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> stream = numbers.stream().map(n -> n / 0).filter(n -> n % 2 == 0);
```

在上面所示的代码中，我们将一个数字流中的元素除以0。我们也许认为当代码执行时，它会抛出一个`ArithmeticException`的异常。但是，当你运行该代码时不会有异常抛出。这是因为Stream不会计算直到一个最终的命令调用它。如果在该管道流中添加最终的调用方法，那么该Stream将会执行，并抛出异常。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> stream = numbers.stream().map(n -> n / 0).filter(n -> n % 2 == 0);
stream.collect(toList());
```

你将会得到类似下面的堆栈信息。

```
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at org._7dayswithx.java8.day2.EagerEvaluationExample.lambda$main$0(EagerEvaluationExample.java:13)
	at org._7dayswithx.java8.day2.EagerEvaluationExample$$Lambda$1/1915318863.apply(Unknown Source)
	at java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:193)
	at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
	at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:512)
	at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:502)
	at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708)
	at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
	at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499)
```

## 使用Stream API

Stream API提供了很多开发者可以用来从集合中查询数据的操作。Stream操作分为两类——中间操作和结束操作。

**中间操作**是从已有的Stream产生另一个Stream的函数，有`filter`、`map`、`sorted`等。

**结束操作**是从Stream来产生一个不是Stream的结果的函数，有`collect(toList())`、`forEach`、`count`等。

中间操作允许你来构建管道流，它们会在你调用结束操作时被执行。下面是Stream API部分函数的列表。

<a href="https://whyjava.files.wordpress.com/2015/07/stream-api.png"><img class="aligncenter size-full wp-image-2983" src="https://whyjava.files.wordpress.com/2015/07/stream-api.png" alt="stream-api" height="450" /></a>

### 示例域

在这个教程中，我们将使用任务管理域来解释概念。我们的示例域有一个叫做`Task`的类——一个由用户来执行的任务。这个类如下所示。

```java
import java.time.LocalDate;
import java.util.*;

public class Task {
    private final String id;
    private final String title;
    private final TaskType type;
    private final LocalDate createdOn;
    private boolean done = false;
    private Set<String> tags = new HashSet<>();
    private LocalDate dueOn;

    // removed constructor, getter, and setter for brevity
}
```

该例子的数据集如下所示。我们将在Stream API示例中都使用这个列表。

```java
Task task1 = new Task("Read Version Control with Git book", TaskType.READING, LocalDate.of(2015, Month.JULY, 1)).addTag("git").addTag("reading").addTag("books");

Task task2 = new Task("Read Java 8 Lambdas book", TaskType.READING, LocalDate.of(2015, Month.JULY, 2)).addTag("java8").addTag("reading").addTag("books");

Task task3 = new Task("Write a mobile application to store my tasks", TaskType.CODING, LocalDate.of(2015, Month.JULY, 3)).addTag("coding").addTag("mobile");

Task task4 = new Task("Write a blog on Java 8 Streams", TaskType.WRITING, LocalDate.of(2015, Month.JULY, 4)).addTag("blogging").addTag("writing").addTag("streams");

Task task5 = new Task("Read Domain Driven Design book", TaskType.READING, LocalDate.of(2015, Month.JULY, 5)).addTag("ddd").addTag("books").addTag("reading");

List<Task> tasks = Arrays.asList(task1, task2, task3, task4, task5);
```

> 我们在这一节中不会讨论Java8的日期时间API。现在只要把它当做可读性高的日期相关的API即可。

### 例子1：找到所有的阅读任务并按照它们的创建日期排序

我们讨论的第一个例子是找到所有的阅读任务，并按照它们的创建日期进行排序。我们需要执行的操作如下：

1. 过滤所有的任务来找到任务类型为`READING`的任务。
2. 将过滤后的结果按照`createdOn`域进行排序。
3. 获得每个任务的标题。
4. 将结果的标题收集到一个列表中。

上面的四个操作可以被容易地转换为如下的代码。

```java
private static List<String> allReadingTasks(List<Task> tasks) {
        List<String> readingTaskTitles = tasks.stream().
                filter(task -> task.getType() == TaskType.READING).
                sorted((t1, t2) -> t1.getCreatedOn().compareTo(t2.getCreatedOn())).
                map(task -> task.getTitle()).
                collect(Collectors.toList());
        return readingTaskTitles;
}
```

在上面展示的代码中，我们使用了Stream API如下的方法：

* **filter**：允许你指定一个断言来从数据流中排除一些元素。断言**task->task.getType() ==TaskType.READING**选择了所有任务类型为`READING`的任务。
* **sorted**：允许你指定一个比较器来对数据流进行排序。在这个例子中，你根据创建的日期来进行排序。lambda表达式**(t1,t2)->t1.getCreatedOn().compareTo(t2.getCreatedOn())** 提供了`Comparator`函数式接口的`compare`方法的实现。
* **map**：这需要一个实现了`Function<? super T, ? extends R>`接口的lambda表达式来将一个数据流转化为另一个数据流。lambda表达式**task->task.getTitle()**将一个任务转化为一个标题。
* **collect(toList())**：这是一个结束操作，它收集结果中的阅读任务的标题并放入列表。

我们可以像下面一样通过使用`Comparator`接口的`comparing`方法和方法引用来优化上面的Java8代码。

```java
public List<String> allReadingTasks(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            sorted(Comparator.comparing(Task::getCreatedOn)).
            map(Task::getTitle).
            collect(Collectors.toList());

}
```

> 从Java8开始，接口可以以静态方法和默认方法的形式来拥有方法的实现。这些内容在[第一节](./01-default-static-interface-methods.md)中。

在上面展示的代码中，我们用了一个由`Comparator`接口提供的静态辅助方法`comparing`，它以`Function`接口为参数，而`Function`方法提取一个`Comparable`的键，并返回一个根据该键进行比较的比较器。方法引用`Task::getCreatedOn`相当于一个`Function<Task, LocalDate>`。

如下所示，通过使用复合函数，我们可以轻易地通过在比较器上调用`reversed`方法来编写将元素反向排序的代码。

```java
public List<String> allReadingTasksSortedByCreatedOnDesc(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            sorted(Comparator.comparing(Task::getCreatedOn).reversed()).
            map(Task::getTitle).
            collect(Collectors.toList());
}
```

### 例子2：找到唯一的任务

假设我们的数据集中有重复的任务。如下所示，我们可以在数据流上使用`distinct`方法来轻易地去除重复元素从而得到唯一的元素。

```java
public List<Task> allDistinctTasks(List<Task> tasks) {
    return tasks.stream().distinct().collect(Collectors.toList());
}
```

`distinct`方法将一个数据流转化成另一个没有重复元素的数据流。它使用对象的`equals`方法来决定对象是否相等。根据对象`equals`方法的约定，当两个对象相等的时候，它们被认为是重复的，然后其中一个会从结果数据流中被移除。

### 例子3：找到根据创建日期排序的前5名的阅读任务

`limit`方法可以用来将结果集限定为特定的大小。`limit`方法是一个逻辑短路操作，也就是说它不会遍历所有的元素来得到结果。

```java
public List<String> topN(List<Task> tasks, int n){
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            sorted(comparing(Task::getCreatedOn)).
            map(Task::getTitle).
            limit(n).
            collect(toList());
}
```

如下所示，你可以将`limit`方法和`skip`方法一起使用来创建分页。

```java
// page starts from 0. So to view a second page `page` will be 1 and n will be 5.
List<String> readingTaskTitles = tasks.stream().
                filter(task -> task.getType() == TaskType.READING).
                sorted(comparing(Task::getCreatedOn).reversed()).
                map(Task::getTitle).
                skip(page * n).
                limit(n).
                collect(toList());
```

### 例子4：计算所有阅读任务的数量

为了得到所有阅读任务的数量，我们可以在数据流上使用`count`方法。这个方法是一个结束操作。

```java
public long countAllReadingTasks(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            count();
}
```

### 例子5：从所有的任务中找出所有不同的标签

为了找出所有不同的标签，我们需要进行如下的操作：

1. 为每一个任务提取标签。
2. 将所有的标签收集进一个数据流。
3. 将重复的标签除去。
4. 最后将收集的结果放入一个列表。

第一个和第二个操作可以通过在`tasks`数据流上使用`flatMap`操作来完成。`flatMap`操作将每次调用`tasks.getTags().stream()`产生的数据流合并到一个中。一旦我们将所有标签放入一个数据流中，我们可以仅仅通过`distinct`方法来获取所有不同的标签。

```java
private static List<String> allDistinctTags(List<Task> tasks) {
        return tasks.stream().flatMap(task -> task.getTags().stream()).distinct().collect(toList());
}
```

### 例子6：检查是否所有的阅读任务都有`books`标签

Stream API提供了方法让用户来检查数据集中元素的某一属性是否符合要求。这些方法是`allMatch`、`anyMatch`、`findFirst`和`findAny`。为了检查是否所有的阅读任务都一个名叫`books`的标签，我们可以编写如下的代码。

```java
public boolean isAllReadingTasksWithTagBooks(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            allMatch(task -> task.getTags().contains("books"));
}
```

为了检查是否有阅读任务有`java8`标签，我们可以像下面这样使用`anyMatch`操作。

```java
public boolean isAnyReadingTasksWithTagJava8(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            anyMatch(task -> task.getTags().contains("java8"));
}
```

### 例子7：创建一个所有标题的总结

假设你想创建一个所有标题的总结。使用`reduce`操作，它将数据流缩减为一个值。`reduce`方法以将数据流中元素进行连接的lambda表达式为参数。

```java
public String joinAllTaskTitles(List<Task> tasks) {
    return tasks.stream().
            map(Task::getTitle).
            reduce((first, second) -> first + " *** " + second).
            get();
}
```

### 例子8：与原始流一同工作

除了作用在对象上的通用的数据流，Java8还提供了特殊的数据流来处理原始类型，像int、long和double。让我看一些原始数据流的例子。

为了创建一个范围的值，我们可以使用`range`方法，它能创建一个从0开始到9的数据流，它不包括10。

```java
IntStream.range(0, 10).forEach(System.out::println);
```

`rangeClosed`方法允许你创建包含右边界的数据流。所以下面的数据流将从1开始，到10结束。

```java
IntStream.rangeClosed(1, 10).forEach(System.out::println);
```

你也可以像下面这样通过`iterate`方法在原始数据流上创建一个无限的数据流。

```java
LongStream infiniteStream = LongStream.iterate(1, el -> el + 1);
```

为过滤掉无限数据流中所有的偶数，我们可以编写如下的代码。

```java
infiniteStream.filter(el -> el % 2 == 0).forEach(System.out::println);
```

我们可以像下面这样通过使用`limit`操作来限制结果数据流的数量。

```java
infiniteStream.filter(el -> el % 2 == 0).limit(100).forEach(System.out::println);
```

### 例子9：从Arrays来创建数据流

如下所示，你可以通过使用`Arrays`类的静态方法`stream`来从数组创建数据流。

```java
String[] tags = {"java", "git", "lambdas", "machine-learning"};
Arrays.stream(tags).map(String::toUpperCase).forEach(System.out::println);
```

你也可像下面这样从一个数组特定的起始下标到结束下标来创建一个数据流。在这里，起始下标被包含在内，而结束下标没有。

```java
Arrays.stream(tags, 1, 3).map(String::toUpperCase).forEach(System.out::println);
```

## 并行数据流

你使用`Stream`抽象的一大优势就是这个库能够有效地管理并行，就像迭代器也在容器内部一样。你可以通过调用数据流的`parallel`方法来使一个数据流并行。`parallel`方法底层使用了jdk7的`fork-join` API。默认地，它会将线程数上升到与主机CPU数量相等的数目。在下面所示的代码中，我们将数字按照处理它们的线程来进行分组。你将会在第四章中学习到`collect`和`groupingBy`方法。现在只需要理解它们允许你根据一个键来将元素分组。

```java
public class ParallelStreamExample {

    public static void main(String[] args) {
        Map<String, List<Integer>> numbersPerThread = IntStream.rangeClosed(1, 160)
                .parallel()
                .boxed()
                .collect(groupingBy(i -> Thread.currentThread().getName()));

        numbersPerThread.forEach((k, v) -> System.out.println(String.format("%s >> %s", k, v)));
    }
}
```

该程序在我电脑上的输出如下所示。

```
ForkJoinPool.commonPool-worker-7 >> [46, 47, 48, 49, 50]
ForkJoinPool.commonPool-worker-1 >> [41, 42, 43, 44, 45, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130]
ForkJoinPool.commonPool-worker-2 >> [146, 147, 148, 149, 150]
main >> [106, 107, 108, 109, 110]
ForkJoinPool.commonPool-worker-5 >> [71, 72, 73, 74, 75]
ForkJoinPool.commonPool-worker-6 >> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160]
ForkJoinPool.commonPool-worker-3 >> [21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 76, 77, 78, 79, 80]
ForkJoinPool.commonPool-worker-4 >> [91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145]
```

不是每一个线程处理一样多的元素。你可以通过设置系统属性`System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism",
"2")`来控制`fork-join`线程池的大小。

如下所示，另一个你可以使用到`parallel`操作的地方是你操作一个URL的列表。

```java
String[] urls = {"https://www.google.co.in/", "https://twitter.com/", "http://www.facebook.com/"};
Arrays.stream(urls).parallel().map(url -> getUrlContent(url)).forEach(System.out::println);
```

如果你要理解何时来使用并行数据流，我建议你阅读Doug Lea et al的这篇文章 [http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html](http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html)。
