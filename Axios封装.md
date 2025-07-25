响应拦截器

```js
service.interceptors.response.use(
  /** 第一个函数（成功的响应） */
  (response) => {
    // ...
  },

  /** 第二个函数（失败的响应） */
  (error) => {
    // ...
  }
)
```

✅ 第一个函数触发的条件

**只要服务器成功返回了响应（HTTP 状态码为 2xx 或 3xx）就会触发**，即使业务上是失败的。

```json
HTTP 200 OK
{
  "code": 400,
  "msg": "无权限访问",
  "data": null
}
```

这种情况，**HTTP 层是成功的，Axios 会进入第一个函数**，你需要在这里处理业务失败的 `code !== 200` 情况。



❌ 第二个函数触发的条件

**只有在 HTTP 层出错时才会触发**，比如：

- 网络异常（断网、超时）
- 请求被取消
- HTTP 状态码是：
  - `400 Bad Request`
  - `401 Unauthorized`
  - `403 Forbidden`
  - `404 Not Found`
  - `500 Internal Server Error`
  - 其他非 2xx 状态码

在这些情况下，**Axios 不会走第一个函数，直接走第二个函数的 `error` 分支**。









