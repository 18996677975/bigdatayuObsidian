- [Java 魔法类：Unsafe 应用解析 - 美团技术团队 -2019open in new window](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)
- [Java 双刃剑之 Unsafe 类详解 - 码农参上 - 2021](https://xie.infoq.cn/article/8b6ed4195e475bfb32dacc5cb)

阅读过 JUC 源码的同学，一定会发现很多并发工具类都调用了一个叫做 `Unsafe` 的类。

那这个类主要是用来干什么的呢？有什么使用场景呢？这篇文章就带你搞清楚！
