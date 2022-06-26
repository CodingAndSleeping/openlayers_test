<script setup>
import { ref, watch } from 'vue';
const emits = defineEmits(["changeLayer", "changeOpacity"])
// 定义一个变量记录当前图层名字
const currentLayer = ref("OSM");
const OSMOpacity = ref(1);
const tianOpacity = ref(1);
// 当前图层透明度
const currnetOpacity = ref(1);
// 切换图层
function changeLayer(e) {
  // 更换当前图层名字为所点单选框的值
  currentLayer.value = e.target.value;
  currentLayer.value == "OSM" ? currnetOpacity.value = OSMOpacity.value : currnetOpacity.value = tianOpacity.value;
  emits("changeLayer", currentLayer.value);
}
// 监视透明度变化
watch(currnetOpacity, newVal => {
  currentLayer.value == "OSM" ? OSMOpacity.value = newVal : tianOpacity.value = newVal;
  emits("changeOpacity", newVal, currentLayer.value);
})
</script>

<template>
  <!-- 图层切换控件 -->
  <div class="changeLayer">
    <ul>
      <li>
        <input type="radio" value="OSM" v-model="currentLayer" @click="changeLayer">
        <label>OSM</label>
      </li>
      <li>
        <input type="radio" value="Tian" v-model="currentLayer" @click="changeLayer">
        <label>天地图</label>
      </li>
    </ul>
    <input type="range" :min="0" :max="1" :step="0.01" v-model="currnetOpacity">
  </div>
</template>

<style lang="less" scoped>
// 图层切换控件
.changeLayer {
  position: fixed;
  background-color: #7C9ABE;
  outline: 5px solid rgba(255, 255, 255, 0.5);
  border-radius: 10px;
  top: 10px;
  right: 10px;
  z-index: 100;

  ul {
    padding: 0;
    margin: 5px;

    li {
      padding: 0;
      list-style: none;
    }
  }
}
</style>
