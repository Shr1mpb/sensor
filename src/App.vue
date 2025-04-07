<template>
  <div class="sensor">
    <div class="cyber-bg"></div>
    <h2 class="cyber-title">水下推进器三维传感器监控系统</h2>

    <div class="dashboard">
      <div class="chart-container">
        <!-- 水流方向图表组 -->
        <div class="chart-group">
          <div ref="dirXChart" class="chart-item"></div>
          <div ref="dirYChart" class="chart-item"></div>
          <div ref="dirZChart" class="chart-item"></div>
        </div>

        <!-- 水压图表组 -->
        <div class="chart-group">
          <div ref="pressXChart" class="chart-item"></div>
          <div ref="pressYChart" class="chart-item"></div>
          <div ref="pressZChart" class="chart-item"></div>
        </div>

        <!-- 扭矩图表组 -->
        <div class="chart-group">
          <div ref="twistXChart" class="chart-item"></div>
          <div ref="twistYChart" class="chart-item"></div>
          <div ref="twistZChart" class="chart-item"></div>
        </div>

        <!-- 推力图表组 -->
        <div class="chart-group">
          <div ref="forceXChart" class="chart-item"></div>
          <div ref="forceYChart" class="chart-item"></div>
          <div ref="forceZChart" class="chart-item"></div>
        </div>
      </div>
    </div>

    <!-- 数据面板 -->
    <div class="data-panels">
      <div class="data-card glow-blue">
        <h3>水流方向 (π)</h3>
        <div class="axis-group">
          <div class="axis-row">
            <div class="axis-item">X: <span class="value">{{ dirX }}</span></div>
            <div class="axis-item">Y: <span class="value">{{ dirY }}</span></div>
          </div>
          <div class="axis-row">
            <div class="axis-item">Z: <span class="value">{{ dirZ }}</span></div>
          </div>
        </div>
      </div>

      <div class="data-card glow-green">
        <h3>水压 (N)</h3>
        <div class="axis-group">
          <div class="axis-row">
            <div class="axis-item">X: <span class="value">{{ pressX }}</span></div>
            <div class="axis-item">Y: <span class="value">{{ pressY }}</span></div>
          </div>
          <div class="axis-row">
            <div class="axis-item">Z: <span class="value">{{ pressZ }}</span></div>
          </div>
        </div>
      </div>

      <div class="data-card glow-purple">
        <h3>扭矩 (N·M)</h3>
        <div class="axis-group">
          <div class="axis-row">
            <div class="axis-item">X: <span class="value">{{ twistX }}</span></div>
            <div class="axis-item">Y: <span class="value">{{ twistY }}</span></div>
          </div>
          <div class="axis-row">
            <div class="axis-item">Z: <span class="value">{{ twistZ }}</span></div>
          </div>
        </div>
      </div>

      <div class="data-card glow-orange">
        <h3>推力 (N)</h3>
        <div class="axis-group">
          <div class="axis-row">
            <div class="axis-item">X: <span class="value">{{ forceX }}</span></div>
            <div class="axis-item">Y: <span class="value">{{ forceY }}</span></div>
          </div>
          <div class="axis-row">
            <div class="axis-item">Z: <span class="value">{{ forceZ }}</span></div>
          </div>
        </div>
      </div>
    </div>

  </div>

</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as echarts from 'echarts';

// 响应式数据（数值类型）
const dirX = ref(0);
const dirY = ref(0);
const dirZ = ref(0);
const pressX = ref(0);
const pressY = ref(0);
const pressZ = ref(0);
const twistX = ref(0);
const twistY = ref(0);
const twistZ = ref(0);
const forceX = ref(0);
const forceY = ref(0);
const forceZ = ref(0);

// 图表元素引用
const dirXChart = ref<HTMLElement>();
const dirYChart = ref<HTMLElement>();
const dirZChart = ref<HTMLElement>();
const pressXChart = ref<HTMLElement>();
const pressYChart = ref<HTMLElement>();
const pressZChart = ref<HTMLElement>();
const twistXChart = ref<HTMLElement>();
const twistYChart = ref<HTMLElement>();
const twistZChart = ref<HTMLElement>();
const forceXChart = ref<HTMLElement>();
const forceYChart = ref<HTMLElement>();
const forceZChart = ref<HTMLElement>();

// 图表实例与数据键的映射
type DataKey =
  | 'dirX' | 'dirY' | 'dirZ'
  | 'pressX' | 'pressY' | 'pressZ'
  | 'twistX' | 'twistY' | 'twistZ'
  | 'forceX' | 'forceY' | 'forceZ';

const chartMap = new Map<echarts.ECharts, DataKey>();
const history = new Map<DataKey, number[]>();

// 图表配置
const chartOption = (title: string, color: string) => ({
  title: {
    text: title,
    textStyle: {
      color: '#fff',
      fontFamily: 'Orbitron, sans-serif',
      fontSize: 14
    }
  },
  grid: {
    left: '12%',
    right: '8%',
    bottom: '15%'
  },
  xAxis: {
    type: 'category',
    data: Array.from({length: 120}, (_,i) => i),
    axisLine: { lineStyle: { color: '#7ec1ff' }},
    axisLabel: { color: '#7ec1ff', fontSize: 12 }
  },
  // 添加动画配置
  animation: true,
  animationDuration: 1000,
  animationEasing: 'linear',
  yAxis: {
    type: 'value',
    axisLine: { lineStyle: { color: '#7ec1ff' }},
    axisLabel: { color: '#7ec1ff', fontSize: 12 }
  },
  series: [{
    type: 'line',
    smooth: true,
    showSymbol: false,
    lineStyle: { width: 2, color },
    areaStyle: {
      color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
        { offset: 0, color: `${color}60` },
        { offset: 1, color: `${color}10` }
      ])
    },
    data: [] as number[]
  }]
});

// 初始化图表
const initCharts = () => {
  const configs: [typeof dirXChart, DataKey, string, string][] = [
    [dirXChart, 'dirX', 'X轴水流方向', '#00f2ff'],
    [dirYChart, 'dirY', 'Y轴水流方向', '#00ff9d'],
    [dirZChart, 'dirZ', 'Z轴水流方向', '#ff00ff'],
    [pressXChart, 'pressX', 'X轴水压', '#00f2ff'],
    [pressYChart, 'pressY', 'Y轴水压', '#00ff9d'],
    [pressZChart, 'pressZ', 'Z轴水压', '#ff00ff'],
    [twistXChart, 'twistX', 'X轴扭矩', '#00f2ff'],
    [twistYChart, 'twistY', 'Y轴扭矩', '#00ff9d'],
    [twistZChart, 'twistZ', 'Z轴扭矩', '#ff00ff'],
    [forceXChart, 'forceX', 'X轴推力', '#00f2ff'],
    [forceYChart, 'forceY', 'Y轴推力', '#00ff9d'],
    [forceZChart, 'forceZ', 'Z轴推力', '#ff00ff'],
  ];

  configs.forEach(([chartRef, dataKey, title, color]) => {
    if (!chartRef.value) return;

    const chart = echarts.init(chartRef.value);
    chart.setOption(chartOption(title, color));
    chartMap.set(chart, dataKey);
    history.set(dataKey, []);
  });
};

// 生成三维数据（数值类型）
const generate3DData = () => {
  const vector = {
    x: Math.random(),
    y: Math.random(),
    z: Math.random()
  };
  const magnitude = Math.sqrt(vector.x**2 + vector.y**2 + vector.z**2);

  return {
    direction: {
      x: Math.acos(vector.x/magnitude)/Math.PI,
      y: Math.acos(vector.y/magnitude)/Math.PI,
      z: Math.acos(vector.z/magnitude)/Math.PI
    },
    pressure: {
      x: Math.random() * 10,
      y: Math.random() * 10,
      z: Math.random() * 10
    },
    torque: {
      x: Math.random() * 5,
      y: Math.random() * 5,
      z: Math.random() * 5
    },
    force: {
      x: 20 + Math.random() * 80,
      y: 20 + Math.random() * 80,
      z: 20 + Math.random() * 80
    }
  };
};

// 更新数据
const updateCharts = () => {
  const newData = generate3DData();

  // 更新显示值
  dirX.value = Number(newData.direction.x.toFixed(4));
  dirY.value = Number(newData.direction.y.toFixed(4));
  dirZ.value = Number(newData.direction.z.toFixed(4));
  pressX.value = Number(newData.pressure.x.toFixed(3));
  pressY.value = Number(newData.pressure.y.toFixed(3));
  pressZ.value = Number(newData.pressure.z.toFixed(3));
  twistX.value = Number(newData.torque.x.toFixed(3));
  twistY.value = Number(newData.torque.y.toFixed(3));
  twistZ.value = Number(newData.torque.z.toFixed(3));
  forceX.value = Number(newData.force.x.toFixed(3));
  forceY.value = Number(newData.force.y.toFixed(3));
  forceZ.value = Number(newData.force.z.toFixed(3));

  // 更新历史数据
  const updateHistory = (key: DataKey, value: number) => {
    const data = history.get(key) || [];
    if (data.length >= 60) data.shift();
    data.push(value);
    history.set(key, data);
  };

  updateHistory('dirX', newData.direction.x);
  updateHistory('dirY', newData.direction.y);
  updateHistory('dirZ', newData.direction.z);
  updateHistory('pressX', newData.pressure.x);
  updateHistory('pressY', newData.pressure.y);
  updateHistory('pressZ', newData.pressure.z);
  updateHistory('twistX', newData.torque.x);
  updateHistory('twistY', newData.torque.y);
  updateHistory('twistZ', newData.torque.z);
  updateHistory('forceX', newData.force.x);
  updateHistory('forceY', newData.force.y);
  updateHistory('forceZ', newData.force.z);

  // 更新图表
  chartMap.forEach((dataKey, chart) => {
    const chartData = history.get(dataKey) || [];
    chart.setOption({
      series: [{
        data: chartData
      }]
    });
  });
};

// 生命周期管理
let timer: number;
onMounted(() => {
  initCharts();
  timer = setInterval(updateCharts, 1000);
  updateCharts(); // 初始数据
});

onBeforeUnmount(() => {
  clearInterval(timer);
  chartMap.forEach((_, chart) => chart.dispose());
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap');



.sensor {
  position: relative;
  min-height: 100vh;
  padding: 2rem;
  background: #001219;
  font-family: 'Orbitron', sans-serif;
}

.cyber-bg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background:
    repeating-linear-gradient(
      -45deg,
      transparent,
      transparent 2px,
      rgba(0, 242, 255, 0.1) 3px,
      rgba(0, 242, 255, 0.1) 4px
    ),
    linear-gradient(45deg, #001219 0%, #003d4d 100%);
  z-index: 0;
}

.cyber-title {
  position: relative;
  color: #00f2ff;
  text-align: center;
  font-size: 2.5rem;
  margin: 1rem 0 3rem;
  text-shadow: 0 0 15px rgba(0, 242, 255, 0.7);
}

.dashboard {
  position: relative;
  display: grid;
  grid-template-columns: 1fr; /* 改为单列布局 */
  gap: 2rem;
  max-width: 1800px;
  margin: auto auto;
}

.chart-container {
  display: grid;
  gap: 2rem; /* 图表组间距 */
}

.chart-group {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 每行3个图表 */
  gap: 1.5rem;
}

/* 响应式调整 */
@media (max-width: 1600px) {
  .dashboard {
    grid-template-columns: 1fr;
  }

  .chart-group {
    grid-template-columns: repeat(2, 1fr); /* 中等屏幕显示2个 */
  }
}

@media (max-width: 1200px) {
  .chart-group {
    grid-template-columns: 1fr; /* 小屏幕显示1个 */
  }

}

.chart-group {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.chart-item {
  height: 280px;
  background: rgba(0, 32, 64, 0.3);
  border: 1px solid rgba(0, 242, 255, 0.3);
  border-radius: 8px;
  backdrop-filter: blur(5px);
}

.data-card {
  padding: 1.2rem;
  min-height: 140px;
  border-radius: 12px;
  background: rgba(0, 32, 64, 0.4);
  backdrop-filter: blur(8px);
  transition: all 0.3s ease;
}

.data-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(0, 242, 255, 0.2);
}

.glow-blue { border: 1px solid #00f2ff; box-shadow: 0 0 20px rgba(0, 242, 255, 0.3); }
.glow-green { border: 1px solid #00ff9d; box-shadow: 0 0 20px rgba(0, 255, 157, 0.3); }
.glow-purple { border: 1px solid #ff00ff; box-shadow: 0 0 20px rgba(255, 0, 255, 0.3); }
.glow-orange { border: 1px solid #ff6b00; box-shadow: 0 0 20px rgba(255, 107, 0, 0.3); }

.axis-group {
  margin-top: 0.8rem;
}

.axis-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 0.6rem;
}

.axis-item {
  flex: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.4rem 0.8rem;
  background: rgba(0, 60, 120, 0.3);
  border-radius: 4px;
}

.value {
  color: #00f2ff;
  font-size: 1.05rem;
  min-width: 70px;
  text-align: right;
  text-shadow: 0 0 8px rgba(0, 242, 255, 0.5);
}
.data-panels {
  display: grid;
  grid-template-columns: repeat(2, minmax(300px, 400px)); /* 限制卡片最小和最大宽度 */
  gap: 2rem;
  justify-content: center; /* 水平居中 */
  width: 100%;
  max-width: 1200px; /* 限制最大宽度 */
  margin: 0 auto; /* 水平居中 */
}
@media (max-width: 1600px) {
  .dashboard {
    grid-template-columns: 1fr;
    max-width: 1200px;
  }

  .chart-group {
    grid-template-columns: 1fr;
  }

  .chart-item {
    height: 280px !important;
    width: 100% !important;
  }
}
</style>
