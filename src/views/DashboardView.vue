<script setup lang="ts">
import { computed, ref } from 'vue'
import { Bar, Line } from 'vue-chartjs'
import {
  BarElement,
  CategoryScale,
  Chart as ChartJS,
  Filler,
  Legend,
  LineElement,
  LinearScale,
  PointElement,
  Title,
  Tooltip,
} from 'chart.js'
import dashboardData from '../data/dashboardData.json'

type MonthData = (typeof dashboardData)[number]

type MetricKey = 'revenue' | 'visitors' | 'conversions' | 'orders'

ChartJS.register(
  CategoryScale,
  LinearScale,
  BarElement,
  LineElement,
  PointElement,
  Filler,
  Title,
  Tooltip,
  Legend,
)

const months = ['All', ...dashboardData.map((item) => item.month)]
const selectedMonth = ref('All')
const labels = dashboardData.map((item) => item.month)

const selectedIndex = computed(() => dashboardData.findIndex((item) => item.month === selectedMonth.value))
const selectedData = computed<MonthData | undefined>(() => dashboardData[selectedIndex.value])
const previousData = computed<MonthData | undefined>(() => selectedIndex.value > 0 ? dashboardData[selectedIndex.value - 1] : undefined)

const filteredData = computed(() => selectedData.value ? [selectedData.value] : dashboardData)
const trend = (key: MetricKey) => {
  if (!selectedData.value || !previousData.value) return null
  const current = selectedData.value[key]
  const previous = previousData.value[key]
  return ((current - previous) / previous) * 100
}

const formatCurrency = (value: number) => `$${value.toLocaleString('en-US')}`
const formatNumber = (value: number) => value.toLocaleString('en-US')
const formatPercent = (value: number) => `${value.toFixed(1)}%`
const formatTrend = (value: number | null) => value === null ? '' : `${value >= 0 ? '▲ +' : '▼ '}${Math.abs(value).toFixed(1)}%`

const kpis = computed(() => {
  const data = filteredData.value
  const sum = (key: Exclude<MetricKey, 'conversions'>) => data.reduce((total, item) => total + item[key], 0)
  const averageConversion = data.reduce((total, item) => total + item.conversions, 0) / data.length

  return [
    { label: 'TOTAL REVENUE', value: formatCurrency(sum('revenue')), icon: 'mdi-currency-usd', color: 'teal', trend: trend('revenue') },
    { label: 'TOTAL VISITORS', value: formatNumber(sum('visitors')), icon: 'mdi-account-outline', color: 'blue', trend: trend('visitors') },
    { label: 'AVG. CONVERSIONS', value: formatPercent(averageConversion), icon: 'mdi-arrow-up-right', color: 'indigo', trend: trend('conversions') },
    { label: 'TOTAL ORDERS', value: formatNumber(sum('orders')), icon: 'mdi-cart-outline', color: 'coral', trend: trend('orders') },
  ]
})

const revenueChartData = computed(() => ({
  labels,
  datasets: [{
    label: 'Revenue',
    data: dashboardData.map((item) => item.revenue),
    backgroundColor: dashboardData.map((item) => selectedMonth.value === 'All' || item.month === selectedMonth.value ? '#159a8c' : 'rgba(21, 154, 140, .25)'),
    borderRadius: 5,
    borderSkipped: false,
    maxBarThickness: 34,
  }],
}))

const visitorChartData = computed(() => ({
  labels,
  datasets: [{
    label: 'Visitors',
    data: dashboardData.map((item) => item.visitors),
    borderColor: '#4b5fd3',
    backgroundColor: 'rgba(75, 95, 211, .14)',
    pointBackgroundColor: dashboardData.map((item) => item.month === selectedMonth.value ? '#4b5fd3' : '#fff'),
    pointBorderColor: '#4b5fd3',
    pointRadius: dashboardData.map((item) => selectedMonth.value === 'All' || item.month === selectedMonth.value ? 5 : 3),
    pointHoverRadius: 7,
    fill: true,
    tension: .36,
    borderWidth: 3,
  }],
}))

const conversionChartData = computed(() => ({
  labels,
  datasets: [{
    label: 'Conversion rate',
    data: dashboardData.map((item) => item.conversions),
    backgroundColor: dashboardData.map((item) => selectedMonth.value === 'All' || item.month === selectedMonth.value ? '#4b5fd3' : 'rgba(75, 95, 211, .25)'),
    borderRadius: 5,
    borderSkipped: false,
    maxBarThickness: 34,
  }],
}))

const baseChartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { display: false },
    tooltip: { displayColors: false, padding: 12 },
  },
}

const revenueChartOptions = {
  ...baseChartOptions,
  plugins: {
    ...baseChartOptions.plugins,
    tooltip: {
      ...baseChartOptions.plugins.tooltip,
      callbacks: { label: (context: { parsed: { y: number | null } }) => ` ${formatCurrency(context.parsed.y ?? 0)}` },
    },
  },
  scales: {
    x: { grid: { display: false }, border: { display: false } },
    y: {
      beginAtZero: true,
      grid: { color: '#e5e9ef' },
      border: { display: false },
      ticks: { callback: (value: string | number) => `$${Number(value) / 1000}K` },
    },
  },
}

const visitorChartOptions = {
  ...baseChartOptions,
  plugins: {
    ...baseChartOptions.plugins,
    tooltip: {
      ...baseChartOptions.plugins.tooltip,
      callbacks: { label: (context: { parsed: { y: number | null } }) => ` ${(context.parsed.y ?? 0).toLocaleString('en-US')} visitors` },
    },
  },
  scales: {
    x: { grid: { display: false }, border: { display: false } },
    y: {
      beginAtZero: true,
      grid: { color: '#e5e9ef' },
      border: { display: false },
      ticks: { callback: (value: string | number) => `${Number(value) / 1000}K` },
    },
  },
}

const conversionChartOptions = {
  ...baseChartOptions,
  plugins: {
    ...baseChartOptions.plugins,
    tooltip: {
      ...baseChartOptions.plugins.tooltip,
      callbacks: { label: (context: { parsed: { y: number | null } }) => ` ${(context.parsed.y ?? 0).toFixed(1)}% conversion rate` },
    },
  },
  scales: {
    x: { grid: { display: false }, border: { display: false } },
    y: {
      beginAtZero: true,
      grid: { color: '#e5e9ef' },
      border: { display: false },
      ticks: { callback: (value: string | number) => `${value}%` },
    },
  },
}
</script>

<template>
  <v-main class="dashboard-shell">
    <header class="filter-bar">
      <div class="header-copy">
        <div class="brand-kicker">PROTОGEN 200 CAPSTONE</div>
        <h1>Business Analytics — 2025</h1>
        <p>Your store at a glance</p>
      </div>
      <div class="month-picker" aria-label="Filter dashboard by month">
        <v-btn-toggle v-model="selectedMonth" class="month-toggle" mandatory divided color="primary" density="comfortable">
          <v-btn v-for="month in months" :key="month" :value="month" class="month-button">{{ month }}</v-btn>
        </v-btn-toggle>
      </div>
    </header>

    <v-container class="dashboard-content px-4 px-md-8 py-7 py-md-10">
      <section class="kpi-grid" aria-label="Key performance indicators">
        <v-card v-for="kpi in kpis" :key="kpi.label" class="scorecard" elevation="0">
          <v-card-text>
            <div class="scorecard-topline">
              <span class="metric-icon" :class="kpi.color"><v-icon size="20">{{ kpi.icon }}</v-icon></span>
              <span v-if="selectedMonth !== 'All' && kpi.trend !== null" class="trend-badge" :class="kpi.trend >= 0 ? 'trend-up' : 'trend-down'">{{ formatTrend(kpi.trend) }}</span>
            </div>
            <div class="metric-label">{{ kpi.label }}</div>
            <div class="metric-value">{{ kpi.value }}</div>
            <div v-if="selectedMonth !== 'All'" class="metric-context">{{ selectedMonth }} 2025 <span v-if="previousData">vs. {{ previousData.month }}</span></div>
          </v-card-text>
        </v-card>
      </section>

      <section class="charts-grid" aria-label="Business charts">
        <v-card class="chart-card" elevation="0">
          <v-card-item>
            <template #title><span class="chart-title">Monthly Revenue</span></template>
            <template #subtitle>Revenue performance across 2025</template>
          </v-card-item>
          <v-card-text class="chart-wrap"><Bar :data="revenueChartData" :options="revenueChartOptions" /></v-card-text>
        </v-card>

        <v-card class="chart-card" elevation="0">
          <v-card-item>
            <template #title><span class="chart-title">Visitor Trends</span></template>
            <template #subtitle>Unique visitors across 2025</template>
          </v-card-item>
          <v-card-text class="chart-wrap"><Line :data="visitorChartData" :options="visitorChartOptions" /></v-card-text>
        </v-card>
      </section>

      <v-card class="chart-card conversion-card" elevation="0">
        <v-card-item>
          <template #title><span class="chart-title">Conversion Rate Trend</span></template>
          <template #subtitle>Percentage of visitors who completed a purchase</template>
        </v-card-item>
        <v-card-text class="chart-wrap"><Bar :data="conversionChartData" :options="conversionChartOptions" /></v-card-text>
      </v-card>

      <footer class="dashboard-footer">Data is simulated for demonstration purposes. Protogen 200 Capstone — 2025.</footer>
    </v-container>
  </v-main>
</template>

<style scoped>
.chart-wrap { height: 360px; padding-top: 12px !important; }
@media (max-width: 700px) {
  .chart-wrap { height: 290px; }
}
</style>
