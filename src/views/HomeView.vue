<script setup lang="ts">
import MultiSelect from '@/components/MultiSelect.vue'
import { ref } from 'vue'

const options = [
  { value: 'option1', name: '选项一' },
  { value: 'option2', name: '选项二' },
  { value: 'option3', name: '选项三' },
  { value: 'option4', name: '选项四' },
  { value: 'option5', name: '选项五' },
]
const selectedOptions = ref<string[]>([])
</script>

<template>
  <main class="mt-30">
    <h1 class="text-center text-2xl font-bold">多选下拉菜单</h1>
    <MultiSelect
      class="max-w-md mx-auto mt-10"
      v-model="selectedOptions"
      :options="options"
      id="multi-select"
      name="multi-select"
      required
      placeholder="请选择"
    />
  </main>
</template>
