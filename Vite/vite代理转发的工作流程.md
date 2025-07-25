### vite.config.js配置

```js
server:{
    proxy: {
      '/b2b': {
        target: 'https://b2b.ebcrack.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(new RegExp('^' + '/b2b'), '')
      }
  	}
}
```

浏览器（或你的代码）发出的请求是：`/b2b/seller.php?mode=resource_dump&resource_id=30`

#### **代理转发的工作流程 ：**

##### 1. 浏览器发起请求：

​	你的前端应用（运行在 Vite dev server 上，例如 `http://localhost:xxxx`）通过 fetch、axios 或者 <img> 标签的 src 发起了一个请求。
请求的 URL 是：`http://localhost:xxxx/b2b/seller.php?mode=resource_dump&resource_id=30`

##### 2. Vite Dev Server 拦截请求：

​	Vite dev server 运行在 `localhost:xxxx`。它会检查所有传入的请求。
它发现请求的路径 `/b2b/seller.php?mode=resource_dump&resource_id=30` 以 `/b2b` 开头，这与你代理配置中的 `'/b2b'` 键匹配。

##### **3. 应用代理规则*：**

​	Vite dev server 确定这个请求需要通过代理处理。

​	**目标 (Target)：** 它知道目标服务器是 `https://b2b.ebcrack.com`。

​	**路径重写 (Rewrite)：** 这是关键的一步。

​		**原始请求路径 (path from browser):** `/b2b/seller.php?mode=resource_dump&resource_id=30`

​		**重写规则:** `path.replace(new RegExp('^' + '/b2b'), '')`

​		**应用重写：**
​			原始路径中开头的 `/b2b` 被替换为空字符串 ''。
​			重写后的路径: `/seller.php?mode=resource_dump&resource_id=30`

​	**构造目标 URL**

​		Vite dev server 会将重写后的路径与 `target` 结合起来，形成最终要请求的 URL。

​		目标服务器基础 `URL: https://b2b.ebcrack.com`

​		重写后的路径: `/seller.php?mode=resource_dump&resource_id=30`

​		**最终请求的 URL (由 Vite dev server 向目标服务器发出):** `https://b2b.ebcrack.com/seller.php?mode=resource_dump&resource_id=30`

##### 4. Vite Dev Server 向目标服务器发起请求：

Vite dev server 向 `https://b2b.ebcrack.com/seller.php?mode=resource_dump&resource_id=30` 发起一个新的 HTTP GET 请求。

- `Host` 请求头设置为 `b2b.ebcrack.com` (因为 `changeOrigin: true`)。
- 其他相关的原始请求头部、方法、查询参数和请求体（如果有）都会被转发。

##### 5. 目标服务器 (b2b.ebcrack.com) 处理请求：

目标服务器接收到请求，seller.php 脚本执行，基于参数和可能的 Cookie 生成响应。
假设它返回了图片数据和正确的 `Content-Type: image/...`。

##### 6. 目标服务器返回响应给 Vite Dev Server：

`b2b.ebcrack.com` 将 HTTP 响应发送回 Vite dev server。

##### 7. Vite Dev Server 将响应转发给浏览器：

Vite dev server 将响应转发回浏览器。浏览器认为它收到了来自 `http://localhost:xxxx/b2b/seller.php?...` 的响应。

##### 8. 浏览器处理响应：

浏览器根据 `Content-Type` 和响应体显示图片。