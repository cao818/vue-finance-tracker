<template>
  <el-dialog title="添加记录" :model-value="visible" @update:modelValue="$emit('update:visible', $event)">
    <el-form :model="form">
      <el-form-item label="类型">
        <el-radio-group v-model="form.type">
          <el-radio-button label="income">收入</el-radio-button>
          <el-radio-button label="expense">支出</el-radio-button>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="分类">
        <el-input v-model="form.category" placeholder="如：餐饮、交通等"/>
      </el-form-item>
      <el-form-item label="金额">
        <el-input-number v-model="form.amount" :min="0"/>
      </el-form-item>
      <el-form-item label="备注">
        <el-input v-model="form.remark"/>
      </el-form-item>
      <el-form-item label="日期">
        <el-date-picker v-model="form.date" type="date"/>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="$emit('update:visible', false)">取消</el-button>
      <el-button type="primary" @click="handleSave">保存</el-button>
    </template>
  </el-dialog>
</template>

<script setup>
import {reactive} from 'vue'

const props = defineProps(['visible'])
const emit = defineEmits(['update:visible', 'save'])

const form = reactive({
  type: 'expense',
  category: '',
  amount: 0,
  remark: '',
  date: ''
})

function handleSave() {
  if (!form.category || !form.amount || !form.date) return
  emit('save', {...form})
  emit('update:visible', false)
  Object.assign(form, {
    type: 'expense',
    category: '',
    amount: 0,
    remark: '',
    date: ''
  })
}
</script>
