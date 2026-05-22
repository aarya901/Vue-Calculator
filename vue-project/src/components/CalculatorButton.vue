<script setup>
defineProps({
  label: { type: String, required: true },
  type: { type: String, default: 'number' },
  id: { type: String, default: '' }
})

defineEmits(['click'])
</script>

<template>
  <button
    class="calc-btn"
    :class="[`btn-${type}`]"
    :id="id"
    @click="$emit('click')"
  >
    <span class="btn-label">{{ label }}</span>
    <span class="btn-ripple"></span>
  </button>
</template>

<style scoped>
.calc-btn {
  position: relative;
  border: none;
  border-radius: var(--radius-sm);
  font-family: var(--font-main);
  font-size: 20px;
  font-weight: 500;
  cursor: pointer;
  padding: 18px 10px;
  overflow: hidden;
  transition: all var(--transition-fast);
  display: flex;
  align-items: center;
  justify-content: center;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

.calc-btn:active {
  transform: scale(0.93);
}

.btn-label {
  position: relative;
  z-index: 1;
}

/* Number buttons */
.btn-number {
  background: var(--bg-button);
  color: var(--text-button);
  box-shadow: var(--shadow-button);
}

.btn-number:hover {
  background: var(--bg-button-hover);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
}

.btn-number:active {
  background: var(--bg-button-active);
  transform: scale(0.93) translateY(0);
}

/* Operator buttons */
.btn-operator {
  background: var(--bg-operator);
  color: var(--text-operator);
  box-shadow: var(--shadow-glow-purple), var(--shadow-button);
  font-size: 22px;
}

.btn-operator:hover {
  background: var(--bg-operator-hover);
  transform: translateY(-1px);
  box-shadow: 0 0 40px rgba(108, 92, 231, 0.25), 0 4px 12px rgba(0, 0, 0, 0.3);
}

.btn-operator:active {
  background: var(--bg-operator-active);
  transform: scale(0.93) translateY(0);
}

/* Equals button */
.btn-equals {
  background: var(--bg-equals);
  color: white;
  box-shadow: var(--shadow-glow-green), var(--shadow-button);
  font-size: 24px;
  font-weight: 600;
}

.btn-equals:hover {
  background: var(--bg-equals-hover);
  transform: translateY(-1px);
  box-shadow: 0 0 40px rgba(0, 184, 148, 0.25), 0 4px 12px rgba(0, 0, 0, 0.3);
}

.btn-equals:active {
  background: var(--bg-equals-active);
  transform: scale(0.93) translateY(0);
}

/* Clear button */
.btn-clear {
  background: var(--bg-clear);
  color: white;
  font-weight: 600;
  box-shadow: var(--shadow-button);
}

.btn-clear:hover {
  background: var(--bg-clear-hover);
  transform: translateY(-1px);
}

.btn-clear:active {
  background: var(--bg-clear-active);
  transform: scale(0.93) translateY(0);
}

/* Special buttons (CE, %, ±) */
.btn-special {
  background: var(--bg-special);
  color: var(--text-secondary);
  font-size: 18px;
}

.btn-special:hover {
  background: var(--bg-special-hover);
  color: var(--text-primary);
  transform: translateY(-1px);
}

.btn-special:active {
  transform: scale(0.93) translateY(0);
}
</style>
