<template>
  <div class="p-3">
    <h2 class="text-base font-semibold mb-2">Analyze 2</h2>

    <div v-if="!users.length" class="text-sm text-gray-500">Žádní uživatelé k analýze.</div>
    <div v-else-if="!activeUser" class="text-sm text-gray-500">Žádný aktivní uživatel.</div>
    <div v-else-if="!(activeUser.results && activeUser.results.length)" class="text-sm text-gray-500">
      Aktivní uživatel zatím nemá výsledky.
    </div>

    <div v-else class="space-y-3">
      <!-- Top 5 výroků -->
      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Top 5 výroků o profilu</div>
        <ProfileHighlights :items="activeUser.results" />
      </div>

      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Důstojnosti planet</div>
        <Dignities :items="activeUser.results" />
      </div>

      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Stellia</div>
        <Stelliums :items="activeUser.results" />
      </div>

      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Vládci znamení</div>
        <Rulers :items="activeUser.results" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import ProfileHighlights from './ProfileHighlights.vue'
import Dignities from './Dignities.vue'
import Stelliums from './Stelliums.vue'
import Rulers from './Rulers.vue'

const props = defineProps({
  users: { type: Array, default: () => [] },
  activeUserId: { type: [String, Number], default: '' }
})

const users = computed(() => props.users || [])
const activeUser = computed(() => users.value.find(u => u.id === props.activeUserId))
</script>
