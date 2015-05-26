---
layout: post
title: "Java 8 Lambda 初体验 - Functional Interfaces"
date: 2013-05-28 12:30
comments: true
categories: 
---

先来看看JSR335是如何定义它的：

```
A functional interface is an interface that has just one abstract method, and thus represents a single function contract. (In some cases, this "single" method may take the form of multiple abstract methods with override-equivalent signatures inherited from superinterfaces; in this case, the inherited methods logically represent a single method.)
```


这是什么意思呢？第一点很容易理解：这个接口只能有一个抽象方法。换句话说，就是一个接口只定义一个方法，那么这样的接口就可以看做是一个Functional Interface了。很简单，而且这样的接口在当前的JDK里很容易找到，比如Runnable就是一个，因为只有一个run方法。


但是，最讨厌的来了，这个定义后面还有一个很长的括号，这句话看起来不是很容易理解，那么我们来几个例子：

```
interface X { int m(Iterable<String> arg); }
interface Y { int m(Iterable<String> arg); }

interface Z extends X, Y {}
```

看这个例子，我们首先来判定X和Y是不是functional interface？肯定是，因为他们都只有一个方法。那么，Z是不是functional interface呢？我们回过头来再看看那个括号里面如何描述的：the inherited methods logically represent a single method。这下清楚了，Z应该也只有一个方法，因为这个m不管是来自X还是Y，都是一样的。所以，Z也是functional interface。

再来看个例子：

```
interface X { Iterable m(Iterable<String> arg); }
interface Y { Iterable<String> m(Iterable arg); }
interface Z extends X, Y {}
```

Z还是不是functional interface呢？不是特别清楚？那先看返回值，X.m的返回值是Iterable，而Y.m的返回值是Iterable<String>，是不是Y.m的返回值可以被X.m的返回值替换？这种情况叫做return-type-substatutable。好，接着看参数，貌似也是类似的样子，Y.m的参数是可以包含X.m的参数的，所以，我们的Z应该生成一个什么样的方法？也许是下面的：

```
Iterable m(Iterable arg);
```

或者

```
Iterable<String> m(Iterable arg);
```

具体是哪个，取决于Z.m是否只返回Iterable<String>。那么，现在我们可以做判断了，Z是不是functional interface呢？肯定是了。


到现在为止，我们也许可以把这个定义说的更直白一点，一个接口是不是functional interface，取决于：
1. 必须只有一个抽象方法
2. 如果有继承关系，这个方法还必须可以重写（override）父接口的抽象方法

好了，定义说了一大堆，来看几个实际的例子。

**数组排序** 

如果我们要对一个数组进行简单的排序，以前可以这么做：

```
String[] strArray = {"2", "1", "3"};
Arrays.sort(strArray, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.compareTo(o2);
            }
        });
```

那么现在呢？我们看到接口Comparator其实是个functional interface，那么他到底带给我们什么好处？看下面的代码：

```
String[] strArray = {"2", "1", "3"};
Arrays.sort(strArray, (o1, o2) -> o1.compareTo(o2));
```

是不是简单很多？

**自定义functional interface**

```
public interface Foo {
    void bar(String name);
}
```

我们先定义了一个这样的接口或者就叫他functional interface，那么，怎么用它？

```
Foo foo = (x) -> System.out.println("Hello " + x);
foo.bar("David");

// output is Hello David
```

是不是有点函数式的意思了，使用一个接口不需要再为它添加一个实现类了。
