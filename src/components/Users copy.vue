/*
<template>
  <div class="space-y-4">
    <h3 class="text-xl font-semibold">Správa uživatelů</h3>

    <!-- Form pro přidání -->
    <div class="flex space-x-2">
      <input
        v-model="newUserName"
        type="text"
        placeholder="Jméno uživatele"
        class="border rounded p-2 flex-1"
      />
      <button @click="addUser" class="bg-blue-500 text-white px-4 rounded">
        Přidat
      </button>
    </div>

    <!-- Seznam uživatelů -->
    <div v-if="users.length > 0" class="space-y-2">
      <div
        v-for="(user, index) in users"
        :key="user.id"
        class="border rounded p-2 flex justify-between items-center"
      >
        <div @click="selectUser(user)" class="cursor-pointer hover:underline">
          <strong>{{ user.name }}</strong>
        </div>
        <div class="space-x-2">
          <button
            @click="editUser(index)"
            class="text-yellow-500 hover:underline"
          >
            Edit
          </button>
          <button
            @click="deleteUser(index)"
            class="text-red-500 hover:underline"
          >
            Smazat
          </button>
        </div>
      </div>
    </div>
    <p v-else class="text-gray-600">Žádní uživatelé</p>

    <!-- Zobrazení výsledků vybraného uživatele -->
    <div
      v-if="selectedUser"
      class="border p-4 rounded space-y-2 bg-white shadow"
    >
      <h4 class="font-semibold text-lg mb-2">
        Výsledky uživatele: {{ selectedUser.name }}
      </h4>
      <div
        v-for="(item, index) in parsedResults"
        :key="index"
        class="p-3 border rounded bg-gray-50"
      >
        <p class="text-sm text-gray-500 font-semibold mb-1">{{ item.key }}</p>
        <p class="text-gray-800 whitespace-pre-wrap">{{ item.text }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from "vue";

const props = defineProps({
  results: { type: Array, default: () => [] },
});

const users = ref(JSON.parse(localStorage.getItem("users") || "[]"));
const newUserName = ref("");
const selectedUser = ref(null);

// Watch pro automatické ukládání
watch(
  users,
  (newVal) => localStorage.setItem("users", JSON.stringify(newVal)),
  { deep: true }
);

// Přidání uživatele
function addUser() {
  if (!newUserName.value) return;
  users.value.push({
    id: Date.now(),
    name: newUserName.value,
    result: props.results, // uložíme rovnou jako objekt
  });
  newUserName.value = "";
}

// Smazání uživatele
function deleteUser(index) {
  const wasSelected = selectedUser.value?.id === users.value[index]?.id;
  users.value.splice(index, 1);
  if (wasSelected) selectedUser.value = null;
}

// Editace jména
function editUser(index) {
  const newName = prompt("Nové jméno:", users.value[index].name);
  if (newName) users.value[index].name = newName;
}

// Výběr uživatele
function selectUser(user) {
  selectedUser.value = user;
}

// Převod na pole objektů (pokud výsledek byl uložen jako JSON string)
const parsedResults = computed(() => {
  if (!selectedUser.value) return [];
  const res = selectedUser.value.result;
  try {
    return typeof res === "string" ? JSON.parse(res) : res;
  } catch {
    return [];
  }
});
</script>
