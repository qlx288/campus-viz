<template>
  <div class="map-container">
    <div id="scene3d" class="scene"></div>

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
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

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

let scene, camera, renderer, controls
let animationId

const typeColors = {
  teaching: 0x3b82f6,
  library: 0xec4899,
  dormitory: 0x06b6d4,
  lab: 0x8b5cf6,
  gym: 0x10b981
}

onMounted(() => {
  initScene()
  animate()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('resize', handleResize)
  if (renderer) {
    renderer.dispose()
  }
})

const initScene = () => {
  const container = document.getElementById('scene3d')
  if (!container) return

  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x0a0e27)

  camera = new THREE.PerspectiveCamera(
    60,
    container.clientWidth / container.clientHeight,
    0.1,
    1000
  )
  camera.position.set(50, 50, 50)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.clientWidth, container.clientHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  container.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.minDistance = 20
  controls.maxDistance = 150
  controls.maxPolarAngle = Math.PI / 2

  addLights()
  addGround()
  addBuildings()
  addTrees()
  addRoads()
}

const addLights = () => {
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.4)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
  directionalLight.position.set(50, 100, 50)
  directionalLight.castShadow = true
  directionalLight.shadow.mapSize.width = 2048
  directionalLight.shadow.mapSize.height = 2048
  directionalLight.shadow.camera.near = 0.5
  directionalLight.shadow.camera.far = 500
  directionalLight.shadow.camera.left = -100
  directionalLight.shadow.camera.right = 100
  directionalLight.shadow.camera.top = 100
  directionalLight.shadow.camera.bottom = -100
  scene.add(directionalLight)

  const pointLight = new THREE.PointLight(0x3b82f6, 0.5, 100)
  pointLight.position.set(0, 30, 0)
  scene.add(pointLight)
}

const addGround = () => {
  const groundGeometry = new THREE.PlaneGeometry(200, 200)
  const groundMaterial = new THREE.MeshStandardMaterial({
    color: 0x1a2332,
    roughness: 0.8,
    metalness: 0.2
  })
  const ground = new THREE.Mesh(groundGeometry, groundMaterial)
  ground.rotation.x = -Math.PI / 2
  ground.receiveShadow = true
  scene.add(ground)

  const gridHelper = new THREE.GridHelper(200, 50, 0x3b82f6, 0x1a2332)
  gridHelper.position.y = 0.01
  scene.add(gridHelper)
}

const createBuilding = (x, z, width, depth, height, color, name) => {
  const geometry = new THREE.BoxGeometry(width, height, depth)
  const material = new THREE.MeshStandardMaterial({
    color: color,
    roughness: 0.3,
    metalness: 0.5
  })
  const building = new THREE.Mesh(geometry, material)
  building.position.set(x, height / 2, z)
  building.castShadow = true
  building.receiveShadow = true
  building.userData = { name, type: 'building' }
  scene.add(building)
  return building
}

const addBuildings = () => {
  const buildings = [
    { x: 0, z: 0, w: 20, d: 15, h: 12, color: typeColors.teaching, name: '主教学楼' },
    { x: -30, z: 10, w: 18, d: 12, h: 10, color: typeColors.library, name: '图书馆' },
    { x: 30, z: 0, w: 16, d: 14, h: 8, color: typeColors.dormitory, name: '1号宿舍楼' },
    { x: 30, z: 25, w: 16, d: 14, h: 8, color: typeColors.dormitory, name: '2号宿舍楼' },
    { x: 30, z: -25, w: 16, d: 14, h: 8, color: typeColors.dormitory, name: '3号宿舍楼' },
    { x: -20, z: -30, w: 14, d: 10, h: 9, color: typeColors.lab, name: '实验楼' },
    { x: 0, z: -40, w: 22, d: 16, h: 6, color: typeColors.gym, name: '体育馆' },
    { x: -40, z: -15, w: 12, d: 10, h: 8, color: typeColors.teaching, name: '2号教学楼' },
    { x: -40, z: 25, w: 12, d: 10, h: 8, color: typeColors.teaching, name: '3号教学楼' },
    { x: 20, z: -60, w: 15, d: 12, h: 7, color: typeColors.library, name: '科技楼' },
  ]

  buildings.forEach(b => {
    createBuilding(b.x, b.z, b.w, b.d, b.h, b.color, b.name)
  })
}

const addTrees = () => {
  const treePositions = [
    [-45, 5], [-45, -5], [-50, 0], [-55, 10],
    [45, 15], [45, -15], [50, 0], [55, 5],
    [-10, -50], [10, -50], [0, -55], [-20, -45],
    [-25, 45], [25, 45], [20, 55], [-20, 55],
  ]

  treePositions.forEach(([x, z]) => {
    const trunkGeometry = new THREE.CylinderGeometry(0.5, 0.6, 3, 8)
    const trunkMaterial = new THREE.MeshStandardMaterial({ color: 0x4a3728 })
    const trunk = new THREE.Mesh(trunkGeometry, trunkMaterial)
    trunk.position.set(x, 1.5, z)
    trunk.castShadow = true
    scene.add(trunk)

    const foliageGeometry = new THREE.ConeGeometry(3, 6, 8)
    const foliageMaterial = new THREE.MeshStandardMaterial({
      color: 0x228B22,
      roughness: 0.8
    })
    const foliage = new THREE.Mesh(foliageGeometry, foliageMaterial)
    foliage.position.set(x, 5, z)
    foliage.castShadow = true
    scene.add(foliage)
  })
}

const addRoads = () => {
  const roadMaterial = new THREE.MeshStandardMaterial({
    color: 0x2c3e50,
    roughness: 0.9
  })

  const mainRoad = new THREE.Mesh(
    new THREE.PlaneGeometry(60, 6),
    roadMaterial
  )
  mainRoad.rotation.x = -Math.PI / 2
  mainRoad.position.set(0, 0.02, 0)
  scene.add(mainRoad)

  const sideRoad = new THREE.Mesh(
    new THREE.PlaneGeometry(6, 100),
    roadMaterial
  )
  sideRoad.rotation.x = -Math.PI / 2
  sideRoad.position.set(0, 0.02, 0)
  scene.add(sideRoad)

  const road2 = new THREE.Mesh(
    new THREE.PlaneGeometry(80, 5),
    roadMaterial
  )
  road2.rotation.x = -Math.PI / 2
  road2.rotation.z = Math.PI / 4
  road2.position.set(-20, 0.02, -20)
  scene.add(road2)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)
  controls.update()
  renderer.render(scene, camera)
}

const handleResize = () => {
  const container = document.getElementById('scene3d')
  if (!container) return

  camera.aspect = container.clientWidth / container.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(container.clientWidth, container.clientHeight)
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

.scene {
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
</style>
