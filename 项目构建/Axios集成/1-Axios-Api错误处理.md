### Axios网络请求的响应结构：

 axios **成功时（HTTP 2xx）**的标准响应结构

```js
{
  data: {},  // `data` 由服务器提供的响应
      
  status: 200, // `status` 来自服务器响应的 HTTP 状态码

  statusText: 'OK',  // `statusText` 来自服务器响应的 HTTP 状态信息

  // `headers` 是服务器响应头
  // 所有的 header 名称都是小写，而且可以使用方括号语法访问
  // 例如: `response.headers['content-type']`
  headers: {},

  config: {},   // `config` 是 `axios` 请求的配置信息

  // `request` 是生成此响应的请求
  // 在node.js中它是最后一个ClientRequest实例 (in redirects)，
  // 在浏览器中则是 XMLHttpRequest 实例
  request: {}
}
```

其中`status`字段标识HTTP 状态码



### HTTP 响应码 vs 业务 code

#### ✔ HTTP status 描述的是“传输层是否 OK”

比如：

- `200` = 请求成功
- `401` = 没登录/没权限
- `404` = 接口不存在
- `500` = 后端炸了

这些都是 **协议规定好的**。



当`status` 不是 2xx（即 3xx、4xx、5xx），此时返回的是`error`对象，不是`response`对象



------



#### ✔ 业务 code 描述的是“业务层是否 OK”

完全由后端自定义，比如：

```json
{
  "code": 0,
  "message": "success",
  "data": {...}
}
```

或者失败：

```json
{
  "code": 30001,
  "message": "用户名不能为空"
}
```

或者 token 过期：

```json
{
  "code": 40101,
  "message": "token expired"
}
```

**只要 HTTP 是 200，这些情况都会触发 axios 的第一个拦截器函数。**

此时结构如下：

```js
{
  data: {
      {
          "code": 0,
          "message": "success",
          "data": {...}
		}
  },  
      
  status: 200,

  statusText: 'OK',  

  headers: {},

  config: {},   

  request: {}
}
```





### API 错误处理策略（API Error Handling Strategy）

也可以说：

- **API 错误表达方式**（API Error Representation）
- **服务端错误处理设计**（Server Error Handling Design）
- **错误信号传递模型**（Error Signaling Model）

也就是说，这两套风格的本质都是：

> 服务端到底应该用什么方式把“错误”传达给客户端？

它是 “错误传递模型” 上的两个分支。



___



#### 🅰 **风格 1：使用真实 HTTP 状态码的风格**

常见叫法：

**✔ RESTful 风格（RESTful API）**

**✔ HTTP 语义驱动风格**（HTTP-meaningful design）

**✔ HTTP 状态码语义设计**（Use HTTP status semantics）

特点：

- 401 就真的表示认证失败
- 404 就是真的资源不存在
- 错误靠 HTTP 状态码表达

**非常贴协议，非常“正统”。**



#### **后端返回 HTTP 401 时，会是什么样？**

🚨 **关键点：401 是 HTTP 层面的失败**
 → 所以 axios 会走 **响应拦截器的第二个函数**（error 回调）
 → 你拿到的是一个 **error 对象**，不是 response 对象

#### 那 error 对象里长啥样？

大概是这样👇（重点字段我用注释标出来）

```js
// error对象内部结构
{
  message: 'Request failed with status code 401',
  name: 'AxiosError',
  code: 'ERR_BAD_REQUEST',    
  config: { ... },
  request: { ... },

  // ⭐⭐重点：这个才是响应内容
  response: {
    data: {
      message: 'token expired'
    },
    status: 401,
    statusText: 'Unauthorized',
    headers: {
      'content-type': 'application/json'
    },
    config: { ... },
    request: { ... }
  }
}
```



| 字段                        | 值              |
| --------------------------- | --------------- |
| response.status             | 401             |
| response.data               | 后端返回的 body |
| axios 不会进入 success 回调 | ✔               |
| axios 进入 error 回调       | ✔               |

也就是说：

**后端设置 HTTP 401 → 响应结构变成 error.response.xxx，而不是 response.xxx**

这就是它和业务 code 最大的差别之一。



这时候根据不同的响应码做业务处理，需要在响应拦截器的第二个回调中处理

```js
axios.interceptors.response.use(
  res => res,
  error => {
    if (error.response && error.response.status === 401) {
      // token 过期
      logout();
    }
    return Promise.reject(error);
  }
);
```



即使使用了`非2xx`状态码，仍然需要进一步根据业务`code`判断（具体根据后端Api的设计来做）

```js

// 响应拦截器
service.interceptors.response.use(
  // 第一个回调
  (response: AxiosResponse) => response,
    
 // 此时进入第二个回调
  async (error) => {
    // 从error对象中获取config与response
    const { config, response } = error;  
  
    if (response) {
      // 此时再根据业务code值，进行不同的操作
      const { code, msg } = response.data;
      if (code === ResultEnum.ACCESS_TOKEN_INVALID) {
        // Token 过期，刷新 Token
        return handleTokenRefresh(config);
      } else if (code === ResultEnum.REFRESH_TOKEN_INVALID) {
        // 刷新 Token 过期，跳转登录页
        await handleSessionExpired();
        return Promise.reject(new Error(msg || "Error"));
      } else {
        ElMessage.error(msg || "系统出错");
      }
    }
    return Promise.reject(error.message);
  }
);
```



___



### 🅱 **风格 2：永远返回 200 + 自定义业务 code**

常见叫法：

**✔ 统一响应包装**（Unified Response Wrapper）

**✔ 统一返回结构风格**（Unified Response Body）

**✔ 业务码驱动风格**（Business-code-driven API）

**✔ 内部 RPC 风格**（RPC-like API）

特点：

- 所有 HTTP 状态码都是 200
- 成功/失败靠业务 code 判断
- 很像 RPC 或微服务内部通信的做法（gRPC、Thrift 风）

**实用主义，国内极常用。**



###  **HTTP 始终返回 200，但业务 code 表示错误，会是什么样？**

这种情况 axios 会认为请求是“成功的”，所以：

✔ 会进入 `response => {}` 的第一个回调
 ✔ 你拿到的是正常的 response 对象

比如后端返回：

```json
{
  "code": 40101,
  "message": "token expired"
}
```

axios 的响应结构就会是：

```js
{
  data: {
    code: 40101,
    message: 'token expired'
  },
  status: 200,
  statusText: 'OK',
  headers: {
    'content-type': 'application/json'
  },
  config: { ... },
  request: { ... }
}
```

结构和成功没区别，唯一变化是：

- `data` 里的业务 code 是异常状态（业务错误）
- 但 HTTP status 还是 200
- 所以 axios 完全不会触发 error 回调



这种情况下，如果需要根据业务code，进行逻辑操作，需要在axios的响应拦截器的第一个回调中处理。

→ **不会进第二个函数**
 → **会进第一个函数**，因为 HTTP 是成功的

那么你要在第一个函数里判断业务 code ▼

```js
axios.interceptors.response.use(
  (response) => {
    const res = response.data;
    if (res.code === 40101) {
      logout();
      return Promise.reject(res);
    }
    return res;
  },
  (error) => {
    // 这里一般处理 HTTP 层面错误
    return Promise.reject(error);
  }
);
```

***



### 🧁 对比表

| 场景                              | HTTP status | axios 是否进入 error 回调？ | data 内部内容            | 前端处理点                    |
| --------------------------------- | ----------- | --------------------------- | ------------------------ | ----------------------------- |
| **后端返回 401**                  | 401         | ✔ 是                        | error.response.data      | error.response.status === 401 |
| **后端返回 200 + 业务 code 异常** | 200         | ❌ 否                        | response.data.code=401xx | response.data.code === 401xx  |

### 🧸 小结

- HTTP 401 → axios 当作失败 → 结构在 `error.response`
- HTTP 200 + code异常 → axios 当作成功 → 结构在 `response`

所以本质上：

> **HTTP status 决定 axios 走哪个回调**
>
> **业务 code 决定你的系统逻辑要不要继续走**

需要根据后端怎么设计，来决定前端axios是在响应拦截器的哪一个回调中处理逻辑

### 



