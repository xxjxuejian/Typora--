

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

每个 `value` 是一个函数：调用这个函数时会动态加载对应组件（懒加载）