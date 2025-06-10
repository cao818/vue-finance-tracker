<template>
  <el-table :data="records" style="width: 100%">
    <el-table-column prop="date" label="日期" width="120" />
    <el-table-column prop="type" label="类型" width="100">
      <template #default="{ row }">
        <el-tag :type="row.type === 'income' ? 'success' : 'danger'">
          {{ row.type === 'income' ? '收入' : '支出' }}
        </el-tag>
      </template>
    </el-table-column>
    <el-table-column prop="category" label="分类" />
    <el-table-column prop="amount" label="金额" />
    <el-table-column prop="remark" label="备注" />
    <el-table-column label="操作" width="80">
      <template #default="{ row }">
        <el-button type="danger" size="small" @click="$emit('delete', row.id)">删除</el-button>
      </template>
    </el-table-column>
  </el-table>
</template>

<script setup>
defineProps(['records'])
defineEmits(['delete'])
</script>
