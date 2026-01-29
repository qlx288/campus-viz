<template>
  <div class="map-container">
    <div id="cesiumContainer" class="cesium-container"></div>

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

    <div class="scene-info">
      <div class="info-item">
        <span class="info-icon">🎮</span>
        <span class="info-text">左键旋转 | 右键平移 | 滚轮缩放</span>
      </div>
      <div class="info-item">
        <span class="info-icon">📍</span>
        <span class="info-text">内蒙古师范大学 - 和林格尔校区</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as Cesium from 'cesium'

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

let viewer = null
let entities = []

const toggleLayer = (layerId) => {
  activeLayer.value = layerId
}

onMounted(() => {
  initCesium()
})

onUnmounted(() => {
  if (viewer) {
    viewer.destroy()
  }
})

const initCesium = () => {
  Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJlYWE1OWUxNy1mMWZiLTQzYjYtYTQ0OS1kMWFjYmFkNjc5YzciLCJpZCI6NTc3MzcsImlhdCI6MTYxMzQyMjczOH0.XcKpgANiY19MC4bdFUXMVEBToBmqS8kuYpUlxJHYZxk'

  viewer = new Cesium.Viewer('cesiumContainer', {
    terrainProvider: Cesium.createWorldTerrain(),
    animation: false,
    timeline: false,
    baseLayerPicker: false,
    geocoder: false,
    homeButton: false,
    sceneModePicker: false,
    navigationHelpButton: false,
    fullscreenButton: false,
    vrButton: false,
    infoBox: true,
    selectionIndicator: true,
    shadows: true,
    shouldAnimate: true
  })

  viewer.scene.globe.enableLighting = true
  viewer.scene.globe.depthTestAgainstTerrain = true

  const lat = 40.7562
  const lng = 111.8263
  const height = 500

  viewer.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(lng, lat, height),
    orientation: {
      heading: Cesium.Math.toRadians(0),
      pitch: Cesium.Math.toRadians(-45),
      roll: 0
    }
  })

  addCampusBuildings()
  addCampusLabels()
  addCampusArea()
}

const addCampusArea = () => {
  const positions = Cesium.Cartesian3.fromDegreesArray([
    111.8220, 40.7590,
    111.8300, 40.7590,
    111.8300, 40.7540,
    111.8220, 40.7540
  ])

  const entity = viewer.entities.add({
    name: '校园区域',
    polygon: {
      hierarchy: new Cesium.PolygonHierarchy(positions),
      material: Cesium.Color.BLUE.withAlpha(0.2),
      outline: true,
      outlineColor: Cesium.Color.BLUE,
      outlineWidth: 2,
      height: 0,
      perPositionHeight: false
    }
  })

  entities.push(entity)
}

const addCampusBuildings = () => {
  const buildings = [
    { name: '主教学楼', lng: 111.8263, lat: 40.7562, height: 25, color: '#3b82f6', type: 'teaching' },
    { name: '图书馆', lng: 111.8280, lat: 40.7575, height: 20, color: '#ec4899', type: 'library' },
    { name: '1号宿舍楼', lng: 111.8250, lat: 40.7550, height: 18, color: '#06b6d4', type: 'dormitory' },
    { name: '2号宿舍楼', lng: 111.8255, lat: 40.7560, height: 18, color: '#06b6d4', type: 'dormitory' },
    { name: '3号宿舍楼', lng: 111.8260, lat: 40.7570, height: 18, color: '#06b6d4', type: 'dormitory' },
    { name: '实验楼', lng: 111.8285, lat: 40.7555, height: 22, color: '#8b5cf6', type: 'lab' },
    { name: '体育馆', lng: 111.8240, lat: 40.7580, height: 15, color: '#10b981', type: 'gym' },
    { name: '2号教学楼', lng: 111.8235, lat: 40.7565, height: 20, color: '#3b82f6', type: 'teaching' },
    { name: '3号教学楼', lng: 111.8275, lat: 40.7570, height: 20, color: '#3b82f6', type: 'teaching' },
    { name: '科技楼', lng: 111.8290, lat: 40.7550, height: 18, color: '#f59e0b', type: 'tech' },
    { name: '行政楼', lng: 111.8270, lat: 40.7585, height: 16, color: '#f97316', type: 'admin' },
    { name: '食堂', lng: 111.8245, lat: 40.7535, height: 12, color: '#ef4444', type: 'dining' },
  ]

  buildings.forEach(building => {
    const entity = viewer.entities.add({
      name: building.name,
      position: Cesium.Cartesian3.fromDegrees(building.lng, building.lat, building.height / 2),
      box: {
        dimensions: new Cesium.Cartesian3(60.0, building.height, 45.0),
        material: Cesium.Color.fromCssColorString(building.color).withAlpha(0.8),
        outline: true,
        outlineColor: Cesium.Color.WHITE,
        outlineWidth: 2,
        shadows: Cesium.ShadowMode.ENABLED
      },
      description: `
        <div style="color: #333; padding: 15px; font-family: Arial, sans-serif;">
          <h3 style="margin: 0 0 10px 0; color: #3b82f6;">${building.name}</h3>
          <p style="margin: 5px 0;"><strong>类型：</strong>${getTypeName(building.type)}</p>
          <p style="margin: 5px 0;"><strong>高度：</strong>${building.height}米</p>
          <p style="margin: 5px 0;"><strong>坐标：</strong>${building.lat.toFixed(4)}, ${building.lng.toFixed(4)}</p>
        </div>
      `
    })

    entities.push(entity)

    const label = viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(building.lng, building.lat, building.height + 5),
      label: {
        text: building.name,
        font: '14px sans-serif',
        fillColor: Cesium.Color.WHITE,
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 2,
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
        pixelOffset: new Cesium.Cartesian2(0, -10),
        showBackground: true,
        backgroundColor: Cesium.Color.fromCssColorString(building.color).withAlpha(0.7)
      }
    })

    entities.push(label)
  })
}

const addCampusLabels = () => {
  const labels = [
    { name: '内蒙古师范大学', lng: 111.8263, lat: 40.7562, offsetY: 80 },
    { name: '和林格尔校区', lng: 111.8263, lat: 40.7562, offsetY: 60 },
    { name: '北门', lng: 111.8263, lat: 40.7590, offsetY: 10 },
    { name: '南门', lng: 111.8263, lat: 40.7540, offsetY: 10 },
  ]

  labels.forEach(label => {
    const entity = viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(label.lng, label.lat, label.offsetY),
      label: {
        text: label.name,
        font: 'bold 16px sans-serif',
        fillColor: Cesium.Color.WHITE,
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 3,
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
        horizontalOrigin: Cesium.HorizontalOrigin.CENTER,
        showBackground: true,
        backgroundColor: Cesium.Color.fromCssColorString('#1a2332').withAlpha(0.9)
      }
    })

    entities.push(entity)
  })
}

const getTypeName = (type) => {
  const types = {
    teaching: '教学楼',
    library: '图书馆',
    dormitory: '宿舍楼',
    lab: '实验楼',
    gym: '体育馆',
    tech: '科技楼',
    admin: '行政楼',
    dining: '食堂'
  }
  return types[type] || '其他'
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

.cesium-container {
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

.scene-info {
  position: absolute;
  bottom: 20px;
  right: 20px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  z-index: 1000;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  background: rgba(16, 24, 56, 0.95);
  border: 1px solid rgba(59, 130, 246, 0.3);
  border-radius: 10px;
  backdrop-filter: blur(10px);
}

.info-icon {
  font-size: 16px;
}

.info-text {
  font-size: 12px;
  color: #e2e8f0;
}

:deep(.cesium-widget-credits) {
  display: none !important;
}

:deep(.cesium-viewer-bottom) {
  display: none !important;
}
</style>
