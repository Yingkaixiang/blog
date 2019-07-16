# 正则

## 替换正则表达式的保留字符

```js
// 替换正则表达式的标识符
// 从而可以匹配 () {} 等字符
const invalid = /[°"§%()\\[\\]{}=\\\\?´`'\#\<\>|,;.:+\_-]/g;
const final = value.replace(invalid, function($1) {
 return '\\\\' + $1;
})
const reg = new RegExp(final, 'ig');
```