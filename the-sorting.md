# 1.4-排序: Guava 强大的”流畅风格比较器”

[排序器[Ordering]](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html)是 Guava 流畅风格比较器[Comparator]的实现，它可以用来为构建复杂的比较器，以完成集合排序的功能。

从实现上说，Ordering 实例就是一个特殊的 Comparator 实例。Ordering 把很多基于 Comparator 的静态方法（如 Collections.max）包装为自己的实例方法（非静态方法），并且提供了链式调用方法，来定制和增强现有的比较器。

**创建排序器**：常见的排序器可以由下面的静态方法创建

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="199"><b>方法</b><b></b></td>
<td width="419"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#natural()"><tt>natural()</tt></a></td>
<td width="419">对可排序类型做自然排序，如数字按大小，日期按先后排序</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#usingToString()"><tt>usingToString()</tt></a></td>
<td width="419">按对象的字符串形式做字典排序[lexicographical ordering]</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#from(java.util.Comparator)"><tt>from(Comparator)</tt></a></td>
<td width="419">把给定的 Comparator 转化为排序器</td>
</tr>
</tbody>
</table>

实现自定义的排序器时，除了用上面的 from 方法，也可以跳过实现 Comparator，而直接继承 Ordering：

```


    Ordering<String> byLengthOrdering = new Ordering<String>() {
      public int compare(String left, String right) {
        return Ints.compare(left.length(), right.length());
      }
    };

```

**链式调用方法**：通过链式调用，可以由给定的排序器衍生出其它排序器

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="199"><b>方法</b></td>
<td width="419"><b>描述</b></td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#reverse()"><tt>reverse()</tt></a></td>
<td width="419">获取语义相反的排序器</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#nullsFirst()"><tt>nullsFirst()</tt></a></td>
<td width="419">使用当前排序器，但额外把 null 值排到最前面。</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#nullsLast()"><tt>nullsLast()</tt></a></td>
<td width="419">使用当前排序器，但额外把 null 值排到最后面。</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#compound(java.util.Comparator)"><tt>compound(Comparator)</tt></a></td>
<td width="419">合成另一个比较器，以处理当前排序器中的相等情况。</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#lexicographical()"><tt>lexicographical()</tt></a></td>
<td width="419">基于处理类型 T 的排序器，返回该类型的可迭代对象 Iterable&lt;T&gt;的排序器。</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#onResultOf(com.google.common.base.Function)"><tt>onResultOf(Function)</tt></a></td>
<td width="419">对集合中元素调用 Function，再按返回值用当前排序器排序。</td>
</tr>
</tbody>
</table>

例如，你需要下面这个类的排序器。

```

    class Foo {
        @Nullable String sortedBy;
        int notSortedBy;
    }

```

考虑到排序器应该能处理 sortedBy 为 null 的情况，我们可以使用下面的链式调用来合成排序器：

```

    Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(new Function<Foo, String>() {
      public String apply(Foo foo) {
        return foo.sortedBy;
      }
    });

```

当阅读链式调用产生的排序器时，应该从后往前读。上面的例子中，排序器首先调用 apply 方法获取 sortedBy 值，并把 sortedBy 为 null 的元素都放到最前面，然后把剩下的元素按 sortedBy 进行自然排序。之所以要从后往前读，是因为每次链式调用都是用后面的方法包装了前面的排序器。

*注：用 compound 方法包装排序器时，就不应遵循从后往前读的原则。为了避免理解上的混乱，请不要把 compound 写在一长串链式调用的中间，你可以另起一行，在链中最先或最后调用 compound。*

超过一定长度的链式调用，也可能会带来阅读和理解上的难度。我们建议按下面的代码这样，在一个链中最多使用三个方法。此外，你也可以把 Function 分离成中间对象，让链式调用更简洁紧凑。

```

    Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(sortKeyFunction)

```

**运用排序器**：Guava 的排序器实现有若干操纵集合或元素值的方法

<table>
<tbody>
<tr>
<td><strong>方法</strong></td>
<td><strong>描述</strong></td>
<td><strong>另请参见</strong></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#greatestOf(java.lang.Iterable, int)" rel="nofollow"><tt>greatestOf(Iterable iterable, int k)</tt></a></td>
<td>获取可迭代对象中最大的k个元素。</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#leastOf(java.lang.Iterable, int)" rel="nofollow"><tt>leastOf</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#isOrdered(java.lang.Iterable)" rel="nofollow"><tt>isOrdered(Iterable)</tt></a></td>
<td>判断可迭代对象是否已按排序器排序：允许有排序值相等的元素。</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#isStrictlyOrdered(java.lang.Iterable)" rel="nofollow"><tt>isStrictlyOrdered</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#sortedCopy(java.lang.Iterable)" rel="nofollow"><tt>sortedCopy(Iterable)</tt></a></td>
<td>判断可迭代对象是否已严格按排序器排序：不允许排序值相等的元素。</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#immutableSortedCopy(java.lang.Iterable)" rel="nofollow"><tt>immutableSortedCopy</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min(E, E)" rel="nofollow"><tt>min(E, E)</tt></a></td>
<td>返回两个参数中最小的那个。如果相等，则返回第一个参数。</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max(E, E)" rel="nofollow"><tt>max(E, E)</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min(E, E, E, E...)" rel="nofollow"><tt>min(E, E, E, E...)</tt></a></td>
<td>返回多个参数中最小的那个。如果有超过一个参数都最小，则返回第一个最小的参数。</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max(E, E, E, E...)" rel="nofollow"><tt>max(E, E, E, E...)</tt></a></td>
</tr>
<tr>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min(java.lang.Iterable)" rel="nofollow"><tt>min(Iterable)</tt></a></td>
<td>返回迭代器中最小的元素。如果可迭代对象中没有元素，则抛出 NoSuchElementException。</td>
<td><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max(java.lang.Iterable)" rel="nofollow"><tt>max(Iterable)</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#min(java.util.Iterator)" rel="nofollow"><tt>min(Iterator)</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#max(java.util.Iterator)" rel="nofollow"><tt>max(Iterator)</tt></a></td>
</tr>
</tbody>
</table>

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 排序: Guava 强大的”流畅风格比较器”](http://ifeve.com/google-guava-ordering/)