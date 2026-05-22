<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import CalculatorDisplay from './CalculatorDisplay.vue'
import CalculatorButton from './CalculatorButton.vue'
import CalculatorHistory from './CalculatorHistory.vue'

// State
const currentValue = ref('0')
const previousValue = ref('')
const operator = ref(null)
const waitingForOperand = ref(false)
const expression = ref('')
const history = ref([])
const showHistory = ref(false)
const animateResult = ref(false)

// Max display digits
const MAX_DIGITS = 15

// Computed
const displayValue = computed(() => {
  const num = parseFloat(currentValue.value)
  if (isNaN(num)) return currentValue.value
  if (currentValue.value.endsWith('.')) return currentValue.value
  if (currentValue.value.includes('.') && currentValue.value.endsWith('0')) return currentValue.value
  if (Math.abs(num) >= 1e15) return num.toExponential(6)
  return currentValue.value
})

const displayExpression = computed(() => {
  return expression.value
})

// Methods
function inputDigit(digit) {
  if (waitingForOperand.value) {
    currentValue.value = String(digit)
    waitingForOperand.value = false
  } else {
    if (currentValue.value.replace(/[^0-9]/g, '').length >= MAX_DIGITS) return
    currentValue.value = currentValue.value === '0' ? String(digit) : currentValue.value + digit
  }
}

function inputDecimal() {
  if (waitingForOperand.value) {
    currentValue.value = '0.'
    waitingForOperand.value = false
    return
  }
  if (!currentValue.value.includes('.')) {
    currentValue.value += '.'
  }
}

function handleOperator(nextOperator) {
  const inputValue = parseFloat(currentValue.value)

  if (operator.value && !waitingForOperand.value) {
    const result = performCalculation(parseFloat(previousValue.value), inputValue, operator.value)
    const resultStr = formatResult(result)
    currentValue.value = resultStr
    expression.value = resultStr + ' ' + getOperatorSymbol(nextOperator)
    previousValue.value = resultStr
  } else {
    previousValue.value = currentValue.value
    expression.value = currentValue.value + ' ' + getOperatorSymbol(nextOperator)
  }

  operator.value = nextOperator
  waitingForOperand.value = true
}

function getOperatorSymbol(op) {
  const symbols = { '+': '+', '-': '−', '*': '×', '/': '÷' }
  return symbols[op] || op
}

function performCalculation(left, right, op) {
  switch (op) {
    case '+': return left + right
    case '-': return left - right
    case '*': return left * right
    case '/': return right !== 0 ? left / right : 'Error'
    default: return right
  }
}

function formatResult(result) {
  if (result === 'Error') return 'Error'
  if (!isFinite(result)) return 'Error'
  if (Number.isInteger(result)) return String(result)
  // Limit decimal places
  const str = parseFloat(result.toPrecision(12)).toString()
  return str
}

function calculate() {
  if (!operator.value) return
  const inputValue = parseFloat(currentValue.value)
  const prevValue = parseFloat(previousValue.value)
  const result = performCalculation(prevValue, inputValue, operator.value)
  const resultStr = formatResult(result)

  // Build history entry
  const historyEntry = {
    expression: previousValue.value + ' ' + getOperatorSymbol(operator.value) + ' ' + currentValue.value,
    result: resultStr,
    timestamp: Date.now()
  }
  history.value.unshift(historyEntry)
  if (history.value.length > 20) history.value.pop()

  expression.value = ''
  currentValue.value = resultStr
  previousValue.value = ''
  operator.value = null
  waitingForOperand.value = true

  // Trigger result animation
  animateResult.value = true
  setTimeout(() => { animateResult.value = false }, 400)
}

function clear() {
  currentValue.value = '0'
  previousValue.value = ''
  operator.value = null
  waitingForOperand.value = false
  expression.value = ''
}

function clearEntry() {
  currentValue.value = '0'
  waitingForOperand.value = false
}

function toggleSign() {
  if (currentValue.value === '0' || currentValue.value === 'Error') return
  currentValue.value = currentValue.value.startsWith('-')
    ? currentValue.value.slice(1)
    : '-' + currentValue.value
}

function percentage() {
  const val = parseFloat(currentValue.value)
  if (isNaN(val)) return
  currentValue.value = formatResult(val / 100)
}

function backspace() {
  if (waitingForOperand.value || currentValue.value === 'Error') return
  if (currentValue.value.length === 1 || (currentValue.value.length === 2 && currentValue.value.startsWith('-'))) {
    currentValue.value = '0'
  } else {
    currentValue.value = currentValue.value.slice(0, -1)
  }
}

function toggleHistory() {
  showHistory.value = !showHistory.value
}

function clearHistory() {
  history.value = []
}

function useHistoryResult(result) {
  currentValue.value = result
  waitingForOperand.value = false
  showHistory.value = false
}

// Keyboard support
function handleKeyDown(e) {
  if (e.key >= '0' && e.key <= '9') {
    e.preventDefault()
    inputDigit(parseInt(e.key))
  } else if (e.key === '.') {
    e.preventDefault()
    inputDecimal()
  } else if (e.key === '+') {
    e.preventDefault()
    handleOperator('+')
  } else if (e.key === '-') {
    e.preventDefault()
    handleOperator('-')
  } else if (e.key === '*') {
    e.preventDefault()
    handleOperator('*')
  } else if (e.key === '/') {
    e.preventDefault()
    handleOperator('/')
  } else if (e.key === 'Enter' || e.key === '=') {
    e.preventDefault()
    calculate()
  } else if (e.key === 'Escape') {
    e.preventDefault()
    clear()
  } else if (e.key === 'Backspace') {
    e.preventDefault()
    backspace()
  } else if (e.key === '%') {
    e.preventDefault()
    percentage()
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
})

// Button layout
const buttons = [
  { label: 'AC', action: clear, type: 'clear', id: 'btn-clear' },
  { label: 'CE', action: clearEntry, type: 'special', id: 'btn-clear-entry' },
  { label: '%', action: percentage, type: 'special', id: 'btn-percent' },
  { label: '÷', action: () => handleOperator('/'), type: 'operator', id: 'btn-divide' },

  { label: '7', action: () => inputDigit(7), type: 'number', id: 'btn-7' },
  { label: '8', action: () => inputDigit(8), type: 'number', id: 'btn-8' },
  { label: '9', action: () => inputDigit(9), type: 'number', id: 'btn-9' },
  { label: '×', action: () => handleOperator('*'), type: 'operator', id: 'btn-multiply' },

  { label: '4', action: () => inputDigit(4), type: 'number', id: 'btn-4' },
  { label: '5', action: () => inputDigit(5), type: 'number', id: 'btn-5' },
  { label: '6', action: () => inputDigit(6), type: 'number', id: 'btn-6' },
  { label: '−', action: () => handleOperator('-'), type: 'operator', id: 'btn-subtract' },

  { label: '1', action: () => inputDigit(1), type: 'number', id: 'btn-1' },
  { label: '2', action: () => inputDigit(2), type: 'number', id: 'btn-2' },
  { label: '3', action: () => inputDigit(3), type: 'number', id: 'btn-3' },
  { label: '+', action: () => handleOperator('+'), type: 'operator', id: 'btn-add' },

  { label: '±', action: toggleSign, type: 'special', id: 'btn-sign' },
  { label: '0', action: () => inputDigit(0), type: 'number', id: 'btn-0' },
  { label: '.', action: inputDecimal, type: 'number', id: 'btn-decimal' },
  { label: '=', action: calculate, type: 'equals', id: 'btn-equals' },
]
</script>

<template>
  <div class="calculator-wrapper">
    <!-- History panel -->
    <Transition name="slide">
      <CalculatorHistory
        v-if="showHistory"
        :history="history"
        @use-result="useHistoryResult"
        @clear-history="clearHistory"
        @close="showHistory = false"
      />
    </Transition>

    <div class="calculator" id="calculator">
      <!-- Top bar -->
      <div class="calculator-topbar">
        <span class="calculator-title">Calculator</span>
        <button
          class="history-toggle"
          id="btn-history"
          @click="toggleHistory"
          :class="{ active: showHistory }"
          title="Toggle history"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="10"/>
            <polyline points="12 6 12 12 16 14"/>
          </svg>
        </button>
      </div>

      <!-- Display -->
      <CalculatorDisplay
        :value="displayValue"
        :expression="displayExpression"
        :animate="animateResult"
      />

      <!-- Backspace row -->
      <div class="backspace-row">
        <button class="backspace-btn" id="btn-backspace" @click="backspace" title="Backspace">
          <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 4H8l-7 8 7 8h13a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2z"/>
            <line x1="18" y1="9" x2="12" y2="15"/>
            <line x1="12" y1="9" x2="18" y2="15"/>
          </svg>
        </button>
      </div>

      <!-- Button grid -->
      <div class="button-grid">
        <CalculatorButton
          v-for="btn in buttons"
          :key="btn.id"
          :label="btn.label"
          :type="btn.type"
          :id="btn.id"
          @click="btn.action"
        />
      </div>

      <!-- Keyboard hint -->
      <div class="keyboard-hint">
        <span>⌨ Keyboard supported</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.calculator-wrapper {
  display: flex;
  gap: 16px;
  align-items: flex-start;
  position: relative;
}

.calculator {
  width: 380px;
  background: var(--bg-calculator);
  border-radius: var(--radius-xl);
  padding: 24px;
  box-shadow: var(--shadow-card);
  border: 1px solid var(--border-subtle);
  backdrop-filter: blur(20px);
  position: relative;
  overflow: hidden;
}

/* Subtle inner glow */
.calculator::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.1), transparent);
}

.calculator-topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding: 0 4px;
}

.calculator-title {
  font-size: 14px;
  font-weight: 500;
  color: var(--text-secondary);
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.history-toggle {
  background: transparent;
  border: none;
  color: var(--text-secondary);
  cursor: pointer;
  padding: 8px;
  border-radius: var(--radius-sm);
  transition: all var(--transition-fast);
  display: flex;
  align-items: center;
  justify-content: center;
}

.history-toggle:hover {
  background: var(--bg-button);
  color: var(--text-primary);
}

.history-toggle.active {
  color: var(--bg-operator);
  background: rgba(108, 92, 231, 0.1);
}

.backspace-row {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 12px;
  padding: 0 4px;
}

.backspace-btn {
  background: transparent;
  border: none;
  color: var(--text-secondary);
  cursor: pointer;
  padding: 8px 12px;
  border-radius: var(--radius-sm);
  transition: all var(--transition-fast);
  display: flex;
  align-items: center;
}

.backspace-btn:hover {
  background: var(--bg-button);
  color: var(--bg-clear);
}

.backspace-btn:active {
  transform: scale(0.92);
}

.button-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.keyboard-hint {
  text-align: center;
  margin-top: 16px;
  font-size: 11px;
  color: var(--text-secondary);
  opacity: 0.5;
  letter-spacing: 0.3px;
}

/* History slide transition */
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-enter-from {
  opacity: 0;
  transform: translateX(-20px);
}

.slide-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}
</style>
