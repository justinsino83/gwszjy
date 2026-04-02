<template>
  <div class="app-container">
    <!-- 顶部栏 -->
    <header class="top-bar">
      <div class="logo">
        <div class="logo-icon">
          <svg viewBox="0 0 100 100" width="36" height="36">
            <circle cx="50" cy="50" r="48" fill="#2ECC71"/>
            <path d="M50 20 Q30 40 30 60 Q30 80 50 85 Q70 80 70 60 Q70 40 50 20" 
                  fill="none" stroke="#E8F8F0" stroke-width="4" stroke-linecap="round"/>
            <line x1="50" y1="30" x2="50" y2="75" stroke="#E8F8F0" stroke-width="3" stroke-linecap="round"/>
            <line x1="50" y1="45" x2="38" y2="55" stroke="#E8F8F0" stroke-width="2" stroke-linecap="round"/>
            <line x1="50" y1="55" x2="62" y2="50" stroke="#E8F8F0" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </div>
        <a href="https://e6lemviix2yqm.ok.kimi.link/#" target="_blank" class="logo-link">绿电数字农业平台</a>
      </div>
      <div class="top-center">
        <div class="time-display">{{ currentTime }}</div>
        <div class="date-display">
          <div>{{ currentDate }}</div>
          <div class="week">{{ currentWeek }}</div>
        </div>
      </div>
      <div class="top-info">
        <div class="info-pill">
          <span class="status-dot"></span>
          <span>系统在线</span>
        </div>
        <div class="info-pill">
          <span>📍</span>
          <span>江苏省泰州市</span>
        </div>
        <div class="info-pill">
          <span>🌤</span>
          <span>晴 26°C</span>
        </div>
      </div>
    </header>

    <!-- 主内容区 - 全屏GIS地图 -->
    <main class="main-scene">
      <SceneHeader
        :mode="currentMode"
        :location="currentLocation"
        :coords="currentCoords"
      />
      
      <!-- 左侧数据抽屉 -->
      <aside class="drawer-panel left-drawer" :class="{ collapsed: !leftDrawerOpen }">
        <DataPanel title="🌡 环境监测" :data="envData" />
        <DataPanel title="🌱 土壤监测" :data="soilData" />
      </aside>
      
      <!-- 左侧抽屉开关 -->
      <button class="drawer-toggle left-toggle" @click="leftDrawerOpen = !leftDrawerOpen">
        <span>{{ leftDrawerOpen ? '◀' : '▶' }}</span>
      </button>

      <!-- 右侧数据抽屉 -->
      <aside class="drawer-panel right-drawer" :class="{ collapsed: !rightDrawerOpen }">
        <DevicePanel title="⚙️ 设备状态" :devices="devices" />
        <DataPanel title="📊 产量统计" :data="productionData" />
      </aside>
      
      <!-- 右侧抽屉开关 -->
      <button class="drawer-toggle right-toggle" @click="rightDrawerOpen = !rightDrawerOpen">
        <span>{{ rightDrawerOpen ? '▶' : '◀' }}</span>
      </button>

      <!-- 控制按钮区域 -->
      <div class="control-bar">
        <button class="control-btn" @click="toggleAllDrawers">
          <span>{{ allDrawersOpen ? '📊 收起数据' : '📊 展开数据' }}</span>
        </button>
        <label class="toggle-label">
          <input type="checkbox" v-model="showThreeJS" />
          <span class="toggle-text">🌐 3D</span>
        </label>
      </div>

      <div class="scene-viewport">
        <CesiumMap ref="cesiumMap" @mapClick="handleMapClick" />
        <ThreeScene
          v-if="showThreeJS"
          ref="threeScene"
          :currentMode="currentMode"
          @popup="handlePopup"
          @flyToBuilding="handleFlyToBuilding"
        />
        <BubblePopup
          v-if="showPopup"
          :show="showPopup"
          :data="popupData"
          :style="popupStyle"
          @close="showPopup = false"
        />
      </div>
    </main>

    <!-- 底部菜单 -->
    <nav class="bottom-menu">
      <button
        v-for="menu in menus"
        :key="menu.id"
        :class="['menu-btn', { active: currentMode === menu.id }]"
        @click="switchMode(menu.id)"
      >
        <span class="menu-icon">{{ menu.icon }}</span>
        <span>{{ menu.name }}</span>
      </button>
    </nav>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue'
import CesiumMap from './components/CesiumMap.vue'
import ThreeScene from './components/ThreeScene.vue'
import DataPanel from './components/DataPanel.vue'
import DevicePanel from './components/DevicePanel.vue'
import SceneHeader from './components/SceneHeader.vue'
import BubblePopup from './components/BubblePopup.vue'

const cesiumMap = ref(null)
const threeScene = ref(null)
const currentMode = ref('overview')
const showThreeJS = ref(false)
const showPopup = ref(false)
const popupData = ref({})
const popupStyle = ref({})
const leftDrawerOpen = ref(true)
const rightDrawerOpen = ref(true)
const allDrawersOpen = ref(true)

function toggleAllDrawers() {
  allDrawersOpen.value = !allDrawersOpen.value
  leftDrawerOpen.value = allDrawersOpen.value
  rightDrawerOpen.value = allDrawersOpen.value
}

const currentTime = ref('00:00:00')
const currentDate = ref('2026-04-02')
const currentWeek = ref('星期四')

const menus = [
  { id: 'overview', name: '总览', icon: '🌍' },
  { id: 'testfield', name: '试验田', icon: '🌱' },
  { id: 'workshop', name: '烘干车间', icon: '🏭' },
  { id: 'warehouse', name: '仓库', icon: '📦' }
]

const currentLocation = ref('泰兴市新街镇叶垛家利')
const currentCoords = ref('32.18°N, 120.07°E')

const envData = reactive([
  { label: '土壤温度', value: '25.3', unit: '°C', status: '正常', chart: [60, 75, 65, 80, 70, 85, 78] },
  { label: '空气湿度', value: '68', unit: '%', status: '正常', chart: [55, 60, 58, 65, 70, 68, 72] },
  { label: '光照强度', value: '45800', unit: 'lux', status: '充足', chart: [90, 85, 92, 88, 95, 91, 93] }
])

const soilData = reactive([
  { label: '土壤 pH 值', value: '6.8', status: '6.8' },
  { label: '氮 N', value: '45', unit: '' },
  { label: '磷 P', value: '32', unit: '' },
  { label: '钾 K', value: '180', unit: '' }
])

const devices = reactive([
  { name: '🌾 智能灌溉系统', status: 'online', text: '运行中' },
  { name: '🌡 通风控制', status: 'online', text: '运行中' },
  { name: '💡 补光灯控', status: 'warning', text: '节能' },
  { name: '🔥 烘干机组 #1', status: 'online', text: '运行中' },
  { name: '🔥 烘干机组 #2', status: 'offline', text: '停机' }
])

const productionData = reactive([
  { label: '今日采收', value: '1,280', unit: 'kg' },
  { label: '本周累计', value: '8,450', unit: 'kg', chart: [45, 55, 50, 70, 65, 80, 60] },
  { label: '本月累计', value: '32,580', unit: 'kg' },
  { label: '年度目标', value: '68', unit: '%', status: '68%' }
])

function switchMode(mode) {
  currentMode.value = mode
  showPopup.value = false

  const locations = {
    overview: { loc: '泰兴市新街镇叶垛家利', coords: '32.18°N, 120.07°E' },
    testfield: { loc: '泰兴市新街镇试验田', coords: '点击地图获取坐标' },
    workshop: { loc: '根蔡线附近', coords: '32.17°N, 120.08°E' },
    warehouse: { loc: '根蔡线附近', coords: '32.17°N, 120.08°E' }
  }

  currentLocation.value = locations[mode].loc
  currentCoords.value = locations[mode].coords
  
  // 总览模式默认隐藏3D模型，其他模式显示
  showThreeJS.value = mode !== 'overview'

  if (cesiumMap.value) {
    cesiumMap.value.flyTo(mode)
  }
}

function handleMapClick({ lon, lat }) {
  if (currentMode.value === 'testfield') {
    currentCoords.value = `${lat}°N, ${lon}°E`
  }
}

function handlePopup(data) {
  popupData.value = {
    name: data.name,
    moisture: data.moisture,
    temp: data.temp,
    ph: data.ph
  }
  showPopup.value = true

  // Position popup in center-top area of viewport
  popupStyle.value = {
    position: 'absolute',
    top: '80px',
    left: '50%',
    transform: 'translateX(-50%)',
    zIndex: 200
  }
}

function handleFlyToBuilding({ mode, buildingId }) {
  if (threeScene.value) {
    threeScene.value.flyToBuilding(mode, buildingId)
  }

  // Also fly Cesium camera
  if (cesiumMap.value) {
    cesiumMap.value.flyTo(mode)
  }
}

function updateTime() {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false })
  currentDate.value = now.toLocaleDateString('zh-CN', { year: 'numeric', month: '2-digit', day: '2-digit' })
  const weeks = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六']
  currentWeek.value = weeks[now.getDay()]
}

let timeInterval = null

onMounted(() => {
  updateTime()
  timeInterval = setInterval(updateTime, 1000)
  // Fly to initial location
  setTimeout(() => {
    if (cesiumMap.value) {
      cesiumMap.value.flyTo('overview')
    }
  }, 500)
})

onUnmounted(() => {
  if (timeInterval) clearInterval(timeInterval)
})
</script>

<style>
/* 全局样式 - 绿电数字农业平台 清新风格 */
:root {
  --primary-green: #2ECC71;
  --dark-green: #27AE60;
  --light-green: #E8F8F0;
  --accent-orange: #E67E22;
  --accent-red: #E74C3C;
  --bg-light: #F0F4F8;
  --bg-white: #FFFFFF;
  --text-dark: #2C3E50;
  --text-gray: #7F8C8D;
  --shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  --shadow-hover: 0 4px 20px rgba(0, 0, 0, 0.1);
}

* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: -apple-system, 'Microsoft YaHei', 'PingFang SC', sans-serif;
  background: #000;
  color: var(--text-dark);
  overflow: hidden;
}

.app-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

/* 顶部栏 - 半透明悬浮 */
.top-bar {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(46, 204, 113, 0.3);
  z-index: 100;
}

.logo { display: flex; align-items: center; gap: 12px; }
.logo-icon {
  width: 36px; height: 36px;
  border-radius: 8px;
  display: flex; align-items: center; justify-content: center;
  overflow: hidden;
}
.logo-link {
  font-size: 16px; font-weight: 700;
  color: var(--dark-green);
  letter-spacing: 1px;
  text-decoration: none;
}
.logo-link:hover {
  color: var(--primary-green);
}

.top-center { display: flex; align-items: center; gap: 16px; }
.time-display {
  font-size: 20px; font-weight: 700;
  color: var(--dark-green);
  font-family: 'Roboto Mono', 'Courier New', monospace;
}
.date-display { font-size: 11px; color: var(--text-gray); }
.week { margin-top: 1px; }

.top-info { display: flex; gap: 12px; }
.info-pill {
  display: flex; align-items: center; gap: 5px;
  background: var(--light-green);
  border-radius: 14px; padding: 5px 10px;
  font-size: 11px; color: var(--dark-green);
}
.status-dot {
  width: 6px; height: 6px; border-radius: 50%;
  background: var(--primary-green);
  animation: pulse 1.5s ease-in-out infinite;
}
@keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.6; } }

/* 主场景 - 全屏地图 */
.main-scene {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
.scene-viewport {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

/* 抽屉面板 - 半透明悬浮 */
.drawer-panel {
  position: absolute;
  top: 66px;
  bottom: 70px;
  width: 270px;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  overflow-y: auto;
  z-index: 50;
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

.left-drawer {
  left: 0;
  border-right: 1px solid rgba(46, 204, 113, 0.3);
  border-radius: 0 12px 12px 0;
}

.right-drawer {
  right: 0;
  border-left: 1px solid rgba(46, 204, 113, 0.3);
  border-radius: 12px 0 0 12px;
}

.drawer-panel.collapsed {
  width: 0;
  padding: 0;
  opacity: 0;
  overflow: hidden;
}

/* 抽屉开关按钮 */
.drawer-toggle {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 60;
  width: 22px;
  height: 45px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(10px);
  border: 1px solid var(--primary-green);
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  color: var(--primary-green);
  transition: all 0.3s;
}

.drawer-toggle:hover {
  background: var(--primary-green);
  color: white;
}

.left-toggle {
  left: 280px;
  border-left: none;
  border-radius: 0 6px 6px 0;
}

.right-toggle {
  right: 280px;
  border-right: none;
  border-radius: 6px 0 0 6px;
}

/* 抽屉收起时按钮位置跟随 */
.left-drawer.collapsed ~ .left-toggle {
  left: 0;
}

.right-drawer.collapsed ~ .right-toggle {
  right: 0;
}

/* 控制按钮区域 - 半透明悬浮 */
.control-bar {
  position: absolute;
  top: 66px;
  right: 10px;
  z-index: 100;
  display: flex;
  gap: 6px;
  align-items: center;
}

.control-btn {
  display: flex;
  align-items: center;
  gap: 5px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(10px);
  border: 1px solid var(--primary-green);
  border-radius: 18px;
  padding: 6px 12px;
  cursor: pointer;
  font-size: 11px;
  color: var(--dark-green);
  font-weight: 500;
  transition: all 0.3s;
}

.control-btn:hover {
  background: var(--primary-green);
  color: white;
}

.toggle-label {
  display: flex;
  align-items: center;
  gap: 5px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(10px);
  border: 1px solid var(--primary-green);
  border-radius: 18px;
  padding: 6px 12px;
  cursor: pointer;
  transition: all 0.3s;
}
.toggle-label input[type="checkbox"] {
  width: 12px;
  height: 12px;
  accent-color: var(--primary-green);
  cursor: pointer;
}
.toggle-text {
  font-size: 10px;
  color: var(--dark-green);
  white-space: nowrap;
  font-weight: 500;
}

/* 底部菜单 - 半透明悬浮 */
.bottom-menu {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  display: flex; align-items: center;
  gap: 10px;
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(46, 204, 113, 0.3);
  border-radius: 25px;
  z-index: 100;
}
.menu-btn {
  display: flex; align-items: center; gap: 6px;
  padding: 10px 20px;
  background: var(--bg-white);
  border: 1px solid #E8E8E8;
  border-radius: 20px; color: var(--text-gray);
  font-size: 13px; cursor: pointer;
  transition: all 0.3s;
}
.menu-btn:hover {
  border-color: var(--primary-green);
  color: var(--primary-green);
  background: var(--light-green);
}
.menu-btn.active {
  border-color: var(--primary-green);
  color: white;
  background: var(--primary-green);
  box-shadow: 0 4px 15px rgba(46, 204, 113, 0.4);
}
.menu-icon { font-size: 14px; }
</style>
