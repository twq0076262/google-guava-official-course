# 字符串处理：分割，连接，填充

## 连接器[Joiner]

用分隔符把字符串序列连接起来也可能会遇上不必要的麻烦。如果字符串序列中含有 null，那连接操作会更难。Fluent 风格的 [Joiner](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Joiner.html) 让连接字符串更简单。

```

    Joiner joiner = Joiner.on("; ").skipNulls();
    return joiner.join("Harry", null, "Ron", "Hermione");

```

上述代码返回”Harry; Ron; Hermione”。另外，useForNull(String)方法可以给定某个字符串来替换 null，而不像 skipNulls()方法是直接忽略 null。 Joiner 也可以用来连接对象类型，在这种情况下，它会把对象的 toString()值连接起来。

```

    Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"

```

*警告：joiner 实例总是不可变的。用来定义 joiner 目标语义的配置方法总会返回一个新的 joiner 实例。这使得 joiner 实例都是线程安全的，你可以将其定义为 static final常量。*

## 拆分器[Splitter]

JDK 内建的字符串拆分工具有一些古怪的特性。比如，String.split 悄悄丢弃了尾部的分隔符。 问题：”,a,,b,”.split(“,”)返回？

1. “”, “a”, “”, “b”, “”
2. null, “a”, null, “b”, null
3. “a”, null, “b”
4. “a”, “b”
5. 以上都不对

正确答案是 5：””, “a”, “”, “b”。只有尾部的空字符串被忽略了。 [Splitter](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html) 使用令人放心的、直白的流畅 API 模式对这些混乱的特性作了完全的掌控。

```

    Splitter.on(',')
	        .trimResults()
	        .omitEmptyStrings()
	        .split("foo,bar,,   qux");

```

上述代码返回 Iterable<String>，其中包含”foo”、”bar”和”qux”。Splitter 可以被设置为按照任何模式、字符、字符串或字符匹配器拆分。

### 拆分器工厂

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="216"><b>方法</b><b></b></td>
<td width="162"><b>描述</b><b></b></td>
<td width="234"><b>范例</b><b></b></td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on(char)"><tt>Splitter.on(char)</tt></a></td>
<td width="162">按单个字符拆分</td>
<td width="234">Splitter.on(&#8216;;&#8217;)</td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on(com.google.common.base.CharMatcher)"><tt>Splitter.on(CharMatcher)</tt></a></td>
<td width="162">按字符匹配器拆分</td>
<td width="234">Splitter.on(CharMatcher.BREAKING_WHITESPACE)</td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on(java.lang.String)"><tt>Splitter.on(String)</tt></a></td>
<td width="162">按字符串拆分</td>
<td width="234">Splitter.on(&#8220;,   &#8220;)</td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#on(java.util.regex.Pattern)"><tt>Splitter.on(Pattern)</tt></a> <a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#onPattern(java.lang.String)"><tt>Splitter.onPattern(String)</tt></a></td>
<td width="162">按正则表达式拆分</td>
<td width="234">Splitter.onPattern(&#8220;\r?\n&#8221;)</td>
</tr>
<tr>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#fixedLength(int)"><tt>Splitter.fixedLength(int)</tt></a></td>
<td width="162">按固定长度拆分；最后一段可能比给定长度短，但不会为空。</td>
<td width="234">Splitter.fixedLength(3)</td>
</tr>
</tbody>
</table>

### 拆分器修饰符

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="204"><b>方法</b><b></b></td>
<td width="408"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#omitEmptyStrings()"><tt>omitEmptyStrings()</tt></a></td>
<td width="408">从结果中自动忽略空字符串</td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#trimResults()"><tt>trimResults()</tt></a></td>
<td width="408">移除结果字符串的前导空白和尾部空白</td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#trimResults(com.google.common.base.CharMatcher)"><tt>trimResults(CharMatcher)</tt></a></td>
<td width="408">给定匹配器，移除结果字符串的前导匹配字符和尾部匹配字符</td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Splitter.html#limit(int)"><tt>limit(int)</tt></a></td>
<td width="408">限制拆分出的字符串数量</td>
</tr>
</tbody>
</table>

如果你想要拆分器返回 List，只要使用 Lists.newArrayList(splitter.split(string))或类似方法。 *警告：splitter 实例总是不可变的。用来定义 splitter 目标语义的配置方法总会返回一个新的 splitter 实例。这使得 splitter 实例都是线程安全的，你可以将其定义为 static final 常量。*

## 字符匹配器[CharMatcher]

在以前的 Guava 版本中，StringUtil 类疯狂地膨胀，其拥有很多处理字符串的方法：allAscii、collapse、collapseControlChars、collapseWhitespace、indexOfChars、lastIndexNotOf、numSharedChars、removeChars、removeCrLf、replaceChars、retainAllChars、strip、stripAndCollapse、stripNonDigits。 所有这些方法指向两个概念上的问题：

1. 怎么才算匹配字符？
2. 如何处理这些匹配字符？

为了收拾这个泥潭，我们开发了 CharMatcher。

直观上，你可以认为一个 CharMatcher 实例代表着某一类字符，如数字或空白字符。事实上来说，CharMatcher 实例就是对字符的布尔判断——CharMatcher 确实也实现了 [Predicate<Character>](http://code.google.com/p/guava-libraries/wiki/FunctionalExplained#Predicate)——但类似”所有空白字符”或”所有小写字母”的需求太普遍了，Guava 因此创建了这一 API。

然而使用 CharMatcher 的好处更在于它提供了一系列方法，让你对字符作特定类型的操作：修剪[trim]、折叠[collapse]、移除[remove]、保留[retain]等等。CharMatcher 实例首先代表概念 1：怎么才算匹配字符？然后它还提供了很多操作概念 2：如何处理这些匹配字符？这样的设计使得 API 复杂度的线性增加可以带来灵活性和功能两方面的增长。

```

    String noControl = CharMatcher.JAVA_ISO_CONTROL.removeFrom(string); //移除control字符
    String theDigits = CharMatcher.DIGIT.retainFrom(string); //只保留数字字符
    String spaced = CharMatcher.WHITESPACE.trimAndCollapseFrom(string, ' ');
    //去除两端的空格，并把中间的连续空格替换成单个空格
    String noDigits = CharMatcher.JAVA_DIGIT.replaceFrom(string, "*"); //用*号替换所有数字
    String lowerAndDigit = CharMatcher.JAVA_DIGIT.or(CharMatcher.JAVA_LOWER_CASE).retainFrom(string);
    // 只保留数字和小写字母

```

注：CharMatcher 只处理 char 类型代表的字符；它不能理解 0x10000 到 0x10FFFF 的 Unicode  增补字符。这些逻辑字符以代理对[surrogate pairs]的形式编码进字符串，而 CharMatcher 只能将这种逻辑字符看待成两个独立的字符。

### 获取字符匹配器

CharMatcher 中的常量可以满足大多数字符匹配需求：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#ANY"><tt>ANY</tt></a></td>
<td width="138"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#NONE"><tt>NONE</tt></a></td>
<td width="138"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#WHITESPACE"><tt>WHITESPACE</tt></a></td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#BREAKING_WHITESPACE"><tt>BREAKING_WHITESPACE</tt></a></td>
</tr>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#INVISIBLE"><tt>INVISIBLE</tt></a></td>
<td width="138"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#DIGIT"><tt>DIGIT</tt></a></td>
<td width="138"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#JAVA_LETTER"><tt>JAVA_LETTER</tt></a></td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#JAVA_DIGIT"><tt>JAVA_DIGIT</tt></a></td>
</tr>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#JAVA_LETTER_OR_DIGIT"><tt>JAVA_LETTER_OR_DIGIT</tt></a></td>
<td width="138"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#JAVA_ISO_CONTROL"><tt>JAVA_ISO_CONTROL</tt></a></td>
<td width="138"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#JAVA_LOWER_CASE"><tt>JAVA_LOWER_CASE</tt></a></td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#JAVA_UPPER_CASE"><tt>JAVA_UPPER_CASE</tt></a></td>
</tr>
<tr>
<td width="174"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#ASCII"><tt>ASCII</tt></a></td>
<td width="138"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#SINGLE_WIDTH"><tt>SINGLE_WIDTH</tt></a></td>
<td width="138"></td>
<td width="162"></td>
</tr>
</tbody>
</table>

其他获取字符匹配器的常见方法包括：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="162"><b>方法</b><b></b></td>
<td width="450"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#anyOf(java.lang.CharSequence)"><tt>anyOf(CharSequence)</tt></a></td>
<td width="450">枚举匹配字符。如 CharMatcher.anyOf(&#8220;aeiou&#8221;)匹配小写英语元音</td>
</tr>
<tr>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#is(char)"><tt>is(char)</tt></a></td>
<td width="450">给定单一字符匹配。</td>
</tr>
<tr>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#inRange(char, char)"><tt>inRange(char, char)</tt></a></td>
<td width="450">给定字符范围匹配，如 CharMatcher.inRange(&#8216;a&#8217;, &#8216;z&#8217;)</td>
</tr>
</tbody>
</table>

此外，CharMatcher 还有 [negate()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#negate())、[and(CharMatcher)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#and(com.google.common.base.CharMatcher))和 [or(CharMatcher)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#or(com.google.common.base.CharMatcher))方法。

### 使用字符匹配器

CharMatcher 提供了[多种多样的方法](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#method_summary)操作 CharSequence 中的特定字符。其中最常用的罗列如下：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="264"><b>方法</b><b></b></td>
<td width="348"><b>描述</b><b></b></td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#collapseFrom(java.lang.CharSequence, char)"><tt>collapseFrom(CharSequence,   char)</tt></a></td>
<td width="348">把每组连续的匹配字符替换为特定字符。如 WHITESPACE.collapseFrom(string, &#8216; &#8216;)把字符串中的连续空白字符替换为单个空格。</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#matchesAllOf(java.lang.CharSequence)"><tt>matchesAllOf(CharSequence)</tt></a></td>
<td width="348">测试是否字符序列中的所有字符都匹配。</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#removeFrom(java.lang.CharSequence)"><tt>removeFrom(CharSequence)</tt></a></td>
<td width="348">从字符序列中移除所有匹配字符。</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#retainFrom(java.lang.CharSequence)"><tt>retainFrom(CharSequence)</tt></a></td>
<td width="348">在字符序列中保留匹配字符，移除其他字符。</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#trimFrom(java.lang.CharSequence)"><tt>trimFrom(CharSequence)</tt></a></td>
<td width="348">移除字符序列的前导匹配字符和尾部匹配字符。</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CharMatcher.html#replaceFrom(java.lang.CharSequence, java.lang.CharSequence)"><tt>replaceFrom(CharSequence,   CharSequence)</tt></a></td>
<td width="348">用特定字符序列替代匹配字符。</td>
</tr>
</tbody>
</table>

所有这些方法返回 String，除了 matchesAllOf 返回的是 boolean。

## 字符集[Charsets]

不要这样做字符集处理：

```

    try {
        bytes = string.getBytes("UTF-8");
     } catch (UnsupportedEncodingException e) {
        // how can this possibly happen?
        throw new AssertionError(e);
    }

```

试试这样写：

```

    bytes = string.getBytes(Charsets.UTF_8);

```

[Charsets](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Charsets.html) 针对所有 Java 平台都要保证支持的六种字符集提供了常量引用。尝试使用这些常量，而不是通过名称获取字符集实例。

## 大小写格式[CaseFormat]

CaseFormat 被用来方便地在各种 ASCII 大小写规范间转换字符串——比如，编程语言的命名规范。CaseFormat 支持的格式如下：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="204"><b>格式</b><b></b></td>
<td width="408"><b>范例</b><b></b></td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CaseFormat.html#LOWER_CAMEL"><tt>LOWER_CAMEL</tt></a></td>
<td width="408">lowerCamel</td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CaseFormat.html#LOWER_HYPHEN"><tt>LOWER_HYPHEN</tt></a></td>
<td width="408">lower-hyphen</td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CaseFormat.html#LOWER_UNDERSCORE"><tt>LOWER_UNDERSCORE</tt></a></td>
<td width="408">lower_underscore</td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CaseFormat.html#UPPER_CAMEL"><tt>UPPER_CAMEL</tt></a></td>
<td width="408">UpperCamel</td>
</tr>
<tr>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/CaseFormat.html#UPPER_UNDERSCORE"><tt>UPPER_UNDERSCORE</tt></a></td>
<td width="408">UPPER_UNDERSCORE</td>
</tr>
</tbody>
</table>

CaseFormat的用法很直接：

```

    CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "CONSTANT_NAME")); // returns "constantName"

```

我们 CaseFormat 在某些时候尤其有用，比如编写代码生成器的时候。

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 6-字符串处理：分割，连接，填充](http://ifeve.com/google-guava-strings/)
