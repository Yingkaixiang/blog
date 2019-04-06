# 手摸手带你阅读 React 源码（一）

本章我们来讲解下 ReactElement.js 的代码。

## 知识点整理

* [Symbol](http://es6.ruanyifeng.com/#docs/symbol)
* [$$typeof 属性的意义](http://tech.colla.me/zh/show/why_react_tags_element_with_$$typeof)

## 代码分析

```js
import {REACT_ELEMENT_TYPE} from 'shared/ReactSymbols';
```

