<template>
  <div class="p-3">
    <h2 class="text-base font-semibold mb-2">Analyze</h2>
    <div v-if="!activeUser" class="text-sm text-gray-500">Žádný aktivní uživatel.</div>
    <div v-else class="space-y-3">
      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Živly (podle zastoupení)</div>
        <Elements :items="activeUser.results || []" />
      </div>

      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Dynamika znamení (Kardinální / Fixní / Pohyblivá)</div>
        <Dynamics :items="activeUser.results || []" />
      </div>

      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Personality</div>
        <Personality :items="activeUser.results || []" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import Elements from './Elements.vue'
import Dynamics from './Dynamics.vue'
import Personality from './Personality.vue'

const props = defineProps({
  users: { type: Array, required: true },
  activeUserId: { type: String, required: true }
})

const activeUser = computed(() => props.users.find(u => u.id === props.activeUserId))
</script>
