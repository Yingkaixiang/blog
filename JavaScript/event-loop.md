# 事件循环（Event Loop）

> 本片博客讨论的是浏览器中的事件循环模型

## 为什么要有事件循环模型？

**因为 `JavaScript` 是单线程的**。但是很多时候我们仍然需要在不同的时间去执行不同的任务，例如给元素添加点击事件，设置一个定时器，或者发起Ajax请求。因此需要一个异步机制来达到这样的目的，事件循环机制也因此而来。

## JavaScript 为什么是单线程的？

JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

## 到底什么是 Event Loop 呢？

- [菲利普·罗伯茨：到底什么是 Event Loop 呢？ | 欧洲 JSConf 2014](https://www.youtube.com/watch?v=8aGhZQkoFbQ&t=8s)
- [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

## 浏览器多线程方案 Web Worker

- [Web Worker 使用教程](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)
