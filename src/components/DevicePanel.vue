<template>
  <div class="panel">
    <div class="panel-title">{{ title }}</div>
    
    <div class="device-list">
      <div v-for="(device, index) in devices" :key="index" class="device-row">
        <span class="device-name">{{ device.name }}</span>
        <span class="device-status">
          <span class="status-dot" :class="device.status"></span>
          {{ device.text }}
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  title: String,
  devices: Array
})
</script>

<style scoped>
.panel {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.panel-title {
  font-size: 13px;
  font-weight: 700;
  color: #27AE60;
  padding-bottom: 8px;
  border-bottom: 2px solid #2ECC71;
  display: flex;
  align-items: center;
  gap: 8px;
  letter-spacing: 0.5px;
}

.device-list {
  background: #FFFFFF;
  border: 1px solid #E8E8E8;
  border-radius: 10px;
  padding: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}

.device-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 10px;
  border-bottom: 1px solid #F0F0F0;
  transition: background 0.2s;
}

.device-row:last-child {
  border-bottom: none;
}

.device-row:hover {
  background: #F8F9FA;
  border-radius: 6px;
}

.device-name {
  font-size: 12px;
  color: #2C3E50;
}

.device-status {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 11px;
  color: #7F8C8D;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.status-dot.online {
  background: #2ECC71;
  box-shadow: 0 0 6px rgba(46, 204, 113, 0.5);
}

.status-dot.warning {
  background: #E67E22;
  box-shadow: 0 0 6px rgba(230, 126, 34, 0.5);
}

.status-dot.offline {
  background: #E74C3C;
  box-shadow: 0 0 6px rgba(231, 76, 60, 0.5);
}
</style>
