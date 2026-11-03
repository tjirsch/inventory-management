<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && costData" class="modal-overlay" @click="close">
        <div class="modal-container" @click.stop>
          <div class="modal-header">
            <h3 class="modal-title">{{ costData.month }} Cost Breakdown</h3>
            <button class="close-button" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="modal-body">
            <div class="cost-summary">
              <div class="summary-card total">
                <div class="summary-label">Total Costs</div>
                <div class="summary-value">{{ currencySymbol }}{{ totalCosts.toLocaleString() }}</div>
              </div>
            </div>

            <div class="cost-breakdown">
              <div class="cost-item procurement">
                <div class="cost-header">
                  <div class="cost-icon">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                      <rect x="4" y="6" width="16" height="14" rx="2" stroke="currentColor" stroke-width="2"/>
                      <path d="M8 6V4M16 6V4M4 10H20" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="cost-info">
                    <div class="cost-name">Procurement</div>
                    <div class="cost-amount">{{ currencySymbol }}{{ costData.procurement.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="cost-percentage">{{ getProcurementPercentage() }}% of total</div>
              </div>

              <div class="cost-item operational">
                <div class="cost-header">
                  <div class="cost-icon">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                      <circle cx="12" cy="12" r="8" stroke="currentColor" stroke-width="2"/>
                      <path d="M12 8V12L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="cost-info">
                    <div class="cost-name">Operational</div>
                    <div class="cost-amount">{{ currencySymbol }}{{ costData.operational.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="cost-percentage">{{ getOperationalPercentage() }}% of total</div>
              </div>

              <div class="cost-item labor">
                <div class="cost-header">
                  <div class="cost-icon">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                      <circle cx="12" cy="8" r="4" stroke="currentColor" stroke-width="2"/>
                      <path d="M6 20C6 16.6863 8.68629 14 12 14C15.3137 14 18 16.6863 18 20" stroke="currentColor" stroke-width="2"/>
                    </svg>
                  </div>
                  <div class="cost-info">
                    <div class="cost-name">Labor</div>
                    <div class="cost-amount">{{ currencySymbol }}{{ costData.labor.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="cost-percentage">{{ getLaborPercentage() }}% of total</div>
              </div>

              <div class="cost-item overhead">
                <div class="cost-header">
                  <div class="cost-icon">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                      <path d="M3 12L5 10M5 10L12 3L19 10M5 10V20C5 20.5523 5.44772 21 6 21H9M19 10L21 12M19 10V20C19 20.5523 18.5523 21 18 21H15M9 21C9 21 9 18 9 16C9 14 10 14 12 14C14 14 15 14 15 16C15 18 15 21 15 21M9 21H15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="cost-info">
                    <div class="cost-name">Overhead</div>
                    <div class="cost-amount">{{ currencySymbol }}{{ costData.overhead.toLocaleString() }}</div>
                  </div>
                </div>
                <div class="cost-percentage">{{ getOverheadPercentage() }}% of total</div>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn-secondary" @click="close">Close</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'
import { useI18n } from '../composables/useI18n'

const { currentCurrency } = useI18n()

const currencySymbol = computed(() => {
  return currentCurrency.value === 'JPY' ? '¥' : '$'
})

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  costData: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

const totalCosts = computed(() => {
  if (!props.costData) return 0
  return props.costData.procurement + props.costData.operational +
         props.costData.labor + props.costData.overhead
})

const getProcurementPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.procurement / totalCosts.value) * 100).toFixed(1)
}

const getOperationalPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.operational / totalCosts.value) * 100).toFixed(1)
}

const getLaborPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.labor / totalCosts.value) * 100).toFixed(1)
}

const getOverheadPercentage = () => {
  if (!props.costData || totalCosts.value === 0) return 0
  return ((props.costData.overhead / totalCosts.value) * 100).toFixed(1)
}

const close = () => {
  emit('close')
}
</script>

<style scoped>
/* Shared modal chrome (keep in sync across *DetailModal / ProfileDetailsModal / TasksModal).
   z-index 2000 intentionally preserved from the original. */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: var(--color-overlay);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: var(--space-4);
}

.modal-container {
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
  max-width: min(600px, calc(100vw - 2 * var(--space-4))); /* never wider than the viewport minus gutters at 375px */
  width: 100%;
  max-height: calc(100vh - 2 * var(--space-4));
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-3);
  padding: var(--space-6);
  border-bottom: 1px solid var(--color-border);
  flex-shrink: 0;
}

.modal-title {
  font-size: var(--text-lg);
  font-weight: 600;
  color: var(--color-text);
  margin: 0;
}

.close-button {
  background: none;
  border: none;
  color: var(--color-text-muted);
  cursor: pointer;
  width: var(--control-height);
  height: var(--control-height);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  border-radius: var(--radius-sm);
  transition: background-color 0.15s ease, color 0.15s ease;
}

.close-button:hover {
  background: var(--color-surface-muted);
  color: var(--color-text);
}

.close-button:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px var(--color-focus-ring);
}

/* Body scrolls internally so header/footer stay visible on short/narrow screens */
.modal-body {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  padding: var(--space-6);
}

.modal-footer {
  padding: var(--space-4) var(--space-6);
  border-top: 1px solid var(--color-border);
  display: flex;
  justify-content: flex-end;
  gap: var(--space-3);
  flex-shrink: 0;
}

/* Matches the global .btn look; scoped copy because the footer button only uses .btn-secondary */
.btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  height: var(--control-height);
  padding: 0 var(--space-4);
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-weight: 500;
  font-size: var(--text-sm);
  color: var(--color-text);
  cursor: pointer;
  transition: background-color 0.15s ease, border-color 0.15s ease;
  font-family: inherit;
}

.btn-secondary:hover {
  background: var(--color-surface-muted);
}

.btn-secondary:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.cost-summary {
  margin-bottom: var(--space-8);
}

.summary-card {
  padding: var(--space-6);
  border-radius: var(--radius-md);
  text-align: center;
}

.summary-card.total {
  background: var(--color-info);
  color: var(--color-surface);
}

.summary-label {
  font-size: var(--text-base);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  opacity: 0.9;
  margin-bottom: var(--space-2);
}

.summary-value {
  font-size: var(--text-2xl);
  font-weight: 600;
}

.cost-breakdown {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
}

.cost-item {
  padding: var(--space-5);
  border-radius: var(--radius-md);
  border: 1px solid;
}

.cost-item.procurement {
  border-color: var(--color-border);
  background: var(--color-info-soft);
}

.cost-item.operational {
  border-color: var(--color-border);
  background: #f5f3ff; /* purple has no status token; categorical colour kept for "operational" */
}

.cost-item.labor {
  border-color: var(--color-border);
  background: var(--color-success-soft);
}

.cost-item.overhead {
  border-color: var(--color-border);
  background: var(--color-warning-soft);
}

.cost-header {
  display: flex;
  align-items: center;
  gap: var(--space-4);
  margin-bottom: var(--space-2);
}

.cost-icon {
  width: 48px;
  height: 48px;
  border-radius: var(--radius-md);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.cost-item.procurement .cost-icon {
  background: var(--color-info);
  color: var(--color-surface);
}

.cost-item.operational .cost-icon {
  background: #8b5cf6; /* categorical purple, see above */
  color: var(--color-surface);
}

.cost-item.labor .cost-icon {
  background: var(--color-success);
  color: var(--color-surface);
}

.cost-item.overhead .cost-icon {
  background: var(--color-warning);
  color: var(--color-surface);
}

.cost-info {
  flex: 1;
}

.cost-name {
  font-weight: 600;
  color: var(--color-text);
  font-size: var(--text-lg);
  margin-bottom: var(--space-1);
}

.cost-amount {
  font-size: var(--text-2xl);
  font-weight: 600;
  color: var(--color-text);
}

.cost-percentage {
  font-size: var(--text-base);
  color: var(--color-text-muted);
  font-weight: 500;
}

/* Modal transition animations */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-container,
.modal-leave-active .modal-container {
  transition: transform 0.2s ease;
}

.modal-enter-from .modal-container,
.modal-leave-to .modal-container {
  transform: scale(0.95);
}

/* Narrow screens: tighter padding so content keeps room at 375px */
@media (max-width: 480px) {
  .modal-header,
  .modal-body,
  .modal-footer {
    padding-left: var(--space-4);
    padding-right: var(--space-4);
  }
}
</style>
