<template>
  <div class="flex items-center gap-4">
    <svg
      viewBox="0 0 40 40"
      class="shrink-0"
      :style="{ width: size + 'px', height: size + 'px' }"
    >
      <!-- background ring (malá rezerva proti ořezu) -->
      <circle cx="20" cy="20" r="16" fill="none" stroke="#eee" stroke-width="6" pathLength="100" />
      <!-- segments -->
      <g v-for="(s, i) in segments" :key="i">
        <circle
          cx="20" cy="20" r="16" fill="none"
          :stroke="s.color" stroke-width="6"
          pathLength="100"
          :stroke-dasharray="`${s.pct} ${100 - s.pct}`"
          :stroke-dashoffset="s.offset"
          stroke-linecap="butt"
        />
      </g>
      <!-- center hole (donut) -->
      <circle cx="20" cy="20" r="11" fill="white" />
      <!-- label -->
      <text x="20" y="21" text-anchor="middle" font-size="3.6" fill="#334155">
        {{ centerLabel }}
      </text>
    </svg>

    <ul class="text-sm space-y-1">
      <li v-for="row in pretty" :key="row.label" class="flex items-center gap-2">
        <span class="inline-block w-3 h-3 rounded" :style="{ background: row.color }"></span>
        <span class="min-w-[5rem]">{{ row.label }}</span>
        <span class="tabular-nums">{{ row.percent.toFixed(0) }}%</span>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  // [{ label, value, color? }]
  data: { type: Array, required: true },
  center: { type: String, default: '' },
  // velikost SVG v pixelech
  size: { type: Number, default: 144 }
})

const palette = ['#ef4444', '#10b981', '#3b82f6', '#f59e0b', '#8b5cf6', '#14b8a6', '#f97316', '#84cc16', '#06b6d4']

const total = computed(() => props.data.reduce((a,b)=> a + (Number(b.value)||0), 0))

const pretty = computed(() => {
  let i = 0
  return props.data.map(d => {
    const color = d.color || palette[i++ % palette.length]
    const percent = total.value ? (Number(d.value)||0) / total.value * 100 : 0
    return { ...d, color, percent }
  })
})

// donut segments v procentech (díky pathLength="100")
const segments = computed(() => {
  let acc = 25 // začátek nahoře (12h)
  return pretty.value.map(p => {
    const pct = Math.max(0, Math.min(100, p.percent))
    const offset = 100 - acc
    acc += pct
    return { pct, offset, color: p.color }
  })
})

const centerLabel = computed(() => props.center || `${Math.round(total.value)}`)
</script>
