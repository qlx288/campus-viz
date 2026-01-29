<template>
  <div class="right-panel">
    <div class="panel-section">
      <div class="section-header">
        <h3 class="section-title">实时监控</h3>
        <div class="section-decoration"></div>
      </div>
      <div ref="monitorChart" class="chart-container"></div>
    </div>

    <div class="panel-section">
      <div class="section-header">
        <h3 class="section-title">能源消耗</h3>
        <div class="section-decoration"></div>
      </div>
      <div ref="energyChart" class="chart-container"></div>
    </div>

    <div class="panel-section">
      <div class="section-header">
        <h3 class="section-title">最新动态</h3>
        <div class="section-decoration"></div>
      </div>
      <div class="news-list">
        <div v-for="(news, index) in newsList" :key="index" class="news-item">
          <div class="news-time">{{ news.time }}</div>
          <div class="news-content">{{ news.content }}</div>
          <div class="news-badge" :class="news.type">{{ news.type }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const monitorChart = ref(null)
const energyChart = ref(null)
let monitorChartInstance = null
let energyChartInstance = null

const newsList = [
  { time: '10:30', content: '图书馆二楼新增智能自习区', type: 'info' },
  { time: '10:15', content: '北门闸机系统升级完成', type: 'success' },
  { time: '09:45', content: '3号宿舍楼用电异常告警', type: 'warning' },
  { time: '09:30', content: '体育馆预约系统开放', type: 'info' },
  { time: '08:50', content: '食堂4号档口设备维护', type: 'info' },
  { time: '08:30', content: '早自习出勤率统计更新', type: 'success' },
]

onMounted(() => {
  initMonitorChart()
  initEnergyChart()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (monitorChartInstance) {
    monitorChartInstance.dispose()
  }
  if (energyChartInstance) {
    energyChartInstance.dispose()
  }
  window.removeEventListener('resize', handleResize)
})

const handleResize = () => {
  monitorChartInstance?.resize()
  energyChartInstance?.resize()
}

const initMonitorChart = () => {
  if (!monitorChart.value) return

  monitorChartInstance = echarts.init(monitorChart.value)

  const hours = Array.from({ length: 24 }, (_, i) => `${i}:00`)
  const data1 = hours.map(() => Math.floor(Math.random() * 500 + 200))
  const data2 = hours.map(() => Math.floor(Math.random() * 300 + 100))

  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'cross',
        label: {
          backgroundColor: '#283b56'
        }
      }
    },
    legend: {
      data: ['入校人数', '出校人数'],
      textStyle: {
        color: '#e2e8f0'
      },
      top: 0
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: [
      {
        type: 'category',
        boundaryGap: false,
        data: hours.filter((_, i) => i % 2 === 0),
        axisLabel: {
          color: '#a0aec0',
          fontSize: 11
        },
        axisLine: {
          lineStyle: {
            color: 'rgba(59, 130, 246, 0.3)'
          }
        }
      }
    ],
    yAxis: [
      {
        type: 'value',
        axisLabel: {
          color: '#a0aec0',
          fontSize: 12
        },
        splitLine: {
          lineStyle: {
            color: 'rgba(59, 130, 246, 0.1)'
          }
        }
      }
    ],
    series: [
      {
        name: '入校人数',
        type: 'line',
        stack: 'Total',
        smooth: true,
        lineStyle: {
          width: 2,
          color: '#3b82f6'
        },
        areaStyle: {
          opacity: 0.3,
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: '#3b82f6' },
            { offset: 1, color: 'rgba(59, 130, 246, 0)' }
          ])
        },
        emphasis: {
          focus: 'series'
        },
        data: data1
      },
      {
        name: '出校人数',
        type: 'line',
        stack: 'Total',
        smooth: true,
        lineStyle: {
          width: 2,
          color: '#06b6d4'
        },
        areaStyle: {
          opacity: 0.3,
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: '#06b6d4' },
            { offset: 1, color: 'rgba(6, 182, 212, 0)' }
          ])
        },
        emphasis: {
          focus: 'series'
        },
        data: data2
      }
    ]
  }

  monitorChartInstance.setOption(option)
}

const initEnergyChart = () => {
  if (!energyChart.value) return

  energyChartInstance = echarts.init(energyChart.value)

  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      }
    },
    legend: {
      data: ['用电量', '用水量'],
      textStyle: {
        color: '#e2e8f0'
      },
      top: 0
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: ['教学楼', '宿舍楼', '图书馆', '实验楼', '体育馆'],
      axisLabel: {
        color: '#a0aec0',
        fontSize: 12,
        rotate: 30
      },
      axisLine: {
        lineStyle: {
          color: 'rgba(59, 130, 246, 0.3)'
        }
      }
    },
    yAxis: {
      type: 'value',
      axisLabel: {
        color: '#a0aec0',
        fontSize: 12
      },
      splitLine: {
        lineStyle: {
          color: 'rgba(59, 130, 246, 0.1)'
        }
      }
    },
    series: [
      {
        name: '用电量',
        type: 'bar',
        data: [8200, 9300, 5500, 4200, 3800],
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: '#f59e0b' },
            { offset: 1, color: '#d97706' }
          ]),
          borderRadius: [4, 4, 0, 0]
        }
      },
      {
        name: '用水量',
        type: 'bar',
        data: [1200, 2800, 900, 650, 450],
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: '#06b6d4' },
            { offset: 1, color: '#0891b2' }
          ]),
          borderRadius: [4, 4, 0, 0]
        }
      }
    ]
  }

  energyChartInstance.setOption(option)
}
</script>

<style scoped>
.right-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 380px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: rgba(59, 130, 246, 0.3) transparent;
}

.right-panel::-webkit-scrollbar {
  width: 4px;
}

.right-panel::-webkit-scrollbar-thumb {
  background: rgba(59, 130, 246, 0.3);
  border-radius: 2px;
}

.panel-section {
  background: linear-gradient(135deg, rgba(22, 36, 71, 0.8) 0%, rgba(16, 24, 56, 0.8) 100%);
  border: 1px solid rgba(59, 130, 246, 0.2);
  border-radius: 12px;
  padding: 20px;
  position: relative;
  overflow: hidden;
}

.panel-section::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, transparent, #3b82f6, transparent);
}

.section-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 16px;
}

.section-title {
  font-size: 16px;
  font-weight: bold;
  color: #ffffff;
  margin: 0;
  text-shadow: 0 0 10px rgba(59, 130, 246, 0.5);
}

.section-decoration {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, #3b82f6, transparent);
  opacity: 0.5;
}

.chart-container {
  width: 100%;
  height: 240px;
}

.news-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-height: 300px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: rgba(59, 130, 246, 0.3) transparent;
}

.news-list::-webkit-scrollbar {
  width: 4px;
}

.news-list::-webkit-scrollbar-thumb {
  background: rgba(59, 130, 246, 0.3);
  border-radius: 2px;
}

.news-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px;
  background: rgba(59, 130, 246, 0.05);
  border-radius: 8px;
  border-left: 3px solid rgba(59, 130, 246, 0.3);
  transition: all 0.3s ease;
}

.news-item:hover {
  background: rgba(59, 130, 246, 0.1);
  transform: translateX(4px);
}

.news-time {
  font-size: 12px;
  color: #a0aec0;
  white-space: nowrap;
  min-width: 50px;
}

.news-content {
  flex: 1;
  font-size: 13px;
  color: #e2e8f0;
}

.news-badge {
  padding: 3px 8px;
  font-size: 11px;
  border-radius: 10px;
  white-space: nowrap;
}

.news-badge.info {
  background: rgba(59, 130, 246, 0.2);
  color: #3b82f6;
  border: 1px solid rgba(59, 130, 246, 0.3);
}

.news-badge.success {
  background: rgba(16, 185, 129, 0.2);
  color: #10b981;
  border: 1px solid rgba(16, 185, 129, 0.3);
}

.news-badge.warning {
  background: rgba(245, 158, 11, 0.2);
  color: #f59e0b;
  border: 1px solid rgba(245, 158, 11, 0.3);
}
</style>
