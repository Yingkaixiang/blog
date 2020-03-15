# shelljs 坑

```js
// shelljs exec 命令默认执行输出的字符串带有空格
// 所以会导致命令在执行时变成两个命令
// 需要手动 trim 一下
const stdout = shelljs.exec('git config --get remote.origin.url').stdout;
stdou.trim();
```