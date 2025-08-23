<template>
  <div>
    <div v-if="total === 0" class="text-sm text-gray-500">Žádná data k analýze.</div>
    <div v-else>
      <PieChart :data="chartData" center="Polarita" :size="168" />
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
  Aries:'Fire', Leo:'Fire', Sagittarius:'Fire',
  Taurus:'Earth', Virgo:'Earth', Capricorn:'Earth',
  Gemini:'Air', Libra:'Air', Aquarius:'Air',
  Cancer:'Water', Scorpio:'Water', Pisces:'Water',
  'Beran':'Fire','Lev':'Fire','Střelec':'Fire',
  'Býk':'Earth','Panna':'Earth','Kozoroh':'Earth',
  'Blíženci':'Air','Váhy':'Air','Vodnář':'Air',
  'Rak':'Water','Štír':'Water','Ryby':'Water'
}

function extractSign(title) {
  const m = title?.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/)
  if (!m) return null
  const raw = m[1].replace(/\.$/,'').trim()
  if (/^\d+\.?\s*House$/i.test(raw)) return null
  const s = raw[0]?.toUpperCase() + raw.slice(1)
  return s
}

const titles = computed(() => (props.items||[]).map(it => typeof it==='string'? it : it?.title).filter(Boolean))

const counts = computed(() => {
  let yang = 0, yin = 0
  for (const t of titles.value) {
    const sign = extractSign(t)
    const el = sign && signToElement[sign]
    if (!el) continue
    if (el === 'Fire' || el === 'Air') yang++
    else yin++
  }
  return { yang, yin, total: yang+yin }
})

const total = computed(() => counts.value.total)
const chartData = computed(() => [
  { label: 'Maskulinní (Oheň+Vzduch)', value: counts.value.yang },
  { label: 'Feminní (Země+Voda)', value: counts.value.yin },
])
</script>
