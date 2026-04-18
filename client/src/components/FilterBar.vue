<template>
  <div class="flex items-center gap-3 px-6 py-3 bg-white border-b border-slate-200">
    <div class="flex items-center gap-3 flex-1">
      <div class="flex items-center gap-2">
        <label class="text-xs font-semibold text-slate-500 uppercase tracking-wide mr-1">{{ t('filters.timePeriod') }}</label>
        <select v-model="selectedPeriod" class="border border-slate-300 rounded-lg text-sm text-slate-700 px-3 py-1.5 bg-white focus:outline-none focus:ring-2 focus:ring-violet-400 focus:border-transparent cursor-pointer hover:border-slate-400 transition-colors">
          <option value="all">{{ t('filters.allMonths') }}</option>
          <option value="2025-01">{{ t('months.january') }}</option>
          <option value="2025-02">{{ t('months.february') }}</option>
          <option value="2025-03">{{ t('months.march') }}</option>
          <option value="2025-04">{{ t('months.april') }}</option>
          <option value="2025-05">{{ t('months.may') }}</option>
          <option value="2025-06">{{ t('months.june') }}</option>
          <option value="2025-07">{{ t('months.july') }}</option>
          <option value="2025-08">{{ t('months.august') }}</option>
          <option value="2025-09">{{ t('months.september') }}</option>
          <option value="2025-10">{{ t('months.october') }}</option>
          <option value="2025-11">{{ t('months.november') }}</option>
          <option value="2025-12">{{ t('months.december') }}</option>
        </select>
      </div>

      <div class="flex items-center gap-2">
        <label class="text-xs font-semibold text-slate-500 uppercase tracking-wide mr-1">{{ t('filters.location') }}</label>
        <select v-model="selectedLocation" class="border border-slate-300 rounded-lg text-sm text-slate-700 px-3 py-1.5 bg-white focus:outline-none focus:ring-2 focus:ring-violet-400 focus:border-transparent cursor-pointer hover:border-slate-400 transition-colors">
          <option value="all">{{ t('filters.all') }}</option>
          <option value="San Francisco">{{ t('warehouses.sanFrancisco') }}</option>
          <option value="London">{{ t('warehouses.london') }}</option>
          <option value="Tokyo">{{ t('warehouses.tokyo') }}</option>
        </select>
      </div>

      <div class="flex items-center gap-2">
        <label class="text-xs font-semibold text-slate-500 uppercase tracking-wide mr-1">{{ t('filters.category') }}</label>
        <select v-model="selectedCategory" class="border border-slate-300 rounded-lg text-sm text-slate-700 px-3 py-1.5 bg-white focus:outline-none focus:ring-2 focus:ring-violet-400 focus:border-transparent cursor-pointer hover:border-slate-400 transition-colors">
          <option value="all">{{ t('filters.all') }}</option>
          <option value="circuit boards">{{ t('categories.circuitBoards') }}</option>
          <option value="sensors">{{ t('categories.sensors') }}</option>
          <option value="actuators">{{ t('categories.actuators') }}</option>
          <option value="controllers">{{ t('categories.controllers') }}</option>
          <option value="power supplies">{{ t('categories.powerSupplies') }}</option>
        </select>
      </div>

      <div class="flex items-center gap-2">
        <label class="text-xs font-semibold text-slate-500 uppercase tracking-wide mr-1">{{ t('filters.orderStatus') }}</label>
        <select v-model="selectedStatus" class="border border-slate-300 rounded-lg text-sm text-slate-700 px-3 py-1.5 bg-white focus:outline-none focus:ring-2 focus:ring-violet-400 focus:border-transparent cursor-pointer hover:border-slate-400 transition-colors">
          <option value="all">{{ t('filters.all') }}</option>
          <option value="delivered">{{ t('status.delivered') }}</option>
          <option value="shipped">{{ t('status.shipped') }}</option>
          <option value="processing">{{ t('status.processing') }}</option>
          <option value="backordered">{{ t('status.backordered') }}</option>
        </select>
      </div>
    </div>

    <button
      class="ml-auto flex items-center gap-1.5 text-sm text-slate-500 hover:text-slate-700 px-3 py-1.5 rounded-lg hover:bg-slate-100 transition-colors disabled:opacity-40 disabled:cursor-not-allowed"
      @click="resetFilters"
      :disabled="!hasActiveFilters"
      title="Reset all filters"
    >
      <svg class="w-4 h-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a1 1 0 011.276.61A5.002 5.002 0 0014.001 13H11a1 1 0 110-2h5a1 1 0 011 1v5a1 1 0 11-2 0v-2.101a7.002 7.002 0 01-11.601-2.566 1 1 0 01.61-1.276z" clip-rule="evenodd" />
      </svg>
    </button>
  </div>
</template>

<script>
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'FilterBar',
  setup() {
    const {
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    } = useFilters()

    const { t } = useI18n()

    return {
      t,
      selectedPeriod,
      selectedLocation,
      selectedCategory,
      selectedStatus,
      hasActiveFilters,
      resetFilters
    }
  }
}
</script>
