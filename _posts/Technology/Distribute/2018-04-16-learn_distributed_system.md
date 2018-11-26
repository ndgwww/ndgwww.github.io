---

layout: post
title: 如何学习分布式系统
category: 技术
tags: Distribute
keywords: 分布式系统

---

## 简介

* TOC
{:toc}

本文主要聊一下 分布式系统（尤其指分布式计算系统）的 基本组成和理论问题

## 从哪里入手

到目前为止，笔者关于分布式这块的博客

* [现有分布式项目小结](http://qiankunli.github.io/2015/07/14/distributed_project.html)
* [分布式配置系统](http://qiankunli.github.io/2015/08/08/distributed_configure_system.html)
* [分布式事务](http://qiankunli.github.io/2017/07/18/distributed_transaction.html)

有没有一个 知识体系/架构、描述方式，能将分布式的各类知识都汇总起来？

![](/public/upload/distribute/study_distribute_system.png)

可以参见[《左耳听风》笔记](http://qiankunli.github.io/2018/09/08/zuoertingfeng_note.html) 看下陈皓大牛对分布式系统的阐述（貌似更多集中在应用与运维层次）

### 按层次来划分

笔者认为，从上到下，主要有以下几块：

1. 分布式应用系统

	* 比如spark、storm 这些，计算逻辑编写完毕后，在集群范围内分发、执行和监控。设计之初，就是基于集群实现。
	* 微服务，应用分别启动。另有监控、日志收集等旁路系统。更多是基于并行开发部署、模块复用等角度从单机拆成分布式的。

	PS：进而演化出paas平台，屏蔽多机物理资源，提供统一抽象，广泛的支持各种应用系统的运行。
2. 分布式中间件，比如zookeeper、kafka这些。将单机多线程/进程间的通信扩展到多机，**使得虽然跑在多机上，但可以彼此通信（共享内存、管道等）和交互。**要不怎么说叫“中间”件呢。
3. 分布式基本原理，包括共识算法等

### 从特征来学习

[分布式学习最佳实践：从分布式系统的特征开始（附思维导图）](https://www.cnblogs.com/xybaby/p/8544715.html) 作者分享了“如何学习分布式系统”的探索历程， 深有体会。

**当一个知识点很复杂时，如何学习它，也是一门学问**

一个大型网站就是一个分布式系统，包含诸多组件，每一个组件也都是一个分布式系统，比如分布式存储就是一个分布式系统，消息队列就是一个分布式系统。

为什么说从思考分布式的特征出发，是一个可行的、系统的、循序渐进的学习方式呢？

1. 先有问题，才会去思考解决问题的办法
2. 解决一个问题，常常会引入新的问题。比如提高可用性 ==> 影响可用性的因素/冗余 ==> 一致性问题 ==> 可靠节点和不可靠节点的一致性问题
3. **这是一个深度优先遍历的过程**（按层次应该算广度优先遍历——知道所有组件，然后找共同点），在这个过程中我们始终知道

	* 自己已经掌握了哪些知识；
	* 还有哪些是已经知道，但未了解的知识；
	* 哪些是空白

脑图来自作者

![](/public/upload/distribute/study_distribute_system_from_feature.png)

## 大咖文章

2018.11.25 补充 [可能是讲分布式系统最到位的一篇文章](http://www.10tiao.com/html/46/201811/2651011019/1.html)回答了几个问题：

1. “分布式系统”等于 SOA、ESB、微服务这些东西吗？
2. “分布式系统”是各种中间件吗？中间件起到的是标准化的作用。**中间件只是承载这些标准化想法的介质、工具**，可以起到引导和约束的效果，以此起到大大降低系统复杂度和协作成本的作用。为了在软件系统的迭代过程中，避免将精力过多地花费在某个子功能下众多差异不大的选项中。
3. 海市蜃楼般的“分布式系统”

我们先思考一下“软件”是什么。 软件的本质是一套代码，而代码只是一段文字，除了提供文字所表述的信息之外，本身无法“动”起来。但是，想让它“动”起来，使其能够完成一件我们指定的事情，前提是需要一个宿主来给予它生命。这个宿主就是计算机，它可以让代码变成一连串可执行的“动作”，然后通过数据这个“燃料”的触发，“动”起来。这个持续的活动过程，又被描述为一个运行中的“进程”。

所以，“单程序 + 单数据库”为什么也是分布式系统这个问题就很明白了。因为我们所编写的程序运行时所在的进程，和程序中使用到的数据库所在的进程，并不是同一个。也因此导致了，让这两个进程（系统）完成各自的部分，而后最终完成一件完整的事，变得不再像由单个个体独自完成这件事那么简单。所以，我们可以这么理解，涉及多个进程协作才能提供一个完整功能的系统就是“分布式系统”。


希望你在学习分布式系统的时候，不要因追逐“术”而丢了“道”。没有“道”只有“术”是空壳，最终会走火入魔，学得越多，会越混乱，到处都是矛盾和疑惑。

[分布式系统的本质其实就是这两个问题](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651011140&idx=1&sn=37b734deb9523dbde221708baa43fb39&chksm=bdbec0178ac9490102e6072967092b5a04445bbe8f2bcf95a154f4e5d7eaf1717a342e7650b5&scene=27#wechat_redirect)

1. 分治
2. 冗余，为了提高可靠性
3. 再连接，如何将拆分后的各个节点再次连接起来，从模式上来说，主要是去中心化与中心化之分。前者完全消除了中心节点故障带来的全盘出错的风险，却带来了更高的节点间协作成本。后者通过中心节点的集中式管理大大降低了协作成本，但是一旦中心节点故障则全盘出错。

## 分布式计算系统的 几个套路

参见[Spark Stream 学习](http://qiankunli.github.io/2018/05/27/spark_stream.html)  中对spark stream 和storm 对比一节，有以下几点：

1. 分布式计算系统，都是用户以代码的方式预定义好计算逻辑，系统将计算 下发到各个节点。这一点都是一样的，不同处是对外提供的抽象不同。比如`rdd.filter(function1).map(function2)`，而在storm 中则可能是 两个bolt
2. 有的计算 逻辑在一个节点即可执行完毕，比如不涉及分区的spark rdd，或分布式运行一个shell。有的计算逻辑则 拆分到不同节点，比如storm和mapreduce，“分段”执行。此时系统就要做好 调度和协调。
3. 分布式系统，总是要涉及到输入源数据的读取、数据在节点间流转、将结果写到输出端。

## 学习分布式的正确姿势（old）

2018.7.16 补充 [漫谈分布式系统、拜占庭将军问题与区块链](http://zhangtielei.com/posts/blog-consensus-byzantine-and-blockchain.html) 作者阐述了分布式系统的核心问题和概念，沿着逻辑上前后一贯的思路，讨论了区块链技术。推荐阅读。

### 不要沉迷与具体的算法

[distributed-systems-theory-for-the-distributed-systems-engineer](http://the-paper-trail.org/blog/distributed-systems-theory-for-the-distributed-systems-engineer/) 文中提到：

My response of old might have been “well, here’s the FLP paper, and here’s the Paxos paper, and here’s the Byzantine generals（拜占庭将军） paper…”, and I’d have prescribed(嘱咐、规定) a laundry list of primary source material which would have taken at least six months to get through if you rushed. **But I’ve come to thinking that recommending a ton of theoretical papers is often precisely the wrong way to go about learning distributed systems theory (unless you are in a PhD program).** Papers are usually deep, usually complex, and require both serious study, and usually significant experience to glean(捡拾) their important contributions and to place them in context. What good is requiring that level of expertise of engineers?

也就是说，具体学习某一个分布式算法用处有限。一个很难理解，一个是你很难  place them in contex（它们在解决分布式问题中的作用）。

### 分布式与一致性

李运华 《从0到1学架构》 关于Robert Greiner 两篇文章的对比 建议细读，要点如下

1. 不是所有的分布式系统都有 cap问题，必须interconnected 和 share data。比如一个简单的微服务系统 没有shar data，便没有cap 问题。
2. 强调了write/read pair 。这跟上一点是一脉相承的。cap 关注的是对数据的读写操作，而不是分布式系统的所有功能。


想要沉迷，可以查看梳理的博客：[串一串一致性协议](http://qiankunli.github.io/2018/09/27/consistency_protocol.html)

## 分布式知识体系

[distributed-systems-theory-for-the-distributed-systems-engineer](http://the-paper-trail.org/blog/distributed-systems-theory-for-the-distributed-systems-engineer/) 

1. Many difficulties that the distributed systems engineer faces can be blamed on two underlying causes:

	* processes may fail
	* there is no good way to tell that they have done so

2. The basic tension of fault tolerance。fault 有两种级别

	* 节点失效
	* 节点返回错误数据（对应拜占庭将军问题中的 叛徒）
3. basic primitives

	* Leader election
	* Consistent snapshotting
	* Consensus
	* Distributed state machine replication

	
问题就是，你如何将 一致性、共识 这些概念 place 到 分布式的 context中。


![](/public/upload/architecture/distributed_system.png)
