title: rust反序列化提升性能1
layout: post
date: 2020-04-27
tag:
- rust
- deserialize
- serde_json
- nom

---

一个多月前入了rust的坑。当时想学学rust就是想尽可能的提升性能，但是又“不敢”写生产环境级别的C++。退而求其次，学学rust吧。

最近遇到了一个之前写java就遇到过的问题——deserialize慢。

client收到了server发送的response，把json反序列化。这是一个非常常见的场景，使用serde_json的derive可以简单干净的完成功能。
但是serde_json会使用vector，而vector是在heap上，allocation就是性能的第一块瓶颈。加上在业务逻辑处理的地方需要频繁的删除vector中的元素，dealloc也很慢。

咋办呢？

# 1. impl DeserializeSeed

第一个想法是向deserializer传递一个struct，遍历一遍input即可完成对struct的修改。对于数组字段，我可以声明一个统计意义不可越界的array，这样所有allocation都发生在stack上。DeserializeSeed可以接受参数传递，我可以实现这个trait。

但现实是，这个并不好实现。

```rust
pub trait DeserializeSeed<'de>: Sized {
    /// The type produced by using this seed.
    type Value;

    /// Equivalent to the more common `Deserialize::deserialize` method, except
    /// with some initial piece of data (the seed) passed in.
    fn deserialize<D>(self, deserializer: D) -> Result<Self::Value, D::Error>
    where
        D: Deserializer<'de>;
}
```

原因之一在于，deserialize这个method的self参数不是mutable。当然用wrapper可以解决。

第二个原因是不好解决nested object。想了两天没想明白，就放弃了。

# 2. Nom

这是第二次手写parser了，上次写java用的parboiled。这次的Nom给我打开了新世界。我一开始一直局限在自己的思想之下，一直想着怎么向parser传递变量。这是与Nom的设计相违背的。

Nom parser的返回签名很简单: **IResult<Input, Output, Error>**

parser执行成功，会返回<剩余input， 生成的输出>；如果失败，则会返回<生于input，Error>。典型的rust返回范式。而parser的输入永远只有一个，默认的实现支持&[u8]和&str。文档说也可以支持自定义的input。

对于parser过程中需要识别的部分，则需要通过组合的方式把子parser合并起来。这也就是combinator的思想。各个combinator会byte by byte的消化input。

Nom对需要循环处理的地方也是用vec实现的，所以这里需要自己实现了。

## serde_json vs nom benchmark

是时候比一比了！

```
Deserialize OrderBookL2/serde_json
                        time:   [11.647 us 11.790 us 11.971 us]
                        change: [+7.7511% +10.118% +12.641%] (p = 0.00 < 0.05)
                        Performance has regressed.
Found 11 outliers among 100 measurements (11.00%)
  4 (4.00%) high mild
  7 (7.00%) high severe
Deserialize OrderBookL2/nom
                        time:   [7.4556 us 7.4677 us 7.4816 us]
                        change: [+3.0453% +3.3319% +3.6136%] (p = 0.00 < 0.05)
                        Performance has regressed.
Found 3 outliers among 100 measurements (3.00%)
  2 (2.00%) high mild
  1 (1.00%) high severe
```

尴尬，没有想象中的好。还停留在微妙级别。

再对比一下，用nom作者实现的json parser呢？

## serde_json vs nom vs nom json

```
Deserialize OrderBookL2/serde_json                                                                             
                        time:   [10.984 us 11.030 us 11.088 us]
                        change: [-10.400% -8.1551% -6.0993%] (p = 0.00 < 0.05)
                        Performance has improved.
Found 11 outliers among 100 measurements (11.00%)
  5 (5.00%) high mild
  6 (6.00%) high severe
Deserialize OrderBookL2/nom_my_parser                                                                             
                        time:   [7.2884 us 7.3109 us 7.3376 us]
Found 15 outliers among 100 measurements (15.00%)
  5 (5.00%) high mild
  10 (10.00%) high severe
Deserialize OrderBookL2/nom_json                                                                            
                        time:   [439.24 ns 439.45 ns 439.66 ns]
Found 10 outliers among 100 measurements (10.00%)
  6 (6.00%) low mild
  3 (3.00%) high mild
  1 (1.00%) high severe
```

这真的很尴尬了，使用vec+map实现的nom_json居然比我自己实现的快这么多，已经到ns级别了。

确实nom本身的优化已经很极致了，[这里](https://www.youtube.com/watch?v=7VNsmlCAmHU)是nom作者做的一次分享。采用了多种优化方式，最终将http parser的性能从500MB/S提升到2GB/s。