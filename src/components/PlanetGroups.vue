<template>
  <div>
    <div v-if="total === 0" class="text-sm text-gray-500">Žádná data k analýze.</div>
    <div v-else>
      <BarChart :data="chartPct" suffix="%" :decimals="0" />
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import BarChart from './BarChart.vue'

const props = defineProps({
  items: { type: Array, default: () => [] }
})

const PERSONAL = ['Sun','Moon','Mercury','Venus','Mars']
const SOCIAL = ['Jupiter','Saturn']
const TRANSP = ['Uranus','Neptune','Pluto']

function extractPlanet(title) {
  const all = [...PERSONAL, ...SOCIAL, ...TRANSP].join('|')
  const m = title?.match(new RegExp(`\\b(${all})\\b`, 'i'))
  return m ? m[1][0].toUpperCase() + m[1].slice(1).toLowerCase() : null
}

const titles = computed(() => (props.items||[]).map(it => typeof it==='string'? it : it?.title).filter(Boolean))

const counts = computed(() => {
  let personal = 0, social = 0, trans = 0, total = 0
  for (const t of titles.value) {
    const p = extractPlanet(t)
    if (!p) continue
    total++
    if (PERSONAL.includes(p)) personal++
    else if (SOCIAL.includes(p)) social++
    else if (TRANSP.includes(p)) trans++
  }
  return { personal, social, trans, total }
})

const total = computed(() => counts.value.total)
const chartPct = computed(() => {
  const { personal, social, trans, total } = counts.value
  const pct = (v) => total ? (v / total) * 100 : 0
  return [
    { label: 'Osobní (Sun–Mars)', value: pct(personal) },
    { label: 'Sociální (Jup–Sat)', value: pct(social) },
    { label: 'Transpersonální (Ura–Plu)', value: pct(trans) },
  ]
})
</script>
