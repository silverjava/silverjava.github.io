---
layout: post
title: "[10 minutes session]如何替换默认值类型的问号表达式"
date: 2014-06-06 22:25
comments: true
categories: 
---
很多种编程语言都提供了问号表达式，它是一种if-else的简易替代方案，我们可以暂且叫它选择性问号表达式。如果你的判断逻辑十分简单，比如像下面的代码：

```
String nextAction = hasMoney ? "travel" : "stay at home";
```

而还有一类问号表达式，它更多是在表达：如果它本身满足不为空，就返回自己；否则，返回一个默认值。例如：

```
String nextAction = userAction == null ? "play" : userAction;
```

对于这样的问题，用问号表达式就略显多余，而且代码的表达性也不是很好。

Guava中提供了几个方案可以让我们更方便的处理这种问题：Objects.firstNotNull()和Optional。

我们分别来看如何用这两种方法重写上面的代码：

```
// Objects.firstNotNull
String nextAction = Objects.firstNotNull(userAction, "play");

// Optional
String nextAction = Optional.fromNullable(userAction).or("play");
```

如果再使用static import，就变成了：

```
// Objects.firstNotNull
String nextAction = firstNotNull(userAction, "play");

// Optional
String nextAction = fromNullable(userAction).or("play");
```

简单明了，易于阅读。
