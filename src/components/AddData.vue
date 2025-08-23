<template>
  <div class="p-4 max-w-6xl mx-auto space-y-5">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold">Add Data</h2>
      <div class="text-sm text-gray-500">
        Vlož řádky a klikni na <span class="font-semibold">Parse</span>, pak <span class="font-semibold">Save to Results</span>.
      </div>
    </div>

    <textarea
      v-model="raw"
      class="w-full min-h-[260px] border rounded p-3 focus:outline-none focus:ring"
      placeholder="Např.:
Sun in Capricorn 8°23’, in 1st House
Moon in Virgo 19°27’, in 8th House
Sun Conjunction Venus (Orb: 6°06’, Applying)
Venus Square Uranus (Orb: 6°25’, Applying)"
    ></textarea>

    <div class="flex flex-wrap items-center gap-3">
      <button
        @click="handleParse"
        class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700 disabled:opacity-50"
        :disabled="!raw.trim() || !activeUser"
      >
        Parse
      </button>

      <button
        @click="handleSave"
        class="px-4 py-2 rounded bg-emerald-600 text-white hover:bg-emerald-700 disabled:opacity-50"
        :disabled="!activeUser || !preview.length"
        title="Uloží parsed výsledky do activeUser.results"
      >
        Save to Results
      </button>

      <button
        @click="clearText"
        class="px-4 py-2 rounded bg-gray-100 hover:bg-gray-200"
      >
        Clear
      </button>

      <span class="text-sm text-gray-500" v-if="activeUser">
        Ukládám do uživatele: <strong>{{ activeUser.name }}</strong>
      </span>
      <span class="text-sm text-red-600" v-else>
        Není vybraný Active User.
      </span>
    </div>

    <!-- zprávy -->
    <div v-if="messages.length" class="space-y-1">
      <div
        v-for="(m, i) in messages"
        :key="i"
        class="text-sm"
        :class="m.type === 'error' ? 'text-red-600' : 'text-green-700'"
      >
        {{ m.text }}
      </div>
    </div>

    <!-- PREVIEW výsledků (co se uloží) -->
    <div v-if="preview.length" class="mt-4">
      <h3 class="text-lg font-semibold mb-2">Preview výsledků</h3>
      <ul class="list-disc pl-5 space-y-1 text-sm">
        <li v-for="(r, i) in preview" :key="i">
          <strong>{{ r.type }}</strong> — {{ r.title }}
        </li>
      </ul>
    </div>

    <!-- Nalezené hodnoty: detailní panel s texty -->
    <div v-if="debug.length" class="mt-6">
      <h3 class="text-lg font-semibold mb-3">Nalezené hodnoty (po parsování každého řádku)</h3>

      <div class="space-y-4">
        <div
          v-for="(d, idx) in debug"
          :key="idx"
          class="border rounded p-3 bg-white"
        >
          <div class="text-sm text-gray-600 mb-2">
            <span class="font-semibold">Line:</span> {{ d.line }}
          </div>

          <template v-if="d.kind === 'pis'">
            <div class="grid md:grid-cols-3 gap-4">
              <!-- Planet in Sign -->
              <div class="border rounded p-2">
                <div class="font-semibold text-sm mb-1">Planet in Sign</div>
                <div class="text-xs text-gray-600">
                  <div><span class="font-semibold">Key:</span> <span class="font-mono">{{ d.k1 || '-' }}</span></div>
                  <div>
                    <span class="font-semibold">Found:</span>
                    <span :class="d.v1 ? 'text-emerald-700' : 'text-red-600'">{{ d.v1 ? '✔' : '✖' }}</span>
                  </div>
                  <div v-if="d.v1Text" class="mt-1 whitespace-pre-wrap">{{ d.v1Text }}</div>
                </div>
              </div>

              <!-- Planet in House -->
              <div class="border rounded p-2">
                <div class="font-semibold text-sm mb-1">Planet in House</div>
                <div class="text-xs text-gray-600">
                  <div><span class="font-semibold">Key:</span> <span class="font-mono">{{ d.k2 || '-' }}</span></div>
                  <div>
                    <span class="font-semibold">Found:</span>
                    <span :class="d.house ? (d.v2 ? 'text-emerald-700' : 'text-red-600') : ''">
                      {{ d.house ? (d.v2 ? '✔' : '✖') : '-' }}
                    </span>
                  </div>
                  <div v-if="d.v2Text" class="mt-1 whitespace-pre-wrap">{{ d.v2Text }}</div>
                </div>
              </div>

              <!-- House in Sign -->
              <div class="border rounded p-2">
                <div class="font-semibold text-sm mb-1">House in Sign</div>
                <div class="text-xs text-gray-600">
                  <div><span class="font-semibold">Key:</span> <span class="font-mono">{{ d.k3 || '-' }}</span></div>
                  <div>
                    <span class="font-semibold">Found:</span>
                    <span :class="d.house ? (d.v3 ? 'text-emerald-700' : 'text-red-600') : ''">
                      {{ d.house ? (d.v3 ? '✔' : '✖') : '-' }}
                    </span>
                  </div>
                  <div v-if="d.v3Text" class="mt-1 whitespace-pre-wrap">{{ d.v3Text }}</div>
                </div>
              </div>
            </div>

            <div class="text-xs text-gray-500 mt-2">
              <span class="font-semibold">Parsed:</span>
              planet={{ d.planetEn }}, sign={{ d.signEn }}, house={{ d.house ?? '-' }}
            </div>
          </template>

          <template v-else-if="d.kind === 'aspect'">
            <div class="grid md:grid-cols-3 gap-4">
              <!-- Aspect -->
              <div class="border rounded p-2 md:col-span-3">
                <div class="font-semibold text-sm mb-1">Aspect</div>
                <div class="text-xs text-gray-600">
                  <div class="mb-1">
                    <span class="font-semibold">Parsed:</span>
                    {{ d.p1En | cap }} {{ d.aspectName }} {{ d.p2En | cap }}
                    <span v-if="d.orb" class="ml-2"> (Orb: {{ d.orb }})</span>
                    <span v-if="d.motion" class="ml-1"> — {{ d.motion }}</span>
                  </div>
                  <div><span class="font-semibold">Degree map:</span> {{ d.deg }}</div>
                  <div><span class="font-semibold">Tried keys:</span> <span class="font-mono">{{ d.triedKeys.join(', ') }}</span></div>
                  <div class="mt-1"><span class="font-semibold">Found key:</span> <span class="font-mono">{{ d.key || '-' }}</span>
                    <span class="ml-2" :class="d.v ? 'text-emerald-700' : 'text-red-600'">{{ d.v ? '✔' : '✖' }}</span>
                  </div>
                  <div v-if="d.text" class="mt-2 whitespace-pre-wrap">{{ d.text }}</div>
                </div>
              </div>
            </div>
          </template>

          <template v-else>
            <div class="text-xs text-gray-500">Řádek rozpoznán, ale nezařazen ({{ d.reason || 'unknown' }})</div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from "vue"
import * as planetInSignMod from "@/planetinsign"
import * as planetInHouseMod from "@/planetinhouse"
import * as houseInSignMod from "@/houseinsign"
import * as aspectsMod from "@/aspects"

const planetInSignObj = (planetInSignMod.default && typeof planetInSignMod.default === "object") ? planetInSignMod.default : (planetInSignMod.planetInSign || planetInSignMod)
const planetInHouseObj = (planetInHouseMod.default && typeof planetInHouseMod.default === "object") ? planetInHouseMod.default : (planetInHouseMod.planetInHouse || planetInHouseMod)
const houseInSignObj = (houseInSignMod.default && typeof houseInSignMod.default === "object") ? houseInSignMod.default : (houseInSignMod.houseInSign || houseInSignMod)
const aspectsObj = (aspectsMod.default && typeof aspectsMod.default === "object") ? aspectsMod.default : (aspectsMod.aspects || aspectsMod)

const props = defineProps({ users: { type: Array, required: true }, activeUserId: { type: [String, Number], default: "" } })
const emit = defineEmits(["parsed", "saved"])

const raw = ref("")
const messages = ref([])
const preview = ref([])
const debug = ref([])
const activeUser = computed(() => props.users.find(u => u.id === props.activeUserId))

// ignorovaná slova (case-insensitive), odstraní i čárku/mezery
const IGNORE_WORDS_RE = /\b(?:Regrodable|Stationare|Stationary|Retrograde|Direct|Rx|R)\b[, ]*/gi

function normalizeTextKeepNewlines(s) {
  return s
    .replace(/\u00A0/g, " ")
    .replace(/[’]/g, "'")
    .replace(/[°]/g, "°")
    .replace(IGNORE_WORDS_RE, "")
    .replace(/\s*,\s*,\s*/g, ", ")
}

function normalizeLine(s) {
  return s
    .replace(/\u00A0/g, " ")
    .replace(/[’]/g, "'")
    .replace(/[°]/g, "°")
    .replace(IGNORE_WORDS_RE, "")
    .replace(/\s*,\s*,\s*/g, ", ")
    .replace(/\s+/g, " ")
    .trim()
}

const SIGN_MAP = { aries:"aries", taurus:"taurus", gemini:"gemini", cancer:"cancer", leo:"leo", virgo:"virgo", libra:"libra", scorpio:"scorpio", sagittarius:"sagittarius", capricorn:"capricorn", aquarius:"aquarius", pisces:"pisces" }
const PLANET_KEY_EN = { sun:"sun", moon:"luna", mercury:"mercury", venus:"venus", mars:"mars", jupiter:"jupiter", saturn:"saturn", uranus:"uranus", neptune:"neptune", pluto:"pluto", "north node":"northnode", lilith:"lilith", chiron:"chiron", fortune:"fortune", vertex:"vertex", asc:"asc", mc:"mc" }
const ASPECTS_PLANET_LABELS = { sun:["sun","slunce"], moon:["moon","luna"], mercury:["mercury","merkur"], venus:["venus","venuse","venus"], mars:["mars"], jupiter:["jupiter"], saturn:["saturn"], uranus:["uranus","uran"], neptune:["neptune","neptun"], pluto:["pluto"] }
const ASPECT_DEG = { conjunction:0, sextile:120, trine:120, square:180, opposition:180 }

function keyPlanetInSign(p, s){return `${p}in${s}`}
function keyPlanetInHouse(p, h){return `${p}inhouse${h}`}
function keyHouseInSign(h, s){return `house${h}in${s}`}

function findAspectKey(p1En, p2En, deg){
  const lefts = ASPECTS_PLANET_LABELS[p1En] || [p1En]
  const rights = ASPECTS_PLANET_LABELS[p2En] || [p2En]
  const tried=[]
  for(const L of lefts){
    for(const R of rights){
      const k1=`${L}${deg}${R}`.toLowerCase()
      tried.push(k1)
      if(k1 in aspectsObj) return {key:k1,tried}
      const k2=`${R}${deg}${L}`.toLowerCase()
      tried.push(k2)
      if(k2 in aspectsObj) return {key:k2,tried}
    }
  }
  return {key:null,tried}
}

const LINE_RE = /^\s*(?<planet>[A-Za-z ]+?)\s+in\s+(?<sign>[A-Za-z]+)(?:\s+[0-9]{1,2}°[0-9]{1,2}['’]?,?)?(?:\s*,\s*in\s+(?<house>\d{1,2})(?:st|nd|rd|th)\s+House)?\s*$/i
const SPECIAL_RE = /^\s*(ASC|MC)\s+in\s+[A-Za-z]+\s+[0-9]{1,2}°[0-9]{1,2}['’]?\s*$/i
const ASPECT_LINE_RE = /^\s*(?<p1>[A-Za-z]+)\s+(?<aspect>Conjunction|Sextile|Trine|Square|Opposition)\s+(?<p2>[A-Za-z]+)\s*\((?:\s*Orb:\s*(?<orb>[0-9]{1,2}°[0-9]{1,2})['’]?\s*,\s*)?(?<motion>Applying|Separating)?\)\s*$/i

function parseAll(text){
  const safe = normalizeTextKeepNewlines(text)
  const lines = safe.split(/\r?\n/)
  const items = [], errors = [], dbg = []
  for(let rawLine of lines){
    const line = normalizeLine(rawLine)
    if(!line) continue
    const a=line.match(ASPECT_LINE_RE)
    if(a){
      const p1En=a.groups.p1.toLowerCase(), p2En=a.groups.p2.toLowerCase(), aspectName=a.groups.aspect.toLowerCase()
      const orb=a.groups.orb||null, motion=a.groups.motion||null
      const deg=ASPECT_DEG[aspectName]
      if(deg==null){errors.push(`Neznámý aspekt: "${aspectName}" v řádku "${line}"`);dbg.push({kind:"aspect",line,reason:"unknown-aspect",p1En,p2En,aspectName});continue}
      const {key,tried}=findAspectKey(p1En,p2En,deg)
      const text=key?aspectsObj[key]:null
      dbg.push({kind:"aspect",line,p1En,p2En,aspectName:capitalize(aspectName),deg,triedKeys:tried,key,v:!!text,text,orb,motion})
      if(text){items.push({type:"Aspects",title:`${capitalize(p1En)} ${capitalize(aspectName)} ${capitalize(p2En)}${orb||motion?` (${[orb?`Orb: ${orb}`:'',motion].filter(Boolean).join(', ')})`:''}`,text})} else {errors.push(`Nenašel jsem aspekt klíčem: ${tried.join(", ")}`)}
      continue
    }
    if(SPECIAL_RE.test(line)){dbg.push({kind:"skip",line,reason:"special (ASC/MC), skipped"});continue}
    const m=line.match(LINE_RE)
    if(!m){errors.push(`Nepodařilo se rozpoznat řádek: "${line}"`);dbg.push({kind:"unknown",line,reason:"regex-no-match"});continue}
    let planet=(m.groups.planet||"").trim().toLowerCase(), sign=(m.groups.sign||"").trim().toLowerCase(), house=m.groups.house?parseInt(m.groups.house,10):null
    planet=planet.replace(/\s+/g," ")
    if(!(planet in PLANET_KEY_EN)){
      if(["north node","lilith","chiron","fortune","vertex"].includes(planet)){dbg.push({kind:"skip",line,planet,reason:"point-skipped"});continue}
      planet=planet.split(" ")[0]
    }
    const planetEn=PLANET_KEY_EN[planet]||planet
    const signEn=SIGN_MAP[sign]
    if(!signEn){errors.push(`Neznámé znamení: "${sign}" v řádku "${line}"`);dbg.push({kind:"pis",line,planetEn,sign,reason:"unknown-sign"});continue}
    const k1=keyPlanetInSign(planetEn,signEn), v1=planetInSignObj[k1]
    let k2=null,v2=null,k3=null,v3=null
    if(house&&house>=1&&house<=12){k2=keyPlanetInHouse(planetEn,house);v2=planetInHouseObj[k2];k3=keyHouseInSign(house,signEn);v3=houseInSignObj[k3]}
    dbg.push({kind:"pis",line,planetEn,signEn,house,k1,v1:!!v1,v1Text:v1||"",k2,v2:!!v2,v2Text:v2||"",k3,v3:!!v3,v3Text:v3||""})
    if(v1){items.push({type:"PlanetInSign",title:`${cap(planetEn)} in ${cap(signEn)}`,text:v1})} else {errors.push(`Nenašel jsem klíč v planetinsign: "${k1}"`)}
    if(house){
      if(v2){items.push({type:"PlanetInHouse",title:`${cap(planetEn)} in ${house}. House`,text:v2})} else {errors.push(`Nenašel jsem klíč v planetinhouse: "${k2}"`)}
      if(v3){items.push({type:"HouseInSign",title:`${house}. House in ${cap(signEn)}`,text:v3})} else {errors.push(`Nenašel jsem klíč v houseinsign: "${k3}"`)}
    }
  }
  return {items,errors,debug:dbg}
}

function cap(s){return s?s.charAt(0).toUpperCase()+s.slice(1):s}
function capitalize(s){return s?s.charAt(0).toUpperCase()+s.slice(1):s}
function clearText(){raw.value="";preview.value=[];messages.value=[];debug.value=[]}
function handleParse(){messages.value=[];preview.value=[];debug.value=[];if(!activeUser.value){messages.value.push({type:"error",text:"Vyber prosím Active User."});return}const {items,errors,debug:dbg}=parseAll(raw.value);preview.value=items;debug.value=dbg;emit("parsed",{items,errors,debug:dbg,activeUserId:props.activeUserId});if(errors.length)errors.forEach(e=>messages.value.push({type:"error",text:e}));messages.value.push({type:"ok",text:`Parsed položek: ${items.length}`})}
function handleSave(){if(!activeUser.value||!preview.value.length)return;const now=new Date().toISOString();if(!Array.isArray(activeUser.value.results))activeUser.value.results=[];const toSave=preview.value.map(it=>({...it,id:`${now}-${Math.random().toString(36).slice(2,8)}`,timestamp:now}));activeUser.value.results.push(...toSave);emit("saved",{items:toSave,activeUserId:props.activeUserId});messages.value.push({type:"ok",text:`Uloženo ${toSave.length} položek do výsledků uživatele ${activeUser.value.name}.`})}
</script>

<script>
export default { filters: { cap(v){return v?v.charAt(0).toUpperCase()+v.slice(1):v} } }
</script>