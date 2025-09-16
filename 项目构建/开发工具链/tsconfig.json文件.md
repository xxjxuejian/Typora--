

通过 `create-vue` 脚手架创建的 Vite 项目)

```ini
npm create vue@latest
```

会产生三个 `tsconfig.*.json` 文件。前提是`ts`项目



### 🏗️ 总览：这三份配置的关系

```ini
tsconfig.json
 ├─ tsconfig.app.json   （编译项目源代码 src/**）
 └─ tsconfig.node.json  （编译 Node 相关的配置文件）
```

这套 `tsconfig.*.json` 文件结构是现代 Vue (特别是通过 create-vue 脚手架创建的 Vite 项目) 中非常典型和优秀的实践。它的核心思想是**将项目代码根据其运行环境进行分离管理**，从而实现更精确、更安全的类型检查。



`tsconfig.json` 系列文件只是告诉 `tsc` **要编译哪些文件、如何编译**。

**TypeScript 编译**指的是：

> **把 `.ts` / `.tsx` / `.vue` 文件中的 TypeScript 代码（带类型注解的代码）转换成浏览器或 Node 可以直接运行的 JavaScript 代码，同时做类型检查。**

换句话说，它做两件事：

1. **类型检查（Type Checking）**
   - 看你代码中有没有类型错误，比如你把一个 `number` 当作 `string` 来用。
   - 这部分只在开发/构建时进行，不会出现在最终产物中。
2. **代码转换（Transpile）**
   - 把 TypeScript 语法（如 `interface`、`enum`、`type`、`?:`、`as` 等）去掉，转成标准 JavaScript。
   - 也可以顺便把较新的 ES 语法转换为旧版兼容语法（如果配置了 `target`）。



例子：

```ts
// TypeScript 源码
const add = (a: number, b: number): number => a + b;
```

编译后（JavaScript）：

```js
// JS 输出
const add = (a, b) => a + b;
```



`tsconfig.json`

```json
{
  "files": [],
  "references": [
    {
      "path": "./tsconfig.node.json"
    },
    {
      "path": "./tsconfig.app.json"
    }
  ]
}
```

**`tsconfig.json`**

- 是**根配置文件（Root Project Config）**，不直接参与编译。
- 用来使用 `references` 字段组织多个子项目配置（Project References）。
- 让 TypeScript 知道你有多个构建单元：一个是浏览器端（`app`），一个是 Node 端（`node`）。



`tsconfig.app.json`

```ts
{
  "extends": "@vue/tsconfig/tsconfig.dom.json",
  "include": ["env.d.ts", "src/**/*", "src/**/*.vue"],
  "exclude": ["src/**/__tests__/*"],
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",

    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

**`tsconfig.app.json`**

- 是**应用程序代码（浏览器端）**的编译配置。
- 会包含 `src/**`、`.vue` 文件等前端源码。
- `extends` 了 `@vue/tsconfig/tsconfig.dom.json`，这个预设里包含了针对浏览器环境（DOM 类型）的配置。



`extends: "@vue/tsconfig/tsconfig.dom.json"`：继承 Vue 官方提供的 DOM 环境配置。

`include`：指定要编译的前端源码文件。

`exclude`：排除测试目录。

`paths`：配置 `@/` 别名映射到 `src/` 目录。

`tsBuildInfoFile`：指定增量编译生成的缓存文件存放位置。







`tsconfig.node.json`

```json
{
  "extends": "@tsconfig/node22/tsconfig.json",
  "include": [
    "vite.config.*",
    "vitest.config.*",
    "cypress.config.*",
    "nightwatch.conf.*",
    "playwright.config.*",
    "eslint.config.*"
  ],
  "compilerOptions": {
    "noEmit": true,
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.node.tsbuildinfo",

    "module": "ESNext",
    "moduleResolution": "Bundler",
    "types": ["node"]
  }
}
```

是**Node 端（工具/配置文件）**的编译配置。

用来编译 `vite.config.ts`、`vitest.config.ts`、`eslint.config.ts` 等只在 Node 环境运行的脚本。

`extends` 了 `@tsconfig/node22/tsconfig.json`，带有 Node 22 的类型支持。

`types: ["node"]` 用来引入 Node.js 类型声明



`extends: "@tsconfig/node22/tsconfig.json"`：使用 Node.js 22 的类型环境。

`include`：只包含各种配置文件（在 Node 环境下运行）。

`noEmit: true`：告诉编译器只做类型检查，不生成 .js 文件（配置文件不需要输出）。

`types: ["node"]`：引入 Node 类型定义（如 `process`, `__dirname`）。

`moduleResolution: "Bundler"`：兼容 Vite 的打包方式。





### 🧩 tsconfig.json 的结构大纲

一个典型的 `tsconfig.json` 可以包含四类主要字段：

```json
{
  "extends": "...",
  "compilerOptions": { ... },
  "include": ["..."],
  "exclude": ["..."],
  "files": ["..."],
  "references": [{ "path": "..." }]
}
```

#### ⚙️ `compilerOptions` —— 编译选项（最核心）

这是 **最重要也是最常用** 的字段，用来控制编译器的行为。

| 常见子项              | 作用                                                      | 备注                                    |
| --------------------- | --------------------------------------------------------- | --------------------------------------- |
| `target`              | 编译输出的 JavaScript 版本（`ES5`, `ES2015`, `ES2020`等） | 决定转成多新/多旧的 JS                  |
| `module`              | 使用的模块系统（`CommonJS`, `ESNext`, `AMD`等）           | Node 多为 `CommonJS`，前端多为 `ESNext` |
| `moduleResolution`    | 模块解析策略（`Node`, `Bundler`, `Classic`）              | Vite 等 bundler 项目建议用 `Bundler`    |
| `jsx`                 | 如何处理 JSX（`preserve`, `react-jsx`等）                 | Vue 中通常不用                          |
| `baseUrl`             | 模块导入的基础路径                                        | 和 `paths` 搭配用                       |
| `paths`               | 路径别名映射                                              | 解决 `@/xxx` 这种路径别名               |
| `typeRoots` / `types` | 指定要包含的类型声明文件                                  | 常用于引入 `node`、`jest` 等类型        |
| `strict`              | 是否开启严格类型检查模式                                  | 推荐 `true`                             |
| `esModuleInterop`     | 允许默认导入 CommonJS 模块                                | 推荐 `true`                             |
| `allowJs`             | 是否允许编译 `.js` 文件                                   | 混合项目时有用                          |
| `noEmit`              | 只做类型检查，不输出 JS 文件                              | 一般在 Node 脚本配置中用                |
| `outDir` / `rootDir`  | 编译输出与输入根目录                                      | 输出 JS 文件时才用                      |
| `tsBuildInfoFile`     | 指定增量编译的缓存文件路径                                | 提升构建性能                            |



#### 📁 `include` / `exclude` / `files` —— 指定文件范围

| 字段      | 作用                   | 说明                         |
| --------- | ---------------------- | ---------------------------- |
| `include` | 指定要编译的文件或目录 | 支持通配符，如 `"src/**/*"`  |
| `exclude` | 排除不需要编译的文件   | 默认排除 `node_modules`      |
| `files`   | 精确指定文件列表       | 较少使用，只在要精确控制时用 |

> ⚠️ `include` 和 `files` 不要混用，否则很难维护。



#### 🧩 `extends` —— 继承其它 tsconfig

- 用来继承一个基础配置（官方或社区提供的预设）。
- 可以减少重复配置，常见例如：
  - `@vue/tsconfig/tsconfig.dom.json`
  - `@tsconfig/node22/tsconfig.json`

继承后可以覆盖其中的字段：

```json
{
  "extends": "@tsconfig/node22/tsconfig.json",
  "compilerOptions": {
    "noEmit": true
  }
}
```



#### 🏗️ `references` —— 项目引用（Project References）

用来把一个大项目拆成多个子项目**（模块化编译）**。

`tsc --build` 模式会用到。

每个 `references` 里的路径就是另一个 `tsconfig.json` 文件。

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

