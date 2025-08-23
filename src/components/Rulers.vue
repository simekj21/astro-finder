<template>
  <div>
    <div v-if="!countsClassic.total" class="text-sm text-gray-500">Žádné planety k analýze.</div>

    <div v-else class="grid md:grid-cols-2 gap-4">
      <div>
        <div class="text-sm font-medium mb-1">Vládci (klasicky)</div>
        <BarChart :data="classicChart" suffix="" />
      </div>
      <div>
        <div class="text-sm font-medium mb-1">Vládci (moderně)</div>
        <BarChart :data="modernChart" suffix="" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import BarChart from './BarChart.vue'

const props = defineProps({
  items: { type: Array, default: () => [] }
})

const signAlias = {
  'Beran':'Aries','Býk':'Taurus','Blíženci':'Gemini','Rak':'Cancer','Lev':'Leo','Panna':'Virgo',
  'Váhy':'Libra','Štír':'Scorpio','Střelec':'Sagittarius','Kozoroh':'Capricorn','Vodnář':'Aquarius','Ryby':'Pisces'
}
const PLANETS = ['Sun','Moon','Mercury','Venus','Mars','Jupiter','Saturn','Uranus','Neptune','Pluto']

const classicRuler = {
  Aries:'Mars', Taurus:'Venus', Gemini:'Mercury', Cancer:'Moon', Leo:'Sun', Virgo:'Mercury',
  Libra:'Venus', Scorpio:'Mars', Sagittarius:'Jupiter', Capricorn:'Saturn', Aquarius:'Saturn', Pisces:'Jupiter'
}
const modernRuler = {
  ...classicRuler,
  Scorpio:'Pluto', Aquarius:'Uranus', Pisces:'Neptune'
}

function parseSign(title) {
  if (!title) return null
  if (!new RegExp(`\\b(${PLANETS.join('|')})\\b`, 'i').test(title)) return null
  const sm = title.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/i)
  if (!sm) return null
  let sign = sm[1].replace(/\.$/,'').trim()
  sign = signAlias[sign] || (sign[0]?.toUpperCase()+sign.slice(1))
  return sign
}

function countRulers(mapper) {
  const counts = {}
  let total = 0
  for (const it of (props.items||[])) {
    const title = typeof it === 'string' ? it : it?.title
    const sign = parseSign(title)
    const ruler = sign && mapper[sign]
    if (ruler) { counts[ruler] = (counts[ruler]||0)+1; total++ }
  }
  const rows = Object.entries(counts).map(([label,value]) => ({ label, value })).sort((a,b)=>b.value-a.value)
  return { counts, rows, total }
}

const countsClassic = computed(() => countRulers(classicRuler))
const countsModern  = computed(() => countRulers(modernRuler))

const classicChart = computed(() => countsClassic.value.rows)
const modernChart  = computed(() => countsModern.value.rows)
</script>
