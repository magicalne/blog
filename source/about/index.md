magicalne
==============

github [@magicalne](https://github.com/magicalne/)
twitter [@magicalne](https://twitter.com/magicalne)
gmail <magicalne@gmail.com>

浙江大学 软件工程 硕士

Skill
---

* 夯实的计算机科学基础，包括操作系统、网络、数据结构、算法。
* Java核心，包括范型、集合、IO、多线程、JMM、GC、JVM。
* Spring全家桶，包括Springboot、Spring MVC、Spring security、Spring Data、SpringAMQP、Spring。
* Mongodb
* Gradle
* Vim and Emacs😎

Experience(2 years +)
---

目前就职于上海某P2P公司，是我毕业以来的第一份正式工作，一直从事公司借款端核心风控产品的开发工作。最早开发协助风控人员的人工贷款审批系统。 近一年多以来，专注于开发公司主打的risk engine和rule engine，用于支持自动线上风控。

Project
---

1. 2016.03-**Now**: **Rule Engine**——利用[jsr223](https://www.jcp.org/en/jsr/detail?id=223)支持python和groovy，线上实时发布生效，使用guava cache缓存脚本实例，支持[PMML](https://en.wikipedia.org/wiki/Predictive_Model_Markup_Language)，以及根据facebook的[PlanOut](https://facebook.github.io/planout/)开发一套内置的A/B testing框架。对比[Drools](https://www.drools.org/)，从单请求执行时间、并发能力、易用性和扩展性等方面碾压Drools。更适合互联网金融的场景。
2. 2016.03-**Now**: **Rule Engine UI**——[reactjs](https://facebook.github.io/react/)全家桶和[material-ui](http://www.material-ui.com/#/)构建一套基于[material design](https://material.io/guidelines/)的模块化前端开发框架。
3. 2015.11-**Now**: **Risk Engine**——使用适配器模式抽象第三方数据源，配合观察者模式开发基于rabbitmq的无阻塞队列。
4. 2016.09-2016.09: **Knowledge Graph UI**——修改[orientdb-studio](https://github.com/orientechnologies/orientdb-studio)的源代码，用来支持展示我们自己的知识图谱项目。
5. 2016.09-2016.10: **Knowledge Graph**——使用Orientdb的接口实现图搜索风控策略。
6. 2014.12-2015.10: CRC——在这个项目中，我将原本风控常用的计算公式从前端放到后端，使用逆波兰式和中缀表达式处理，后端灵活配置，前端只负责展示。

Side Project
---

1. [Jache](https://github.com/magicalne/Jache) Java cache的轮子，目前开发中。
2. [Taber](https://github.com/magicalne/taber) -- Chrome extension. 可以通过模糊匹配页面的title和url，快速跳转到目标tab，特别是在full screen模式中，使得tab切换极为方便。（由于上传到chrome app store需要付5刀，可我又没有信用卡😤，所以你最近还不会在app store中看到它...）

Focusing on
---
目前对Java底层、JVM底层以及AKKA最感兴趣，关注Java社区最新进展。

Future
---
希望从事优秀App的后端开发或高并发场景／IO密集型场景的中间件开发。   
其实还是很喜欢做线上风控开发的，风控很有意思。但是互金的现状就是人人都喊风控重要，却**不够**重视风控。技术没能解决问题，却成了大佬们的吹B利器。