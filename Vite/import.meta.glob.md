`import.meta.glob()`

Vite 支持使用特殊的 `import.meta.glob` 函数从文件系统导入多个模块：

const modules = import.meta.glob('./dir/*.js')