## grid

直角坐标系内绘图网格，单个 grid 内最多可以放置上下两个 X 轴，左右两个 Y 轴。可以在网格上绘制折线图，柱状图，散点图（气泡图）。

在 ECharts 2.x 里单个 echarts 实例中最多只能存在一个 grid 组件，在 ECharts 3 中可以存在任意个 grid 组件。



✅ **并不是每一个图都是绘制在 `grid` 中的。**

ECharts 中有**多种坐标系统**，不同类型的图表会用不同的“图表容器”，`grid` 只是其中之一。



✅ 哪些图是绘制在 `grid` 中的？

📦 `grid` 是**笛卡尔直角坐标系**的容器。

所有基于 **x/y 轴的二维图表**，都绘制在 `grid` 中，例如：

| 图表类型               | 是否使用 `grid`   |
| ---------------------- | ----------------- |
| 折线图 `line` ✅        | ✔ 是              |
| 柱状图 `bar` ✅         | ✔ 是              |
| 散点图 `scatter` ✅     | ✔ 是              |
| K 线图 `candlestick` ✅ | ✔ 是              |
| 面积图 ✅               | ✔ 是              |
| 双坐标轴图 ✅           | ✔ 是（两个 grid） |



❌ 哪些图不是绘制在 `grid` 中？

这些图不是基于 x/y 轴的，所以**不使用 `grid`**，而是用其他坐标系统或画布直接绘制：

| 图表类型                                           | 坐标系统          | 是否使用 `grid` |
| -------------------------------------------------- | ----------------- | --------------- |
| 饼图 `pie`                                         | 无坐标系统        | ❌ 否            |
| 仪表盘 `gauge`                                     | 无坐标系统        | ❌ 否            |
| 雷达图 `radar`                                     | `radar` 坐标系    | ❌ 否            |
| 地图 `map`                                         | `geo` 坐标系      | ❌ 否            |
| 热力图 `heatmap`（地图上）                         | `geo` / `bmap`    | ❌ 否            |
| 极坐标图 `polar` 系列                              | `polar` 坐标系    | ❌ 否            |
| 旭日图 `sunburst`、桑基图 `sankey`、树图 `treemap` | 无坐标系统        | ❌ 否            |
| 日历图 `calendar`                                  | `calendar` 坐标系 | ❌ 否            |

✅ 总结：

并不是所有图表都使用 `grid`，**只有基于笛卡尔坐标（x/y 轴）的图表才用 `grid`。**



### 默认的 `grid`

如果没有配置 `grid`，ECharts 会自动应用一个默认的 `grid` 配置。

ECharts 的**直角坐标系图表**（如柱状图、折线图）都是基于 `grid` 容器进行渲染的，所以即使你不显式写出来，**底层其实还是有一个默认的 `grid`。**

在你不手动配置 `grid` 的时候，ECharts 内部相当于自动使用了类似这样的配置：

```js
grid: {
  left: '10%',
  top: 60,
  right = '10%',
  bottom: 60,
  containLabel: false
}
```

`grid.left` `grid.right` `grid.top` `grid.bottom` `grid.width` `grid.height` 决定的是由坐标轴形成的矩形的尺寸和位置。

比如`grid.left`指的是grid 组件离容器左侧的距离。



`grid.containLabel`：grid 区域是否包含坐标轴的刻度标签。

设置容器样式，设置grid样式，对比，类似于`box-size`的`border-box`和`content-box`的区别。