# 散列

## 概述

Java 内建的散列码[hash code]概念被限制为 32 位，并且没有分离散列算法和它们所作用的数据，因此很难用备选算法进行替换。此外，使用 Java 内建方法实现的散列码通常是劣质的，部分是因为它们最终都依赖于 JDK 类中已有的劣质散列码。

Object.hashCode 往往很快，但是在预防碰撞上却很弱，也没有对分散性的预期。这使得它们很适合在散列表中运用，因为额外碰撞只会带来轻微的性能损失，同时差劲的分散性也可以容易地通过再散列来纠正（Java 中所有合理的散列表都用了再散列方法）。然而，在简单散列表以外的散列运用中，Object.hashCode 几乎总是达不到要求——因此，有了 [com.google.common.hash](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/package-summary.html) 包。

## 散列包的组成

在这个包的 Java doc 中，我们可以看到很多不同的类，但是文档中没有明显地表明它们是怎样 一起配合工作的。在介绍散列包中的类之前，让我们先来看下面这段代码范例：

```

    HashFunction hf = Hashing.md5();
    HashCode hc = hf.newHasher()
        .putLong(id)
        .putString(name, Charsets.UTF_8)
        .putObject(person, personFunnel)
        .hash();
```

**HashFunction**

[HashFunction](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/HashFunction.html) 是一个单纯的（引用透明的）、无状态的方法，它把任意的数据块映射到固定数目的位指，并且保证相同的输入一定产生相同的输出，不同的输入尽可能产生不同的输出。

**Hasher**

HashFunction 的实例可以提供有状态的 [Hasher](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/Hasher.html)，Hasher 提供了流畅的语法把数据添加到散列运算，然后获取散列值。Hasher 可以接受所有原生类型、字节数组、字节数组的片段、字符序列、特定字符集的字符序列等等，或者任何给定了 Funnel 实现的对象。

Hasher 实现了 PrimitiveSink 接口，这个接口为接受原生类型流的对象定义了 fluent 风格的 API

**Funnel**

Funnel 描述了如何把一个具体的对象类型分解为原生字段值，从而写入 PrimitiveSink。比如，如果我们有这样一个类：

```

    class Person {
        final int id;
        final String firstName;
        final String lastName;
        final int birthYear;
    }

```

它对应的 Funnel 实现可能是：

```


    Funnel<Person> personFunnel = new Funnel<Person>() {
	    @Override
	    public void funnel(Person person, PrimitiveSink into) {
	        into
	            .putInt(person.id)
	            .putString(person.firstName, Charsets.UTF_8)
	            .putString(person.lastName, Charsets.UTF_8)
	            .putInt(birthYear);
	    }
    }

```

*注：putString(“abc”, Charsets.UTF_8).putString(“def”, Charsets.UTF_8)完全等同于 putString(“ab”, Charsets.UTF_8).putString(“cdef”, Charsets.UTF_8)，因为它们提供了相同的字节序列。这可能带来预料之外的散列冲突。增加某种形式的分隔符有助于消除散列冲突。*

**HashCode**

一旦 Hasher 被赋予了所有输入，就可以通过 [hash()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/Hasher.html#hash())方法获取 [HashCode](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/HashCode.html) 实例（多次调用 hash()方法的结果是不确定的）。HashCode 可以通过 [asInt()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/HashCode.html#asInt())、[asLong()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/HashCode.html#asLong())、[asBytes()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/HashCode.html#asBytes())方法来做相等性检测，此外，<a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/HashCode.html#writeBytesTo(byte[], int, int)">writeBytesTo(array, offset, maxLength)</a>把散列值的前 maxLength字节写入字节数组。

## 布鲁姆过滤器[BloomFilter]

布鲁姆过滤器是哈希运算的一项优雅运用，它可以简单地基于 Object.hashCode()实现。简而言之，布鲁姆过滤器是一种概率数据结构，它允许你检测某个对象是一定不在过滤器中，还是可能已经添加到过滤器了。[布鲁姆过滤器的维基页面](http://en.wikipedia.org/wiki/Bloom_filter)对此作了全面的介绍，同时我们推荐 github 中的一个[教程](http://billmill.org/bloomfilter-tutorial/)。

Guava 散列包有一个内建的布鲁姆过滤器实现，你只要提供 Funnel 就可以使用它。你可以使用 <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/BloomFilter.html#create(com.google.common.hash.Funnel, int, double)">create(Funnel funnel, int expectedInsertions, double falsePositiveProbability)</a>方法获取 [BloomFilter<T>](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/BloomFilter.html)，缺省误检率[falsePositiveProbability]为 3%。BloomFilter<T> 提供了 [boolean mightContain(T)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/BloomFilter.html#mightContain(T)) 和 [void put(T)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/hash/BloomFilter.html#put(T))，它们的含义都不言自明了。

```

    BloomFilter<Person> friends = BloomFilter.create(personFunnel, 500, 0.01);
    for(Person friend : friendsList) {
    friends.put(friend);
    }
    
    // 很久以后
    if (friends.mightContain(dude)) {
    //dude不是朋友还运行到这里的概率为1%
    //在这儿，我们可以在做进一步精确检查的同时触发一些异步加载
    }

```

## Hashing 类

Hashing 类提供了若干散列函数，以及运算 HashCode 对象的工具方法。

### 已提供的散列函数

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="126"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#md5()"><tt>md5()</tt></a></td>
<td width="118"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#murmur3_128()"><tt>murmur3_128()</tt></a></td>
<td width="205"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#murmur3_32()"><tt>murmur3_32()</tt></a></td>
<td width="169"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#sha1()"><tt>sha1()</tt></a></td>
</tr>
<tr>
<td width="126"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#sha256()"><tt>sha256()</tt></a></td>
<td width="118"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#sha512()"><tt>sha512()</tt></a></td>
<td width="205"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#goodFastHash(int)"><tt>goodFastHash(int bits)</tt></a></td>
<td width="169"></td>
</tr>
</tbody>
</table>

### HashCode 运算

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="216"><b>方法</b><b></b></td>
<td width="403"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#combineOrdered(java.lang.Iterable)"><tt>HashCode</tt><tt> </tt><tt>combineOrdered( Iterable&lt;HashCode&gt;)</tt></a></td>
<td width="403">以有序方式联接散列码，如果两个散列集合用该方法联接出的散列码相同，那么散列集合的元素可能是顺序相等的</td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#combineUnordered(java.lang.Iterable)"><tt>HashCode   combineUnordered( Iterable&lt;HashCode&gt;)</tt></a></td>
<td width="403">以无序方式联接散列码，如果两个散列集合用该方法联接出的散列码相同，那么散列集合的元素可能在某种排序下是相等的</td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release12/javadoc/com/google/common/hash/Hashing.html#consistentHash(com.google.common.hash.HashCode, int)"><tt>int   consistentHash( HashCode, int buckets)</tt></a></td>
<td width="403">为给定的&#8221;桶&#8221;大小返回一致性哈希值。当&#8221;桶&#8221;增长时，该方法保证最小程度的一致性哈希值变化。详见<a href="http://en.wikipedia.org/wiki/Consistent_hashing">一致性哈希</a>。</td>
</tr>
</tbody>
</table>

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 10-散列](http://ifeve.com/google-guava-hashing/)