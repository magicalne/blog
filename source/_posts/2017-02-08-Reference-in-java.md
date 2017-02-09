layout: post
title: Java中的引用类型
date: 2017-02-08
tag:
- java
- reference
---

最近在写一个Java实现的cache，就是一个guava cache的轮子。   

之前没有刻意研究guava cache，这几天重温guava源码的时候，发现guava cache中继承了SoftReference, WeakReference, PhantomReference，里面还有实现了的queue，这些reference会被enqueue，好奇心因此产生。   

看到上述代码的时候才发现自己不知道如何在java中使用这些reference，于是研究了一番。大家经常在面试、笔试中碰到的reference的问题大都是问：“Java中有几种reference？分别是什么？”但是很少问它们都是用来干什么的？   

[java.lang.ref](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/package-summary.html)上有详细的解释，但是不是很通俗。由于我是想自己写一个cache，而SoftReference非常适合开发cache程序，我就特别关注这个问题。总的来说，这些reference都是跟垃圾回收器相关，会针对不同的reference类型，产生不同的回收策略。   

>Soft references are for implementing memory-sensitive caches, weak references are for implementing canonicalizing mappings that do not prevent their keys (or values) from being reclaimed, and phantom references are for scheduling pre-mortem cleanup actions in a more flexible way than is possible with the Java finalization mechanism.   

为什么说SoftReference适合做cache？因为当你为一个普通对象创建一个SoftReverence，即使当这个对象不再拥有任何其他对象的强连接，垃圾收集器也可能不会立即将SoftReference回收，而是等到剩余内存达到阀值了，不得不进行垃圾回收的时候才将SoftReference引用的对象回收。因此，在构建cache程序时，这里就可以使用SoftReference构成多级cache架构。第一层是你的cache数据结构，可能是一个自己实现的Map，第二层是SoftReference。

### 对比SoftReference、WeakReference和PhantomReference   

```java
ReferenceQueue<Foo> fooQueue = new ReferenceQueue<>();
Foo foo = new Foo();
Reference<Foo> reference = new WeakReference<>(foo, fooQueue);
foo = null;
System.gc();
System.out.println(reference.get() == null); //true
```

在JVM开始gc的时候，由于foo没有任何连接对象，WeakRefeence会直接被GC回收。   

```java
ReferenceQueue<Foo> fooQueue = new ReferenceQueue<>();
Foo foo = new Foo();
Reference<Foo> reference = new SoftReference<>(foo, fooQueue);
foo = null;
System.gc();
System.out.println(reference.get() == null); //false
```

而当使用SoftReference时，即使foo没有任何强连接，在发生GC的时候，由于内存尚且充足，SoftReference不会被立即回收，而是在将来内存不够用的时候再回收。   

```java
ReferenceQueue<Foo> fooQueue = new ReferenceQueue<>();
Foo foo = new Foo();
Reference<Foo> reference = new PhantomReference<>(foo, fooQueue);
foo = null;
System.out.println(reference.get() == null); //true
```

PhantomReference作为最弱的引用对象，当一个对象只被一个PhantomReference引用时，不管是否发生GC，这个对象都会被回收。   

有趣的是，以上对于PhantomReference的解释是一个错觉。因为PhantomReference.get()永远返回**null**。   

```java
/**
 * Returns this reference object's referent.  Because the referent of a
 * phantom reference is always inaccessible, this method always returns
 * <code>null</code>.
 *
 * @return  <code>null</code>
 */
public T get() {
    return null;
}
```

### ReferenceQueue是个什么鬼？
TODO

# Reference
1. [Difference between WeakReference vs SoftReference vs PhantomReference vs Strong reference in Java](https://www.javacodegeeks.com/2014/03/difference-between-weakreference-vs-softreference-vs-phantomreference-vs-strong-reference-in-java.html)
2. [What is a Soft Reference Cache Anyway?](https://www.ortussolutions.com/blog/tip-of-the-week-what-is-a-soft-reference-cache-anyway)
