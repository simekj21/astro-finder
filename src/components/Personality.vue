<template>
  <div>
    <div v-if="!items || !items.length" class="text-sm text-gray-500">Žádná data k analýze.</div>

    <div v-else class="space-y-6">
      <!-- Introvert / Extrovert -->
      <div>
        <div class="text-sm font-medium mb-1">Introvert / Extrovert</div>
        <PieChart :data="introExtraChart" center="I/E" :size="168" />
      </div>

      <!-- Temperamenty (česky) -->
      <div>
        <div class="text-sm font-medium mb-1">Temperamenty</div>
        <PieChart :data="temperamentChartCz" center="Temp." :size="168" />
      </div>

      <!-- Enneagram (všechny typy) -->
      <div>
        <div class="text-sm font-medium mb-1">Enneagram (heuristika)</div>
        <div class="text-xs text-gray-500 mb-2">
          Odhad z poměru živlů a modalit, s váhami planet a pravidlovými boosty. Zobrazeno všech 9 typů.
        </div>
        <BarChart :data="enneagramChartData" suffix="%" :decimals="0" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import PieChart from './PieChart.vue'
import BarChart from './BarChart.vue'

const props = defineProps({
  items: { type: Array, default: () => [] },
  includeHouses: { type: Boolean, default: true },
  usePlanetWeights: { type: Boolean, default: true },
  dominanceBoostK: { type: Number, default: 0.25 }
})

/* ===== Váhy ===== */
const PLANET_WEIGHTS = {
  Sun: 1.5, Moon: 1.4,
  Mercury: 1.0, Venus: 1.0, Mars: 1.0,
  Jupiter: 0.85, Saturn: 0.85,
  Uranus: 0.6, Neptune: 0.6, Pluto: 0.6
}
const DEFAULT_PLANET_WEIGHT = 0.8
const HOUSE_WEIGHT = 0.35

/* ===== Mapy znamení ===== */
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
const signToModality = {
  Aries: 'Cardinal', Cancer: 'Cardinal', Libra: 'Cardinal', Capricorn: 'Cardinal',
  Taurus: 'Fixed', Leo: 'Fixed', Scorpio: 'Fixed', Aquarius: 'Fixed',
  Gemini: 'Mutable', Virgo: 'Mutable', Sagittarius: 'Mutable', Pisces: 'Mutable',
  'Beran': 'Cardinal', 'Rak': 'Cardinal', 'Váhy': 'Cardinal', 'Kozoroh': 'Cardinal',
  'Býk': 'Fixed', 'Lev': 'Fixed', 'Štír': 'Fixed', 'Vodnář': 'Fixed',
  'Blíženci': 'Mutable', 'Panna': 'Mutable', 'Střelec': 'Mutable', 'Ryby': 'Mutable'
}

/* ===== Parsování ===== */
function extractSign(title) {
  const m = title?.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/)
  if (!m) return null
  let s = m[1].replace(/\.$/, '').trim()
  if (/^\d+\.?\s*House$/i.test(s)) return null
  return s.replace(/^([a-zá-ž])/, (c) => c.toUpperCase())
}
function extractPlanet(title) {
  const planets = Object.keys(PLANET_WEIGHTS).join('|')
  const m = title?.match(new RegExp(`\\b(${planets})\\b`, 'i'))
  return m ? m[1][0].toUpperCase() + m[1].slice(1).toLowerCase() : null
}
function isHouseOccupancy(title) {
  return /\b(\d+\.?\s*House|House\s*\d+|Dům\s*\d+)\b/i.test(title || '')
}

/* ===== Vstupní položky -> vážené záznamy ===== */
const titles = computed(() => props.items.map(it => typeof it === 'string' ? it : it?.title).filter(Boolean))

const counted = computed(() => {
  return titles.value.map(t => {
    const sign = extractSign(t)
    const planet = extractPlanet(t)
    const houseFlag = isHouseOccupancy(t)

    let weight = DEFAULT_PLANET_WEIGHT * 0.8
    if (planet && props.usePlanetWeights) weight = PLANET_WEIGHTS[planet] ?? DEFAULT_PLANET_WEIGHT
    else if (houseFlag) weight = props.includeHouses ? HOUSE_WEIGHT : 0
    else if (planet && !props.usePlanetWeights) weight = 1

    return { title: t, sign, planet, weight }
  }).filter(x => !!x.sign && x.weight > 0)
})

/* ===== Intro / Extra ===== */
const introExtra = computed(() => {
  let ext = 0, intr = 0, total = 0
  for (const x of counted.value) {
    const el = signToElement[x.sign]
    if (!el) continue
    total += x.weight
    if (el === 'Fire' || el === 'Air') ext += x.weight
    else intr += x.weight
  }
  if (!total) return { extrovert: 0, introvert: 0 }
  return { extrovert: (ext/total)*100, introvert: (intr/total)*100 }
})

/* ===== Temperamenty ===== */
const temperamentMap = { Fire: 'Choleric', Air: 'Sanguine', Earth: 'Melancholic', Water: 'Phlegmatic' }
const temperamentOrder = ['Choleric', 'Sanguine', 'Melancholic', 'Phlegmatic']
const temperamentCz = { Choleric: 'Cholerik', Sanguine: 'Sangvinik', Melancholic: 'Melancholik', Phlegmatic: 'Flegmatik' }

const temperamentStats = computed(() => {
  const counts = { Choleric: 0, Sanguine: 0, Melancholic: 0, Phlegmatic: 0 }
  let total = 0
  for (const x of counted.value) {
    const el = signToElement[x.sign]
    const temp = temperamentMap[el]
    if (temp) { counts[temp] += x.weight; total += x.weight }
  }
  const rows = temperamentOrder.map(key => {
    const val = counts[key]
    const percent = total ? (val / total) * 100 : 0
    return { key, value: val, percent }
  })
  return { counts, total, rows }
})
const sortedTemperamentsCz = computed(() =>
  [...temperamentStats.value.rows]
    .sort((a,b)=>b.value-a.value)
    .map(r => ({ ...r, label: temperamentCz[r.key] }))
)

/* ===== Enneagram ===== */
const enneagramDefs = [
  { id: 1, nameCz: 'Reformátor (Perfekcionista)' },
  { id: 2, nameCz: 'Pomáhající' },
  { id: 3, nameCz: 'Dosahovač (Výkonný)' },
  { id: 4, nameCz: 'Individualista (Romantik)' },
  { id: 5, nameCz: 'Badatel (Pozorovatel)' },
  { id: 6, nameCz: 'Loajalista (Skeptik)' },
  { id: 7, nameCz: 'Nadšenec' },
  { id: 8, nameCz: 'Vyzývatel (Vůdce)' },
  { id: 9, nameCz: 'Mírotvůrce (Mediátor)' },
]
const elementToTypes = {
  Fire: [3,7,8], Earth: [1,5,6,9], Air: [2,3,5,7], Water: [2,4,6,9],
}
const modalityToTypes = {
  Cardinal: [1,2,3,8], Fixed: [1,4,8,9], Mutable:[5,6,7,9],
}

const elementPerc = computed(() => {
  const counts = { Fire:0, Earth:0, Air:0, Water:0 }
  let total = 0
  for (const x of counted.value) {
    const el = signToElement[x.sign]
    if (el) { counts[el] += x.weight; total += x.weight }
  }
  const rows = Object.keys(counts).map(k => ({ name:k, value:counts[k], percent: total ? counts[k]/total*100 : 0 }))
  rows.sort((a,b)=>b.value-a.value)
  return rows
})
const modalityPerc = computed(() => {
  const counts = { Cardinal:0, Fixed:0, Mutable:0 }
  let total = 0
  for (const x of counted.value) {
    const mod = signToModality[x.sign]
    if (mod) { counts[mod] += x.weight; total += x.weight }
  }
  const rows = Object.keys(counts).map(k => ({ name:k, value:counts[k], percent: total ? counts[k]/total*100 : 0 }))
  rows.sort((a,b)=>b.value-a.value)
  return rows
})

const enneagramAll = computed(() => {
  const scores = { 1:0,2:0,3:0,4:0,5:0,6:0,7:0,8:0,9:0 }
  for (const x of counted.value) {
    const el = signToElement[x.sign]
    const mod = signToModality[x.sign]
    if (el && elementToTypes[el]) for (const t of elementToTypes[el]) scores[t] += 1.0 * x.weight
    if (mod && modalityToTypes[mod]) for (const t of modalityToTypes[mod]) scores[t] += 0.8 * x.weight
  }
  const k = Math.max(0, Math.min(1, props.dominanceBoostK))
  if (elementPerc.value.length) {
    const [top, second={percent:0}] = elementPerc.value
    const diff = Math.max(0, top.percent - second.percent) * k
    for (const t of (elementToTypes[top.name] || [])) scores[t] += diff
  }
  if (modalityPerc.value.length) {
    const [top, second={percent:0}] = modalityPerc.value
    const diff = Math.max(0, top.percent - second.percent) * k
    for (const t of (modalityToTypes[top.name] || [])) scores[t] += diff
  }
  const total = Object.values(scores).reduce((a,b)=>a+b,0)
  return enneagramDefs.map(def => ({
    id: def.id, nameCz: def.nameCz,
    value: scores[def.id],
    percent: total ? (scores[def.id]/total)*100 : 0
  })).sort((a,b)=>b.value-a.value)
})

/* ===== Data pro grafy ===== */
const introExtraChart = computed(() => ([
  { label: 'Extrovert', value: introExtra.value?.extrovert || 0 },
  { label: 'Introvert', value: introExtra.value?.introvert || 0 },
]))
const temperamentChartCz = computed(() =>
  (sortedTemperamentsCz.value || []).map(r => ({ label: r.label, value: r.value }))
)
const enneagramChartData = computed(() =>
  (enneagramAll.value || []).map(r => ({ label: `Typ ${r.id} — ${r.nameCz}`, value: r.percent }))
)
</script>
