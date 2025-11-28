### markRaw()

将一个对象标记为不可被转为代理。返回该对象本身。

- **类型**

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

> 谨慎使用
>
> `markRaw()` 和类似 `shallowReactive()` 这样的浅层式 API 使你可以有选择地避开默认的深度响应/只读转换，并在状态关系谱中嵌入原始的、非代理的对象。它们可能出于各种各样的原因被使用：
>
> - 有些值不应该是响应式的，例如复杂的第三方类实例或 Vue 组件对象。
> - 当呈现带有不可变数据源的大型列表时，跳过代理转换可以提高性能。
>
> 这应该是一种进阶需求，因为只在根层能访问到原始值，所以如果把一个嵌套的、没有标记的原始对象设置成一个响应式对象，然后再次访问它，你获取到的是代理的版本。这可能会导致**对象身份风险**，即执行一个依赖于对象身份的操作，但却同时使用了同一对象的原始版本和代理版本：
>
> js
>
> ```ts
> const foo = markRaw({
>   nested: {}
> })
> 
> const bar = reactive({
>   // 尽管 `foo` 被标记为了原始对象，但 foo.nested 却没有
>   nested: foo.nested
> })
> 
> console.log(foo.nested === bar.nested) // false
> ```
>
> 识别风险一般是很罕见的。然而，要正确使用这些 API，同时安全地避免这样的风险，需要你对响应性系统的工作方式有充分的了解。





### shallowRef:

[`ref()`](https://cn.vuejs.org/api/reactivity-core.html#ref) 的浅层作用形式。

- **类型**

  ```ts
  function shallowRef<T>(value: T): ShallowRef<T>
  
  interface ShallowRef<T> {
    value: T
  }
  ```

- **详细信息**

  和 `ref()` 不同，浅层 ref 的内部值将会原样存储和暴露，并且不会被深层递归地转为响应式。只有对 `.value` 的访问是响应式的。

  `shallowRef()` 常常用于对大型数据结构的性能优化或是与外部的状态管理系统集成。

- **示例**

  ```js
  const state = shallowRef({ count: 1 })
  
  // 不会触发更改
  state.value.count = 2
  
  // 会触发更改
  state.value = { count: 2 }
  ```

