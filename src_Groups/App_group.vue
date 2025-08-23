<template>
  <div>
    <nav class="p-4 bg-gray-100 flex gap-4">
      <button @click="tab = 'users'">Users</button>
      <button @click="tab = 'groups'">Groups</button>
      <button @click="tab = 'result'">Result</button>
    </nav>

    <div v-if="tab === 'users'">
      <Users :users="users" :groups="groups" @update:users="users = $event" @update:groups="groups = $event" />
    </div>
    <div v-else-if="tab === 'groups'">
      <Groups :groups="groups" :users="users" @update:groups="groups = $event" />
    </div>
    <div v-else-if="tab === 'result'">
      <Result :users="users" />
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import Users from "./components/Users.vue";
import Groups from "./components/Groups.vue";
import Result from "./components/Result.vue";

const tab = ref("users");

const users = ref([]);
const groups = ref([{"id": "group_1", "name": "Family", "users": []}, {"id": "group_2", "name": "Friends", "users": []}, {"id": "group_3", "name": "Job", "users": []}, {"id": "group_4", "name": "Clients", "users": []}, {"id": "group_5", "name": "Celebrites", "users": []}, {"id": "group_6", "name": "Group1", "users": []}, {"id": "group_7", "name": "Group2", "users": []}, {"id": "group_8", "name": "Group3", "users": []}, {"id": "group_9", "name": "Group4", "users": []}, {"id": "group_10", "name": "Group5", "users": []}]);
</script>
