<template>
  <div class="p-6">
    <div class="mb-6">
      <h2 class="text-2xl font-bold text-slate-900 tracking-tight mb-1">Performance Reports</h2>
      <p class="text-sm text-slate-500 mt-1">View quarterly performance metrics and monthly trends</p>
    </div>

    <div v-if="loading" class="text-center py-12 text-slate-500">Loading reports...</div>
    <div v-else-if="error" class="bg-rose-100 text-rose-700 border border-rose-200 px-4 py-3 rounded-lg my-4">{{ error }}</div>
    <div v-else>
      <!-- Quarterly Performance -->
      <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 mb-6">
        <h3 class="text-base font-semibold text-slate-800 mb-4">Quarterly Performance</h3>
        <div class="overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="border-b border-slate-200">
              <tr>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Quarter</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Total Orders</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Total Revenue</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Avg Order Value</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Fulfillment Rate</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(q, index) in quarterlyData" :key="index" class="border-b border-slate-100 hover:bg-blue-50 transition-colors">
                <td class="py-3 px-4 text-slate-700"><strong>{{ q.quarter }}</strong></td>
                <td class="py-3 px-4 text-slate-700">{{ q.total_orders }}</td>
                <td class="py-3 px-4 text-slate-700">${{ formatNumber(q.total_revenue) }}</td>
                <td class="py-3 px-4 text-slate-700">${{ formatNumber(q.avg_order_value) }}</td>
                <td class="py-3 px-4">
                  <span
                    :class="[
                      'inline-flex items-center text-xs font-semibold px-2.5 py-0.5 rounded-full',
                      q.fulfillment_rate >= 90 ? 'bg-emerald-200 text-emerald-700' :
                      q.fulfillment_rate >= 75 ? 'bg-amber-200 text-amber-700' :
                      'bg-rose-200 text-rose-700'
                    ]"
                  >
                    {{ q.fulfillment_rate }}%
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Monthly Revenue Trend Chart -->
      <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 mb-6">
        <h3 class="text-base font-semibold text-slate-800 mb-4">Monthly Revenue Trend</h3>
        <div class="px-4 py-8" style="min-height: 300px;">
          <div class="flex items-end justify-around gap-2" style="height: 250px;">
            <div
              v-for="(month, index) in monthlyData"
              :key="index"
              class="flex flex-col items-center flex-1 max-w-[80px]"
            >
              <div class="flex items-end w-full" style="height: 200px;">
                <div
                  class="w-full rounded-t-md cursor-pointer transition-all duration-300"
                  style="background: linear-gradient(to top, #2563eb, #60a5fa);"
                  :style="{ height: getBarHeight(month.revenue) + 'px' }"
                  :title="'$' + formatNumber(month.revenue)"
                ></div>
              </div>
              <div class="text-xs text-slate-500 text-center whitespace-nowrap mt-6" style="transform: rotate(-45deg);">{{ formatMonth(month.month) }}</div>
            </div>
          </div>
        </div>
      </div>

      <!-- Month-over-Month Comparison -->
      <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 mb-6">
        <h3 class="text-base font-semibold text-slate-800 mb-4">Month-over-Month Analysis</h3>
        <div class="overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="border-b border-slate-200">
              <tr>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Month</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Orders</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Revenue</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Change</th>
                <th class="text-left text-xs font-semibold text-slate-500 uppercase tracking-wide py-3 px-4">Growth Rate</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(month, index) in monthlyData" :key="index" class="border-b border-slate-100 hover:bg-blue-50 transition-colors">
                <td class="py-3 px-4 text-slate-700"><strong>{{ formatMonth(month.month) }}</strong></td>
                <td class="py-3 px-4 text-slate-700">{{ month.order_count }}</td>
                <td class="py-3 px-4 text-slate-700">${{ formatNumber(month.revenue) }}</td>
                <td class="py-3 px-4">
                  <span
                    v-if="index > 0"
                    :class="[
                      'font-semibold',
                      getChangeClass(month.revenue, monthlyData[index - 1].revenue)
                    ]"
                  >
                    {{ getChangeValue(month.revenue, monthlyData[index - 1].revenue) }}
                  </span>
                  <span v-else class="text-slate-400">-</span>
                </td>
                <td class="py-3 px-4">
                  <span
                    v-if="index > 0"
                    :class="[
                      'font-semibold',
                      getChangeClass(month.revenue, monthlyData[index - 1].revenue)
                    ]"
                  >
                    {{ getGrowthRate(month.revenue, monthlyData[index - 1].revenue) }}
                  </span>
                  <span v-else class="text-slate-400">-</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Summary Stats -->
      <div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-4 mt-6">
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 border-l-4 border-l-blue-400">
          <div class="text-sm text-slate-500 mb-1">Total Revenue (YTD)</div>
          <div class="text-3xl font-bold text-slate-900">${{ formatNumber(totalRevenue) }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 border-l-4 border-l-blue-400">
          <div class="text-sm text-slate-500 mb-1">Avg Monthly Revenue</div>
          <div class="text-3xl font-bold text-slate-900">${{ formatNumber(avgMonthlyRevenue) }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 border-l-4 border-l-blue-400">
          <div class="text-sm text-slate-500 mb-1">Total Orders (YTD)</div>
          <div class="text-3xl font-bold text-slate-900">{{ totalOrders }}</div>
        </div>
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 p-5 border-l-4 border-l-blue-400">
          <div class="text-sm text-slate-500 mb-1">Best Performing Quarter</div>
          <div class="text-3xl font-bold text-slate-900">{{ bestQuarter }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Reports',
  data() {
    return {
      loading: true,
      error: null,
      quarterlyData: [],
      monthlyData: [],
      totalRevenue: 0,
      avgMonthlyRevenue: 0,
      totalOrders: 0,
      bestQuarter: ''
    }
  },
  mounted() {
    console.log('Reports component mounted')
    this.loadData()
  },
  methods: {
    async loadData() {
      console.log('Loading reports data...')
      try {
        this.loading = true

        // Fetch quarterly data
        console.log('Fetching quarterly data...')
        const quarterlyResponse = await axios.get('http://localhost:8001/api/reports/quarterly')
        this.quarterlyData = quarterlyResponse.data
        console.log('Quarterly data:', this.quarterlyData)

        // Fetch monthly data
        console.log('Fetching monthly data...')
        const monthlyResponse = await axios.get('http://localhost:8001/api/reports/monthly-trends')
        this.monthlyData = monthlyResponse.data
        console.log('Monthly data:', this.monthlyData)

        // Calculate summary stats
        console.log('Calculating summary stats...')
        this.calculateSummaryStats()
        console.log('Summary stats calculated')

      } catch (err) {
        console.log('Error loading reports:', err)
        this.error = 'Failed to load reports: ' + err.message
      } finally {
        this.loading = false
        console.log('Loading complete')
      }
    },

    calculateSummaryStats() {
      // Calculate total revenue
      var total = 0
      for (var i = 0; i < this.monthlyData.length; i++) {
        total = total + this.monthlyData[i].revenue
      }
      this.totalRevenue = total

      // Calculate average monthly revenue
      if (this.monthlyData.length > 0) {
        this.avgMonthlyRevenue = total / this.monthlyData.length
      } else {
        this.avgMonthlyRevenue = 0
      }

      // Calculate total orders
      var orders = 0
      for (var i = 0; i < this.monthlyData.length; i++) {
        orders = orders + this.monthlyData[i].order_count
      }
      this.totalOrders = orders

      // Find best quarter
      var bestQ = ''
      var bestRevenue = 0
      for (var i = 0; i < this.quarterlyData.length; i++) {
        if (this.quarterlyData[i].total_revenue > bestRevenue) {
          bestRevenue = this.quarterlyData[i].total_revenue
          bestQ = this.quarterlyData[i].quarter
        }
      }
      this.bestQuarter = bestQ
    },

    formatNumber(num) {
      console.log('Formatting number:', num)
      // Format number with commas
      var str = num.toString()
      var parts = str.split('.')
      var intPart = parts[0]
      var decPart = parts.length > 1 ? parts[1] : '00'

      var formatted = ''
      var count = 0
      for (var i = intPart.length - 1; i >= 0; i--) {
        if (count > 0 && count % 3 === 0) {
          formatted = ',' + formatted
        }
        formatted = intPart[i] + formatted
        count++
      }

      if (decPart.length === 1) {
        decPart = decPart + '0'
      }
      if (decPart.length > 2) {
        decPart = decPart.substring(0, 2)
      }

      return formatted + '.' + decPart
    },

    formatMonth(monthStr) {
      console.log('Formatting month:', monthStr)
      // Convert YYYY-MM to readable format
      var parts = monthStr.split('-')
      var year = parts[0]
      var month = parts[1]

      var monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
      var monthIndex = parseInt(month) - 1

      return monthNames[monthIndex] + ' ' + year
    },

    getBarHeight(revenue) {
      console.log('Calculating bar height for revenue:', revenue)
      // Calculate bar height (max height 200px)
      var maxRevenue = 0
      for (var i = 0; i < this.monthlyData.length; i++) {
        if (this.monthlyData[i].revenue > maxRevenue) {
          maxRevenue = this.monthlyData[i].revenue
        }
      }

      if (maxRevenue === 0) {
        return 0
      }

      var height = (revenue / maxRevenue) * 200
      return height
    },

    getFulfillmentClass(rate) {
      if (rate >= 90) {
        return 'badge success'
      } else if (rate >= 75) {
        return 'badge warning'
      } else {
        return 'badge danger'
      }
    },

    getChangeValue(current, previous) {
      var change = current - previous
      if (change > 0) {
        return '+$' + this.formatNumber(change)
      } else if (change < 0) {
        return '-$' + this.formatNumber(Math.abs(change))
      } else {
        return '$0.00'
      }
    },

    getChangeClass(current, previous) {
      var change = current - previous
      if (change > 0) {
        return 'text-emerald-600'
      } else if (change < 0) {
        return 'text-rose-600'
      } else {
        return ''
      }
    },

    getGrowthRate(current, previous) {
      if (previous === 0) {
        return 'N/A'
      }

      var rate = ((current - previous) / previous) * 100
      var sign = rate > 0 ? '+' : ''

      return sign + rate.toFixed(1) + '%'
    }
  }
}
</script>
