---
layout: post
title:  "Java8新特性之lambdas"
date:   2016-09-07
categories: Java
tags: Java API lambda

---

# Java8新特性之lambda

---

Java8中最重要的特性之一就是引入了lambda表达式。这能够使你的代码更加简练，并允许你将行为传递到各处。一段时间以来，Java因为自身的冗长和缺少函数式编程的能力而受到批评。随着函数式编程变得越来越流行和有价值，Java也在努力接受函数式编程。否则，Java将会变得没有价值。

Java8在使世界上最受欢迎的编程语言之一在接纳函数式编程的过程中向前迈了一大步。为了支持函数式编程，编程语言必须将函数作为[第一类对象](https://zh.wikipedia.org/wiki/%E7%AC%AC%E4%B8%80%E9%A1%9E%E7%89%A9%E4%BB%B6)。在Java8之前，如果没有使用一个匿名内部类模板是没法写出清晰的函数式代码的。随着lambda表达式的引入，函数已经成为第一类对象，并能够像其它变量一样被到处传递。

lambda表达式允许你定义一个不与标识符绑定的匿名函数。你可以像编程语言中的其它概念一样使用它们，比如变量的声明。当一个编程语言需要支持高阶函数时，就需要用到lambda表达式。高阶函数是指以其它函数作为参数或者返回函数作为结果的函数。

> 这一节的代码在[ch02](https://github.com/shekhargulati/java8-the-missing-tutorial/tree/master/code/src/main/java/com/shekhargulati/java8_tutorial/ch02)包中

现在，随着在Java8中引进了lambda表达式，Java已经支持高阶函数。让我来看一个lambda表达式的典型例子——`Collections`类中的`sort`方法。`sort`方法有两种变体——一种以一个`List`作为参数，另一个以`List`和`Comparator`作为参数。如下面的代码块所示，第二种`sort`方法是一个接受lambda表达式的高阶函数的例子。

```java
List<String> names = Arrays.asList("shekhar", "rahul", "sameer");
Collections.sort(names, (first, second) -> first.length() - second.length());
```

上面的代码将姓名链表按照元素的长度进行排序。该程序的输出如下所示。

```
[rahul, sameer, shekhar]
```

上面代码块中的表达式`(first, second) -> first.length() - second.length()`是一个`Comparator<String>`类型的lambda表达式。

* `(first, second)`是比较器`Comparator`的`compare`方法。
* `first.length() - second.length()` 是用来比较两个名字长度的方法实体。
* `->`是lambda操作符，用来将参数和方法体分离开。

在我们继续深挖Java8的lambda表达式之前，让我们来看看lambda的历史来理解为什么会存在lambda。

## lambda的历史

lambda表达式源自λ演算。[λ演算](https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97)由[Alonzo Church](https://zh.wikipedia.org/wiki/%E9%98%BF%E9%9A%86%E4%BD%90%C2%B7%E9%82%B1%E5%A5%87)在将带有函数的符号计算进行公式化时提出。λ演算是具有图灵完备性的，它通过数学形式来展现计算过程。图灵完备性表示你可以通过lambda表达任何的数学计算。

λ演算成为了函数式编程语言的一个坚实的理论基础。很多有名的函数式编程语言，像Haskell和Lisp都是构建在λ演算的基础上的。高阶函数的概念，比如接受其他函数为输入的函数也来自λ演算。

λ演算的核心概念是表达式。一个lambda表达式可以表示为如下形式：

```
<expression> := <variable> | <function>| <application>
```

* **variable**--变量就是类似x,y,z的占位符，它们用来表示具体的像1,2之类的值，或者lambda方法。
* **functrion**--这是一个匿名的方法定义，它需要一个变量，并产生另一个lambda表达式。例如，`λx.x*x`是一个用来计算数的平方的方法。
* **application**--这是将具体的参数应用在函数上的行为。假设你想得到10的平方，那么在λ演算中你会写一个平方函数`λx.x*x`，并把10代入。这个函数应用将得到`(λx.x*x) 10 = 10*10 = 100`。你不仅仅能够代入简单的像10一样的值，你可以将一个函数代入另一个函数来得到一个新的函数。例如，`(λx.x*x) (λz.z+10)`将会生成一个函数`λz.(z+10)*(z+10)`。现在，你可以用这个函数得到一个数加上10以后的平方。这是一个高阶函数的例子。

现在你理解了λ演算和它在函数式编程语言中的影响。让我们来学习它是如何在Java8中实现的。

## 在Java8之前传递行为的方式

在Java8之前，唯一能够用来传递行为的方式是通过匿名类。假设你想要在用户完成注册的同时在另一个线程中给该用户发送一封邮件。在Java8之前，你会写出类似下面的代码。

```java
sendEmail(new Runnable() {
            @Override
            public void run() {
                System.out.println("Sending email...");
            }
        });
```

`sendEmail`方法拥有如下的方法签名。

```java
public static void sendEmail(Runnable runnable)
```

上面提到的代码的问题不仅仅是我们需要封装我们的行为，如将`run`方法直接放在一个对象中，更严重的问题是它丢失了程序员的意图，如将行为传递到`sendEmail`方法中。如果你使用过Guava类库，你肯定感受到了编写匿名类的痛苦。一个简单的用来过滤所有任务的标题中有**lambda**的例子如下所示。

```java
Iterable<Task> lambdaTasks = Iterables.filter(tasks, new Predicate<Task>() {
            @Override
            public boolean apply(Task task) {
                return input.getTitle().contains("lambda");
            }
});
```

有了Java8的Stream API，你可以在不使用像Guava一样的第三方库的情况下写出上面提及的代码。我们将在[第三章]()中讲解Stream，敬请期待。

## Java8 lambda表达式

在Java8中，我们将使用lambda表达式写出如下的代码。这与我们上面提及过的代码段相同。

```java
sendEmail(() -> System.out.println("Sending email..."));
```

上面的展示的代码非常简练，也没有污染程序员想要传递的行为。`()`用来表示这个lambda表达式没有参数，像`Runnable`接口中的`run`方法就没有任何参数。`->`是将参数和用来打印出`Sending email`的方法主体分隔开的lambda操作符。

让我再来看看`Collections.sort`这个例子来了解lambda表达式是如何使用参数的。为了使名字能够按照它们的长度进行排列，我们向排序方法传入了一个`Comparator`。该`Comparator`如下所示。

```java
Comparator<String> comparator = (first, second) -> first.length() - second.length();
```

我们编写的lambda表达式与`Comparator`接口中的`compare`方法相关联。`compare`方法的签名如下。

```java
int compare(T o1, T o2);
```

`T`是传给`Comparator`接口的类型参数。由于我们是对一组表示名字的字符串进行操作，所以这个例子中它将是字符串类型的。在lambda表达式中我们不需要特意提供该类型——字符串。`javac`编译器会从上下文中推断出它的类型信息。由于我们在给一组字符串排序，Java编译器会推测出两个参数都应该是字符串，而`compare`方法只标明需要`T`这一种类型。像这样通过上下文推断类型的行为称作类型推断。Java8优化了Java原有的类型推断机制，使得它更具有鲁棒性，并能够更好地支持lambda表达式。`javac`会在后台寻找与你lambda表达式相关的信息，并使用该信息来找到参数正确的类型。

>在大多数情况下，`javac`会从上下文中推断出类型。如果由于上下文缺失或不完整导致代码不能进行编译，它也就不能推断出类型。例如如果我们将`String`的类型信息从`Comparator`中移除，那么代码会像下面一样编译失败。

```java
Comparator comparator = (first, second) -> first.length() - second.length(); // compilation error - Cannot resolve method 'length()'
```

## lambda表达式是如何在Java8中工作的？

你也许已经发现lambda表达式是与上面例子中的`Comparator`类似的一些接口。你不能对任意的接口使用lambda表达式。***只有那些除了Object的方法外只定义了唯一抽象方法的接口可以使用lambda表达式。***这一类的接口被称作**函数式接口**，它们可以通过`@FunctionalInterface`注解来进行注解。如下所示，`Runnable`接口就是一个函数式接口。

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

`@FunctionalInterface`注解不是强制需要的，它能够帮助其他工具知道这个接口是一个函数式接口，以此展现出有意义的行为。如果你试图编译一个有`@FunctionalInterface`的接口，而该接口有多个抽象方法，那么编译器将会抛出一个***发现多个没有重写的抽象方法***的异常。同样的，如果你对一个没有任何方法的接口添加`@FunctionalInterface`的注解，比如一个标记接口，那么你将会得到一条***没有找到目标方法的***的消息。

让我们来解答一个你也许会想到的最重要的问题。***Java8中的lambda表达式是仅仅针对匿名类的语法糖吗，或者说函数式接口是如何转换为字节码的？***简单的答案是**不是**。Java8不使用匿名内部类主要有两个原因：

1. **性能开销**：如果lambda表达式是通过使用匿名类来实现的，那么每一个lambda表达式都要在磁盘上产生一个文件。如果这些类在JVM启动时被加载，那么JVM的启动时间将会增加，因为所有的类在使用前都要进行加载和验证。

2. **未来改变的可能性**：如果Java8的设计者从开始就使用了匿名类，那么这将限制lambda表达式的实现方式在将来的变化。

### 使用invokedynamic

Java8设计者决定使用在Java7中添加的`invokedynamic`指令来在运行时推迟编译策略的执行。当`javac`编译代码的时候，它会捕捉到lambda表达式并生成一个`invokedynamic`的调用（被叫做lambda工厂）。当`invokedynamic`命令被调用时，它会返回一个lambda要转化的函数式接口的实例。例如，我来查看`Collections.sort`的字节码，它如下所示。

```
public static void main(java.lang.String[]);
    Code:
       0: iconst_3
       1: anewarray     #2                  // class java/lang/String
       4: dup
       5: iconst_0
       6: ldc           #3                  // String shekhar
       8: aastore
       9: dup
      10: iconst_1
      11: ldc           #4                  // String rahul
      13: aastore
      14: dup
      15: iconst_2
      16: ldc           #5                  // String sameer
      18: aastore
      19: invokestatic  #6                  // Method java/util/Arrays.asList:([Ljava/lang/Object;)Ljava/util/List;
      22: astore_1
      23: invokedynamic #7,  0              // InvokeDynamic #0:compare:()Ljava/util/Comparator;
      28: astore_2
      29: aload_1
      30: aload_2
      31: invokestatic  #8                  // Method java/util/Collections.sort:(Ljava/util/List;Ljava/util/Comparator;)V
      34: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
      37: aload_1
      38: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
      41: return
}
```

该字节码有意思的地方在第23行`23: invokedynamic #7,  0              // InvokeDynamic #0:compare:()Ljava/util/Comparator;`，也就是生成一个`invokedynamic`的地方。

第二步是将lambda表达式的主体部分转化成通过`invokedynamic`指令调用的方法。这一步让JVM实现者能够自由地选取他们自己的策略。我省略了这个话题相关的内容，你可以在http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html中阅读到更多的内容。

## 匿名类 vs lambda

让我们通过比较匿名类和lambda表达式来比较它们的不同。

1. 在匿名类中，`this`表示匿名类自己，而在lambda表达式中，`this`表示包含了lambda表达式的类。
2. 你可以在匿名类这个封闭类中隐藏变量。在lambda表达式中这么做时将产生一个编译错误。
3. lambda表达式的类型是由上下文决定的，而匿名类的类型是由你创建匿名类时指定的。

## 我需要自己编写函数式接口吗？

Java8默认提供了好多函数式编程接口来供你在代码中使用。它们在`java.util.function`包中。让我们看一下其中的一部分。

### java.util.function.Predicate<T>

这个函数式接口被用来定义某些情形的检查，类似于断言。`Predicate`接口有一个叫做`test`的方法，它以泛型`T`为参数，返回一个布尔值。举例来说，如果我们想从一串名字中找到所有以**s**开头的名字，那么我们将向下面这样使用`Predicate`。

```java
Predicate<String> namesStartingWithS = name -> name.startsWith("s");
```

### java.util.function.Consumer<T>

这个函数式接口被用来执行一些不用产生输出的动作。`Comsumer`接口有一个以泛型`T`为参数且没有返回值的`accept`方法。比如将一条给定的信息通过邮件发出。

```java
Consumer<String> messageConsumer = message -> System.out.println(message);
```

### java.util.function.Function<T,R>

这个函数式接口接受一个参数并产生一个结果。例如，如果我们想要将姓名列表中的所有名字都大写，我们可以写一个像下面这样的方法。

```java
Function<String, String> toUpperCase = name -> name.toUpperCase();
```

### java.util.function.Supplier<T>

这个函数式接口不需要任何参数，却会产生一个值。这可以被用来像下面这样生成唯一标志码。

```java
Supplier<String> uuidGenerator= () -> UUID.randomUUID().toString();
```

我们将在这一系列教程中涉及更多的函数式接口。

## 方法引用

有时候你会创建一些只调用特定方法的lambda表达式，比如`Function<String, Integer> strToLength = str ->
str.length();`。这个lambda只在`String`对象上调用`length()`方法。这种情况可以通过使用方法引用来简化成`Function<String, Integer>
strToLength = String::length;`。这可以被看做是只调用单个方法的lambda表达式的简化标记。在该表达式`String::length`中，`String`是目标引用，`::`是分隔符，`length`是在目标引用中将会被调用的方法。你在静态方法和实例方法中都可以使用方法引用。

### 静态方法引用

假设我们要找到一串数中最大的一个，那么我们可以写一个像`Function<List<Integer>, Integer> maxFn =
Collections::max`这样的方法引用。`max`是`Collections`类中一个以`list`为参数的静态方法。然后你可以像`maxFn.apply(Arrays.asList(1, 10, 3, 5))`这样来调用。上面的lambda表达式是与`Function<List<Integer>, Integer> maxFn = (numbers) ->
Collections.max(numbers);`等价的。

### 实例方法引用

这是一类为实例方法使用的方法引用，比如在`String::toUpperCase`在`String`引用上调用了`toUpperCase`方法。你也可以对有参数的方法使用方法引用，像`BiFunction<String,
String, String> concatFn = String::concat`。`concatFn`可以像`concatFn.apply("shekhar", "gulati")`这样被调用。`concat`方法是字符串对象的需要一个参数的方法，形式为`"shekhar".concat("gulati")`。

## 练习>>写自己的lambda

让我们看一下下面的代码，并把我们学的应用起来。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks);
        for (String title : titles) {
            System.out.println(title);
        }
    }

    public static List<String> taskTitles(List<Task> tasks) {
        List<String> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (task.getType() == TaskType.READING) {
                readingTitles.add(task.getTitle());
            }
        }
        return readingTitles;
    }

}
```

上面的代码首先从一个工具方法`getTasks`中获取所有的任务。我们对`getTasks`方法的内部实现不感兴趣。`getTasks`方法可以从web、数据库或者内存中来获取任务。一旦你有了任务，我们过滤出所有的阅读任务并抽取出这些任务的标题。我们将抽取的标题存入一个链表并最终返回所有的阅读标题。

让我们从最简单的重构开始——通过方法引用在链表上使用`foreach`方法。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks);
        titles.forEach(System.out::println);
    }

    public static List<String> taskTitles(List<Task> tasks) {
        List<String> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (task.getType() == TaskType.READING) {
                readingTitles.add(task.getTitle());
            }
        }
        return readingTitles;
    }

}
```

用`Predicate<T>`来过滤我们的任务。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks, task -> task.getType() == TaskType.READING);
        titles.forEach(System.out::println);
    }

    public static List<String> taskTitles(List<Task> tasks, Predicate<Task> filterTasks) {
        List<String> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (filterTasks.test(task)) {
                readingTitles.add(task.getTitle());
            }
        }
        return readingTitles;
    }

}
```

用`Function<T,R>`来从我们的任务中抽取标题。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks, task -> task.getType() == TaskType.READING, task -> task.getTitle());
        titles.forEach(System.out::println);
    }

    public static <R> List<R> taskTitles(List<Task> tasks, Predicate<Task> filterTasks, Function<Task, R> extractor) {
        List<R> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (filterTasks.test(task)) {
                readingTitles.add(extractor.apply(task));
            }
        }
        return readingTitles;
    }
}
```

对提取器使用方法引用。

```java
public static void main(String[] args) {
    List<Task> tasks = getTasks();
    List<String> titles = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, Task::getTitle);
    titles.forEach(System.out::println);
    List<LocalDate> createdOnDates = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, Task::getCreatedOn);
    createdOnDates.forEach(System.out::println);
    List<Task> filteredTasks = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, Function.identity());
    filteredTasks.forEach(System.out::println);
}
```

我们也可以通过编写我们自己的**函数式接口**，这样可以清楚地描述开发者的意图。我们可以创建一个继承于`Function`接口的`TaskExtractor`接口。该接口的输入类型被限定为`Task`，输出类型由lambda的实现决定。这样由于输入类型始终是`Task`，开发者只需要关注返回值的类型，

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<Task> filteredTasks = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, TaskExtractor.identityOp());
        filteredTasks.forEach(System.out::println);
    }

    public static <R> List<R> filterAndExtract(List<Task> tasks, Predicate<Task> filterTasks, TaskExtractor<R> extractor) {
        List<R> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (filterTasks.test(task)) {
                readingTitles.add(extractor.apply(task));
            }
        }
        return readingTitles;
    }

}


interface TaskExtractor<R> extends Function<Task, R> {

    static TaskExtractor<Task> identityOp() {
        return t -> t;
    }
}
```
