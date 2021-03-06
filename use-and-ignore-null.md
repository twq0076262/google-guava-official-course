# 1.1-使用和避免 null

[Doug Lea](http://en.wikipedia.org/wiki/Doug_Lea) 说，“Null 真糟糕。”

当 [Sir C. A. R. Hoare](http://en.wikipedia.org/wiki/C._A._R._Hoare) 使用了 null 引用后说，”使用它导致了十亿美金的错误。”

轻率地使用 null 可能会导致很多令人惊愕的问题。通过学习 Google 底层代码库，我们发现 95%的集合类不接受 null 值作为元素。我们认为， 相比默默地接受 null，使用快速失败操作拒绝 null 值对开发者更有帮助。

此外，Null 的含糊语义让人很不舒服。Null 很少可以明确地表示某种语义，例如，Map.get(key)返回 Null 时，可能表示 map 中的值是 null，亦或 map 中没有 key 对应的值。Null 可以表示失败、成功或几乎任何情况。使用 Null 以外的特定值，会让你的逻辑描述变得更清晰。

Null 确实也有合适和正确的使用场景，如在性能和速度方面 Null 是廉价的，而且在对象数组中，出现 Null 也是无法避免的。但相对于底层库来说，在应用级别的代码中，Null 往往是导致混乱，疑难问题和模糊语义的元凶，就如同我们举过的 Map.get(key)的例子。最关键的是，Null 本身没有定义它表达的意思。

鉴于这些原因，很多 Guava 工具类对 Null 值都采用快速失败操作，除非工具类本身提供了针对 Null 值的因变措施。此外，Guava 还提供了很多工具类，让你更方便地用特定值替换 Null 值。

## 具体案例

不要在 Set 中使用 null，或者把 null 作为 map 的键值。使用特殊值代表 null 会让查找操作的语义更清晰。

如果你想把 null 作为 map 中某条目的值，更好的办法是 不把这一条目放到 map 中，而是单独维护一个”值为 null 的键集合” (null keys)。Map 中对应某个键的值是 null，和 map 中没有对应某个键的值，是非常容易混淆的两种情况。因此，最好把值为 null 的键分离开，并且仔细想想，null 值的键在你的项目中到底表达了什么语义。

如果你需要在列表中使用 null——并且这个列表的数据是稀疏的，使用 Map<Integer, E>可能会更高效，并且更准确地符合你的潜在需求。

此外，考虑一下使用自然的 null 对象——特殊值。举例来说，为某个 enum 类型增加特殊的枚举值表示 null，比如 java.math.RoundingMode 就定义了一个枚举值 UNNECESSARY，它表示一种不做任何舍入操作的模式，用这种模式做舍入操作会直接抛出异常。

如果你真的需要使用 null 值，但是 null 值不能和 Guava 中的集合实现一起工作，你只能选择其他实现。比如，用 JDK 中的 Collections.unmodifiableList 替代 Guava 的 ImmutableList

## Optional

大多数情况下，开发人员使用 null 表明的是某种缺失情形：可能是已经有一个默认值，或没有值，或找不到值。例如，Map.get 返回 null 就表示找不到给定键对应的值。

Guava 用 Optional<T>表示可能为 null 的 T 类型引用。一个 Optional 实例可能包含非 null 的引用（我们称之为引用存在），也可能什么也不包括（称之为引用缺失）。它从不说包含的是 null 值，而是用存在或缺失来表示。但 Optional 从不会包含 null 值引用。

```

    Optional<Integer> possible = Optional.of(5);
    
    possible.isPresent(); // returns true
    
    possible.get(); // returns 5

```

Optional 无意直接模拟其他编程环境中的”可选” or “可能”语义，但它们的确有相似之处。

Optional 最常用的一些操作被罗列如下：

**创建 Optional 实例（以下都是静态方法）：**

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#of(T)"><tt>Optional.of(T)</tt> </a></td>
<td width="419">创建指定引用的 Optional 实例，若引用为 null 则快速失败</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#absent()"><tt>Optional.absent()</tt></a></td>
<td width="419">创建引用缺失的 Optional 实例</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#fromNullable(T)"><tt>Optional.fromNullable(T)</tt></a></td>
<td width="419">创建指定引用的 Optional 实例，若引用为 null 则表示缺失</td>
</tr>
</tbody>
</table>

**用 Optional 实例查询引用（以下都是非静态方法）：**

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#isPresent()"><tt>boolean isPresent()</tt></a></td>
<td width="419">如果 Optional 包含非 null 的引用（引用存在），返回true</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#get()"><tt>T get()</tt></a></td>
<td width="419">返回 Optional 所包含的引用，若引用缺失，则抛出 java.lang.IllegalStateException</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#or(T)"><tt>T or(T)</tt></a></td>
<td width="419">返回 Optional 所包含的引用，若引用缺失，返回指定的值</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#orNull()"><tt>T orNull()</tt></a></td>
<td width="419">返回 Optional 所包含的引用，若引用缺失，返回 null</td>
</tr>
<tr>
<td width="199"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#asSet()"><tt>Set&lt;T&gt; asSet()</tt></a></td>
<td width="419">返回 Optional 所包含引用的单例不可变集，如果引用存在，返回一个只有单一元素的集合，如果引用缺失，返回一个空集合。</td>
</tr>
</tbody>
</table>

**使用 Optional 的意义在哪儿？**

使用 Optional 除了赋予 null 语义，增加了可读性，最大的优点在于它是一种傻瓜式的防护。Optional 迫使你积极思考引用缺失的情况，因为你必须显式地从 Optional 获取引用。直接使用 null 很容易让人忘掉某些情形，尽管 FindBugs 可以帮助查找 null 相关的问题，但是我们还是认为它并不能准确地定位问题根源。

如同输入参数，方法的返回值也可能是 null。和其他人一样，你绝对很可能会忘记别人写的方法 method(a,b)会返回一个 null，就好像当你实现 method(a,b)时，也很可能忘记输入参数 a 可以为 null。将方法的返回类型指定为 Optional，也可以迫使调用者思考返回的引用缺失的情形。

**其他处理 null 的便利方法**

当你需要用一个默认值来替换可能的 null，请使用 Objects.firstNonNull(T, T) 方法。如果两个值都是 null，该方法会抛出 NullPointerException。Optional 也是一个比较好的替代方案，例如：Optional.of(first).or(second).

还有其它一些方法专门处理 null 或空字符串：emptyToNull(String)，nullToEmpty(String)，isNullOrEmpty(String)。我们想要强调的是，这些方法主要用来与混淆 null/空的 API 进行交互。当每次你写下混淆 null/空的代码时，Guava 团队都泪流满面。（好的做法是积极地把 null 和空区分开，以表示不同的含义，在代码中把 null 和空同等对待是一种令人不安的坏味道。

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 1.1-使用和避免 null](http://ifeve.com/google-guava-using-and-avoiding-null/)