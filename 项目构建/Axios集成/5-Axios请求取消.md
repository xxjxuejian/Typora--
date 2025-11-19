### 🌟 一句话先讲明白：为什么需要请求取消？

**因为有些请求是“无意义的”或“过时的”，继续等待只会浪费资源、造成状态错乱、甚至触发异常。**

比如：

#### ✔ 场景 1：快速输入搜索框（联想搜索）

用户输入 `a` → 发请求
 立刻输入 `ap` → 又发请求
 再输入 `app` → 再发请求

如果不取消前面的请求：

- 旧请求可能比新的更晚回来
- 页面会被**旧数据覆盖新数据**
- 搜索结果抖来抖去，用户以为前端“抽风”

请求取消可避免这一切。

------

#### ✔ 场景 2：页面切换（特别是表格页）

你 A 页面发了请求，还没回来，就跳去 B 页面。
 这时候请求回来了，会发生：

- setState / 更新组件，却组件已经卸载 ⚠
- Vue 报错：“尝试更新已卸载的组件”
- React 组件更新状态警告
- 内存泄漏风险

取消请求 = 让它彻底别回来捣乱。

------

#### ✔ 场景 3：重复点击按钮（提交表单）

用户一激动连续点“提交”，发了 3 个请求：

- 会产生重复数据
- 后端会被你连续戳

Abort 可以 **阻止前几个请求执行**。

------

#### ✔ 场景 4：轮询 / 定时刷新

当用户离开页面后，之前的轮询请求必须中断，否则浪费资源。

------



### 🧨 那 request abort 是怎么用的？

axios 原生 v0.x 不支持 abort controller（但是可以用 AbortController + adapter），
 axios v1.x 开始已经支持 **fetch 风格的 abort**。

我给你写一份“现代 + 好理解 + 易用”的封装。

------

### 🚀 典型 axios + AbortController 封装（通用）

```js
// src/utils/request.ts
import axios, { AxiosRequestConfig } from "axios";

const controllerMap = new Map<string, AbortController>();

function getRequestKey(config: AxiosRequestConfig) {
  return `${config.method}-${config.url}`;
}

const service = axios.create({
  baseURL: import.meta.env.VITE_BASE_API,
  timeout: 8000,
});

// 请求拦截器
service.interceptors.request.use((config) => {
  const key = getRequestKey(config);

  // 如果已有同类型请求，则取消前一个
  if (controllerMap.has(key)) {
    controllerMap.get(key)!.abort();   // 取消旧请求
    controllerMap.delete(key);
  }

  // 新建一个 controller
  const controller = new AbortController();
  config.signal = controller.signal;
  controllerMap.set(key, controller);

  return config;
});

// 响应拦截器
service.interceptors.response.use(
  (response) => {
    const key = getRequestKey(response.config);
    controllerMap.delete(key);
    return response.data;
  },
  (error) => {
    if (axios.isCancel(error)) {
      console.warn("请求被取消：", error.message);
    }
    return Promise.reject(error);
  }
);

export default service;
```

------



### 🍜 如何使用？

#### 自动取消前一个同类型请求

适用于搜索框

```js
request.get("/search", { q: keyword });
```

你快速连续输入时，前一次搜索会自动被 abort。

------

#### 🧩 手动取消请求（高级版）

如果你要在组件卸载时取消请求，也可以手动控制：

```js
const controller = new AbortController();

request.get("/list", {
  signal: controller.signal
});

// 当组件卸载
controller.abort();
```

------



### 💡 再强调一次：为什么一定要用请求取消？

给你总结三点，超级核心的👇：

#### 🟣 1. **避免竞态条件（race condition）**

大部分后台系统都有复杂搜索和表格加载，你绝对不想旧请求覆盖新请求。

#### 🟣 2. **避免内存泄漏**

特别是频繁切换页面时候。

#### 🟣 3. **节省资源**

减少重复无意义请求，提升性能。