<script setup>
import 'ol/ol.css'
import Map from 'ol/Map';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import XYZ from 'ol/source/XYZ';
import View from 'ol/View';
import { Zoom, ZoomSlider, MousePosition, ScaleLine, OverviewMap, ZoomToExtent } from 'ol/control';
import { createStringXY, toStringXY } from 'ol/coordinate';
import { onMounted, ref } from 'vue';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Draw from 'ol/interaction/Draw';
import { createRegularPolygon, createBox } from 'ol/interaction/Draw';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import Style from 'ol/style/Style';
import Icon from 'ol/style/Icon';
import Text from 'ol/style/Text';
import Overlay from 'ol/Overlay';
import LayerChange from './LayerChange.vue';
import DrawFeature from './DrawFeature.vue';


// 初始化一个空地图
const map = ref(null);
const OSMLayer = ref(null);
const tianLayer = ref(null);
// 初始化地图函数
function initMap() {
    // OSM图层
    OSMLayer.value = new TileLayer({
        name: "OSM",
        source: new OSM(),
        visible: true
    })
    // 天地图图层
    tianLayer.value = new TileLayer({
        name: "Tian",
        source: new XYZ({
            url: "http://t0.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=172ce7edc768e16c69e20ca29bdfaee1"
        }),
        visible: false
    })
    // // 初始化一个空的矢量图层
    // vectorLayer.value = new VectorLayer({
    //     name: "Vector"
    // })
    // 创建一个地图
    map.value = new Map({
        // 存放地图的容器
        target: "mapDiv",
        // 设置图层
        layers: [
            // osm图层
            OSMLayer.value,
            // 天地图图层
            tianLayer.value,
            // 矢量图层
            // vectorLayer.value
        ],
        // 设置视图
        view: new View({
            center: [113.300839, 23.048857], // 中心点坐标
            zoom: 10, // 缩放等级
            projection: "EPSG:4326" // 坐标系
        }),
        // 设置地图控件，值为一个数组
        controls: [
            new ZoomToExtent(), //导航控件
            new Zoom(), // 缩放按钮
            new ZoomSlider(), // 缩放滑块
            new ScaleLine(), // 比例尺
            new OverviewMap({ // 鹰眼
                layers: [ // 鹰眼的图层，这里使用OSM地图
                    new TileLayer({
                        name: "osm",
                        source: new OSM(),
                        visible: true
                    })
                ]
            }),
            new MousePosition({ // 鼠标坐标点控件
                target: "mapPoint", // 存放坐标的html元素
                coordinateFormat: createStringXY(3),  // 坐标格式
                placeholder: "" // 当鼠标离开地图，无坐标信息时显示的内容
            })]
    });

}

// 切换图层
function changeLayer(val) {
    if (val == "OSM") {
        OSMLayer.value.setVisible(true);
        tianLayer.value.setVisible(false);
    } else {
        OSMLayer.value.setVisible(false);
        tianLayer.value.setVisible(true);
    }
}
// 改变透明度
function changeOpacity(val, currentLayer) {
    map.value.getLayers().forEach(item => {
        if (item.get("name") == currentLayer) {
            item.setOpacity(parseFloat(val));
        }
    });
}


// 绘制图形
// 创建一个空的Draw
const draw = ref(null);
// 创建一个空的矢量图层源
const vectorSource = ref(null);
const vectorLayer = ref(null);
// 定义一个geometryFunction
let geometryFunction;
// 封装绘制图形函数
function drawFea(map, draw, layer, type, source, geometryFunction, freehand) {
    // 判断source是否为空，若为空，创建一个VectorSource对象，并设置为矢量图层的源
    if (!layer.value) {
        layer.value = new VectorLayer({
            name: "Vector"
        })
    }
    if (!source.value) {
        source.value = new VectorSource({
            wrapX: false,
        })
        map.value.addLayer(layer.value);
    }
    layer.value.setSource(source.value);
    // 创建绘图工具
    draw.value = new Draw({
        type, // 绘制图形类型
        source: source.value, // 绘制的目标源
        geometryFunction, // 绘图的几何函数
        freehand // 绘图时鼠标点击后是否松手进行绘制
    });
    // 给地图添加绘图交互
    map.value.addInteraction(draw.value);
}
// 绘图事件函数
function drawInteraction(val) {
    if (draw.value) {
        // 判断draw是否为空，若不为空移除上一个绘图交互
        map.value.removeInteraction(draw.value);
    }
    switch (val) {
        case "Cancel": // 取消绘制
            break;
        case "Clear": // 清空绘制
            vectorSource.value = null; // 矢量图层源置空
            if (vectorLayer.value) {
                vectorLayer.value.setSource(vectorSource.value); // 设置空的矢量图层源
            }
            map.value.removeLayer(vectorLayer.value);
            break;
        case "Curve": // 绘制曲线
            drawFea(map, draw, vectorLayer, "LineString", vectorSource, undefined, true);
            break;
        case "ArbPolygon": // 绘制不规则图形
            drawFea(map, draw, vectorLayer, "Polygon", vectorSource, undefined, true);
            break;
        case "Square": // 绘制正方形
            // 定义一个4条边的多边形几何函数
            geometryFunction = createRegularPolygon(4)
            drawFea(map, draw, vectorLayer, "Circle", vectorSource, geometryFunction, false);
            break;
        case "Rectangle": // 绘制矩形
            // 定义一个矩形几何函数
            geometryFunction = createBox()
            drawFea(map, draw, vectorLayer, "Circle", vectorSource, geometryFunction, true);
            break;
        default: // 绘制点、圆、折线、规则多边形
            geometryFunction = undefined;
            drawFea(map, draw, vectorLayer, val, vectorSource, undefined, false);
    }
}





// 添加标注
let event
const labelSource = ref(null);
const labelLayer = ref(null);
function setFeatureSytle(feature) {
    return new Style({
        text: new Text({
            text: feature.get("name"),
            offsetY: 10
        }),
        image: new Icon({
            src: "src/assets/position.png",
            anchor: [0.5, 0.85],
            scale: 0.2
        }),
    })
}

function addLabel() {
    map.value.un(propEvent.type, propEvent.listener)
    if (!labelLayer.value) {
        labelLayer.value = new VectorLayer({
            name: "Label"
        })
    }
    if (!labelSource.value) {
        labelSource.value = new VectorSource({
            wrapX: false,
        })
        map.value.addLayer(labelLayer.value);
    }
    labelLayer.value.setSource(labelSource.value);
    event = map.value.on("click", (e) => {
        const coordinate = toStringXY(e.coordinate, 3);
        const feature = new Feature({
            name: "标注",
            coordinate,
            geometry: new Point(e.coordinate)
        })
        feature.setStyle(setFeatureSytle(feature));
        labelSource.value.addFeature(feature);
    })

}
function cancelAddLabel() {
    if (event) {
        map.value.un(event.type, event.listener);
    }
    if (propEvent) {
        map.value.on(propEvent.type, propEvent.listener);
    }
}

function clearLabel() {
    if (event) {
        map.value.un(event.type, event.listener);
    }
    labelSource.value = null;
    if (labelLayer.value) {
        labelLayer.value.setSource(labelSource.value);
    }
    map.value.removeLayer(labelLayer.value);
}

  

const coordinate = ref("");
let propEvent;
const popupElement = ref(null);
const popup = ref(null);
// 组件挂载时初始化地图
onMounted(() => {
    initMap();
    propEvent = map.value.on("click", (e) => {
        popup.value = new Overlay({
            element: popupElement.value
        })
        const feature = map.value.forEachFeatureAtPixel(e.pixel, (feature) => {
            return feature;
        }, {
            layerFilter(layer) {
                return layer.get("name") == "Label"
            }
        });
        if (feature) {
            coordinate.value = feature.get("coordinate")
            popup.value.setPosition(e.coordinate);
            map.value.addOverlay(popup.value);
        } else {
            coordinate.value = "";
            map.value.removeOverlay(popup);
        }
    })
})

</script>

<template>
    <!-- 定义一个容器存放地图 -->
    <div id="mapDiv">
        <!-- 存放坐标的容器 -->
        <div id="mapPoint"></div>
        <layer-change @changeLayer="changeLayer" @changeOpacity="changeOpacity"></layer-change>
        <draw-feature @drawInteraction="drawInteraction"></draw-feature>
        <div class="addLabel">
            <span class="title">添加标记</span>
            <div @click="addLabel">点击添加</div>
            <div @click="cancelAddLabel">取消添加</div>
            <div @click="clearLabel">清除标记</div>
        </div>
        <div class="popup" ref="popupElement">{{ coordinate }}</div>
    </div>
</template>

<style lang="less" >
#mapDiv {
    // 地图容器
    width: 100vw;
    height: 100vh;
    position: absolute;

    #mapPoint {
        // 坐标点
        width: 140px;
        height: 30px;
        position: fixed;
        left: 200px;
        bottom: 1px;
        z-index: 1000;
    }

    .addLabel {
        color: #000000;
        width: 113px;
        padding: 10px;
        background-color: #7C9ABE;
        outline: 5px solid rgba(255, 255, 255, 0.5);
        border-radius: 10px;
        position: fixed;
        right: 10px;
        top: 240px;
        z-index: 100;


        .title {
            display: block;
            color: #f0f0f0;
            text-align: center;
        }

        div {
            text-align: center;
            background-color: #f0f0f0;
            border-radius: 10px;
            margin: 5px 0;
            cursor: pointer;
        }
    }

    .popup {
        // border: 1px solid black;
        background-color: wheat;
    }
}

// 调整鹰眼位置
.ol-overviewmap {
    left: auto;
    right: 0;
}

//调整导航控件位置
.ol-zoom-extent {
    top: 300px;
}
</style>
