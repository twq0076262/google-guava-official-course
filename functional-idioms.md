# 函数式编程

## 注意事项

截至 JDK7，Java 中也只能通过笨拙冗长的匿名类来达到近似函数式编程的效果。预计 JDK8 中会有所改变，但 Guava 现在就想给 JDK5 以上用户提供这类支持。

过度使用 Guava 函数式编程会导致冗长、混乱、可读性差而且低效的代码。这是迄今为止最容易（也是最经常）被滥用的部分，如果你想通过函数式风格达成一行代码，致使这行代码长到荒唐，Guava 团队会泪流满面。

比较如下代码：

```

    Function<String, Integer> lengthFunction = new Function<String, Integer>() {
        public Integer apply(String string) {
        return string.length();
    }
    };
    Predicate<String> allCaps = new Predicate<String>() {
        public boolean apply(String string) {
        return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
    }
    };
    Multiset<Integer> lengths = HashMultiset.create(
         Iterables.transform(Iterables.filter(strings, allCaps), lengthFunction));

```

或 FluentIterable 的版本

```

    Multiset<Integer> lengths = HashMultiset.create(
	    FluentIterable.from(strings)
	        .filter(new Predicate<String>() {
	            public boolean apply(String string) {
	                return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
	            }
	        })
	        .transform(new Function<String, Integer>() {
	            public Integer apply(String string) {
	                return string.length();
	            }
	        }));

```

还有

```

    Multiset<Integer> lengths = HashMultiset.create();
    for (String string : strings) {
	    if (CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string)) {
	        lengths.add(string.length());
	    }
    }

```

即使用了静态导入，甚至把 Function 和 Predicate 的声明放到别的文件，第一种代码实现仍然不简洁，可读性差并且效率较低。

截至 JDK7，命令式代码仍应是默认和第一选择。不应该随便使用函数式风格，除非你绝对确定以下两点之一：

- 使用函数式风格以后，整个工程的代码行会净减少。在上面的例子中，函数式版本用了 11 行， 命令式代码用了 6 行，把函数的定义放到另一个文件或常量中，并不能帮助减少总代码行。
- 为了提高效率，转换集合的结果需要懒视图，而不是明确计算过的集合。此外，确保你已经阅读和重读了 Effective Java 的第 55 条，并且除了阅读本章后面的说明，你还真正做了性能测试并且有测试数据来证明函数式版本更快。


请务必确保，当使用 Guava 函数式的时候，用传统的命令式做同样的事情不会更具可读性。尝试把代码写下来，看看它是不是真的那么糟糕？会不会比你想尝试的极其笨拙的函数式 更具可读性。

## Functions[函数]和 Predicates[断言]

本节只讨论直接与 Function 和 Predicate 打交道的 Guava 功能。一些其他工具类也和”函数式风格”相关，例如 [Iterables.concat(Iterable<Iterable>)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#concat(java.lang.Iterable))，和其他用常量时间返回视图的方法。尝试看看 [2.3 节的集合工具类](http://ifeve.com/google-guava-collectionutilities)。

Guava 提供两个基本的函数式接口：

- Function<A, B>，它声明了单个方法 B apply(A input)。Function 对象通常被预期为引用透明的——没有副作用——并且引用透明性中的”相等”语义与 equals 一致，如 a.equals(b)意味着 function.apply(a).equals(function.apply(b))。
- Predicate<T>，它声明了单个方法 boolean apply(T input)。Predicate 对象通常也被预期为无副作用函数，并且”相等”语义与 equals 一致。

### 特殊的断言

字符类型有自己特定版本的 Predicate——[CharMatcher](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html)，它通常更高效，并且在某些需求方面更有用。CharMatcher 实现了 Predicate<Character>，可以当作 Predicate 一样使用，要把 Predicate 转成 CharMatcher，可以使用 [CharMatcher.forPredicate](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#forPredicate(com.google.common.base.Predicate))。更多细节请参考第 6 章-字符串处理。

此外，对可比较类型和基于比较逻辑的 Predicate，Range 类可以满足大多数需求——它表示一个不可变区间。Range 类实现了 Predicate，用以判断值是否在区间内。例如，Range.atMost(2)就是个完全合法的 Predicate<Integer>。更多使用 Range 的细节请参照第8章。

### 操作 Functions 和 Predicates

[Functions](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Functions.html) 提供简便的 Function 构造和操作方法，包括：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Functions.html#forMap(java.util.Map)"><tt>forMap(Map&lt;A, B&gt;)</tt></a></td>
<td valign="top" width="330"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Functions.html#compose(com.google.common.base.Function, com.google.common.base.Function)"><tt>compose(Function&lt;B, C&gt;, Function&lt;A, B&gt;)</tt></a></td>
<td width="126"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Functions.html#constant(E)"><tt>constant(T)</tt></a></td>
</tr>
<tr>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Functions.html#identity()"><tt>identity()</tt></a></td>
<td valign="top" width="330"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Functions.html#toStringFunction()"><tt>toStringFunction()</tt></a></td>
<td width="126"></td>
</tr>
</tbody>
</table>

细节请参考 Javadoc。

相应地，[Predicates](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Predicates.html) 提供了更多构造和处理 Predicate 的方法，下面是一些例子：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="150"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#instanceOf(java.lang.Class)"><tt>instanceOf(Class)</tt></a></td>
<td valign="top" width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#assignableFrom(java.lang.Class)"><tt>assignableFrom(Class)</tt></a></td>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#contains(java.util.regex.Pattern)"><tt>contains(Pattern)</tt></a></td>
</tr>
<tr>
<td width="150"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#in(java.util.Collection)"><tt>in(Collection)</tt></a></td>
<td valign="top" width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#isNull()"><tt>isNull()</tt></a></td>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#alwaysFalse()"><tt>alwaysFalse()</tt></a></td>
</tr>
<tr>
<td width="150"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#alwaysTrue()"><tt>alwaysTrue()</tt></a></td>
<td valign="top" width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#equalTo(T)"><tt>equalTo(Object)</tt></a></td>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#compose(com.google.common.base.Predicate, com.google.common.base.Function)"><tt>compose(Predicate, Function)</tt></a></td>
</tr>
<tr>
<td width="150"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#and(com.google.common.base.Predicate...)"><tt>and(Predicate...)</tt></a></td>
<td valign="top" width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#or(com.google.common.base.Predicate...)"><tt>or(Predicate...)</tt></a></td>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Predicates.html#not(com.google.common.base.Predicate)"><tt>not(Predicate)</tt></a></td>
</tr>
</tbody>
</table>

细节请参考Javadoc。

## 使用函数式编程

Guava 提供了很多工具方法，以便用 Function 或 Predicate 操作集合。这些方法通常可以在集合工具类找到，如 Iterables，Lists，Sets，Maps，Multimaps 等。

### 断言

断言的最基本应用就是过滤集合。所有 Guava 过滤方法都返回”视图”——*译者注：即并非用一个新的集合表示过滤，而只是基于原集合的视图。*

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="150"><b>集合类型</b><b></b></td>
<td width="462"><b>过滤方法</b><b></b></td>
</tr>
<tr>
<td width="150">Iterable</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#filter(java.lang.Iterable, com.google.common.base.Predicate)"><tt>Iterables.filter(Iterable, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#filter(com.google.common.base.Predicate)"><tt>FluentIterable.filter(Predicate)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Iterator</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#filter(java.util.Iterator, com.google.common.base.Predicate)"><tt>Iterators.filter(Iterator, Predicate)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Collection</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Collections2.html#filter(java.util.Collection, com.google.common.base.Predicate)"><tt>Collections2.filter(Collection, Predicate)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Set</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#filter(java.util.Set, com.google.common.base.Predicate)"><tt>Sets.filter(Set, Predicate)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">SortedSet</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.html#filter(java.util.SortedSet, com.google.common.base.Predicate)"><tt>Sets.filter(SortedSet, Predicate)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Map</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#filterKeys(java.util.Map, com.google.common.base.Predicate)"><tt>Maps.filterKeys(Map, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#filterValues(java.util.Map, com.google.common.base.Predicate)"><tt>Maps.filterValues(Map, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#filterEntries(java.util.Map, com.google.common.base.Predicate)"><tt>Maps.filterEntries(Map, Predicate)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">SortedMap</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#filterKeys(java.util.SortedMap, com.google.common.base.Predicate)"><tt>Maps.filterKeys(SortedMap, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#filterValues(java.util.SortedMap, com.google.common.base.Predicate)"><tt>Maps.filterValues(SortedMap, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#filterEntries(java.util.SortedMap, com.google.common.base.Predicate)"><tt>Maps.filterEntries(SortedMap, Predicate)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Multimap</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#filterKeys(com.google.common.collect.Multimap, com.google.common.base.Predicate)"><tt>Multimaps.filterKeys(Multimap, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#filterValues(com.google.common.collect.Multimap, com.google.common.base.Predicate)"><tt>Multimaps.filterValues(Multimap, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#filterEntries(com.google.common.collect.Multimap, com.google.common.base.Predicate)"><tt>Multimaps.filterEntries(Multimap, Predicate)</tt></a><b></b></td>
</tr>
</tbody>
</table>

*List 的过滤视图被省略了，因为不能有效地支持类似 get(int)的操作。请改用 Lists.newArrayList(Collections2.filter(list, predicate))做拷贝过滤。

除了简单过滤，Guava 另外提供了若干用 Predicate 处理 Iterable 的工具——通常在 [Iterables](http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/Iterables.html)  工具类中，或者是 [FluentIterable](http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html) 的”fluent”（链式调用）方法。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="180"><b>Iterables</b><b>方法签名</b><b></b></td>
<td width="150"><b>说明</b><b></b></td>
<td width="282"><b>另请参见</b><b></b></td>
</tr>
<tr>
<td width="180"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#all(java.lang.Iterable, com.google.common.base.Predicate)"><tt>boolean all(Iterable, Predicate)</tt></a></td>
<td width="150">是否所有元素满足断言？懒实现：如果发现有元素不满足，不会继续迭代</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#all(java.util.Iterator, com.google.common.base.Predicate)"><tt>Iterators.all(Iterator, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#allMatch(com.google.common.base.Predicate)"><tt>FluentIterable.allMatch(Predicate)</tt></a></td>
</tr>
<tr>
<td width="180"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#any(java.lang.Iterable, com.google.common.base.Predicate)"><tt>boolean any(Iterable, Predicate)</tt></a></td>
<td width="150">是否有任意元素满足元素满足断言？懒实现：只会迭代到发现满足的元素</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#any(java.util.Iterator, com.google.common.base.Predicate)"><tt>Iterators.any(Iterator, Predicate)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#anyMatch(com.google.common.base.Predicate)"><tt>FluentIterable.anyMatch(Predicate)</tt></a></td>
</tr>
<tr>
<td width="180"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#find(java.lang.Iterable, com.google.common.base.Predicate)"><tt>T find(Iterable, Predicate)</tt></a></td>
<td width="150">循环并返回<b>一个</b>满足元素满足断言的元素，如果没有则抛出 NoSuchElementException</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#find(java.util.Iterator, com.google.common.base.Predicate)"><tt>Iterators.find(Iterator, Predicate)</tt></a><br />
<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#find(java.lang.Iterable, com.google.common.base.Predicate, T)"><tt>Iterables.find(Iterable, Predicate, T default)</tt></a><br />
<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#find(java.util.Iterator, com.google.common.base.Predicate, T)"><tt>Iterators.find(Iterator, Predicate, T default)</tt></a></td>
</tr>
<tr>
<td width="180"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#tryFind(java.lang.Iterable, com.google.common.base.Predicate)"><tt>Optional&lt;T&gt; tryFind(Iterable, Predicate)</tt></a></td>
<td width="150">返回<b>一个</b>满足元素满足断言的元素，若没有则返回<tt>O</tt>ptional.absent()</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#find(java.util.Iterator, com.google.common.base.Predicate)"><tt>Iterators.find(Iterator, Predicate)</tt></a><br />
<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#find(java.lang.Iterable, com.google.common.base.Predicate, T)"><tt>Iterables.find(Iterable, Predicate, T default)</tt></a><br />
<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#find(java.util.Iterator, com.google.common.base.Predicate, T)"><tt>Iterators.find(Iterator, Predicate, T default)</tt></a></td>
</tr>
<tr>
<td width="180"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#indexOf(java.lang.Iterable, com.google.common.base.Predicate)"><tt>indexOf(Iterable, Predicate)</tt></a></td>
<td width="150">返回第一个满足元素满足断言的元素索引值，若没有返回-1</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#indexOf(java.util.Iterator, com.google.common.base.Predicate)"><tt>Iterators.indexOf(Iterator, Predicate)</tt></a></td>
</tr>
<tr>
<td width="180"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#removeIf(java.lang.Iterable, com.google.common.base.Predicate)"><tt>removeIf(Iterable, Predicate)</tt></a></td>
<td width="150">移除所有满足元素满足断言的元素，实际调用Iterator.remove()方法</td>
<td width="282"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#removeIf(java.util.Iterator, com.google.common.base.Predicate)"><tt>Iterators.removeIf(Iterator, Predicate)</tt></a></td>
</tr>
</tbody>
</table>

### 函数

到目前为止，函数最常见的用途为转换集合。同样，所有的 Guava 转换方法也返回原集合的视图。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="150"><b>集合类型</b><b></b></td>
<td width="462"><b>转换</b><b>方法</b><b></b></td>
</tr>
<tr>
<td width="150">Iterable</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#transform(java.lang.Iterable, com.google.common.base.Function)"><tt>Iterables.transform(Iterable, Function)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/collect/FluentIterable.html#transform(com.google.common.base.Function)"><tt>FluentIterable.transform(Function)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Iterator</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterators.html#transform(java.util.Iterator, com.google.common.base.Function)"><tt>Iterators.transform(Iterator, Function)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Collection</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Collections2.html#transform(java.util.Collection, com.google.common.base.Function)"><tt>Collections2.transform(Collection, Function)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">List</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#transform(java.util.List, com.google.common.base.Function)"><tt>Lists.transform(List, Function)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Map*</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#transformValues(java.util.Map, com.google.common.base.Function)"><tt>Maps.transformValues(Map, Function)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#transformEntries(java.util.Map, com.google.common.collect.Maps.EntryTransformer)"><tt>Maps.transformEntries(Map, EntryTransformer)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">SortedMap*</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#transformValues(java.util.SortedMap, com.google.common.base.Function)"><tt>Maps.transformValues(SortedMap, Function)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.html#transformEntries(java.util.SortedMap, com.google.common.collect.Maps.EntryTransformer)"><tt>Maps.transformEntries(SortedMap, EntryTransformer)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Multimap*</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#transformValues(com.google.common.collect.Multimap, com.google.common.base.Function)"><tt>Multimaps.transformValues(Multimap, Function)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#transformEntries(com.google.common.collect.Multimap, com.google.common.collect.Maps.EntryTransformer)"><tt>Multimaps.transformEntries(Multimap, EntryTransformer)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">ListMultimap*</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#transformValues(com.google.common.collect.ListMultimap, com.google.common.base.Function)"><tt>Multimaps.transformValues(ListMultimap, Function)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimaps.html#transformEntries(com.google.common.collect.ListMultimap, com.google.common.collect.Maps.EntryTransformer)"><tt>Multimaps.transformEntries(ListMultimap, EntryTransformer)</tt></a><b></b></td>
</tr>
<tr>
<td width="150">Table</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Tables.html#transformValues(com.google.common.collect.Table, com.google.common.base.Function)"><tt>Tables.transformValues(Table, Function)</tt></a><b></b></td>
</tr>
</tbody>
</table>

*Map 和 [Multimap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimap.html) 有特殊的方法，其中有个   [EntryTransformer<K, V1, V2>](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Maps.EntryTransformer.html)参数，它可以使用旧的键值来计算，并且用计算结果替换旧值。

*对 Set 的转换操作被省略了，因为不能有效支持 contains(Object)操作——*译者注：懒视图实际上不会全部计算转换后的 Set 元素，因此不能高效地支持 contains(Object)。*请改用 Sets.newHashSet(Collections2.transform(set, function))进行拷贝转换。

```

    List<String> names;
    Map<String, Person> personWithName;
    List<Person> people = Lists.transform(names, Functions.forMap(personWithName));
    
    ListMultimap<String, String> firstNameToLastNames;
    // maps first names to all last names of people with that first name
    
    ListMultimap<String, String> firstNameToName = Multimaps.transformEntries(firstNameToLastNames,
	    new EntryTransformer<String, String, String> () {
	        public String transformEntry(String firstName, String lastName) {
	            return firstName + " " + lastName;
	        }
	    });

```

可以组合 Function 使用的类包括：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="150">Ordering</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#onResultOf(com.google.common.base.Function)"><tt>Ordering.onResultOf(Function)</tt></a></td>
</tr>
<tr>
<td width="150">Predicate</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Predicates.html#compose(com.google.common.base.Predicate, com.google.common.base.Function)"><tt>Predicates.compose(Predicate, Function)</tt></a></td>
</tr>
<tr>
<td width="150">Equivalence</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Equivalence.html#onResultOf(com.google.common.base.Function)"><tt>Equivalence.onResultOf(Function)</tt></a></td>
</tr>
<tr>
<td width="150">Supplier</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Suppliers.html#compose(com.google.common.base.Function, com.google.common.base.Supplier)"><tt>Suppliers.compose(Function, Supplier)</tt></a></td>
</tr>
<tr>
<td width="150">Function</td>
<td width="462"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Functions.html#compose(com.google.common.base.Function, com.google.common.base.Function)"><tt>Functions.compose(Function, Function)</tt></a></td>
</tr>
</tbody>
</table>

此外，[ListenableFuture](http://code.google.com/p/guava-libraries/wiki/ListenableFutureExplained) API 支持转换 ListenableFuture。Futures 也提供了接受 [AsyncFunction](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/AsyncFunction.html) 参数的方法。

AsyncFunction 是 Function 的变种，它允许异步计算值。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="612"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function)"><tt>Futures.transform(ListenableFuture, Function)</tt></a></td>
</tr>
<tr>
<td width="612"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function, java.util.concurrent.Executor)"><tt>Futures.transform(ListenableFuture, Function, Executor)</tt></a></td>
</tr>
<tr>
<td width="612"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction)"><tt>Futures.transform(ListenableFuture, AsyncFunction)</tt></a></td>
</tr>
<tr>
<td width="612"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction, java.util.concurrent.Executor)"><tt>Futures.transform(ListenableFuture, AsyncFunction, Executor)</tt></a></td>
</tr>
</tbody>
</table>

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 4-函数式编程](http://ifeve.com/google-guava-functional/)