# 引言

Guava 工程包含了若干被 Google 的 Java 项目广泛依赖 的核心库，例如：集合 [collections] 、缓存 [caching] 、原生类型支持 [primitives support] 、并发库 [concurrency libraries] 、通用注解 [common annotations] 、字符串处理 [string processing] 、I/O 等等。 所有这些工具每天都在被 Google 的工程师应用在产品服务中。

查阅 Javadoc 并不一定是学习这些库最有效的方式。在此，我们希望通过此文档为 Guava 中最流行和最强大的功能，提供更具可读性和解释性的说明。


**译文格式说明**

- Guava 中的类被首次引用时，都会链接到 Guava 的 API 文档。如：[Optional<T>](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html)。
- Guava 和 JDK 中的方法被引用时，一般都会链接到 Guava 或 JDK 的 API 文档，一些人所共知的 JDK 方法除外。如：[Optional.of(T)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/base/Optional.html#of(T)), Map.get(key)。
- 译者对文档的额外说明以斜体显示，并且以“译者注：”开始。