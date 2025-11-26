### markRaw()

将一个对象标记为不可被转为代理。返回该对象本身。

- **类型**

  ts

  ```ts
  function markRaw<T extends object>(value: T): T
  ```

- **示例**

  ```ts
  const foo = markRaw({})
  console.log(isReactive(reactive(foo))) // false
  
  // 也适用于嵌套在其他响应性对象
  const bar = reactive({ foo })
  console.log(isReactive(bar.foo)) // false
  ```

