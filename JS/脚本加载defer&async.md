### 🌱 **整体大逻辑：HTML 解析优先，但会被脚本“打断”**

浏览器在加载页面时，其核心流程是：

1. **加载 HTML → 解析成 DOM**
2. **遇到资源标签（script、link、img…）决定要不要停下来等它**
3. **等所有必须的东西完成后 → DOM 构建完毕（DOMContentLoaded）**

但关键点来了：

#### **普通 `<script>` 会中断 DOM 解析**

因为浏览器需要先执行脚本，以防脚本修改后续 DOM（比如 `document.write`）。

------



### 🧱 **详细顺序：DOM & script 的相爱相杀**

我按场景给你讲，最清楚～

------

#### 🧩 **情况 1：普通 `<script>`（同步 script）**

```html
<p>1</p>
<script src="a.js"></script>
<p>2</p>
```

浏览器流程：

1. 解析到 `<p>1</p>` → 加到 DOM。
2. 解析到 `<script>`
    ⛔ **停止构建 DOM**
3. 下载 a.js（若未缓存）
4. **立即执行 a.js**
5. 执行完继续解析 → `<p>2</p>` 加到 DOM

👉 **同步 script 会阻塞 DOM 解析与渲染。**

------



#### 🧩 **情况 2：`<script defer>`**

```html
<p>1</p>
<script src="a.js" defer></script>
<p>2</p>
```

流程：

1. 解析 HTML → DOM 正常往下
2. 遇到 `script defer`：
    ➜ **异步下载，但不阻塞 DOM**
3. DOM 全部构建完成后，但在 `DOMContentLoaded` 事件之前执行。
    ➜ **按顺序执行所有 defer 脚本**

👉 特点：

- **不阻塞 DOM**
- **执行顺序按页面出现顺序**
- **在 DOM 完成后执行**

------



**具有 `defer` 特性的脚本保持其相对顺序，就像常规脚本一样。**

假设，我们有两个具有 `defer` 特性的脚本：`long.js` 在前，`small.js` 在后。

```html
<script defer src="https://javascript.info/article/script-async-defer/long.js"></script>
<script defer src="https://javascript.info/article/script-async-defer/small.js"></script>
```

浏览器扫描页面寻找脚本，然后**并行下载**它们，以提高性能。因此，在上面的示例中，两个脚本是并行下载的。`small.js` 可能会先下载完成。

……但是，`defer` 特性除了告诉浏览器“不要阻塞页面”之外，还可以确保脚本执行的相对顺序。因此，即使 `small.js` 先加载完成，它也需要等到 `long.js` 执行结束才会被执行。

**当我们需要先加载 JavaScript 库，然后再加载依赖于它的脚本时，这可能会很有用。**

> **`defer` 特性仅适用于外部脚本**
>
> 如果 `<script>` 脚本没有 `src`，则会忽略 `defer` 特性。



#### 🧩 **情况 3：`<script async>`**

```html
<p>1</p>
<script src="a.js" async></script>
<p>2</p>
```

流程：

1. 解析 HTML → DOM 正常往下
2. 遇到 async script：
    ➜ **异步下载，不阻塞 DOM**
3. 下载完后：
    ⛔ **立即执行（会打断 DOM 解析）**

👉 特点：

- **不阻塞下载**
- 🚨 **下载完成时会阻塞 DOM 解析来执行**
- **执行顺序不保证**

------



`async` 特性意味着脚本是完全独立的：

- 浏览器不会因 `async` 脚本而阻塞（与 `defer` 类似）。

- 其他脚本不会等待 `async` 脚本加载完成，同样，`async` 脚本也不会等待其他脚本。

  `DOMContentLoaded`和异步脚本不会彼此等待：

  - `DOMContentLoaded` 可能会发生在异步脚本之前（如果异步脚本在页面完成后才加载完成）
  - `DOMContentLoaded` 也可能发生在异步脚本之后（如果异步脚本很短，或者是从 HTTP 缓存中加载的）

换句话说，`async` 脚本会在后台加载，并在加载就绪时运行。DOM 和其他脚本不会等待它们，它们也不会等待其它的东西。`async` 脚本就是一个会在加载完成时执行的完全独立的脚本。

```html
<p>...content before scripts...</p>

<script>
  document.addEventListener('DOMContentLoaded', () => alert("DOM ready!"));
</script>

<script async src="https://javascript.info/article/script-async-defer/long.js"></script>
<script async src="https://javascript.info/article/script-async-defer/small.js"></script>

<p>...content after scripts...</p>
```

- 页面内容立刻显示出来：加载写有 `async` 的脚本不会阻塞页面渲染。
- `DOMContentLoaded` 可能在 `async` 之前或之后触发，不能保证谁先谁后。
- 较小的脚本 `small.js` 排在第二位，但可能会比 `long.js` 这个长脚本先加载完成，所以 `small.js` 会先执行。虽然，可能是 `long.js` 先加载完成，如果它被缓存了的话，那么它就会先执行。
- 换句话说，异步脚本以“加载优先”的顺序执行。（**先下载完成先执行**）



当我们将**独立的第三方脚本**集成到页面时，此时采用**异步加载**方式是非常棒的：**计数器，广告等**，因为它们**不依赖于**我们的脚本，**我们的脚本也不应该等待它们**：

```html
<!-- Google Analytics 脚本通常是这样嵌入页面的 -->
<script async src="https://google-analytics.com/analytics.js"></script>
```



> **`async` 特性仅适用于外部脚本**
>
> 就像 `defer` 一样，如果 `<script>` 标签没有 `src` 特性（attribute），那么 `async` 特性会被忽略。



#### 🧩 **情况 4：模块脚本 `<script type="module">`**

```html
<script type="module" src="main.js"></script>
```

特点：

- **默认 defer 行为**（不阻塞 DOM，按顺序执行）
- 但 **import 的模块是异步加载**
   且每个模块只初始化一次（模块缓存）

------

### 🌿 **最终总结图（你可以直接截图保存）**

| 类型            | 下载是否阻塞 DOM | 执行是否阻塞 DOM | 执行时机         | 执行顺序                 |
| --------------- | ---------------- | ---------------- | ---------------- | ------------------------ |
| **普通 script** | ✔️阻塞            | ✔️阻塞            | 立即             | 按出现顺序               |
| **async**       | ❌不阻塞          | ✔️阻塞            | 下载完成立即执行 | 不保证顺序               |
| **defer**       | ❌不阻塞          | ❌不阻塞          | DOM 完成后执行   | 按出现顺序               |
| **module**      | ❌不阻塞          | ❌不阻塞          | DOM 完成后执行   | 按出现顺序（类似 defer） |





## [总结](https://zh.javascript.info/script-async-defer#zong-jie)

`async` 和 `defer` 有一个共同点：加载这样的脚本都不会阻塞页面的渲染。因此，用户可以立即阅读并了解页面内容。

但是，它们之间也存在一些本质的区别：

|         | 顺序                                                         | `DOMContentLoaded`                                           |
| :------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `async` | **加载优先顺序**。脚本在文档中的顺序不重要 —— 先加载完成的先执行 | 不相关。可能在文档加载完成前加载并执行完毕。如果脚本很小或者来自于缓存，同时文档足够长，就会发生这种情况。 |
| `defer` | **文档顺序**（它们在文档中的顺序）                           | 在文档加载和解析完成之后（如果需要，则会等待），即在 `DOMContentLoaded` 之前执行。 |

在实际开发中，`defer` 用于需要整个 DOM 的脚本，和/或脚本的相对执行顺序很重要的时候。

`async` 用于**独立脚本**，例如计数器或广告，这些**脚本的相对执行顺序无关紧要**。



> **没有脚本的页面应该也是可用的**
>
> 请注意：如果你使用的是 `defer` 或 `async`，那么用户将在脚本加载完成 **之前** 先看到页面。
>
> 在这种情况下，某些图形组件可能尚未初始化完成。
>
> 因此，请记得添加一个**“正在加载”的提示**，并禁用尚不可用的按钮。以让用户可以清楚地看到，他现在可以在页面上做什么，以及还有什么是正在准备中的。