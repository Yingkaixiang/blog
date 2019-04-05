# 深入理解 Virtual DOM

## 目录
* MVC 与 MVVM 的区别？
* 为什么要使用 virtual DOM?
* 为什么说浏览器的 DOM 操作很慢？
* 树的优先深度遍历
* 编辑距离算法（理解为主）
* React DOM-diff 算法中的编辑距离


### React DOM-diff 算法中的编辑距离



两个需要被比较的数组的数据结构为 `object[]`，在指定被比较的属性（React 中 即为 key）的情况下，逻辑如下（[参考代码](https://github.com/livoras/list-diff)）：

![](./list-diff.jpg)