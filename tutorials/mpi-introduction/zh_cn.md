---
layout: post
title: MPI 教程介绍
author: Wes Kendall
categories: Beginner MPI
tags:
redirect_from: '/mpi-introduction/zh_cn'
---

分布式计算现在对于我们来说，就跟日常生活里的手机和电脑一样普及。你很明显应该认同这个观点，因为你发现了这个了不起的 MPI 教程网站！不管你是出于什么原因想学习并行编程（parallel programming），或者说分布式编程、并行编程，也许是因为课程需要，或者是工作，或者单纯地觉得好玩，我觉得你都应该选择一项在未来几年依然十分有价值的技术去学习。我觉得「消息传递接口」（Message Passing Interface, MPI）就是这样一项技术，而且学习它确实可以让你的并行编程知识变得更深厚。尽管 MPI 比大多数并行框架要更底层（比如 Hadoop），但是学习 MPI 会为你的并行编程打下良好的基础。

在我开始介绍 MPI 之前，我想要解释下我为什么做这个教程。当我在读研究生的时候，我大量的用到了 MPI。当我在 [Argonne National Laboratory](http://www.anl.gov) 实习的时候，我很幸运地可以跟 MPI 社区里很厉害的一些人一起工作，并且使用 MPI 在庞大的超级计算（supercomputing）集群上面做了很多疯狂的事情。然而，即使有这些资源和懂行的人可以问，我还是觉得学习 MPI 是件苦差事。

对我来说学习 MPI 很难主要是因为以下三个方面。第一，网上关于 MPI 的资料几乎都是过时的，或者不那么全的。第二，我想要自己简单地搭建一个可以运行 MPI 的集群环境，但是找不到这样的教程。最后，我读研究生的时候能买到的最便宜的关于 MPI 的书要60美元 - 对研究生来说太贵了。就目前分布式编程对我们生活的重要性来说，我觉得提供更好的教程能让别人学习到最基础的分布式协议之一同样重要。

尽管我不敢自称是 MPI 专家，我觉得以简单已读的教程形式传播这些我在研究生阶段学习到的知识还是有必要的，最重要的事，你可以根据教程在*你自己*的集群上运行 MPI 程序！我希望这个教程能对你所有帮助，也许是事业上的，也许是学习上的，或者可能是生活上的帮助 - 因为分布式编程不仅仅意味着现在，它*还是*未来！

## MPI 的历史简介
在 90 年代之前，程序员可没我们这么幸运。对于不同的计算架构写并发程序是一件困难而且冗长的事情。当时，很多软件库可以帮助写并发程序，但是没有一个大家都接受的标准来做这个事情。

在当时，大多数的并发程序只出现在科学和研究的领域。最广为接受的模型就是消息传递模型。什么是消息传递模型？它其实只是指程序通过在进程间传递消息（消息可以理解成带有一些信息和数据的一个数据结构）来完成某些任务。在实践中，并发程序用这个模型去实现特别容易。举例来说，主进程（master process）可以通过对从进程（slave process）发送一个描述工作的消息来把这个工作分配给它。另一个例子就是一个并发的排序程序可以在当前进程中对当前进程可见的（我们称作本地的，locally）数据进行排序，然后把排好序的数据发送的邻居进程上面来进行合并的操作。几乎所有的并行程序可以使用消息传递模型来描述。

由于当时很多软件库都用到了这个消息传递模型，但是在定义上有些微小的差异，这些库的作者以及一些其他人为了解决这个问题就在 Supercomputing 1992 大会上定义了一个消息传递接口的标准- 也就是 MPI。这个标准接口使得程序员写的并发程序可以在所有主流的并发框架中运行。并且允许他们可以使用当时已经在使用的一些流行库的特性和模型。

到 1994 年的时候，一个完整的接口标准定义好了（MPI-1）。我们要记住 MPI *只是*一个接口的定义而已。然后需要程序员去根据不同的架构去实现这个接口。很幸运的是，仅仅一年之后，一个完整的 MPI 实现就已经出现了。在第一个实现之后，MPI 就被大量地使用在消息传递应用程序中，并且依然是写这类程序的*标准*（de-facto）。


![An accurate representation of the first MPI programmers.](../90s_nerd.jpg)
*An accurate representation of the first MPI programmers.*

## MPI 对于消息传递模型的设计
在开始教程之前，我会先解释一下 MPI 在消息传递模型设计上的一些经典概念。第一个概念是*通讯器*（communicator）。通讯器定义了一组能够互相发消息的进程。在这组进程中，每个进程会被分配一个序号，称作*秩*（rank），进程间显性地通过指定秩来进行通信。

通信的基础建立在不同进程间发送和接收操作。一个进程可以通过指定另一个进程的秩以及一个独一无二的消息*标签*（*tag*）来发送消息给另一个进程。接受者可以发送一个接收特定标签标记的消息的请求（或者也可以完全不管标签，接收任何消息），然后依次处理接收到的数据。类似与这样的涉及一个发送者以及一个接受者的通信被称作*点对点*（point-to-point）通信。


当然在很多情况下，某个进程可能需要更所有其他进程通信。比如主进程想发一个广播给所有的从进程。在这种情况下，手动去写一个个进程点对点的信息传递就显得很笨拙。而且事实上这样会导致网络利用率低下。MPI 有专门的接口来帮我们处理这类所有进程间的*集体性*（collective）通信。

把点对点通信和集体性通信这两个机制合在一起已经可以创造十分复杂的并发程序了。事实上，这两个功能已经强大到我现在不需要再介绍任何 MPI 高级的特性了，我会把那些放到后面的教程中。现在，我们可以从[在单机上安装 MPI]({{ site.baseurl }}/tutorials/installing-mpich2/)或 [启动一个 Amazon EC2 MPI 集群]({{ site.baseurl }}/tutorials/launching-an-amazon-ec2-mpi-cluster/) 开始我们的 MPI 旅途了！如果你已经把 MPI 装好了，那太好了，直接开始[MPI Hello World 课程]({{ site.baseurl }}/tutorials/mpi-hello-world)这个吧。
