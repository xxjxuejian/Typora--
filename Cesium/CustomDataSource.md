`Cesium.CustomDataSource` 是 Cesium 中一个非常常用、也非常实用的**实体管理容器**。
 它可以理解为一个**自定义的图层（Layer）**，用来组织和管理一组 `Entity`，而不是直接把实体全部丢到全局的 `viewer.entities` 里。



## 介绍

### 🧩 一、作用概述

在 Cesium 中：

- 所有的实体 (`Entity`) 必须属于某个 **EntityCollection**；
- `viewer.entities` 就是默认的全局 `EntityCollection`；
- `CustomDataSource` 提供了一个 **独立的数据源（独立的实体集合）**，它内部也有自己的 `entities`；
- 你可以把它看作是**一个可命名、可独立控制显隐/移除的实体图层**。

👉 用它可以方便地：

- 组织不同类型的实体（比如“隧道点位层”、“车辆层”、“警报层”等）；
- 一次性添加或移除一整组实体；
- 单独控制某一类实体的显示隐藏；
- 对实体进行逻辑分组管理，避免 `viewer.entities` 过于混乱。

------



### 🧱 二、基本用法

#### ✅ 1. 创建一个 CustomDataSource

```js
const tunnelLayer = new Cesium.CustomDataSource("tunnel-layer");
```

#### ✅ 2. 添加实体到该数据源

```js
tunnelLayer.entities.add({
  id: "tunnel-1",
  name: "A号隧道",
  position: Cesium.Cartesian3.fromDegrees(116.39, 39.9, 50),
  point: {
    pixelSize: 10,
    color: Cesium.Color.YELLOW,
  },
});
```

#### ✅ 3. 把数据源添加到 Viewer

```js
viewer.dataSources.add(tunnelLayer);
```

现在，`tunnelLayer` 里的所有实体都会显示在地图上。

------

### 🪄 三、常用操作

#### 1️⃣ 获取某个数据源

```js
const layer = viewer.dataSources.getByName("tunnel-layer")[0];
```

#### 2️⃣ 移除整个数据源

```js
viewer.dataSources.remove(layer);
```

> 移除后图层和其中的实体都不会显示，但仍然可以保留在内存中（可选第二个参数）。

```js
viewer.dataSources.remove(layer, false); // 不销毁对象
```

#### 3️⃣ 显示/隐藏图层

```js
layer.show = false; // 隐藏整层
layer.show = true;  // 显示整层
```

#### 4️⃣ 清空当前数据源的实体

```js
layer.entities.removeAll();
```

#### 5️⃣ 查找或更新某个实体

```js
const e = layer.entities.getById("tunnel-1");
if (e) e.point.color = Cesium.Color.RED;
```

------

### ⚙️ 四、典型应用场景

| 场景                          | 用法                                                        |
| ----------------------------- | ----------------------------------------------------------- |
| 分组管理不同业务数据          | 建立多个 `CustomDataSource`，如道路监控层、隧道层、传感器层 |
| 动态更新                      | 更新时清空 `layer.entities` 再重新添加                      |
| 按需显示隐藏图层              | 用 UI 按钮控制 `layer.show`                                 |
| 加载 GeoJSON 或其他自定义数据 | 解析后逐个 `entities.add()`                                 |
| 临时缓存数据                  | 不添加到 `viewer` 就不会显示                                |

------

### 🧠 五、与其他类的区别

| 类名                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| `EntityCollection`  | 基础的实体集合（CustomDataSource 内部就是它）                |
| `viewer.entities`   | 全局默认集合，所有未分组的实体都放这里                       |
| `CustomDataSource`  | **可命名、可控显隐的自定义实体集合**                         |
| `GeoJsonDataSource` | 专门解析 GeoJSON 格式的 DataSource，继承自 `CustomDataSource` |

------

### 🌈 六、示例：多个图层管理

```js
const tunnelLayer = new Cesium.CustomDataSource("tunnels");
const carLayer = new Cesium.CustomDataSource("cars");

// 添加到 viewer
viewer.dataSources.add(tunnelLayer);
viewer.dataSources.add(carLayer);

// 添加实体
tunnelLayer.entities.add({ id: "t1", position: ..., point: {...} });
carLayer.entities.add({ id: "c1", position: ..., billboard: {...} });

// 控制显隐
tunnelLayer.show = true;
carLayer.show = false;
```

------

### ✅ 总结

| 特性     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 类型     | `Cesium.CustomDataSource`                                    |
| 功能     | 管理一组实体，逻辑上作为独立图层                             |
| 优点     | 可独立控制显示、隐藏、清空、移除                             |
| 使用场景 | 分层管理不同业务数据、动态更新实体、简化 viewer.entities 管理 |





## 图层Layer

简而言之：

> 🧩 **Cesium 中没有传统意义上的“图层（Layer）”概念。**
>  Cesium 的“Layer”思想是由多个不同的体系来**分别模拟实现的**。
>  `CustomDataSource` 就是其中用于**“实体层（Entity Layer）”管理的替代方案**。

下面我帮你系统地解释清楚 👇👇👇

------

### 🗺️ 一、传统 GIS 与 Cesium 的差异

在传统的 2D GIS（如 ArcGIS、OpenLayers、Mapbox）中，**Layer 是核心概念**：

- 每个 Layer 代表一个可显示/隐藏/独立管理的数据层；
- Layer 可以是矢量图层（点、线、面）或栅格图层（影像底图）。

但是在 **Cesium（3D 地球）** 中，内部架构更底层和灵活，**并没有通用的 “Layer” 类**。
 Cesium 把不同类型的可视化内容拆成几个体系：

------

### 🧱 二、Cesium 中类似 “Layer” 的几个体系

| 类型                              | 代表类                                 | 功能                              | 是否“Layer”          |
| --------------------------------- | -------------------------------------- | --------------------------------- | -------------------- |
| **底图图层（Imagery Layer）**     | `ImageryLayer`                         | 管理影像底图，如卫星图、街道图    | ✅ 是真正意义的图层   |
| **地形图层（Terrain Provider）**  | `CesiumTerrainProvider`                | 控制地形起伏数据来源              | ✅ 可以看作是地形层   |
| **实体层（Entity Layer）**        | `CustomDataSource`                     | 管理一组 `Entity`（点线面模型等） | ⚙️ 模拟实现的图层     |
| **原始几何层（Primitive Layer）** | `PrimitiveCollection`                  | 底层几何集合（性能更高）          | ⚙️ 底层版“图层”       |
| **数据源层（DataSource Layer）**  | `GeoJsonDataSource` / `CzmlDataSource` | 加载外部数据（GeoJSON、CZML 等）  | ⚙️ 高层封装的图层概念 |

------

### 🌐 三、`CustomDataSource` 扮演的角色

`CustomDataSource` 并不是 Cesium 内部的“正式 Layer 类”，
 但它承担了 **“逻辑图层”** 的功能：

- 它是一个可命名的数据容器；
- 内部有一个独立的 `EntityCollection`；
- 可以整体添加、移除、显示、隐藏；
- 支持批量操作实体；
- 可以通过 `viewer.dataSources` 管理多个这样的“图层”。

📘 换句话说：

> `CustomDataSource` 是 Cesium 在 Entity 系统里用来**模拟图层概念的机制**。

------

### 🧠 四、实际项目中的理解方式

在工程实践中，我们通常这么理解：

| 想法                       | Cesium 实现方式                                 |
| -------------------------- | ----------------------------------------------- |
| “地图底图层”               | 用 `ImageryLayer` 控制（添加或移除影像服务）    |
| “地形层”                   | 用 `TerrainProvider` 控制                       |
| “业务数据图层（点/线/面）” | 用 `CustomDataSource`（或 `GeoJsonDataSource`） |
| “底层高性能绘制层”         | 用 `PrimitiveCollection` 直接操作 Primitive     |

举个例子：

```js
// 底图层：卫星影像
viewer.imageryLayers.addImageryProvider(
  new Cesium.UrlTemplateImageryProvider({ url: "https://tiles.mapbox.com/{z}/{x}/{y}.png" })
);

// 业务层：隧道实体
const tunnelLayer = new Cesium.CustomDataSource("tunnel-layer");
tunnelLayer.entities.add({ id: "t1", position: ..., point: {...} });
viewer.dataSources.add(tunnelLayer);

// 控制显隐
tunnelLayer.show = false; // 相当于隐藏该“图层”
```

------

### ⚙️ 五、如果你想严格“分层管理”Cesium数据

可以按层分类：

| 图层类型    | 管理对象                | 示例类                               |
| ----------- | ----------------------- | ------------------------------------ |
| 影像图层    | viewer.imageryLayers    | ImageryLayer                         |
| 地形图层    | viewer.terrainProvider  | CesiumTerrainProvider                |
| 实体图层    | viewer.dataSources      | CustomDataSource / GeoJsonDataSource |
| Primitive层 | viewer.scene.primitives | PrimitiveCollection                  |

------

### ✅ 六、总结一句话

> Cesium 没有通用的 `Layer` 类。
>  它把“图层”的概念拆成多个体系：
>
> - 影像/地形 → 真正的 Layer
> - 实体/业务数据 → 用 `CustomDataSource` 模拟 Layer
> - 底层几何 → 用 `PrimitiveCollection` 实现 Layer