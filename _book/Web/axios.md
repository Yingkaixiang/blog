# axios

## axios 的坑

- `axios` 使用 `POST` 请求时必须带上 `data` 参数，如果参数为空则可以直接使用 `{}`。否则请求中不会带上 `Content-type`。
