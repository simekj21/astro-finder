<template>
  <div>
    <div v-if="!users.length" class="text-sm text-gray-500">Žádní uživatelé k exportu.</div>

    <div v-else class="space-y-3">
      <div class="flex items-center gap-3">
        <label class="inline-flex items-center gap-2 text-sm">
          <input type="checkbox" v-model="selectAll" />
          <span>Vybrat vše</span>
        </label>

        <button
          class="px-3 py-1.5 rounded-md border text-sm transition-colors bg-indigo-600 text-white border-indigo-600 shadow-sm disabled:opacity-50"
          :disabled="selectedIds.size === 0"
          @click="downloadJson"
        >
          Stáhnout JSON
        </button>
      </div>

      <div class="rounded border bg-white">
        <div v-for="(group, gi) in groupsView" :key="gi" class="border-b last:border-b-0">
          <div class="px-3 py-2 text-sm font-medium bg-slate-50">
            {{ group.name || 'Bez skupiny' }} <span class="text-xs text-slate-500">({{ group.users.length }})</span>
          </div>
          <div class="divide-y">
            <label
              v-for="u in group.users"
              :key="u.id || u.name"
              class="flex items-center gap-3 px-3 py-2 text-sm"
            >
              <input type="checkbox" :value="u.id || u.name" v-model="selectedRaw" />
              <span class="font-medium">{{ u.name || u.id }}</span>
              <span class="text-xs text-slate-500 ml-auto">{{ resolveGroupName(u) || '—' }}</span>
            </label>
          </div>
        </div>
      </div>

      <div class="text-xs text-gray-500">
        Export obsahuje: <code>users</code> (vybraní uživatelé – včetně jejich skupin) a <code>groups</code> (unikátní přehled skupin nalezených u vybraných uživatelů).
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue'

const props = defineProps({
  users: { type: Array, default: () => [] },
  groups: { type: Array, default: () => [] }
})

/* ---- pomocné dohledání názvu skupiny ---- */
function resolveGroupName(u) {
  // 1) přímo na uživateli
  if (u?.groupName) return u.groupName
  if (u?.group) return u.group

  // 2) podle groupId → groups[].name
  if (u?.groupId) {
    const g = props.groups.find(g => g.id === u.groupId)
    if (g?.name) return g.name
  }

  // 3) podle členství v groups[*].users (podporujeme různé tvary)
  for (const g of props.groups || []) {
    const gu = Array.isArray(g.users) ? g.users : []
    const found = gu.find(item => {
      if (typeof item === 'string') return item === u.id || item === u.name
      if (item && typeof item === 'object') return item.id ? item.id === u.id : false
      return false
    })
    if (found) return g.name || g.id || ''
  }

  return ''
}

/* ---- výběr ---- */
const selectedIds = ref(new Set())
const selectedRaw = computed({
  get() { return Array.from(selectedIds.value) },
  set(v) { selectedIds.value = new Set(v) }
})
const selectAll = computed({
  get() { return props.users.length > 0 && selectedIds.value.size === props.users.length },
  set(val) {
    if (val) selectedIds.value = new Set(props.users.map(u => u.id || u.name))
    else selectedIds.value.clear()
  }
})
watch(() => props.users, (nu) => {
  selectedIds.value = new Set((nu || []).map(u => u.id || u.name)) // default: vše
}, { immediate: true })

/* ---- seskupení dle skupiny (přes resolveGroupName) ---- */
const groupsView = computed(() => {
  const map = new Map()
  for (const u of (props.users || [])) {
    const gname = resolveGroupName(u) || ''
    if (!map.has(gname)) map.set(gname, [])
    map.get(gname).push(u)
  }
  return Array.from(map.entries()).map(([name, users]) => ({ name, users }))
})

/* ---- export JSON ---- */
function buildPayload() {
  const picked = (props.users || []).filter(u => selectedIds.value.has(u.id || u.name))

  // zajistíme, že každý exportovaný uživatel má groupName
  const usersOut = picked.map(u => {
    const gname = resolveGroupName(u)
    return gname && !u.groupName ? { ...u, groupName: gname } : { ...u }
  })

  // skupiny z vybraných uživatelů
  const groupMap = new Map()
  for (const u of usersOut) {
    const gname = resolveGroupName(u)
    if (gname) {
      // zkusíme dohledat i id skupiny
      const g = (props.groups || []).find(x => x.name === gname || x.id === u.groupId)
      const gid = g?.id || gname
      groupMap.set(gid, { id: gid, name: gname })
    }
  }

  return {
    version: '1',
    app: 'astro-finder',
    exportedAt: new Date().toISOString(),
    users: usersOut,
    groups: Array.from(groupMap.values())
  }
}

function downloadJson() {
  const payload = buildPayload()
  const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `astro-finder-export-${new Date().toISOString().slice(0,19).replace(/[:T]/g,'-')}.json`
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}
</script>
