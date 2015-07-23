# 5.1-google Guava 包的 ListenableFuture 解析

并发编程是一个难题，但是一个强大而简单的抽象可以显著的简化并发的编写。出于这样的考虑，Guava  定义了 [ListenableFuture](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListenableFuture.html) 接口并继承了 JDK concurrent 包下的 Future 接口。

**我们强烈地建议你在代码中多使用 ListenableFuture 来代替 JDK 的 Future, 因为：**

- 大多数 Futures 方法中需要它。
- 转到 ListenableFuture 编程比较容易。
- Guava 提供的通用公共类封装了公共的操作方方法，不需要提供 Future 和 ListenableFuture 的扩展方法。

## 接口

传统 JDK 中的 Future 通过异步的方式计算返回结果:在多线程运算中可能或者可能在没有结束返回结果，Future 是运行中的多线程的一个引用句柄，确保在服务执行返回一个 Result。

ListenableFuture 可以允许你注册回调方法(callbacks)，在运算（多线程执行）完成的时候进行调用,  或者在运算（多线程执行）完成后立即执行。这样简单的改进，使得可以明显的支持更多的操作，这样的功能在 JDK concurrent 中的 Future 是不支持的。

ListenableFuture 中的基础方法是 <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListenableFuture.html#addListener(java.lang.Runnable, java.util.concurrent.Executor)" rel="nofollow"><tt>addListener(Runnable, Executor)</tt></a>, 该方法会在多线程运算完的时候，指定的 Runnable 参数传入的对象会被指定的 Executor 执行。

## 添加回调（Callbacks）

多数用户喜欢使用 <tt><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#addCallback(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.FutureCallback, java.util.concurrent.Executor)" rel="nofollow">Futures.addCallback(ListenableFuture&lt;V&gt;, FutureCallback&lt;V&gt;, Executor)</a>的方式</tt>, 或者 另外一个版本<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#addCallback(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.FutureCallback)" rel="nofollow">version</a>（译者注：<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/src-html/com/google/common/util/concurrent/Futures.html#line.1106">addCallback</a>(<a title="interface in com.google.common.util.concurrent" href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListenableFuture.html">ListenableFuture</a>&lt;V&gt; future,<a title="interface in com.google.common.util.concurrent" href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/FutureCallback.html">FutureCallback</a>&lt;? super V&gt; callback)），默认是采用 MoreExecutors.sameThreadExecutor()线程池, 为了简化使用，Callback 采用轻量级的设计.  <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/FutureCallback.html" rel="nofollow"><tt>FutureCallback&lt;V&gt;</tt></a> 中实现了两个方法:</p>

- [onSuccess(V)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/FutureCallback.html#onSuccess(V)),在 Future 成功的时候执行，根据 Future 结果来判断。
- [onFailure(Throwable)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/FutureCallback.html#onFailure(java.lang.Throwable)), 在 Future 失败的时候执行，根据 Future 结果来判断。

## ListenableFuture 的创建

对应 JDK 中的 [ExecutorService.submit(Callable)](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/ExecutorService.html#submit(java.util.concurrent.Callable)) 提交多线程异步运算的方式，Guava 提供了 [ListeningExecutorService](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListeningExecutorService.html) 接口, 该接口返回  ListenableFuture 而相应的 ExecutorService 返回普通的  Future。将 ExecutorService 转为 ListeningExecutorService，可以使用 [MoreExecutors.listeningDecorator(ExecutorService)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/MoreExecutors.html#listeningDecorator(java.util.concurrent.ExecutorService))进行装饰。

```

    ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));
    ListenableFuture explosion = service.submit(new Callable() {
      public Explosion call() {
    return pushBigRedButton();
      }
    });
    Futures.addCallback(explosion, new FutureCallback() {
      // we want this handler to run immediately after we push the big red button!
      public void onSuccess(Explosion explosion) {
    walkAwayFrom(explosion);
      }
      public void onFailure(Throwable thrown) {
    battleArchNemesis(); // escaped the explosion!
      }
    });

```

另外, 假如你是从 [FutureTask](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/FutureTask.html) 转换而来的, Guava 提供 [ListenableFutureTask.create(Callable<V>)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListenableFutureTask.html#create(java.util.concurrent.Callable)) 和 <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/ListenableFutureTask.html#create(java.lang.Runnable, V)" rel="nofollow"><tt>ListenableFutureTask.create(Runnable, V)</tt></a>. 和 JDK 不同的是, ListenableFutureTask 不能随意被继承（译者注：ListenableFutureTask 中的 done 方法实现了调用 listener 的操作）。

假如你喜欢抽象的方式来设置 future 的值，而不是想实现接口中的方法，可以考虑继承抽象类 [AbstractFuture<V>](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/AbstractFuture.html) 或者直接使用 [SettableFuture](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/SettableFuture.html) 。

假如你必须将其他 API 提供的 Future 转换成 ListenableFuture，你没有别的方法只能采用硬编码的方式 [JdkFutureAdapters.listenInPoolThread(Future)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/JdkFutureAdapters.html) 来将 Future 转换成  ListenableFuture。尽可能地采用修改原生的代码返回 ListenableFuture 会更好一些。

## Application

使用 ListenableFuture 最重要的理由是它可以进行一系列的复杂链式的异步操作。

```

    ListenableFuture rowKeyFuture = indexService.lookUp(query);
    AsyncFunction<RowKey, QueryResult> queryFunction =
    new AsyncFunction<RowKey, QueryResult>() {
    public ListenableFuture apply(RowKey rowKey) {
    return dataService.read(rowKey);
    }
    };
    ListenableFuture queryFuture = Futures.transform(rowKeyFuture, queryFunction, queryExecutor);

```

其他更多的操作可以更加有效的支持而 JDK 中的 Future 是没法支持的.

不同的操作可以在不同的 Executors 中执行，单独的 ListenableFuture 可以有多个操作等待。

当一个操作开始的时候其他的一些操作也会尽快开始执行–“fan-out”–ListenableFuture 能够满足这样的场景：促发所有的回调（callbacks）。反之更简单的工作是，同样可以满足“fan-in”场景，促发 ListenableFuture 获取（get）计算结果，同时其它的 Futures 也会尽快执行：可以参考 [the implementation of Futures.allAsList](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/src-html/com/google/common/util/concurrent/Futures.html#line.1276) 。（译者注：fan-in 和 fan-out 是软件设计的一个术语，可以参考这里：http://baike.baidu.com/view/388892.htm#1 或者看这里的解析 [Design Principles: Fan-In vs Fan-Out](http://it.toolbox.com/blogs/enterprise-solutions/design-principles-fanin-vs-fanout-16088)，这里 fan-out 的实现就是封装的 ListenableFuture 通过回调，调用其它代码片段。fan-in 的意义是可以调用其它 Future）

<table>
<tbody>
<tr>
<td>方法</td>
<td>描述</td>
<td>参考</td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction, java.util.concurrent.Executor)" rel="nofollow"><tt>transform(ListenableFuture&lt;A&gt;, AsyncFunction&lt;A, B&gt;, Executor)</tt></a><tt>*</tt></td>
<td><tt><span style="font-family: Verdana, Arial, Helvetica, sans-serif">返回一个新的 </span>ListenableFuture</tt> ，该 <tt>ListenableFuture</tt> 返回的 result 是由传入的 <tt>AsyncFunction</tt>  参数指派到传入的 <tt>ListenableFuture 中</tt>.</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction)" rel="nofollow"><tt>transform(ListenableFuture&lt;A&gt;, AsyncFunction&lt;A, B&gt;)</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function, java.util.concurrent.Executor)" rel="nofollow"><tt>transform(ListenableFuture&lt;A&gt;, Function&lt;A, B&gt;, Executor)</tt></a></td>
<td><tt>返回一个新的 ListenableFuture</tt> ，该 <tt>ListenableFuture</tt> 返回的 result 是由传入的<tt>Function</tt> 参数指派到传入的 <tt>ListenableFuture 中</tt>.</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function)" rel="nofollow"><tt>transform(ListenableFuture&lt;A&gt;, Function&lt;A, B&gt;)</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#allAsList(java.lang.Iterable)" rel="nofollow"><tt>allAsList(Iterable&lt;ListenableFuture&lt;V&gt;&gt;)</tt></a></td>
<td><tt>返回一个 ListenableFuture</tt> ，该<tt>ListenableFuture</tt> 返回的 result 是一个 List，List 中的值是每个 ListenableFuture 的返回值，假如传入的其中之一 fails 或者 cancel，这 个Future fails 或者 canceled</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#allAsList(com.google.common.util.concurrent.ListenableFuture...)" rel="nofollow"><tt>allAsList(ListenableFuture&lt;V&gt;...)</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#successfulAsList(java.lang.Iterable)" rel="nofollow"><tt>successfulAsList(Iterable&lt;ListenableFuture&lt;V&gt;&gt;)</tt></a></td>
<td><tt>返回一个 ListenableFuture</tt> ，该 Future 的结果包含所有成功的 Future，按照原来的顺序，当其中之一 Failed 或者 cancel，则用 null 替代</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#successfulAsList(com.google.common.util.concurrent.ListenableFuture...)" rel="nofollow"><tt>successfulAsList(ListenableFuture&lt;V&gt;...)</tt></a></td>
</tr>
</tbody>
</table>

[AsyncFunction<A, B>](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/AsyncFunction.html) 中提供一个方法 <tt>ListenableFuture&lt;B&gt; apply(A input)，它可以被用于异步变换值。

```

    List<ListenableFuture> queries;
    // The queries go to all different data centers, but we want to wait until they're all done or failed.
    
    ListenableFuture<List> successfulQueries = Futures.successfulAsList(queries);
    
    Futures.addCallback(successfulQueries, callbackOnSuccessfulQueries);

```

## CheckedFuture

Guava 也提供了 [CheckedFuture<V, X extends Exception>](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/CheckedFuture.html) 接口。CheckedFuture 是一个 ListenableFuture ，其中包含了多个版本的 get 方法，方法声明抛出检查异常.这样使得创建一个在执行逻辑中可以抛出异常的 Future 更加容易 。将 ListenableFuture 转换成 CheckedFuture，可以使用  <tt><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#makeChecked(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function)" rel="nofollow">Futures.makeChecked(ListenableFuture&lt;V&gt;, Function&lt;Exception, X&gt;)</a>。</tt>

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [google Guava 包的 ListenableFuture 解析](http://ifeve.com/google-guava-listenablefuture/)