<template>
  <div class="bubble-popup" v-if="show">
    <div class="bubble-header">
      <span class="bubble-title">{{ data.name }}</span>
      <button class="bubble-close" @click="$emit('close')">✕</button>
    </div>
    <div class="bubble-metrics">
      <div class="metric-box">
        <div class="metric-label">土壤湿度</div>
        <div class="metric-value">{{ data.moisture }}<span class="metric-unit">%</span></div>
      </div>
      <div class="metric-box">
        <div class="metric-label">温度</div>
        <div class="metric-value">{{ data.temp }}<span class="metric-unit">°C</span></div>
      </div>
      <div class="metric-box">
        <div class="metric-label">pH值</div>
        <div class="metric-value">{{ data.ph }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  show: Boolean,
  data: Object
})

defineEmits(['close'])
</script>

<style scoped>
.bubble-popup {
  position: absolute;
  width: 340px;
  background: rgba(255, 255, 255, 0.98);
  backdrop-filter: blur(10px);
  border: 1px solid #2ECC71;
  border-radius: 14px;
  padding: 0;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  z-index: 200;
  animation: bubbleIn 0.3s ease;
}

@keyframes bubbleIn {
  from { opacity: 0; transform: scale(0.9) translateY(10px); }
  to { opacity: 1; transform: scale(1) translateY(0); }
}

.bubble-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  background: linear-gradient(90deg, #E8F8F0 0%, transparent 100%);
  border-bottom: 1px solid #A9DFBF;
  border-radius: 14px 14px 0 0;
}

.bubble-title {
  font-size: 14px;
  font-weight: 700;
  color: #27AE60;
}

.bubble-close {
  width: 28px;
  height: 28px;
  border: 1px solid #E8E8E8;
  border-radius: 50%;
  background: white;
  color: #7F8C8D;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  transition: all 0.2s;
}

.bubble-close:hover {
  background: #E74C3C;
  border-color: #E74C3C;
  color: white;
}

.bubble-metrics {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  padding: 16px;
}

.metric-box {
  background: #F8F9FA;
  border: 1px solid #E8E8E8;
  border-radius: 10px;
  padding: 16px 12px;
  text-align: center;
  transition: all 0.2s;
}

.metric-box:hover {
  background: #E8F8F0;
  border-color: #2ECC71;
}

.metric-label {
  font-size: 11px;
  color: #7F8C8D;
  margin-bottom: 8px;
}

.metric-value {
  font-size: 24px;
  font-weight: 700;
  color: #2C3E50;
}

.metric-unit {
  font-size: 12px;
  color: #95A5A6;
  font-weight: 400;
  margin-left: 2px;
}
</style>
