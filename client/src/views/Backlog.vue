<template>
  <div class="p-6">
    <div class="mb-6">
      <h2 class="text-2xl font-bold text-slate-900 tracking-tight mb-1">Backlog Management</h2>
      <p class="text-sm text-slate-500 mt-1">Track and resolve inventory shortages</p>
    </div>

    <div v-if="loading" class="text-center py-12 text-slate-500">Loading backlog...</div>
    <div v-else-if="error" class="bg-rose-100 text-rose-700 border border-rose-200 px-4 py-3 rounded-lg my-4">{{ error }}</div>
    <div v-else>
      <!-- Priority Summary Cards -->
      <div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-6 mb-8">
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 border-l-4 border-l-rose-400">
          <div class="text-sm text-slate-500 font-medium mb-1">High Priority</div>
          <div class="text-2xl font-bold text-slate-900 tabular-nums">{{ getBacklogByPriority('high').length }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 border-l-4 border-l-amber-400">
          <div class="text-sm text-slate-500 font-medium mb-1">Medium Priority</div>
          <div class="text-2xl font-bold text-slate-900 tabular-nums">{{ getBacklogByPriority('medium').length }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 border-l-4 border-l-sky-500">
          <div class="text-sm text-slate-500 font-medium mb-1">Low Priority</div>
          <div class="text-2xl font-bold text-slate-900 tabular-nums">{{ getBacklogByPriority('low').length }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5">
          <div class="text-sm text-slate-500 font-medium mb-1">Total Backlog Items</div>
          <div class="text-2xl font-bold text-slate-900 tabular-nums">{{ backlogItems.length }}</div>
        </div>
      </div>

      <!-- Backlog Table -->
      <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5">
        <h3 class="text-base font-semibold text-slate-800 mb-4">Backlog Items</h3>

        <div v-if="backlogItems.length === 0" class="py-12 text-center">
          <p class="text-lg text-emerald-600 font-semibold">No backlog items - all orders can be fulfilled!</p>
        </div>
        <div v-else class="overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="border-b border-slate-200">
              <tr>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Order ID</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">SKU</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Item Name</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Qty Needed</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Qty Available</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Shortage</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Days Delayed</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Priority</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in backlogItems" :key="item.id" class="border-b border-slate-100 hover:bg-blue-50 transition-colors">
                <td class="py-3 px-4 text-slate-700"><strong>{{ item.order_id }}</strong></td>
                <td class="py-3 px-4 text-slate-700"><strong>{{ item.item_sku }}</strong></td>
                <td class="py-3 px-4 text-slate-700">{{ item.item_name }}</td>
                <td class="py-3 px-4 text-slate-700">{{ item.quantity_needed }}</td>
                <td class="py-3 px-4 text-slate-700">{{ item.quantity_available }}</td>
                <td class="py-3 px-4">
                  <span class="inline-flex items-center bg-rose-200 text-rose-700 text-xs font-semibold px-2.5 py-0.5 rounded-full">
                    {{ item.quantity_needed - item.quantity_available }} units short
                  </span>
                </td>
                <td class="py-3 px-4">
                  <span :style="{ color: item.days_delayed > 7 ? '#ef4444' : '#f59e0b' }">
                    {{ item.days_delayed }} days
                  </span>
                </td>
                <td class="py-3 px-4">
                  <span
                    :class="[
                      'text-xs font-semibold px-2.5 py-0.5 rounded-full',
                      item.priority === 'high' ? 'bg-rose-200 text-rose-700' :
                      item.priority === 'medium' ? 'bg-amber-200 text-amber-700' :
                      'bg-slate-100 text-slate-700'
                    ]"
                  >
                    {{ item.priority }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'

export default {
  name: 'Backlog',
  setup() {
    const loading = ref(true)
    const error = ref(null)
    const allBacklogItems = ref([])
    const inventoryItems = ref([])

    // Use shared filters
    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    // Filter backlog based on inventory filters
    const backlogItems = computed(() => {
      if (selectedLocation.value === 'all' && selectedCategory.value === 'all') {
        return allBacklogItems.value
      }

      // Get SKUs of items that match the filters
      const validSkus = new Set(inventoryItems.value.map(item => item.sku))
      return allBacklogItems.value.filter(b => validSkus.has(b.item_sku))
    })

    const loadBacklog = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()

        const [backlogData, inventoryData] = await Promise.all([
          api.getBacklog(),
          api.getInventory({
            warehouse: filters.warehouse,
            category: filters.category
          })
        ])

        allBacklogItems.value = backlogData
        inventoryItems.value = inventoryData
      } catch (err) {
        error.value = 'Failed to load backlog: ' + err.message
      } finally {
        loading.value = false
      }
    }

    const getBacklogByPriority = (priority) => {
      return backlogItems.value.filter(item => item.priority === priority)
    }

    // Watch for filter changes and reload data
    watch([selectedLocation, selectedCategory], () => {
      loadBacklog()
    })

    onMounted(loadBacklog)

    return {
      loading,
      error,
      backlogItems,
      getBacklogByPriority
    }
  }
}
</script>
