# 2.2-新集合类型

Guava 引入了很多 JDK 没有的、但我们发现明显有用的新集合类型。这些新类型是为了和 JDK 集合框架共存，而没有往 JDK 集合抽象中硬塞其他概念。作为一般规则，Guava 集合非常精准地遵循了 JDK 接口契约。

## Multiset

统计一个词在文档中出现了多少次，传统的做法是这样的：

```

    Map<String, Integer> counts = new HashMap<String, Integer>();
    for (String word : words) {
	    Integer count = counts.get(word);
	    if (count == null) {
	        counts.put(word, 1);
	    } else {
	        counts.put(word, count + 1);
	    }
    }

```

这种写法很笨拙，也容易出错，并且不支持同时收集多种统计信息，如总词数。我们可以做的更好。

Guava 提供了一个新集合类型 [Multiset](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multiset.html)，它可以多次添加相等的元素。维基百科从数学角度这样定义 Multiset：”集合[set]概念的延伸，它的元素可以重复出现…与集合[set]相同而与元组[tuple]相反的是，Multiset 元素的顺序是无关紧要的：Multiset {a, a, b}和{a, b, a}是相等的”。——*译者注：这里所说的集合[set]是数学上的概念，Multiset继承自 JDK 中的 Collection 接口，而不是 Set 接口，所以包含重复元素并没有违反原有的接口契约。*

可以用两种方式看待 Multiset：

- 没有元素顺序限制的 ArrayList<E>
- Map<E, Integer>，键为元素，值为计数

Guava 的 Multiset API 也结合考虑了这两种方式：  
当把 Multiset 看成普通的 Collection 时，它表现得就像无序的 ArrayList：

- add(E)添加单个给定元素
- iterator()返回一个迭代器，包含 Multiset 的所有元素（包括重复的元素）
- size()返回所有元素的总个数（包括重复的元素）

当把 Multiset 看作 Map<E, Integer>时，它也提供了符合性能期望的查询操作：

- count(Object)返回给定元素的计数。HashMultiset.count 的复杂度为 O(1)，TreeMultiset.count 的复杂度为 O(log n)。
- entrySet()返回 Set<Multiset.Entry<E>>，和 Map 的 entrySet 类似。
- elementSet()返回所有不重复元素的 Set<E>，和 Map 的 keySet()类似。
- 所有 Multiset 实现的内存消耗随着不重复元素的个数线性增长。

值得注意的是，除了极少数情况，Multiset 和 JDK 中原有的 Collection 接口契约完全一致——具体来说，TreeMultiset 在判断元素是否相等时，与 TreeSet 一样用 compare，而不是 Object.equals。另外特别注意，Multiset.addAll(Collection)可以添加 Collection 中的所有元素并进行计数，这比用 for 循环往 Map 添加元素和计数方便多了。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="142"><b>方法</b><b></b></td>
<td width="470"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="142"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multiset.html#count(java.lang.Object)">count(E)</a><b></b></td>
<td width="470">给定元素在 Multiset 中的计数</td>
</tr>
<tr>
<td width="142"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multiset.html#elementSet()">elementSet()</a></td>
<td width="470">Multiset 中不重复元素的集合，类型为 Set&lt;E&gt;</td>
</tr>
<tr>
<td width="142"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multiset.html#entrySet()">entrySet()</a></td>
<td width="470">和 Map 的 entrySet 类似，返回 Set&lt;Multiset.Entry&lt;E&gt;&gt;，其中包含的 Entry 支持 getElement()和 getCount()方法</td>
</tr>
<tr>
<td width="142"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multiset.html#add(java.lang.Object,int)">add(E, int)</a></td>
<td width="470">增加给定元素在 Multiset 中的计数</td>
</tr>
<tr>
<td width="142"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multiset.html#remove(java.lang.Object, int)">remove(E, int)</a></td>
<td width="470">减少给定元素在 Multiset 中的计数</td>
</tr>
<tr>
<td width="142"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multiset.html#setCount(E, int)">setCount(E, int)</a></td>
<td width="470">设置给定元素在 Multiset 中的计数，不可以为负数</td>
</tr>
<tr>
<td width="142">size()</td>
<td width="470">返回集合元素的总个数（包括重复的元素）</td>
</tr>
</tbody>
</table>

### Multiset 不是 Map

请注意，Multiset<E>不是 Map<E, Integer>，虽然 Map 可能是某些 Multiset 实现的一部分。准确来说 Multiset 是一种 Collection 类型，并履行了 Collection 接口相关的契约。关于 Multiset 和 Map 的显著区别还包括：

- Multiset 中的元素计数只能是正数。任何元素的计数都不能为负，也不能是 0。elementSet()和 entrySet()视图中也不会有这样的元素。
- multiset.size()返回集合的大小，等同于所有元素计数的总和。对于不重复元素的个数，应使用 elementSet().size()方法。（因此，add(E)把 multiset.size()增加 1）
- multiset.iterator()会迭代重复元素，因此迭代长度等于 multiset.size()。
- Multiset 支持直接增加、减少或设置元素的计数。setCount(elem, 0)等同于移除所有 elem。
- 对 multiset 中没有的元素，multiset.count(elem)始终返回 0。

### Multiset 的各种实现

Guava 提供了多种 Multiset 的实现，大致对应 JDK 中 Map 的各种实现：

<table width="614" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="161"><b>Map</b></td>
<td width="198"><b>对应的</b><b>Multiset</b><b></b></td>
<td valign="top" width="255"><b>是否支持</b><b>null</b><b>元素</b><b></b></td>
</tr>
<tr>
<td width="161">HashMap</td>
<td width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/HashMultiset.html">HashMultiset</a></td>
<td valign="top" width="255">是</td>
</tr>
<tr>
<td width="161">TreeMap</td>
<td width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/TreeMultiset.html">TreeMultiset</a></td>
<td valign="top" width="255">是（如果 comparator 支持的话）</td>
</tr>
<tr>
<td width="161">LinkedHashMap</td>
<td width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/LinkedHashMultiset.html">LinkedHashMultiset</a></td>
<td valign="top" width="255">是</td>
</tr>
<tr>
<td width="161">ConcurrentHashMap</td>
<td width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ConcurrentHashMultiset.html">ConcurrentHashMultiset</a></td>
<td valign="top" width="255">否</td>
</tr>
<tr>
<td width="161">ImmutableMap</td>
<td width="198"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ImmutableMultiset.html">ImmutableMultiset</a></td>
<td valign="top" width="255">否</td>
</tr>
</tbody>
</table>

### SortedMultiset

[SortedMultiset](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/SortedMultiset.html) 是 Multiset 接口的变种，它支持高效地获取指定范围的子集。比方说，你可以用 latencies.subMultiset(0,BoundType.CLOSED, 100, BoundType.OPEN).size()来统计你的站点中延迟在 100 毫秒以内的访问，然后把这个值和 latencies.size()相比，以获取这个延迟水平在总体访问中的比例。

TreeMultiset 实现 SortedMultiset 接口。在撰写本文档时，ImmutableSortedMultiset 还在测试和 GWT 的兼容性。

## Multimap

每个有经验的 Java 程序员都在某处实现过 Map<K, List<V>>或 Map<K, Set<V>>，并且要忍受这个结构的笨拙。例如，Map<K, Set<V>>通常用来表示非标定有向图。Guava 的 [Multimap](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimap.html) 可以很容易地把一个键映射到多个值。换句话说，Multimap 是把键映射到任意多个值的一般方式。

可以用两种方式思考 Multimap 的概念：”键-单个值映射”的集合：

a -> 1 a -> 2 a ->4 b -> 3 c -> 5

或者”键-值集合映射”的映射：

a -> [1, 2, 4] b -> 3 c -> 5

一般来说，Multimap 接口应该用第一种方式看待，但 asMap()视图返回 Map<K, Collection<V>>，让你可以按另一种方式看待 Multimap。重要的是，不会有任何键映射到空集合：一个键要么至少到一个值，要么根本就不在 Multimap 中。

很少会直接使用 Multimap 接口，更多时候你会用 ListMultimap 或 SetMultimap 接口，它们分别把键映射到 List 或 Set。

### 修改 Multimap

[Multimap.get(key)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimap.html#get(K))以集合形式返回键所对应的值视图，即使没有任何对应的值，也会返回空集合。ListMultimap.get(key)返回 List，SetMultimap.get(key)返回 Set。

对值视图集合进行的修改最终都会反映到底层的 Multimap。例如：

```

    Set<Person> aliceChildren = childrenMultimap.get(alice);
    aliceChildren.clear();
    aliceChildren.add(bob);
    aliceChildren.add(carol);

```

其他（更直接地）修改 Multimap 的方法有：

<table width="614" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="132"><b>方法签名</b><b></b></td>
<td width="246"><b>描述</b><b></b></td>
<td valign="top" width="236"><b>等价于</b><b></b></td>
</tr>
<tr>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimap.html#put(K, V)">put(K, V)</a></td>
<td width="246">添加键到单个值的映射</td>
<td valign="top" width="236">multimap.get(key).add(value)</td>
</tr>
<tr>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimap.html#putAll(K, java.lang.Iterable)">putAll(K, Iterable&lt;V&gt;)</a></td>
<td width="246">依次添加键到多个值的映射</td>
<td valign="top" width="236">Iterables.addAll(multimap.get(key), values)</td>
</tr>
<tr>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimap.html#remove(java.lang.Object, java.lang.Object)">remove(K, V)</a></td>
<td width="246">移除键到值的映射；如果有这样的键值并成功移除，返回 true。</td>
<td valign="top" width="236">multimap.get(key).remove(value)</td>
</tr>
<tr>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimap.html#removeAll(java.lang.Object)">removeAll(K)</a></td>
<td width="246">清除键对应的所有值，返回的集合包含所有之前映射到 K 的值，但修改这个集合就不会影响  Multimap 了。</td>
<td valign="top" width="236">multimap.get(key).clear()</td>
</tr>
<tr>
<td width="132"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimap.html#replaceValues(K, java.lang.Iterable)">replaceValues(K, Iterable&lt;V&gt;)</a></td>
<td width="246">清除键对应的所有值，并重新把 key 关联到 Iterable 中的每个元素。返回的集合包含所有之前映射到 K 的值。</td>
<td valign="top" width="236">multimap.get(key).clear(); Iterables.addAll(multimap.get(key), values)</td>
</tr>
</tbody>
</table>

### Multimap 的视图

Multimap 还支持若干强大的视图：

- [asMap](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimap.html#asMap())为 Multimap<K, V>提供 Map<K,Collection<V>>形式的视图。返回的 Map 支持 remove 操作，并且会反映到底层的 Multimap，但它不支持 put 或 putAll 操作。更重要的是，如果你想为 Multimap 中没有的键返回 null，而不是一个新的、可写的空集合，你就可以使用 asMap().get(key)。（你可以并且应当把 asMap.get(key)返回的结果转化为适当的集合类型——如 SetMultimap.asMap.get(key)的结果转为 Set，ListMultimap.asMap.get(key)的结果转为 List——Java 类型系统不允许 ListMultimap 直接为 asMap.get(key)返回 List——*译者注：也可以用 Multimaps 中的 asMap 静态方法帮你完成类型转换）*
- [entries](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimap.html#entries())用 Collection<Map.Entry<K, V>>返回 Multimap 中所有”键-单个值映射”——包括重复键。（对 SetMultimap，返回的是 Set）
- [keySet](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimap.html#keySet())用 Set 表示 Multimap 中所有不同的键。
- [keys](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimap.html#keys())用 Multiset 表示 Multimap 中的所有键，每个键重复出现的次数等于它映射的值的个数。可以从这个 Multiset 中移除元素，但不能做添加操作；移除操作会反映到底层的 Multimap。
- [values()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Multimap.html#values())用一个”扁平”的Collection<V>包含 Multimap 中的所有值。这有一点类似于 Iterables.concat(multimap.asMap().values())，但它直接返回了单个 Collection，而不像 multimap.asMap().values()那样是按键区分开的 Collection。

### Multimap 不是 Map

Multimap<K, V>不是 Map<K,Collection<V>>，虽然某些 Multimap 实现中可能使用了 map。它们之间的显著区别包括：

- Multimap.get(key)总是返回非 null、但是可能空的集合。这并不意味着 Multimap 为相应的键花费内存创建了集合，而只是提供一个集合视图方便你为键增加映射值——*译者注：如果有这样的键，返回的集合只是包装了 Multimap 中已有的集合；如果没有这样的键，返回的空集合也只是持有 Multimap 引用的栈对象，让你可以用来操作底层的 Multimap。因此，返回的集合不会占据太多内存，数据实际上还是存放在 Multimap 中。*
- 如果你更喜欢像 Map 那样，为 Multimap 中没有的键返回 null，请使用 asMap()视图获取一个 Map<K, Collection<V>>。（或者用静态方法 [Multimaps.asMap()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimaps.html#asMap%28com.google.common.collect.ListMultimap%29)为 ListMultimap 返回一个 Map<K, List<V>>。对于 SetMultimap 和 SortedSetMultimap，也有类似的静态方法存在）
- 当且仅当有值映射到键时，Multimap.containsKey(key)才会返回 true。尤其需要注意的是，如果键 k 之前映射过一个或多个值，但它们都被移除后，Multimap.containsKey(key)会返回 false。
- Multimap.entries()返回 Multimap 中所有”键-单个值映射”——包括重复键。如果你想要得到所有”键-值集合映射”，请使用 asMap().entrySet()。
- Multimap.size()返回所有”键-单个值映射”的个数，而非不同键的个数。要得到不同键的个数，请改用 Multimap.keySet().size()。

### Multimap 的各种实现

Multimap 提供了多种形式的实现。在大多数要使用 Map<K, Collection<V>>的地方，你都可以使用它们：

<table width="614" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="189"><b>实现</b><b></b></td>
<td width="198"><b>键行为类似</b><b></b></td>
<td valign="top" width="227"><b>值行为类似</b><b></b></td>
</tr>
<tr>
<td width="189"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ArrayListMultimap.html">ArrayListMultimap</a><b></b></td>
<td width="198">HashMap</td>
<td width="227">ArrayList</td>
</tr>
<tr>
<td width="189"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/HashMultimap.html">HashMultimap</a><b></b></td>
<td width="198">HashMap</td>
<td width="227">HashSet</td>
</tr>
<tr>
<td width="189"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/LinkedListMultimap.html">LinkedListMultimap</a>*<b></b></td>
<td width="198">LinkedHashMap*</td>
<td width="227">LinkedList*</td>
</tr>
<tr>
<td width="189"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/LinkedHashMultimap.html">LinkedHashMultimap</a>**<b></b></td>
<td width="198">LinkedHashMap</td>
<td width="227">LinkedHashMap</td>
</tr>
<tr>
<td width="189"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/TreeMultimap.html">TreeMultimap</a><b></b></td>
<td width="198">TreeMap</td>
<td width="227">TreeSet</td>
</tr>
<tr>
<td width="189"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ImmutableListMultimap.html"><tt>ImmutableListMultimap</tt></a></td>
<td width="198">ImmutableMap</td>
<td width="227">ImmutableList</td>
</tr>
<tr>
<td width="189"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ImmutableSetMultimap.html">ImmutableSetMultimap</a><b></b></td>
<td width="198">ImmutableMap</td>
<td width="227">ImmutableSet</td>
</tr>
</tbody>
</table>

除了两个不可变形式的实现，其他所有实现都支持 null 键和 null 值

*LinkedListMultimap.entries()保留了所有键和值的迭代顺序。详情见 doc 链接。

**LinkedHashMultimap 保留了映射项的插入顺序，包括键插入的顺序，以及键映射的所有值的插入顺序。

请注意，并非所有的 Multimap 都和上面列出的一样，使用  Map<K, Collection<V>>来实现（特别是，一些 Multimap  实现用了自定义的 hashTable，以最小化开销）

如果你想要更大的定制化，请用 [Multimaps.newMultimap(Map, Supplier<Collection>)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimaps.html#newMultimap(java.util.Map,%20com.google.common.base.Supplier))或 [list](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimaps.html#newListMultimap) 和 [set](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Multimaps.html#newSetMultimap) 版本，使用自定义的 Collection、List 或 Set 实现 Multimap。

## BiMap

传统上，实现键值对的双向映射需要维护两个单独的 map，并保持它们间的同步。但这种方式很容易出错，而且对于值已经在 map 中的情况，会变得非常混乱。例如：

```

    Map<String, Integer> nameToId = Maps.newHashMap();
    Map<Integer, String> idToName = Maps.newHashMap();
    
    nameToId.put("Bob", 42);
    idToName.put(42, "Bob");
    //如果"Bob"和42已经在map中了，会发生什么?
    //如果我们忘了同步两个map，会有诡异的bug发生...

```

BiMap<K, V>是特殊的 Map：

- 可以用 inverse()反转 BiMap<K, V>的键值映射
- 保证值是唯一的，因此 values()返回 Set 而不是普通的 Collection

在 BiMap 中，如果你想把键映射到已经存在的值，会抛出 IllegalArgumentException 异常。如果对特定值，你想要强制替换它的键，请使用 BiMap.forcePut(key, value)。

```

    BiMap<String, Integer> userId = HashBiMap.create();
    ...
    
    String userForId = userId.inverse().get(id);

```

### BiMap 的各种实现

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="170"><b>键</b><b>&#8211;</b><b>值实现</b><b></b></td>
<td valign="top" width="180"><b>值</b><b>&#8211;</b><b>键实现</b><b></b></td>
<td width="265"><b>对应的</b><b>BiMap</b><b>实现</b><b></b></td>
</tr>
<tr>
<td width="170">HashMap</td>
<td width="180">HashMap</td>
<td width="265"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/HashBiMap.html">HashBiMap</a><b></b></td>
</tr>
<tr>
<td width="170">ImmutableMap</td>
<td width="180">ImmutableMap</td>
<td width="265"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ImmutableBiMap.html">ImmutableBiMap</a><b></b></td>
</tr>
<tr>
<td width="170">EnumMap</td>
<td width="180">EnumMap</td>
<td width="265"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/EnumBiMap.html">EnumBiMap</a></td>
</tr>
<tr>
<td width="170">EnumMap</td>
<td width="180">HashMap</td>
<td width="265"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/EnumHashBiMap.html">EnumHashBiMap</a></td>
</tr>
</tbody>
</table>

注：[Maps](https://code.google.com/p/guava-libraries/wiki/CollectionUtilitiesExplained#Maps) 类中还有一些诸如 synchronizedBiMap 的 BiMap 工具方法.

### Table

```

    Table<Vertex, Vertex, Double> weightedGraph = HashBasedTable.create();
    weightedGraph.put(v1, v2, 4);
    weightedGraph.put(v1, v3, 20);
    weightedGraph.put(v2, v3, 5);
    
    weightedGraph.row(v1); // returns a Map mapping v2 to 4, v3 to 20
    weightedGraph.column(v3); // returns a Map mapping v1 to 20, v2 to 5

```

通常来说，当你想使用多个键做索引的时候，你可能会用类似 Map<FirstName, Map<LastName, Person>>的实现，这种方式很丑陋，使用上也不友好。Guava 为此提供了新集合类型 Table，它有两个支持所有类型的键：”行”和”列”。Table 提供多种视图，以便你从各种角度使用它：

- [rowMap()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.html#rowMap())：用 Map<R, Map<C, V>>表现 Table<R, C, V>。同样的， [rowKeySet()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.html#rowKeySet())返回”行”的集合Set<R>。
- [row(r)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.html#row(R)) ：用 Map<C, V>返回给定”行”的所有列，对这个 map 进行的写操作也将写入 Table 中。
- 类似的列访问方法：[columnMap()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.html#columnMap())、[columnKeySet()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.html#columnKeySet())、[column(c)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.html#column(C))。（基于列的访问会比基于的行访问稍微低效点）
- [cellSet()](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.html#cellSet())：用元素类型为 [Table.Cell<R, C, V>](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/Table.Cell.html)的 Set 表现 Table<R, C, V>。Cell 类似于 Map.Entry，但它是用行和列两个键区分的。

Table 有如下几种实现：

- [HashBasedTable](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/HashBasedTable.html)：本质上用 HashMap<R, HashMap<C, V>>实现；
- [TreeBasedTable](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/TreeBasedTable.html)：本质上用 TreeMap<R, TreeMap<C,V>>实现；
- [ImmutableTable](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableTable.html)：本质上用 ImmutableMap<R, ImmutableMap<C, V>>实现；注：ImmutableTable 对稀疏或密集的数据集都有优化。
- [ArrayTable](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ArrayTable.html)：要求在构造时就指定行和列的大小，本质上由一个二维数组实现，以提升访问速度和密集 Table 的内存利用率。ArrayTable 与其他 Table 的工作原理有点不同，请参见 Javadoc 了解详情。

## ClassToInstanceMap

[ClassToInstanceMap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ClassToInstanceMap.html) 是一种特殊的 Map：它的键是类型，而值是符合键所指类型的对象。

为了扩展 Map 接口，ClassToInstanceMap 额外声明了两个方法：[T getInstance(Class<T>)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ClassToInstanceMap.html#getInstance) 和 [T putInstance(Class<T>, T)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ClassToInstanceMap.html#putInstance(java.lang.Class,java.lang.Object))，从而避免强制类型转换，同时保证了类型安全。

ClassToInstanceMap 有唯一的泛型参数，通常称为 B，代表 Map 支持的所有类型的上界。例如：

```

    ClassToInstanceMap<Number> numberDefaults=MutableClassToInstanceMap.create();
    numberDefaults.putInstance(Integer.class, Integer.valueOf(0));

```

从技术上讲，从技术上讲，ClassToInstanceMap&lt;B&gt 实现了 Map<Class<? extends B>, B>——或者换句话说，是一个映射 B 的子类型到对应实例的 Map。这让 ClassToInstanceMap 包含的泛型声明有点令人困惑，但请记住 B 始终是 Map 所支持类型的上界——通常 B 就是 Object。

对于 ClassToInstanceMap，Guava 提供了两种有用的实现：[MutableClassToInstanceMap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/MutableClassToInstanceMap.html) 和  [ImmutableClassToInstanceMap](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableClassToInstanceMap.html)。

## RangeSet

RangeSet描述了一组不相连的、非空的区间。当把一个区间添加到可变的RangeSet时，所有相连的区间会被合并，空区间会被忽略。例如：

```

    RangeSet<Integer> rangeSet = TreeRangeSet.create();
    rangeSet.add(Range.closed(1, 10)); // {[1,10]}
    rangeSet.add(Range.closedOpen(11, 15));//不相连区间:{[1,10], [11,15)}
    rangeSet.add(Range.closedOpen(15, 20)); //相连区间; {[1,10], [11,20)}
    rangeSet.add(Range.openClosed(0, 0)); //空区间; {[1,10], [11,20)}
    rangeSet.remove(Range.open(5, 10)); //分割[1, 10]; {[1,5], [10,10], [11,20)}

```

请注意，要合并 Range.closed(1, 10)和 Range.closedOpen(11, 15)这样的区间，你需要首先用 [Range.canonical(DiscreteDomain)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Range.html#canonical(com.google.common.collect.DiscreteDomain))对区间进行预处理，例如 DiscreteDomain.integers()。

注：RangeSet不支持  GWT，也不支持 JDK5 和更早版本；因为，RangeSet 需要充分利用 JDK6 中 NavigableMap 的特性。

### RangeSet 的视图

RangeSet 的实现支持非常广泛的视图：

- complement()：返回 RangeSet 的补集视图。complement 也是 RangeSet 类型,包含了不相连的、非空的区间。
- subRangeSet(Range<C>)：返回 RangeSet 与给定 Range 的交集视图。这扩展了传统排序集合中的 headSet、subSet 和 tailSet 操作。
- asRanges()：用 Set<Range<C>>表现 RangeSet，这样可以遍历其中的 Range。
- asSet(DiscreteDomain<C>)（仅 ImmutableRangeSet 支持）：用 ImmutableSortedSet<C>表现 RangeSet，以区间中所有元素的形式而不是区间本身的形式查看。（这个操作不支持 DiscreteDomain 和 RangeSet 都没有上边界，或都没有下边界的情况）

### RangeSet 的查询方法

为了方便操作，RangeSet 直接提供了若干查询方法，其中最突出的有:

- contains(C)：RangeSet 最基本的操作，判断 RangeSet 中是否有任何区间包含给定元素。
- rangeContaining(C)：返回包含给定元素的区间；若没有这样的区间，则返回 null。
- encloses(Range<C>)：简单明了，判断 RangeSet 中是否有任何区间包括给定区间。
- span()：返回包括 RangeSet 中所有区间的最小区间。

## RangeMap

RangeMap 描述了”不相交的、非空的区间”到特定值的映射。和 RangeSet 不同，RangeMap 不会合并相邻的映射，即便相邻的区间映射到相同的值。例如：

```

    RangeMap<Integer, String> rangeMap = TreeRangeMap.create();
    rangeMap.put(Range.closed(1, 10), "foo"); //{[1,10] => "foo"}
    rangeMap.put(Range.open(3, 6), "bar"); //{[1,3] => "foo", (3,6) => "bar", [6,10] => "foo"}
    rangeMap.put(Range.open(10, 20), "foo"); //{[1,3] => "foo", (3,6) => "bar", [6,10] => "foo", (10,20) => "foo"}
    rangeMap.remove(Range.closed(5, 11)); //{[1,3] => "foo", (3,5) => "bar", (11,20) => "foo"}

```

### RangeMap 的视图

RangeMap 提供两个视图：

- asMapOfRanges()：用 Map<Range<K>, V>表现 RangeMap。这可以用来遍历 RangeMap。
- subRangeMap(Range<K>)：用 RangeMap 类型返回 RangeMap 与给定 Range 的交集视图。这扩展了传统的 headMap、subMap 和 tailMap 操作。

 转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 2.2-新集合类型](http://ifeve.com/google-guava-newcollectiontypes/)
  
    