<template>
  <div>
    <div v-if="!planetBySign.total" class="text-sm text-gray-500">Žádné planety k analýze.</div>

    <div v-else class="space-y-3">
      <div v-if="stellia.length">
        <div class="text-sm font-medium mb-1">Stellia (≥ 3 planety v jednom znamení)</div>
        <div class="flex flex-wrap gap-2">
          <span v-for="s in stellia" :key="s.sign"
                class="px-2 py-1 rounded-full text-xs font-medium bg-indigo-50 text-indigo-700 border border-indigo-100">
            {{ s.sign }} • {{ s.count }}
          </span>
        </div>
      </div>
      <div v-else class="text-sm text-gray-500">Žádné stellium nenalezeno.</div>

      <div>
        <div class="text-sm font-medium mb-1">Obsazenost znamení (planety)</div>
        <BarChart :data="chartData" suffix="" />
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

function parse(title) {
  if (!title) return null
  const pm = title.match(new RegExp(`\\b(${PLANETS.join('|')})\\b`, 'i'))
  const sm = title.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/i)
  if (!pm || !sm) return null
  let sign = sm[1].replace(/\.$/,'').trim()
  sign = signAlias[sign] || (sign[0]?.toUpperCase()+sign.slice(1))
  return { sign }
}

const planetBySign = computed(() => {
  const counts = {
    Aries:0,Taurus:0,Gemini:0,Cancer:0,Leo:0,Virgo:0,Libra:0,Scorpio:0,Sagittarius:0,Capricorn:0,Aquarius:0,Pisces:0
  }
  let total = 0
  for (const it of (props.items||[])) {
    const title = typeof it === 'string' ? it : it?.title
    const p = parse(title)
    if (p?.sign && counts[p.sign] !== undefined) { counts[p.sign]++; total++ }
  }
  const rows = Object.entries(counts).map(([sign,count]) => ({ sign, count }))
  rows.sort((a,b)=>b.count-a.count)
  return { counts, rows, total }
})

const stellia = computed(() => planetBySign.value.rows.filter(r => r.count >= 3))
const chartData = computed(() => planetBySign.value.rows.map(r => ({ label: r.sign, value: r.count })))
</script>
