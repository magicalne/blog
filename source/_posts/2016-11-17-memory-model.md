---
layout: post,
title: Memory model
date: 2016-11-17
category: java, memory model
---

[memory model](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)

### What is reordering?

### synchronization under the hood

> Synchronization has several aspects. The most well-understood is mutual exclusion -- only one thread can hold a monitor at once, so synchronizing on a monitor means that once one thread enters a synchronized block protected by a monitor, no other thread can enter a block protected by that monitor until the first thread exits the synchronized block.
But there is more to synchronization than mutual exclusion. Synchronization ensures that memory writes by a thread before or during a synchronized block are made visible in a predictable manner to other threads which synchronize on the same monitor. After we exit a synchronized block, we release the monitor, which has the effect of flushing the cache to main memory, so that writes made by this thread can be visible to other threads. Before we can enter a synchronized block, we acquire the monitor, which has the effect of invalidating the local processor cache so that variables will be reloaded from main memory. We will then be able to see all of the writes made visible by the previous release.

TODO
