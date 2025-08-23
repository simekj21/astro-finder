<template>
  <div class="p-4 max-w-2xl mx-auto">
    <h2 class="text-xl font-semibold mb-4">Results</h2>

    <ul class="space-y-2">
      <li v-for="user in users" :key="user.id" class="border rounded p-2">
        <div class="font-semibold">{{ user.name }}</div>
        <div class="text-sm text-gray-500">Group: {{ getGroupName(user.groupId) }}</div>
        <div class="mt-1 text-sm text-gray-700">Result data: <i>(here could be result output)</i></div>
      </li>
    </ul>
  </div>
</template>

<script setup>
const props = defineProps({
  users: Array,
});

function getGroupName(groupId) {
  const group = props.users
    .map(u => u.groupId && ({ id: u.groupId, name: u.groupName }))
    .find(g => g && g.id === groupId);
  return group ? group.name : "Unassigned";
}
</script>
