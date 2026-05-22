<script setup>
defineProps({
  history: { type: Array, default: () => [] }
})

defineEmits(['use-result', 'clear-history', 'close'])

function formatTime(timestamp) {
  return new Date(timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
}
</script>

<template>
  <div class="history-panel" id="history-panel">
    <div class="history-header">
      <h3>History</h3>
      <div class="history-actions">
        <button class="history-clear-btn" id="btn-clear-history" @click="$emit('clear-history')" v-if="history.length">
          Clear
        </button>
        <button class="history-close-btn" id="btn-close-history" @click="$emit('close')">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"/>
            <line x1="6" y1="6" x2="18" y2="18"/>
          </svg>
        </button>
      </div>
    </div>

    <div class="history-list" v-if="history.length">
      <TransitionGroup name="history-item">
        <div
          v-for="(item, index) in history"
          :key="item.timestamp"
          class="history-entry"
          @click="$emit('use-result', item.result)"
          :id="'history-entry-' + index"
        >
          <div class="history-expression">{{ item.expression }}</div>
          <div class="history-result">= {{ item.result }}</div>
          <div class="history-time">{{ formatTime(item.timestamp) }}</div>
        </div>
      </TransitionGroup>
    </div>

    <div class="history-empty" v-else>
      <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" opacity="0.3">
        <circle cx="12" cy="12" r="10"/>
        <polyline points="12 6 12 12 16 14"/>
      </svg>
      <p>No history yet</p>
      <span>Your calculations will appear here</span>
    </div>
  </div>
</template>

<style scoped>
.history-panel {
  width: 280px;
  max-height: 560px;
  background: var(--bg-calculator);
  border-radius: var(--radius-xl);
  padding: 20px;
  box-shadow: var(--shadow-card);
  border: 1px solid var(--border-subtle);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border-subtle);
}

.history-header h3 {
  font-size: 14px;
  font-weight: 600;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.history-actions {
  display: flex;
  gap: 8px;
  align-items: center;
}

.history-clear-btn {
  background: transparent;
  border: none;
  color: var(--bg-clear);
  font-size: 12px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 6px;
  font-family: var(--font-main);
  transition: all var(--transition-fast);
}

.history-clear-btn:hover {
  background: rgba(225, 112, 85, 0.1);
}

.history-close-btn {
  background: transparent;
  border: none;
  color: var(--text-secondary);
  cursor: pointer;
  padding: 4px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  transition: all var(--transition-fast);
}

.history-close-btn:hover {
  background: var(--bg-button);
  color: var(--text-primary);
}

.history-list {
  overflow-y: auto;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding-right: 4px;
}

/* Custom scrollbar */
.history-list::-webkit-scrollbar {
  width: 4px;
}

.history-list::-webkit-scrollbar-track {
  background: transparent;
}

.history-list::-webkit-scrollbar-thumb {
  background: var(--bg-button-hover);
  border-radius: 4px;
}

.history-entry {
  padding: 12px;
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: all var(--transition-fast);
  position: relative;
}

.history-entry:hover {
  background: var(--bg-button);
}

.history-entry:active {
  transform: scale(0.98);
}

.history-expression {
  font-family: var(--font-mono);
  font-size: 13px;
  color: var(--text-secondary);
  margin-bottom: 4px;
}

.history-result {
  font-family: var(--font-mono);
  font-size: 18px;
  font-weight: 500;
  color: var(--text-primary);
}

.history-time {
  font-size: 10px;
  color: var(--text-secondary);
  opacity: 0.5;
  margin-top: 4px;
}

.history-empty {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 40px 0;
}

.history-empty p {
  font-size: 14px;
  color: var(--text-secondary);
  font-weight: 500;
}

.history-empty span {
  font-size: 12px;
  color: var(--text-secondary);
  opacity: 0.5;
}

/* List transition */
.history-item-enter-active {
  transition: all 0.3s ease;
}

.history-item-enter-from {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
