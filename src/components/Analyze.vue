<template>
  <div class="p-3">
    <h2 class="text-base font-semibold mb-2">Analyze</h2>
    <div v-if="!activeUser" class="text-sm text-gray-500">Žádný aktivní uživatel.</div>

    <div v-else class="space-y-3">
      <!-- Živly -->
      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Živly (podle zastoupení)</div>
        <Elements :items="activeUser.results || []" />
      </div>

      <!-- Modalita (celkově + osobní planety) -->
      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Dynamika znamení</div>
        <Dynamics :items="activeUser.results || []" />
      </div>

      <!-- Polarita (Jin/Jang) -->
      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Polarita (Jin/Jang)</div>
        <Polarity :items="activeUser.results || []" />
      </div>

      <!-- Osobní / Sociální / Transpersonální -->
      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Osobní / Sociální / Transpersonální planety</div>
        <PlanetGroups :items="activeUser.results || []" />
      </div>

      <!-- Personality (I/E, Temperamenty, Enneagram) -->
      <div class="rounded border bg-white p-3">
        <div class="text-sm font-medium mb-2">Personality</div>
        <Personality
          :items="activeUser.results || []"
          :include-houses="true"
          :use-planet-weights="true"
          :dominance-boost-k="0.25"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import Elements from './Elements.vue'
import Dynamics from './Dynamics.vue'
import Polarity from './Polarity.vue'
import PlanetGroups from './PlanetGroups.vue'
import Personality from './Personality.vue'

const props = defineProps({
  users: { type: Array, required: true },
  activeUserId: { type: String, required: true }
})
const activeUser = computed(() => props.users.find(u => u.id === props.activeUserId))
</script>
