<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && inventoryItem" class="fixed inset-0 bg-slate-900/50 flex items-center justify-center z-[2000] p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl max-h-[90vh] flex flex-col overflow-hidden" @click.stop>

          <div class="flex items-center justify-between px-6 py-5 border-b border-slate-200">
            <h3 class="text-lg font-semibold text-slate-900">Inventory Item Details</h3>
            <button class="text-slate-400 hover:text-slate-600 hover:bg-slate-100 p-2 rounded-lg transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="flex-1 overflow-y-auto px-6 py-5">

            <div class="flex items-center gap-5 pb-6 border-b border-slate-200 mb-6">
              <div
                class="w-16 h-16 rounded-xl flex items-center justify-center text-white shrink-0"
                :class="{
                  'bg-gradient-to-br from-emerald-500 to-emerald-700': getStockIconClass() === 'success-icon',
                  'bg-gradient-to-br from-amber-400 to-amber-600': getStockIconClass() === 'warning-icon',
                  'bg-gradient-to-br from-red-500 to-red-700': getStockIconClass() === 'danger-icon'
                }"
              >
                <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                  <rect x="8" y="12" width="32" height="28" rx="2" stroke="currentColor" stroke-width="2.5"/>
                  <path d="M16 8V16M32 8V16M8 20H40" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"/>
                  <path d="M16 28H32M16 34H24" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"/>
                </svg>
              </div>

              <div class="flex-1 min-w-0">
                <h4 class="text-2xl font-bold text-slate-900 mb-1">{{ translateProductName(inventoryItem.name) }}</h4>
                <div class="text-sm text-slate-500 font-mono">SKU: {{ inventoryItem.sku }}</div>
              </div>

              <span
                class="shrink-0 px-3 py-1.5 rounded-full text-xs font-semibold uppercase tracking-wide"
                :class="{
                  'bg-emerald-200 text-emerald-700': getStockStatusClass() === 'success',
                  'bg-amber-200 text-amber-700': getStockStatusClass() === 'warning',
                  'bg-rose-200 text-rose-700': getStockStatusClass() === 'danger'
                }"
              >
                {{ getStockStatus() }}
              </span>
            </div>

            <div class="grid grid-cols-2 gap-4 mb-8">
              <div class="p-5 rounded-xl border-2 border-blue-200 bg-blue-50">
                <div class="text-xs font-semibold uppercase tracking-widest text-slate-500 mb-2">Quantity on Hand</div>
                <div class="text-3xl font-bold text-slate-900">{{ inventoryItem.quantity_on_hand }} units</div>
              </div>
              <div
                class="p-5 rounded-xl border-2"
                :class="{
                  'border-emerald-200 bg-emerald-50': getSummaryCardClass() === 'success-card',
                  'border-amber-200 bg-amber-50': getSummaryCardClass() === 'warning-card',
                  'border-rose-200 bg-rose-100': getSummaryCardClass() === 'danger-card'
                }"
              >
                <div class="text-xs font-semibold uppercase tracking-widest text-slate-500 mb-2">Stock Level</div>
                <div class="text-3xl font-bold text-slate-900">{{ stockPercentage }}%</div>
                <div class="text-xs text-slate-500 mt-1">vs. reorder point</div>
              </div>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-6">
              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Category</div>
                <div class="text-sm text-slate-900 font-medium">{{ inventoryItem.category }}</div>
              </div>

              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Location</div>
                <div class="text-sm text-slate-900 font-medium">{{ inventoryItem.location }}</div>
              </div>

              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Reorder Point</div>
                <div class="text-sm text-slate-900 font-medium">{{ inventoryItem.reorder_point }} units</div>
              </div>

              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Units Remaining</div>
                <div class="text-sm font-medium">
                  <span :style="{ color: inventoryItem.quantity_on_hand <= inventoryItem.reorder_point ? '#ef4444' : '#10b981' }">
                    {{ inventoryItem.quantity_on_hand - inventoryItem.reorder_point }} units
                  </span>
                </div>
              </div>

              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Unit Cost</div>
                <div class="text-sm text-slate-900 font-medium">{{ currencySymbol }}{{ inventoryItem.unit_cost.toFixed(2) }}</div>
              </div>

              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Total Value</div>
                <div class="text-lg font-bold text-blue-600">
                  {{ currencySymbol }}{{ totalValue.toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 2}) }}
                </div>
              </div>

              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Warehouse</div>
                <div class="text-sm text-slate-900 font-medium">{{ translateWarehouse(inventoryItem.location) }}</div>
              </div>

              <div class="flex flex-col gap-1">
                <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">Status</div>
                <div>
                  <span
                    class="inline-flex items-center text-xs font-semibold px-2.5 py-0.5 rounded-full"
                    :class="{
                      'bg-emerald-200 text-emerald-700': getStockStatusClass() === 'success',
                      'bg-amber-200 text-amber-700': getStockStatusClass() === 'warning',
                      'bg-rose-200 text-rose-700': getStockStatusClass() === 'danger'
                    }"
                  >
                    {{ getStockStatus() }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <div class="px-6 py-4 border-t border-slate-200 flex justify-end gap-3">
            <button class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-sm font-semibold px-4 py-2 rounded-lg transition-colors" @click="close">Close</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'
import { useI18n } from '../composables/useI18n'

const { currentCurrency, translateProductName, translateWarehouse } = useI18n()

const currencySymbol = computed(() => {
  return currentCurrency.value === 'JPY' ? '¥' : '$'
})

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  inventoryItem: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['close'])

const totalValue = computed(() => {
  if (!props.inventoryItem) return 0
  return props.inventoryItem.quantity_on_hand * props.inventoryItem.unit_cost
})

const stockPercentage = computed(() => {
  if (!props.inventoryItem || props.inventoryItem.reorder_point === 0) return 0
  return Math.round((props.inventoryItem.quantity_on_hand / props.inventoryItem.reorder_point) * 100)
})

const close = () => {
  emit('close')
}

const getStockStatus = () => {
  if (!props.inventoryItem) return 'Unknown'
  if (props.inventoryItem.quantity_on_hand <= props.inventoryItem.reorder_point) {
    return 'Low Stock'
  } else if (props.inventoryItem.quantity_on_hand <= props.inventoryItem.reorder_point * 1.5) {
    return 'Adequate'
  } else {
    return 'In Stock'
  }
}

const getStockStatusClass = () => {
  const status = getStockStatus()
  if (status === 'Low Stock') return 'danger'
  if (status === 'Adequate') return 'warning'
  return 'success'
}

const getStockIconClass = () => {
  const status = getStockStatus()
  if (status === 'Low Stock') return 'danger-icon'
  if (status === 'Adequate') return 'warning-icon'
  return 'success-icon'
}

const getSummaryCardClass = () => {
  const status = getStockStatus()
  if (status === 'Low Stock') return 'danger-card'
  if (status === 'Adequate') return 'warning-card'
  return 'success-card'
}
</script>
