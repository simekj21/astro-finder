<template>
  <div>
    <div v-if="!items || !items.length" class="text-sm text-gray-500">Žádná data k analýze.</div>

    <div v-else class="space-y-4">
      <div>
        <div class="text-sm font-medium mb-1">Dynamika znamení (celkově)</div>
        <PieChart :data="chartAll" center="Modalita" :size="168" />
      </div>

      <div>
        <div class="text-sm font-medium mb-1">Dynamika (jen osobní planety)</div>
        <PieChart :data="chartPersonal" center="Osobní" :size="168" />
        <div v-if="personalTotal === 0" class="text-xs text-gray-500 mt-1">
          Nebyly nalezeny žádné osobní planety (Sun–Mars) v seznamu.
        </div>
      </div>
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
  Aries:'Cardinal', Cancer:'Cardinal', Libra:'Cardinal', Capricorn:'Cardinal',
  Taurus:'Fixed', Leo:'Fixed', Scorpio:'Fixed', Aquarius:'Fixed',
  Gemini:'Mutable', Virgo:'Mutable', Sagittarius:'Mutable', Pisces:'Mutable',
  'Beran':'Cardinal','Rak':'Cardinal','Váhy':'Cardinal','Kozoroh':'Cardinal',
  'Býk':'Fixed','Lev':'Fixed','Štír':'Fixed','Vodnář':'Fixed',
  'Blíženci':'Mutable','Panna':'Mutable','Střelec':'Mutable','Ryby':'Mutable'
}
const PERSONAL = ['Sun','Moon','Mercury','Venus','Mars']

function extractSign(title) {
  const m = title?.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/)
  if (!m) return null
  const raw = m[1].replace(/\.$/,'').trim()
  if (/^\d+\.?\s*House$/i.test(raw)) return null
  return raw[0]?.toUpperCase() + raw.slice(1)
}
function extractPlanet(title) {
  const m = title?.match(/\b(Sun|Moon|Mercury|Venus|Mars|Jupiter|Saturn|Uranus|Neptune|Pluto)\b/i)
  return m ? m[1][0].toUpperCase() + m[1].slice(1).toLowerCase() : null
}

const titles = computed(() => (props.items||[]).map(it => typeof it==='string'? it : it?.title).filter(Boolean))

function counts(modFilter = null) {
  const c = { Cardinal:0, Fixed:0, Mutable:0 }
  let total = 0
  for (const t of titles.value) {
    const p = extractPlanet(t)
    if (modFilter && !modFilter(p)) continue
    const s = extractSign(t)
    const m = s && signToModality[s]
    if (!m) continue
    c[m]++; total++
  }
  return { c, total }
}

const all = computed(() => counts())
const personal = computed(() => counts(p => PERSONAL.includes(p || '')))

const chartAll = computed(() => [
  { label:'Cardinal', value: all.value.c.Cardinal },
  { label:'Fixed',    value: all.value.c.Fixed    },
  { label:'Mutable',  value: all.value.c.Mutable  },
].sort((a,b)=>b.value-a.value))

const chartPersonal = computed(() => [
  { label:'Cardinal', value: personal.value.c.Cardinal },
  { label:'Fixed',    value: personal.value.c.Fixed    },
  { label:'Mutable',  value: personal.value.c.Mutable  },
].sort((a,b)=>b.value-a.value))

const personalTotal = computed(() => personal.value.total)
</script>
