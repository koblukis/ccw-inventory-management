<template>
  <div class="inventory">
    <div class="mb-6">
      <h2 class="text-2xl font-bold text-slate-900 tracking-tight">{{ t('inventory.title') }}</h2>
      <p class="mt-1 text-sm text-slate-500">{{ t('inventory.description') }}</p>
    </div>

    <div v-if="loading" class="flex items-center justify-center py-16 text-slate-400 text-sm">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="flex items-center justify-center py-16 text-red-500 text-sm">{{ error }}</div>
    <div v-else>
      <div class="bg-white rounded-xl shadow-sm border border-slate-200">
        <div class="flex justify-between items-center gap-6 px-6 py-5 border-b border-slate-200">
          <h3 class="text-base font-semibold text-slate-900 m-0">{{ t('inventory.stockLevels') }} ({{ filteredItems.length }} {{ t('inventory.skus') }})</h3>
          <div class="relative flex items-center min-w-[300px]">
            <svg class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-400 pointer-events-none" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z" clip-rule="evenodd" />
            </svg>
            <input
              v-model="searchQuery"
              type="text"
              :placeholder="t('inventory.searchPlaceholder')"
              class="w-full border border-slate-300 rounded-lg pl-10 pr-10 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-violet-400 focus:border-transparent bg-white"
            />
            <button
              v-if="searchQuery"
              @click="searchQuery = ''"
              class="absolute right-3 top-1/2 -translate-y-1/2 text-slate-400 hover:text-slate-600"
              :title="t('inventory.clearSearch')"
            >
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-4 h-4">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
              </svg>
            </button>
          </div>
        </div>
        <div class="overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="border-b border-slate-200">
              <tr>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.sku') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.itemName') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.category') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.quantityOnHand') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.reorderPoint') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.unitCost') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.totalValue') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.location') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('inventory.table.status') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in filteredItems"
                :key="item.id"
                class="border-b border-slate-100 hover:bg-violet-100 transition-colors cursor-pointer"
                @click="showItemDetail(item)"
              >
                <td class="py-3 px-4 text-slate-700"><strong>{{ item.sku }}</strong></td>
                <td class="py-3 px-4 text-slate-700">{{ translateProductName(item.name) }}</td>
                <td class="py-3 px-4 text-slate-700">{{ translateCategory(item.category) }}</td>
                <td class="py-3 px-4 text-slate-700"><strong>{{ item.quantity_on_hand }}</strong></td>
                <td class="py-3 px-4 text-slate-700">{{ item.reorder_point }}</td>
                <td class="py-3 px-4 text-slate-700">{{ currencySymbol }}{{ item.unit_cost.toFixed(2) }}</td>
                <td class="py-3 px-4 text-slate-700"><strong>{{ currencySymbol }}{{ (item.quantity_on_hand * item.unit_cost).toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 2}) }}</strong></td>
                <td class="py-3 px-4 text-slate-700">{{ translateWarehouse(item.location) }}</td>
                <td class="py-3 px-4 text-slate-700">
                  <span :class="[
                    'inline-flex items-center text-xs font-semibold px-2.5 py-0.5 rounded-full',
                    getStockStatusClass(item) === 'success' ? 'bg-emerald-200 text-emerald-700' :
                    getStockStatusClass(item) === 'warning' ? 'bg-amber-200 text-amber-700' :
                    'bg-rose-200 text-rose-700'
                  ]">
                    {{ getStockStatus(item) }}
                  </span>
                </td>
              </tr>
              <tr v-if="filteredItems.length === 0">
                <td colspan="9">
                  <div class="flex flex-col items-center justify-center py-16 text-slate-400">
                    <svg class="w-12 h-12 mb-3 text-slate-300" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-2.586a1 1 0 00-.707.293l-2.414 2.414a1 1 0 01-.707.293h-3.172a1 1 0 01-.707-.293l-2.414-2.414A1 1 0 006.586 13H4" />
                    </svg>
                    <p class="text-sm font-medium">No items found</p>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <InventoryDetailModal
      :is-open="showItemModal"
      :inventory-item="selectedItem"
      @close="showItemModal = false"
    />
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'
import InventoryDetailModal from '../components/InventoryDetailModal.vue'

export default {
  name: 'Inventory',
  components: {
    InventoryDetailModal
  },
  setup() {
    const { t, currentCurrency, translateProductName, translateWarehouse } = useI18n()

    const currencySymbol = computed(() => {
      return currentCurrency.value === 'JPY' ? '¥' : '$'
    })

    const loading = ref(true)
    const error = ref(null)
    const items = ref([])
    const searchQuery = ref('')

    // Modal state
    const showItemModal = ref(false)
    const selectedItem = ref(null)

    // Use shared filters
    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    // Stock status order for sorting (using status keys)
    const STATUS_ORDER = { 'lowStock': 0, 'adequate': 1, 'inStock': 2 }

    // Get stock status key (for sorting and translation)
    const getStockStatusKey = (item) => {
      if (item.quantity_on_hand <= item.reorder_point) {
        return 'lowStock'
      } else if (item.quantity_on_hand <= item.reorder_point * 1.5) {
        return 'adequate'
      } else {
        return 'inStock'
      }
    }

    // Computed property to filter items by search query and sort by stock status
    const filteredItems = computed(() => {
      let filtered = items.value

      // Apply search filter if query exists
      if (searchQuery.value.trim()) {
        const query = searchQuery.value.toLowerCase().trim()
        filtered = filtered.filter(item =>
          item.name.toLowerCase().includes(query)
        )
      }

      // Sort by stock status: Low Stock first, then Adequate, then In Stock
      // Always create a copy to avoid mutating the original array
      return filtered.slice().sort((a, b) => {
        const statusA = getStockStatusKey(a)
        const statusB = getStockStatusKey(b)
        return STATUS_ORDER[statusA] - STATUS_ORDER[statusB]
      })
    })

    const loadInventory = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()
        // Inventory doesn't support month/status filters, only warehouse and category
        items.value = await api.getInventory({
          warehouse: filters.warehouse,
          category: filters.category
        })
      } catch (err) {
        error.value = 'Failed to load inventory: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch for filter changes and reload data
    watch([selectedLocation, selectedCategory], () => {
      loadInventory()
    })

    const getStockStatus = (item) => {
      const key = getStockStatusKey(item)
      return t(`status.${key}`)
    }

    const getStockStatusClass = (item) => {
      if (item.quantity_on_hand <= item.reorder_point) {
        return 'danger'
      } else if (item.quantity_on_hand <= item.reorder_point * 1.5) {
        return 'warning'
      } else {
        return 'success'
      }
    }

    const translateCategory = (category) => {
      const categoryMap = {
        'Circuit Boards': t('categories.circuitBoards'),
        'Sensors': t('categories.sensors'),
        'Actuators': t('categories.actuators'),
        'Controllers': t('categories.controllers'),
        'Power Supplies': t('categories.powerSupplies')
      }
      return categoryMap[category] || category
    }

    const showItemDetail = (item) => {
      selectedItem.value = item
      showItemModal.value = true
    }

    onMounted(loadInventory)

    return {
      t,
      loading,
      error,
      items,
      searchQuery,
      filteredItems,
      getStockStatus,
      getStockStatusClass,
      translateCategory,
      showItemModal,
      selectedItem,
      showItemDetail,
      currencySymbol,
      translateProductName,
      translateWarehouse
    }
  }
}
</script>
