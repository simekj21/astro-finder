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
  "luna",
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

