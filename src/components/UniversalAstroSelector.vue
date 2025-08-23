<template>
  <div
    class="p-4 flex flex-col gap-4 border-2 border-gray-400 rounded-xl text-slate-600 max-w-md mx-auto"
  >
    <h3 class="text-center font-semibold text-lg">{{ title }}</h3>

    <div class="flex flex-col gap-3">
      <!-- První výběrové pole -->
      <select
        v-model="selectedPrimary"
        class="w-full border border-gray-300 rounded-md p-3 text-slate-700 hover:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-400"
      >
        <option v-for="option in primaryOptions" :key="option" :value="option">
          {{ option }}
        </option>
      </select>

      <!-- Druhé výběrové pole -->
      <select
        v-model="selectedSecondary"
        class="w-full border border-gray-300 rounded-md p-3 text-slate-700 hover:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-400"
      >
        <option
          v-for="option in secondaryOptions"
          :key="option"
          :value="option"
        >
          {{ option }}
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
      v-model="dataObject[key]"
      class="w-full border border-gray-300 rounded-xl p-3 bg-amber-50 text-slate-700 resize-y focus:outline-none focus:ring-2 focus:ring-yellow-400"
      placeholder="Zde se zobrazí nebo zadá text..."
    ></textarea>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const props = defineProps({
  title: { type: String, default: "Astro Interpretation" },
  primaryOptions: { type: Array, required: true }, // např. planets
  secondaryOptions: { type: Array, required: true }, // např. signs nebo houses
  keySuffix: { type: String, default: "" }, // např. 'in'
  dataObject: { type: Object, required: true }, // např. planetInSign, planetInHouse
});

const emit = defineEmits(["add-interpretation"]);

const selectedPrimary = ref(props.primaryOptions[0]);
const selectedSecondary = ref(props.secondaryOptions[0]);
const isAdding = ref(false);
const showFeedback = ref(false);

const key = computed(
  () => `${selectedPrimary.value}${props.keySuffix}${selectedSecondary.value}`
);

function addText() {
  if (isAdding.value) return;

  isAdding.value = true;
  showFeedback.value = true;

  const text = props.dataObject[key.value] || "";
  emit("add-interpretation", { key: key.value, text });

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
</style>
