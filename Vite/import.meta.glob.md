

这个东西**不是 JavaScript 原生的**，而是 **Vite 独家 buff**（其实是早年 Snowpack、Rollup 插件家族延伸出来的那种想法）。

我给你一个超级直白的小结：

> **`import.meta.glob()` = 让你用一个正则（或通配符）一次性“按需导入一堆文件”的构建期 API。**
>  有点像“懒加载版的万能文件批量收集器”。



### 🎯 1. 它长什么样？

```js
const modules = import.meta.glob('@/views/**/**.vue')
```

`import.meta.glob('@/views/**/**.vue')`这是**Vite 提供的内置语法**，用于实现**基于文件路径的动态导入**。其中的`@`符号，经过**Vite**解析为`/src`。这是在`vite.config.js`中的配置

```js
// 设置别名@
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },
```

`@/views/`文件夹下面，保存的是系统全部的组件（`.vue`文件）。

`import.meta.glob('@/views/**/**.vue')`会将这些`.vue`文件的路径作为`key`，对应的组件的懒加载函数作为`value`。

返回的结果`modules`是一个对象，结构如下：

```js
{
  '/src/views/product-manage/on-sale/index.vue': () => import('/src/views/product-manage/on-sale/index.vue'),
  '/src/views/product-manage/pre-sale/index.vue': () => import('/src/views/product-manage/pre-sale/index.vue'),
  ...
}
```

每个 `key` 是文件路径

每个 `value` 是一个“**懒加载函数**”：调用这个函数时会动态加载对应组件（懒加载）

也就是说，文件没有立刻加载，只有你调用：

```js
modules['/src/views/product-manage/on-sale/index.vue']().then(...)
```

它才真的去 import。



### 🌈 2. 为什么需要它？

想象一下你要实现一个“动态路由”或“批量引入 markdown 文章”的功能。

如果不用 `import.meta.glob()`，你会这么做：

- 手写几十行 `import fileA …`
- 或者使用 Webpack 的 require.context（不过现在已经显得老了）

有了 `import.meta.glob()`：

- 不用写循环
- 不用关心构建时怎么处理
- 不用管 tree-shaking
- 不用担心 Vite 热更新问题

一句话：**自动化 + 懒加载 + 性能友好。**





### 📦 3. 还可以“直接加载内容”

如果你想立刻加载文件内容，而不是返回懒加载函数：

```js
const modules = import.meta.glob('./snippets/*.txt', { eager: true })
```

此时返回的是：

```js
{
  './snippets/a.txt': 'file content...',
  './snippets/b.txt': 'another content...',
}
```

特别适合加载 markdown、json、配置文件内容。





### 🧊 4. 还可以加 import 的配置

比如：

```js
const posts = import.meta.glob('./posts/*.md', {
  import: 'meta', // 只导入 frontmatter 或某个命名导出
})
```

或：

```js
const posts = import.meta.glob('./posts/*', {
  as: 'raw', // 直接以 raw string 导入
})
```

Vite 玩得很花，真的很贴心。

------



### ⚡ 5. 和 import.meta.hot 一样，它是“宿主 API”

也就是说：

- **浏览器没有这个函数**
- **Node 没有这个**
- **构建工具负责解析并把它展开成真实代码**

它像“预处理器的扩展语法”，在构建时被转换成真实的 import 语句，浏览器最终是看不到这个函数的。

------



### 🍬 小甜点总结

如果 import.meta 是 JS 模块的“身份卡”，
 那 **import.meta.glob() 就像是“带目录扫描器的外挂工具”**。

- 帮你自动扫描文件
- 自动生成 import 映射
- 支持懒加载
- 支持 eager 加载
- 支持 raw 内容
- 支持按需导入命名导出
- Vite 热更新也能自动跟上

写起来很丝滑，像开挂一样。