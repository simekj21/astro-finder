<template>
  <div class="p-4 max-w-2xl mx-auto">
    <h2 class="text-xl font-semibold mb-4">Users</h2>

    <div class="flex gap-2 mb-4">
      <input v-model="newUserName" placeholder="User name"
             class="flex-1 p-2 border rounded" />
      <select v-model="selectedGroupId" class="border p-2 rounded">
        <option disabled value="">Select group</option>
        <option v-for="group in groups" :key="group.id" :value="group.id">{{ group.name }}</option>
      </select>
      <button @click="addUser"
              class="bg-blue-600 text-white px-4 py-2 rounded">Add</button>
    </div>

    <ul class="space-y-2">
      <li v-for="user in users" :key="user.id" class="border rounded p-2 flex justify-between items-center">
        {{ user.name }} <span class="text-sm text-gray-500">({{ getGroupName(user.groupId) }})</span>
        <button @click="removeUser(user.id)" class="text-red-500 text-sm">Remove</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  users: Array,
  groups: Array,
});
const emit = defineEmits(["update:users", "update:groups"]);

const newUserName = ref("");
const selectedGroupId = ref("");

function addUser() {
  if (!newUserName.value || !selectedGroupId.value) return;

  const newUser = {
    id: "user_" + Date.now(),
    name: newUserName.value,
    groupId: selectedGroupId.value,
  };

  const updatedUsers = [...props.users, newUser];

  // Add user to group
  const updatedGroups = props.groups.map(group => {
    if (group.id === selectedGroupId.value) {
      return {
        ...group,
        users: [...group.users, newUser]
      };
    }
    return group;
  });

  emit("update:users", updatedUsers);
  emit("update:groups", updatedGroups);

  newUserName.value = "";
  selectedGroupId.value = "";
}

function removeUser(userId) {
  const updatedUsers = props.users.filter(u => u.id !== userId);
  const updatedGroups = props.groups.map(group => ({
    ...group,
    users: group.users.filter(u => u.id !== userId)
  }));
  emit("update:users", updatedUsers);
  emit("update:groups", updatedGroups);
}

function getGroupName(groupId) {
  const group = props.groups.find(g => g.id === groupId);
  return group ? group.name : "Unassigned";
}
</script>
