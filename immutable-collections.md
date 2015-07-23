# 2.1-不可变集合

## 范例

```


    public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
	        "red",
	        "orange",
	        "yellow",
	        "green",
	        "blue",
	        "purple");


    class Foo {
	    Set<Bar> bars;
	    Foo(Set<Bar> bars) {
	        this.bars = ImmutableSet.copyOf(bars); // defensive copy!
	    }
    }

```

## 为什么要使用不可变集合

不可变对象有很多优点，包括：

- 当对象被不可信的库调用时，不可变形式是安全的；
- 不可变对象被多个线程调用时，不存在竞态条件问题
- 不可变集合不需要考虑变化，因此可以节省时间和空间。所有不可变的集合都比它们的可变形式有更好的内存利用率（分析和测试细节）；
- 不可变对象因为有固定不变，可以作为常量来安全使用。

创建对象的不可变拷贝是一项很好的防御性编程技巧。Guava 为所有 JDK 标准集合类型和 Guava 新集合类型都提供了简单易用的不可变版本。
 
JDK 也提供了 Collections.unmodifiableXXX 方法把集合包装为不可变形式，但我们认为不够好：

- 笨重而且累赘：不能舒适地用在所有想做防御性拷贝的场景；
- 不安全：要保证没人通过原集合的引用进行修改，返回的集合才是事实上不可变的；
- 低效：包装过的集合仍然保有可变集合的开销，比如并发修改的检查、散列表的额外空间，等等。

如果你没有修改某个集合的需求，或者希望某个集合保持不变时，把它防御性地拷贝到不可变集合是个很好的实践。

重要提示：*所有 Guava 不可变集合的实现都不接受 null 值。我们对 Google 内部的代码库做过详细研究，发现只有 5%的情况需要在集合中允许 null 元素，剩下的 95%场景都是遇到 null 值就快速失败。如果你需要在不可变集合中使用 null，请使用 JDK 中的 Collections.unmodifiableXXX 方法。更多细节建议请参考“使用和避免 null”。*

## 怎么使用不可变集合

不可变集合可以用如下多种方式创建：

- copyOf 方法，如 ImmutableSet.copyOf(set);
- of 方法，如 ImmutableSet.of(“a”, “b”, “c”)或 ImmutableMap.of(“a”, 1, “b”, 2);
- Builder 工具，如

```

    public static final ImmutableSet<Color> GOOGLE_COLORS =
	        ImmutableSet.<Color>builder()
	            .addAll(WEBSAFE_COLORS)
	            .add(new Color(0, 191, 255))
	            .build();

```

此外，对有序不可变集合来说，排序是在构造集合的时候完成的，如：

```

    ImmutableSortedSet.of("a", "b", "c", "a", "d", "b");

```

会在构造时就把元素排序为 a, b, c, d。

### 比想象中更智能的 copyOf

请注意，ImmutableXXX.copyOf 方法会尝试在安全的时候避免做拷贝——实际的实现细节不详，但通常来说是很智能的，比如：

```

    ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
    thingamajig(foobar);
    
    void thingamajig(Collection<String> collection) {
        ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
        ...
    }

```

在这段代码中，ImmutableList.copyOf(foobar)会智能地直接返回 foobar.asList(),它是一个 ImmutableSet 的常量时间复杂度的 List 视图。

作为一种探索，ImmutableXXX.copyOf(ImmutableCollection)会试图对如下情况避免线性时间拷贝：

- 在常量时间内使用底层数据结构是可能的——例如，ImmutableSet.copyOf(ImmutableList)就不能在常量时间内完成。
- 不会造成内存泄露——例如，你有个很大的不可变集合 ImmutableList<String>
hugeList， ImmutableList.copyOf(hugeList.subList(0, 10))就会显式地拷贝，以免不必要地持有 hugeList 的引用。
- 不改变语义——所以 ImmutableSet.copyOf(myImmutableSortedSet)会显式地拷贝，因为和基于比较器的 ImmutableSortedSet 相比，ImmutableSet对hashCode()和 equals 有不同语义。

在可能的情况下避免线性拷贝，可以最大限度地减少防御性编程风格所带来的性能开销。

### asList视图

所有不可变集合都有一个 asList()方法提供 ImmutableList 视图，来帮助你用列表形式方便地读取集合元素。例如，你可以使用 sortedSet.asList().get(k)从 ImmutableSortedSet 中读取第 k 个最小元素。

asList()返回的 ImmutableList 通常是——并不总是——开销稳定的视图实现，而不是简单地把元素拷贝进 List。也就是说，asList 返回的列表视图通常比一般的列表平均性能更好，比如，在底层集合支持的情况下，它总是使用高效的 contains 方法。

## 细节：关联可变集合和不可变集合

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="184"><b>可变集合接口</b><b></b></td>
<td valign="top" width="158"><b>属于</b><b>JDK</b><b>还是</b><b>Guava</b><b></b></td>
<td width="277"><b>不可变版本</b><b></b></td>
</tr>
<tr>
<td width="184">Collection</td>
<td width="158">JDK</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableCollection.html"><tt>ImmutableCollection</tt></a></td>
</tr>
<tr>
<td width="184">List</td>
<td width="158">JDK</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableList.html"><tt>ImmutableList</tt></a></td>
</tr>
<tr>
<td width="184">Set</td>
<td width="158">JDK</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSet.html"><tt>ImmutableSet</tt></a></td>
</tr>
<tr>
<td width="184">SortedSet/NavigableSet</td>
<td width="158">JDK</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSortedSet.html"><tt>ImmutableSortedSet</tt></a></td>
</tr>
<tr>
<td width="184">Map</td>
<td width="158">JDK</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableMap.html"><tt>ImmutableMap</tt></a></td>
</tr>
<tr>
<td width="184">SortedMap</td>
<td width="158">JDK</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSortedMap.html"><tt>ImmutableSortedMap</tt></a></td>
</tr>
<tr>
<td width="184"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multiset">Multiset</a></td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableMultiset.html"><tt>ImmutableMultiset</tt></a></td>
</tr>
<tr>
<td width="184">SortedMultiset</td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/ImmutableSortedMultiset.html"><tt>ImmutableSortedMultiset</tt></a></td>
</tr>
<tr>
<td width="184"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Multimap">Multimap</a></td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableMultimap.html"><tt>ImmutableMultimap</tt></a></td>
</tr>
<tr>
<td width="184">ListMultimap</td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableListMultimap.html"><tt>ImmutableListMultimap</tt></a></td>
</tr>
<tr>
<td width="184">SetMultimap</td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableSetMultimap.html"><tt>ImmutableSetMultimap</tt></a></td>
</tr>
<tr>
<td width="184"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#BiMap">BiMap</a></td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableBiMap.html"><tt>ImmutableBiMap</tt></a></td>
</tr>
<tr>
<td width="184"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#ClassToInstanceMap">ClassToInstanceMap</a></td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableClassToInstanceMap.html"><tt>ImmutableClassToInstanceMap</tt></a></td>
</tr>
<tr>
<td width="184"><a href="http://code.google.com/p/guava-libraries/wiki/NewCollectionTypesExplained#Table">Table</a></td>
<td width="158">Guava</td>
<td width="277"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableTable.html"><tt>ImmutableTable</tt></a></td>
</tr>
</tbody>
</table>

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 2.1-不可变集合](http://ifeve.com/google-guava-immutablecollections/)
