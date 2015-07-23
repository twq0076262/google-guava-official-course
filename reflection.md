# google Guava 包的 reflection 解析

译者：[万天慧](http://weibo.com/beneo)(武祖)

由于类型擦除，你不能够在运行时传递泛型类对象——你可能想强制转换它们，并假装这些对象是有泛型的，但实际上它们没有。

举个例子：

```

    ArrayList<String> stringList = Lists.newArrayList();
    ArrayList<Integer> intList = Lists.newArrayList();
    System.out.println(stringList.getClass().isAssignableFrom(intList.getClass()));
    returns true, even though ArrayList<String> is not assignable from ArrayList<Integer>

```

Guava 提供了 [TypeToken](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/reflect/TypeToken.html), 它使用了基于反射的技巧甚至让你在运行时都能够巧妙的操作和查询泛型类型。想象一下 TypeToken 是创建，操作，查询泛型类型（以及，隐含的类）对象的方法。

Guice 用户特别注意：TypeToken 与类 [Guice](http://code.google.com/p/google-guice/) 的 [TypeLiteral](http://google-guice.googlecode.com/git/javadoc/com/google/inject/TypeLiteral.html) 很相似，但是有一个点特别不同：它能够支持非具体化的类型，例如 T，List<T>，甚至是 List<? extends Number>；TypeLiteral 则不能支持。TypeToken 也能支持序列化并且提供了很多额外的工具方法。

## 背景：类型擦除与反射

Java 不能在运行时保留对象的泛型类型信息。如果你在运行时有一个 ArrayList<String> 对象，你不能够判定这个对象是有泛型类型 ArrayList<String> 的 —— 并且通过不安全的原始类型，你可以将这个对象强制转换成 ArrayList<Object>。

但是，反射允许你去检测方法和类的泛型类型。如果你实现了一个返回 List 的方法，并且你用反射获得了这个方法的返回类型，你会获得代表 List<String> 的 ParameterizedType。

TypeToken 类使用这种变通的方法以最小的语法开销去支持泛型类型的操作。

## 介绍

获取一个基本的、原始类的 TypeToken 非常简单：

```

    TypeToken<String> stringTok = TypeToken.of(String.class);
    TypeToken<Integer> intTok = TypeToken.of(Integer.class);

```

为获得一个含有泛型的类型的 TypeToken —— 当你知道在编译时的泛型参数类型 —— 你使用一个空的匿名内部类：

```

    TypeToken<List<String>> stringListTok = new TypeToken<List<String>>() {};

```

或者你想故意指向一个通配符类型：

```

    TypeToken<Map<?, ?>> wildMapTok = new TypeToken<Map<?, ?>>() {};

```

TypeToken 提供了一种方法来动态的解决泛型类型参数，如下所示：

```


    static <K, V> TypeToken<Map<K, V>> mapToken(TypeToken<K> keyToken, TypeToken<V> valueToken) {
	    return new TypeToken<Map<K, V>>() {}
	        .where(new TypeParameter<K>() {}, keyToken)
	        .where(new TypeParameter<V>() {}, valueToken);
    }
    ...
    TypeToken<Map<String, BigInteger>> mapToken = mapToken(
	    TypeToken.of(String.class),
	    TypeToken.of(BigInteger.class)
    );
    TypeToken<Map<Integer, Queue<String>>> complexToken = mapToken(
       TypeToken.of(Integer.class),
       new TypeToken<Queue<String>>() {}
    );

```


注意如果 mapToken 只是返回了 new TypeToken>()，它实际上不能把具体化的类型分配到 K 和 V 上面，举个例子

```
    
    class Util {
    static <K, V> TypeToken<Map<K, V>> incorrectMapToken() {
    return new TypeToken<Map<K, V>>() {};
    }
    }
    System.out.println(Util.<String, BigInteger>incorrectMapToken());
    // just prints out "java.util.Map<K, V>"

```


或者，你可以通过一个子类（通常是匿名）来捕获一个泛型类型并且这个子类也可以用来替换知道参数类型的上下文类。

```

    abstract class IKnowMyType<T> {
    TypeToken<T> type = new TypeToken<T>(getClass()) {};
    }
    ...
    new IKnowMyType<String>() {}.type; // returns a correct TypeToken<String>
    
```


使用这种技术，你可以，例如，获得知道他们的元素类型的类。

## 查询

TypeToken 支持很多种类能支持的查询，但是也会把通用的查询约束考虑在内。

支持的查询操作包括：

<table>
<thead>
<tr>
<td>方法</td>
<td>描述</td>
</tr>
</thead>
<tbody>
<tr>
<td>getType()</td>
<td>获得包装的 java.lang.reflect.Type.</td>
</tr>
<tr>
<td>getRawType()</td>
<td>返回大家熟知的运行时类</td>
</tr>
<tr>
<td>getSubtype(Class&lt;?&gt;)</td>
<td>返回那些有特定原始类的子类型。举个例子，如果这有一个 Iterable 并且参数是 List.class，那么返回将是 List。</td>
</tr>
<tr>
<td>getSupertype(Class&lt;?&gt;)</td>
<td>产生这个类型的超类，这个超类是指定的原始类型。举个例子，如果这是一个 Set 并且参数是Iterable.class，结果将会是 Iterable。</td>
</tr>
<tr>
<td>isAssignableFrom(type)</td>
<td>如果这个类型是 assignable from 指定的类型，并且考虑泛型参数，返回 true。List&lt;? extends Number&gt;是 assignable from List，但 List 没有.</td>
</tr>
<tr>
<td>getTypes()</td>
<td>返回一个 Set，包含了这个所有接口，子类和类是这个类型的类。返回的 Set 同样提供了 classes()和 interfaces()方法允许你只浏览超类和接口类。</td>
</tr>
<tr>
<td>isArray()</td>
<td>检查某个类型是不是数组，甚至是&lt;? extends A[]&gt;。</td>
</tr>
<tr>
<td>getComponentType()</td>
<td>返回组件类型数组。</td>
</tr>
</tbody>
</table>

### resolveType

resolveType 是一个可以用来“替代”context token（译者：不知道怎么翻译，只好去 stackoverflow 去问了）中的类型参数的一个强大而复杂的查询操作。例如，

```

    TypeToken<Function<Integer, String>> funToken = new TypeToken<Function<Integer, String>>() {};

    TypeToken<?> funResultToken = funToken.resolveType(Function.class.getTypeParameters()[1]));
    // returns a TypeToken<String>

```

TypeToken 将 Java 提供的 TypeVariables 和 context token 中的类型变量统一起来。这可以被用来一般性地推断出在一个类型相关方法的返回类型：

```

    TypeToken<Map<String, Integer>> mapToken = new TypeToken<Map<String, Integer>>() {};
    TypeToken<?> entrySetToken = mapToken.resolveType(Map.class.getMethod("entrySet").getGenericReturnType());
    // returns a TypeToken<Set<Map.Entry<String, Integer>>>

```

### Invokable

Guava 的 Invokable 是对 java.lang.reflect.Method 和 java.lang.reflect.Constructor 的流式包装。它简化了常见的反射代码的使用。一些使用例子：

**方法是否是 public 的?**

JDK:

```

    Modifier.isPublic(method.getModifiers())

```

Invokable:

```

    invokable.isPublic()

```

**方法是否是 package private?**

JDK:

```

     !(Modifier.isPrivate(method.getModifiers()) || Modifier.isPublic(method.getModifiers()))

```

Invokable:

```

    invokable.isPackagePrivate()

```

**方法是否能够被子类重写？**

JDK:

```

    !(Modifier.isFinal(method.getModifiers())
    || Modifiers.isPrivate(method.getModifiers())
    || Modifiers.isStatic(method.getModifiers())
    || Modifiers.isFinal(method.getDeclaringClass().getModifiers()))

```


Invokable:

```

    invokable.isOverridable()

```

**方法的第一个参数是否被定义了注解@Nullable？**

JDK:

```

    for (Annotation annotation : method.getParameterAnnotations[0]) {
      if (annotation instanceof Nullable) {
        return true;
      }
    }
    return false;

```

Invokable:

```

    invokable.getParameters().get(0).isAnnotationPresent(Nullable.class)

```

**构造函数和工厂方法如何共享同样的代码？**

你是否很想重复自己，因为你的反射代码需要以相同的方式工作在构造函数和工厂方法中？

Invokable 提供了一个抽象的概念。下面的代码适合任何一种方法或构造函数：

```

    invokable.isPublic();
    invokable.getParameters();
    invokable.invoke(object, args);

```


List 的 List.get(int)返回类型是什么？

Invokable 提供了与众不同的类型解决方案：

```

    Invokable<List<String>, ?> invokable = new TypeToken<List<String>>(){}.method(getMethod);
    invokable.getReturnType(); // String.class

```
    
### Dynamic Proxies
    
**newProxy()**

实用方法 Reflection.newProxy(Class, InvocationHandler)是一种更安全，更方便的 API，它只有一个单一的接口类型需要被代理来创建 Java 动态代理时

JDK:

```

    Foo foo = (Foo) Proxy.newProxyInstance(
    Foo.class.getClassLoader(),
    new Class<?>[] {Foo.class},
    invocationHandler);

```

Guava:

```

    Foo foo = Reflection.newProxy(Foo.class, invocationHandler);

```


**AbstractInvocationHandler**

有时候你可能想动态代理能够更直观的支持 equals()，hashCode()和 toString()，那就是：

1. 一个代理实例 equal 另外一个代理实例，只要他们有同样的接口类型和 equal 的 invocation handlers。
2. 一个代理实例的 toString()会被代理到 invocation handler 的 toString()，这样更容易自定义。

[AbstractInvocationHandler](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/reflect/AbstractInvocationHandler.html) 实现了以上逻辑。

除此之外，AbstractInvocationHandler 确保传递给 <a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/reflect/AbstractInvocationHandler.html#handleInvocation(java.lang.Object,%20java.lang.reflect.Method,%20java.lang.Object%5B%5D">handleInvocation(Object, Method, Object[])</a>)的参数数组永远不会空，从而减少了空指针异常的机会。

### ClassPath

严格来讲，Java 没有平台无关的方式来浏览类和类资源。不过一定的包或者工程下，还是能够实现的，比方说，去检查某个特定的工程的惯例或者某种一直遵从的约束。

[ClassPath](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/reflect/ClassPath.html) 是一种实用工具，它提供尽最大努力的类路径扫描。用法很简单：

```

    ClassPath classpath = ClassPath.from(classloader); // scans the class path used by classloader
    for (ClassPath.ClassInfo classInfo : classpath.getTopLevelClasses("com.mycomp.mypackage")) {
      ...
    }

```

在上面的例子中，[ClassInfo](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/reflect/ClassPath.ClassInfo.html) 是被加载类的句柄。它允许程序员去检查类的名字和包的名字，让类直到需要的时候才被加载。

值得注意的是，ClassPath 是一个尽力而为的工具。它只扫描jar文件中或者某个文件目录下的 class 文件。也不能扫描非 URLClassLoader 的自定义 class loader 管理的 class，所以不要将它用于关键任务生产任务。

### Class Loading

工具方法 Reflection.initialize(Class…)能够确保特定的类被初始化——执行任何静态初始化。

使用这种方法的是一个代码异味，因为静态伤害系统的可维护性和可测试性。在有些情况下，你别无选择，而与传统的框架，操作间，这一方法有助于保持代码不那么丑。

转载自[并发编程网 – ifeve.com](http://ifeve.com/)

本文链接地址: [google Guava 包的 reflection 解析](http://ifeve.com/guava-reflection/)