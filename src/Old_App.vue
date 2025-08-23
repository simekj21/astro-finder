<template>
  <!-- <PlanetSignSelect /> -->

  <planet-in-sign />

  <div class="p-2 flex flex-col gap-2">
    <!-- <h1 class="text-xl text-slate-500">Icon Finder</h1> -->

    <!-- {{ actualPlanet }} in {{ actualSign }} -->

    <!-- {{ actualIndex }} -->

    <div
      class="p-1 border-gray-400 border-solid border-2 rounded-xl text-slate-600 w-96"
    >
      <h3>Planets in sign</h3>

      <select v-model="actualPlanet">
        <option v-for="planet in PLANETS" :key="planet">
          {{ planet }}
        </option>
      </select>
      in
      <select v-model="actualSign">
        <option v-for="sign in SIGNS" :key="sign">
          {{ sign }}
        </option>
      </select>
    </div>

    <!-- {{ planetInSign[0][`suninaries`][0] }} -->

    <textarea
      class="w-96 border-gray-200 border-solid border-2 rounded-xl bg-yellow-50 text-base p-2 text-slate-600"
      rows="30"
      v-model="planetInSign[`${actualPlanet}in${actualSign}`]"
    >
    </textarea>
    <div v-if="planetInSign.length > 1" class="flex flex-row gap-2">
      <button class="w-14 border-2 border-gray-400 rounded-lg" @click="goLeft">
        🡄
      </button>
      <button @click="goRight" class="w-14 border-2 border-gray-400 rounded-lg">
        🡆
      </button>
      <button @clist="test">test</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { planetInSign1 } from "./planetinsign1.ts";
import { planetInSign } from "./planetinsign.ts";
import { sunlunamix } from "./sunlunamix.ts";
import PlanetSignSelect from "./components/PlanetSignSelect.vue";
import PlanetSignSelect2 from "./components/PlanetSignSelect2.vue";

import PlanetInSign from "./components/PlanetInSign.vue";

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
  "libro",
  "scorpio",
  "sagittarius",
  "capricorn",
  "aquarius",
  "pisces",
];

const actualIndex = ref(0);
const actualPlanet = ref(PLANETS[0]);
const actualSign = ref(SIGNS[0]);
const numOfComments = ref(1);

// const planetInSign = ref([]);

// {{ planetInSign[`${actualPlanet}in${actualSign}`] }}
// {{ planetInSign[actualIndex][`${actualPlanet}in${actualSign}`] }}

const test = () => {
  console.log("test");
};

const goRight = () => {
  console.log("goRight");
  actualIndex.value++;

  if (actualIndex.value > planetInSign.value.length - 1) {
    actualIndex.value = 0;
  }

  // if (actualIndex.value < planetInSign.value.length - 1) {
  //   actualIndex.value += 1;
  // }
};

const goLeft = () => {
  console.log("goLeft");
  actualIndex.value--;

  if (actualIndex.value < 0) {
    actualIndex.value = planetInSign.value.length - 1;
  }

  // if (actualIndex.value > 0) {
  //   actualIndex.value -= 1;
  // }
};

onMounted(() => {
  console.log(sunlunamix[`suninaries`][`lunainaries`]);
  // console.log(sunlunamix.suninaries.lunainaries)

  console.log("sunlunamix", sunlunamix);
});

// onMounted(() => {
//   planetInSign.value = [];
//   planetInSign.value.push(planetInSign);
//   planetInSign.value.push(planetInSign);

//   numOfComments.value = planetInSign.value.length;

//   console.log(planetInSign.value[actualIndex][`suninaries`])
// })
</script>

<style scoped></style>
