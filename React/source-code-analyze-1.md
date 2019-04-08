# 手摸手带你阅读 React 源码（一）

本章我们来讲解下 `packages/react/src/ReactElement.js` 的代码。

> 官网的定义：Elements are the smallest building blocks of React apps.

## 知识点整理

* [Symbol](http://es6.ruanyifeng.com/#docs/symbol)
* [$$typeof 属性的意义](http://tech.colla.me/zh/show/why_react_tags_element_with_$$typeof)

## 代码分析

`ReactElement` 是一个工厂方法，用于创建一个 React Element。React Element 本质就是一个对象。

```js
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 此标记允许我们将其唯一标识为 React 元素
    $$typeof: REACT_ELEMENT_TYPE,

    // 属于元素的内置属性
    type: type,
    key: key,
    ref: ref,
    props: props,

    // 记录负责创建此元素的组件。
    _owner: owner,
  };

  // ...

  return element;
};
```

