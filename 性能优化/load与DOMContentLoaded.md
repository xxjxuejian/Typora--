## 先给一句“人话版结论”

- **DOMContentLoaded**
  👉 **DOM 树搭好了，可以开始干活了**（JS 可以放心操作 DOM）
- **load**
  👉 **页面“全员到齐”了**（图片 / 字体 / CSS / iframe 全部加载完成）

一句话差别：

> **DOMContentLoaded 关心“结构”，load 关心“全部资源”**

------

## 一条完整的加载时间线（重点）

```css
开始加载 HTML
│
├─ 解析 HTML
│   ├─ 遇到普通 <script> → 停下来执行
│   ├─ 遇到 async script → 一下完就执行
│   ├─ 遇到 defer script → 先记着
│
├─ DOM 树构建完成
│   └─ 🔔 DOMContentLoaded 触发
│
├─ defer 脚本已执行完
│
├─ 图片 / 字体 / background-image / iframe
│   陆续加载完成
│
└─ 🔔 window load 触发
```

你可以把它想成：

- DOMContentLoaded：**房子结构搭完**
- load：**家具家电窗帘全部进屋**

------

## DOMContentLoaded：JS 的“黄金时间点”

当 HTML 文档完全解析，且所有延迟脚本`<script defer src='...'>`和`<script type='module'>`下载和执行完毕后，会触发 **`DOMContentLoaded`** 事件。它不会等待图片、子框架和异步脚本等其他内容完成加载。

### 什么时候触发？

> **HTML 全部解析完 + DOM 树构建完成**

**不等这些：**

- 图片
- 视频
- 字体
- CSS background-image

### 常见用法

```js
document.addEventListener("DOMContentLoaded", () => {
  // 此时可以安全地：
  // - querySelector
  // - 绑定事件
  // - 初始化组件
});
```

### 为什么它这么重要？

因为：

- 你操作 DOM，只需要 **DOM 存在**
- 等图片？**纯属浪费时间**

👉 这也是为什么：

- Vue / React 的挂载时机
- 大量初始化逻辑
  都发生在 **DOMContentLoaded 之后**

------

## load：一个“非常保守”的事件

### 什么时候触发？

> **所有资源加载完成**

包括：

- `<img>`
- CSS 中的背景图
- webfont
- iframe
- JS（当然）

### 示例

```js
window.addEventListener("load", () => {
  // 页面真的“完全准备好”了
});
```

### 那它还有用吗？

有，但 **很少**，而且很“重”。

**典型场景：**

- 依赖图片尺寸的计算（不只是 DOM）
- 老代码里的一些兜底初始化
- 需要“确保一切都在”的统计/打点

------

## 和你之前问的 lazy / preload 有关系吗？有！

### 1️⃣ 图片 lazy 会影响 load 吗？

**会。**

- `loading="lazy"` 的图片：
  - **不在首屏**
  - 不会阻塞 `load`
- 也就是说：
  - lazy 用得好 → **load 会提前**

### 2️⃣ preload 会影响 DOMContentLoaded 吗？

**基本不会。**

- preload 只是提前下载
- 不阻塞 DOM 解析
- 但会影响：
  - LCP
  - load（可能更早）

------

## 一个很工程化的判断标准 🧠

你可以这样选：

> **“我现在做的事，到底依赖什么？”**

- 只依赖 DOM
  👉 **DOMContentLoaded**
- 依赖图片/字体真实尺寸
  👉 **load**
- 两者都不依赖
  👉 **尽量更早（甚至同步）**

------

## 一个小但很重要的误区 ⚠️

### ❌ `load` 一定比 `DOMContentLoaded` 晚很多？

不一定。

- 页面很简单
- 图片很少 / 已缓存
- 资源很小

👉 两个事件可能**几乎同时**

但工程上：

> **永远不要假设它们“差不多”**

------

## 最后，用一句可爱的总结 🐣

- DOMContentLoaded：
  🧠「骨架就位，灵魂上线」
- load：
  🎉「全员到齐，开门营业」

如果你愿意，下一步我可以帮你：

- 把 **DOMContentLoaded / load / LCP / TTI** 串成一张**真正可用于调优的时间线**
- 或结合你 Vue + Vite 的项目，说说哪些钩子本质上落在 DOMContentLoaded 前后

你已经在“性能理解”的正确轨道上了 🚀