<template>
  <a-drawer
    :open="open"
    title="账号与设置"
    placement="right"
    :width="drawerWidth"
    destroy-on-close
    @close="emit('update:open', false)"
  >
    <!-- 连接状态:移动端没有桌面右上角状态点,统一在这里展示 -->
    <div class="connection-status" :class="health.status.value">
      <span class="status-dot" />
      <span class="status-text">{{ statusText }}</span>
      <a-button v-if="health.status.value === 'down'" size="small" @click="health.refresh()">重试</a-button>
    </div>

    <AccountPanel @session-change="emit('session-change')" />
    <ConnectionPanel />
    <ThemePanel />
    <p class="tips">
      大模型调用相关的 API Key、API URL、模型名称等配置由服务端统一管理。
    </p>
  </a-drawer>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import AccountPanel from './settings/AccountPanel.vue';
import ConnectionPanel from './settings/ConnectionPanel.vue';
import ThemePanel from './settings/ThemePanel.vue';
import { useBackendHealth } from '../composables/useBackendHealth';
import { getServerBase } from '../utils/auth';

defineProps<{ open: boolean }>();
const emit = defineEmits<{
  (e: 'update:open', value: boolean): void;
  (e: 'session-change'): void;
}>();

const drawerWidth = computed(() =>
  Math.min(420, typeof window !== 'undefined' ? window.innerWidth - 24 : 420)
);

const health = useBackendHealth();
const statusText = computed(() => {
  if (health.status.value === 'ok') {
    const base = getServerBase();
    return base ? `服务端已连接(${base.replace(/^https?:\/\//, '')})` : '服务端已连接';
  }
  if (health.status.value === 'down') return '服务端离线,请检查服务端地址';
  return '正在检测服务端…';
});
</script>

<style scoped>
.connection-status {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  border: 1px solid var(--border-color);
  border-radius: 10px;
  background: var(--inset-bg);
  margin-bottom: 16px;
  font-size: 13px;
  color: var(--text-color);
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--text-muted);
  flex-shrink: 0;
}

.connection-status.ok .status-dot {
  background: #52c41a;
}

.connection-status.down .status-dot {
  background: #ff4d4f;
}

.status-text {
  flex: 1;
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.tips {
  font-size: 13px;
  background: rgba(60, 80, 120, 0.08);
  padding: 12px;
  border-radius: 8px;
  color: var(--text-muted);
  line-height: 1.5;
  text-align: justify;
  border: 1px solid var(--border-color);
  margin: 0;
}
</style>
