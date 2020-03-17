# CompletableFuture 实现异步计算

参考：<https://juejin.im/post/5d35d6ebe51d45109b01b270?utm_source=gold_browser_extension#heading-8>

------

[TOC]

## 概述

Java 1.8 新增加的 CompletableFuture 类内部是使用 ForkJoinPool 来实现的，CompletableFuture 实现了 Future接口 和 CompletionStage接口。

在之前的[Future的介绍和基本用法](https://juejin.im/post/5c99f03df265da6109161409)一文中，我们了解到 Future 表示异步计算的结果。

CompletionStage 代表了一个特定的计算的阶段，可以同步或者异步的被完成。

CompletionStage 可以被看作一个计算流水线上的一个单元，最终会产生一个最终结果，这意味着几个 CompletionStage 可以串联起来，一个完成的阶段可以触发下一阶段的执行，接着触发下一次，直到完成所有阶段。

## Future的弊端

Future是Java 5添加的类，用来描述一个异步计算的结果。你可以使用isDone方法检查计算是否完成，或者使用get阻塞住调用线程，直到计算完成返回结果，你也可以使用cancel方法停止任务的执行。

虽然Future以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的CPU资源，而且也不能及时地得到计算结果。

```
package net.ijiangtao.tech.concurrent.jsd.future.completable;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * java5 future
 *
 * @author ijiangtao
 * @create 2019-07-22 9:40
 **/
public class Java5Future {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //通过 while 循环等待异步计算处理成功
        ExecutorService pool = Executors.newFixedThreadPool(10);
        Future<Integer> f1 = pool.submit(() -> {
            // 长时间的异步计算 ……
            Thread.sleep(1);
            // 然后返回结果
            return 1001;
        });

        while (!f1.isDone())
            System.out.println("is not done yet");
        ;
        System.out.println("while isDone,result=" + f1.get());

        //通过阻塞的方式等待异步处理成功
        Future<Integer> f2 = pool.submit(() -> {
            // 长时间的异步计算 ……
            Thread.sleep(1);
            // 然后返回结果
            return 1002;
        });
        System.out.println("after blocking,result=" + f2.get());
    }

}
复制代码
```

## CompletableFuture的方法

前面我们提到 CompletableFuture 为我们提供了异步计算的实现，而这些实现都是通过它的方法实现的。

如果你打开它的文档[CompletableFuture-Java8Docs](https://link.juejin.im/?target=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Fapi%2Fjava%2Futil%2Fconcurrent%2FCompletableFuture.html)，你会发现CompletableFuture提供了将近60个方法。虽然方法很多，如果你仔细看的话，你会这些方法其中很多都是有相似性的。

只要你熟练掌握了这些方法，就能够得心应手地使用 CompletableFuture 来进行异步计算了。

Java的CompletableFuture类总是遵循这样的原则：

- 方法名不以Async结尾的方法由原来的线程计算
- 方法名以Async结尾且参数中没有Executor的方法由默认的线程池ForkJoinPool.commonPool()计算
- 方法名以Async结尾且参数中有Executor的方法由指定的线程池Executor计算

下面不再一一赘述了。

### supplyAsync ... 异步计算结果

| 返回值                  | 方法名及参数                                      | 方法说明                                                     |
| ----------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| staticCompletableFuture | supplyAsync(Supplier supplier)                    | Returns a new CompletableFuture that is asynchronously completed by a task running in the ForkJoinPool.commonPool() with the value obtained by calling the given Supplier. |
| staticCompletableFuture | supplyAsync(Supplier supplier, Executor executor) | Returns a new CompletableFuture that is asynchronously completed by a task running in the given executor with the value obtained by calling the given Supplier. |

supplyAsync方法以Supplier函数式接口类型为参数，CompletableFuture的计算结果类型为U。因为该方法的参数类型都是函数式接口，所以可以使用lambda表达式实现异步任务。后面讲解其他方法的时候，会举例子。

runAsync方法也好理解，它以Runnable函数式接口类型为参数，所以CompletableFuture的计算结果也为空（Runnable的run方法返回值为空）。这里就不再一一介绍了，感兴趣的小伙伴可以查看API文档。

### get ... 阻塞结果

| 返回值 | 方法名及参数                     | 方法说明                                                     |
| ------ | -------------------------------- | ------------------------------------------------------------ |
| T      | get()                            | Waits if necessary for this future to complete, and then returns its result. |
| T      | get(long timeout, TimeUnit unit) | Waits if necessary for at most the given time for this future to complete, and then returns its result, if available. |
| T      | getNow(T valueIfAbsent)          | Returns the result value (or throws any encountered exception) if completed, else returns the given valueIfAbsent. |
| T      | join()                           | Returns the result value when complete, or throws an (unchecked) exception if completed exceptionally. |

你可以像使用Future一样使用CompletableFuture，来进行阻塞式的计算（虽然不推荐使用）。其中getNow方法比较特殊：结果已经计算完则返回结果或者抛出异常，否则返回给定的valueIfAbsent值。

join和get方法都可以阻塞到计算完成然后获得返回结果，但两者的处理流程有所不同，可以参考下面一段代码来比较两者处理异常的不同之处：

```
  public static void main(String[] args) {
        try {
            new CompletableFutureDemo().test2();
            new CompletableFutureDemo().test3();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public void test2() throws Exception {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            int i = 1 / 0;
            return 100;
        });
        future.join();
    }

    public void test3() throws Exception {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            int i = 1 / 0;
            return 100;
        });
        future.get();
    }
复制代码
```

输出如下：

```
java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
	at java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:273)
	at java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:280)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1592)
	at java.util.concurrent.CompletableFuture$AsyncSupply.exec(CompletableFuture.java:1582)
	at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
	at java.util.concurrent.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1056)
	at java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1692)
	at java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:157)
Caused by: java.lang.ArithmeticException: / by zero
	at net.ijiangtao.tech.concurrent.jsd.future.completable.CompletableFutureDemo.lambda$test2$0(CompletableFutureDemo.java:32)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1590)
	... 5 more
复制代码
```

### thenApply ... 转换结果

| 返回值            | 方法名及参数                                                 | 方法说明                                                     |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CompletableFuture | thenApply(Function<? super T,? extends U> fn)                | Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied function. |
| CompletableFuture | thenApplyAsync(Function<? super T,? extends U> fn)           | Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage's result as the argument to the supplied function. |
| CompletableFuture | thenApplyAsync(Function<? super T,? extends U> fn, Executor executor) | Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied Executor, with this stage's result as the argument to the supplied function. |

使用CompletableFuture，我们不必因为等待一个计算完成而阻塞着调用线程，而是告诉CompletableFuture当计算完成的时候请执行某个function。而且我们还可以将这些操作串联起来，或者将CompletableFuture组合起来。

这一组函数的功能是当原来的CompletableFuture计算完后，将结果传递给函数fn，将fn的结果作为新的CompletableFuture计算结果。因此它的功能相当于将CompletableFuture 转换成 CompletableFuture 。

请看下面的例子：

```
  public static void main(String[] args) throws Exception {
        try {
            // new CompletableFutureDemo().test1();
        } catch (Exception e) {
            e.printStackTrace();
        }

        try {
            //new CompletableFutureDemo().test2();
            //new CompletableFutureDemo().test3();
        } catch (Exception e) {
            e.printStackTrace();
        }

        new CompletableFutureDemo().test4();

    }

    public void test4() throws Exception {
        // Create a CompletableFuture
        CompletableFuture<Integer> calculateFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("1");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                throw new IllegalStateException(e);
            }
            System.out.println("2");
            return 1 + 2;
        });

        // Attach a callback to the Future using thenApply()
        CompletableFuture<String> resultFuture = calculateFuture.thenApply(number -> {
            System.out.println("3");
            return "1 + 2 is " + number;
        });

        // Block and get the result of the future.
        System.out.println(resultFuture.get());
    }
复制代码
```

输出结果为：

```
1
2
3
1 + 2 is 3
复制代码
```

### thenAccept ... 纯消费结果

| 返回值            | 方法名及参数                                                 | 方法说明                                                     |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CompletableFuture | thenAccept(Consumer<? super T> action)                       | Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied action. |
| CompletableFuture | thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action) | Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied action. |

> 篇幅原因，Async和Executor方法不再列举。

只对结果执行Action，而不返回新的计算值，因此计算值为Void。这就好像生产者生产了消息，消费者消费消息以后不再进行消息的生产一样，因此thenAccept是对计算结果的纯消费。

例如如下方法：

```
public void test5() throws Exception {
    // thenAccept() example
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
        return "ijiangtao";
    }).thenAccept(name -> {
        System.out.println("Hi, " + name);
    });
    System.out.println(future.get());
}
复制代码
```

thenAccept的get返回为null:

```
Hi, ijiangtao
null
复制代码
```

thenAcceptBoth可以消费两者(生产和消费)的结果，下面提供一个例子：

```
 public void test6() throws Exception {
    CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "hello";
    }).thenAcceptBoth(CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "ijiangtao";
    }), (s1, s2) -> {
        System.out.println(s1 + " " + s2);
    });

    while (true){

    }
}
复制代码
```

输出如下：

```
hello ijiangtao
复制代码
```

### thenCombine ... 消费结果并返回

| 返回值                  | 方法名及参数                                                 | 方法说明                                                     |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| <U,V> CompletableFuture | thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn) | Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied function. |
| <U,V> CompletableFuture | thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn) | Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using this stage's default asynchronous execution facility, with the two results as arguments to the supplied function. |

从功能上来讲, thenCombine的功能更类似thenAcceptBoth，只不过thenAcceptBoth是纯消费，它的函数参数没有返回值，而thenCombine的函数参数fn有返回值。

```
public void test7() throws Exception {
    CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> {
        return 1+2;
    });
    CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
        return "1+2 is";
    });
    CompletableFuture<String> f = future1.thenCombine(future2, (x, y) -> y + " " + x);
    System.out.println(f.get()); // 输出：1+2 is 3
}
复制代码
```

### thenCompose ... 非嵌套整合

| 返回值            | 方法名及参数                                                 | 方法说明                                                     |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CompletableFuture | thenCompose(Function<? super T,? extends CompletionStage> fn) | Returns a new CompletionStage that, when this stage completes normally, is executed with this stage as the argument to the supplied function. |

> 由于篇幅原因，Async和Executor方法不再列举。

thenCompose方法接受一个Function作为参数，这个Function的输入是当前的CompletableFuture的计算值，返回结果将是一个新的CompletableFuture。

假如你需要将两个CompletableFutures相互整合，如果使用thenApply，则结果会是嵌套的CompletableFuture：

```
CompletableFuture<String> getUsersName(Long id) {
    return CompletableFuture.supplyAsync(() -> {
        return "ijiangtao";
    });
}

CompletableFuture<Integer> getUserAge(String userName) {
    return CompletableFuture.supplyAsync(() -> {
        return 20;
    });
}

public void test8(Long id) throws Exception {
    CompletableFuture<CompletableFuture<Integer>> result1 = getUsersName(id)
            .thenApply(userName -> getUserAge(userName));
}
复制代码
```

这时候可以使用thenCompose来获得第二个计算的CompletableFuture：

```
public void test9(Long id) throws Exception {
    CompletableFuture<Integer> result2 = getUsersName(id)
            .thenCompose(userName -> getUserAge(userName));
}
复制代码
```

### whenComplete ... 完成以后

| 返回值            | 方法名及参数                                                 | 方法说明                                                     |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CompletableFuture | whenComplete(BiConsumer<? super T,? super Throwable> action) | Returns a new CompletionStage with the same result or exception as this stage, that executes the given action when this stage completes. |
| CompletableFuture | whenCompleteAsync(BiConsumer<? super T,? super Throwable> action) | Returns a new CompletionStage with the same result or exception as this stage, that executes the given action using this stage's default asynchronous execution facility when this stage completes. |
| CompletableFuture | whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor) | Returns a new CompletionStage with the same result or exception as this stage, that executes the given action using the supplied Executor when this stage completes. |

当CompletableFuture的计算结果完成，或者抛出异常的时候，我们可以执行特定的Action。whenComplete的参数Action的类型是BiConsumer<? super T,? super Throwable>，它可以处理正常的计算结果，或者异常情况。注意这几个方法都会返回CompletableFuture，当Action执行完毕后它的结果返回原始的CompletableFuture的计算结果或者返回异常。

whenComplete方法不以Async结尾，意味着Action使用相同的线程执行，而以Async结尾的方法可能会使用其它的线程去执行(如果使用相同的线程池，也可能会被同一个线程选中执行)。

下面演示一下异常情况：

```java
public void test10() throws Exception {
    String result = CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (1 == 1) {
            throw new RuntimeException("an RuntimeException");
        }
        return "s1";
    }).whenComplete((s, t) -> {
        System.out.println("whenComplete s:"+s);
        System.out.println("whenComplete exception:"+t.getMessage());
    }).exceptionally(e -> {
        System.out.println("exceptionally exception:"+e.getMessage());
        return "hello ijiangtao";
    }).join();

    System.out.println(result);
}
```

输出：

```
whenComplete s:null
whenComplete exception:java.lang.RuntimeException: an RuntimeException
exceptionally exception:java.lang.RuntimeException: an RuntimeException
hello ijiangtao
复制代码
```

## 总结

Java5新增的Future类，可以实现阻塞式的异步计算，但这种阻塞的方式显然和我们的异步编程的初衷相违背。为了解决这个问题，JDK吸收了Guava的设计思想，加入了Future的诸多扩展功能形成了CompletableFuture。

本文重点介绍了CompletableFuture的不同类型的API，掌握了这些API对于使用非阻塞的函数式异步编程进行日常开发非常有帮助，同时也为下面深入了解异步编程的各种原理和特性打下了良好的基础。