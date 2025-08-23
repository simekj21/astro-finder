<template>
  <div class="space-y-3">
    <!-- nahrání souboru -->
    <div class="rounded border bg-white p-3">
      <div class="text-sm font-medium mb-2">Importovat JSON</div>
      <input type="file" accept="application/json" @change="onFile" />
      <div v-if="error" class="mt-2 text-sm text-rose-600">{{ error }}</div>
    </div>

    <!-- náhled uživatelů ze souboru -->
    <div v-if="stage !== 'idle'" class="rounded border bg-white p-3 space-y-3">
      <div class="flex items-center gap-3">
        <label class="inline-flex items-center gap-2 text-sm">
          <input type="checkbox" v-model="selectAll" />
          <span>Vybrat vše</span>
        </label>

        <button
          v-if="stage === 'select'"
          class="px-3 py-1.5 rounded-md border text-sm transition-colors bg-indigo-600 text-white border-indigo-600 shadow-sm disabled:opacity-50"
          :disabled="selectedIds.size === 0"
          @click="beginImport"
        >
          Importovat vybrané
        </button>
      </div>

      <div class="rounded border">
        <div v-for="(g, gi) in importedGroupsView" :key="gi" class="border-b last:border-b-0">
          <div class="px-3 py-2 text-sm font-medium bg-slate-50">
            {{ g.name || 'Bez skupiny' }} <span class="text-xs text-slate-500">({{ g.users.length }})</span>
          </div>
          <div class="divide-y">
            <label v-for="u in g.users" :key="u.__key" class="flex items-center gap-3 px-3 py-2 text-sm">
              <input type="checkbox" :value="u.__key" v-model="selectedRaw" />
              <span class="font-medium">{{ u.name || u.id }}</span>
              <span class="text-xs text-slate-500 ml-auto">{{ u.groupName || u.group || '—' }}</span>
            </label>
          </div>
        </div>
      </div>
    </div>

    <!-- řešení konfliktů -->
    <div v-if="stage === 'conflicts'" class="rounded border bg-white p-3 space-y-3">
      <div class="text-sm font-medium">Nalezeny konflikty — zvol, co udělat:</div>
      <div class="text-xs text-slate-600">Stejný uživatel (podle ID, nebo jména + skupiny) už v databázi existuje.</div>

      <div class="rounded border divide-y">
        <div v-for="c in conflicts" :key="c.key" class="p-3 text-sm">
          <div class="font-medium">
            {{ c.incoming.name || c.incoming.id }}
            <span class="text-xs text-slate-500">({{ c.incoming.groupName || c.incoming.group || '—' }})</span>
          </div>
          <div class="mt-1 flex items-center gap-4">
            <label class="inline-flex items-center gap-2">
              <input type="radio" :name="`conf-${c.key}`" value="keep" v-model="c.decision" />
              <span>Ponechat původní</span>
            </label>
            <label class="inline-flex items-center gap-2">
              <input type="radio" :name="`conf-${c.key}`" value="overwrite" v-model="c.decision" />
              <span>Přepsat novým</span>
            </label>
          </div>
        </div>
      </div>

      <div class="flex items-center gap-2">
        <button
          class="px-3 py-1.5 rounded-md border text-sm transition-colors bg-indigo-600 text-white border-indigo-600 shadow-sm"
          @click="applyImport"
        >
          Potvrdit import
        </button>
        <button class="px-3 py-1.5 rounded-md border text-sm bg-white border-slate-200" @click="cancelConflicts">
          Zpět k výběru
        </button>
      </div>
    </div>

    <!-- hotovo -->
    <div v-if="stage === 'done'" class="rounded border bg-white p-3">
      <div class="text-sm">Import dokončen.</div>
      <div class="text-xs text-slate-600">Můžeš přejít na jiné záložky.</div>
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, ref } from 'vue'

const emit = defineEmits(['replace-users','replace-groups'])

const props = defineProps({
  users: { type: Array, default: () => [] },   // existující v aplikaci
  groups: { type: Array, default: () => [] }   // existující skupiny
})

const error = ref('')
const stage = ref('idle') // 'idle' | 'select' | 'conflicts' | 'done'
const imported = ref({ users: [], groups: [] })
const selectedIds = ref(new Set())
const conflicts = reactive([]) // [{ key, incoming, existing, decision }]

/* ---------- helpers ---------- */
function slug(s) {
  return String(s || '').toLowerCase().trim()
    .replace(/\s+/g, '-')
    .replace(/[^a-z0-9\-]/g, '')
}

/* ---------- výběry ---------- */
const selectedRaw = computed({
  get() { return Array.from(selectedIds.value) },
  set(v) { selectedIds.value = new Set(v) }
})
const selectAll = computed({
  get() { return imported.value.users.length > 0 && selectedIds.value.size === imported.value.users.length },
  set(val) {
    if (val) selectedIds.value = new Set(imported.value.users.map(u => u.__key))
    else selectedIds.value.clear()
  }
})

/* ---------- načtení souboru ---------- */
function onFile(e) {
  error.value = ''
  const file = e.target.files?.[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = () => {
    try {
      const obj = JSON.parse(String(reader.result || '{}'))
      const users = Array.isArray(obj.users) ? obj.users : []
      const groups = Array.isArray(obj.groups) ? obj.groups : []
      for (const u of users) {
        u.__key = String(u.id || `${u.name}-${u.groupName || ''}` || Math.random())
      }
      imported.value = { users, groups }
      selectedIds.value = new Set(users.map(u => u.__key)) // default: vše
      stage.value = 'select'
    } catch (err) {
      console.error(err)
      error.value = 'Soubor není platný JSON export.'
    }
  }
  reader.onerror = () => { error.value = 'Soubor se nepodařilo načíst.' }
  reader.readAsText(file, 'utf-8')
}

/* ---------- náhled: seskupení ze souboru ---------- */
const importedGroupsView = computed(() => {
  const map = new Map()
  for (const u of imported.value.users) {
    const gname = u.groupName || u.group || ''
    if (!map.has(gname)) map.set(gname, [])
    map.get(gname).push(u)
  }
  return Array.from(map.entries()).map(([name, users]) => ({ name, users }))
})

/* ---------- konflikty a shody ---------- */
function beginImport() {
  conflicts.splice(0)
  const selected = new Map(imported.value.users.filter(u => selectedIds.value.has(u.__key)).map(u => [u.__key, u]))
  for (const u of selected.values()) {
    const ex = findExistingMatch(u)
    if (ex) {
      conflicts.push({
        key: u.__key,
        incoming: u,
        existing: ex,
        decision: 'keep' // default bezpečně
      })
    }
  }
  if (conflicts.length > 0) stage.value = 'conflicts'
  else applyImport()
}

function findExistingMatch(u) {
  if (u.id) {
    const byId = (props.users || []).find(x => x.id === u.id)
    if (byId) return byId
  }
  if (u.name) {
    const byNG = (props.users || []).find(x => (x.name === u.name) && ((x.groupName || x.group || '') === (u.groupName || u.group || '')))
    if (byNG) return byNG
  }
  return null
}

function ensureId(u) {
  if (u.id) return u.id
  try {
    const buf = new Uint8Array(8)
    window.crypto.getRandomValues(buf)
    const hex = Array.from(buf).map(b => b.toString(16).padStart(2,'0')).join('')
    u.id = `id-${hex}`
  } catch {
    u.id = `id-${Date.now()}-${Math.floor(Math.random()*1e6)}`
  }
  return u.id
}

/* ---------- sestavení výstupu: users + groups ---------- */
function buildGroupsFinal(finalUsers, existingGroups, importedGroups) {
  const gMap = new Map()

  // upsert pomocník
  const upsert = (g) => {
    if (!g) return
    const id = g.id || slug(g.name)
    const name = g.name || g.id || ''
    const prev = gMap.get(id) || { id, name, users: [] }
    // vždy udrž jméno (novější vyhraje, ale name !== '' preferujeme)
    prev.name = name || prev.name
    // sjednoť typ pole users
    prev.users = Array.isArray(prev.users) ? prev.users : []
    gMap.set(id, prev)
  }

  ;(existingGroups || []).forEach(upsert)
  ;(importedGroups || []).forEach(upsert)

  // skupiny z uživatelů (podle groupName)
  for (const u of finalUsers) {
    const gname = u.groupName || u.group || ''
    if (!gname) continue
    const id = u.groupId || (Array.from(gMap.values()).find(x => x.name === gname)?.id) || slug(gname)
    upsert({ id, name: gname })
  }

  // reset členství a naplnění podle finalUsers
  for (const g of gMap.values()) g.users = []
  for (const u of finalUsers) {
    const gname = u.groupName || u.group || ''
    if (!gname) continue
    const id = u.groupId || (Array.from(gMap.values()).find(x => x.name === gname)?.id) || slug(gname)
    if (gMap.has(id)) gMap.get(id).users.push(u.id)
  }

  return Array.from(gMap.values())
}

function normalizeUserGroup(u, groupsFinal) {
  const gname = u.groupName || u.group || ''
  if (!gname) return { ...u }
  const found = groupsFinal.find(g => g.name === gname) || groupsFinal.find(g => g.id === u.groupId)
  const groupId = found?.id || slug(gname)
  return { ...u, groupId, groupName: gname }
}

/* ---------- finální aplikace importu ---------- */
function applyImport() {
  const selected = imported.value.users.filter(u => selectedIds.value.has(u.__key))
  const existing = Array.isArray(props.users) ? JSON.parse(JSON.stringify(props.users)) : []

  const conflictMap = new Map(conflicts.map(c => [c.key, c.decision]))

  // sloučení uživatelů (bez skupin; ty dořešíme po vybudování groupsFinal)
  for (const u of selected) {
    const decision = conflictMap.get(u.__key)
    const match = findExistingMatch(u)
    if (match) {
      if (decision === 'overwrite') {
        if (match.id) {
          const idx = existing.findIndex(x => x.id === match.id)
          if (idx >= 0) existing[idx] = { ...u }
        } else {
          const key = `${match.name || ''}__${match.groupName || match.group || ''}`
          const idx = existing.findIndex(x => (`${x.name || ''}__${x.groupName || x.group || ''}`) === key)
          if (idx >= 0) existing[idx] = { ...u }
        }
      }
      continue
    }
    ensureId(u)
    existing.push({ ...u })
  }

  // postavené finální groups (merge existujících, importovaných a těch odvozených z user.groupName)
  const groupsFinal = buildGroupsFinal(existing, props.groups, imported.value.groups)

  // normalizace uživatelů (doplnění groupId + groupName) a re-idempotentní seřazení členství
  const usersFinal = existing.map(u => normalizeUserGroup(u, groupsFinal))

  // přepočítej membership (aby odpovídal usersFinal)
  const gMap = new Map(groupsFinal.map(g => [g.id, { ...g, users: [] }]))
  for (const u of usersFinal) {
    if (u.groupId && gMap.has(u.groupId)) gMap.get(u.groupId).users.push(u.id)
  }
  const groupsFinalWithMembers = Array.from(gMap.values())

  emit('replace-users', usersFinal)
  emit('replace-groups', groupsFinalWithMembers)
  stage.value = 'done'
}

function cancelConflicts() { stage.value = 'select' }
</script>
