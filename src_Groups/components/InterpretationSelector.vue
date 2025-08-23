<template>
  <div class="p-2 flex flex-col gap-2 max-w-md mx-auto">
    <div class="p-2 border-gray-400 border-2 rounded-xl text-slate-600">
      <h3 class="text-center font-semibold mb-2">{{ title }}</h3>

      <div class="flex flex-wrap items-center gap-3">
        <select
          v-for="(options, index) in selections"
          :key="index"
          v-model="selected[index]"
          class="flex-grow min-w-[120px] p-2 border border-gray-300 rounded-md"
        >
          <option v-for="option in options" :key="option">{{ option }}</option>
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
      :value="currentText"
      readonly
    ></textarea>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

const props = defineProps({
  title: { type: String, default: "Interpretation" },
  selections: {
    // pole polí hodnot pro jednotlivé dropdowny
    type: Array,
    required: true,
  },
  dataMap: {
    // objekt s daty pro interpretace
    type: Object,
    required: true,
  },
  keyBuilder: {
    // funkce na vytvoření klíče z vybraných hodnot
    type: Function,
    required: true,
  },
});

const emit = defineEmits(["add-interpretation"]);

const selected = ref(props.selections.map((options) => options[0]));
const isAdding = ref(false);

const currentKey = computed(() => props.keyBuilder(selected.value));
const currentText = computed(() => props.dataMap[currentKey.value] || "");

function handleAddText() {
  if (isAdding.value) return;
  isAdding.value = true;

  emit("add-interpretation", {
    key: currentKey.value,
    text: currentText.value,
  });

  setTimeout(() => {
    isAdding.value = false;
  }, 1500);
}

// Reset selected values pokud se změní selections prop
watch(
  () => props.selections,
  (newSelections) => {
    selected.value = newSelections.map((options) => options[0]);
  }
);
</script>
