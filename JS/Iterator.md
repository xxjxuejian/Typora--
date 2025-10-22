### 一、迭代器（Iterator）是什么？

> **迭代器（Iterator）是一种用于遍历数据结构的机制。**

简单来说，它定义了一个**统一的遍历接口**，让不同的数据结构（数组、Set、Map、字符串等）都能**按相同的方式被循环访问**。

在 JavaScript 中：

- **迭代器对象（iterator）** 是一个具有 `.next()` 方法的对象；

- **每次调用 `next()`** 都会返回一个包含两个属性的对象：

  ```js
  { value: any, done: boolean }
  ```

  - `value`：当前的值；
  - `done`：是否遍历结束（`true` 表示没有更多值）。

------

### 二、基本示例

```js
const arr = [10, 20, 30];

// 获取该数组的迭代器对象
const iterator = arr[Symbol.iterator]();

console.log(iterator.next()); // { value: 10, done: false }
console.log(iterator.next()); // { value: 20, done: false }
console.log(iterator.next()); // { value: 30, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

👉 解释：

- `arr[Symbol.iterator]()` 返回一个**迭代器对象**；
- 每次调用 `.next()`，迭代器会返回下一个元素；
- 当遍历完所有元素时，`done` 变为 `true`。

------

### 三、可迭代对象（Iterable）

如果一个对象实现了 `Symbol.iterator` 方法，那么它就是**可迭代的（Iterable）**。

例如以下这些都是可迭代对象：

- 数组（`Array`）
- 字符串（`String`）
- `Map`
- `Set`
- `arguments` 对象
- `NodeList`（DOM 集合）

```js
console.log(typeof [][Symbol.iterator]); // function
console.log(typeof 'abc'[Symbol.iterator]); // function
console.log(typeof new Set()[Symbol.iterator]); // function
```

------

### 四、迭代器与 `for...of` 循环

`for...of` 循环会自动调用可迭代对象的 `Symbol.iterator` 方法，因此你不用手动 `.next()`：

```js
const arr = [1, 2, 3];

for (const item of arr) {
  console.log(item);
}
// 输出：1 2 3
```

等价于手动执行：

```js
const iterator = arr[Symbol.iterator]();
let result = iterator.next();
while (!result.done) {
  console.log(result.value);
  result = iterator.next();
}
```

------

### 五、自定义迭代器

你可以让自己的对象变成“可迭代”的：

```js
const myObj = {
  data: [100, 200, 300],
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        } else {
          return { done: true };
        }
      }
    };
  }
};

for (const value of myObj) {
  console.log(value);
}
// 输出：100 200 300
```

这样 `myObj` 就能被 `for...of` 或扩展运算符 `...` 使用了。

------

### 六、与生成器（Generator）的关系

**生成器（Generator）** 是创建迭代器的语法糖。

用生成器写上面的例子更简单：

```js
const myObj = {
  data: [100, 200, 300],
  *[Symbol.iterator]() {
    for (const item of this.data) {
      yield item;
    }
  }
};

for (const value of myObj) {
  console.log(value);
}
// 输出：100 200 300
```

👉 `yield` 会自动帮你生成 `value/done` 的结果对象。

------

### 七、常见用途

| 用途                    | 示例                                |
| ----------------------- | ----------------------------------- |
| 支持 `for...of`         | `for (let v of arr)`                |
| 扩展运算符              | `[...set]`、`[...map.keys()]`       |
| 解构赋值                | `const [a,b] = new Set([1,2]);`     |
| 与生成器配合            | `function* generator(){ yield 1; }` |
| 实现异步迭代（ES2018+） | `for await...of`                    |

------

### 八、总结对比表

| 概念                   | 含义                                |
| ---------------------- | ----------------------------------- |
| Iterable（可迭代对象） | 实现了 `Symbol.iterator` 方法的对象 |
| Iterator（迭代器）     | 拥有 `.next()` 方法的对象           |
| `.next()` 返回值       | `{ value, done }`                   |
| `for...of`             | 自动使用迭代器接口                  |
| Generator              | 用于简化迭代器实现的语法糖          |

------

✅ **一句话总结：**

> **Iterator 是一种遍历机制，可统一访问各种数据结构。**
>  可迭代对象实现 `Symbol.iterator` 方法，返回一个迭代器对象（具有 `next()` 方法）。
>  `for...of`、`...` 展开运算符、`yield` 等都依赖迭代器协议。