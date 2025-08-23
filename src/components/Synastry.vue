<template>
  <UniversalAstroSelector
    title="SYNASTRY"
    :primary-options="SIGNS"
    :secondary-options="SIGNS"
    key-suffix="vs"
    :data-object="synastry"
    @add-interpretation="onAdd"
  />
</template>

<script setup>
import UniversalAstroSelector from "./UniversalAstroSelector.vue";
import { synastry } from "./../synastry.ts";

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
