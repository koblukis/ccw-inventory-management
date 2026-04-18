<template>
  <div class="p-6">
    <div class="mb-6">
      <h2 class="text-2xl font-bold text-slate-900 tracking-tight mb-1">{{ t('demand.title') }}</h2>
      <p class="text-sm text-slate-500 mt-1">{{ t('demand.description') }}</p>
    </div>

    <div v-if="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error">{{ error }}</div>
    <div v-else>
      <!-- Trend Summary Cards -->
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
        <!-- Increasing -->
        <div class="bg-emerald-50 border border-emerald-200 rounded-xl p-4 hover:shadow-md transition-shadow duration-200">
          <div class="flex items-center gap-4 mb-4 pb-4 border-b border-emerald-100">
            <div class="w-12 h-12 flex items-center justify-center rounded-xl bg-emerald-200 text-emerald-700 text-2xl font-bold flex-shrink-0">↑</div>
            <div>
              <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide">{{ t('demand.increasingDemand') }}</div>
              <div class="text-2xl font-bold text-slate-900 mt-0.5">{{ t('demand.itemsCount', { count: getForecastsByTrend('increasing').length }) }}</div>
            </div>
          </div>
          <div class="flex flex-col gap-3">
            <div
              v-for="item in getForecastsByTrend('increasing').slice(0, 5)"
              :key="item.id"
              class="flex justify-between items-center px-3 py-2 bg-white/70 rounded-md hover:bg-white transition-colors"
            >
              <span class="text-sm text-slate-900 font-medium flex-1 truncate mr-4">{{ item.item_name }}</span>
              <span class="text-xs font-bold text-emerald-700 flex-shrink-0">+{{ getChangePercent(item) }}%</span>
            </div>
            <div v-if="getForecastsByTrend('increasing').length > 5" class="text-xs text-slate-500 italic text-center py-1">
              +{{ getForecastsByTrend('increasing').length - 5 }} {{ t('demand.more') }}
            </div>
          </div>
        </div>

        <!-- Stable -->
        <div class="bg-blue-100 border border-blue-200 rounded-xl p-4 hover:shadow-md transition-shadow duration-200">
          <div class="flex items-center gap-4 mb-4 pb-4 border-b border-blue-100">
            <div class="w-12 h-12 flex items-center justify-center rounded-xl bg-blue-100 text-blue-700 text-2xl font-bold flex-shrink-0">→</div>
            <div>
              <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide">{{ t('demand.stableDemand') }}</div>
              <div class="text-2xl font-bold text-slate-900 mt-0.5">{{ t('demand.itemsCount', { count: getForecastsByTrend('stable').length }) }}</div>
            </div>
          </div>
          <div class="flex flex-col gap-3">
            <div
              v-for="item in getForecastsByTrend('stable').slice(0, 5)"
              :key="item.id"
              class="flex justify-between items-center px-3 py-2 bg-white/70 rounded-md hover:bg-white transition-colors"
            >
              <span class="text-sm text-slate-900 font-medium flex-1 truncate mr-4">{{ item.item_name }}</span>
              <span class="text-xs font-bold text-blue-600 flex-shrink-0">{{ getChangePercent(item) }}%</span>
            </div>
            <div v-if="getForecastsByTrend('stable').length > 5" class="text-xs text-slate-500 italic text-center py-1">
              +{{ getForecastsByTrend('stable').length - 5 }} {{ t('demand.more') }}
            </div>
          </div>
        </div>

        <!-- Decreasing -->
        <div class="bg-rose-100 border border-rose-200 rounded-xl p-4 hover:shadow-md transition-shadow duration-200">
          <div class="flex items-center gap-4 mb-4 pb-4 border-b border-rose-100">
            <div class="w-12 h-12 flex items-center justify-center rounded-xl bg-rose-200 text-red-700 text-2xl font-bold flex-shrink-0">↓</div>
            <div>
              <div class="text-xs font-semibold text-slate-500 uppercase tracking-wide">{{ t('demand.decreasingDemand') }}</div>
              <div class="text-2xl font-bold text-slate-900 mt-0.5">{{ t('demand.itemsCount', { count: getForecastsByTrend('decreasing').length }) }}</div>
            </div>
          </div>
          <div class="flex flex-col gap-3">
            <div
              v-for="item in getForecastsByTrend('decreasing').slice(0, 5)"
              :key="item.id"
              class="flex justify-between items-center px-3 py-2 bg-white/70 rounded-md hover:bg-white transition-colors"
            >
              <span class="text-sm text-slate-900 font-medium flex-1 truncate mr-4">{{ item.item_name }}</span>
              <span class="text-xs font-bold text-red-700 flex-shrink-0">{{ getChangePercent(item) }}%</span>
            </div>
            <div v-if="getForecastsByTrend('decreasing').length > 5" class="text-xs text-slate-500 italic text-center py-1">
              +{{ getForecastsByTrend('decreasing').length - 5 }} {{ t('demand.more') }}
            </div>
          </div>
        </div>
      </div>

      <!-- Demand Forecasts Table -->
      <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5">
        <h3 class="text-base font-semibold text-slate-800 mb-4">{{ t('demand.demandForecasts') }}</h3>
        <div class="overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="border-b border-slate-200">
              <tr>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('demand.table.sku') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('demand.table.itemName') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('demand.table.currentDemand') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('demand.table.forecastedDemand') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('demand.table.change') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('demand.table.trend') }}</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">{{ t('demand.table.period') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="forecast in forecasts" :key="forecast.id" class="border-b border-slate-100 hover:bg-blue-50 transition-colors">
                <td class="py-3 px-4 text-slate-700"><strong>{{ forecast.item_sku }}</strong></td>
                <td class="py-3 px-4 text-slate-700">{{ forecast.item_name }}</td>
                <td class="py-3 px-4 text-slate-700">{{ forecast.current_demand }}</td>
                <td class="py-3 px-4 text-slate-700"><strong>{{ forecast.forecasted_demand }}</strong></td>
                <td class="py-3 px-4">
                  <span :style="{ color: getChangeColor(forecast) }">
                    {{ getChangePercent(forecast) }}%
                  </span>
                </td>
                <td class="py-3 px-4">
                  <span
                    :class="[
                      'inline-flex items-center text-xs font-semibold px-2.5 py-0.5 rounded-full',
                      forecast.trend === 'increasing' ? 'bg-emerald-200 text-emerald-700' :
                      forecast.trend === 'decreasing' ? 'bg-rose-200 text-rose-700' :
                      'bg-sky-200 text-sky-700'
                    ]"
                  >
                    {{ t(`trends.${forecast.trend}`) }}
                  </span>
                </td>
                <td class="py-3 px-4 text-slate-700">{{ translatePeriod(forecast.period) }}</td>
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
  name: 'Demand',
  setup() {
    const { t } = useI18n()
    const loading = ref(true)
    const error = ref(null)
    const allForecasts = ref([])
    const inventoryItems = ref([])

    // Use shared filters
    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    // Filter forecasts based on inventory filters
    const forecasts = computed(() => {
      if (selectedLocation.value === 'all' && selectedCategory.value === 'all') {
        return allForecasts.value
      }

      // Get SKUs of items that match the filters
      const validSkus = new Set(inventoryItems.value.map(item => item.sku))
      return allForecasts.value.filter(f => validSkus.has(f.item_sku))
    })

    const loadForecasts = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()

        const [forecastsData, inventoryData] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory({
            warehouse: filters.warehouse,
            category: filters.category
          })
        ])

        allForecasts.value = forecastsData
        inventoryItems.value = inventoryData
      } catch (err) {
        error.value = 'Failed to load demand forecasts: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch for filter changes and reload data
    watch([selectedLocation, selectedCategory], () => {
      loadForecasts()
    })

    const getForecastsByTrend = (trend) => {
      return forecasts.value.filter(f => f.trend === trend)
    }

    const getChangePercent = (forecast) => {
      const change = ((forecast.forecasted_demand - forecast.current_demand) / forecast.current_demand * 100).toFixed(1)
      return change > 0 ? `+${change}` : change
    }

    const getChangeColor = (forecast) => {
      const change = forecast.forecasted_demand - forecast.current_demand
      const changePercent = Math.abs((change / forecast.current_demand) * 100)

      // If change is within ±2%, consider it stable and show blue
      if (changePercent <= 2) {
        return '#3b82f6' // Blue for stable
      }

      if (change > 0) return '#10b981' // Green for increasing
      if (change < 0) return '#ef4444' // Red for decreasing
      return '#3b82f6' // Blue for no change
    }

    const translatePeriod = (period) => {
      // Period values like "Next 3 months", "Q1 2025", "30 days", etc.
      const { currentLocale } = useI18n()
      if (currentLocale.value === 'ja') {
        return period
          .replace(/Next\s+/i, '次の')
          .replace(/\s+months/i, 'か月')
          .replace(/\s+month/i, 'か月')
          .replace(/\s+days/i, '日間')
          .replace(/\s+day/i, '日')
          .replace('Q1', '第1四半期')
          .replace('Q2', '第2四半期')
          .replace('Q3', '第3四半期')
          .replace('Q4', '第4四半期')
      }
      return period
    }

    onMounted(loadForecasts)

    return {
      t,
      loading,
      error,
      forecasts,
      getForecastsByTrend,
      getChangePercent,
      getChangeColor,
      translatePeriod
    }
  }
}
</script>
