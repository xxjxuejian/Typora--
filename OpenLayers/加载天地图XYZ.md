天地图提供了经纬度坐标系（4326）投影的地图，也提供了墨卡托坐标系（3857）的投影。

加载地图是，就应该确定好选择哪一个。通常来说，使用3857的地图投影。

根据后缀`_c`或`_w`区分。

```js
img_w : 球面墨卡托投影的影像地图（EPSG:3857） img_c ：经纬度投影的影像地图（EPSG:4326）
```



```js
// "TDT-img": `http://t{0-7}.tianditu.gov.cn/DataServer?T=img_c&x={x}&y={y}&l={z}&tk=${TDT_KEY}`,
// "TDT-img-label": `http://t{0-7}.tianditu.gov.cn/DataServer?T=cia_c&x={x}&y={y}&l={z}&tk=${TDT_KEY}`,

"TDT-img": `http://t{0-7}.tianditu.gov.cn/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=${TDT_KEY}`,
"TDT-img-label": `http://t{0-7}.tianditu.gov.cn/DataServer?T=cia_w&x={x}&y={y}&l={z}&tk=${TDT_KEY}`,
```



```ts
const { center = [115.118459, 33.696687], zoom = 14 } = config;

    // 底图层
    // 如果我需要使用4326的地图，那么就要加载地图服务商提供的对应4326的地图切片，而不是设置source的projection
    const baseLayer = new TileLayer({
      source: new XYZ({
        url: mapUrls["TDT-img"],
        crossOrigin: "anonymous", // 解决跨域问题
      }),
      zIndex: 0, // 放在最底层
    });

    // 注记层 (文字标注，叠加在底图之上)
    const annotationLayer = new TileLayer({
      source: new XYZ({
        url: mapUrls["TDT-img-label"],
        crossOrigin: "anonymous",
      }),
      zIndex: 1, // 放在底图上面
    });

    const map = new Map({
      target: domRef.value,
      controls: defaultControls({ zoom: false }), // 关闭放大缩小控件
      layers: [
        // new TileLayer({
        //   source: new OSM(), // 这里演示使用 OSM，项目中可替换为天地图/高德等
        // }),
        baseLayer,
        annotationLayer,
      ],
      view: new View({
        // 不设置projection 默认坐标系是 EPSG:3857，经纬度不能直接显示
        // 调整坐标系为 EPSG:4326,可以直接使用经纬度坐标, 对应的天地图 也要使用经纬度投影的fromLonLat(center)
        center: fromLonLat(center),
        zoom: zoom,
        projection: "EPSG:3857",
      }),
    });
```

坐标系直接设置为`"EPSG:3857"`固定，如果有标记物坐标是4326的，就转换一下。