# 数学运算

## 范例

```

    int logFloor = LongMath.log2(n, FLOOR);
    int mustNotOverflow = IntMath.checkedMultiply(x, y);
    long quotient = LongMath.divide(knownMultipleOfThree, 3, RoundingMode.UNNECESSARY); // fail fast on non-multiple of 3
    BigInteger nearestInteger = DoubleMath.roundToBigInteger(d, RoundingMode.HALF_EVEN);
    BigInteger sideLength = BigIntegerMath.sqrt(area, CEILING);

```

## 为什么使用 Guava Math

- Guava Math 针对各种不常见的溢出情况都有充分的测试；对溢出语义，Guava 文档也有相应的说明；如果运算的溢出检查不能通过，将导致快速失败；
- Guava Math 的性能经过了精心的设计和调优；虽然性能不可避免地依据具体硬件细节而有所差异，但 Guava Math 的速度通常可以与 Apache Commons 的 MathUtils 相比，在某些场景下甚至还有显著提升；
- Guava Math 在设计上考虑了可读性和正确的编程习惯；IntMath.log2(x, CEILING) 所表达的含义，即使在快速阅读时也是清晰明确的。而 32-Integer.numberOfLeadingZeros(x – 1)对于阅读者来说则不够清晰。

*注意：Guava Math 和 GWT 格外不兼容，这是因为 Java 和 Java Script 语言的运算溢出逻辑不一样。*

## 整数运算
Guava Math 主要处理三种整数类型：int、long 和 BigInteger。这三种类型的运算工具类分别叫做 [IntMath](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html)、[LongMath](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html) 和 [BigIntegerMath](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html)。

### 有溢出检查的运算

Guava Math 提供了若干有溢出检查的运算方法：结果溢出时，这些方法将快速失败而不是忽略溢出

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#checkedAdd(int, int)"><tt>IntMath.checkedAdd</tt></a></td>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#checkedAdd(long, long)"><tt>LongMath.checkedAdd</tt></a></td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#checkedSubtract(int, int)"><tt>IntMath.checkedSubtract</tt></a></td>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#checkedSubtract(long, long)"><tt>LongMath.checkedSubtract</tt></a></td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#checkedMultiply(int, int)"><tt>IntMath.checkedMultiply</tt></a></td>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#checkedMultiply(long, long)"><tt>LongMath.checkedMultiply</tt></a></td>
</tr>
<tr>
<td width="228"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#checkedPow(int, int)"><tt>IntMath.checkedPow</tt></a></td>
<td width="204"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#checkedPow(long, long)"><tt>LongMath.checkedPow</tt></a></td>
</tr>
</tbody>
</table>

```

    IntMath.checkedAdd(Integer.MAX_VALUE, Integer.MAX_VALUE); // throws ArithmeticException

```

## 实数运算

IntMath、LongMath 和 BigIntegerMath 提供了很多实数运算的方法，并把最终运算结果舍入成整数。这些方法接受一个 [java.math.RoundingMode](http://docs.oracle.com/javase/7/docs/api/java/math/RoundingMode.html) 枚举值作为舍入的模式：

- DOWN：向零方向舍入（去尾法）
- UP：远离零方向舍入
- FLOOR：向负无限大方向舍入
- CEILING：向正无限大方向舍入
- UNNECESSARY：不需要舍入，如果用此模式进行舍入，应直接抛出 ArithmeticException
- HALF_UP：向最近的整数舍入，其中 x.5 远离零方向舍入
- HALF_DOWN：向最近的整数舍入，其中 x.5 向零方向舍入
- HALF_EVEN：向最近的整数舍入，其中 x.5 向相邻的偶数舍入

这些方法旨在提高代码的可读性，例如，divide(x, 3, CEILING) 即使在快速阅读时也是清晰。此外，这些方法内部采用构建整数近似值再计算的实现，除了在构建 sqrt（平方根）运算的初始近似值时有浮点运算，其他方法的运算全过程都是整数或位运算，因此性能上更好。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="105"><b>运算</b><b></b></td>
<td width="141"><b>IntMath</b><b></b></td>
<td width="162"><b>LongMath</b><b></b></td>
<td width="211"><b>BigIntegerMath</b><b></b></td>
</tr>
<tr>
<td width="105">除法</td>
<td width="141"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#divide(int, int, java.math.RoundingMode)"><tt>divide(int, int, RoundingMode)</tt></a></td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#divide(long, long, java.math.RoundingMode)"><tt>divide(long, long, RoundingMode)</tt></a></td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html#divide(java.math.BigInteger, java.math.BigInteger, java.math.RoundingMode)"><tt>divide(BigInteger, BigInteger, RoundingMode)</tt></a></td>
</tr>
<tr>
<td width="105">2为底的对数</td>
<td width="141"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#log2(int, java.math.RoundingMode)"><tt>log2(int, RoundingMode)</tt></a></td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#log2(long, java.math.RoundingMode)"><tt>log2(long, RoundingMode)</tt></a></td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html#log2(java.math.BigInteger, java.math.RoundingMode)"><tt>log2(BigInteger, RoundingMode)</tt></a></td>
</tr>
<tr>
<td width="105">10为底的对数</td>
<td width="141"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#log10(int, java.math.RoundingMode)"><tt>log10(int, RoundingMode)</tt></a></td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#log10(long, java.math.RoundingMode)"><tt>log10(long, RoundingMode)</tt></a></td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html#log10(java.math.BigInteger, java.math.RoundingMode)"><tt>log10(BigInteger, RoundingMode)</tt></a></td>
</tr>
<tr>
<td width="105">平方根</td>
<td width="141"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#sqrt(int, java.math.RoundingMode)"><tt>sqrt(int, RoundingMode)</tt></a></td>
<td width="162"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#sqrt(long, java.math.RoundingMode)"><tt>sqrt(long, RoundingMode)</tt></a></td>
<td width="211"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html#sqrt(java.math.BigInteger, java.math.RoundingMode)"><tt>sqrt(BigInteger, RoundingMode)</tt></a></td>
</tr>
</tbody>
</table>

```

    // returns 31622776601683793319988935444327185337195551393252
    BigIntegerMath.sqrt(BigInteger.TEN.pow(99), RoundingMode.HALF_EVEN);

```

### 附加功能

Guava 还另外提供了一些有用的运算函数

<table width="624" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="96"><b>运算</b><b></b></td>
<td width="156"><b>IntMath</b><b></b></td>
<td width="156"><b>LongMath</b><b></b></td>
<td width="216"><b>BigIntegerMath</b><b>*</b></td>
</tr>
<tr>
<td width="96">最大公约数</td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#gcd(int, int)"><tt>gcd(int, int)</tt></a></td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#gcd(long, long)"><tt>gcd(long, long)</tt></a></td>
<td width="216"><a href="http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#gcd(java.math.BigInteger)"><tt>BigInteger.gcd(BigInteger)</tt></a></td>
</tr>
<tr>
<td width="96">取模</td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#mod(int, int)"><tt>mod(int, int)</tt></a></td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#mod(long, long)"><tt>mod(long, long)</tt></a></td>
<td width="216"><a href="http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#mod(java.math.BigInteger)"><tt>BigInteger.mod(BigInteger)</tt></a></td>
</tr>
<tr>
<td width="96">取幂</td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#pow(int, int)"><tt>pow(int, int)</tt></a></td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#pow(long, int)"><tt>pow(long, int)</tt></a></td>
<td width="216"><a href="http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#pow(int)"><tt>BigInteger.pow(int)</tt></a></td>
</tr>
<tr>
<td width="96">是否 2的幂</td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#isPowerOfTwo(int)"><tt>isPowerOfTwo(int)</tt></a></td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#isPowerOfTwo(long)"><tt>isPowerOfTwo(long)</tt></a></td>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html#isPowerOfTwo(java.math.BigInteger)"><tt>isPowerOfTwo(BigInteger)</tt></a></td>
</tr>
<tr>
<td width="96">阶乘*</td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#factorial(int)"><tt>factorial(int)</tt></a></td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#factorial(int)"><tt>factorial(int)</tt></a></td>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html#factorial(int)"><tt>factorial(int)</tt></a></td>
</tr>
<tr>
<td width="96">二项式系数*</td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/IntMath.html#binomial(int, int)"><tt>binomial(int, int)</tt></a></td>
<td width="156"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/LongMath.html#binomial(int, int)"><tt>binomial(int, int)</tt></a></td>
<td width="216"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/BigIntegerMath.html#binomial(int, int)"><tt>binomial(int, int)</tt></a></td>
</tr>
</tbody>
</table>

*BigInteger 的最大公约数和取模运算由 JDK 提供

*阶乘和二项式系数的运算结果如果溢出，则返回 MAX_VALUE

## 浮点数运算

JDK 比较彻底地涵盖了浮点数运算，但 Guava 在 DoubleMath 类中也提供了一些有用的方法。

<table width="612" border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="318"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/DoubleMath.html#isMathematicalInteger(double)"><tt>isMathematicalInteger(double)</tt></a></td>
<td width="294">判断该浮点数是不是一个整数</td>
</tr>
<tr>
<td width="318"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/DoubleMath.html#roundToInt(double, java.math.RoundingMode)"><tt>roundToInt(double, RoundingMode)</tt></a></td>
<td width="294">舍入为 int；对无限小数、溢出抛出异常</td>
</tr>
<tr>
<td width="318"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/DoubleMath.html#roundToLong(double, java.math.RoundingMode)"><tt>roundToLong(double, RoundingMode)</tt></a></td>
<td width="294">舍入为 long；对无限小数、溢出抛出异常</td>
</tr>
<tr>
<td width="318"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/DoubleMath.html#roundToBigInteger(double, java.math.RoundingMode)"><tt>roundToBigInteger(double, RoundingMode)</tt></a></td>
<td width="294">舍入为 BigInteger；对无限小数抛出异常</td>
</tr>
<tr>
<td width="318"><a href="http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/math/DoubleMath.html#log2(double, java.math.RoundingMode)"><tt>log2(double, RoundingMode)</tt></a></td>
<td width="294">2 的浮点对数，并且舍入为 int，比 JDK 的 Math.log(double) 更快</td>
</tr>
</tbody>
</table>

 转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [[Google Guava] 12-数学运算](http://ifeve.com/google-guava-math/)
