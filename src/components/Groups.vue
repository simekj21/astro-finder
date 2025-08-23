<template>
  <div class="p-4 max-w-2xl mx-auto">
    <h2 class="text-xl font-semibold mb-4">Groups</h2>

    <!-- Formulář pro vytvoření nové skupiny -->
    <div class="flex gap-2 mb-4">
      <input v-model="newGroupName" placeholder="New group name"
             class="flex-1 p-2 border rounded" />
      <button @click="addGroup"
              class="bg-green-600 text-white px-4 py-2 rounded">Add</button>
    </div>

    <!-- Výpis skupin -->
    <ul class="space-y-4">
      <li v-for="group in groups" :key="group.id" class="border rounded p-4">
        <div class="flex justify-between items-center mb-2">
          <input v-model="group.name"
                 class="text-lg font-semibold border-b w-full mr-2" />
          <button @click="deleteGroup(group.id)"
                  class="text-red-600">Delete</button>
        </div>

        <!-- Seznam uživatelů ve skupině -->
        <ul v-if="group.users.length" class="ml-4 list-disc text-sm text-slate-700">
          <li v-for="user in group.users" :key="user.id" class="flex justify-between">
            {{ user.name }}
            <button @click="removeUserFromGroup(group.id, user.id)"
                    class="text-xs text-red-500">remove</button>
          </li>
        </ul>

        <p v-else class="text-sm italic text-gray-400">No users in this group.</p>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

const props = defineProps({
  groups: Array,
  users: Array,
});

const emit = defineEmits(["update:groups"]);

const newGroupName = ref("");

function addGroup() {
  if (!newGroupName.value.trim()) return;
  const newGroup = {
    id: "group_" + Date.now(),
    name: newGroupName.value,
    users: [],
  };
  emit("update:groups", [...props.groups, newGroup]);
  newGroupName.value = "";
}

function deleteGroup(groupId) {
  const updated = props.groups.filter((g) => g.id !== groupId);
  emit("update:groups", updated);
}

function removeUserFromGroup(groupId, userId) {
  const updated = props.groups.map((group) => {
    if (group.id === groupId) {
      return {
        ...group,
        users: group.users.filter((u) => u.id !== userId),
      };
    }
    return group;
  });
  emit("update:groups", updated);
}
</script>
