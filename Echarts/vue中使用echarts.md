### 一、安装 ECharts

```bash
npm install echarts
```



###  二、基础用法：单文件组件中使用（快速测试）

#### 模板部分

绘图前我们需要为 `ECharts` 准备一个定义了高宽的 `DOM 容器。`

```vue
<template>
  <div ref="chartRef" style="width: 100%; height: 400px"></div>
</template>
```

说明：

- `ref="chartRef"` 是绑定图表 DOM 容器的关键
- 一定要给容器设置宽高（否则图表不会显示）

#### 引入 echarts 和初始化

```vue
<script setup>
const chartRef = ref(null)

// 定义配置项
const option = {
    title: {
      text: '水果销量柱状图'
    },
    tooltip: {},
    xAxis: {
      data: ['苹果', '香蕉', '橙子']
    },
    yAxis: {},
    series: [
      {
        name: '销量',
        type: 'bar',
        data: [5, 20, 36]
      }
    ]
}

onMounted(() => {
  const myChart = echarts.init(chartRef.value)
  myChart.setOption(option)
  // 让图表随窗口大小变化而自适应
  window.addEventListener('resize', () => {
    myChart.resize()
  })
})
</script>
```

#### 等 DOM 渲染完成之后再初始化

在 `setup()` 执行时，**DOM 元素还没有挂载到页面上**，所以 `chartRef.value` 还是 `null`，此时你如果调用 `echarts.init(chartRef.value)` 会报错：

Vue 3 提供的 `onMounted()` 生命周期钩子，保证了组件 DOM 已经渲染完成。

```ts
onMounted(() => {
  const chart = echarts.init(chartRef.value)
  chart.setOption(option)
})
```