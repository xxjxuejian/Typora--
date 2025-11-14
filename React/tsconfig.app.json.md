

`tsconfig.app.json`

```json
{
  "compilerOptions": {
    "baseUrl": ".", // ➕ 新增
    "paths": { "@/*": ["./src/*"] }, // ➕ 新增

    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "target": "ES2022",
    "useDefineForClassFields": true,
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "types": ["vite/client"],
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "erasableSyntaxOnly": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedSideEffectImports": true
  },
  "include": ["src"]
}
```



## 🌟 tsconfig.app.json 完整人话解读

### 📌 整体概念

`tsconfig.app.json` 是 **给 React 业务代码（src）用的 TS 配置**。
 它不管 Node、不管 Vite，只管你写 React 时类型系统怎么工作。

------

### 🧩 compilerOptions 内各项详解（逐条讲，简单易懂）

------

### **1. tsBuildInfoFile**

```json
"tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo"
```

➡️ **TS 按需增量编译的缓存文件位置**
 放在 node_modules/.tmp 是为了不污染项目结构。

它只影响性能，不影响行为。

------

### **2. target**

```json
"target": "ES2022"
```

➡️ **编译目标语法版本**
 告诉 TS 你希望输出的 JS 语法级别是多少。

React + Vite 里这个值对最终运行影响不大，因为实际转译由 Vite + esbuild 处理。

------

### **3. useDefineForClassFields**

```json
"useDefineForClassFields": true
```

➡️ **类字段是否使用 JS 新标准语法（define 语义）**
 现代标准，永远应该开。
 不开可能导致行为与 Babel 转出来不一致。

------

### **4. lib**

```json
"lib": ["ES2022", "DOM", "DOM.Iterable"]
```

➡️ **告诉 TS 你代码的“运行环境”是什么**

- ES2022 → 支持当年的 JS 内置对象（Promise、Array 方法等）
- DOM → 允许使用 document / window
- DOM.Iterable → 让 DOM 对象（NodeList）可被 for...of 循环

React 在浏览器运行，所以必须有 DOM。

------

### **5. module**

```json
"module": "ESNext"
```

➡️ **告诉 TS 你写的是 ES 模块**
 不让它自己处理打包方式（因为 bundler 会处理）。

------

### **6. types**

```json
"types": ["vite/client"]
```

➡️ **让 TS 识别 Vite 注入的全局类型**
 比如：

```json
import.meta.env.VITE_SOME_KEY
```

没有这个会报错。

------

### **7. skipLibCheck**

```json
"skipLibCheck": true
```

➡️ **跳过第三方库的类型检查**
 不然 node_modules 会给你 10 亿个错误。
 所有项目几乎都会开这个。

------

### 🧱 Bundler Mode：Vite 定制优化的核心部分

这些配置都是为了更好地配合 Vite 的“bundler 模式”。

------



### **8. moduleResolution**

```json
"moduleResolution": "bundler"
```

➡️ **告诉 TS：请像 Vite 那样解析模块，而不是按照 Node 规则解析**

这是 Vite 为 TS 定制的模式，会自动适配现代打包器行为。

（超级重要！不然 TS 和 Vite 的模块解析会不一致）

------



### **9. allowImportingTsExtensions**

```json
"allowImportingTsExtensions": true
```

➡️ **允许你 import .ts 文件时写后缀**

```json
import something from './utils.ts'
```

Vite 支持这种写法，所以 TS 也要支持。

------



### **10. verbatimModuleSyntax**

```json
"verbatimModuleSyntax": true
```

➡️ **不要偷偷改 import/export 语句的语义**
 让 TS 只做类型检查，不干扰模块系统。

这是 Vite 的推荐模式。

------

### **11. moduleDetection**

```json
"moduleDetection": "force"
```

➡️ **强制所有文件按 ES 模块解析**
 哪怕你没有写 import/export。

避免某些 .ts 文件被 TS 当成脚本文件处理。

------

### **12. noEmit**

```json
"noEmit": true
```

➡️ **TS 不生成 JS 输出**
 因为最终交给 Vite + esbuild 处理。

------

### 🧹 Linting 系列（全是帮助你查错误的）

这些都是类型检查层面的小帮手。

------

### **13. strict**

```json
"strict": true
```

➡️ TS 的“最严格模式”，开了以后更安全。

------



### **14. noUnusedLocals**

```json
"noUnusedLocals": true
```

➡️ 未使用的变量报错
 避免写垃圾代码。

------

### **15. noUnusedParameters**

```json
"noUnusedParameters": true,
```

➡️ 未使用的函数参数报错
 帮你发现冗余代码。

------

### **16. erasableSyntaxOnly**

```json
"erasableSyntaxOnly": true
```

➡️ **只允许可被 JS 安全擦除的 TS 语法**
 是 Vite + TS 的推荐配置。

比如这句会报错：

```
const a: number = 1; // OK
type A = number;    // OK
import type X from './xx' // OK

enum Something {}   // ❌（因为 enum 会生成 JS 代码）
```

它会禁止一些不是“纯类型”的 TS 特性（比如 enum）。

------



### **17. noFallthroughCasesInSwitch**

```json
"noFallthroughCasesInSwitch": true
```

➡️ switch/case 如果没有 break，会警告。

------



### **18. noUncheckedSideEffectImports**

```json
"noUncheckedSideEffectImports": true
```

➡️ 不允许你 import 某个东西却不使用
 除非显式声明是为了 side effect。

例如：

```ts
import "reset.css"; // 允许
import "./setup";   // 不允许，必须写注释标明用途
```

------

## 🧩 最后：include

```ts
"include": ["src"]
```

➡️ 很简单：
 **这个配置只影响 src/ 里的代码。**

不会扫描 node 环境文件（vite.config.ts）。

------



### 🎉 总结成一句大白话

`tsconfig.app.json` 是给你的 React 代码用的。
 里面所有配置的目的只有两个：

#### **1. 让 TypeScript 的行为和 Vite 完全一致（不打架）**

- bundler 模式
- 不输出 JS
- 原样模块语法
- TS 扮演“类型系统”的角色

#### **2. 提供严格、现代、浏览器友好的类型环境**

- ES2022 + DOM
- 严格模式
- 未使用变量报错
- switch fallthrough 检测
- side effect import 校验

整体就是：
 **快速、干净、现代、跟 Vite 完美契合。**