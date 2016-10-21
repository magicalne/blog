---
layout: post
title: "javascript的一些问题"
date: 2016-10-21
category: javascript, frontend
---

### 版本问题   
这里说的javascript版本问题不是指浏览器兼容性。其实对于开发来说，有了babel的话，根本不需要担心版本问题，我一样可以用最新的语法特性。   

但是问题是，每一次javascript的版本升级，不像是真正意义上的版本升级，诚然，每个版本都在解决一部分问题，但是各个升级后的版本看起来更像是一个新的方言。光导入就有AMD、commonjs和es6 import。平常用的时候可能感觉没啥问题，我能用es6的地方都是用import的，但是有些地方使用import是不能解决问题的，你需要使用require。比如在webpack使用react引入图片的问题，你只能使用require，而不能使用import！！！

```jsx
 <Image src={require('../img/xxx.jpg')} />
```

当时各种场景都试过了，根本没有思考要使用require去做这件事，结果试了一天都没把一个破图片搞出来。。。最后换了require试了一下居然出来了。。。后来认为应该跟webpack的加载方式有关系，尽管babel是会把import翻译成require的，但是webpack是把图片hash过的，这里需要一层mapping吧。总之，看前端项目的时候，就是import中混合了零星的require。   

TODO
