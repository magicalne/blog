title: 初探golang
layout: post
date: 2020-11-05
tag:
- go
---

go作为一门现代语言，没有历史包袱，语法简单。就是由于起初设计语言的时候，把go设计的太简单，导致很多决策都是后面版本迭代出来的。go特别像是简易c语言+runtime。

说go更加现代，首先在变量声明就能提现。到底是强类型好？还是弱类型好？我自己写的代码，变量起个名字就好，为什么还要声明类型？当下的论调就是，自己开发的方法里面的变量声明，可以省略类型声明，如果要暴露给别人调用，应该为入参和返回值声明类型。go的变量声明可以是编译器通过value推演的，天然可以省略类型声明。但是返回和入参是必须声明类型的。

没有exception，error只不过是一个value而已。没有采纳try-catch，而是简单的使用多返回值的形式，强制调用者对error进行处理。这里rust跟go很像，也是没有exception。但是rust吸收了更多函数式编成语言特性，使用Result对返回值进行包装，而不是作为多返回值的形式。

go有指针的概念，而且真的是指针，但是没有c各种神奇的功能。因为go设计为pass by value，每次向一个方法里传递变量都是一份copy。那怎么在这个方法里修改这个参数变量的内容呢？答案就是传一个指针进去。尽管这个指针也是一份copy，但是指向的内容确是真实存在的。这里就很不java了，java通过封装可见性(private/protected/public/final)来防止用户修改某个object。

没有class？用method！这里rust也跟go很像。除了普通的func，还可以为struct实现自己的func，也就是method。加上指针的存在，就可以区分“只读”method和“读写”method。

[Go Proverbs](https://www.youtube.com/watch?v=PAAkCSZUG1c)
[A tour of Go](https://tour.golang.org/)
[faq](https://golang.org/doc/faq)
[Go memory model](https://golang.org/ref/mem)