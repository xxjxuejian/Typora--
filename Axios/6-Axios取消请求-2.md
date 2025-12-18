在二次封装 Axios 时结合拦截器实现取消请求，通常是为了达成一个**全局自动化目标**：**取消重复请求**。

即：当一个请求还在 `pending` 状态时，如果又发起了一个**一模一样**的请求（相同的 URL、Method、Params），则默认视为用户操作过快（如狂点按钮），此时拦截器应该**自动取消前一个请求**，只保留最新的。

下面是一个企业级的封装方案，包含了**自动取消重复请求**和**允许手动取消**的兼容处理。

---

### 完整的封装方案 (`request.js`)

我们需要维护一个 **Map** 来存储正在进行中的请求，以便在拦截器中查找并取消它们。

#### 1. 核心工具类：管理 Pending 请求

首先定义一个类（或简单的对象）来管理 `Map`。

```javascript
// requestCanceler.js
import axios from 'axios';

// 用于存储每个请求的标识和取消函数
const pendingMap = new Map();

/**
 * 生成请求的唯一 Key
 * 格式：[method] url?paramString
 */
const getPendingKey = (config) => {
  return [
    config.method,
    config.url,
    JSON.stringify(config.params),
    JSON.stringify(config.data)
  ].join('&');
};

/**
 * 添加请求到 Map
 */
export const addPending = (config) => {
  // 如果当前请求没有配置 signal (即用户没有自己传入 AbortController)，我们才帮他创建
  if (!config.signal) {
    const controller = new AbortController();
    config.signal = controller.signal;
    
    // 把 controller 存下来，以便后续取消
    // 注意：这里把 controller 挂载到 config 上，方便 removePending 使用
    config.abortController = controller; 
  }
  
  const key = getPendingKey(config);
  
  // 如果 Map 中已经存在该 Key，说明有重复请求，需要取消前一个
  if (pendingMap.has(key)) {
    const abortController = pendingMap.get(key);
    abortController.abort('取消重复请求'); // 触发取消
    pendingMap.delete(key);
  }
  
  // 将当前的 controller 存入 Map (如果用户自己传了 signal，我们无法获取 controller，这里需要根据业务决定是否仅支持自动生成的 signal)
  // 为了简化，这里假设主要处理自动取消场景：
  if (config.abortController) {
    pendingMap.set(key, config.abortController);
  }
};

/**
 * 移除请求 (请求完成或报错时调用)
 */
export const removePending = (config) => {
  const key = getPendingKey(config);
  if (pendingMap.has(key)) {
    pendingMap.delete(key);
  }
};
```

#### 2. Axios 拦截器封装

接下来在你的 `axios` 实例中使用这个工具类。

```javascript
// service.js
import axios from 'axios';
import { addPending, removePending } from './requestCanceler';

const service = axios.create({
  baseURL: '/api',
  timeout: 10000,
});

// --- 请求拦截器 ---
service.interceptors.request.use(
  (config) => {
    // 1. 在请求发送前，加入 Pending 队列（会自动检查并取消重复的）
    // 你可以通过配置个别参数来跳过自动取消，例如 config.repeatAllowed = true
    if (!config.repeatAllowed) {
      addPending(config);
    }
    
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// --- 响应拦截器 ---
service.interceptors.response.use(
  (response) => {
    // 2. 请求成功返回，从队列中移除
    removePending(response.config);
    return response.data;
  },
  (error) => {
    // 3. 请求失败或被取消，也要从队列中移除
    if (error.config) {
      removePending(error.config);
    }

    // 4. 特殊处理取消请求的报错
    if (axios.isCancel(error)) {
      console.log('请求被拦截取消了:', error.message);
      // 关键点：中断 Promise 链，防止前端报错
      // 返回一个 pending 的 Promise，或者返回特定格式的对象
      return new Promise(() => {}); 
    }

    // 处理其他 HTTP 错误
    return Promise.reject(error);
  }
);

export default service;
```

---

### 使用方式

#### 1. 自动取消重复请求（默认生效）

在组件中，如果你连续点击按钮触发请求，拦截器会自动取消前一次，只发最后一次。

```javascript
import service from './service';

// 用户狂点按钮
const handleClick = () => {
    // 第一次点击：发起请求 A
    service.get('/user/list'); 
    
    // 第二次点击（假设 A 还没回）：拦截器发现重复，取消 A，发起请求 B
    service.get('/user/list'); 
};
```

#### 2. 手动取消请求（传入 signal）

如果某个特定场景（如组件销毁、点击取消按钮）需要手动控制，可以在调用封装好的 `service` 时传入 `signal`。

> **注意：** 在上面的封装逻辑中，如果检测到用户传入了 `signal`，`addPending` 应该做兼容处理（不覆盖用户的 signal）。

```vue
<script setup>
import { onUnmounted } from 'vue';
import service from './service';

const controller = new AbortController();

const fetchData = () => {
  service.get('/user/large-data', {
    signal: controller.signal, // 手动传入
    repeatAllowed: true // 告诉拦截器不要自动去重（可选）
  });
};

const cancel = () => {
  controller.abort('用户手动停止');
};

onUnmounted(() => {
  controller.abort();
});
</script>
```

---

### 高级优化：白名单配置

并不是所有重复请求都需要取消。例如：
*   **埋点上报**：用户快速点击两个按钮，应该上报两次，而不是取消前一次。
*   **并发加载**：同一个接口，参数不同（Key 会不同，不会冲突）。
*   **轮询**：有时需要保持上一个还在连接中。

你可以在封装时约定一个配置项，比如 `enableCancel` 或 `repeatAllowed`。

**修改 addPending 逻辑：**

```javascript
export const addPending = (config) => {
  // 如果配置了 repeatAllowed: true，则直接跳过取消逻辑
  if (config.repeatAllowed) return; 

  // ...原有的取消逻辑
};
```

**调用时：**

```javascript
// 这个请求就算狂点，也会发出多次，不会被取消
service.post('/log/click', { id: 1 }, { repeatAllowed: true });
```

### 总结

结合拦截器使用的核心在于：

1.  **Request 拦截器**：生成请求指纹（Key），检查 `Map` 中是否存在。若存在 -> `abort()` 旧请求 -> 移除旧请求 -> 添加新请求。
2.  **Response 拦截器**：无论成功还是失败，根据 config 从 `Map` 中移除记录。
3.  **错误处理**：利用 `axios.isCancel` 捕获异常，避免控制台飘红。