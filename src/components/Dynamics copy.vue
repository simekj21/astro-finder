<template>
  <div>
    <div v-if="!items || !items.length" class="text-sm text-gray-500">Žádná data k analýze.</div>
    <div v-else>
      <PieChart :data="chartData" center="Modalita" :size="168" />
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import PieChart from './PieChart.vue'

const props = defineProps({
  items: { type: Array, default: () => [] }
})

const signToModality = {
  Aries: 'Cardinal', Cancer: 'Cardinal', Libra: 'Cardinal', Capricorn: 'Cardinal',
  Taurus: 'Fixed', Leo: 'Fixed', Scorpio: 'Fixed', Aquarius: 'Fixed',
  Gemini: 'Mutable', Virgo: 'Mutable', Sagittarius: 'Mutable', Pisces: 'Mutable',
  'Beran': 'Cardinal', 'Rak': 'Cardinal', 'Váhy': 'Cardinal', 'Kozoroh': 'Cardinal',
  'Býk': 'Fixed', 'Lev': 'Fixed', 'Štír': 'Fixed', 'Vodnář': 'Fixed',
  'Blíženci': 'Mutable', 'Panna': 'Mutable', 'Střelec': 'Mutable', 'Ryby': 'Mutable'
}

const titles = computed(() => props.items.map(it => typeof it === 'string' ? it : it?.title).filter(Boolean))

function extractSign(title) {
  const m = title?.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/)
  if (!m) return null
  let s = m[1].replace(/\.$/, '').trim()
  if (/^\d+\.?\s*House$/i.test(s)) return null
  return s.replace(/^([a-zá-ž])/, c => c.toUpperCase())
}

const chartData = computed(() => {
  const counts = { Cardinal: 0, Fixed: 0, Mutable: 0 }
  for (const t of titles.value) {
    const sign = extractSign(t)
    const mod = sign && signToModality[sign]
    if (mod) counts[mod]++
  }
  return Object.entries(counts)
    .map(([label, value]) => ({ label, value }))
    .sort((a,b)=>b.value-a.value)
})
</script>
