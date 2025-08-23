<script setup>
import InterpretationSelector from "./InterpretationSelector.vue";
import { aspects } from "./../aspects.ts";

const PLANETS = [
  "sun",
  "luna",
  "mercury",
  "venus",
  "mars",
  "jupiter",
  "saturn",
  "uranus",
  "neptune",
  "pluto",
];

const ASPECTS = ["0", "120", "180"];

function handleAdd(payload) {
  console.log("Added aspect:", payload);
}
</script>

<template>
  <InterpretationSelector
    title="PLANETARY ASPECTS"
    :selections="[PLANETS, ASPECTS, PLANETS]"
    :dataMap="aspects"
    :keyBuilder="(vals) => `${vals[0]}${vals[1]}${vals[2]}`"
    @add-interpretation="handleAdd"
  />
</template>

<!-- <template>
  <div class="p-2 flex flex-col gap-2">
    <div
      class="p-2 border-gray-400 border-solid border-2 rounded-xl text-slate-600 max-w-md"
    >
      <h3 class="text-center font-semibold mb-2">PLANETARY ASPECTS</h3>

      <div class="flex flex-row gap-2 items-center">
        <select
          v-model="planet1"
          class="flex-1 p-1 rounded border border-gray-300"
        >
          <option v-for="planet in PLANETS" :key="'p1-' + planet">
            {{ planet }}
          </option>
        </select>

        <select
          v-model="aspect"
          class="flex-1 p-1 rounded border border-gray-300"
        >
          <option v-for="a in ASPECTS" :key="'asp-' + a">
            {{ a }}
          </option>
        </select>

        <select
          v-model="planet2"
          class="flex-1 p-1 rounded border border-gray-300"
        >
          <option v-for="planet in PLANETS" :key="'p2-' + planet">
            {{ planet }}
          </option>
        </select>

        <button
          class="ml-auto px-3 py-1 bg-blue-500 text-white rounded-md disabled:opacity-50"
          @click="addText"
          :disabled="isAdding"
        >
          {{ isAdding ? "Adding..." : "ADD" }}
        </button>
      </div>
    </div>

    <textarea
      class="max-w-md border-gray-200 border-solid border-2 rounded-xl bg-amber-50 text-base p-2 text-slate-600 resize-none"
      rows="30"
      v-model="aspects[`${planet1}${aspect}${planet2}`]"
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { aspects } from "./../aspects.ts";

const emit = defineEmits(["add-interpretation"]);

const PLANETS = [
  "sun",
  "luna",
  "mercury",
  "venus",
  "mars",
  "jupiter",
  "saturn",
  "uranus",
  "neptune",
  "pluto",
];

const ASPECTS = ["0", "60", "90", "120", "180"]; // běžnější aspekty

const planet1 = ref(PLANETS[0]);
const planet2 = ref(PLANETS[1]);
const aspect = ref(ASPECTS[0]);

const isAdding = ref(false);

function addText() {
  if (isAdding.value) return;
  isAdding.value = true;

  const key = `${planet1.value}${aspect.value}${planet2.value}`;
  const text = aspects[key] || "";

  emit("add-interpretation", { key, text });

  setTimeout(() => {
    isAdding.value = false;
  }, 1000);
}
</script> -->
