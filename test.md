## 三、XYZ 瓦片地址是怎么来的？（最直观版）

### 1️⃣ 后端 / GIS 拿到原始 `.tif`

```
raw.tif   （可能 5GB）
```

------

### 2️⃣ 做两件“必须的事”

#### （1）投影转换（通常转 3857）

```
gdalwarp -t_srs EPSG:3857 raw.tif web.tif
```

#### （2）建金字塔（多分辨率）

```
gdaladdo web.tif 2 4 8 16 32
```

这一步让它：

- 缩小时不拉原图
- 性能能活

------

### 3️⃣ 切成瓦片（XYZ）

```
gdal2tiles.py web.tif ./tiles
```

生成的东西长这样：

```
tiles/
 ├─ 0/0/0.png
 ├─ 1/0/0.png
 ├─ 1/1/0.png
 ├─ ...
```

------

### 4️⃣ 放到服务器上

比如用 Nginx：

```
https://example.com/tiles/{z}/{x}/{y}.png
```

🎉
 **这，就是 XYZ 瓦片地址的来源。**

👉 前端看到的只是 URL
 👉 背后是 `.tif` 被“拆成小图块”

------

## 四、那 WMTS 地址又是怎么来的？（更工程化）

XYZ 是“文件级别”的切片
 WMTS 是“服务级别”的切片

------

### 1️⃣ 后端安装 GeoServer

```
GeoServer
```

------

### 2️⃣ 把 `.tif` 加进 GeoServer

- 建 Workspace
- 建 DataStore
- 发布 Coverage（影像）

GeoServer 内部会：

- 识别坐标
- 利用金字塔
- 对接 GeoWebCache

------

### 3️⃣ GeoServer 自动帮你“切片”

但注意：

> **不是立刻生成所有 png**

而是：

- 前端请求 `/z/x/y`
- GeoServer 动态渲染
- 缓存下来

------

### 4️⃣ GeoServer 对外暴露 WMTS

```
https://xxx/geoserver/gwc/service/wmts
```

你在浏览器里请求的，其实是：

```
tilematrix = z
tilecol    = x
tilerow    = y
```

👉 **本质上和 XYZ 一模一样，只是协议更规范**

------

## 五、所以你可以这样理解（这个比喻很重要）

### `.tif` 是什么？

> **一整本超高清原稿**

### XYZ / WMTS 是什么？

> **按页、按缩放级别撕开的“复印件”**

前端：

- 只看复印件
- 永远不碰原稿