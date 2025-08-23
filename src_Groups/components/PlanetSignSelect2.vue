<template>
  <div class="flex flex-col items-center space-y-8">
    <!-- Row of planet icons -->
    <div class="flex space-x-4">
      <button
        v-for="planet in planets"
        :key="planet.name"
        @click="selectPlanet(planet.name)"
        :class="['p-4 rounded-full', selectedPlanet === planet.name ? 'bg-blue-500' : 'bg-gray-200']"
      >
        <span v-html="planet.icon" class="w-8 h-8"></span>
      </button>
    </div>

    <!-- Row of zodiac sign icons -->
    <div class="flex space-x-4">
      <button
        v-for="sign in signs"
        :key="sign.name"
        @click="selectSign(sign.name)"
        :class="['p-4 rounded-full', selectedSign === sign.name ? 'bg-green-500' : 'bg-gray-200']"
      >
        <span v-html="sign.icon" class="w-8 h-8"></span>
      </button>
    </div>

    <!-- Display selected combination -->
    <div class="mt-4 text-lg font-bold">
      Selected combination: {{ selectedPlanet }} / {{ selectedSign }}
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
const planets = ref([
  { name: "Sun", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Moon", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><path d="M13.127 2.96A9.97 9.97 0 0 0 2 12a10 10 0 0 0 15.127 8.96A8.333 8.333 0 0 1 13.127 2.96z"/></svg>' },
  { name: "Mercury", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/><path d="M12 6v4H8v2h4v4h2v-4h4v-2h-4V6z"/></svg>' },
  { name: "Venus", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Mars", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Jupiter", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Saturn", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Uranus", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Neptune", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Pluto", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
]);

// Inline SVG for zodiac signs
const signs = ref([
  { name: "Aries", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Taurus", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Gemini", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Cancer", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Leo", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Virgo", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Libra", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Scorpio", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Sagittarius", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Capricorn", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Aquarius", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
  { name: "Pisces", icon: '<svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg>' },
]);

const selectedPlanet = ref(planets.value[0].name);
const selectedSign = ref(signs.value[0].name);

const selectPlanet = (planet) => {
  selectedPlanet.value = planet;
};

const selectSign = (sign) => {
  selectedSign.value = sign;
};
 
</script>
