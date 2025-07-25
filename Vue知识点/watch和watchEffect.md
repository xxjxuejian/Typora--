在 Vue 中，`watch` 和 `watchEffect` 都是用于**响应式监听**的工具，但它们的使用场景、机制和行为有明显区别。

### ✅ 1. **监听对象的方式不同**

`watch`：

```ts
// 侦听单个来源
function watch<T>(
  source: WatchSource<T>,
  callback: WatchCallback<T>,
  options?: WatchOptions
): WatchHandle

// 侦听多个来源
function watch<T>(
  sources: WatchSource<T>[],
  callback: WatchCallback<T[]>,
  options?: WatchOptions
): WatchHandle
```

- 明确指定要监听的**响应式数据或 getter 函数**。
- 更适合监听 **特定数据变化**，如：一个 `ref`、一个 `reactive` 的某个字段、或计算属性等。

`watch()` 默认是懒侦听的，即仅在侦听源发生变化时才执行回调函数。

第一个参数是侦听器的**源**。这个来源可以是以下几种：

- 一个函数，返回一个值
- 一个 ref
- 一个响应式对象
- ...或是由以上类型的值组成的数组