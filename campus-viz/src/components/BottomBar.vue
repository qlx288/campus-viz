<template>
  <div class="bottom-bar">
    <div class="bottom-left">
      <div class="status-item">
        <span class="status-icon">🟢</span>
        <span class="status-text">系统运行正常</span>
      </div>
      <div class="status-item">
        <span class="status-icon">📡</span>
        <span class="status-text">网络连接良好</span>
      </div>
      <div class="status-item">
        <span class="status-icon">🔄</span>
        <span class="status-text">数据实时更新</span>
      </div>
    </div>

    <div class="bottom-center">
      <div class="scrolling-text">
        <div class="scrolling-content">
          <span v-for="(item, index) in scrollItems" :key="index" class="scroll-item">{{ item }}</span>
        </div>
      </div>
    </div>

    <div class="bottom-right">
      <div class="contact-info">
        <span class="contact-icon">📞</span>
        <span class="contact-text">技术服务：0471-12345678</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const scrollItems = ref([
  '热烈欢迎2024级新生入学！',
  '图书馆24小时自习区已开放',
  '校园卡充值点：食堂一楼、学生服务中心',
  '本周六举办校园开放日活动',
  '心理健康中心预约电话：0471-1234567',
  '校园公交时刻表已更新',
  '食堂新增健康套餐窗口',
  '实验室安全培训通知',
])
</script>

<style scoped>
.bottom-bar {
  height: 50px;
  background: linear-gradient(90deg, rgba(16, 24, 56, 0.95) 0%, rgba(22, 36, 71, 0.95) 100%);
  border-top: 1px solid rgba(59, 130, 246, 0.3);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 30px;
  position: relative;
  z-index: 100;
}

.bottom-bar::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, #3b82f6, transparent);
  box-shadow: 0 -1px 10px rgba(59, 130, 246, 0.3);
}

.bottom-left {
  display: flex;
  align-items: center;
  gap: 25px;
}

.status-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.status-icon {
  font-size: 14px;
}

.status-text {
  font-size: 13px;
  color: #a0aec0;
}

.bottom-center {
  flex: 1;
  display: flex;
  justify-content: center;
  padding: 0 30px;
  overflow: hidden;
}

.scrolling-text {
  width: 100%;
  overflow: hidden;
  white-space: nowrap;
}

.scrolling-content {
  display: inline-block;
  animation: scroll 30s linear infinite;
}

@keyframes scroll {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-50%);
  }
}

.scroll-item {
  font-size: 13px;
  color: #e2e8f0;
  margin-right: 50px;
}

.bottom-right {
  display: flex;
  align-items: center;
}

.contact-info {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 16px;
  background: rgba(59, 130, 246, 0.1);
  border-radius: 20px;
  border: 1px solid rgba(59, 130, 246, 0.2);
}

.contact-icon {
  font-size: 16px;
}

.contact-text {
  font-size: 13px;
  color: #e2e8f0;
}
</style>
