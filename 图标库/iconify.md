### 🎨 那 Iconify 究竟是什么？

你可以把 Iconify 理解成一个**图标聚合平台 + 统一规范 + 工具生态**。

它提供：

#### ① 超大的图标集合（20w+ 图标）

Iconify 把整个世界上主流的图标库都统一管理了，比如：

- Icon Park
- Material Icons
- Heroicons
- Font Awesome（大部分）
- Tabler Icons
- Bootstrap Icons
- Carbon
- Octicons
- Emoji icons…

这些全部都能用，统一格式统一接口。 

**Iconify 就是一个巨型图标合集大平台（库的库）。**

------



#### ② 统一的图标命名规范

例如：

```less
icon-park:add
mdi:home
heroicons-outline:cloud
fa-solid:user
carbon:search
```

不管你来自哪个图标库，Iconify 都给你一个统一的格式来调用。

------



#### ③ 图标 JSON 格式（底层数据结构）

每个图标本质上都是一个 JSON 格式的 SVG 描述。

这让：

- tree-shaking 超容易
- 可以动态修改颜色、尺寸
- 客户端渲染、服务端渲染都能工作
- 可以组合、缓存、注入

------



#### ④ 各种工具 & 插件（生态）

比如：

- `unplugin-icons`（自动按需加载）
- `@iconify/tools`（做图标转换）
- `@iconify/json`（图标数据库）
- 在线浏览器（icon-sets.iconify.design）

Iconify 不是一个库，而是一整套“图标生态系统”。

***



### 🧩  @iconify/vue 是什么？

它只是一个——
 👉 **官方提供的，用于在 Vue 中显示 Iconify 图标的组件库。**

类似这样：

```vue
<script setup>
import { Icon } from '@iconify/vue'
</script>

<template>
  <Icon icon="mdi:home" />
</template>
```

它的作用只有一个：

**把 Iconify 的 JSON 图标 → 渲染成 Vue 的 SVG。**

**不负责：**

- 图标来源
- 图标下载
- 图标集合
- 解析命名空间

这些都属于 Iconify 核心生态做的。

你甚至可以不用 `@iconify/vue`，而用：

- React 的 `@iconify/react`
- Svelte 的 `@iconify/svelte`
- Vue 的 `unplugin-icons`（自动组件方式）

都是同一个 Iconify 图标数据库。



### 🧠 比喻，让你完全掌握：

> **Iconify = 一个拥有所有图标的大图书馆（20w+本书）。**
>  **@iconify/vue = 图书管理员，负责帮你把书搬到 Vue 页面上展示。**

如果你不用图书馆管理员，也可以自己搬（例如用 unplugin-icons 自动直接搬）。

***



### 🎁 它们的关系：

**Iconify = 图标数据 + 标准 + 工具链。**
 **@iconify/vue = 一个展示这些图标的 Vue 组件实现。**

------



### Vue3+Vite项目使用iconify

安装 `@iconify/vue`

```ini
npm install --save-dev @iconify/vue 
yarn add --dev @iconify/vue
```

它是 Iconify 的渲染器（Renderer）。

#### 组件中使用

```ts
import { Icon } from "@iconify/vue";
```

在`template` 中使用Icon组件，把图标名称作为`icon`参数传递

```vue
<Icon icon="mdi-light:home" />
```

其中`mdi-light`是使用的图标库前缀，`home`则是具体的图标名称。



### Iconify前缀对应表

Iconify 里面的**每个图标库**都有一个固定的“前缀”（namespace）。

比如：

| 图标库（来源）        | Iconify 前缀        | 示例图标                  |
| --------------------- | ------------------- | ------------------------- |
| Material Design Icons | `mdi`               | `mdi:home`                |
| Material Symbols      | `material-symbols`  | `material-symbols:search` |
| IconPark              | `icon-park`         | `icon-park:add`           |
| Heroicons Outline     | `heroicons-outline` | `heroicons-outline:home`  |
| Heroicons Solid       | `heroicons-solid`   | `heroicons-solid:heart`   |
| Font Awesome Solid    | `fa-solid`          | `fa-solid:user`           |
| Font Awesome Brands   | `fa-brands`         | `fa-brands:github`        |
| Bootstrap Icons       | `bi`                | `bi:alarm`                |
| Tabler Icons          | `tabler`            | `tabler:bell`             |
| Carbon Icons          | `carbon`            | `carbon:search`           |





### 使用方式

#### **① 在线请求 Iconify API（开箱即用）**

什么都不额外装！

只需要安装

```ini
npm i @iconify/vue
```

写这个：

```vue
<script setup>
import { Icon } from "@iconify/vue";
</script>

<template>
    <Icon icon="mdi:home" width="24" height="24" />
    <Icon icon="icon-park:add" />
    <Icon icon="fa-solid:user" />
</template>
```

就能直接显示，因为 Iconify 会从它的 CDN 加载图标小 JSON。

优点：

- 无需安装大量图标包
- 随用随取，懒加载
- 支持图标库超级多（200+）

缺点：

- 需要网络
- 图标是运行时加载（虽然体积很小）

Iconify CDN：`https://api.iconify.design/mdi-light/home.json`

Vite 开发模式下更快，几乎秒显示

生产环境首次加载也只有几百 bytes

👉 因此需要网络；离线会报错（除非你做了 cache）

------



#### **②（更专业）本地导入图标 JSON 来实现 tree-shaking**

如果你不想 API 加载，你可以：

```ini
npm i @iconify-json/mdi @iconify-json/icon-park @iconify-json/fa-solid
```

然后：

```js
import home from '@iconify-json/mdi/icons/home.json'
import add from '@iconify-json/icon-park/icons/add.json'
import user from '@iconify-json/fa-solid/icons/user.json'
```

然后配合 `unplugin-icons` 或 `iconify.runtime.esm.js` 实现本地渲染。

优点：

- **真正 tree-shaking**
- 零网络依赖
- 图标只包含你用到的

缺点：

- 配置稍复杂
- 想用哪个图标库，就得安装对应 JSON 包



**③按需自动导入**

使用 [unplugin-icons](https://github.com/antfu/unplugin-icons) 和 [unplugin-auto-import](https://github.com/antfu/unplugin-auto-import) 从 iconify 中自动导入任何图标集。



整理



### 🧠 **总结一句话（精华）**

> **安装了 `@iconify/vue` 后，你可以使用 Iconify 所收录的全部图标库，但加载方式取决于你是否配置 API 方式或安装对应 JSON 包。**

所以你写：

```html
<Icon icon="mdi:home" />
```

这个 `mdi`（Material Design Icons）
 不是你安装的 npm 包，而是 Iconify 图标集合里的一部分。