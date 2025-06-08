<template>
  <div class="mapdiv"></div>
</template>

<script>
// import WebMap from "@arcgis/core/WebMap";
// import MapView from "@arcgis/core/views/MapView";
// import Bookmarks from "@arcgis/core/widgets/Bookmarks";
// import Expand from "@arcgis/core/widgets/Expand"; 
//MAPS SDK FOR JAVASCRIPT
import Map from "@arcgis/core/Map";
import MapView from "@arcgis/core/views/MapView";
import MapImageLayer from "@arcgis/core/layers/MapImageLayer";
import BasemapToggle from "@arcgis/core/widgets/BasemapToggle";

export default {
  name: 'App',
  async mounted() {
    // 创建地图服务图层
    const layer = new MapImageLayer({
      url: "https://renshou.ruankun.xyz:6443/arcgis/rest/services/gis_1/NDVI2/MapServer",
      sublayers: [
        { id: 0, visible: true }  // 12月NDVI.tif (0)
      ]
    });

    // 创建地图实例
    const map = new Map({
      basemap: "topo-vector",  // 添加基础底图
      layers: [layer]          // 添加你的服务图层
    });

     // 创建视图实例
    const view = new MapView({
      container: this.$el,
      map: map,
      extent: {  // 设置初始显示范围
        xmin: 103.74156173024906,
        ymin: 30.088144956279066,
        xmax: 103.87557912703059,
        ymax: 30.179769094895022,
        spatialReference: { wkid: 4326 }
      },
      constraints: {
        minZoom: 12,  // 限制最小缩放级别
        rotationEnabled: false  // 禁用旋转
      }
    });

     // 可选：添加缩放控件
    view.ui.add("zoom", "top-left");

    // 添加底图切换控件
    const basemapToggle = new BasemapToggle({
      view: view,
      nextBasemap: "satellite"  // 切换到卫星底图
    });
    view.ui.add(basemapToggle, "bottom-right");

    
    // 等待视图加载
    view.when(() => {
      console.log("Map loaded successfully");
      // 可以在此添加其他操作
    }).catch(error => {
      console.error("Map load error:", error);
    });
  }
}
</script>

<style scoped>
  @import './main.css';
</style>



