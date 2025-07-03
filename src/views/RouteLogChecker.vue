<template>
  <div>
    <MyMap
      class="main-container"
      @init="onMapInit"
      :autoComplete="false"
      :resize="false"
    >
      <div class="inputs">
        <el-input
          v-for="(item, index) in inputs"
          :key="index"
          v-model="item.text"
          :placeholder="`路线${index + 1}（${item.color}线）请输入经纬度对`"
          @keyup.enter.native="drawLine(index)"
          clearable
        />
        <el-button type="danger" @click="clearAllPolylines">清除全部路线</el-button>
      </div>
    </MyMap>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';
import MyMap from '@/components/MyMap/index.vue';

const map = ref(null);
const inputs = reactive([
  { text: '', color: 'red' },
  { text: '', color: 'green' },
  { text: '', color: 'blue' },
  { text: '', color: 'purple' },
]);

const polylines = [];
const markers = [];

const onMapInit = (m) => {
  map.value = m;
};

function drawLine(index) {
  const input = inputs[index];
  if (!input.text.trim()) return;
  const coords = input.text
    .split(';')
    .map(pair => {
      const [lng, lat] = pair.split(',').map(Number);
      return [lng, lat];
    })
    .filter(p => p.length === 2 && !isNaN(p[0]) && !isNaN(p[1]));

  if (!coords.length || !window.AMap || !map.value) return;

  // 绘制折线
  const polyline = new window.AMap.Polyline({
    path: coords,
    strokeColor: input.color,
    strokeWeight: 4,
    strokeOpacity: 1,
    lineJoin: 'round'
  });
  polyline.setMap(map.value);
  polylines.push(polyline);

  // 添加坐标点标记
  coords.forEach(([lng, lat], i) => {
    const marker = new window.AMap.Marker({
      position: [lng, lat],
      label: {
        content: `${lng.toFixed(6)},${lat.toFixed(6)}`,
        offset: new window.AMap.Pixel(10, -20),
      },
      icon: 'https://webapi.amap.com/theme/v1.3/markers/n/mark_b.png',
      map: map.value
    });
    markers.push(marker);
  });

  map.value.setFitView([polyline]);
}

function clearAllPolylines() {
  polylines.forEach(polyline => {
    if (map.value && polyline) {
      map.value.remove(polyline);
    }
  });
  polylines.length = 0;

  markers.forEach(marker => {
    if (map.value && marker) {
      map.value.remove(marker);
    }
  });
  markers.length = 0;
}
</script>

<style scoped>
.main-container {
  height: 100vh;
}
.inputs {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  gap: 10px;
  background: rgba(255, 255, 255, 0.9);
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
  width: 300px;
}
</style>
