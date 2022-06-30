<script setup>
import { ref, watch } from 'vue';
const emits = defineEmits(["drawInteraction"]); // 接收事件 
// 绘制图形的类型名
const featureName = ref(null);
// 监听类型名的变化，发生变化时提交事件
watch(featureName, newVal => {
    if (newVal) {
        emits("drawInteraction", newVal);
    }
})
// 取消绘制
function cancelDraw() {
    featureName.value = null; // 类型名置空
    emits("drawInteraction", "Cancel"); // 提交事件
}
// 清空绘制的图形
function clearFeature() {
    featureName.value = null; // 类型名置空
    emits("drawInteraction", "Clear"); // 提交事件
}
</script>

<template>
    <div class="drawFeature">
        <div class="title">绘制图形</div>
        <select class="select" v-model="featureName">
            <option disabled selected value="null">请选择</option>
            <option value="Point">点</option>
            <option value="LineString">折线</option>
            <option value="Curve">曲线</option>
            <option value="Circle">圆形</option>
            <option value="Square">正方形</option>
            <option value="Rectangle">矩形</option>
            <option value="Polygon">规则多边形</option>
            <option value="ArbPolygon">不规则图形</option>
        </select>
        <div class="cancelDraw" @click="cancelDraw">取消绘制</div>
        <div class="clearFeature" @click="clearFeature">清除</div>
    </div>
</template>


<style lang="less">
.drawFeature {
    width: 113px;
    padding: 10px;
    position: fixed;
    top: 100px;
    right: 10px;
    background-color: #7C9ABE;
    outline: 5px solid rgba(255, 255, 255, 0.5);
    border-radius: 10px;
    z-index: 100;
    .title {
        color: #f0f0f0;
        text-align: center;
    }
    .select {
        display: block;
        margin: 5px auto;
        border: none;
    }

    .clearFeature,
    .cancelDraw {
        margin-top: 10px;
        text-align: center;
        border-radius: 10px;
        background-color: #F6F5F1;
        cursor: pointer;
    }
}
</style>