### 第一部分：JWT (JSON Web Token) 详解

#### 1. 什么是 JWT？
JWT 本质上是一个开放标准（RFC 7519），它定义了一种紧凑且自包含的方式，用于在各方之间以 JSON 对象安全地传输信息。

**核心逻辑：** “我（服务器）给你（客户端）一张盖了公章的纸条，以后你办事亮出这张纸条，我验一下公章，就知道你是谁了，不用再去档案室（数据库）查你的资料。”

#### 2. JWT 的结构
JWT 是一个字符串，由三个部分组成，中间用点 `.` 分隔：
`Header.Payload.Signature`

*   **Part 1: Header (头部)**
    *   声明类型（JWT）和加密算法（如 HMAC SHA256）。
    *   *例如：* `{"alg": "HS256", "typ": "JWT"}`
*   **Part 2: Payload (负载)**
    *   这是存放有效信息的地方（Claims）。
    *   **标准字段：** `iss` (签发人), `exp` (过期时间), `sub` (面向的用户).
    *   **自定义字段：** `userId`, `role`, `username`.
    *   **注意：** 这里的数据是 Base64 编码的，**可以被解码看到明文**，所以**千万不要放密码等敏感信息**。
*   **Part 3: Signature (签名)**
    *   这是防止篡改的关键。
    *   **生成公式：** `HMACSHA256(base64UrlEncode(Header) + "." + base64UrlEncode(Payload), secret)`
    *   只有服务器持有 `secret`（密钥），如果黑客修改了 Payload 中的权限（比如把普通用户改成管理员），由于他不知道密钥，生成的签名就会和 Header 中的签名不一致，验证就会失败。

#### 3. JWT 的优缺点
*   **优点：**
    *   **无状态：** 服务器不保存 Session，节省内存，天生支持分布式/微服务。
    *   **解析快：** 只要验签通过，Payload 里就有用户 ID 和角色，不用每次查库。
*   **缺点：**
    *   **一经签发，不可撤销：** 只要没过期，Token 丢了别人也能用。这也是为什么需要双 Token 机制的原因。

---

### 第二部分：双 Token 机制 (Access Token + Refresh Token)

#### 1. 为什么要搞两个 Token？
在单 Token 方案中，我们面临一个两难的选择：
*   **Token 有效期设长了（比如 7 天）：** 用户体验好，不用老登录。但是如果不小心泄漏，黑客能连续 7 天操作你的账户，很不安全。
*   **Token 有效期设短了（比如 30 分钟）：** 很安全。但是用户每 30 分钟就被踢下线要求重登，体验极差。

**双 Token 机制就是为了平衡“安全”与“体验”。**

#### 2. 两个 Token 的分工
*   **Access Token (短效 Token):**
    *   **有效期：** 极短（通常 10 - 30 分钟）。
    *   **作用：** 专门用于请求业务接口（如获取列表、保存数据）。
    *   **特点：** 就算泄漏，黑客也只能用十几分钟，危害可控。
*   **Refresh Token (长效 Token):**
    *   **有效期：** 较长（通常 7 天 - 30 天）。
    *   **作用：** **只用于** 在 Access Token 过期时，向服务器换取一个新的 Access Token。
    *   **特点：** 必须保存在服务端（数据库或 Redis）有记录，且通常绑定客户端指纹，安全性要求极高。

#### 3. 详细交互流程
这是一个非常经典的“无感刷新”流程：

1.  **登录：**
    *   用户输入账号密码。
    *   服务端校验成功，生成 `Access Token` 和 `Refresh Token`。
    *   服务端将 `Refresh Token` 存入 Redis（用于后续校验或强制下线）。
    *   客户端收到两个 Token 并存储。

2.  **正常请求：**
    *   客户端请求业务接口，Header 携带 `Access Token`。
    *   服务端验证 `Access Token` 有效，返回数据。

3.  **Token 过期（核心步骤）：**
    *   客户端请求接口，携带 `Access Token`。
    *   服务端发现 `Access Token` 过期，返回特定状态码（如 `401 Unauthorized` 或自定义代码 `4001`）。
    *   **前端拦截器** 捕获到这个错误代码。
    *   前端**暂停**当前的所有业务请求，单独发送一个请求到 `/refresh` 接口，携带 `Refresh Token`。

4.  **刷新 Token：**
    *   服务端验证 `Refresh Token`：
        1.  验签是否正确。
        2.  检查是否过期。
        3.  **关键：** 检查 Redis 中是否存在这个 `Refresh Token`（这一步让服务端拥有了“踢人”的权利）。
    *   验证通过后，生成**新的** `Access Token` （通常建议同时也生成新的 `Refresh Token` 实现轮换）返回给前端。

5.  **重试：**
    *   前端拿到新 Token，替换旧 Token。
    *   前端**自动重发**刚才那个失败的请求，用户完全感知不到 Token 发生过变化。

6.  **强制重新登录：**
    *   如果 `Refresh Token` 也过期了（比如用户一个月没来了），或者 Redis 里查不到（被管理员踢了）。
    *   服务端返回“登录失效”，前端跳转回登录页。

#### 4. 双 Token 方案如何实现“踢人下线”？
单 JWT 做不到踢人，但双 Token 可以：
*   管理员在后台点击“强制下线”。
*   服务端从 Redis 中删除该用户的 `Refresh Token`。
*   该用户的 `Access Token` 会在几分钟内自然过期。
*   当他想用 `Refresh Token` 换新票时，服务端发现 Redis 里没有记录，拒绝刷新，用户就被踢出去了。

---

### 第三部分：前端实现细节（避坑指南）

在前端（Vue/React）实现双 Token 时，有几个关键的技术点：

1.  **存储策略：**
    *   **Access Token:** 建议存在内存（Vuex/Pinia）中，刷新页面会丢失，丢失了就用 Refresh Token 重换。这样最安全，防止 XSS 攻击窃取。也可以存 `localStorage`。
    *   **Refresh Token:** 最安全的做法是存放在 **HttpOnly Cookie** 中（防止 JS 读取），或者存 `localStorage`。

2.  **并发刷新问题（重要）：**
    *   **场景：** 页面并发发出了 3 个请求，Access Token 刚好过期。这 3 个请求都会返回 401。
    *   **问题：** 如果不做处理，前端会发 3 次刷新请求，导致服务器生成了 3 个新 Token，甚至报错。
    *   **解决：** 使用一个标志位 `isRefreshing`。
        *   当第一个请求发现过期，将 `isRefreshing = true`，并发起刷新。
        *   后续的请求发现 `isRefreshing` 为 true，不要发起刷新，而是把请求挂起到一个**队列**（Promise 队列）中。
        *   等第一个刷新请求成功返回新 Token 后，遍历队列，把新 Token 填进去，重新发送所有挂起的请求。

### 总结
*   **JWT：** 是一张自包含的身份证明。
*   **双 Token：** 是 **JWT 的最佳实践**。
    *   **Access Token** 负责挡子弹（高频使用，过期快）。
    *   **Refresh Token** 负责保安全（低频使用，可控性强）。
    这是目前后台管理系统最推荐的认证架构。



### 交互流程示例（部分代码）

1. **登录：**

   登录成功以后，前端收到两个Token，然后存储

   ```ts
     function login(LoginFormData: LoginFormData) {
       return new Promise<void>((resolve, reject) => {
         AuthAPI.login(LoginFormData)
           .then((data) => {
             const { accessToken, refreshToken } = data;
             setAccessToken(accessToken); // eyJhbGciOiJIUzI1NiJ9.xxx.xxx
             setRefreshToken(refreshToken);
             resolve();
           })
           .catch((error) => {
             reject(error);
           });
       });
     }
   ```

   ```ts
   // 访问 token 缓存的 key
   const ACCESS_TOKEN_KEY = "access_token";
   // 刷新 token 缓存的 key
   const REFRESH_TOKEN_KEY = "refresh_token";
   
   function setAccessToken(token: string) {
     localStorage.setItem(ACCESS_TOKEN_KEY, token);
   }
   
   function setRefreshToken(token: string) {
     localStorage.setItem(REFRESH_TOKEN_KEY, token);
   }
   ```

   

2. **正常请求：**

   客户端的业务请求，在Axios请求拦截器中添加`Access Token`

   请求拦截器中，只考虑添加`Access Token`，不考虑是否过期，过期是交给服务器验证的，

   ```ts
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
   ```

   ```ts
   const ACCESS_TOKEN_KEY = "access_token"; // 访问 token 缓存的 key
   const REFRESH_TOKEN_KEY = "refresh_token"; // 刷新 token 缓存的 key
   
   function getAccessToken(): string {
     return localStorage.getItem(ACCESS_TOKEN_KEY) || "";
   }
   
   function getRefreshToken(): string {
     return localStorage.getItem(REFRESH_TOKEN_KEY) || "";
   }
   ```

   这里获取`Token`要么就是`空字符串""`要么就是已经`过期`，总之，后端会校验的，前端这里添加就行了

   

3. **Token过期**

   假如过期了，那么会返回特定的状态码，具体是什么状态码，那就是后端定义的了。

   ```ts
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
   ```

   ` Token`过期时，是进入响应拦截器的第一个回调，还是第二个回调，这取决于后端返回的`http`状态码

   上面的代码应该是针对`token`过期问题，让`http`状态码为`非200`，进入了第二个回调，不过这不重要。

   当然这里Token过期，可能是Access Token 或者 Refresh Token 过期，具体是哪一个，会返回不同的状态码，分别处理。

   

4. 刷新Token

   **a.** 首先是`Access Token过期`

   ​	这时候就是要借助**Refresh Token**，重新获取新的**Access Token**（通常建议同时也生成新的 **Refresh Token** 实现轮换）返回给前端。

   这里存在一个问题：

   **场景：** 页面并发发出了 3 个请求，`Access Token` 刚好过期。这 3 个请求都会返回 401（或者其它）。

   **问题：** 如果不做处理，前端会发 3 次刷新请求，导致服务器生成了 3 个新 `Token`，甚至报错。

   **解决：** 使用一个标志位 `isRefreshing`。

   - 当第一个请求发现过期，将 `isRefreshing = true`，并发起刷新。
   - 后续的请求发现 `isRefreshing` 为 true，不要发起刷新，而是把请求挂起到一个**队列**（Promise 队列）中。
   - 等第一个刷新请求成功返回新 Token 后，遍历队列，把新 Token 填进去，重新发送所有挂起的请求。

   

   触发了响应回调时，是可以获取到本次的请求的`config`的

   ```ts
     const { config, response } = error;
   ```

   `config`中是包含了这个请求的各种配置的，比如`url`，`method`，`data`，`params`等等，相当于拿到了这个`config`，就可以利用它再次发送请求。

   

   假如第一个使用过期token的请求，这时候进入到`handleTokenRefresh`中处理，在它获取新的token期间，

   剩下的请求也会进去这个函数，但是他们会被卡在`if (!isRefreshing)`，不会调用多次刷新token的接口，所以不会生成多个新的token；这些请求全部进入了`waitingQueue`队列

   当第一个过期请求，获取了新的`token`以后，就请求`waitingQueue`队列，重新发送所有挂起的请求。

   ```ts
   // 是否正在刷新标识，避免重复刷新
   let isRefreshing = false;
   // 因 Token 过期导致的请求等待队列
   const waitingQueue: Array<() => void> = [];
   
   // 刷新 Token 处理
   async function handleTokenRefresh(config: InternalAxiosRequestConfig) {
     return new Promise((resolve) => {
       // 封装需要重试的请求
       // 这里只是函数定义，不会执行
       const retryRequest = () => {
         // 情况队列时，执行这个函数，这时候有了新的token,所以要更新
         // 当前config中的Authorization值
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

   

   b. 其次是`Refresh Token过期`

   这个情况比较简单，直接情况数据，跳转到登录页面，重新登录。

   不过最好是获取`Acess Token`时，也生成新的`Refresh Token`