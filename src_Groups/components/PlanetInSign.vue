<template>
  <UniversalAstroSelector
    title="PLANETS IN SIGN"
    :primary-options="PLANETS"
    :secondary-options="SIGNS"
    key-suffix="in"
    :data-object="planetInSign"
    @add-interpretation="onAdd"
  />
</template>

<script setup>
import UniversalAstroSelector from "./UniversalAstroSelector.vue";
import { planetInSign } from "./../planetinsign.ts";

const PLANETS = [
  "sun",
  "moon",
  "mercury",
  "mars",
  "venus",
  "jupiter",
  "saturn",
  "uranus",
  "neptune",
  "pluto",
];
const SIGNS = [
  "aries",
  "taurus",
  "gemini",
  "cancer",
  "leo",
  "virgo",
  "libra",
  "scorpio",
  "sagittarius",
  "capricorn",
  "aquarius",
  "pisces",
];

function onAdd(data) {
  console.log("Added interpretation:", data);
}
</script>

<!-- <template>
  <div
    class="p-4 flex flex-col gap-4 border-2 border-gray-400 rounded-xl text-slate-600 max-w-md mx-auto"
  >
    <h3 class="text-center font-semibold text-lg">PLANETS IN SIGN</h3>

    <div class="flex flex-col gap-3">
      <select
        v-model="actualPlanet"
        class="w-full border border-gray-300 rounded-md p-3 text-slate-700 hover:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-400"
      >
        <option v-for="planet in PLANETS" :key="planet" :value="planet">
          {{ planet }}
        </option>
      </select>

      <select
        v-model="actualSign"
        class="w-full border border-gray-300 rounded-md p-3 text-slate-700 hover:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-400"
      >
        <option v-for="sign in SIGNS" :key="sign" :value="sign">
          {{ sign }}
        </option>
      </select>

      <button
        :disabled="isAdding"
        @click="addText"
        class="w-full px-6 py-3 bg-blue-600 text-white rounded-md font-semibold transition-colors duration-200 disabled:bg-blue-300 disabled:cursor-not-allowed hover:bg-blue-700 disabled:hover:bg-blue-300"
      >
        {{ isAdding ? "Adding..." : "ADD" }}
      </button>

      <transition name="fade">
        <p
          v-if="showFeedback"
          class="text-center text-green-600 font-semibold mt-2 select-none"
        >
          Přidáno!
        </p>
      </transition>
    </div>

    <textarea
      rows="20"
      v-model="planetInSign[`${actualPlanet}in${actualSign}`]"
      class="w-full border border-gray-300 rounded-xl p-3 bg-amber-50 text-slate-700 resize-y focus:outline-none focus:ring-2 focus:ring-yellow-400"
      placeholder="Zde se zobrazí nebo zadá text..."
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { planetInSign } from "./../planetinsign.ts";

const emit = defineEmits(["add-interpretation"]);

const PLANETS = [
  "sun",
  "moon",
  "mercury",
  "mars",
  "venus",
  "jupiter",
  "saturn",
  "uranus",
  "neptune",
  "pluto",
];
const SIGNS = [
  "aries",
  "taurus",
  "gemini",
  "cancer",
  "leo",
  "virgo",
  "libra",
  "scorpio",
  "sagittarius",
  "capricorn",
  "aquarius",
  "pisces",
];

const actualPlanet = ref(PLANETS[0]);
const actualSign = ref(SIGNS[0]);

const isAdding = ref(false);
const showFeedback = ref(false);

function addText() {
  if (isAdding.value) return;

  isAdding.value = true;
  showFeedback.value = true;

  const key = `${actualPlanet.value}in${actualSign.value}`;
  const text = planetInSign[key] || "";
  emit("add-interpretation", { key, text });

  setTimeout(() => {
    isAdding.value = false;
  }, 1000);

  setTimeout(() => {
    showFeedback.value = false;
  }, 1500);
}
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style> -->
