<template>
  <div class="navbar">
    <div class="navbar-left">
      <div class="logo-section">
        <svg class="logo-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M12 2L2 7L12 12L22 7L12 2Z" fill="#3b82f6"/>
          <path d="M2 17L12 22L22 17" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M2 7V17" stroke="#3b82f6" stroke-width="2" stroke-linecap="round"/>
          <path d="M22 7V17" stroke="#3b82f6" stroke-width="2" stroke-linecap="round"/>
          <path d="M12 12V22" stroke="#3b82f6" stroke-width="2" stroke-linecap="round"/>
        </svg>
        <div class="logo-text">
          <h1 class="logo-title">内蒙古师范大学</h1>
          <p class="logo-subtitle">和林格尔校区 · 智慧校园可视化平台</p>
        </div>
      </div>
    </div>

    <div class="navbar-center">
      <div class="nav-items">
        <div v-for="item in navItems" :key="item.id" class="nav-item" :class="{ active: activeItem === item.id }" @click="handleNavClick(item.id)">
          <span class="nav-icon">{{ item.icon }}</span>
          <span class="nav-text">{{ item.name }}</span>
        </div>
      </div>
    </div>

    <div class="navbar-right">
      <div class="time-display">
        <div class="time">{{ currentTime }}</div>
        <div class="date">{{ currentDate }}</div>
      </div>
      <div class="weather-info">
        <span class="weather-icon">☀️</span>
        <span class="weather-text">12°C 晴</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const activeItem = ref('map')
const currentTime = ref('')
const currentDate = ref('')

const navItems = [
  { id: 'map', name: '校园地图', icon: '🗺️' },
  { id: 'overview', name: '总览分析', icon: '📊' },
  { id: 'security', name: '校园安防', icon: '🔒' },
  { id: 'energy', name: '能源管理', icon: '⚡' },
  { id: 'facility', name: '设施管理', icon: '🏢' },
]

const handleNavClick = (id) => {
  activeItem.value = id
}

const updateTime = () => {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('zh-CN', { hour12: false })
  currentDate.value = now.toLocaleDateString('zh-CN', { year: 'numeric', month: '2-digit', day: '2-digit', weekday: 'long' })
}

let timer

onMounted(() => {
  timer = setInterval(updateTime, 1000)
  updateTime()
})

onUnmounted(() => {
  clearInterval(timer)
})
</script>

<style scoped>
.navbar {
  height: 80px;
  background: linear-gradient(180deg, rgba(16, 24, 56, 0.95) 0%, rgba(22, 36, 71, 0.95) 100%);
  border-bottom: 2px solid rgba(59, 130, 246, 0.3);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 30px;
  position: relative;
  z-index: 100;
}

.navbar::before {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, #3b82f6, transparent);
  box-shadow: 0 0 20px rgba(59, 130, 246, 0.5);
}

.navbar-left {
  display: flex;
  align-items: center;
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 15px;
}

.logo-icon {
  width: 50px;
  height: 50px;
  filter: drop-shadow(0 0 10px rgba(59, 130, 246, 0.5));
}

.logo-text {
  display: flex;
  flex-direction: column;
}

.logo-title {
  font-size: 22px;
  font-weight: bold;
  color: #ffffff;
  margin: 0;
  text-shadow: 0 0 10px rgba(59, 130, 246, 0.5);
}

.logo-subtitle {
  font-size: 14px;
  color: #a0aec0;
  margin: 0;
}

.navbar-center {
  flex: 1;
  display: flex;
  justify-content: center;
}

.nav-items {
  display: flex;
  gap: 30px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 8px;
  transition: all 0.3s ease;
  position: relative;
}

.nav-item:hover {
  background: rgba(59, 130, 246, 0.15);
  transform: translateY(-2px);
}

.nav-item.active {
  background: rgba(59, 130, 246, 0.25);
  color: #3b82f6;
}

.nav-item.active::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80%;
  height: 2px;
  background: #3b82f6;
  box-shadow: 0 0 10px rgba(59, 130, 246, 0.8);
}

.nav-icon {
  font-size: 20px;
}

.nav-text {
  font-size: 16px;
  color: #e2e8f0;
  white-space: nowrap;
}

.navbar-right {
  display: flex;
  align-items: center;
  gap: 30px;
}

.time-display {
  text-align: right;
}

.time {
  font-size: 24px;
  font-weight: bold;
  color: #ffffff;
  text-shadow: 0 0 10px rgba(59, 130, 246, 0.5);
}

.date {
  font-size: 12px;
  color: #a0aec0;
  margin-top: 2px;
}

.weather-info {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: rgba(59, 130, 246, 0.1);
  border-radius: 20px;
  border: 1px solid rgba(59, 130, 246, 0.3);
}

.weather-icon {
  font-size: 20px;
}

.weather-text {
  font-size: 14px;
  color: #e2e8f0;
}
</style>
