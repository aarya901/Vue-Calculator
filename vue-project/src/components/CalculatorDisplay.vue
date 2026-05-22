<script setup>
import { computed } from 'vue'

const props = defineProps({
  value: { type: String, default: '0' },
  expression: { type: String, default: '' },
  animate: { type: Boolean, default: false }
})

// Auto-size the font based on value length
const fontSize = computed(() => {
  const len = props.value.length
  if (len <= 8) return '48px'
  if (len <= 12) return '36px'
  if (len <= 16) return '28px'
  return '22px'
})
</script>

<template>
  <div class="display" id="calculator-display">
    <div class="display-expression" id="display-expression">
      {{ expression || '\u00A0' }}
    </div>
    <div
      class="display-value"
      id="display-value"
      :class="{ 'animate-pop': animate }"
      :style="{ fontSize }"
    >
      {{ value }}
    </div>
  </div>
</template>

<style scoped>
.display {
  background: var(--bg-display);
  border-radius: var(--radius-md);
  padding: 20px 24px;
  margin-bottom: 12px;
  min-height: 110px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: flex-end;
  border: 1px solid var(--border-subtle);
  position: relative;
  overflow: hidden;
}

/* Subtle gradient overlay on display */
.display::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 50%;
  background: linear-gradient(to top, rgba(108, 92, 231, 0.03), transparent);
  pointer-events: none;
}

.display-expression {
  font-size: 14px;
  color: var(--text-secondary);
  font-family: var(--font-mono);
  font-weight: 300;
  margin-bottom: 8px;
  min-height: 20px;
  text-align: right;
  width: 100%;
  letter-spacing: 1px;
}

.display-value {
  font-family: var(--font-mono);
  font-weight: 400;
  color: var(--text-display);
  text-align: right;
  width: 100%;
  line-height: 1.2;
  transition: font-size 0.2s ease;
  word-break: break-all;
  position: relative;
  z-index: 1;
}

/* Result pop animation */
.animate-pop {
  animation: popIn 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes popIn {
  0% {
    transform: scale(0.85);
    opacity: 0.6;
  }
  60% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style>
