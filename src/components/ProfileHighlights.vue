<template>
  <div>
    <div v-if="highlights.length === 0" class="text-sm text-gray-500">Zatím žádné zvýrazněné rysy.</div>
    <div v-else class="grid md:grid-cols-2 gap-2">
      <div v-for="(h, i) in top5" :key="i" class="rounded border bg-white px-3 py-2 text-sm">
        <div class="font-medium">{{ h.title }}</div>
        <div class="text-slate-600 text-xs mt-0.5" v-if="h.note">{{ h.note }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  items: { type: Array, default: () => [] }
})

/* --- mapy --- */
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
const signToModality = {
  Aries:'Cardinal', Cancer:'Cardinal', Libra:'Cardinal', Capricorn:'Cardinal',
  Taurus:'Fixed', Leo:'Fixed', Scorpio:'Fixed', Aquarius:'Fixed',
  Gemini:'Mutable', Virgo:'Mutable', Sagittarius:'Mutable', Pisces:'Mutable',
  'Beran':'Cardinal','Rak':'Cardinal','Váhy':'Cardinal','Kozoroh':'Cardinal',
  'Býk':'Fixed','Lev':'Fixed','Štír':'Fixed','Vodnář':'Fixed',
  'Blíženci':'Mutable','Panna':'Mutable','Střelec':'Mutable','Ryby':'Mutable'
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

function extractSign(title) {
  const m = title?.match(/\b(?:in|v)\s+([A-Za-zÁ-ž\.]+(?:\s*[A-Za-zÁ-ž]+)*)$/)
  if (!m) return null
  const raw = m[1].replace(/\.$/,'').trim()
  if (/^\d+\.?\s*House$/i.test(raw)) return null
  return raw[0]?.toUpperCase() + raw.slice(1)
}
function extractPlanet(title) {
  const m = title?.match(new RegExp(`\\b(${PLANETS.join('|')})\\b`, 'i'))
  return m ? m[1][0].toUpperCase() + m[1].slice(1).toLowerCase() : null
}
const titles = computed(() => (props.items||[]).map(it => typeof it==='string'? it : it?.title).filter(Boolean))

/* --- statistiky elementů a modalit --- */
const elementStats = computed(() => {
  const c = { Fire:0, Earth:0, Air:0, Water:0 }, total = { v:0 }
  for (const t of titles.value) {
    const s = extractSign(t); const el = s && signToElement[s]
    if (el) { c[el]++; total.v++ }
  }
  const rows = Object.keys(c).map(k => ({ k, v:c[k], p: total.v ? (c[k]/total.v)*100 : 0 }))
  rows.sort((a,b)=>b.v-a.v)
  return { rows, total: total.v }
})
const modalityStats = computed(() => {
  const c = { Cardinal:0, Fixed:0, Mutable:0 }, total = { v:0 }
  for (const t of titles.value) {
    const s = extractSign(t); const m = s && signToModality[s]
    if (m) { c[m]++; total.v++ }
  }
  const rows = Object.keys(c).map(k => ({ k, v:c[k], p: total.v ? (c[k]/total.v)*100 : 0 }))
  rows.sort((a,b)=>b.v-a.v)
  return { rows, total: total.v }
})

/* --- důstojnosti pro highlight --- */
function dignityStatus(planet, sign) {
  if (domicile[planet]?.includes(sign)) return 'Domicil'
  if (exaltation[planet] === sign) return 'Exaltace'
  return null
}
const dignityHighlights = computed(() => {
  const out = []
  for (const t of titles.value) {
    const p = extractPlanet(t); const s = extractSign(t)
    if (!p || !s) continue
    const st = dignityStatus(p, s)
    if (st) out.push({ planet:p, sign:s, status:st })
  }
  return out
})

/* --- generátor výroků --- */
const pairText = {
  'Fire+Cardinal':  ['Akční tah na branku', 'Oheň + Kardinalita'],
  'Fire+Fixed':     ['Vytrvalá vášeň a vůdcovství', 'Oheň + Fixní'],
  'Fire+Mutable':   ['Spontánnost a hravost', 'Oheň + Pohyblivá'],
  'Earth+Cardinal': ['Praktické vedení', 'Země + Kardinalita'],
  'Earth+Fixed':    ['Stabilita a vytrvalost', 'Země + Fixní'],
  'Earth+Mutable':  ['Praktičnost a adaptace', 'Země + Pohyblivá'],
  'Air+Cardinal':   ['Komunikativní leadership', 'Vzduch + Kardinalita'],
  'Air+Fixed':      ['Mentální stálost', 'Vzduch + Fixní'],
  'Air+Mutable':    ['Zvídavost a networking', 'Vzduch + Pohyblivá'],
  'Water+Cardinal': ['Pečující iniciativa', 'Voda + Kardinalita'],
  'Water+Fixed':    ['Hloubka a loajalita', 'Voda + Fixní'],
  'Water+Mutable':  ['Empatie a přizpůsobení', 'Voda + Pohyblivá'],
}
const planetThemes = {
  Sun:'identita/ego', Moon:'emoce', Mercury:'myšlení/komunikace', Venus:'vztahy/estetika',
  Mars:'energie/akce', Jupiter:'expanze/víra', Saturn:'struktura/disciplína'
}

const highlights = computed(() => {
  const out = []

  // 1) Kombinace top element + top modalita
  const el = elementStats.value.rows[0]
  const md = modalityStats.value.rows[0]
  if (el && md) {
    const key = `${el.k}+${md.k}`
    const txt = pairText[key]
    if (txt) out.push({ title: txt[0], note: txt[1] })
  }

  // 2) Polaritní převaha
  if (elementStats.value.total > 0) {
    const yang = (['Fire','Air'].includes(elementStats.value.rows[0]?.k) ? elementStats.value.rows[0].p : 100 - elementStats.value.rows[0].p)
    const diff = Math.abs(yang - 50)
    if (diff >= 20) {
      out.push({ title: yang > 50 ? 'Více jang (mask.)' : 'Více jin (fem.)', note: 'Oheň+Vzduch vs. Země+Voda' })
    }
  }

  // 3–4) Důstojnosti – pozitivní (domicil/exaltace)
  const seenPlanets = new Set()
  for (const d of dignityHighlights.value) {
    if (seenPlanets.has(d.planet)) continue
    seenPlanets.add(d.planet)
    const theme = planetThemes[d.planet] || 'téma planety'
    out.push({ title: `${d.planet} — ${d.status}`, note: `Posílené ${theme}` })
    if (out.length >= 4) break
  }

  // 5) Rezerva: pokud má voda výrazně >40 %
  const water = elementStats.value.rows.find(r => r.k==='Water')
  if (water?.p >= 40) out.push({ title: 'Vysoká citlivost', note: 'Silný prvek Vody (≥ 40 %)' })

  return out
})

const top5 = computed(() => highlights.value.slice(0,5))
</script>
