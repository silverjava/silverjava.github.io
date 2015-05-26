---
layout: post
title: "[10 Munites Session] 也许你还不知道的MessageFormat"
date: 2014-06-30 18:44
comments: true
categories:
---
为什么突然想到MessageFormat呢？原因是今天一个同事问我：在Spring的properties文件中，为什么有的时候单引号会被过滤掉？这个问题的答案是：Spring在处理properties时，会先判断调用时是否传入参数，如果参数为空，那么Spring将不会调用MessageFormat而直接返回字符串本身；如果参数列表不为空，那么，Spring会用MessageFormat处理消息的国际化以及占位符的替换，这个时候，由于单引号在MessageFormat是特殊字符，所以，需要同时使用两个单引号来显示。

如果仔细看看MessageFormat的文档，那么，你会发现其实它的功能不止这些，这里引用关于它的格式的定义：

{% blockquote %}
MessageFormat uses patterns of the following form:
    MessageFormatPattern:
         String
         MessageFormatPattern FormatElement String

    FormatElement:
         { ArgumentIndex }
         { ArgumentIndex , FormatType }
         { ArgumentIndex , FormatType , FormatStyle }

    FormatType: one of
        number date time choice

    FormatStyle:
        short
        medium
        long
        full
        integer
        currency
        percent
        SubformatPattern
{% endblockquote %}

###使用number

对于number最常用的场景就是格式化输出了，以前我们喜欢在Java端先把需要格式化的数字使用NumberFormat格式化为字符串，然后再传入MessageFormat。而有了
number，这件事情变的简单很多：

``` java number example

System.out.println(MessageFormat.format("{0,number,##.00}", 100)); // output is 100.00

```

实际上，你也许已经发现当我们使用number时，MessagFormat会自动调用NumberFormat来格式化我们的输出。同理适用于date，time。


###使用choice

choice是比较有趣的一个特性，它可以根据某一个参数有选择的输出。说起来比较抽象，我们来看个例子：

``` java choice example

System.out.println(MessageFormat.format(
                "There {0,choice,0#are no files|1#is one file|1<are {0,number,integer} files} in {1}.",
                new Long(10), "MyDisk"));
// output is "There are 10 files in MyDisk."

System.out.println(MessageFormat.format(
                "There {0,choice,0#are no files|1#is one file|1<are {0,number,integer} files} in {1}.",
                new Long(0), "MyDisk"));
// output is "There are no files in MyDisk."

```

应该可以发现它的奥秘了吧：）