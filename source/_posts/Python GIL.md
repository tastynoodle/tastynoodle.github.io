---
title: Python GIL
date: 2017-05-31 19:13:21
categories: Python
tags: Python
---

Python GIL的全称是Global Interpreter Lock，顾名思义，它是一个全局的mutex，用来保证Python中的线程安全。

既然锁是全局的，那Python是如何实现多线程的呢？很可惜，**Python只支持单线程**。

### GIL设计初衷

Python面世时，thread这个概念还未被提出，所以Python最初的设计中并没有考虑到多线程。

在之后，越来越多的C程序被port到Python里，因为C并不是thread safe的，所以Python加了GIL以确保程序里不会有race condition。

GIL确保了Python的易用性，虽然现在GIL经常被诟病，但GIL无疑是Python能够成功的原因之一。

### 为什么GIL一直被保留

虽然GIL使得Python只能以单线程运行，但它一直被保留了下来。甚至Python3都还在用着GIL，主要原因有两个：

1. GIL保证了Python的thread safe。如果在移除GIL的同时要保证thread safe，需要大量的修改，会使得易用性大打折扣
2. GIL提高了单线程的性能。因为Python只用一个锁，在单线程的情况下，和其他语言编写的程序相比，节省了acquire/release lock的时间，提高了性能。

虽然目前Python还在用着GIL，开发者依旧在积极地移除GIL，毕竟多线程是趋势。

### 并行

虽然Python只支持单线程，但如果实在需要并行处理，也是有办法的：

1. 使用multi-process而不是multi-thread。
2. 使用别的interpreter，如Jython和IronPython。GIL只适用于CPython。