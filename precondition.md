# 1.2-前置条件

前置条件：让方法调用的前置条件判断更简单。

Guava 在 [Preconditions](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html) 类中提供了若干前置条件判断的实用方法，我们强烈建议[在 Eclipse 中静态导入这些方法](http://ifeve.com/eclipse-static-import/)。每个方法都有三个变种：

- 没有额外参数：抛出的异常中没有错误消息；
- 有一个 Object 对象作为额外参数：抛出的异常使用 Object.toString() 作为错误消息；
- 有一个 String 对象作为额外参数，并且有一组任意数量的附加 Object 对象：这个变种处理异常消息的方式有点类似 printf，但考虑 GWT 的兼容性和效率，只支持%s 指示符。例如：

```

    checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
    checkArgument(i < j, "Expected i < j, but %s > %s", i, j);

```

<table border="1" width="631" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="199"><b>方法声明（不包括额外参数）</b><b></b></td>
<td valign="top" width="222"><b>描述</b><b></b></td>
<td valign="top" width="210"><b>检查失败时抛出的异常</b><b></b></td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkArgument(boolean)"><tt>checkArgument(boolean)</tt></a></td>
<td width="222">检查 boolean 是否为 true，用来检查传递给方法的参数。</td>
<td width="210">IllegalArgumentException</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkNotNull(T)"><tt>checkNotNull(T)</tt></a></td>
<td width="222">检查 value 是否为 null，该方法直接返回 value，因此可以内嵌使用 checkNotNull<tt>。</tt></td>
<td width="210">NullPointerException</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkState(boolean)"><tt>checkState(boolean)</tt></a></td>
<td width="222">用来检查对象的某些状态。</td>
<td width="210">IllegalStateException</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkElementIndex(int, int)"><tt>checkElementIndex(int index, int size)</tt></a></td>
<td width="222">检查 index 作为索引值对某个列表、字符串或数组是否有效。index&gt;=0 &amp;&amp; index&lt;size *</td>
<td width="210">IndexOutOfBoundsException</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkPositionIndex(int, int)"><tt>checkPositionIndex(int index, int size)</tt></a></td>
<td width="222">检查 index 作为位置值对某个列表、字符串或数组是否有效。index&gt;=0 &amp;&amp; index&lt;=size *</td>
<td width="210">IndexOutOfBoundsException</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Preconditions.html#checkPositionIndexes(int, int, int)"><tt>checkPositionIndexes(int start, int end, int size)</tt></a></td>
<td width="222">检查[start, end]表示的位置范围对某个列表、字符串或数组是否有效*</td>
<td width="210">IndexOutOfBoundsException</td>
</tr>
</tbody>
</table>

译者注：

*索引值常用来查找列表、字符串或数组中的元素，如 List.get(int), String.charAt(int)*

*位置值和位置范围常用来截取列表、字符串或数组，如 List.subList(int，int), String.substring(int)*

相比 Apache Commons 提供的类似方法，我们把 Guava 中的 Preconditions 作为首选。Piotr Jagielski 在[他的博客](http://piotrjagielski.com/blog/google-guava-vs-apache-commons-for-argument-validation/)中简要地列举了一些理由：

- 在静态导入后，Guava 方法非常清楚明晰。checkNotNull 清楚地描述做了什么，会抛出什么异常；
- checkNotNull 直接返回检查的参数，让你可以在构造函数中保持字段的单行赋值风格：this.field = checkNotNull(field)
- 简单的、参数可变的 printf 风格异常信息。鉴于这个优点，在 JDK7 已经引入 [Objects.requireNonNull](http://docs.oracle.com/javase/7/docs/api/java/util/Objects.html#requireNonNull(java.lang.Object,java.lang.String)) 的情况下，我们仍然建议你使用 checkNotNull。


*在编码时，如果某个值有多重的前置条件，我们建议你把它们放到不同的行，这样有助于在调试时定位。此外，把每个前置条件放到不同的行，也可以帮助你编写清晰和有用的错误消息。*

转载自[并发编程网 – ifeve.com](http://ifeve.com/)  
本文链接地址: [[Google Guava] 1.2-前置条件](http://ifeve.com/google-guava-preconditions/)