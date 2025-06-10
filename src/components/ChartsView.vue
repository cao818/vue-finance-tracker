<template>
  <div>
    <div ref="pieChart" style="width: 100%; height: 300px; margin-bottom: 40px;"></div>
    <div ref="barChart" style="width: 100%; height: 300px;"></div>
  </div>
</template>

<script setup>
import * as echarts from 'echarts'
import { onMounted, onBeforeUnmount, watch, ref } from 'vue'
import { toRaw } from 'vue'

const props = defineProps({
  records: {
    type: Array,
    required: true
  }
})

const pieChart = ref(null)
const barChart = ref(null)
let pieInstance = null
let barInstance = null

function initPieChart() {
  if (!pieChart.value) return
  pieInstance = echarts.init(pieChart.value)
}

function initBarChart() {
  if (!barChart.value) return
  barInstance = echarts.init(barChart.value)
}

function updatePieChart() {
  if (!pieInstance) return
  const incomeSum = props.records.filter(r => r.type === 'income').reduce((sum, r) => sum + Number(r.amount), 0)
  const expenseSum = props.records.filter(r => r.type === 'expense').reduce((sum, r) => sum + Number(r.amount), 0)

  const option = {
    title: {
      text: '收入 vs 支出占比',
      left: 'center'
    },
    tooltip: {
      trigger: 'item'
    },
    legend: {
      bottom: 10,
      left: 'center'
    },
    series: [
      {
        name: '金额',
        type: 'pie',
        radius: '50%',
        data: [
          { value: incomeSum, name: '收入' },
          { value: expenseSum, name: '支出' }
        ],
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)'
          }
        }
      }
    ]
  }
  pieInstance.setOption(option)
}

function updateBarChart() {
  if (!barInstance) return
  const mapIncome = {}
  const mapExpense = {}

  props.records.forEach(r => {
    if (r.type === 'income') {
      mapIncome[r.category] = (mapIncome[r.category] || 0) + Number(r.amount)
    } else if (r.type === 'expense') {
      mapExpense[r.category] = (mapExpense[r.category] || 0) + Number(r.amount)
    }
  })

  // 所有分类合并
  const categories = Array.from(new Set([...Object.keys(mapIncome), ...Object.keys(mapExpense)]))

  const incomeData = categories.map(c => mapIncome[c] || 0)
  const expenseData = categories.map(c => mapExpense[c] || 0)

  const option = {
    title: {
      text: '各分类收支金额',
      left: 'center'
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' }
    },
    legend: {
      top: 30,
      data: ['收入', '支出']
    },
    xAxis: {
      type: 'category',
      data: categories,
      axisLabel: { rotate: 30 }
    },
    yAxis: {
      type: 'value'
    },
    series: [
      {
        name: '收入',
        type: 'bar',
        data: incomeData,
        itemStyle: { color: '#67C23A' }
      },
      {
        name: '支出',
        type: 'bar',
        data: expenseData,
        itemStyle: { color: '#F56C6C' }
      }
    ]
  }
  barInstance.setOption(option)
}

onMounted(() => {
  initPieChart()
  initBarChart()
  updatePieChart()
  updateBarChart()
})

// 监听数据变化，更新图表
watch(() => props.records, () => {
  updatePieChart()
  updateBarChart()
}, { deep: true })

onBeforeUnmount(() => {
  if (pieInstance) pieInstance.dispose()
  if (barInstance) barInstance.dispose()
})
</script>
