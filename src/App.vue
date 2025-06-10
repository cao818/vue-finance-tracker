<template>
  <div style="padding: 24px">
    <div style="margin-bottom: 16px; display: flex; align-items: center;">
      <el-button type="primary" @click="dialogVisible = true" style="margin-right: 20px;">
        添加记录
      </el-button>

      <el-button @click="exportData" style="margin-right: 10px;">
        导出数据
      </el-button>
      <el-upload
        :show-file-list="false"
        accept=".json"
        :before-upload="handleImport"
      >
        <el-button>导入数据</el-button>
      </el-upload>

      <el-select v-model="filters.type" placeholder="全部类型" clearable style="width: 120px; margin-right: 10px;">
        <el-option label="全部" value="" />
        <el-option label="收入" value="income" />
        <el-option label="支出" value="expense" />
      </el-select>

      <el-input
        v-model="filters.category"
        placeholder="分类关键词"
        clearable
        style="width: 160px; margin-right: 10px;"
      />

      <el-date-picker
        v-model="filters.dateRange"
        type="daterange"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
        clearable
        style="margin-right: 10px;"
      />
    </div>

    <!-- 统计信息 -->
    <div style="display: flex; gap: 20px; margin-bottom: 24px;">
      <el-card shadow="hover" style="flex: 1; text-align: center;">
        <div style="font-size: 20px; color: green;">总收入</div>
        <div style="font-size: 24px; font-weight: bold;">¥ {{ totalIncome }}</div>
      </el-card>
      <el-card shadow="hover" style="flex: 1; text-align: center;">
        <div style="font-size: 20px; color: red;">总支出</div>
        <div style="font-size: 24px; font-weight: bold;">¥ {{ totalExpense }}</div>
      </el-card>
      <el-card shadow="hover" style="flex: 1; text-align: center;">
        <div style="font-size: 20px; color: #333;">结余</div>
        <div style="font-size: 24px; font-weight: bold;">¥ {{ balance }}</div>
      </el-card>
    </div>
    <!-- 新增图表组件 -->
    <charts-view :records="filteredRecords" />
    <!-- 分类汇总 -->
    <div style="margin-bottom: 24px;">
      <h3>按分类统计（当前筛选）</h3>
      <el-table :data="categoryStats" style="width: 100%">
        <el-table-column prop="category" label="分类" />
        <el-table-column prop="type" label="类型" width="100">
          <template #default="{ row }">
            <el-tag :type="row.type === 'income' ? 'success' : 'danger'">
              {{ row.type === 'income' ? '收入' : '支出' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="amount" label="金额" />
      </el-table>
    </div>

    <record-list :records="filteredRecords" @delete="deleteRecord" />

    <add-record-dialog
      v-model:visible="dialogVisible"
      @save="addRecord"
    />
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import RecordList from './components/RecordList.vue'
import AddRecordDialog from './components/AddRecordDialog.vue'
import ChartsView from './components/ChartsView.vue'
import { ElMessage } from 'element-plus'

const dialogVisible = ref(false)
const records = ref([])

const filters = reactive({
  type: '',
  category: '',
  dateRange: []
})

onMounted(() => {
  const data = localStorage.getItem('records')
  records.value = data ? JSON.parse(data) : []
})

function addRecord(record) {
  record.id = Date.now().toString()
  records.value.push(record)
  localStorage.setItem('records', JSON.stringify(records.value))
}

function deleteRecord(id) {
  records.value = records.value.filter(r => r.id !== id)
  localStorage.setItem('records', JSON.stringify(records.value))
}

const filteredRecords = computed(() => {
  return records.value.filter(r => {
    if (filters.type && r.type !== filters.type) return false
    if (filters.category && !r.category.includes(filters.category)) return false
    if (filters.dateRange.length === 2) {
      const [start, end] = filters.dateRange
      const date = new Date(r.date)
      if (date < new Date(start) || date > new Date(end)) return false
    }
    return true
  })
})

// 统计总收入
const totalIncome = computed(() => {
  return filteredRecords.value
    .filter(r => r.type === 'income')
    .reduce((sum, r) => sum + Number(r.amount), 0)
})

// 统计总支出
const totalExpense = computed(() => {
  return filteredRecords.value
    .filter(r => r.type === 'expense')
    .reduce((sum, r) => sum + Number(r.amount), 0)
})

// 结余 = 收入 - 支出
const balance = computed(() => totalIncome.value - totalExpense.value)

// 按分类汇总
const categoryStats = computed(() => {
  const map = {}
  filteredRecords.value.forEach(r => {
    const key = r.type + '|' + r.category
    if (!map[key]) {
      map[key] = {
        type: r.type,
        category: r.category,
        amount: 0
      }
    }
    map[key].amount += Number(r.amount)
  })
  // 转数组并按类型分开排一下，收入先展示
  return Object.values(map).sort((a, b) => {
    if (a.type === b.type) return b.amount - a.amount
    return a.type === 'income' ? -1 : 1
  })
})
// 导出数据函数
function exportData() {
  const dataStr = JSON.stringify(records.value, null, 2)
  const blob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(blob)

  const a = document.createElement('a')
  a.href = url
  a.download = `records_backup_${Date.now()}.json`
  a.click()

  URL.revokeObjectURL(url)
}

// 导入数据函数（el-upload 的 before-upload 钩子）
function handleImport(file) {
  const reader = new FileReader()
  reader.onload = e => {
    try {
      const importedData = JSON.parse(e.target.result)
      if (!Array.isArray(importedData)) {
        ElMessage.error('导入失败：数据格式不正确，应该是数组')
        return
      }
      // 简单校验每条记录结构
      const valid = importedData.every(item =>
        item.id && item.type && item.category && item.amount && item.date
      )
      if (!valid) {
        ElMessage.error('导入失败：部分记录格式不正确')
        return
      }
      // 合并，去重（根据id）
      const existingIds = new Set(records.value.map(r => r.id))
      const filteredNew = importedData.filter(r => !existingIds.has(r.id))
      if (filteredNew.length === 0) {
        ElMessage.info('导入完成，无新增记录')
        return
      }
      records.value.push(...filteredNew)
      localStorage.setItem('records', JSON.stringify(records.value))
      ElMessage.success(`成功导入 ${filteredNew.length} 条记录`)
    } catch (error) {
      ElMessage.error('导入失败：文件内容解析错误')
    }
  }
  reader.readAsText(file)
  // 阻止 el-upload 组件自动上传文件
  return false
}
</script>
