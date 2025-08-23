
<template>
  <div>
    <div v-if="!items || !items.length" class="text-sm text-gray-500">Žádná data k analýze.</div>
    <ul v-else class="text-sm space-y-1">
      <li v-for="row in sortedElements" :key="row.name" class="flex items-center justify-between">
        <span class="font-medium">{{ row.name }}</span>
        <span>{{ row.percent.toFixed(0) }}%</span>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  // očekává pole stringů nebo objektů { title: string }
  items: { type: Array, default: () => [] }
})

// Mapování znamení -> živel
const signToElement = {
  Aries: 'Fire', Leo: 'Fire', Sagittarius: 'Fire',
  Taurus: 'Earth', Virgo: 'Earth', Capricorn: 'Earth',
  Gemini: 'Air', Libra: 'Air', Aquarius: 'Air',
  Cancer: 'Water', Scorpio: 'Water', Pisces: 'Water',
  // české varianty pro jistotu
  'Beran': 'Fire', 'Lev': 'Fire', 'Střelec': 'Fire',
  'Býk': 'Earth', 'Panna': 'Earth', 'Kozoroh': 'Earth',
  'Blíženci': 'Air', 'Váhy': 'Air', 'Vodnář': 'Air',
  'Rak': 'Water', 'Štír': 'Water', 'Ryby': 'Water'
}

const elementOrder = ['Fire', 'Earth', 'Air', 'Water']

// vytáhneme tituly (stringy)
const titles = computed(() => props.items.map((it) => typeof it === 'string' ? it : it?.title).filter(Boolean))

// detekce znamení z titulku "X in Capricorn" nebo "1. House in Capricorn" apod.
function extractSign(title) {
  // vzory: "... in Capricorn", "... v Kozoroh", "in 1. House" (ten ignorujeme)
  // vezmeme poslední slovo/segment po " in " nebo " v "
  const m = title.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/)
  if (!m) return null
  let signRaw = m[1].replace(/\.$/, '').trim()
  // U českých variant může být tečka po číslovce – to není znamení
  // Vyloučíme případy typu "1. House"
  if (/^\d+\.?\s*House$/i.test(signRaw)) return null

  // normalizace prvního písmena
  signRaw = signRaw.replace(/^([a-zá-ž])/, (c) => c.toUpperCase())
  return signRaw
}

// které typy počítat: 1) Planety ve znamení, 2) Dům ve znamení (obsazenost)
const counted = computed(() => {
  return titles.value
    .map(t => ({ title: t, sign: extractSign(t) }))
    .filter(x => !!x.sign)
})

const elementStats = computed(() => {
  const counts = { Fire: 0, Earth: 0, Air: 0, Water: 0 }
  let total = 0

  for (const x of counted.value) {
    const el = signToElement[x.sign]
    if (el) {
      counts[el]++
      total++
    }
  }
  const rows = elementOrder.map(name => {
    const val = counts[name]
    const percent = total ? (val / total) * 100 : 0
    return { name, value: val, percent }
  })
  return { counts, total, rows }
})

const sortedElements = computed(() => {
  return [...elementStats.value.rows].sort((a,b) => b.value - a.value)
})

defineExpose({ elementStats, sortedElements })
</script>
