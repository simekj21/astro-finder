<script setup>
import { ref } from 'vue'

const planets = ref([
  { name: "Sun", icon: "☀️" },
  { name: "Moon", icon: "🌙" },
  { name: "Mercury", icon: "☿" },
  { name: "Venus", icon: "♀" },
  { name: "Mars", icon: "♂" },
  { name: "Jupiter", icon: "J" },
  { name: "Saturn", icon: "♄" },
  { name: "Uranus", icon: "♅" },
  { name: "Neptune", icon: "♆" },
  { name: "Pluto", icon: "♇" },
]);

// Data for zodiac signs with English names and icons
const signs = ref([
  { name: "Aries", icon: "♈" },
  { name: "Taurus", icon: "♉" },
  { name: "Gemini", icon: "♊" },
  { name: "Cancer", icon: "♋" },
  { name: "Leo", icon: "♌" },
  { name: "Virgo", icon: "♍" },
  { name: "Libra", icon: "♎" },
  { name: "Scorpio", icon: "♏" },
  { name: "Sagittarius", icon: "♐" },
  { name: "Capricorn", icon: "♑" },
  { name: "Aquarius", icon: "♒" },
  { name: "Pisces", icon: "♓" },
]);

// State for selected planet and sign
const selectedPlanet = ref(planets.value[0].name); // Default planet
const selectedSign = ref(signs.value[0].name); // Default sign

// Functions for selecting planet and sign
const selectPlanet = (planet) => {
  selectedPlanet.value = planet;
};

const selectSign = (sign) => {
  selectedSign.value = sign;
};

</script>

<template>
  <div class="flex flex-col items-center space-y-8">
    <!-- Row of planet icons -->
    <div class="flex space-x-4">
      <button
        v-for="planet in planets"
        :key="planet.name"
        @click="selectPlanet(planet.name)"
        :class="['p-4 rounded-full text-xl', selectedPlanet === planet.name ? 'bg-blue-500 text-white' : 'bg-gray-200']"
      >
        <span>{{ planet.icon }}</span>
      </button>
    </div>

    <!-- Row of zodiac sign icons -->
    <div class="flex space-x-4">
      <button
        v-for="sign in signs"
        :key="sign.name"
        @click="selectSign(sign.name)"
        :class="['p-4 rounded-full text-xl', selectedSign === sign.name ? 'bg-green-500 text-white' : 'bg-gray-200']"
      >
        <span>{{ sign.icon }}</span>
      </button>
    </div>

    <!-- Display selected combination -->
    <div class="mt-4 text-lg font-bold">
      Selected combination: {{ selectedPlanet }} / {{ selectedSign }}
    </div>
  </div>
</template>

<style scoped>
  button {
    width: 60px;
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
