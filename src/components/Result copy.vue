<template>
  <div class="space-y-2 relative">
    <h3 class="text-xl font-semibold mb-2">Výsledky</h3>
    <button
      @click="generateSummary"
      class="mb-4 px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600"
    >
      Generate Summary
    </button>
    <div
      v-for="(item, index) in results"
      :key="index"
      class="flex justify-between items-center border p-2 rounded"
    >
      <div>
        <strong>{{ item.key }}</strong>
        <p class="text-sm text-gray-700 whitespace-pre-wrap">
          {{ item.text }}
        </p>
      </div>
      <button
        @click="$emit('remove-item', index)"
        class="text-red-500 hover:underline"
      >
        Smazat
      </button>
    </div>
    <button
      v-if="results.length > 0"
      @click="$emit('clear-all')"
      class="text-red-600 font-bold hover:underline mt-2"
    >
      Vymazat vše
    </button>

    <!-- Overlay s generovaným souhrnem -->
    <transition name="fade">
      <div
        v-if="showSummaryOverlay"
        class="fixed inset-0 bg-black bg-opacity-70 flex justify-center items-center p-4 z-50"
      >
        <div
          class="bg-white max-w-3xl w-full max-h-[80vh] overflow-y-auto p-6 rounded-lg shadow-lg relative"
        >
          <button
            @click="closeSummary"
            class="absolute top-2 right-2 text-gray-700 hover:text-gray-900 font-bold text-xl"
          >
            &times;
          </button>
          <h2 class="text-2xl mb-4 font-semibold">Souhrn výsledků</h2>

          <!-- Tlačítko pro kopírování do clipboardu -->
          <button
            @click="copyToClipboard"
            class="mb-4 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            COPY ALL
          </button>

          <div class="space-y-6">
            <div v-if="summary.planet.length > 0">
              <h3 class="text-xl font-bold mb-2 border-b border-gray-300 pb-1">
                Planety
              </h3>
              <div
                v-for="(item, i) in summary.planet"
                :key="'planet-' + i"
                class="mb-3 p-3 border rounded bg-yellow-50"
              >
                <strong>{{ item.key }}</strong>
                <p class="whitespace-pre-wrap">{{ item.text }}</p>
              </div>
            </div>
            <div v-if="summary.aspects.length > 0">
              <h3 class="text-xl font-bold mb-2 border-b border-gray-300 pb-1">
                Aspekty
              </h3>
              <div
                v-for="(item, i) in summary.aspects"
                :key="'aspect-' + i"
                class="mb-3 p-3 border rounded bg-green-50"
              >
                <strong>{{ item.key }}</strong>
                <p class="whitespace-pre-wrap">{{ item.text }}</p>
              </div>
            </div>
            <div v-if="summary.houses.length > 0">
              <h3 class="text-xl font-bold mb-2 border-b border-gray-300 pb-1">
                Domy
              </h3>
              <div
                v-for="(item, i) in summary.houses"
                :key="'house-' + i"
                class="mb-3 p-3 border rounded bg-blue-50"
              >
                <strong>{{ item.key }}</strong>
                <p class="whitespace-pre-wrap">{{ item.text }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  results: {
    type: Array,
    required: true,
  },
});

const showSummaryOverlay = ref(false);

const summary = ref({
  planet: [],
  aspects: [],
  houses: [],
});

function generateSummary() {
  summary.value = { planet: [], aspects: [], houses: [] };
  for (const item of props.results) {
    const key = item.key.toLowerCase();
    if (key.includes("planet")) summary.value.planet.push(item);
    else if (key.includes("aspect")) summary.value.aspects.push(item);
    else if (key.includes("house")) summary.value.houses.push(item);
    else summary.value.aspects.push(item);
  }
  showSummaryOverlay.value = true;
}

function closeSummary() {
  showSummaryOverlay.value = false;
}

function copyToClipboard() {
  let textToCopy = "";

  if (summary.value.planet.length > 0) {
    textToCopy += "Planety:\n";
    summary.value.planet.forEach((item) => {
      textToCopy += `${item.key}: ${item.text}\n\n`;
    });
  }
  if (summary.value.aspects.length > 0) {
    textToCopy += "Aspekty:\n";
    summary.value.aspects.forEach((item) => {
      textToCopy += `${item.key}: ${item.text}\n\n`;
    });
  }
  if (summary.value.houses.length > 0) {
    textToCopy += "Domy:\n";
    summary.value.houses.forEach((item) => {
      textToCopy += `${item.key}: ${item.text}\n\n`;
    });
  }

  navigator.clipboard
    .writeText(textToCopy)
    .then(() => {
      alert("Souhrn zkopírován do schránky!");
    })
    .catch((err) => {
      alert("Nepodařilo se zkopírovat text: " + err);
    });
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
