#### new Style(options)

参数结构：

```js
new Style({
    geometry:"",
    renderer:"",
    hitDetectionRenderer:"",
    image:"",
    fill:"",
    stroke:"",
    text:"",
    zIndex:1
})
```



```js
import Style from 'ol/style/Style.js';
import Icon from 'ol/style/Icon.js';
import Fill from 'ol/style/Fill.js';
import Stroke from 'ol/style/Stroke.js';
import Text from 'ol/style/Text.js';

const style = new Style({
  // 点样式（仅对 Point / MultiPoint 生效）
  image: new Icon({
    src: '/assets/marker.png',
    scale: 1,
    anchor: [0.5, 1],
  }),

  // 面填充（仅对 Polygon / MultiPolygon 生效）
  fill: new Fill({
    color: 'rgba(0, 150, 255, 0.4)',
  }),

  // 线条样式（LineString，Polygon 边界）
  stroke: new Stroke({
    color: '#ff6600',
    width: 2,
  }),

  // 文字（Point / Line / Polygon 都能加）
  text: new Text({
    text: 'Hello',
    font: '14px sans-serif',
    offsetY: -20,
    fill: new Fill({ color: '#000' }),
    stroke: new Stroke({ color: '#fff', width: 2 }),
  }),
});

export default style;
```





### 🎯 **OpenLayers 的 `new Style(options)` 各属性到底作用在哪？**

你可以把一个**Style**当成专门装饰 **某一个要素（Feature）** 的“化妆包”。

**一个 Vector Layer 通常会给自己的每个 feature 应用同一套 Style（除非 feature 自己有特定 style）。**

------

#### 🖼️ **1. `image` 作用在哪？**

👉 **作用在点（Point）或点状呈现的几何体（MultiPoint）上。**

- 它决定“点的外观”。
- 比如 `new Icon()`、`new CircleStyle()`、`new RegularShape()` 全部都属于 image。
- 所以 **image = 点的样式对象**。

也就是说：

- 如果你的 feature 的 geometry 是 `Point`，那 `image` 就是主角。
- 如果你的 geometry 不是点（LineString、Polygon），`image` 完全不会被用到。

------

#### 🎨 **2. `fill` 作用在哪？**

👉 **作用在 Polygon / MultiPolygon 上的内部区域。**

- 用于设置面填充颜色。
- 如果是线（LineString）或者点，fill 不起作用。

比如你画一个区域、行政区、多边形，fill 就是决定它里面那坨颜色。

------

#### ✏️ **3. `stroke` 作用在哪？**

👉 **作用在所有具备“边界”的几何图形上：**

- LineString / MultiLineString（线的颜色/宽度）
- Polygon / MultiPolygon 的外轮廓线

对于 Point，不用 stroke（点没有边界线）。

------

#### 📝 **4. `text` 作用在哪？**

👉 **作用在所有几何图形上（Point、Line、Polygon 都可）。**

- Text 是通用的。
- 对点：文字会显示在点旁边/上方。
- 对线：可沿线或线上某个点显示文字。
- 对面：出现在面中心附近。

------

#### ⭐ 最核心的“一句话真相”

```ini
image → 画点  
stroke → 画线/边界  
fill → 填充面  
text → 显示文字（谁都能加）
```

------

### 📌 **你可能关心的一点（你这种做地图标记的情况）**

你现在做的“添加地图标记”，本质上就是：

- 创建一个 Point feature
- 给它设置 Style
- 然后你的 Style 里最重要的是 **image**
- text 也可以一起用（比如显示名字）

而 fill / stroke 就算写了也不会起作用，因为你的 geometry 是点。

------





### 🎯 **image 到底什么时候会用？**

**只要你的几何类型是：**

- `Point`
- `MultiPoint`
- 或者是 `GeometryCollection` 中包含 Point

👉 **你就会用到 `image` 样式。**

因为 `image` 本质上就是“点的外观设计师”。

------

### 🌟 image 能放哪些类型？

你已经说了三个最常用的：

- `new Icon()`
- `new CircleStyle()`
- `new RegularShape()`

这三个就是实际开发里 99% 的场景。
 除此之外，还有一个不太常见但也属于 image 的：

- **`new OlStyleImage()` 的子类都算（但它是抽象类，一般不直接用）**

也就是说 **没有其他独立的“特殊 image 类型”了**。
 OpenLayers 官方提供的点样式，就是这三大类。

你已经掌握了全部“家族成员”。

------

### 🍉 那这三类点样式分别什么时候用？

下面这个超清晰对比表你看看就懂：

------

#### 🖼️ **1. `new Icon()` → 图片图标（最常用，做标记的标准方案）**

使用场景：

- 想显示 `PNG/SVG/JPG` 等外部图片
- 做地图标记（marker）
- 想自定义一张漂亮的小图标
- 支持旋转、缩放、anchor 定位等

常见例子：定位点、POI 图标、汽车图标等

```js
image: new Icon({
  src: 'xxx.png',
  anchor: [0.5, 1]
})
```

------

#### 🔴 **2. `new CircleStyle()` → 圆形点（简单粗暴）**

使用场景：

- 想画简单的圆形点，不需要导入图片
- 想通过设置 `stroke/fill` 让圆点有层次
- 用在热点、GPS 点、实时点位等场景

**优点：**

- 不用图片资源，渲染快
- 可随 zoom 动态调整半径
- 想做“跳动点”动画也很方便

```js
image: new CircleStyle({
  radius: 6,
  fill: new Fill({ color: 'red' })
})
```

------

#### 🔷 **3. `new RegularShape()` → 多边形图形（三角形、菱形、五角星…）**

使用场景：

- 想用几何图形来当点样式
- 想做方向性图标（三角形箭头）
- 想画星星、菱形、方形、小箭头等

常见用途例子：

- “我人在这里”的小箭头
- 朝向标（表示方向的点）
- 美观的小图形 POI

```js
image: new RegularShape({
  points: 5,
  radius: 10,
  fill: new Fill({ color: 'yellow' }),
  stroke: new Stroke({ color: 'black', width: 1 })
})
```

------

### 🌈 **那还有没有 image 的其他类型？**

🎉 **没有啦！只有这三个。**

OpenLayers 官方就是把“点样式”定义成这三大类，够用了。

你看到的 `StyleImage`、`StyleIcon`、`StyleCircle`、`StyleRegularShape` 都属于内部继承体系，并没有新的类型。

------

### 🎁 最后的记忆口诀（超好用）

```css
Icon → 图片点样式（标记最常用）
Circle → 圆点（简单、干净、轻量）
RegularShape → 几何形状点（三角形、星形、箭头）
```







