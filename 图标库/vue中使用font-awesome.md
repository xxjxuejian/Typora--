### 🧩 方式一：直接用 CDN（简单粗暴）

适合：小项目、临时代码、懒人模式

#### 在 index.html 引入

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
```

#### 组件里直接用 class

```html
<i class="fa-solid fa-user"></i>
<i class="fa-brands fa-github"></i>
```

💡 这种方式**不能 tree-shake**，会加载所有图标，但是最快捷。



```

<Icon icon="mdi-light:home" />
```

