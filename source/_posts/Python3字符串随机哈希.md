---
title: Python3字符串随机哈希
date: 2019-09-13 12:20:33
categories: Python
tags: Python
---

最近遇到了一个由于字符串随机哈希引起的问题，在此记录一下。

#### 起因
我们有个基于java的service，在进行了一次新的deployment后，有一部分的container挂了。从error msg来看，似乎是有些jvm property的property和value位置互换了。


#### Root Cause

出于某些原因，我们在配置jvm的时候，是先把所有的property和value写在一个Python的dictonary里，然后再写回到一个config file。

每当我们需要加一个新的property，就会做

```
dict.update([(property, value)])
```

但是在一个新加的property里，由于人为的因素，有个括号被误写成了花括号，原来的tuple就变成了set

```
dict.update([{property, value}])
```

这种情况下，Python会把set返回的第一个值当成property，第二个值当成value。由于Python中set的返回顺序不固定，就造成了在一部分的container中，dict被解析成了正确的config，而在另一部分container中，dict被解析成了错误的config。

Fix就很简单了，把花括号改成普通括号就行了。

#### 为什么set返回顺序不固定
set简单地讲就是一个hash table，当有一个新的element被加进来时，Python直接call这个element的`__hash__()`来取哈希值，然后加入hash table里。

在Python3里，string object的哈希返回值是不固定的。

#### 为什么string object的哈希值不固定
Python的随机哈希，简单地说就是对普通哈希进行了加盐。原因主要是为了防止Dos攻击。试想如果hash算法是公开的，那么我们可以精心设计一些值，使得他们拥有相同的哈希值。这样的话，在执行Python程序时，set的操作效率就会变成O(n^2)。这对于web application来说无疑是致命的。

#### 关闭随机哈希
Python在每次启动解释器时，都会设置一个random seed。所以在同一个解释器进程里，相同字符串的哈希值总是固定的。但每当另起一个解释器时，seed就会被重置，字符串的哈希值也会随之改变。

想要关闭随机哈希，只要把`PYTHONHASHSEED`这个环境变量设置成`0`就可以了（默认是`random`）。其实在Python2.x版本里，这个环境变量的默认值就是`0`。


