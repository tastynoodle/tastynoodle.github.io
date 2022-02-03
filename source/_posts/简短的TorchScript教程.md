---
title: 简短的TorchScript教程
draft: true
date: 2021-01-11 22:56:44
tags:
---

花了点时间读了PyTorch中的Torch Script的[官方教程](https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html)，希望能以简短的语言概括一下。

### 什么是TorchScript

简单地讲，TorchScript就是PyTorch模型的一种中间表示。我们可以将一个PyTorch模型转化成TorchScript并导出，然后运行在别的环境中（例如C++）。

在此，我想提一下ML framework常见的两种运行模式：

* Eager mode。一个例子是在Python console的代码会立即被执行，也就是写一行跑一行。PyTorch属于eager mode。
* Graph mode。类似于C++程序，代码写完后会被编译成可执行文件，然后一次性从头跑到尾。Caffe2和TensorFlow属于Graph mode。

Eager mode的好处主要在于方便调试（加断点），开发效率高。

而在graph mode下，当我们有了整个graph后，跨平台就变得很容易，而且也更容易对整体做优化。我们甚至可以开发一些算法，将一个graph拆分成多块，然后分布到多个机器上进行计算。TorchScript就是PyTorch的graph mode的实现。

### TorchScript的生成方法

PyTorch提供了两种方法生成TorchScript:

#### Scripting (`torch.jit.script`)

这也是比较推荐的一种方法。使用方法很简单 - 输入PyTorch模型，Scripting模块会生成对应的TorchScript。

#### Tracing (`torch.jit.tracing`)

和Scripting不同的是，Tracing除了需要PyTorch模型外，还需要一组sample input。Tracing模块会用sample input跑一遍模型，然后记录下所有的相关操作，并省略掉所有未执行的代码。比如说if-else语句，因为一组sample input只可能触发if和else中的一个，Tracing不会讲未执行的那部分代码编译到最后生成的TorchScript里。所以Tracing适用于不怎么有control flow的模型，或者说是对于性能要求比较苛刻的场景。

#### 混合模式

除了使用Scripting或者Tracing外，我们还可以讲两种模式混合使用。