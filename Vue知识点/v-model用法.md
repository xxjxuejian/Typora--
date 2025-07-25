#### v-model用法

这不也是一个v-model在组件上的应用吗

```vue
 <el-radio-group size="small" v-model="recentDaysRange">
          <el-radio-button :label="7">近7天</el-radio-button>
          <el-radio-button :label="30">近30天</el-radio-button>
 </el-radio-group>
```

#### 1. 什么时候用到了v-model

在通知管理页面，新增通知时，需要使用富文本编辑器，就需要让表单内容和编辑器的内容双向绑定。

```vue
<WangEditor v-model="formData.content"></WangEditor>
```

很明显的是，`WangEditor`是一个组件，`v-model`现在是在组件上使用实现双向绑定。先看一下在普通的表单元素上的用法。

#### 2. v-model在表单元素上使用

在表单元素上使用时，可以将响应式变量和表单元素的值进行双向绑定

```vue
<template>
  <input v-model="username" />
</template>

<script setup>
import { ref } from 'vue'

const username = ref('')
</script>
```

用户在输入框输入内容后，`username` 的值会自动更新；反过来如果你修改 `username`，输入框的值也会跟着变。

##### 2.1 原理解析

###### (1) v-model 实际是语法糖

```vue
<input v-model="username" />
```

等价于

```vue
<input
  :value="username"
  @input="username = $event.target.value"
/>
```

[`value`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Elements/input#value) 属性是input，把`username`变量绑定到了`value`属性上

也就是说：

- `:value="xxx"` 是从数据更新到视图
- `@input` 是从视图更新到数据

###### (2) 不同表单元素的绑定

在不同的表单元素上，`v-model` 绑定的默认属性（prop）和事件（event）**是不完全一样**的。虽然很多情况下是 `value`，但也有例外。

以`input`元素为例，是浏览器原生DOM，`value`属性或者`checked`等属性是原生属性，`vue`会自动绑定到这些原生属性上，并不需要我们手动的定义`prop`。

```vue
 <!-- input -->
<input v-model="msg" />

<input
  :value="msg"
  @input="msg = $event.target.value"
/>
```

```vue
<!-- Checkbox -->
<input type="checkbox" v-model="isChecked" />

<input
  type="checkbox"
  :checked="isChecked"
  @change="isChecked = $event.target.checked"
/>
```

等等...

#### 3. v-model在组件上使用

​	回到上面，在自定义组件上，`v-model` 默认绑定 `modelValue` 属性，并监听 `update:modelValue` 事件。

```vue
<!-- 父组件中 -->
<WangEditor v-model="formData.content"></WangEditor>

<!-- 等价于 -->
<WangEditor
  :modelValue="formData.content"
  @update:modelValue="val => formData.content = val"
/>
```

父组件绑定 `v-model`，当`formData.content`变化时，自动同步到`WangEditor`组件内，在组件内，修改富文本内容时，也会更新父组件的`formData.content`

##### WangEditor.vue

`WangEditor`组件是自定义组件，并没有`modelValue`属性，所以在`WangEditor`组件中，我们要定义`prop.modelValue`。

```vue
<template>
  <div>
    <Toolbar :editor="editorRef" mode="default" style="border-bottom: 1px solid #ccc;" />
    <Editor
      v-model="valueProxy"
      :defaultConfig="editorConfig"
      mode="default"
      style="height: 300px; overflow-y: hidden;"
      @onCreated="handleCreated"
    />
  </div>
</template>

<script setup>
import { ref, watch, computed, onBeforeUnmount } from 'vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'

// 接收外部 v-model 的绑定
const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
})
const emit = defineEmits(['update:modelValue'])

const editorRef = ref(null)
const editorConfig = {
  placeholder: '请输入内容...',
  // 更多配置
}

// computed 方式做双向绑定桥接
const valueProxy = computed({
  get() {
    return props.modelValue
  },
  set(val) {
    emit('update:modelValue', val)
  }
})

// 创建和销毁 editor
function handleCreated(editor) {
  editorRef.value = editor
}

onBeforeUnmount(() => {
  if (editorRef.value) {
    editorRef.value.destroy()
  }
})
</script>

<style scoped>
/* 可自定义样式 */
</style>
```

模板中的`Editor`需要使用`v-mode`l绑定一个响应式变量，为什么不直接`v-model=props.modelValue`，是因为`props.modelValue` 是 **只读的**，Vue 会在开发模式下直接报错：

> ❌ "Cannot assign to read only property 'modelValue' of object '#<Object>'"

所以通过一个中间变量`valueProxy`进行读写

```vue
<Editor
  v-model="valueProxy"
  // ......
/>
<script setup>

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
})
// computed 方式做双向绑定桥接
const valueProxy = computed({
  get() {
    return props.modelValue
  },
  set(val) {
   	// 触发set是，就修改了valueProxy的值，但是要同步更新父组件的formData.content值，就要发送事件过去，把新值传递出去
    emit('update:modelValue', val)
  }
})

</script>
```

注意到`Editor`中也使用`v-model`，则也是类似的逻辑。

```vue
<Editor v-model="valueProxy"/>
```

一旦 `<Editor />` 内部编辑内容时，就会触发`@update:modelValue`，此时就会更新`valueProxy`值，`valueProxy`是一个computed，修改的同时触发`set()`，向父组件`emit('update:modelValue', val)`

在父组件中：

```vue
<WangEditor v-model="formData.content"></WangEditor>

<!-- 等价于 -->
<WangEditor
  :modelValue="formData.content"
  @update:modelValue="val => formData.content = val"
/>
```

监听到`update:modelValue`时，就更新了`formData.content`



执行链条：

```sql
[父组件]
<WangEditor v-model="formData.content"></WangEditor>
       ↓ 等价于
<WangEditor 
	:modelValue="formData.content" 
	@update:modelValue="formData.content = val">
</WangEditor>
       ↓
当formData.content 改变
	   ↓ 向下传递
[封装的 <WangEditor> 组件内部]
valueProxy = computed(() => props.modelValue / emit update)
	   ↓ valueProxy 改变
get:valueProxy = prop.modelValue 
       ↓ 
[三方库里的 <Editor> 组件（wangeditor 提供）]
<Editor v-model="valueProxy"> 
	   ↓ 等价于
<Editor :modelValue="valueProxy" @update:modelValue="valueProxy = val">
       ↓ 向下传递
Editor 编辑器内容变了，文本区域的值改变
```



```sql
[父组件]   
<WangEditor v-model="formData.content"></WangEditor>
捕获update:modelValue事件，执行formData.content = newContent
	   ↑ 
[<WangEditor> 组件]
<Editor v-model="valueProxy"> 
→ 触发 valueProxy 的 setter : emit('update:modelValue', newContent)（往上发给父组件）
→ 执行 valueProxy = newContent
	   ↑ 执行
<Editor :modelValue="valueProxy" @update:modelValue="valueProxy = newContent">
       ↑ 捕获update:modelValue事件
[<Editor> 组件]
Editor 编辑器内容变了
→ emit('update:modelValue', newContent)
```



#### 结合@vueuse/core

使用`useVModel`可以简化这个双向绑定的过程

原先的代码：

```js
const prop = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
})
const emit = defineEmits(['update:modelValue'])
const valueProxy = computed({
  get() {
    return prop.modelValue
  },
  set(newValue) {
    emit('update:modelValue', newValue)
  }
})
```

使用`useVModel`：

```js
const prop = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
})
const emit = defineEmits(['update:modelValue'])
const valueProxy = useVModel(props, 'modelValue', emit)
```

也类似于语法糖
