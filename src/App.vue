<template>
  <div class="min-h-screen bg-slate-50 text-slate-800">
    <!-- Top bar: Active user -->
    <header class="sticky top-0 z-30 backdrop-blur bg-white/70 border-b border-slate-200">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 py-3 flex items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="h-8 w-8 rounded-xl bg-slate-900 text-white grid place-items-center text-xs font-semibold shadow">
            AF
          </div>
          <h1 class="text-sm sm:text-base font-semibold tracking-tight">Astro-finder</h1>
        </div>
        <div class="flex items-center gap-2">
          <label for="activeUser" class="text-xs sm:text-sm text-slate-500">Active User</label>
          <select
            id="activeUser"
            v-model="activeUserId"
            class="text-sm border rounded-lg px-2 py-1"
          >
            <option disabled value="">— select —</option>
            <option v-for="u in users" :key="u.id" :value="u.id">{{ u.name }}</option>
          </select>
        </div>
      </div>

      <!-- Navigation -->
      <nav class="border-t border-slate-200 bg-white/70 backdrop-blur sticky top-[48px] z-20">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 py-3 space-y-3">
          <!-- Section 1: Controls (collapsible) -->
          <div class="rounded-2xl border border-slate-200 bg-gradient-to-br from-slate-100 to-slate-50">
            <div class="flex items-center justify-between px-3 py-2">
              <div class="text-[11px] uppercase tracking-wide font-semibold text-slate-600">Controls</div>
              <button
                class="inline-flex items-center gap-1 text-slate-600 hover:text-slate-800 text-xs px-2 py-1 rounded-lg hover:bg-white"
                @click="showControls = !showControls"
                :aria-expanded="showControls.toString()"
              >
                <span>{{ showControls ? 'Hide' : 'Show' }}</span>
                <svg :class="['h-4 w-4 transition-transform', showControls ? 'rotate-180' : '']" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                  <path fill-rule="evenodd" d="M5.23 7.21a.75.75 0 011.06.02L10 10.94l3.71-3.71a.75.75 0 111.06 1.06l-4.24 4.24a.75.75 0 01-1.06 0L5.21 8.29a.75.75 0 01.02-1.08z" clip-rule="evenodd" />
                </svg>
              </button>
            </div>
            <transition name="fade-slide">
              <div v-show="showControls" class="px-3 pb-3">
                <div class="flex flex-wrap gap-2">
                  <button
                    v-for="tab in firstRowTabs"
                    :key="tab.name"
                    @click="activeTab = tab.name"
                    :class="tabButtonClass(tab.name, 'emerald')"
                  >
                    {{ tab.label }}
                  </button>
                </div>
              </div>
            </transition>
          </div>

          <!-- Section 2: Interpretace (collapsible) -->
          <div class="rounded-2xl border border-indigo-100 bg-gradient-to-br from-indigo-50 to-white">
            <div class="flex items-center justify-between px-3 py-2">
              <div class="text-[11px] uppercase tracking-wide font-semibold text-indigo-600">Interpretace</div>
              <button
                class="inline-flex items-center gap-1 text-indigo-600 hover:text-indigo-800 text-xs px-2 py-1 rounded-lg hover:bg-white"
                @click="showInterpretace = !showInterpretace"
                :aria-expanded="showInterpretace.toString()"
              >
                <span>{{ showInterpretace ? 'Hide' : 'Show' }}</span>
                <svg :class="['h-4 w-4 transition-transform', showInterpretace ? 'rotate-180' : '']" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                  <path fill-rule="evenodd" d="M5.23 7.21a.75.75 0 011.06.02L10 10.94l3.71-3.71a.75.75 0 111.06 1.06l-4.24 4.24a.75.75 0 01-1.06 0L5.21 8.29a.75.75 0 01.02-1.08z" clip-rule="evenodd" />
                </svg>
              </button>
            </div>
            <transition name="fade-slide">
              <div v-show="showInterpretace" class="px-3 pb-3">
                <div class="flex flex-wrap gap-2">
                  <button
                    v-for="tab in secondRowTabs"
                    :key="tab.name"
                    @click="activeTab = tab.name"
                    :class="tabButtonClass(tab.name, 'indigo')"
                  >
                    {{ tab.label }}
                  </button>
                </div>
              </div>
            </transition>
          </div>
        </div>
      </nav>
    </header>

    <!-- Main content -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 py-4">
      <div class="grid grid-cols-1 gap-4">
        <!-- Interpretace components -->
        <div class="rounded-2xl border border-indigo-100 bg-white shadow-sm p-4 sm:p-6">
          <planet-in-sign
            v-if="activeTab === 'planet'"
            :users="users"
            :activeUserId="activeUserId"
            @add-interpretation="addInterpretation"
          />
          <aspects
            v-else-if="activeTab === 'aspects'"
            :users="users"
            :activeUserId="activeUserId"
            @add-interpretation="addInterpretation"
          />
          <house-in-sign
            v-else-if="activeTab === 'sun'"
            :users="users"
            :activeUserId="activeUserId"
            @add-interpretation="addInterpretation"
          />
          <planet-in-house
            v-else-if="activeTab === 'phouses'"
            :users="users"
            :activeUserId="activeUserId"
            @add-interpretation="addInterpretation"
          />
          <synastry
            v-else-if="activeTab === 'synastry'"
          />
          <analyze
            v-else-if="activeTab === 'analyze'"
            :users="users"
            :activeUserId="activeUserId"
          />
          <Analyze2
            v-if="activeTab === 'analyze2'"
            :users="users"
            :active-user-id="activeUserId"
          />

          <!-- Users -->
          <div v-else-if="activeTab === 'users'">
            <Users
              :users="users"
              :groups="groups"
              @update:users="users = $event"
              @update:groups="groups = $event"
              @set-active-user="activeUserId = $event"
            />
          </div>

          <!-- Groups -->
          <div v-else-if="activeTab === 'groups'">
            <Groups
              :groups="groups"
              :users="users"
              @update:groups="groups = $event"
              @update:users="users = $event"
            />
          </div>

          <!-- Add Data -->
          <add-data
            v-else-if="activeTab === 'adddata'"
            :users="users"
            :active-user-id="activeUserId"
            @add-interpretation="addInterpretation"
          />

          <!-- Import -->
          <!-- Import -->
          <div v-else-if="activeTab === 'import'">
            <ImportData
              :users="users"
              :groups="groups"
              @replace-users="replaceUsers"
              @replace-groups="replaceGroups"
            />
          </div>


          <!-- Export -->
          <div v-else-if="activeTab === 'export'">
            <ExportData :users="users" :groups="groups" />
          </div>

          <!-- Result -->
          <result-tab
            v-else-if="activeTab === 'result'"
            :users="users"
            :groups="groups"
            :activeUserId="activeUserId"
            @remove-item="removeItem"
            @clear-all="clearAll"
          />
        </div>
      </div> <!-- ⬅️ doplněné uzavření grid wrapperu -->
    </main>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import PlanetInSign from './components/PlanetInSign.vue'
import PlanetInHouse from './components/PlanetInHouse.vue'
import Aspects from './components/Aspects.vue'
import HouseInSign from './components/HouseInSign.vue'
import Synastry from './components/Synastry.vue'
import Analyze from './components/Analyze.vue'
import Groups from './components/Groups.vue'
import Users from './components/Users.vue'
import ResultTab from './components/Result.vue'
import AddData from '@/components/AddData.vue'
import Analyze2 from './components/Analyze2.vue'
import ImportData from './components/ImportData.vue'
import ExportData from './components/ExportData.vue'

const firstRowTabs = [
  { name: 'groups', label: 'Groups' },
  { name: 'users', label: 'Users' },
  { name: 'adddata', label: 'Add Data' },
  { name: 'result', label: 'Result' },
  { name: 'analyze', label: 'Analyze' },
  { name: 'analyze2', label: 'Analyze2' },
  { name: 'import', label: 'Import' },   // ⬅️ nové
  { name: 'export', label: 'Export' },   // ⬅️ nové
]
const secondRowTabs = [
  { name: 'planet', label: 'Planety Zn.' },
  { name: 'aspects', label: 'Aspekty' },
  { name: 'sun', label: 'Domy' },
  { name: 'phouses', label: 'Planety Dom.' },
  { name: 'synastry', label: 'Synastry' },
]

const activeTab = ref('planet')
const activeUserId = ref('')

// collapsible state
const showControls = ref(true)
const showInterpretace = ref(true)

// LocalStorage: USERS
const users = ref(JSON.parse(localStorage.getItem('users') || '[]'))
if (users.value.length === 0) {
  users.value = [
    { id: 'u1', name: 'User 1', results: [] },
  ]
}
watch(users, (val) => localStorage.setItem('users', JSON.stringify(val)), { deep: true })

// LocalStorage: GROUPS
const groups = ref(JSON.parse(localStorage.getItem('groups') || '[]'))
if (groups.value.length === 0) {
  groups.value = [
    { id: 'group_1', name: 'Family', users: [] },
    { id: 'group_2', name: 'Friends', users: [] },
    { id: 'group_3', name: 'Job', users: [] },
    { id: 'group_4', name: 'Clients', users: [] },
    { id: 'group_5', name: 'Celebrites', users: [] },
    { id: 'group_6', name: 'Group1', users: [] },
    { id: 'group_7', name: 'Group2', users: [] },
  ]
}
watch(groups, (val) => localStorage.setItem('groups', JSON.stringify(val)), { deep: true })

function addInterpretation(item) {
  const user = users.value.find((u) => u.id === activeUserId.value)
  if (!user) return
  if (!user.results) user.results = []
  user.results.unshift({
    title: item?.title || '—',
    text: item?.text || '',
    type: item?.type || 'Unknown',
    ts: Date.now(),
  })
}

function removeItem(index) {
  const user = users.value.find((u) => u.id === activeUserId.value)
  if (user && user.results) user.results.splice(index, 1)
}

function clearAll() {
  const user = users.value.find((u) => u.id === activeUserId.value)
  if (user) user.results = []
}

/* handler pro import: nahradí users a udrží aktivního uživatele, padá-li mimo sadu, vybere prvního */
function replaceUsers(newUsers) {
  users.value = Array.isArray(newUsers) ? newUsers : []
  if (!users.value.find(u => u.id === activeUserId.value)) {
    activeUserId.value = users.value[0]?.id || ''
  }
}

function tabButtonClass(name, tone = 'emerald') {
  const active = activeTab.value === name
  const base = 'px-3 sm:px-4 py-1.5 rounded-xl text-sm font-medium border transition-colors'
  const passiveTone = tone === 'indigo'
    ? 'border-slate-200 bg-white hover:bg-slate-50 text-slate-600'
    : 'border-slate-200 bg-white hover:bg-slate-50 text-slate-700'
  const activeTone = tone === 'indigo'
    ? 'border-indigo-300 bg-indigo-50 text-indigo-700'
    : 'border-emerald-300 bg-emerald-50 text-emerald-700'
  return [base, active ? activeTone : passiveTone]
}
</script>

<style scoped>
.fade-slide-enter-active, .fade-slide-leave-active {
  transition: all .18s ease;
}
.fade-slide-enter-from, .fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}
</style>
