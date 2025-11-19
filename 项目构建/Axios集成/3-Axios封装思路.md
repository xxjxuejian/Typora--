### **中后台系统的常规设计：**

- 路由守卫：负责判断 token 是否存在、有效、是否需要动态注册路由
- 请求拦截器：只负责附加 token
- 响应拦截器：负责处理 token 过期（401）并做统一行为



### 🧩 **结论先说：请求拦截器里一般 不需要 判断 token 是否存在或过期**

这不是偷懒，而是**职责拆分**导致的结果。
 每个模块只做属于自己的那一部分，不搞“一锅炖”。



### 🧭 为什么请求拦截器里不判断 token 有效性？

#### ✔ 1. **路由守卫负责控制“是否能进页面”**

路由守卫是“门卫”：

- 你有没有 token？
- token 还行不行？
- 没 token 就送你去登录页。

这是页面级别的权限。

它的职责就是：
 **不给你看到不该看的页面。**
 网络请求发生不发生，那是后面的事。

------



#### ✔ 2. 网络请求拦截器只负责一个事：把 token 带上**

拦截器主要职责：

- 给请求加上 Authorization 头
- 做些公共参数
- 改个 content-type 之类的

它像“挂件”，**不做权限控制**这种大事。

如果你让它也做 token 校验，会出现“路由守卫 + 请求拦截器同时判断 token 的混乱局面”。

而且会让请求拦截器逻辑变得臃肿。

------



#### ✔ 3. **判断 token 是否过期是后端的责任**

即便前端做了“本地过期时间判断”，后端也必须判断。不然太危险。

所以很多中后台项目直接让后端返回 401，然后统一处理。

👉 **错误拦截器** 才是处理 token 过期的地方（响应拦截器）
 （而不是请求拦截器）

通常逻辑是这样的：

##### 🧱 请求拦截器

只加头：

```js
if (token) {
  config.headers.Authorization = `Bearer ${token}`
}
```

##### 🧱 响应拦截器

统一判断 token 是否过期：

```js
if (res.code === 401) {
  logout();
  redirectToLogin();
}
```

这也是绝大多数开源后台框架的套路。





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





### 🚀 **常用 axios 封装示例**



使用刷新 token (refresh token) 体系

`accessToken` 有效期很短

`refreshToken` 自动续期

`src/utils/request.ts`

```ts
import axios, { type InternalAxiosRequestConfig, type AxiosResponse } from "axios";
import qs from "qs";
import { useUserStoreHook } from "@/store/modules/user.store";
import { ResultEnum } from "@/enums/api/result.enum";
import { getAccessToken } from "@/utils/auth";
import router from "@/router";

// 创建 axios 实例
const service = axios.create({
  baseURL: import.meta.env.VITE_APP_BASE_API,
  timeout: 50000,
  headers: { "Content-Type": "application/json;charset=utf-8" },
  paramsSerializer: (params) => qs.stringify(params),
});
// 请求拦截器
service.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const accessToken = getAccessToken();
    // 如果 Authorization 设置为 no-auth，则不携带 Token
    if (config.headers.Authorization !== "no-auth" && accessToken) {
      config.headers.Authorization = `Bearer ${accessToken}`;
    } else {
      delete config.headers.Authorization;
    }
    return config;
  },
  (error) => Promise.reject(error)
);
// 响应拦截器
service.interceptors.response.use(
  (response: AxiosResponse) => {
    // 如果响应是二进制流，则直接返回，用于下载文件、Excel 导出等
    if (response.config.responseType === "blob") {
      return response;
    }
    const { code, data, msg } = response.data;
    if (code === ResultEnum.SUCCESS) {
      return data;
    }
    ElMessage.error(msg || "系统出错");
    return Promise.reject(new Error(msg || "Error"));
  },
  async (error) => {
    console.error("request error", error); // for debug
    const { config, response } = error;
    if (response) {
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
export default service;
// 是否正在刷新标识，避免重复刷新
let isRefreshing = false;
// 因 Token 过期导致的请求等待队列
const waitingQueue: Array<() => void> = [];
// 刷新 Token 处理
async function handleTokenRefresh(config: InternalAxiosRequestConfig) {
  return new Promise((resolve) => {
    // 封装需要重试的请求
    const retryRequest = () => {
      config.headers.Authorization = `Bearer ${getAccessToken()}`;
      resolve(service(config));
    };
    waitingQueue.push(retryRequest);
    if (!isRefreshing) {
      isRefreshing = true;
      useUserStoreHook()
        .refreshToken()
        .then(() => {
          // 依次重试队列中所有请求, 重试后清空队列
          waitingQueue.forEach((callback) => callback());
          waitingQueue.length = 0;
        })
        .catch(async (error) => {
          console.error("handleTokenRefresh error", error);
          // 刷新 Token 失败，跳转登录页
          await handleSessionExpired();
        })
        .finally(() => {
          isRefreshing = false;
        });
    }
  });
}
// 处理会话过期
async function handleSessionExpired() {
  ElNotification({
    title: "提示",
    message: "您的会话已过期，请重新登录",
    type: "info",
  });
  await useUserStoreHook().clearSessionAndCache();
  router.push("/login");
}
```







