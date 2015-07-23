# 原生类型

## 概述

Java 的原生类型就是指基本类型：byte、short、int、long、float、double、char 和 boolean。

*在从 Guava 查找原生类型方法之前，可以先查查 [Arrays](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/Arrays.html) 类，或者对应的基础类型包装类，如 [Integer](http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/Integer.html)。*

原生类型不能当作对象或泛型的类型参数使用，这意味着许多通用方法都不能应用于它们。Guava 提供了若干通用工具，包括原生类型数组与集合 API 的交互，原生类型和字节数组的相互转换，以及对某些原生类型的无符号形式的支持。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="204"><b>原生类型</b><b></b></td>
<td width="408"><b>Guava</b><b> 工具类（都在 </b><b>com.google.common.primitives</b><b> 包</b><b>）</b><b></b></td>
</tr>
<tr>
<td width="204">byte</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Bytes.html"><tt>Bytes</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/SignedBytes.html"><tt>SignedBytes</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedBytes.html"><tt>UnsignedBytes</tt></a></td>
</tr>
<tr>
<td width="204">short</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Shorts.html"><tt>Shorts</tt></a></td>
</tr>
<tr>
<td width="204">int</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Ints.html"><tt>Ints</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedInteger.html"><tt>UnsignedInteger</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedInts.html"><tt>UnsignedInts</tt></a></td>
</tr>
<tr>
<td width="204">long</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Longs.html"><tt>Longs</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedLong.html"><tt>UnsignedLong</tt></a>, <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedLongs.html"><tt>UnsignedLongs</tt></a></td>
</tr>
<tr>
<td width="204">float</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Floats.html"><tt>Floats</tt></a></td>
</tr>
<tr>
<td width="204">double</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Doubles.html"><tt>Doubles</tt></a></td>
</tr>
<tr>
<td width="204">char</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Chars.html"><tt>Chars</tt></a></td>
</tr>
<tr>
<td width="204">boolean</td>
<td width="408"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/Booleans.html"><tt>Booleans</tt></a></td>
</tr>
</tbody>
</table>

Bytes 工具类没有定义任何区分有符号和无符号字节的方法，而是把它们都放到了 SignedBytes 和 UnsignedBytes 工具类中，因为字节类型的符号性比起其它类型要略微含糊一些。

int 和 long 的无符号形式方法在 UnsignedInts 和 UnsignedLongs 类中，但由于这两个类型的大多数用法都是有符号的，Ints 和 Longs 类按照有符号形式处理方法的输入参数。

此外，Guava 为 int 和 long 的无符号形式提供了包装类，即 UnsignedInteger 和 UnsignedLong，以帮助你使用类型系统，以极小的性能消耗对有符号和无符号值进行强制转换。

在本章下面描述的方法签名中，我们用 Wrapper 表示 JDK 包装类，prim 表示原生类型。（Prims 表示相应的 Guava 工具类。）

## 原生类型数组工具

原生类型数组是处理原生类型集合的最有效方式（从内存和性能双方面考虑）。Guava 为此提供了许多工具方法。

<table width="666" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="264"><b>方法签名</b><b></b></td>
<td valign="top" width="162"><b>描述</b><b></b></td>
<td valign="top" width="162"><b>类似方法</b><b></b></td>
<td width="78"><b>可用性</b><b></b></td>
</tr>
<tr>
<td width="264">List&lt;Wrapper&gt; asList(prim&#8230; backingArray)</td>
<td valign="top" width="162">把数组转为相应包装类的 List<b></b></td>
<td width="162"><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Arrays.html#asList(T...)">Arrays.asList</a><b></b></td>
<td width="78">符号无关*</td>
</tr>
<tr>
<td width="264">prim[] toArray(Collection&lt;Wrapper&gt; collection)</td>
<td valign="top" width="162">把集合拷贝为数组，和 collection.toArray()一样线程安全</td>
<td width="162"><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collection.html#toArray()">Collection.toArray()</a><b></b></td>
<td width="78">符号无关</td>
</tr>
<tr>
<td width="264">prim[] concat(prim[]&#8230; arrays)</td>
<td valign="top" width="162">串联多个原生类型数组</td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Iterables.html#concat(java.lang.Iterable...)">Iterables.concat</a><b></b></td>
<td width="78">符号无关</td>
</tr>
<tr>
<td width="264">boolean contains(prim[] array, prim target)</td>
<td valign="top" width="162">判断原生类型数组是否包含给定值</td>
<td width="162"><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collection.html#contains(java.lang.Object)">Collection.contains</a><b></b></td>
<td width="78">符号无关</td>
</tr>
<tr>
<td width="264">int indexOf(prim[] array, prim target)</td>
<td valign="top" width="162">给定值在数组中首次出现处的索引，若不包含此值返回-1</td>
<td width="162"><a href="http://docs.oracle.com/javase/6/docs/api/java/util/List.html#indexOf(java.lang.Object)">List.indexOf</a><b></b></td>
<td width="78">符号无关</td>
</tr>
<tr>
<td width="264">int lastIndexOf(prim[] array, prim target)</td>
<td valign="top" width="162">给定值在数组最后出现的索引，若不包含此值返回-1</td>
<td width="162"><a href="http://docs.oracle.com/javase/6/docs/api/java/util/List.html#lastIndexOf(java.lang.Object)">List.lastIndexOf</a><b></b></td>
<td width="78">符号无关</td>
</tr>
<tr>
<td width="264">prim min(prim&#8230; array)</td>
<td valign="top" width="162">数组中最小的值</td>
<td width="162"><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#min(java.util.Collection)">Collections.min</a><b></b></td>
<td width="78">符号相关*</td>
</tr>
<tr>
<td width="264">prim max(prim&#8230; array)</td>
<td valign="top" width="162">数组中最大的值</td>
<td width="162"><a href="http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#max(java.util.Collection)">Collections.max</a><b></b></td>
<td width="78">符号相关</td>
</tr>
<tr>
<td width="264">String join(String separator, prim&#8230; array)</td>
<td valign="top" width="162">把数组用给定分隔符连接为字符串</td>
<td width="162"><a href="http://code.google.com/p/guava-libraries/wiki/StringsExplained#Joiner">Joiner.on(separator).join</a><b></b></td>
<td width="78">符号相关</td>
</tr>
<tr>
<td width="264">Comparator&lt;prim[]&gt;   lexicographicalComparator()</td>
<td valign="top" width="162">按字典序比较原生类型数组的 Comparator</td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Ordering.html#lexicographical()">Ordering.natural().lexicographical()</a><b></b></td>
<td width="78">符号相关</td>
</tr>
</tbody>
</table>

*符号无关方法存在于 Bytes, Shorts, Ints, Longs, Floats, Doubles, Chars, Booleans。而UnsignedInts, UnsignedLongs, SignedBytes, 或 UnsignedBytes 不存在。

*符号相关方法存在于 SignedBytes, UnsignedBytes, Shorts, Ints, Longs, Floats, Doubles, Chars, Booleans, UnsignedInts, UnsignedLongs。而 Bytes 不存在。

## 通用工具方法

Guava 为原生类型提供了若干 JDK6 没有的工具方法。但请注意，其中某些方法已经存在于 JDK7 中。

<table width="667" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="232"><b>方法签名</b><b></b></td>
<td valign="top" width="302"><b>描述</b><b></b></td>
<td width="132"><b>可用性</b><b></b></td>
</tr>
<tr>
<td width="232">int compare(prim a, prim b)</td>
<td valign="top" width="302">传统的 Comparator.compare 方法，但针对原生类型。JDK7 的原生类型包装类也提供这样的方法<b></b></td>
<td width="132">符号相关<b></b></td>
</tr>
<tr>
<td width="232">prim checkedCast(long value)</td>
<td valign="top" width="302">把给定 long 值转为某一原生类型，若给定值不符合该原生类型，则抛出  IllegalArgumentException<b></b></td>
<td width="132">仅适用于符号相关的整型*</td>
</tr>
<tr>
<td width="232">prim saturatedCast(long value)</td>
<td valign="top" width="302">把给定 long 值转为某一原生类型，若给定值不符合则使用最接近的原生类型值<b></b></td>
<td width="132">仅适用于符号相关的整型</td>
</tr>
</tbody>
</table>

*这里的整型包括 byte, short, int, long。不包括 char, boolean, float, 或 double。

***译者注：不符合主要是指 long 值超出 prim 类型的范围，比如过大的 long 超出 int 范围。*

注：com.google.common.math.DoubleMath 提供了舍入 double 的方法，支持多种舍入模式。相见第 12 章的”浮点数运算”。

## 字节转换方法

Guava 提供了若干方法，用来把原生类型按**大字节序**与字节数组相互转换。所有这些方法都是符号无关的，此外 Booleans 没有提供任何下面的方法。

<table width="624" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="300"><b>方法或字段签名</b><b></b></td>
<td width="324"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="300">int BYTES</td>
<td width="324">常量：表示该原生类型需要的字节数</td>
</tr>
<tr>
<td width="300">prim fromByteArray(byte[] bytes)</td>
<td width="324">使用字节数组的前 Prims.BYTES 个字节，按<b>大字节序</b>返回原生类型值；如果 bytes.length &lt;= Prims.BYTES，抛出 IAE</td>
</tr>
<tr>
<td width="300">prim fromBytes(byte b1, &#8230;, byte bk)</td>
<td width="324">接受 Prims.BYTES 个字节参数，按<b>大字节序</b>返回原生类型值</td>
</tr>
<tr>
<td width="300">byte[] toByteArray(prim value)</td>
<td width="324">按<b>大字节序</b>返回 value 的字节数组</td>
</tr>
</tbody>
</table>

## 无符号支持

JDK 原生类型包装类提供了针对有符号类型的方法，而 UnsignedInts 和 UnsignedLongs 工具类提供了相应的无符号通用方法。UnsignedInts 和 UnsignedLongs 直接处理原生类型：使用时，由你自己保证只传入了无符号类型的值。

此外，对 int 和 long，Guava 提供了无符号包装类（[UnsignedInteger](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedInteger.html) 和 [UnsignedLong](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedLong.html)），来帮助你以极小的性能消耗，对有符号和无符号类型进行强制转换。

### 无符号通用工具方法

JDK 的原生类型包装类提供了有符号形式的类似方法。

<table width="624" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="473"><b>方法签名</b><b></b></td>
<td width="151"><b>说明</b><b></b></td>
</tr>
<tr>
<td width="473"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String)"><tt>int UnsignedInts.parseUnsignedInt(String)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)"><tt>long UnsignedLongs.parseUnsignedLong(String)</tt></a></td>
<td width="151">按无符号十进制解析字符串</td>
</tr>
<tr>
<td width="473"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String, int)"><tt>int UnsignedInts.parseUnsignedInt(String string, int radix)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)"><tt>long UnsignedLongs.parseUnsignedLong(String string, int radix)</tt></a></td>
<td width="151">按无符号的特定进制解析字符串</td>
</tr>
<tr>
<td width="473"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedInts.html#toString(int)"><tt>String UnsignedInts.toString(int)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedLongs.html#toString(long)"><tt>String UnsignedLongs.toString(long)</tt></a></td>
<td width="151">数字按无符号十进制转为字符串</td>
</tr>
<tr>
<td width="473"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedInts.html#toString(int, int)"><tt>String UnsignedInts.toString(int   value, int radix)</tt></a><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/primitives/UnsignedLongs.html#toString(long, int)"><tt>String UnsignedLongs.toString(long value, int radix)</tt></a></td>
<td width="151">数字按无符号特定进制转为字符串</td>
</tr>
</tbody>
</table>

### 无符号包装类

无符号包装类包含了若干方法，让使用和转换更容易。

<table width="624" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="321"><b>方法签名</b><b></b></td>
<td width="303"><b>说明</b><b></b></td>
</tr>
<tr>
<td width="321">UnsignedPrim add(UnsignedPrim), subtract, multiply, divide, remainder</td>
<td width="303">简单算术运算</td>
</tr>
<tr>
<td width="321">UnsignedPrim valueOf(BigInteger)</td>
<td width="303">按给定 BigInteger 返回无符号对象，若 BigInteger 为负或不匹配，抛出 IAE</td>
</tr>
<tr>
<td width="321">UnsignedPrim valueOf(long)</td>
<td width="303">按给定 long 返回无符号对象，若 long 为负或不匹配，抛出 IAE</td>
</tr>
<tr>
<td width="321">UnsignedPrim asUnsigned(prim value)</td>
<td width="303">把给定的值当作无符号类型。例如，UnsignedInteger.asUnsigned(1&lt;&lt;31)的值为 2<sup>31</sup>,尽管 1&lt;&lt;31 当作 int 时是负的</td>
</tr>
<tr>
<td width="321">BigInteger bigIntegerValue()</td>
<td width="303">用 BigInteger 返回该无符号对象的值</td>
</tr>
<tr>
<td width="321">toString(),  toString(int radix)</td>
<td width="303">返回无符号值的字符串表示</td>
</tr>
</tbody>
</table>

*译者注：UnsignedPrim 指各种无符号包装类，如 UnsignedInteger、UnsignedLong。*

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 7-原生类型](http://ifeve.com/google-guava-primitives/)
