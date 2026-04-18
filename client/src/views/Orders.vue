<template>
  <div class="p-6">
    <div class="mb-6">
      <h2 class="text-2xl font-bold text-slate-900 tracking-tight">{{ t('orders.title') }}</h2>
      <p class="text-sm text-slate-500 mt-1">{{ t('orders.description') }}</p>
    </div>

    <div v-if="loading" class="flex items-center justify-center py-16 text-slate-400 text-sm">
      {{ t('common.loading') }}
    </div>
    <div v-else-if="error" class="flex items-center justify-center py-16 text-red-500 text-sm">
      {{ error }}
    </div>
    <div v-else>
      <div class="grid grid-cols-4 gap-4 mb-6">
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-4">
          <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">{{ t('status.delivered') }}</div>
          <div class="text-2xl font-bold text-emerald-600">{{ getOrdersByStatus('Delivered').length }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-4">
          <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">{{ t('status.shipped') }}</div>
          <div class="text-2xl font-bold text-sky-600">{{ getOrdersByStatus('Shipped').length }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-4">
          <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">{{ t('status.processing') }}</div>
          <div class="text-2xl font-bold text-amber-600">{{ getOrdersByStatus('Processing').length }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-4">
          <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide mb-1">{{ t('status.backordered') }}</div>
          <div class="text-2xl font-bold text-rose-600">{{ getOrdersByStatus('Backordered').length }}</div>
        </div>
      </div>

      <div class="bg-white rounded-xl shadow-sm border border-slate-200">
        <div class="px-6 py-4 border-b border-slate-200">
          <h3 class="text-base font-semibold text-slate-900">{{ t('orders.allOrders') }} ({{ orders.length }})</h3>
        </div>
        <div class="overflow-x-auto">
          <table class="w-full text-sm" style="table-layout: fixed;">
            <colgroup>
              <col style="width: 130px;" />
              <col style="width: 180px;" />
              <col style="width: 200px;" />
              <col style="width: 130px;" />
              <col style="width: 140px;" />
              <col style="width: 140px;" />
              <col style="width: 120px;" />
            </colgroup>
            <thead class="border-b border-slate-200">
              <tr>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('orders.table.orderNumber') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('orders.table.customer') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('orders.table.items') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('orders.table.status') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('orders.table.orderDate') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('orders.table.expectedDelivery') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('orders.table.totalValue') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="order in orders"
                :key="order.id"
                class="border-b border-slate-100 hover:bg-blue-100 transition-colors cursor-pointer"
              >
                <td class="py-3 px-4 text-slate-700">
                  <strong>{{ order.order_number }}</strong>
                </td>
                <td class="py-3 px-4 text-slate-700">{{ translateCustomerName(order.customer) }}</td>
                <td class="py-3 px-4 text-slate-700">
                  <details class="relative">
                    <summary class="text-blue-600 hover:text-blue-700 text-sm font-medium cursor-pointer list-none select-none inline-block">
                      {{ t('orders.itemsCount', { count: order.items.length }) }}
                    </summary>
                    <div class="absolute z-10 bg-white border border-slate-200 rounded-xl shadow-lg p-3 mt-1 w-64">
                      <div
                        v-for="(item, idx) in order.items"
                        :key="idx"
                        class="py-1.5 border-b border-slate-100 last:border-0"
                      >
                        <span class="block text-sm font-medium text-slate-900">{{ translateProductName(item.name) }}</span>
                        <span class="block text-xs text-slate-500">{{ t('orders.quantity') }}: {{ item.quantity }} @ {{ currencySymbol }}{{ item.unit_price }}</span>
                      </div>
                    </div>
                  </details>
                </td>
                <td class="py-3 px-4 text-slate-700">
                  <span
                    class="inline-flex items-center text-xs font-semibold px-2.5 py-0.5 rounded-full"
                    :class="{
                      'bg-emerald-200 text-emerald-700': order.status === 'Delivered',
                      'bg-sky-200 text-sky-700': order.status === 'Shipped',
                      'bg-amber-200 text-amber-700': order.status === 'Processing',
                      'bg-rose-200 text-rose-700': order.status === 'Backordered',
                    }"
                  >
                    {{ t(`status.${order.status.toLowerCase()}`) }}
                  </span>
                </td>
                <td class="py-3 px-4 text-slate-700">{{ formatDate(order.order_date) }}</td>
                <td class="py-3 px-4 text-slate-700">{{ formatDate(order.expected_delivery) }}</td>
                <td class="py-3 px-4 text-slate-700">
                  <strong>{{ currencySymbol }}{{ order.total_value.toLocaleString() }}</strong>
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
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Orders',
  setup() {
    const { t, currentCurrency, translateProductName, translateCustomerName } = useI18n()

    const currencySymbol = computed(() => {
      return currentCurrency.value === 'JPY' ? '¥' : '$'
    })
    const loading = ref(true)
    const error = ref(null)
    const orders = ref([])

    // Use shared filters
    const {
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      getCurrentFilters
    } = useFilters()

    const loadOrders = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()
        const fetchedOrders = await api.getOrders(filters)

        // Sort orders by order_date (earliest first)
        orders.value = fetchedOrders.sort((a, b) => {
          const dateA = new Date(a.order_date)
          const dateB = new Date(b.order_date)
          return dateA - dateB
        })
      } catch (err) {
        error.value = 'Failed to load orders: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch for filter changes and reload data
    watch([selectedPeriod, selectedLocation, selectedCategory, selectedStatus], () => {
      loadOrders()
    })

    const getOrdersByStatus = (status) => {
      return orders.value.filter(order => order.status === status)
    }

    const getOrderStatusClass = (status) => {
      const statusMap = {
        'Delivered': 'success',
        'Shipped': 'info',
        'Processing': 'warning',
        'Backordered': 'danger'
      }
      return statusMap[status] || 'info'
    }

    const formatDate = (dateString) => {
      const { currentLocale } = useI18n()
      const locale = currentLocale.value === 'ja' ? 'ja-JP' : 'en-US'
      return new Date(dateString).toLocaleDateString(locale, {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }

    onMounted(loadOrders)

    return {
      t,
      loading,
      error,
      orders,
      getOrdersByStatus,
      getOrderStatusClass,
      formatDate,
      currencySymbol,
      translateProductName,
      translateCustomerName
    }
  }
}
</script>
