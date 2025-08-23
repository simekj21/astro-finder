<template>
  <div class="space-y-2">
    <div v-for="row in pretty" :key="row.label">
      <div class="flex items-center justify-between text-sm">
        <div class="flex items-center gap-2">
          <span class="inline-block w-3 h-3 rounded" :style="{ background: row.color }"></span>
          <span>{{ row.label }}</span>
        </div>
        <span class="tabular-nums">
          {{ row.value.toFixed(decimals) }}{{ suffix }}
        </span>
      </div>
      <div class="h-2 rounded bg-slate-100 overflow-hidden">
        <div class="h-full rounded" :style="{ width: row.norm + '%', background: row.color }"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  // data: [{ label: string, value: number, color?: string }]
  data: { type: Array, required: true },
  // pokud hodnoty už jsou v procentech, nech suffix="%" a bars pojedou 1:1
  suffix: { type: String, default: '%' },
  decimals: { type: Number, default: 0 }
})

const palette = ['#ef4444', '#10b981', '#3b82f6', '#f59e0b', '#8b5cf6', '#14b8a6', '#f97316', '#84cc16', '#06b6d4']

const maxVal = computed(() => {
  const vals = props.data.map(d => Number(d.value) || 0)
  return Math.max(1, ...vals)
})

const pretty = computed(() => {
  let i = 0
  const isPercent = props.suffix.trim() === '%'
  return props.data.map(d => {
    const color = d.color || palette[i++ % palette.length]
    const value = Number(d.value) || 0
    const norm = Math.max(0, Math.min(100, isPercent ? value : (value / maxVal.value) * 100))
    return { ...d, color, value, norm }
  })
})
</script>
