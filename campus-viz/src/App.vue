<template>
  <div class="app-container">
    <Navbar />
    <div class="main-content">
      <LeftPanel />
      <MapContainer />
      <RightPanel />
    </div>
    <BottomBar />
  </div>
</template>

<script setup>
import { onMounted, onUnmounted } from 'vue'
import Navbar from './components/Navbar.vue'
import LeftPanel from './components/LeftPanel.vue'
import RightPanel from './components/RightPanel.vue'
import MapContainer from './components/MapContainer.vue'
import BottomBar from './components/BottomBar.vue'

const updateTime = () => {
  const now = new Date()
  const timeStr = now.toLocaleTimeString('zh-CN', { hour12: false })
  const dateStr = now.toLocaleDateString('zh-CN', { year: 'numeric', month: '2-digit', day: '2-digit' })
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
.app-container {
  width: 100vw;
  height: 100vh;
  background: linear-gradient(135deg, #0a0e27 0%, #111838 50%, #0a0e27 100%);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.main-content {
  flex: 1;
  display: flex;
  padding: 20px;
  gap: 20px;
  overflow: hidden;
}
</style>
