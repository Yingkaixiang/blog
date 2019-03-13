# 事件循环 Event Loop（浏览器）

- [菲利普·罗伯茨：到底什么是 Event Loop 呢？ | 欧洲 JSConf 2014](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)

## JavaScript 为什么是单线程的？

单线程是必要的，也是 JavaScript 这门语言的基石，原因之一在其最初也是最主要的执行环境浏览器中，我们需要进行各种各样的 dom 操作。试想一下，如果 JavaScript 是多线程的，那么当两个线程同时对 dom 进行一项操作，例如一个向其添加事件，而另一个删除了这个 dom，此时该如何处理呢？因此，为了保证不会发生类似于这个例子中的情景，JavaScript 选择只用一个主线程来执行代码，这样就保证了**程序执行的一致性**。

## Web Worker

- [Web Worker 使用教程](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)
