### 背景知识：跨域

浏览器出于安全考虑，**不允许一个网页去请求另一个不同源（协议+域名+端口）的资源**。这就是“跨域”。

比如：

- 你前端页面在：`http://localhost:3000`
- 你要请求的API在：`http://localhost:8000`

这就是两个不同的源，请求会被浏览器拦截，提示跨域（CORS）错误。



### 配置代理的目的：

解决本地开发中的跨域问题，并模拟生产环境请求 API 的情况。

本地开发时，执行`npm run dev`以后，

- 前端运行在 `http://localhost:3000`
- 后端 API 服务运行在 `http://localhost:8000`
- 前端代码中你希望请求 后端接口`/api/user/list`，但这会跨域



🚫 没有代理时：

```js
// axios 请求代码
axios.get('/api/user/list')
```

你期望访问的是 `http://localhost:8000/api/user/list`，但实际上浏览器默认会请求当前 `origin：`

```http
GET http://localhost:3000/api/user/list  ← 这个地址是不存在的
```

如果你改成：

```js
axios.get('http://localhost:8000/api/user/list')
```

则会跨域，被浏览器拦截。



#### 为什么会这样呢？

这是因为 **浏览器在处理相对路径时遵循默认的 URL 解析规则** —— 相对路径是基于**当前页面的 origin（协议 + 域名 + 端口）**进行解析的。

```js
axios.get('/api/user/list')
```

这是一个**相对路径请求**，它**没有指定协议（http/https）、域名、端口**。浏览器就会自动把它补全为：

```js
[当前页面的 origin] + /api/user/list
// 即
http://localhost:3000/api/user/list
```

对于浏览器来说，`'/api/user/list'`，它**不是一个“完整的 URL”**，因为它没有：

- 协议（http://）
- 域名（localhost）
- 端口（3000）

只有这些都有时，浏览器才认为它是“完全的 URL”，比如：

```
axios.get('http://localhost:8000/api/user/list')  // 完整 URL
```

`/api/user/list`，是相对于当前域名根目录 `/` 的路径。

在执行`npm run dev`以后，前端是开启了一个开发服务器，开启一个HTTP服务：

默认运行在http://localhost:3000域名下，将项目文件（如 HTML、CSS、JS 等）托管在这个临时环境中，通过浏览器访问http://localhost:3000，加载的就是开发服务器托管的前端页面。

`axios.get('/api/user/list')`针对路径`/api/user/list`，浏览器会认为这是当前域名下的绝对路径，会自动的将当前页面的`origin`（协议 + 域名 + 端口）拼接到路径前，形成完整的 URL

```js
http://localhost:3000/api/user/list
```

也就是说，对于这种路径，浏览器默认是当前域名下的路径，因为这样是不会产生跨域的。

但是当前域名下并没有后端的接口服务，所以访问不到，还是需要访问

```js
http://localhost:8000/api/user/list
```

但是域名不同，产生了跨域问题，这就需要配置代理解决。



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