# 2.3-强大的集合工具类：java.util.Collections 中未包含的集合工具

     尚未完成: Queues, Tables 工具类

任何对 JDK 集合框架有经验的程序员都熟悉和喜欢 [java.util.Collections](http://docs.oracle.com/javase/7/docs/api/java/util/Collections.html) 包含的工具方法。Guava 沿着这些路线提供了更多的工具方法：适用于所有集合的静态方法。这是 Guava 最流行和成熟的部分之一。

我们用相对直观的方式把工具类与特定集合接口的对应关系归纳如下：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="96"><b>集合接口</b><b></b></td>
<td width="144"><b>属于</b><b>JDK</b><b>还是</b><b>Guava</b></td>
<td width="372"><b>对应的</b><b>Guava</b><b>工具类</b><b></b></td>
</tr>
<tr>
<td width="96">Collection</td>
<td width="144">JDK</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Collections2.html"><tt>Collections2</tt></a>：不要和 java.util.Collections 混淆</td>
</tr>
<tr>
<td width="96">List</td>
<td width="144">JDK</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Lists.html"><tt>Lists</tt></a></td>
</tr>
<tr>
<td width="96">Set</td>
<td width="144">JDK</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Sets.html"><tt>Sets</tt></a></td>
</tr>
<tr>
<td width="96">SortedSet</td>
<td width="144">JDK</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Sets.html"><tt>Sets</tt></a></td>
</tr>
<tr>
<td width="96">Map</td>
<td width="144">JDK</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Maps.html"><tt>Maps</tt></a></td>
</tr>
<tr>
<td width="96">SortedMap</td>
<td width="144">JDK</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Maps.html"><tt>Maps</tt></a></td>
</tr>
<tr>
<td width="96">Queue</td>
<td width="144">JDK</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Queues.html"><tt>Queues</tt></a></td>
</tr>
<tr>
<td width="96"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multiset">Multiset</a></td>
<td width="144">Guava</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html"><tt>Multisets</tt></a></td>
</tr>
<tr>
<td width="96"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multimap">Multimap</a></td>
<td width="144">Guava</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html"><tt>Multimaps</tt></a></td>
</tr>
<tr>
<td width="96"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#BiMap">BiMap</a></td>
<td width="144">Guava</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html"><tt>Maps</tt></a></td>
</tr>
<tr>
<td width="96"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Table">Table</a></td>
<td width="144">Guava</td>
<td width="372"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html"><tt>Tables</tt></a></td>
</tr>
</tbody>
</table>

*在找类似转化、过滤的方法？请看第四章，函数式风格。*

## 静态工厂方法

在 JDK 7之前，构造新的范型集合时要讨厌地重复声明范型：

```

    List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<TypeThatsTooLongForItsOwnGood>();

```

我想我们都认为这很讨厌。因此 Guava 提供了能够推断范型的静态工厂方法：

```

    List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();
    Map<KeyType, LongishValueType> map = Maps.newLinkedHashMap();

```

可以肯定的是，JDK7 版本的钻石操作符(<>)没有这样的麻烦：

```

     List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<>();

```

但 Guava 的静态工厂方法远不止这么简单。用工厂方法模式，我们可以方便地在初始化时就指定起始元素。

```

    Set<Type> copySet = Sets.newHashSet(elements);
    List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");

```

此外，通过为工厂方法命名（Effective Java 第一条），我们可以提高集合初始化大小的可读性：

```

    List<Type> exactly100 = Lists.newArrayListWithCapacity(100);
    List<Type> approx100 = Lists.newArrayListWithExpectedSize(100);
    Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);

```

确切的静态工厂方法和相应的工具类一起罗列在下面的章节。

注意：Guava 引入的新集合类型没有暴露原始构造器，也没有在工具类中提供初始化方法。而是直接在集合类中提供了静态工厂方法，例如：

```

    Multiset<String> multiset = HashMultiset.create();

```

## Iterables

在可能的情况下，Guava 提供的工具方法更偏向于接受 Iterable 而不是 Collection 类型。在 Google，对于不存放在主存的集合——比如从数据库或其他数据中心收集的结果集，因为实际上还没有攫取全部数据，这类结果集都不能支持类似 size()的操作 ——通常都不会用 Collection 类型来表示。

因此，很多你期望的支持所有集合的操作都在 [Iterables](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html) 类中。大多数 Iterables 方法有一个在 [Iterators](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html) 类中的对应版本，用来处理 Iterator。

截至 Guava 1.2 版本，Iterables 使用 [FluentIterable](http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html) 类进行了补充，它包装了一个 Iterable 实例，并对许多操作提供了”fluent”（链式调用）语法。

下面列出了一些最常用的工具方法，但更多 Iterables 的函数式方法将在第四章讨论。

### 常规方法

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#concat(java.lang.Iterable)"><tt>concat(Iterable&lt;Iterable&gt;)</tt></a></td>
<td width="222">串联多个 iterables 的懒视图*</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#concat(java.lang.Iterable...)"><tt>concat(Iterable...)</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#frequency(java.lang.Iterable, java.lang.Object)"><tt>frequency(Iterable, Object)</tt></a></td>
<td width="222">返回对象在 iterable 中出现的次数</td>
<td width="211">与 Collections.frequency (Collection,   Object)比较；<tt></tt><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multiset">Multiset</a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#partition(java.lang.Iterable, int)"><tt>partition(Iterable, int)</tt></a></td>
<td width="222">把 iterable 按指定大小分割，得到的子集都不能进行修改操作</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#partition(java.util.List, int)"><tt>Lists.partition(List, int)</tt></a>；<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#paddedPartition(java.lang.Iterable, int)"><tt>paddedPartition(Iterable, int)</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#getFirst(java.lang.Iterable, T)"><tt>getFirst(Iterable, T default)</tt></a></td>
<td width="222">返回 iterable 的第一个元素，若 iterable 为空则返回默认值</td>
<td width="211">与Iterable.iterator(). next()比较;<a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#first()"><tt>FluentIterable.first()</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#getLast(java.lang.Iterable)"><tt>getLast(Iterable)</tt></a></td>
<td width="222">返回 iterable 的最后一个元素，若 iterable 为空则抛出NoSuchElementException</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#getLast(java.lang.Iterable, T)"><tt>getLast(Iterable, T default)</tt></a>；<br />
<a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#last()"><tt>FluentIterable.last()</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#elementsEqual(java.lang.Iterable, java.lang.Iterable)"><tt>elementsEqual(Iterable, Iterable)</tt></a></td>
<td width="222">如果两个 iterable 中的所有元素相等且顺序一致，返回 true</td>
<td width="211">与 List.equals(Object)比较</td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#unmodifiableIterable(java.lang.Iterable)"><tt>unmodifiableIterable(Iterable)</tt></a></td>
<td width="222">返回 iterable 的不可变视图</td>
<td width="211">与 Collections. unmodifiableCollection(Collection)比较</td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#limit(java.lang.Iterable, int)"><tt>limit(Iterable, int)</tt></a></td>
<td width="222">限制 iterable 的元素个数限制给定值</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#limit(int)"><tt>FluentIterable.limit(int)</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#getOnlyElement(java.lang.Iterable)"><tt>getOnlyElement(Iterable)</tt></a></td>
<td width="222">获取 iterable 中唯一的元素，如果 iterable 为空或有多个元素，则快速失败</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#getOnlyElement(java.lang.Iterable, T)"><tt>getOnlyElement(Iterable, T default)</tt></a></td>
</tr>
</tbody>
</table>

*译者注：懒视图意味着如果还没访问到某个 iterable 中的元素，则不会对它进行串联操作。*

```


    Iterable<Integer> concatenated = Iterables.concat(
	        Ints.asList(1, 2, 3),
	        Ints.asList(4, 5, 6)); // concatenated包括元素 1, 2, 3, 4, 5, 6
    String lastAdded = Iterables.getLast(myLinkedHashSet);
    String theElement = Iterables.getOnlyElement(thisSetIsDefinitelyASingleton);
    //如果set不是单元素集，就会出错了！

```

### 与 Collection 方法相似的工具方法

通常来说，Collection 的实现天然支持操作其他 Collection，但却不能操作 Iterable。

下面的方法中，如果传入的 Iterable 是一个 Collection 实例，则实际操作将会委托给相应的 Collection 接口方法。例如，往 Iterables.size 方法传入是一个 Collection 实例，它不会真的遍历 iterator 获取大小，而是直接调用 Collection.size。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="186"><b>方法</b><b></b></td>
<td width="222"><b>类似的</b><b> Collection </b><b>方法</b><b></b></td>
<td width="211"><b>等价的</b><b> FluentIterable </b><b>方法</b><b></b></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#addAll(java.util.Collection, java.lang.Iterable)"><tt>addAll(Collection addTo,   Iterable toAdd)</tt></a></td>
<td width="222">Collection.addAll(Collection)</td>
<td width="211"></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#contains(java.lang.Iterable, java.lang.Object)"><tt>contains(Iterable, Object)</tt></a></td>
<td width="222">Collection.contains(Object)</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#contains(java.lang.Object)"><tt>FluentIterable.contains(Object)</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#removeAll(java.lang.Iterable, java.util.Collection)"><tt>removeAll(Iterable   removeFrom, Collection toRemove)</tt></a></td>
<td width="222">Collection.removeAll(Collection)</td>
<td width="211"></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#retainAll(java.lang.Iterable, java.util.Collection)"><tt>retainAll(Iterable   removeFrom, Collection toRetain)</tt></a></td>
<td width="222">Collection.retainAll(Collection)</td>
<td width="211"></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#size(java.lang.Iterable)"><tt>size(Iterable)</tt></a></td>
<td width="222">Collection.size()</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#size()"><tt>FluentIterable.size()</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#toArray(java.lang.Iterable, java.lang.Class)"><tt>toArray(Iterable, Class)</tt></a></td>
<td width="222">Collection.toArray(T[])</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#toArray(java.lang.Class)"><tt>FluentIterable.toArray(Class)</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#isEmpty(java.lang.Iterable)"><tt>isEmpty(Iterable)</tt></a></td>
<td width="222">Collection.isEmpty()</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#isEmpty()"><tt>FluentIterable.isEmpty()</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#get(java.lang.Iterable, int)"><tt>get(Iterable, int)</tt></a></td>
<td width="222">List.get(int)</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-%20history/release12/javadoc/com/google/common/collect/FluentIterable.html#get(int)"><tt>FluentIterable.get(int)</tt></a></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#toString(java.lang.Iterable)"><tt>toString(Iterable)</tt></a></td>
<td width="222">Collection.toString()</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#toString()"><tt>FluentIterable.toString()</tt></a></td>
</tr>
</tbody>
</table>

### FluentIterable

除了上面和第四章提到的方法，FluentIterable 还有一些便利方法用来把自己拷贝到不可变集合

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="186">ImmutableList</td>
<td width="282"></td>
</tr>
<tr>
<td width="186">ImmutableSet</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#toImmutableSet()"><tt>toImmutableSet()</tt></a></td>
</tr>
<tr>
<td width="186">ImmutableSortedSet</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#toImmutableSortedSet(java.util.Comparator)"><tt>toImmutableSortedSet(Comparator)</tt></a></td>
</tr>
</tbody>
</table>

## Lists

除了静态工厂方法和函数式编程方法，[Lists](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Lists.html) 为 List 类型的对象提供了若干工具方法。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="186"><b>方法</b><b></b></td>
<td width="426"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#partition(java.util.List, int)"><tt>partition(List, int)</tt></a></td>
<td width="426">把 List 按指定大小分割</td>
</tr>
<tr>
<td width="186"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#reverse(java.util.List)"><tt>reverse(List)</tt></a></td>
<td width="426">返回给定 List 的反转视图。注: 如果 List 是不可变的，考虑改用<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableList.html#reverse()"><tt> ImmutableList.reverse()</tt></a>。</td>
</tr>
</tbody>
</table>

```

    List countUp = Ints.asList(1, 2, 3, 4, 5);
    List countDown = Lists.reverse(theList); // {5, 4, 3, 2, 1}
    List<List> parts = Lists.partition(countUp, 2);//{{1,2}, {3,4}, {5}}

```

### 静态工厂方法

Lists 提供如下静态工厂方法：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="108"><b>具体实现类型</b><b></b></td>
<td width="504"><b>工厂方法</b><b></b></td>
</tr>
<tr>
<td width="108">ArrayList</td>
<td width="504"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newArrayList()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newArrayList(E...)">with elements</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newArrayList(java.lang.Iterable)">from <tt>Iterable</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newArrayListWithCapacity(int)">with exact capacity</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newArrayListWithExpectedSize(int)">with expected size</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newArrayList(java.util.Iterator)">from <tt>Iterator</tt></a></td>
</tr>
<tr>
<td width="108">LinkedList</td>
<td width="504"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newLinkedList()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#newLinkedList(java.lang.Iterable)">from <tt>Iterable</tt></a></td>
</tr>
</tbody>
</table>

## Sets

[Sets](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Sets.html) 工具类包含了若干好用的方法。

### 集合理论方法

我们提供了很多标准的集合运算（Set-Theoretic）方法，这些方法接受 Set 参数并返回 SetView，可用于：

- 直接当作 Set 使用，因为 SetView 也实现了 Set 接口；
- 用 [copyInto(Set)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.SetView.html#copyInto(S)) 拷贝进另一个可变集合；
- 用 [immutableCopy()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.SetView.html#immutableCopy())对自己做不可变拷贝。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="252"><b>方法</b><b></b></td>
</tr>
<tr>
<td width="252"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#union(java.util.Set, java.util.Set)"><tt>union(Set, Set)</tt></a></td>
</tr>
<tr>
<td width="252"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#intersection(java.util.Set, java.util.Set)"><tt>intersection(Set, Set)</tt></a></td>
</tr>
<tr>
<td width="252"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#difference(java.util.Set, java.util.Set)"><tt>difference(Set, Set)</tt></a></td>
</tr>
<tr>
<td width="252"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#symmetricDifference(java.util.Set, java.util.Set)"><tt>symmetricDifference(Set,   Set)</tt></a></td>
</tr>
</tbody>
</table>

使用范例：

```

    Set<String> wordsWithPrimeLength = ImmutableSet.of("one", "two", "three", "six", "seven", "eight");
    Set<String> primes = ImmutableSet.of("two", "three", "five", "seven");
    SetView<String> intersection = Sets.intersection(primes,wordsWithPrimeLength);
    // intersection包含"two", "three", "seven"
    return intersection.immutableCopy();//可以使用交集，但不可变拷贝的读取效率更高

```

### 其他 Set 工具方法

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="228"><b>方法</b><b></b></td>
<td width="180"><b>描述</b><b></b></td>
<td width="211"><b>另请参见</b><b></b></td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#cartesianProduct(java.util.List)"><tt>cartesianProduct(List&lt;Set&gt;)</tt></a></td>
<td width="180">返回所有集合的笛卡儿积</td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#cartesianProduct(java.util.Set...)"><tt>cartesianProduct(Set...)</tt></a></td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#powerSet(java.util.Set)"><tt>powerSet(Set)</tt></a></td>
<td width="180">返回给定集合的所有子集</td>
<td width="211"></td>
</tr>
</tbody>
</table>


```

    Set<String> animals = ImmutableSet.of("gerbil", "hamster");
    Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");
    
    Set<List<String>> product = Sets.cartesianProduct(animals, fruits);
    // {{"gerbil", "apple"}, {"gerbil", "orange"}, {"gerbil", "banana"},
    //  {"hamster", "apple"}, {"hamster", "orange"}, {"hamster", "banana"}}
    
    Set<Set<String>> animalSets = Sets.powerSet(animals);
    // {{}, {"gerbil"}, {"hamster"}, {"gerbil", "hamster"}}

```

### 静态工厂方法

Sets 提供如下静态工厂方法：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="120"><b>具体实现类型</b><b></b></td>
<td width="492"><b>工厂方法</b><b></b></td>
</tr>
<tr>
<td width="120">HashSet</td>
<td width="492"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newHashSet()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newHashSet(E...)">with elements</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newHashSet(java.lang.Iterable)">from <tt>Iterable</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newHashSetWithExpectedSize(int)">with expected size</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newHashSet(java.util.Iterator)">from <tt>Iterator</tt></a></td>
</tr>
<tr>
<td width="120">LinkedHashSet</td>
<td width="492"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newLinkedHashSet()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newLinkedHashSet(java.lang.Iterable)">from <tt>Iterable</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newLinkedHashSetWithExpectedSize(int)">with expected size</a></td>
</tr>
<tr>
<td width="120">TreeSet</td>
<td width="492"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newTreeSet()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newTreeSet(java.util.Comparator)">with <tt>Comparator</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#newTreeSet(java.lang.Iterable)">from <tt>Iterable</tt></a></td>
</tr>
</tbody>
</table>

## Maps

[Maps](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html) 类有若干值得单独说明的、很酷的方法。

**uniqueIndex**

<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#uniqueIndex(java.lang.Iterable, com.google.common.base.Function)"><tt>Maps.uniqueIndex(Iterable,Function)</tt></a> 通常针对的场景是：有一组对象，它们在某个属性上分别有独一无二的值，而我们希望能够按照这个属性值查找对象——*译者注：这个方法返回一个 Map，键为 Function 返回的属性值，值为 Iterable 中相应的元素，因此我们可以反复用这个 Map 进行查找操作。*

比方说，我们有一堆字符串，这些字符串的长度都是独一无二的，而我们希望能够按照特定长度查找字符串：

```


    ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings,
    new Function<String, Integer> () {
        public Integer apply(String string) {
            return string.length();
        }
    });

```

如果索引值不是独一无二的，请参见下面的 Multimaps.index 方法。

**difference**


<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#difference(java.util.Map, java.util.Map)"><tt>Maps.difference(Map, Map)</tt></a> 用来比较两个 Map 以获取所有不同点。该方法返回 MapDifference 对象，把不同点的维恩图分解为：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/MapDifference.html#entriesInCommon()"><tt>entriesInCommon()</tt></a></td>
<td width="438">两个 Map 中都有的映射项，包括匹配的键与值</td>
</tr>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/MapDifference.html#entriesDiffering()"><tt>entriesDiffering()</tt></a></td>
<td width="438">键相同但是值不同值映射项。返回的 Map 的值类型为 <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/MapDifference.ValueDifference.html"><tt>MapDifference.ValueDifference</tt></a>，以表示左右两个不同的值</td>
</tr>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/MapDifference.html#entriesOnlyOnLeft()"><tt>entriesOnlyOnLeft()</tt></a></td>
<td width="438">键只存在于左边 Map 的映射项</td>
</tr>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/MapDifference.html#entriesOnlyOnRight()"><tt>entriesOnlyOnRight()</tt></a></td>
<td width="438">键只存在于右边 Map 的映射项</td>
</tr>
</tbody>
</table>

```

    Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
    Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
    MapDifference<String, Integer> diff = Maps.difference(left, right);
    
    diff.entriesInCommon(); // {"b" => 2}
    diff.entriesInCommon(); // {"b" => 2}
    diff.entriesOnlyOnLeft(); // {"a" => 1}
    diff.entriesOnlyOnRight(); // {"d" => 5}

```

### 处理 BiMap 的工具方法

Guava 中处理 BiMap 的工具方法在 Maps 类中，因为 BiMap 也是一种 Map 实现。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="300"><b>BiMap</b><b>工具方法</b><b></b></td>
<td width="312"><b>相应的 </b><b>Map</b><b> 工具方法</b><b></b></td>
</tr>
<tr>
<td width="300"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#synchronizedBiMap(com.google.common.collect.BiMap)"><tt>synchronizedBiMap(BiMap)</tt></a></td>
<td width="312">Collections.synchronizedMap(Map)</td>
</tr>
<tr>
<td width="300"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#unmodifiableBiMap(com.google.common.collect.BiMap)"><tt>unmodifiableBiMap(BiMap)</tt> </a></td>
<td width="312">Collections.unmodifiableMap(Map)</td>
</tr>
</tbody>
</table>

### 静态工厂方法

Maps 提供如下静态工厂方法：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="300"><b>具体实现类型</b><b></b></td>
<td width="312"><b>工厂方法</b><b></b></td>
</tr>
<tr>
<td width="300">HashMap</td>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newHashMap()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newHashMap(java.util.Map)">from <tt>Map</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newHashMapWithExpectedSize(int)">with expected size</a></td>
</tr>
<tr>
<td width="300">LinkedHashMap</td>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newLinkedHashMap()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newLinkedHashMap(java.util.Map)">from <tt>Map</tt></a></td>
</tr>
<tr>
<td width="300">TreeMap</td>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newTreeMap()">basic</a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newTreeMap(java.util.Comparator)">from <tt>Comparator</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newTreeMap(java.util.SortedMap)">from <tt>SortedMap</tt></a></td>
</tr>
<tr>
<td width="300">EnumMap</td>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newEnumMap(java.lang.Class)">from <tt>Class</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newEnumMap(java.util.Map)">from <tt>Map</tt></a></td>
</tr>
<tr>
<td width="300">ConcurrentMap：支持所有操作</td>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newConcurrentMap()">basic</a></td>
</tr>
<tr>
<td width="300">IdentityHashMap</td>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#newIdentityHashMap()">basic</a></td>
</tr>
</tbody>
</table>

## Multisets

标准的 Collection 操作会忽略 Multiset 重复元素的个数，而只关心元素是否存在于 Multiset 中，如 containsAll 方法。为此，[Multisets](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multisets.html) 提供了若干方法，以顾及 Multiset 元素的重复性：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="228"><b>方法</b><b></b></td>
<td width="180"><b>说明</b><b></b></td>
<td width="211"><b>和 </b><b>Collection</b><b> 方法的区别</b><b></b></td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html#containsOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)"><tt>containsOccurrences(Multiset   sup, Multiset sub)</tt></a></td>
<td width="180">对任意 o，如果 sub.count(o)&lt;=super.count(o)，返回 true</td>
<td width="211">Collection.containsAll忽略个数，而只关心 sub 的元素是否都在 super 中</td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html#removeOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)"><tt>removeOccurrences(Multiset   removeFrom, Multiset toRemove)</tt></a></td>
<td width="180">对 toRemove 中的重复元素，仅在 removeFrom 中删除相同个数。</td>
<td width="211">Collection.removeAll 移除所有出现在 toRemove 的元素</td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html#retainOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)"><tt>retainOccurrences(Multiset   removeFrom, Multiset toRetain)</tt></a></td>
<td width="180">修改 removeFrom，以保证任意 o 都符合removeFrom.count(o)&lt;=toRetain.count(o)</td>
<td width="211">Collection.retainAll 保留所有出现在 toRetain 的元素</td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html#intersection(com.google.common.collect.Multiset, com.google.common.collect.Multiset)"><tt>intersection(Multiset,   Multiset)</tt></a></td>
<td width="180">返回两个 multiset 的交集;</td>
<td width="211">没有类似方法</td>
</tr>
</tbody>
</table>

```

    Multiset<String> multiset1 = HashMultiset.create();
    multiset1.add("a", 2);
    
    Multiset<String> multiset2 = HashMultiset.create();
    multiset2.add("a", 5);
    
    multiset1.containsAll(multiset2); //返回true；因为包含了所有不重复元素，
    //虽然multiset1实际上包含2个"a"，而multiset2包含5个"a"
    Multisets.containsOccurrences(multiset1, multiset2); // returns false
    
    multiset2.removeOccurrences(multiset1); // multiset2 现在包含3个"a"
    multiset2.removeAll(multiset1);//multiset2移除所有"a"，虽然multiset1只有2个"a"
    multiset2.isEmpty(); // returns true

```

Multisets 中的其他工具方法还包括：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multisets.html#copyHighestCountFirst(com.google.common.collect.Multiset)"><tt>copyHighestCountFirst(Multiset)</tt></a></td>
<td width="270">返回 Multiset 的不可变拷贝，并将元素按重复出现的次数做降序排列</td>
</tr>
<tr>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html#unmodifiableMultiset(com.google.common.collect.Multiset)"><tt>unmodifiableMultiset(Multiset)</tt></a></td>
<td width="270">返回 Multiset 的只读视图</td>
</tr>
<tr>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multisets.html#unmodifiableSortedMultiset(com.google.common.collect.SortedMultiset)"><tt>unmodifiableSortedMultiset(SortedMultiset)</tt></a></td>
<td width="270">返回 SortedMultiset 的只读视图</td>
</tr>
</tbody>
</table>

```

    Multiset<String> multiset = HashMultiset.create();
    multiset.add("a", 3);
    multiset.add("b", 5);
    multiset.add("c", 1);
    
    ImmutableMultiset highestCountFirst = Multisets.copyHighestCountFirst(multiset);
    //highestCountFirst，包括它的entrySet和elementSet，按{"b", "a", "c"}排列元素

```

## Multimaps

[Multimaps](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html) 提供了若干值得单独说明的通用工具方法

**index**

作为 Maps.uniqueIndex 的兄弟方法，[Multimaps.index(Iterable, Function)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#index(java.lang.Iterable, com.google.common.base.Function))通常针对的场景是：有一组对象，它们有共同的特定属性，我们希望按照这个属性的值查询对象，但属性值不一定是独一无二的。

比方说，我们想把字符串按长度分组。

```

    ImmutableSet digits = ImmutableSet.of("zero", "one", "two", "three", "four", "five", "six", "seven", "eight", "nine");
    Function<String, Integer> lengthFunction = new Function<String, Integer>() {
    public Integer apply(String string) {
    return string.length();
    }
    };
    
    ImmutableListMultimap<Integer, String> digitsByLength= Multimaps.index(digits, lengthFunction);
    /*
    *  digitsByLength maps:
    *  3 => {"one", "two", "six"}
    *  4 => {"zero", "four", "five", "nine"}
    *  5 => {"three", "seven", "eight"}
    */

```

**invertFrom**

鉴于 Multimap 可以把多个键映射到同一个值（译者注：实际上这是任何 map 都有的特性），也可以把一个键映射到多个值，反转 Multimap 也会很有用。Guava 提供了 <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#invertFrom(com.google.common.collect.Multimap, M)"><tt>invertFrom(Multimap toInvert,<br />
Multimap dest)</tt></a> 做这个操作，并且你可以自由选择反转后的 Multimap 实现。

注：如果你使用的是 ImmutableMultimap，考虑改用 [ImmutableMultimap.inverse()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableMultimap.html#inverse())做反转。


```

    ArrayListMultimap<String, Integer> multimap = ArrayListMultimap.create();
    multimap.putAll("b", Ints.asList(2, 4, 6));
    multimap.putAll("a", Ints.asList(4, 2, 1));
    multimap.putAll("c", Ints.asList(2, 5, 3));
    
    TreeMultimap<Integer, String> inverse = Multimaps.invertFrom(multimap, TreeMultimap<String, Integer>.create());
    //注意我们选择的实现，因为选了TreeMultimap，得到的反转结果是有序的
    /*
    * inverse maps:
    *  1 => {"a"}
    *  2 => {"a", "b", "c"}
    *  3 => {"c"}
    *  4 => {"a", "b"}
    *  5 => {"c"}
    *  6 => {"b"}
    */

```

**forMap**

想在 Map 对象上使用 Multimap 的方法吗？[forMap(Map)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#forMap(java.util.Map))把 Map 包装成 SetMultimap。这个方法特别有用，例如，与 Multimaps.invertFrom 结合使用，可以把多对一的 Map 反转为一对多的 Multimap。

```

    Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 1, "c", 2);
    SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
    // multimap：["a" => {1}, "b" => {1}, "c" => {2}]
    Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap<Integer, String>.create());
    // inverse：[1 => {"a","b"}, 2 => {"c"}]

```

**包装器**

Multimaps 提供了传统的包装方法，以及让你选择 Map 和 Collection 类型以自定义 Multimap 实现的工具方法。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="114">只读包装</td>
<td width="84"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#unmodifiableMultimap(com.google.common.collect.Multimap)"><tt>Multimap</tt></a></td>
<td width="114"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#unmodifiableListMultimap(com.google.common.collect.ListMultimap)"><tt>ListMultimap</tt></a></td>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#unmodifiableSetMultimap(com.google.common.collect.SetMultimap)"><tt>SetMultimap</tt></a></td>
<td width="168"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#unmodifiableSortedSetMultimap(com.google.common.collect.SortedSetMultimap)"><tt>SortedSetMultimap</tt></a></td>
</tr>
<tr>
<td width="114">同步包装</td>
<td width="84"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#synchronizedMultimap(com.google.common.collect.Multimap)"><tt>Multimap</tt></a></td>
<td width="114"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#synchronizedListMultimap(com.google.common.collect.ListMultimap)"><tt>ListMultimap</tt></a></td>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#synchronizedSetMultimap(com.google.common.collect.SetMultimap)"><tt>SetMultimap</tt></a></td>
<td width="168"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#synchronizedSortedSetMultimap(com.google.common.collect.SortedSetMultimap)"><tt>SortedSetMultimap</tt></a></td>
</tr>
<tr>
<td width="114">自定义实现</td>
<td width="84"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#newMultimap(java.util.Map, com.google.common.base.Supplier)"><tt>Multimap</tt></a></td>
<td width="114"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#newListMultimap(java.util.Map, com.google.common.base.Supplier)"><tt>ListMultimap</tt></a></td>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#newSetMultimap(java.util.Map, com.google.common.base.Supplier)"><tt>SetMultimap</tt></a></td>
<td width="168"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#newSortedSetMultimap(java.util.Map, com.google.common.base.Supplier)"><tt>SortedSetMultimap</tt></a></td>
</tr>
</tbody>
</table>

自定义 Multimap 的方法允许你指定 Multimap 中的特定实现。但要注意的是：

- Multimap 假设对 Map 和 Supplier 产生的集合对象有完全所有权。这些自定义对象应避免手动更新，并且在提供给 Multimap 时应该是空的，此外还不应该使用软引用、弱引用或虚引用。
- 无法保证修改了 Multimap 以后，底层 Map 的内容是什么样的。
- 即使 Map 和 Supplier 产生的集合都是线程安全的，它们组成的 Multimap 也不能保证并发操作的线程安全性。并发读操作是工作正常的，但需要保证并发读写的话，请考虑用同步包装器解决。
- 只有当 Map、Supplier、Supplier 产生的集合对象、以及 Multimap 存放的键值类型都是可序列化的，Multimap 才是可序列化的。
- Multimap.get(key)返回的集合对象和 Supplier 返回的集合对象并不是同一类型。但如果 Supplier 返回的是随机访问集合，那么 Multimap.get(key)返回的集合也是可随机访问的。

请注意，用来自定义 Multimap 的方法需要一个 Supplier 参数，以创建崭新的集合。下面有个实现 ListMultimap 的例子——用 TreeMap 做映射，而每个键对应的多个值用 LinkedList 存储。

```


    ListMultimap<String, Integer> myMultimap = Multimaps.newListMultimap(
	    Maps.<String, Collection>newTreeMap(),
	    new Supplier<LinkedList>() {
	        public LinkedList get() {
	            return Lists.newLinkedList();
	        }
	    });

```

## Tables

[Tables](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html) 类提供了若干称手的工具方法。

### 自定义 Table

堪比 Multimaps.newXXXMultimap(Map, Supplier)工具方法，<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html#newCustomTable(java.util.Map, com.google.common.base.Supplier)"><tt>Tables.newCustomTable(Map, Supplier&lt;Map&gt;)</tt></a> 允许你指定 Table 用什么样的 map 实现行和列。

```

    // 使用LinkedHashMaps替代HashMaps
    Table<String, Character, Integer> table = Tables.newCustomTable(
    Maps.<String, Map<Character, Integer>>newLinkedHashMap(),
    new Supplier<Map<Character, Integer>> () {
    public Map<Character, Integer> get() {
    return Maps.newLinkedHashMap();
    }
    });

```

**transpose**

[transpose(Table<R, C, V>)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html#transpose(com.google.common.collect.Table))方法允许你把 Table<C, R, V>转置成 Table<R, C, V>。例如，如果你在用 Table 构建加权有向图，这个方法就可以把有向图反转。

### 包装器

还有很多你熟悉和喜欢的 Table 包装类。然而，在大多数情况下还请使用 [ImmutableTable](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableTable.html)

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="108">Unmodifiable</td>
<td width="90"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html#unmodifiableTable(com.google.common.collect.Table)"><tt>Table</tt></a></td>
<td width="150"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html#unmodifiableRowSortedTable(com.google.common.collect.RowSortedTable)"><tt>RowSortedTable</tt></a></td>
</tr>
</tbody>
</table>

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 2.3-强大的集合工具类：java.util.Collections 中未包含的集合工具](http://ifeve.com/google-guava-collectionutilities/)

