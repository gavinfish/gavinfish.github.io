---
layout: post
title:  "Java8新特性之collectors"
date:   2016-09-14
categories: Java
tags: Java API

---

在[第二天](./03-streams.md)，你已经学习了Stream API能够让你以声明式的方式帮助你处理集合。我们看到`collect`是一个将管道流的结果集到一个`list`中的结束操作。`collect`是一个将数据流缩减为一个值的归约操作。这个值可以是集合、映射，或者一个值对象。你可以使用`collect`达到以下目的：

1. **将数据流缩减为一个单一值：**一个流执行后的结果能够被缩减为一个单一的值。单一的值可以是一个`Collection`，或者像int、double等的数值，再或者是一个用户自定义的值对象。

2. **将一个数据流中的元素进行分组：**根据任务类型将流中所有的任务进行分组。这将产生一个`Map<TaskType, List<Task>>`的结果，其中每个实体包含一个任务类型以及与它相关的任务。你也可以使用除了列表以外的任何其他的集合。如果你不需要与一任务类型相关的所有的任务，你可以选择产生一个`Map<TaskType, Task>`。这是一个能够根据任务类型对任务进行分类并获取每类任务中第一个任务的例子。

3. **分割一个流中的元素：**你可以将一个流分割为两组——比如将任务分割为要做和已经做完的任务。

## Collector实际应用

为了感受到`Collector`的威力，让我们来看一下我们要根据任务类型来对任务进行分类的例子。在Java8中，我们可以通过编写如下的代码达到将任务根据类型分组的目的。**请参考[第二天](./03-streams.md)的博文，也就是我们讨论的在这一系列文章中我们将使用的任务域。**

```java
private static Map<TaskType, List<Task>> groupTasksByType(List<Task> tasks) {
    return tasks.stream().collect(Collectors.groupingBy(task -> task.getType()));
}
```

上面的代码使用了定义在辅助类`Collectors`中的`groupingBy`收集器。它创建了一个映射，其中`TaskType`是它的键，而包含了所有拥有相同`TaskType`的任务的列表是它的值。为了在Java7中达到相同的效果，你需要编写如下的代码。

```java
public static void main(String[] args) {
    List<Task> tasks = getTasks();
    Map<TaskType, List<Task>> allTasksByType = new HashMap<>();
    for (Task task : tasks) {
        List<Task> existingTasksByType = allTasksByType.get(task.getType());
        if (existingTasksByType == null) {
            List<Task> tasksByType = new ArrayList<>();
            tasksByType.add(task);
            allTasksByType.put(task.getType(), tasksByType);
        } else {
            existingTasksByType.add(task);
        }
    }
    for (Map.Entry<TaskType, List<Task>> entry : allTasksByType.entrySet()) {
        System.out.println(String.format("%s =>> %s", entry.getKey(), entry.getValue()));
    }
}
```

## 收集器：常用的规约操作

`Collectors`辅助类提供了大量的静态辅助方法来创建收集器为常见的使用场景服务，像将元素收集到一个集合中、分组和分割元素，或者根据不同的标准来概述元素。我们将在这篇博文中涵盖大部分常见的`Collector`。

## 缩减为一个值

正如上面讨论的，收集器可以被用来收集流的输出到一个集合，或者产生一个单一的值。

### 将数据收集进一个列表

让我们编写我们的第一个测试用例——给定一个任务列表，我们想将他们的标题收集进一个列表。

```java
import static java.util.stream.Collectors.toList;

public class Example2_ReduceValue {
    public List<String> allTitles(List<Task> tasks) {
        return tasks.stream().map(Task::getTitle).collect(toList());
    }
}
```

`toList`收集器使用了列表的`add`方法来向结果列表中添加元素。`toList`收集器使用了`ArrayList`作为列表的实现。

### 将数据收集进一个集合

如果我们想要确保返回的标题都是唯一的，并且我们不在乎元素的顺序，那么我们可以使用`toSet`收集器。

```java
import static java.util.stream.Collectors.toSet;

public Set<String> uniqueTitles(List<Task> tasks) {
    return tasks.stream().map(Task::getTitle).collect(toSet());
}
```

`toSet`方法使用了`HashSet`作为集合的实现来存储结果集。

### 将数据收集进一个映射

你可以使用`toMap`收集器将一个流转换为一个映射。`toMap`收集器需要两个映射方法来获得映射的键和值。在下面展示的代码中，`Task::getTitle`是接收一个任务并产生一个只包含该任务标题的键的`Function`。**task -> task**是一个用来返回任务本身的lambda表达式。

```java
private static Map<String, Task> taskMap(List<Task> tasks) {
  return tasks.stream().collect(toMap(Task::getTitle, task -> task));
}
```

我们可以通过使用`Function`接口中的默认方法`identity`来改进上面展示的代码，如下所示，这样可以让代码更加简洁，并更好地传达开发者的意图。

```java
import static java.util.function.Function.identity;

private static Map<String, Task> taskMap(List<Task> tasks) {
  return tasks.stream().collect(toMap(Task::getTitle, identity()));
}
```

从一个流中创建映射的代码会在存在重复的键时抛出异常。你将会得到一个类似下面的错误。

```
Exception in thread "main" java.lang.IllegalStateException: Duplicate key Task{title='Read Version Control with Git book', type=READING}
at java.util.stream.Collectors.lambda$throwingMerger$105(Collectors.java:133)
```

你可以通过使用`toMap`方法的另一个变体来处理重复问题，它允许我们指定一个合并方法。这个合并方法允许用户他们指定想如何处理多个值关联到同一个键的冲突。在下面展示的代码中，我们只是使用了新的值，当然你也可以编写一个智能的算法来处理冲突。

```java
private static Map<String, Task> taskMap_duplicates(List<Task> tasks) {
  return tasks.stream().collect(toMap(Task::getTitle, identity(), (t1, t2) -> t2));
}
```

你可以通过使用`toMap`方法的第三个变体来指定其他的映射实现。这需要你指定将用来存储结果的`Map`和`Supplier`。

```
public Map<String, Task> collectToMap(List<Task> tasks) {
    return tasks.stream().collect(toMap(Task::getTitle, identity(), (t1, t2) -> t2, LinkedHashMap::new));
}
```

类似于`toMap`收集器，也有`toConcurrentMap`收集器，它产生一个`ConcurrentMap`而不是`HashMap`。

### 使用其它的收集器

像`toList`和`toSet`这类特定的收集器不允许你指定内部的列表或者集合实现。当你想要将结果收集到其它类型的集合中时，你可以像下面这样使用`toCollection`收集器。

```
private static LinkedHashSet<Task> collectToLinkedHaskSet(List<Task> tasks) {
  return tasks.stream().collect(toCollection(LinkedHashSet::new));
}
```

### 找到拥有最长标题的任务

```java
public Task taskWithLongestTitle(List<Task> tasks) {
    return tasks.stream().collect(collectingAndThen(maxBy((t1, t2) -> t1.getTitle().length() - t2.getTitle().length()), Optional::get));
}
```

### 统计标签的总数

```java
public int totalTagCount(List<Task> tasks) {
    return tasks.stream().collect(summingInt(task -> task.getTags().size()));
}
```

### 生成任务标题的概述

```java
public String titleSummary(List<Task> tasks) {
    return tasks.stream().map(Task::getTitle).collect(joining(";"));
}
```

## 分类收集器

收集器最常见的使用场景之一是对元素进行分类。让我来看一下不同的例子来理解我们如何进行分类。

### 例子1：根据类型对任务分类

我们看一下下面展示的例子，我们想要根据`TaskType`来对所有的任务进行分类。我们可以通过使用`Collectors`辅助类中的`groupingBy`方法来轻易地进行该项任务。你可以通过使用方法引用和静态导入来使它更加高效。

```java
import static java.util.stream.Collectors.groupingBy;
private static Map<TaskType, List<Task>> groupTasksByType(List<Task> tasks) {
       return tasks.stream().collect(groupingBy(Task::getType));
}
```

它将会产生如下的输出。

```
{CODING=[Task{title='Write a mobile application to store my tasks', type=CODING, createdOn=2015-07-03}], WRITING=[Task{title='Write a blog on Java 8 Streams', type=WRITING, createdOn=2015-07-04}], READING=[Task{title='Read Version Control with Git book', type=READING, createdOn=2015-07-01}, Task{title='Read Java 8 Lambdas book', type=READING, createdOn=2015-07-02}, Task{title='Read Domain Driven Design book', type=READING, createdOn=2015-07-05}]}
```

### 例子2：根据标签分类

```java
private static Map<String, List<Task>> groupingByTag(List<Task> tasks) {
        return tasks.stream().
                flatMap(task -> task.getTags().stream().map(tag -> new TaskTag(tag, task))).
                collect(groupingBy(TaskTag::getTag, mapping(TaskTag::getTask,toList())));
}

    private static class TaskTag {
        final String tag;
        final Task task;

        public TaskTag(String tag, Task task) {
            this.tag = tag;
            this.task = task;
        }

        public String getTag() {
            return tag;
        }

        public Task getTask() {
            return task;
        }
    }
```

### 例子3：根据标签和数量对任务分类

将分类器和收集器结合起来。

```java
private static Map<String, Long> tagsAndCount(List<Task> tasks) {
        return tasks.stream().
        flatMap(task -> task.getTags().stream().map(tag -> new TaskTag(tag, task))).
        collect(groupingBy(TaskTag::getTag, counting()));
    }
```

### 例子4：根据任务类型和创建日期分类

```java
private static Map<TaskType, Map<LocalDate, List<Task>>> groupTasksByTypeAndCreationDate(List<Task> tasks) {
        return tasks.stream().collect(groupingBy(Task::getType, groupingBy(Task::getCreatedOn)));
    }
```

## 分割

很多时候你想根据一个断言来将一个数据集分割成两个数据集。举例来说，我们可以通过定义一个将任务分割为两组的分割方法来将任务分割成两组，一组是在今天之前已经到期的，另一组是其他的任务。

```java
private static Map<Boolean, List<Task>> partitionOldAndFutureTasks(List<Task> tasks) {
  return tasks.stream().collect(partitioningBy(task -> task.getDueOn().isAfter(LocalDate.now())));
}
```

## 生成统计信息

另一组非常有用的收集器是用来产生统计信息的收集器。这能够在像`int`、`double`和`long`这样的原始数据类型上起到作用；并且能被用来生成像下面这样的统计信息。

```java
IntSummaryStatistics summaryStatistics = tasks.stream().map(Task::getTitle).collect(summarizingInt(String::length));
System.out.println(summaryStatistics.getAverage()); //32.4
System.out.println(summaryStatistics.getCount()); //5
System.out.println(summaryStatistics.getMax()); //44
System.out.println(summaryStatistics.getMin()); //24
System.out.println(summaryStatistics.getSum()); //162
```

也有其它的变种形式，像针对其它原生类型的`LongSummaryStatistics`和`DoubleSummaryStatistics`。

你也可以通过使用`combine`操作来将一个`IntSummaryStatistics`与另一个组合起来。

```java
firstSummaryStatistics.combine(secondSummaryStatistics);
System.out.println(firstSummaryStatistics)
```

## 连接所有的标题

```java
private static String allTitles(List<Task> tasks) {
  return tasks.stream().map(Task::getTitle).collect(joining(", "));
}
```

## 编写一个定制的收集器

```java
import com.google.common.collect.HashMultiset;
import com.google.common.collect.Multiset;

import java.util.Collections;
import java.util.EnumSet;
import java.util.Set;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

public class MultisetCollector<T> implements Collector<T, Multiset<T>, Multiset<T>> {

    @Override
    public Supplier<Multiset<T>> supplier() {
        return HashMultiset::create;
    }

    @Override
    public BiConsumer<Multiset<T>, T> accumulator() {
        return (set, e) -> set.add(e, 1);
    }

    @Override
    public BinaryOperator<Multiset<T>> combiner() {
        return (set1, set2) -> {
            set1.addAll(set2);
            return set1;
        };
    }

    @Override
    public Function<Multiset<T>, Multiset<T>> finisher() {
        return Function.identity();
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH));
    }
}
```

```java
import com.google.common.collect.Multiset;

import java.util.Arrays;
import java.util.List;

public class MultisetCollectorExample {

    public static void main(String[] args) {
        List<String> names = Arrays.asList("shekhar", "rahul", "shekhar");
        Multiset<String> set = names.stream().collect(new MultisetCollector<>());

        set.forEach(str -> System.out.println(str + ":" + set.count(str)));

    }
}
```

## Java8中的字数统计

我们将通过使用流和收集器在Java8中编写有名的字数统计样例来结束这一节。

```java
public static void wordCount(Path path) throws IOException {
    Map<String, Long> wordCount = Files.lines(path)
            .parallel()
            .flatMap(line -> Arrays.stream(line.trim().split("\\s")))
            .map(word -> word.replaceAll("[^a-zA-Z]", "").toLowerCase().trim())
            .filter(word -> word.length() > 0)
            .map(word -> new SimpleEntry<>(word, 1))
            .collect(groupingBy(SimpleEntry::getKey, counting()));
    wordCount.forEach((k, v) -> System.out.println(String.format("%s ==>> %d", k, v)));
}
```
