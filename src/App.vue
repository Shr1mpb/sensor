<template>
  <div class="sensor">
    <h2>传感器数据
      <button class="copy-button" id="copyButton">复制当前数据
        <span class="tooltip" id="tooltip" ref="tooltip"></span>
      </button>
    </h2>

    <div class="data">
      <div>
        <span>水流方向:(与坐标轴夹角)</span>
        <div class="axis">X轴: <span class="inline-data">{{waterDirX}}π</span></div>
        <div class="axis">Y轴: <span class="inline-data">{{waterDirY}}π</span></div>
        <div class="axis">Z轴: <span class="inline-data">{{waterDirZ}}π</span></div>
      </div>
      <div>
        <span>水流压力:(N)</span>
        <div class="axis">X轴: <span class="inline-data">{{waterPressX}}</span></div>
        <div class="axis">Y轴: <span class="inline-data">{{waterPressY}}</span></div>
        <div class="axis">Z轴: <span class="inline-data">{{waterPressZ}}</span></div>
      </div>
      <div>
        <span>水流扭矩:(N·M)</span>
        <div class="axis">X轴: <span class="inline-data">{{waterTwistX}}</span></div>
        <div class="axis">Y轴: <span class="inline-data">{{waterTwistY}}</span></div>
        <div class="axis">Z轴: <span class="inline-data">{{waterTwistZ}}</span></div>
      </div>
      <div>
        <span>喷口所需推力:(N)</span>
        <div class="axis">X轴: <span class="inline-data">{{forceX}}</span></div>
        <div class="axis">Y轴: <span class="inline-data">{{forceY}}</span></div>
        <div class="axis">Z轴: <span class="inline-data">{{forceZ}}</span></div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import {ref} from "vue";

  //双向绑定
  const waterDirX = ref();
  const waterDirY = ref();
  const waterDirZ = ref();

  const waterPressX = ref();
  const waterPressY = ref();
  const waterPressZ = ref();

  const waterTwistX = ref();
  const waterTwistY = ref();
  const waterTwistZ = ref();

  const forceX = ref();
  const forceY = ref();
  const forceZ = ref();

  //生成介于 min 和 max 之间的随机数
  const getRandomFloat = (min: number, max: number, decimals: number): number => {

    const factor = Math.pow(10, decimals);
    const randomNumber = Math.random() * (max - min) + min;
    return Math.round(randomNumber * factor) / factor; //格式化
  }
  const calculateAngles = (x: number, y: number, z: number): [string, string, string] => {
    const r = Math.sqrt(x ** 2 + y ** 2 + z ** 2);

    const alpha = Math.acos(x / r);
    const beta = Math.acos(y / r);
    const gamma = Math.acos(z / r);

    return [
      alpha.toFixed(8),
      beta.toFixed(8),
      gamma.toFixed(8)
    ]
  }


  const x = getRandomFloat(0, 1, 5);
  const y = getRandomFloat(0, 1, 5);
  const z = getRandomFloat(0, 1, 5);
  const angles = calculateAngles(x, y, z);
  waterDirX.value = angles[0];
  waterDirY.value = angles[1];
  waterDirZ.value = angles[2];

  waterPressX.value = getRandomFloat(0, 1, 5);
  waterPressY.value = getRandomFloat(0, 1, 5);
  waterPressZ.value = getRandomFloat(0, 1, 5);

  waterTwistX.value = getRandomFloat(0, 1, 5);
  waterTwistY.value = getRandomFloat(0, 1, 5);
  waterTwistZ.value = getRandomFloat(0, 1, 5);

  forceX.value = getRandomFloat(10, 100, 5);
  forceY.value = getRandomFloat(10, 100, 5);
  forceZ.value = getRandomFloat(10, 100, 5);

  //水流扭矩和喷口所需推力(这里要有公式了)

</script>

<style scoped>
/* 添加淡入动画 */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.sensor {
  animation: fadeIn 1s ease-out forwards;
  opacity: 0;
}

.data > div {
  animation: fadeIn 0.5s ease-out forwards;
  opacity: 0;
}

/* 为每个数据组添加延迟 */
.data > div:nth-child(1) { animation-delay: 0.2s; }
.data > div:nth-child(2) { animation-delay: 0.4s; }
.data > div:nth-child(3) { animation-delay: 0.6s; }
.data > div:nth-child(4) { animation-delay: 0.8s; }

.sensor {
  background-color: #f9f9f9;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 30px;
  max-width: 600px;
  margin: 20px auto;
  font-family: Arial, sans-serif;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.sensor h2 {
  font-size: 1.8em;
  margin-bottom: 20px;
  margin-right: 20px;
}

.sensor .data {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.sensor .data div {
  /*flex-grow flex-shrink flex-basis*/
  flex: 0 1 calc(50% - 20px); /* 每个项占据50%的宽度，减去间距 */
  padding: 15px;
  background-color: #fff;
  border-radius: 5px; /* 圆角 */
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.sensor .data span {
  font-weight: bold;
}

.axis {
  margin-top: 10px;
}

.copy-button {
  margin-left: 20px;
  padding: 10px 15px;
  background-color: #24c2ea;
  color: #000000;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-family: Arial, sans-serif;
  display: none;
}

.tooltip {
  display: none;
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background-color: #333;
  color: white;
  padding: 8px;
  border-radius: 5px;
  white-space: nowrap; /* 不换行 */
  z-index: 10; /* 保证浮窗在其他元素之上 */
  font-size: 0.9em;
}
</style>
