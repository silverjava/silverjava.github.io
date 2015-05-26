---
layout: post
title: "[10 Minutes Session]Guava与防御式编程"
date: 2014-06-11 20:28
comments: true
categories: 
---
第一次接触防御式编程这个概念还是上大学时候，当时花了99块钱买了一本《代码大全》。其中有一章就是介绍什么是防御式编程，下面就是书中对于它的描述：

```
“Defensive programming doesn't mean being defensive about your programming—"It does so work!”
```

简单来说，采用防御式编程的原因是我们无法控制调用者如何使用我们提供的接口，那么，我们就要做到即使是垃圾数据进来，我的程序也出淤泥而不染。

在Java世界，曾经我们可以使用断言、异常以及条件判断等方式增强程序的健壮性，然而，现在，Guava提供更好的方式帮助我们设计接口和实现接口。

###使用Opional设计API

在Guava中，我们使用Optional的原因是避免NullPointerException（NPE），然后，如果我们做这样的约定俗成：如果参数或返回值并没有使用Optional包裹，那么我们认为它们一定不为Null。例如：

```
Result playMusic(Optional<Playlist> listOptional) {
	if (listOptional.isPresent()) {
		return playMusic(listOptional.get());
	}
	return new Result("Nothing to play");
}
```

根据之前的约定，当我们在阅读这个接口的时候，就可以认定返回Result一定不可能Null，同时Playlist是可以为空的，这个时候它也许代表播放列表为空。

而作为调用者：

```
// nothing to play
Result result = playMusic(Optional.absent());

// or
Result result = playMusic(Optional.fromNullable(playlist)); 

// print the message directly and no need to care about NPE
System.out.println(result.getMessage());
```

###使用Preconditions做条件检查

Preconditions可以帮助我们做很多种条件检查，比如我们对上述接口做一定的修改：

```
Result playMusic(Playlist list) {
	Preconditions.checkNotNull(list, "Playlist MUST NOT be NULL.")
	return playMusic(listOptional.get());
}
```

Preconditions还提供了更多的条件检查：

* checkArgument
* checkElementIndex
* checkState
* checkPositionIndex
