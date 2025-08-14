

### ES 模块加载的真实流程（简化版）

#### （1）静态解析阶段

JS 引擎会从入口模块（比如 `main.js`）开始扫描源码

**遇到 `import` / `export`**：

- 提取出依赖路径（例如 `./router`）
- 加入模块依赖图
- 标记这个依赖模块需要加载

这里的“静态解析”是指**不执行任何顶层代码**，只是读取依赖信息和导出符号

这里的模块依赖图可能类似这种：

```bash
main.js
 ├── ./assets/main.css
 ├── vue
 ├── pinia
 ├── ./App.vue
 └── ./router/index.js
      ├── vue-router
      ├── ../views/HomeView.vue
      └── ../stores/counter.js
           ├── vue
           └── pinia
```



#### （2）生成 AST（Abstract Syntax Tree）

- 对于每个模块，引擎会**完整解析源码**生成 AST
- AST 里会包含 `ImportDeclaration`、`ExportNamedDeclaration` 等节点
- 通过这些节点，引擎能持续更新模块依赖图
- 所以依赖图不是一次性生成的，而是**递归构建**的



##### AST

抽象语法树（AST, Abstract Syntax Tree）就是**用树状结构来表示源码语法结构的一种数据结构**。

```js
const a = 5 + 3;

// AST
Program
 └── VariableDeclaration (const)
      └── VariableDeclarator
           ├── Identifier (a)
           └── BinaryExpression (+)
                ├── Literal (5)
                └── Literal (3)
```

这里：

- `Program` 是整个程序
- `VariableDeclaration` 表示声明变量
- `BinaryExpression` 表示二元运算（这里是 `+`）
- `Literal` 表示字面量值



> 是先进行静态解析，生成模块依赖图，然后再生成 AST 吗？

不是完全分开的两个阶段，而是：

- **扫描 + AST 生成** 是递归、交织进行的
- 在扫描模块源码时，会同时建立依赖关系，并为模块生成 AST
- 所以依赖图是在解析所有模块 AST 的过程中逐步构建的



> 生成模块依赖图以后会按照依赖图执行代码吗？

是的，**执行顺序由依赖图决定**：

- 按照依赖关系，先执行**所有依赖模块**的顶层代码
- 再回到当前模块执行
- 每个模块的顶层代码只执行一次（有缓存）





#### （3）构建模块记录（Module Record）

- 对每个模块，保存：
  - 它的 AST
  - 它的导出（export bindings）
  - 它的依赖（import 绑定）
- 这样就有了一个完整的模块依赖图 + 每个模块的语法树



#### （4）执行阶段（Module Evaluation）

- 从入口模块开始，按照依赖图的顺序**先执行依赖、再执行当前模块**
- 依赖是**深度优先**执行的（即先把所有依赖执行完再回到调用方）
- 这一步会真正跑你的顶层代码（`console.log`、函数调用、`useCounterStore()` 等）



##### 举例

依赖关系（忽略 CSS 和动态 import）：

```bash
main.js
 └── router/index.js
      └── stores/counter.js
```

执行顺序：

1. `stores/counter.js`（叶子节点，没依赖其他模块）
2. `router/index.js`（依赖 `stores/counter.js`，叶子执行完后执行）
3. `main.js`（依赖 `router/index.js`，叶子执行完后执行）

所以当 `router/index.js` 顶层调用 `useCounterStore()` 时：

- `stores/counter.js` 已经执行完（模块本身没有问题）
- 但是 Pinia **还没在 main.js 中初始化**，所以 `getActivePinia()` 报错