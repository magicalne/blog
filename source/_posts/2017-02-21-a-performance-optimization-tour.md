title: 一次性能优化之旅
layout: post
date: 2017-02-21
tag:
- GC tuning
- java
---

近期对我所开发的规则引擎进行了性能调优，过程很好玩！

### 起因

起因是，之前有对接的同事询问我**规则引擎**的tps大约是多少。之前也没有压测过，当场也说不出来，回头我拿自己的mbp测了一下，tps=~44。后来我就跟对面的测试哥说了一下这件事，测试哥找了台机器压测了一顿，tps大约也是44的样子。由于生产环境没有出现瓶颈的征兆，我就本打算不管这事了。幸亏测试哥经验丰富。。。顺便又观察了一下GC，才发现了一个天大的秘密！

### 现象

我们的规则引擎在执行过程中会频繁的进行full GC。且full GC并不是有效的，就是说GC过后内存没有被回收，而且停顿时间明显，系统响应性越来越差，慢慢死掉。。。典型的雪崩。

### 第一次尝试

当时看代码发现了有两处实现的有问题，构成了两个朝生西死的大对象。干掉之后，压测结果好看了一些。但依然没有解决频繁full GC且GC无效的问题。

### 第二次尝试

由于我使用了guava cache，然后就思考会不会跟cache有关呢？重新看guava cache源码的时候，发现自己配制cache的地方是有问题的。然后更改了配置项，压测后结果又好了一点，但是仍然没解决问题。

### 第三次尝试

我开始思考，是不是这些大对象直接进入老年代，而且full GC之后依然存在的原因，是因为这些对象真的没有被使用完？

我们用的是tomcat，tomcat会对每一个connection建立一个线程，这些都是要消耗大量内存的。如果full GC的时候这些请求没有被处理完（因为处理慢），那这些线程也自然无法释放。但是jetty默认却是单线程处理connection的，如果connection可以每次不被分配一个线程，那么自然也就不需要消耗那么多内存了。而且，我当时认为，测试的时候只有一个tcp连接，这个connection应该被重用的。

结果是，jetty无效。后来跟测试哥沟通才发现，测试哥发请求的时候，真的是并发的。即每次使用不同的端口号发请求，这样就导致虽然是两台机器建立tcp连接，但是由于发出端的端口号不同，所建立的connection也是不同的，不能复用，因此jetty并不能解决这个并发场景下的连接问题。

### 第四次尝试

我开始思考，如果根本没有对象进入老年代呢？这可是规则引擎啊，用完就走的系统，我们当时测试使用的请求，确实没有啥朝生西死的大对象。那到底是什么在触发full GC呢？

首先我尝试调大heap大小，压测效果提升明显，但是仍然频繁full GC。

换G1！屌起了！tps稳定在200，且响应分布均匀，没有timeout，基本没有full GC，minor GC十分高效。配合设置停顿时间为200ms，整个系统焕然一新。

### 结论

到这里，原因都清楚了。规则引擎使用Java 8，Java 8默认GC算法使用CMS。CMS的minor GC最大的问题就是不能处理framentation，因为CMS没有compact的操作。在高并发的执行规则的时候，会创建很多小对象，minor GC过后会回收这部分对象，但是由于没有compact，heap上有很多碎片。导致CMS必须依赖full GC来触发compact操作。然而full GC的停顿时间长，导致系统响应性降低，当有后续请求进来时，系统会越来越慢，进而发生雪崩现象。

G1确实是这个场景下最合适的垃圾回收策略。G1可以更好的利用大内存，并有效降低停顿时间。而且在进行minor GC的时候就会触发compact操作，解决了碎片化的问题。从而保证了系统响应性，提高了吞吐量。

目前看来，本地单机压测的结果一致看好G1，还未发现其他问题，找个生产环境再压压看吧。

GC调优还是很好玩的，我本地使用两个工具用于观测GC，一个是VisualVM，另一个是jstat。貌似jconsole也不错。这次调优之旅不仅让我用实践的方式理解了GC，还让我更好的理解业务。处理输入输出的那部分代码看来需要重构，否则如果输入输出太大，直接进入老年代就不好玩了。