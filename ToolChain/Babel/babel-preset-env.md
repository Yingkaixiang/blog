# babel-preset-env

待验证

```js
module.exports = {
  "presets": [
    [
      "@babel/preset-env",
      {
        // babel 默认只转换 syntax（如：箭头函数）
        // 所以如果需要使用 api 的话（如：window.Promise, Array.from）
        // 需要使用相关 polyfill
        // 使用以下配置可以
        // 自动按需引入 corejs 以及 regenerator runtime
        // 但是这样引入会污染全局变量
        // 建议在完整的独立项目中使用
        "useBuiltIns": "usage",
        // 无需自行安装 corejs
        "corejs": 3
      }
    ]
  ],
  // 如果开发的是一个单独的库
  // 则可以添加以下的配置
  // 这样可以使得所有的 polyfill 以运行时的方法进行注入
  // 需要同时安装 @babel/runtime
  // 如果编译时报错 can't resolve 某个模块时
  // 尝试 include 指定的文件夹进行编译
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": 3
      }
    ]
  ],
}
```