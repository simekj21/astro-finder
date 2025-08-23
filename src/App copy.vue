<template>
  <div class="p-4">
    <!-- Nový řádek pro výběr aktivního uživatele -->
    <div
      class="sticky top-0 z-20 mb-2 p-3 rounded-lg bg-red-100 border border-red-300 flex items-center justify-between"
    >
      <span class="font-semibold text-red-700">Active User:</span>
      <select
        v-model="activeUserId"
        class="p-1 border rounded bg-white text-sm"
      >
        <option disabled value="">-- Select --</option>
        <option v-for="u in users" :key="u.id" :value="u.id">
          {{ u.name }}
        </option>
      </select>
    </div>

    <!-- Tabs -->
    <div
      :class="[
        'sticky top-[66px] bg-white z-10 pb-2 mb-4 transition-shadow duration-300',
        hasShadow ? 'shadow-md' : '',
      ]"
    >
      <!-- 1. skupina: Navigace -->
      <div
        class="mb-4 p-3 rounded-lg bg-blue-50 border border-blue-200 flex space-x-4"
      >
        <button
          v-for="tab in firstRowTabs"
          :key="tab.name"
          @click="activeTab = tab.name"
          :class="[
            'pb-2 px-4 font-semibold rounded',
            activeTab === tab.name
              ? 'border-b-2 border-blue-500 text-blue-600 bg-blue-100'
              : 'text-gray-600 hover:text-blue-500 hover:bg-blue-100',
          ]"
        >
          {{ tab.label }}
        </button>
      </div>

      <!-- 2. skupina: Výkladové komponenty -->
      <div
        class="p-3 rounded-lg bg-green-50 border border-green-200 flex flex-wrap space-x-4"
      >
        <button
          v-for="tab in secondRowTabs"
          :key="tab.name"
          @click="activeTab = tab.name"
          :class="[
            'pb-2 px-4 font-semibold mb-1 rounded',
            activeTab === tab.name
              ? 'border-b-2 border-green-500 text-green-600 bg-green-100'
              : 'text-gray-600 hover:text-green-500 hover:bg-green-100',
          ]"
        >
          {{ tab.label }}
        </button>
      </div>
    </div>

    <!-- Obsah -->
    <div>
      <planet-in-sign
        v-if="activeTab === 'planet'"
        :users="users"
        :activeUserId="activeUserId"
        @add-interpretation="addInterpretation"
      />
      <aspects
        v-if="activeTab === 'aspects'"
        :users="users"
        :activeUserId="activeUserId"
        @add-interpretation="addInterpretation"
      />
      <house-in-sign
        v-if="activeTab === 'sun'"
        :users="users"
        :activeUserId="activeUserId"
        @add-interpretation="addInterpretation"
      />
      <planet-in-house
        v-if="activeTab === 'phouses'"
        :users="users"
        :activeUserId="activeUserId"
        @add-interpretation="addInterpretation"
      />
      <synastry
        v-if="activeTab === 'synastry'"
        :users="users"
        :activeUserId="activeUserId"
        @add-interpretation="addInterpretation"
      />
      <div v-if="activeTab === 'users'">
        <Users
          :users="users"
          :groups="groups"
          @update:users="users = $event"
          @update:groups="groups = $event"
          @set-active-user="activeUserId = $event"
        />
      </div>
      <div v-if="activeTab === 'groups'">
        <Groups
          :groups="groups"
          :users="users"
          @update:groups="groups = $event"
        />
      </div>
      <!-- NOVÁ POLOŽKA: Add Data -->
      <AddData
        v-else-if="activeTab === 'adddata'"
        :users="users"
        :active-user-id="activeUserId"
      />
      <result-tab
        v-if="activeTab === 'result'"
        :users="users"
        :groups="groups"
        :activeUserId="activeUserId"
      @remove-item="removeItem"
      @clear-all="clearAll"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount } from "vue";
import PlanetInSign from "./components/PlanetInSign.vue";
import PlanetInHouse from "./components/PlanetInHouse.vue";
import Aspects from "./components/Aspects.vue";
import HouseInSign from "./components/HouseInSign.vue";
import Synastry from "./components/Synastry.vue";
import Groups from "./components/Groups.vue";
import Users from "./components/Users.vue";
import ResultTab from "./components/Result.vue";
import AddData from "@/components/AddData.vue";


const firstRowTabs = [
  { name: "groups", label: "Groups" },
  { name: "users", label: "Users" },
  { name: "adddata", label: "Add Data" },
  { name: "result", label: "Result" },
];
const secondRowTabs = [
  { name: "planet", label: "Planety Zn." },
  { name: "aspects", label: "Aspekty" },
  { name: "sun", label: "Domy" },
  { name: "phouses", label: "Planety Dom." },
  { name: "synastry", label: "Synastry" },
];

const activeTab = ref("planet");
const activeUserId = ref("");

// LocalStorage: USERS
const users = ref(JSON.parse(localStorage.getItem("users") || "[]"));
watch(
  users,
  (val) => {
    localStorage.setItem("users", JSON.stringify(val));
    if (!activeUserId.value && val.length > 0) {
      activeUserId.value = val[0].id;
    }
  },
  { deep: true }
);

// LocalStorage: GROUPS
const groups = ref(JSON.parse(localStorage.getItem("groups") || "[]"));
if (groups.value.length === 0) {
  groups.value = [
    { id: "group_1", name: "Family", users: [] },
    { id: "group_2", name: "Friends", users: [] },
    { id: "group_3", name: "Job", users: [] },
    { id: "group_4", name: "Clients", users: [] },
    { id: "group_5", name: "Celebrites", users: [] },
    { id: "group_6", name: "Group1", users: [] },
    { id: "group_7", name: "Group2", users: [] },
  ];
}
watch(
  groups,
  (val) => {
    localStorage.setItem("groups", JSON.stringify(val));
  },
  { deep: true }
);

function addInterpretation(item) {
  const user = users.value.find((u) => u.id === activeUserId.value);
  if (user) {
    if (!user.results) user.results = [];
    user.results.push(item);
  }
}

const hasShadow = ref(false);
function onScroll() {
  hasShadow.value = window.scrollY > 0;
}
onMounted(() => window.addEventListener("scroll", onScroll));
onBeforeUnmount(() => window.removeEventListener("scroll", onScroll));

function removeItem(index) {
  const user = users.value.find((u) => u.id === activeUserId.value);
  if (user && user.results) {
    user.results.splice(index, 1);
  }
}

function clearAll() {
  const user = users.value.find((u) => u.id === activeUserId.value);
  if (user) {
    user.results = [];
  }
}

</script>


