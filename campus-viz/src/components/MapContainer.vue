<template>
  <div class="map-container">
    <div id="map" class="map"></div>

    <div class="map-overlay">
      <div class="map-controls">
        <div v-for="layer in layers" :key="layer.id" class="control-item" :class="{ active: activeLayer === layer.id }" @click="toggleLayer(layer.id)">
          <span class="control-icon">{{ layer.icon }}</span>
          <span class="control-label">{{ layer.name }}</span>
        </div>
      </div>

      <div class="map-legend">
        <div v-for="item in legend" :key="item.name" class="legend-item">
          <span class="legend-color" :style="{ background: item.color }"></span>
          <span class="legend-label">{{ item.name }}</span>
        </div>
      </div>
    </div>

    <div class="map-stats">
      <div class="map-stat-item">
        <div class="map-stat-value">{{ stats.buildings }}</div>
        <div class="map-stat-label">建筑</div>
      </div>
      <div class="map-stat-item">
        <div class="map-stat-value">{{ stats.students }}</div>
        <div class="map-stat-label">在校人数</div>
      </div>
      <div class="map-stat-item">
        <div class="map-stat-value">{{ stats.facilities }}</div>
        <div class="map-stat-label">设施点</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const map = ref(null)
const activeLayer = ref('buildings')
const stats = ref({
  buildings: 32,
  students: 25680,
  facilities: 156
})

const layers = [
  { id: 'buildings', name: '建筑', icon: '🏢' },
  { id: 'facilities', name: '设施', icon: '📍' },
  { id: 'traffic', name: '交通', icon: '🚗' },
  { id: 'security', name: '安防', icon: '🔒' },
]

const legend = [
  { name: '教学楼', color: '#3b82f6' },
  { name: '宿舍楼', color: '#06b6d4' },
  { name: '实验楼', color: '#8b5cf6' },
  { name: '图书馆', color: '#ec4899' },
  { name: '体育馆', color: '#10b981' },
]

const toggleLayer = (layerId) => {
  activeLayer.value = layerId
}

onMounted(() => {
  initMap()
})

onUnmounted(() => {
  if (map.value) {
    map.value.remove()
  }
})

const initMap = () => {
  const lat = 40.7562
  const lng = 111.8263
  const zoom = 15

  map.value = L.map('map', {
    center: [lat, lng],
    zoom: zoom,
    zoomControl: false,
    attributionControl: false
  })

  L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
    maxZoom: 19,
    subdomains: 'abcd'
  }).addTo(map.value)

  addMarkers()
  addPolygons()
}

const addMarkers = () => {
  const locations = [
    { lat: 40.7562, lng: 111.8263, name: '主教学楼', type: 'teaching' },
    { lat: 40.7575, lng: 111.8280, name: '图书馆', type: 'library' },
    { lat: 40.7550, lng: 111.8250, name: '1号宿舍楼', type: 'dormitory' },
    { lat: 40.7555, lng: 111.8285, name: '实验楼', type: 'lab' },
    { lat: 40.7580, lng: 111.8240, name: '体育馆', type: 'gym' },
  ]

  const typeColors = {
    teaching: '#3b82f6',
    library: '#ec4899',
    dormitory: '#06b6d4',
    lab: '#8b5cf6',
    gym: '#10b981'
  }

  locations.forEach(loc => {
    const color = typeColors[loc.type] || '#3b82f6'

    const icon = L.divIcon({
      className: 'custom-marker',
      html: `<div style="width: 30px; height: 30px; background: ${color}; border-radius: 50%; border: 3px solid white; box-shadow: 0 2px 8px rgba(0,0,0,0.3); display: flex; align-items: center; justify-content: center;">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="white">
          <path d="M12 2L2 7L12 12L22 7L12 2Z"/>
        </svg>
      </div>`,
      iconSize: [30, 30],
      iconAnchor: [15, 15]
    })

    const marker = L.marker([loc.lat, loc.lng], { icon })
      .addTo(map.value)

    marker.bindPopup(`<div style="color: #333; padding: 8px;"><strong>${loc.name}</strong></div>`)
  })
}

const addPolygons = () => {
  const campusArea = [
    [40.7590, 111.8220],
    [40.7590, 111.8300],
    [40.7540, 111.8300],
    [40.7540, 111.8220]
  ]

  L.polygon(campusArea, {
    color: '#3b82f6',
    weight: 2,
    fillColor: '#3b82f6',
    fillOpacity: 0.1,
    dashArray: '5, 10'
  }).addTo(map.value)
}
</script>

<style scoped>
.map-container {
  flex: 1;
  position: relative;
  background: #0a0e27;
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid rgba(59, 130, 246, 0.2);
}

.map {
  width: 100%;
  height: 100%;
}

.map-overlay {
  position: absolute;
  top: 20px;
  right: 20px;
  display: flex;
  flex-direction: column;
  gap: 15px;
  z-index: 1000;
}

.map-controls {
  display: flex;
  flex-direction: column;
  gap: 8px;
  background: rgba(16, 24, 56, 0.95);
  border: 1px solid rgba(59, 130, 246, 0.3);
  border-radius: 10px;
  padding: 12px;
  backdrop-filter: blur(10px);
}

.control-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
  background: rgba(59, 130, 246, 0.05);
  border: 1px solid rgba(59, 130, 246, 0.15);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.control-item:hover {
  background: rgba(59, 130, 246, 0.15);
  transform: translateX(-4px);
}

.control-item.active {
  background: rgba(59, 130, 246, 0.25);
  border-color: rgba(59, 130, 246, 0.5);
  box-shadow: 0 0 15px rgba(59, 130, 246, 0.3);
}

.control-icon {
  font-size: 18px;
}

.control-label {
  font-size: 13px;
  color: #e2e8f0;
  white-space: nowrap;
}

.map-legend {
  background: rgba(16, 24, 56, 0.95);
  border: 1px solid rgba(59, 130, 246, 0.3);
  border-radius: 10px;
  padding: 12px;
  backdrop-filter: blur(10px);
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 0;
}

.legend-color {
  width: 16px;
  height: 16px;
  border-radius: 4px;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.legend-label {
  font-size: 12px;
  color: #e2e8f0;
}

.map-stats {
  position: absolute;
  bottom: 20px;
  left: 20px;
  display: flex;
  gap: 15px;
  z-index: 1000;
}

.map-stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 12px 20px;
  background: rgba(16, 24, 56, 0.95);
  border: 1px solid rgba(59, 130, 246, 0.3);
  border-radius: 10px;
  backdrop-filter: blur(10px);
  min-width: 100px;
}

.map-stat-value {
  font-size: 20px;
  font-weight: bold;
  color: #3b82f6;
  text-shadow: 0 0 10px rgba(59, 130, 246, 0.5);
}

.map-stat-label {
  font-size: 11px;
  color: #a0aec0;
}

:deep(.custom-marker) {
  background: transparent !important;
  border: none !important;
}

:deep(.leaflet-popup-content-wrapper) {
  background: rgba(22, 36, 71, 0.95);
  border: 1px solid rgba(59, 130, 246, 0.3);
  border-radius: 8px;
  color: #ffffff;
}

:deep(.leaflet-popup-tip) {
  background: rgba(22, 36, 71, 0.95);
  border: 1px solid rgba(59, 130, 246, 0.3);
}
</style>
