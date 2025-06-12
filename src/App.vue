<template>
  <div class="mapdiv" ref="mapdiv"></div>
  <Chat />
</template>

<script>
// import WebMap from "@arcgis/core/WebMap";
// import MapView from "@arcgis/core/views/MapView";
// import Bookmarks from "@arcgis/core/widgets/Bookmarks";
// import Expand from "@arcgis/core/widgets/Expand"; 

// https://10.255.0.172:6443/arcgis/rest/services/xiangchengliangbanji2/FeatureServer

//MAPS SDK FOR JAVASCRIPT
import Map from "@arcgis/core/Map";
import MapView from "@arcgis/core/views/MapView";
import MapImageLayer from "@arcgis/core/layers/MapImageLayer";
import BasemapToggle from "@arcgis/core/widgets/BasemapToggle";
import Chat from './components/Chat/Chat.vue';
import FeatureServer from "@arcgis/core/layers/FeatureLayer";
import Point from '@arcgis/core/geometry/Point'
import SpatialReference from '@arcgis/core/geometry/SpatialReference'
import Camera from '@arcgis/core/Camera'

export default {
  name: 'App',
  components: { Chat },
  async mounted() {
    // 创建地图服务图层
    const layer = new MapImageLayer({
      url: "	https://10.255.0.172:6443/arcgis/rest/services/NDVI_TEST_SERVER/MapServer",
      sublayers: [
        { id: 0, visible: true }  // 12月NDVI.tif (0)
      ]
    });

    let layer2 = new FeatureServer({
      url: "https://10.255.0.172:6443/arcgis/rest/services/xiangchengliangbanji2/FeatureServer/8",
      outFields: ["*"],  // 获取所有字段
      popupEnabled: true,  // 启用弹出窗口
      title: "水系",  // 图层标题
      popupTemplate: {
        content: element => {
          console.log(element.graphic.layer.title, element.graphic.attributes)
          //将element.graphic.attributes.srname存入localStorage中，并弹出聊天窗口
          localStorage.setItem('srname', element.graphic.attributes.rname);
          return `<b>${element.graphic.layer.title}</b><br>水系名称: ${element.graphic.attributes.srname}<br>水系长度: ${element.graphic.attributes.srlength}米`;
        }
      }
    });

    // 创建地图实例
    const map = new Map({
      basemap: "topo-vector",  // 添加基础底图
      layers: [layer, layer2]          // 添加你的服务图层
    });

     // 创建视图实例
    const view = new MapView({
      container: this.$refs.mapdiv,  // 挂载到组件的 mapdiv
      map: map,
      center: [99.74587169702623, 29.074594554358285],
      constraints: {
        minZoom: 10,  // 限制最小缩放级别
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



