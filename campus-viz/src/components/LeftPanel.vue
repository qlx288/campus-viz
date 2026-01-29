<template>
  <div class="left-panel">
    <div class="panel-section">
      <div class="section-header">
        <h3 class="section-title">校园概况</h3>
        <div class="section-decoration"></div>
      </div>
      <div class="stats-grid">
        <div v-for="(stat, index) in stats" :key="index" class="stat-card">
          <div class="stat-icon" :style="{ color: stat.color }">{{ stat.icon }}</div>
          <div class="stat-info">
            <div class="stat-value">{{ stat.value }}</div>
            <div class="stat-label">{{ stat.label }}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="panel-section">
      <div class="section-header">
        <h3 class="section-title">学生统计</h3>
        <div class="section-decoration"></div>
      </div>
      <div ref="studentChart" class="chart-container"></div>
    </div>

    <div class="panel-section">
      <div class="section-header">
        <h3 class="section-title">设施分布</h3>
        <div class="section-decoration"></div>
      </div>
      <div ref="facilityChart" class="chart-container"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const studentChart = ref(null)
const facilityChart = ref(null)
let studentChartInstance = null
let facilityChartInstance = null

const stats = [
  { icon: '🎓', label: '在校学生', value: '25,680', color: '#3b82f6' },
  { icon: '👨‍🏫', label: '教职工', value: '2,150', color: '#06b6d4' },
  { icon: '🏢', label: '教学楼', value: '32', color: '#8b5cf6' },
  { icon: '🛏️', label: '宿舍楼', value: '18', color: '#ec4899' },
]

onMounted(() => {
  initStudentChart()
  initFacilityChart()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (studentChartInstance) {
    studentChartInstance.dispose()
  }
  if (facilityChartInstance) {
    facilityChartInstance.dispose()
  }
  window.removeEventListener('resize', handleResize)
})

const handleResize = () => {
  studentChartInstance?.resize()
  facilityChartInstance?.resize()
}

const initStudentChart = () => {
  if (!studentChart.value) return

  studentChartInstance = echarts.init(studentChart.value)

  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      }
    },
    grid: {
      top: '10%',
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: ['大一', '大二', '大三', '大四', '研究生'],
      axisLabel: {
        color: '#a0aec0',
        fontSize: 12
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
        name: '男生',
        type: 'bar',
        data: [3200, 3100, 2900, 2800, 1500],
        itemStyle: {
          color: '#3b82f6',
          borderRadius: [4, 4, 0, 0]
        }
      },
      {
        name: '女生',
        type: 'bar',
        data: [3300, 3200, 3000, 2900, 1600],
        itemStyle: {
          color: '#06b6d4',
          borderRadius: [4, 4, 0, 0]
        }
      }
    ],
    legend: {
      data: ['男生', '女生'],
      textStyle: {
        color: '#e2e8f0'
      },
      top: 0
    }
  }

  studentChartInstance.setOption(option)
}

const initFacilityChart = () => {
  if (!facilityChart.value) return

  facilityChartInstance = echarts.init(facilityChart.value)

  const option = {
    tooltip: {
      trigger: 'item'
    },
    legend: {
      orient: 'horizontal',
      bottom: 0,
      textStyle: {
        color: '#e2e8f0'
      }
    },
    series: [
      {
        name: '设施类型',
        type: 'pie',
        radius: ['40%', '70%'],
        center: ['50%', '45%'],
        avoidLabelOverlap: false,
        itemStyle: {
          borderRadius: 8,
          borderColor: '#111838',
          borderWidth: 2
        },
        label: {
          show: false,
          position: 'center'
        },
        emphasis: {
          label: {
            show: true,
            fontSize: 16,
            fontWeight: 'bold',
            color: '#ffffff'
          }
        },
        labelLine: {
          show: false
        },
        data: [
          { value: 32, name: '教学楼', itemStyle: { color: '#3b82f6' } },
          { value: 18, name: '宿舍楼', itemStyle: { color: '#06b6d4' } },
          { value: 8, name: '实验楼', itemStyle: { color: '#8b5cf6' } },
          { value: 5, name: '图书馆', itemStyle: { color: '#ec4899' } },
          { value: 12, name: '行政楼', itemStyle: { color: '#f59e0b' } },
          { value: 6, name: '体育馆', itemStyle: { color: '#10b981' } }
        ]
      }
    ]
  }

  facilityChartInstance.setOption(option)
}
</script>

<style scoped>
.left-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 380px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: rgba(59, 130, 246, 0.3) transparent;
}

.left-panel::-webkit-scrollbar {
  width: 4px;
}

.left-panel::-webkit-scrollbar-thumb {
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

.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.stat-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  background: rgba(59, 130, 246, 0.08);
  border-radius: 8px;
  border: 1px solid rgba(59, 130, 246, 0.15);
  transition: all 0.3s ease;
  cursor: pointer;
}

.stat-card:hover {
  background: rgba(59, 130, 246, 0.15);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.2);
}

.stat-icon {
  font-size: 32px;
  flex-shrink: 0;
}

.stat-info {
  display: flex;
  flex-direction: column;
}

.stat-value {
  font-size: 20px;
  font-weight: bold;
  color: #ffffff;
  line-height: 1.2;
}

.stat-label {
  font-size: 12px;
  color: #a0aec0;
  margin-top: 4px;
}

.chart-container {
  width: 100%;
  height: 240px;
}
</style>
