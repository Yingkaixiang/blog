# Unicode

## &#65279

&#65279 是一个特殊的 unicode 字符叫做 `ZERO WIDTH NO-BREAK SPACE (U+FEFF)` 当你在复制/黏贴的时候可能会带上这个字符，但是不会显示出来，但是你可以通过以下方法去除

```js
str.replace(/\ufeff/g, '');
```