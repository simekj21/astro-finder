<template>
  <UniversalAstroSelector
    title="HOUSES IN SIGN"
    :primary-options="HOUSES"
    :secondary-options="SIGNS"
    key-suffix="in"
    :data-object="houseInSign"
    @add-interpretation="onAdd"
  />
</template>

<script setup>
import UniversalAstroSelector from "./UniversalAstroSelector.vue";
import { houseInSign } from "./../houseinsign.ts";

const HOUSES = [
  "house1",
  "house2",
  "house3",
  "house4",
  "house5",
  "house6",
  "house7",
  "house8",
  "house9",
  "house10",
  "house11",
  "house12",
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

<!-- <script setup>
import InterpretationSelector from "./InterpretationSelector.vue";
import { houseInSign } from "./../houseinsign.ts";

const HOUSES = [
  "house1",
  "house2",
  "house3",
  "house4",
  "house5",
  "house6",
  "house7",
  "house8",
  "house9",
  "house10",
  "house11",
  "house12",
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

function handleAdd(payload) {
  console.log("Added houseInSign:", payload);
}
</script>

<template>
  <InterpretationSelector
    title="HOUSES IN SIGN"
    :selections="[HOUSES, SIGNS]"
    :dataMap="houseInSign"
    :keyBuilder="(vals) => `${vals[0]}in${vals[1]}`"
    @add-interpretation="handleAdd"
  />
</template>

<!-- <template>
  <div class="p-2 flex flex-col gap-2 max-w-md mx-auto">
    <div class="p-2 border-gray-400 border-2 rounded-xl text-slate-600">
      <h3 class="text-center font-semibold mb-2">HOUSES IN SIGN</h3>

      <div class="flex flex-wrap items-center gap-3">
        <select
          v-model="actualHouse"
          class="flex-grow min-w-[120px] p-2 border border-gray-300 rounded-md"
        >
          <option v-for="house in HOUSES" :key="house">{{ house }}</option>
        </select>

        <span class="self-center text-lg font-medium select-none">in</span>

        <select
          v-model="actualSign"
          class="flex-grow min-w-[120px] p-2 border border-gray-300 rounded-md"
        >
          <option v-for="sign in SIGNS" :key="sign">{{ sign }}</option>
        </select>

        <button
          :disabled="isAdding"
          class="ml-auto bg-blue-600 disabled:bg-blue-300 text-white px-4 py-2 rounded-md transition-colors duration-300"
          @click="handleAddText"
        >
          {{ isAdding ? "Added!" : "ADD" }}
        </button>
      </div>
    </div>

    <textarea
      class="w-full border-gray-200 border-2 rounded-xl bg-amber-50 text-base p-3 text-slate-600 resize-none"
      rows="20"
      v-model="houseInSign[`${actualHouse}in${actualSign}`]"
      readonly
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { houseInSign } from "./../houseinsign.ts";

const emit = defineEmits(["add-interpretation"]);

const HOUSES = [
  "house1",
  "house2",
  "house3",
  "house4",
  "house5",
  "house6",
  "house7",
  "house8",
  "house9",
  "house10",
  "house11",
  "house12",
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

const actualHouse = ref(HOUSES[0]);
const actualSign = ref(SIGNS[0]);

const isAdding = ref(false);

function handleAddText() {
  if (isAdding.value) return;
  isAdding.value = true;

  const key = `${actualHouse.value}in${actualSign.value}`;
  const text = houseInSign[key] || "";

  emit("add-interpretation", { key, text });

  // feedback + zablokování tlačítka na 1.5 sekundy
  setTimeout(() => {
    isAdding.value = false;
  }, 1500);
}
</script> -->
-->
