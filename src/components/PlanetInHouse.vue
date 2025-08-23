<template>
  <UniversalAstroSelector
    title="PLANETS IN HOUSE"
    :primary-options="PLANETS"
    :secondary-options="HOUSES"
    key-suffix="in"
    :data-object="planetInHouse"
    @add-interpretation="onAdd"
  />
</template>

<script setup>
import UniversalAstroSelector from "./UniversalAstroSelector.vue";
import { planetInHouse } from "./../planetinhouse.ts";

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

function onAdd(data) {
  console.log("Added interpretation:", data);
}
</script>
