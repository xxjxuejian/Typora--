### ⭐ 写法 1：对象式 props + PropType（最常见）

```ts
<script lang="ts" setup>
import { PropType } from "vue";

const props = defineProps({
  title: {
    type: String,
    required: true,
  },
  count: {
    type: Number as PropType<number>,
    default: 0,
  },
  list: {
    type: Array as PropType<string[]>,
    default: () => []
  },
  user: {
    type: Object as PropType<{ name: string; age: number }>,
    required: false
  }
});
</script>
```

🧠 **适用场景**：你想要 runtime 校验 + TS 类型双保险时，用它最稳。

------

### ⭐ 写法 2：基于 TS interface / type（最爽写法）

```ts
<script lang="ts" setup>
interface Props {
  title: string;
  count?: number;
  list?: string[];
  user?: { name: string; age: number };
}

const props = defineProps<Props>();
</script>
```

⚠️ 但注意：**没有 runtime 校验**，TypeScript-only 的类型安全。

🧠 **适用场景**：你们团队比较偏 TS，追求写法干净利落、类型更灵活。

------

### ⭐ 写法 3：使用 `withDefaults`（加默认值超舒服）

```ts
<script lang="ts" setup>
interface Props {
  title: string;
  count?: number;
}

const props = withDefaults(defineProps<Props>(), {
  count: 0
});
</script>
```

🧁 写起来贼甜：不用写 `default()`，纯类型层面解决默认值。

------

### 🔥 哪个最推荐？

如果你让我提个小意见：

- **组件库 / 通用组件用写法 1（带 runtime 校验）**
- **业务组件用写法 2 + withDefaults（最高效、最干净）**

