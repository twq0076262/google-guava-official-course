# I/O

## 字节流和字符流

Guava 使用术语”流” 来表示可关闭的，并且在底层资源中有位置状态的 I/O 数据流。术语”字节流”指的是 InputStream 或 OutputStream，”字符流”指的是 Reader 或 Writer（虽然他们的接口 Readable 和 Appendable 被更多地用于方法参数）。相应的工具方法分别在 [ByteStreams](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteStreams.html) 和 [CharStreams](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/CharStreams.html) 中。

大多数 Guava 流工具一次处理一个完整的流，并且/或者为了效率自己处理缓冲。还要注意到，接受流为参数的 Guava 方法不会关闭这个流：关闭流的职责通常属于打开流的代码块。

其中的一些工具方法列举如下：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="312"><b>ByteStreams</b></td>
<td width="307"><b>CharStreams</b></td>
</tr>
<tr>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteStreams.html#toByteArray(java.io.InputStream)"><tt>byte[] toByteArray(InputStream)</tt></a></td>
<td width="307"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/CharStreams.html#toString(java.lang.Readable)"><tt>String toString(Readable)</tt></a></td>
</tr>
<tr>
<td width="312">N/A</td>
<td width="307"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharStreams.html#readLines(java.lang.Readable)"><tt>List&lt;String&gt; readLines(Readable)</tt></a></td>
</tr>
<tr>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteStreams.html#copy(java.io.InputStream, java.io.OutputStream)"><tt>long copy(InputStream, OutputStream)</tt></a></td>
<td width="307"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/CharStreams.html#copy(java.lang.Readable, java.lang.Appendable)"><tt>long copy(Readable, Appendable)</tt></a></td>
</tr>
<tr>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteStreams.html#readFully(java.io.InputStream, byte[])"><tt>void readFully(InputStream, byte[])</tt></a></td>
<td width="307">N/A</td>
</tr>
<tr>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteStreams.html#skipFully(java.io.InputStream, long)"><tt>void skipFully(InputStream, long)</tt></a></td>
<td width="307"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharStreams.html#skipFully(java.io.Reader, long)"><tt>void skipFully(Reader, long)</tt></a></td>
</tr>
<tr>
<td width="312"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteStreams.html#nullOutputStream()"><tt>OutputStream nullOutputStream()</tt></a></td>
<td width="307"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharStreams.html#nullWriter()"><tt>Writer nullWriter()</tt></a></td>
</tr>
</tbody>
</table>

**关于 InputSupplier 和 OutputSupplier 要注意：**

在 ByteStreams、CharStreams 以及 com.google.common.io 包中的一些其他类中，某些方法仍然在使用 InputSupplier 和 OutputSupplier 接口。这两个借口和相关的方法是不推荐使用的：它们已经被下面描述的 source 和 sink 类型取代了，并且最终会被移除。

## 源与汇

通常我们都会创建 I/O 工具方法，这样可以避免在做基础运算时总是直接和流打交道。例如，Guava 有 Files.toByteArray(File) 和 Files.write(File, byte[])。然而，流工具方法的创建经常最终导致散落各处的相似方法，每个方法读取不同类型的源

或写入不同类型的汇[sink]。例如，Guava 中的 Resources.toByteArray(URL)和 Files.toByteArray(File)做了同样的事情，只不过数据源一个是 URL，一个是文件。

为了解决这个问题，Guava 有一系列关于源与汇的抽象。源或汇指某个你知道如何从中打开流的资源，比如 File 或 URL。源是可读的，汇是可写的。此外，源与汇按照字节和字符划分类型。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="138"><b> </b></td>
<td width="240"><b>字节</b><b></b></td>
<td width="240"><b>字符</b><b></b></td>
</tr>
<tr>
<td width="138"><b>读</b><b></b></td>
<td width="240"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteSource.html"><tt>ByteSource</tt></a></td>
<td width="240"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/CharSource.html"><tt>CharSource</tt></a></td>
</tr>
<tr>
<td width="138"><b>写</b><b></b></td>
<td width="240"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/ByteSink.html"><tt>ByteSink</tt></a></td>
<td width="240"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/CharSink.html"><tt>CharSink</tt></a></td>
</tr>
</tbody>
</table>

源与汇 API 的好处是它们提供了通用的一组操作。比如，一旦你把数据源包装成了 ByteSource，无论它原先的类型是什么，你都得到了一组按字节操作的方法。

### 创建源与汇

Guava 提供了若干源与汇的实现：

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="276"><b>字节</b><b></b></td>
<td width="342"><b>字符</b><b></b></td>
</tr>
<tr>
<td width="276"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Files.html#asByteSource(java.io.File)"><tt>Files.asByteSource(File)</tt></a></td>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Files.html#asCharSource(java.io.File, java.nio.charset.Charset)"><tt>Files.asCharSource(File, Charset)</tt></a></td>
</tr>
<tr>
<td width="276"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Files.html#asByteSink(java.io.File, com.google.common.io.FileWriteMode...)"><tt>Files.asByteSink(File, FileWriteMode...)</tt></a></td>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Files.html#asCharSink(java.io.File, java.nio.charset.Charset, com.google.common.io.FileWriteMode...)"><tt>Files.asCharSink(File, Charset, FileWriteMode...)</tt></a></td>
</tr>
<tr>
<td width="276"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Resources.html#asByteSource(java.net.URL)"><tt>Resources.asByteSource(URL)</tt></a></td>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Resources.html#asCharSource(java.net.URL, java.nio.charset.Charset)"><tt>Resources.asCharSource(URL, Charset)</tt></a></td>
</tr>
<tr>
<td width="276"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#wrap(byte[])"><tt>ByteSource.wrap(byte[])</tt></a></td>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#wrap(java.lang.CharSequence)"><tt>CharSource.wrap(CharSequence)</tt></a></td>
</tr>
<tr>
<td width="276"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#concat(com.google.common.io.ByteSource...)"><tt>ByteSource.concat(ByteSource...)</tt></a></td>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#concat(com.google.common.io.CharSource...)"><tt>CharSource.concat(CharSource...)</tt></a></td>
</tr>
<tr>
<td width="276"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#slice(long, long)"><tt>ByteSource.slice(long, long)</tt></a></td>
<td width="342">N/A</td>
</tr>
<tr>
<td width="276">N/A</td>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#asCharSource(java.nio.charset.Charset)"><tt>ByteSource.asCharSource(Charset)</tt></a></td>
</tr>
<tr>
<td width="276">N/A</td>
<td width="342"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSink.html#asCharSink(java.nio.charset.Charset)"><tt>ByteSink.asCharSink(Charset)</tt></a></td>
</tr>
</tbody>
</table>

此外，你也可以继承这些类，以创建新的实现。

注：把已经打开的流（比如 InputStream）包装为源或汇听起来是很有诱惑力的，但是应该避免这样做。源与汇的实现应该在每次 openStream()方法被调用时都创建一个新的流。始终创建新的流可以让源或汇管理流的整个生命周期，并且让多次调用 openStream()返回的流都是可用的。此外，如果你在创建源或汇之前创建了流，你不得不在异常的时候自己保证关闭流，这压根就违背了发挥源与汇 API 优点的初衷。

### 使用源与汇

一旦有了源与汇的实例，就可以进行若干读写操作。

#### 通用操作

所有源与汇都有一些方法用于打开新的流用于读或写。默认情况下，其他源与汇操作都是先用这些方法打开流，然后做一些读或写，最后保证流被正确地关闭了。这些方法列举如下：

- openStream()：根据源与汇的类型，返回 InputStream、OutputStream、Reader 或者Writer。
- openBufferedStream()：根据源与汇的类型，返回 InputStream、OutputStream、BufferedReader 或者 BufferedWriter。返回的流保证在必要情况下做了缓冲。例如，从字节数组读数据的源就没有必要再在内存中作缓冲，这就是为什么该方法针对字节源不返回 BufferedInputStream。字符源属于例外情况，它一定返回 BufferedReader，因为 BufferedReader 中才有 readLine()方法。

#### 源操作

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="294"><b>字节源</b><b></b></td>
<td width="324"><b>字符源</b><b></b></td>
</tr>
<tr>
<td width="294"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#read()"><tt>byte[]   read()</tt></a></td>
<td width="324"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#read()"><tt>String   read()</tt></a></td>
</tr>
<tr>
<td width="294">N/A</td>
<td width="324"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#readLines()"><tt>ImmutableList&lt;String&gt;   readLines()</tt></a></td>
</tr>
<tr>
<td width="294">N/A</td>
<td width="324"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#readFirstLine()"><tt>String   readFirstLine()</tt></a></td>
</tr>
<tr>
<td width="294"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#copyTo(com.google.common.io.ByteSink)"><tt>long   copyTo(ByteSink)</tt></a></td>
<td width="324"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#copyTo(com.google.common.io.CharSink)"><tt>long   copyTo(CharSink)</tt></a></td>
</tr>
<tr>
<td width="294"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#copyTo(java.io.OutputStream)"><tt>long   copyTo(OutputStream)</tt></a></td>
<td width="324"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#copyTo(java.lang.Appendable)"><tt>long   copyTo(Appendable)</tt>   </a></td>
</tr>
<tr>
<td width="294"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#size()"><tt>long   size()</tt></a> (in bytes)</td>
<td width="324">N/A</td>
</tr>
<tr>
<td width="294"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#isEmpty()"><tt>boolean   isEmpty()</tt></a></td>
<td width="324"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSource.html#isEmpty()"><tt>boolean   isEmpty()</tt></a></td>
</tr>
<tr>
<td width="294"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#contentEquals(com.google.common.io.ByteSource)"><tt>boolean   contentEquals(ByteSource)</tt></a></td>
<td width="324">N/A</td>
</tr>
<tr>
<td width="294"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSource.html#hash(com.google.common.hash.HashFunction)"><tt>HashCode   hash(HashFunction)</tt></a></td>
<td width="324">N/A</td>
</tr>
</tbody>
</table>

#### 汇操作

<table style="width: 624px; height: 123px;" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="234"><b>字节汇</b><b></b></td>
<td width="384"><b>字符汇</b><b></b></td>
</tr>
<tr>
<td width="234"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSink.html#write(byte[])"><tt>void write(byte[])</tt></a></td>
<td width="384"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSink.html#write(java.lang.CharSequence)"><tt>void write(CharSequence)</tt></a></td>
</tr>
<tr>
<td width="234"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/ByteSink.html#writeFrom(java.io.InputStream)"><tt>long writeFrom(InputStream)</tt></a></td>
<td width="384"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSink.html#writeFrom(java.lang.Readable)"><tt>long writeFrom(Readable)</tt></a></td>
</tr>
<tr>
<td width="234">N/A</td>
<td width="384"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSink.html#writeLines(java.lang.Iterable)"><tt>void writeLines(Iterable&lt;? extends CharSequence&gt;)</tt></a></td>
</tr>
<tr>
<td width="234">N/A</td>
<td width="384"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/CharSink.html#writeLines(java.lang.Iterable, java.lang.String)"><tt>void writeLines(Iterable&lt;? extends CharSequence&gt;, String)</tt></a></td>
</tr>
</tbody>
</table>

#### 范例

```

    //Read the lines of a UTF-8 text file
    ImmutableList<String> lines = Files.asCharSource(file, Charsets.UTF_8).readLines();
    //Count distinct word occurrences in a file
    Multiset<String> wordOccurrences = HashMultiset.create(
    Splitter.on(CharMatcher.WHITESPACE)
    .trimResults()
    .omitEmptyStrings()
    .split(Files.asCharSource(file, Charsets.UTF_8).read()));
    
    //SHA-1 a file
    HashCode hash = Files.asByteSource(file).hash(Hashing.sha1());
    
    //Copy the data from a URL to a file
    Resources.asByteSource(url).copyTo(Files.asByteSink(file));
    

```

## 文件操作

除了创建文件源和文件的方法，Files 类还包含了若干你可能感兴趣的便利方法。

<table style="width: 624px; height: 93px;" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/Files.html#createParentDirs(java.io.File)"><tt>createParentDirs(File)</tt></a></td>
<td width="354">必要时为文件创建父目录</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/Files.html#getFileExtension(java.lang.String)"><tt>getFileExtension(String)</tt></a></td>
<td width="354">返回给定路径所表示文件的扩展名</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Files.html#getNameWithoutExtension(java.lang.String)"><tt>getNameWithoutExtension(String)</tt></a></td>
<td width="354">返回去除了扩展名的文件名</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/Files.html#simplifyPath(java.lang.String)"><tt>simplifyPath(String)</tt></a></td>
<td width="354">规范文件路径，并不总是与文件系统一致，请仔细测试</td>
</tr>
<tr>
<td width="264"><a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/io/Files.html#fileTreeTraverser()"><tt>fileTreeTraverser()</tt></a></td>
<td width="354">返回 TreeTraverser 用于遍历文件树</td>
</tr>
</tbody>
</table>
  
转载自[并发编程网 – ifeve.com]( 转载自并发编程网 – ifeve.com)

本文链接地址: [[Google Guava] 9-I/O](http://ifeve.com/google-guava-io/)