<template>
  <div>
    <div v-if="!items || !items.length" class="text-sm text-gray-500">Žádná data k analýze.</div>
    <div v-else>
      <PieChart :data="chartData" center="Živly" :size="168" />
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import PieChart from './PieChart.vue'

const props = defineProps({
  items: { type: Array, default: () => [] }
})

const signToElement = {
  Aries: 'Fire', Leo: 'Fire', Sagittarius: 'Fire',
  Taurus: 'Earth', Virgo: 'Earth', Capricorn: 'Earth',
  Gemini: 'Air', Libra: 'Air', Aquarius: 'Air',
  Cancer: 'Water', Scorpio: 'Water', Pisces: 'Water',
  'Beran': 'Fire', 'Lev': 'Fire', 'Střelec': 'Fire',
  'Býk': 'Earth', 'Panna': 'Earth', 'Kozoroh': 'Earth',
  'Blíženci': 'Air', 'Váhy': 'Air', 'Vodnář': 'Air',
  'Rak': 'Water', 'Štír': 'Water', 'Ryby': 'Water'
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
  const counts = { Fire: 0, Earth: 0, Air: 0, Water: 0 }
  for (const t of titles.value) {
    const sign = extractSign(t)
    const el = sign && signToElement[sign]
    if (el) counts[el]++
  }
  return Object.entries(counts)
    .map(([label, value]) => ({ label, value }))
    .sort((a,b)=>b.value-a.value)
})
</script>
