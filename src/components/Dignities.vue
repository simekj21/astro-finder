<template>
  <div>
    <div v-if="!rows.length" class="text-sm text-gray-500">Nenalezeny planety v znameních.</div>

    <div v-else class="space-y-4">
      <!-- Síla planet -->
      <div>
        <div class="text-sm font-medium mb-1">Síla planet (součet výskytů + důstojnosti)</div>
        <BarChart :data="strengthChart" suffix="" :decimals="2" />
      </div>

      <!-- Detailní tabulka -->
      <div class="overflow-x-auto">
        <table class="min-w-full text-sm">
          <thead class="text-left text-slate-600">
            <tr>
              <th class="py-2 pr-3">Planeta</th>
              <th class="py-2 pr-3">Znamení</th>
              <th class="py-2 pr-3">Důstojnost</th>
              <th class="py-2 pr-3 text-right">Skóre</th>
            </tr>
          </thead>
          <tbody class="divide-y">
            <tr v-for="r in rows" :key="r.id">
              <td class="py-1 pr-3">{{ r.planet }}</td>
              <td class="py-1 pr-3">{{ r.sign }}</td>
              <td class="py-1 pr-3">
                <span
                  class="px-2 py-0.5 rounded text-xs"
                  :class="badgeClass(r.status)"
                >{{ r.status }}</span>
              </td>
              <td class="py-1 pr-3 text-right tabular-nums">{{ r.score.toFixed(2) }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="text-xs text-gray-500">
        Domicil +2, Exaltace +1.5, Exil −2, Pád −1.5, Neutrál 0. (Moderní transsaturny zde jako neutrální.)
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

/* --- Normalizace znamení (CZ -> EN) --- */
const signAlias = {
  'Beran':'Aries','Býk':'Taurus','Blíženci':'Gemini','Rak':'Cancer','Lev':'Leo','Panna':'Virgo',
  'Váhy':'Libra','Štír':'Scorpio','Střelec':'Sagittarius','Kozoroh':'Capricorn','Vodnář':'Aquarius','Ryby':'Pisces'
}
const opposites = {
  Aries:'Libra', Taurus:'Scorpio', Gemini:'Sagittarius', Cancer:'Capricorn',
  Leo:'Aquarius', Virgo:'Pisces', Libra:'Aries', Scorpio:'Taurus',
  Sagittarius:'Gemini', Capricorn:'Cancer', Aquarius:'Leo', Pisces:'Virgo'
}

const PLANETS = ['Sun','Moon','Mercury','Venus','Mars','Jupiter','Saturn','Uranus','Neptune','Pluto']

const domicile = {
  Sun:['Leo'], Moon:['Cancer'],
  Mercury:['Gemini','Virgo'],
  Venus:['Taurus','Libra'],
  Mars:['Aries','Scorpio'],
  Jupiter:['Sagittarius','Pisces'],
  Saturn:['Capricorn','Aquarius'],
}
const exaltation = {
  Sun:'Aries', Moon:'Taurus', Mercury:'Virgo', Venus:'Pisces',
  Mars:'Capricorn', Jupiter:'Cancer', Saturn:'Libra'
}

/* --- Parsování titulů "Planet in Sign" --- */
function parse(title) {
  if (!title) return null
  const pm = title.match(new RegExp(`\\b(${PLANETS.join('|')})\\b`, 'i'))
  const sm = title.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/i)
  if (!pm || !sm) return null
  let planet = pm[1][0].toUpperCase() + pm[1].slice(1).toLowerCase()
  let sign = sm[1].replace(/\.$/,'').trim()
  sign = signAlias[sign] || (sign[0]?.toUpperCase()+sign.slice(1))
  if (!opposites[sign]) return null
  return { planet, sign }
}

const entries = computed(() =>
  (props.items || [])
    .map(it => typeof it === 'string' ? it : it?.title)
    .filter(Boolean)
    .map(parse)
    .filter(Boolean)
)

/* --- Důstojnosti --- */
function dignityStatus(planet, sign) {
  if (domicile[planet]?.includes(sign)) return 'Domicil'
  if (exaltation[planet] === sign) return 'Exaltace'
  // Exil = opozit k domicilu
  const exil = Object.values(domicile).some(arr => arr.includes(opposites[sign] || '')) &&
               domicile[planet]?.includes(opposites[sign]) // specificky pro planetu
  if (exil) return 'Exil'
  // Pád = opozit k exaltaci
  if (exaltation[planet] && opposites[exaltation[planet]] === sign) return 'Pád'
  return 'Neutrál'
}
function dignityScore(status) {
  switch (status) {
    case 'Domicil': return 2
    case 'Exaltace': return 1.5
    case 'Exil': return -2
    case 'Pád': return -1.5
    default: return 0
  }
}

const rows = computed(() => {
  // sloučíme případné duplikáty (když by data posílala planetu víckrát)
  const map = new Map()
  for (const e of entries.value) {
    const key = `${e.planet}__${e.sign}`
    const status = dignityStatus(e.planet, e.sign)
    const score = dignityScore(status) + 1 // base 1 za výskyt
    const curr = map.get(e.planet) || { planet: e.planet, sign: e.sign, status, score: 0 }
    curr.score += score
    curr.sign = e.sign      // poslední výskyt vyhraje pro zobrazení
    curr.status = status
    map.set(e.planet, curr)
  }
  // planets not present -> skip
  return Array.from(map.values()).sort((a,b)=>b.score - a.score).map((r,i)=>({ ...r, id: i }))
})

const strengthChart = computed(() => rows.value.map(r => ({ label: r.planet, value: Math.max(0, r.score) })))

function badgeClass(s) {
  if (s==='Domicil') return 'bg-green-100 text-green-800'
  if (s==='Exaltace') return 'bg-emerald-100 text-emerald-800'
  if (s==='Exil') return 'bg-rose-100 text-rose-800'
  if (s==='Pád') return 'bg-orange-100 text-orange-800'
  return 'bg-slate-100 text-slate-700'
}
</script>
