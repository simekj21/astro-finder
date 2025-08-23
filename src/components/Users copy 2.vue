<template>
  <div class="p-4 max-w-3xl mx-auto space-y-4">
    <h2 class="text-xl font-semibold">Users</h2>

    <!-- FORM -->
    <div class="space-y-2">
      <div class="flex flex-col sm:flex-row gap-2">
        <input
          v-model="newUserName"
          placeholder="User name"
          class="p-2 border rounded flex-1"
        />
        <select v-model="selectedGroupId" class="p-2 border rounded flex-1">
          <option disabled value="">Select group</option>
          <option v-for="group in groups" :key="group.id" :value="group.id">
            {{ group.name }}
          </option>
        </select>
      </div>
      <button
        @click="addUser"
        class="bg-blue-600 text-white px-4 py-2 rounded w-full sm:w-auto"
      >
        Add
      </button>
    </div>

    <!-- SEZNAM -->
    <ul class="space-y-2">
      <li
        v-for="user in users"
        :key="user.id"
        class="border rounded p-2 flex justify-between items-center"
      >
        <div @click="selectUser(user)" class="cursor-pointer">
          <span class="font-semibold">{{ user.name }}</span>
          <span class="text-sm text-gray-500 ml-2"
            >({{ getGroupName(user.groupId) }})</span
          >
        </div>
        <div class="space-x-2">
          <button @click="editUser(user)" class="text-yellow-600 text-sm">
            Edit
          </button>
          <button @click="removeUser(user.id)" class="text-red-600 text-sm">
            Remove
          </button>
        </div>
      </li>
    </ul>

    <!-- VÝSLEDKY -->
    <div
      v-if="selectedUser"
      class="border p-4 rounded space-y-2 bg-white shadow"
    >
      <h4 class="font-semibold text-lg mb-2">
        Výsledky uživatele: {{ selectedUser.name }}
      </h4>
      <div
        v-for="(item, index) in selectedUser.results || []"
        :key="index"
        class="p-3 border rounded bg-gray-50"
      >
        <div class="text-xs text-gray-500 mb-1">
          {{ item.type }} • {{ item.key }}
        </div>
        <div class="whitespace-pre-wrap text-gray-800">
          {{ item.text }}
        </div>
      </div>
      <p
        v-if="!selectedUser.results?.length"
        class="text-sm text-gray-400 italic"
      >
        Žádné výsledky.
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  users: Array,
  groups: Array,
  activeUserId: String,
});

const emit = defineEmits(["update:users", "update:groups", "set-active-user"]);

const newUserName = ref("");
const selectedGroupId = ref("");
const selectedUser = ref(null);

function addUser() {
  if (!newUserName.value || !selectedGroupId.value) return;

  const newUser = {
    id: "user_" + Date.now(),
    name: newUserName.value,
    groupId: selectedGroupId.value,
    results: [],
  };

  const updatedUsers = [...props.users, newUser];

  const updatedGroups = props.groups.map((group) => {
    if (group.id === selectedGroupId.value) {
      return {
        ...group,
        users: [...group.users, newUser],
      };
    }
    return group;
  });

  emit("update:users", updatedUsers);
  emit("update:groups", updatedGroups);

  // automaticky vybrat nového jako aktivního
  emit("set-active-user", newUser.id);

  newUserName.value = "";
  selectedGroupId.value = "";
}

function removeUser(userId) {
  const updatedUsers = props.users.filter((u) => u.id !== userId);
  const updatedGroups = props.groups.map((group) => ({
    ...group,
    users: group.users.filter((u) => u.id !== userId),
  }));

  emit("update:users", updatedUsers);
  emit("update:groups", updatedGroups);

  if (selectedUser.value?.id === userId) {
    selectedUser.value = null;
  }
}

function editUser(user) {
  const newName = prompt("Nové jméno uživatele:", user.name);
  if (newName) {
    const updatedUsers = props.users.map((u) =>
      u.id === user.id ? { ...u, name: newName } : u
    );
    emit("update:users", updatedUsers);
  }
}

function selectUser(user) {
  selectedUser.value = user;
  emit("set-active-user", user.id);
}

function getGroupName(groupId) {
  const group = props.groups.find((g) => g.id === groupId);
  return group ? group.name : "Unassigned";
}
</script>
