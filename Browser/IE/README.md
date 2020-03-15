# IE

##### 1. AJAX GET 请求缓存

相同的 GET 请求地址 ie 会使用本地的缓存数据而不是向服务器再次发送请求。

解决方法：

- 使用其他的 method
- 在请求的 query 中加入时间戳，如 `http://www.xxxxx.com?__timestamp__=10000`

##### 2. 时间格式兼容性

IE 无法解析 2019-01-01 这样的格式，只能使用 2019/01/01。

##### 3. AJAX 中文乱码

AJAX 中如果带有中文时需要手动 `encodeURI`，否则会发送乱码。`axios` 库请直接使用 `params` 参数，不要手动拼接 query，restful API 禁止使用中文。
