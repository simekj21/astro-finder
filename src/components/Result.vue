<template>
  <div class="p-4 max-w-3xl">
    <h2 class="text-xl font-semibold mb-2">Výsledky uživatele</h2>

    <div class="flex items-center justify-between mb-3">
      <div class="text-sm">
        <span class="text-gray-500">Uživatel:</span>
        <span class="ml-1 inline-block px-2 py-0.5 rounded bg-green-100 text-green-800 font-medium">
          {{ activeUser ? activeUser.name : "—" }}
        </span>
      </div>

      <div class="flex items-center gap-2">
        <button
          class="px-3 py-1.5 rounded bg-emerald-600 text-white hover:bg-emerald-700"
          @click="generateSummary"
          :disabled="!items.length"
          title="Vytvořit souhrn aktuálních výsledků"
        >
          Generovat souhrn
        </button>
        <button
          class="px-3 py-1.5 rounded bg-red-600 text-white hover:bg-red-700"
          :disabled="!items.length"
          @click="confirmClearAll"
          title="Smazat všechny výsledky aktivního uživatele"
        >
          Vymazat vše ({{ items.length }})
        </button>

      </div>
    </div>

    <div v-if="!items.length" class="text-sm text-gray-500">
      Zatím tu nic není. Přidej položky v <strong>Add Data</strong>.
    </div>

    <ul v-else class="rounded border divide-y bg-white">
      <li v-for="(item, index) in items" :key="item.id || index" class="bg-gray-50/40">
        <details>
          <summary
            class="flex items-center justify-between px-3 py-2 cursor-pointer select-none hover:bg-gray-50"
          >
            <span class="font-medium">
              {{ titleOf(item, index) }}
            </span>

            <div class="flex items-center gap-3">
              <button
                class="text-gray-500 hover:text-red-600"
                title="Odstranit položku"
                @click.stop="$emit('remove-item', index)"
              >
                ✕
              </button>
            </div>
          </summary>

          <div class="px-3 pb-3">
            <div class="text-sm text-gray-800 whitespace-pre-wrap" v-if="item.text">
              {{ item.text }}
            </div>

            <div class="grid sm:grid-cols-2 gap-2 text-sm mt-2">
              <template v-for="(val, key) in visibleFields(item)" :key="key">
                <div class="flex justify-between gap-2 border rounded px-2 py-1 bg-white">
                  <span class="text-gray-500">{{ key }}</span>
                  <span class="font-mono text-gray-800 text-right break-all">{{ val }}</span>
                </div>
              </template>
            </div>

            <details class="mt-3">
              <summary class="text-xs text-gray-500 cursor-pointer">Zobrazit surová data</summary>
              <pre class="mt-2 text-xs overflow-auto bg-gray-900 text-gray-100 p-3 rounded">
{{ pretty(item) }}
              </pre>
            </details>
          </div>
        </details>
      </li>
    </ul>

    <!-- Overlay souhrn -->
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
            aria-label="Zavřít"
          >
            &times;
          </button>

          <h2 class="text-2xl mb-4 font-semibold">Souhrn výsledků</h2>

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
                <strong>{{ titleOf(item, i) }}</strong>
                <p class="whitespace-pre-wrap mt-1">{{ item.text || '(bez textu)' }}</p>
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
                <strong>{{ titleOf(item, i) }}</strong>
                <p class="whitespace-pre-wrap mt-1">{{ item.text || '(bez textu)' }}</p>
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
                <strong>{{ titleOf(item, i) }}</strong>
                <p class="whitespace-pre-wrap mt-1">{{ item.text || '(bez textu)' }}</p>
              </div>
            </div>

            <div v-if="summary.other.length > 0">
              <h3 class="text-xl font-bold mb-2 border-b border-gray-300 pb-1">
                Ostatní
              </h3>
              <div
                v-for="(item, i) in summary.other"
                :key="'other-' + i"
                class="mb-3 p-3 border rounded bg-gray-50"
              >
                <strong>{{ titleOf(item, i) }}</strong>
                <p class="whitespace-pre-wrap mt-1">{{ item.text || '(bez textu)' }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { computed, ref } from "vue";

const props = defineProps({
  users: { type: Array, required: true },
  activeUserId: { type: [String, Number], required: false, default: "" },
  groups: { type: Array, required: false, default: () => [] },
});
const emit = defineEmits(["remove-item", "clear-all"]);

const activeUser = computed(() => props.users.find(u => u.id === props.activeUserId) || null);
const items = computed(() => activeUser.value?.results || []);

/** Modal souhrnu */
const showSummaryOverlay = ref(false);
const summary = ref({ planet: [], aspects: [], houses: [], other: [] });

/** Titulek položky s chytrým fallbackem */
function cap(s) {
  if (!s || typeof s !== "string") return s;
  return s.charAt(0).toUpperCase() + s.slice(1);
}
function titleOf(item, idx) {
  if (item?.key) return item.key;
  if (item?.title) return item.title;
  if (item?.name) return item.name;

  if (item?.planet && item?.sign) return `${cap(item.planet)} in ${cap(item.sign)}`;
  if (item?.planet && item?.house) return `${cap(item.planet)} in ${item.house}`;

  const p1 = item?.p1En || item?.k1;
  const asp = item?.aspectName;
  const p2 = item?.p2En || item?.k2;
  if ((p1 && asp && p2) || (item?.type?.toLowerCase?.().includes('aspect'))) {
    const a = cap(p1 || "?");
    const b = cap(asp || "Aspect");
    const c = cap(p2 || "?");
    return `${a} ${b} ${c}`;
  }

  if (item?.type) return `${cap(item.type)} #${idx + 1}`;
  return `Položka #${idx + 1}`;
}

/** Viditelné pole–hodnota pro detail */
function visibleFields(item) {
  const order = [
    "type", "key", "title", "name",
    "planet", "sign", "house",
    "p1En", "aspectName", "p2En",
    "orb", "motion",
    "id",
  ];
  const out = {};
  for (const k of order) {
    if (item?.[k] !== undefined && item?.[k] !== null && item?.[k] !== "") {
      out[k] = String(item[k]);
    }
  }
  for (const k of Object.keys(item || {})) {
    if (!(k in out) && item[k] !== undefined && item[k] !== null && item[k] !== "") {
      out[k] = typeof item[k] === "object" ? JSON.stringify(item[k]) : String(item[k]);
    }
  }
  return out;
}

function pretty(obj) {
  try { return JSON.stringify(obj, null, 2); } catch { return String(obj); }
}

/** Souhrn (seskupení) */
function categorize(item) {
  const t = (item?.type || "").toLowerCase();
  if (t.includes("planet") || (item?.planet && item?.sign)) return "planet";
  if (t.includes("house") || item?.house) return "houses";
  if (t.includes("aspect") || item?.aspectName) return "aspects";
  return "other";
}

function generateSummary() {
  const res = { planet: [], aspects: [], houses: [], other: [] };
  for (const it of items.value) {
    const cat = categorize(it);
    res[cat].push(it);
  }
  summary.value = res;
  showSummaryOverlay.value = true;
}

const confirmClearAll = () => {
  if (confirm('Opravdu chceš smazat všechny výsledky tohoto uživatele?')) {
    emit('clear-all');
  }
};

function closeSummary() {
  showSummaryOverlay.value = false;
}

function copyToClipboard() {
  let textToCopy = "";
  const blocks = [
    { label: "Planety", data: summary.value.planet },
    { label: "Aspekty", data: summary.value.aspects },
    { label: "Domy", data: summary.value.houses },
    { label: "Ostatní", data: summary.value.other },
  ];

  blocks.forEach(({ label, data }) => {
    if (data.length > 0) {
      textToCopy += `${label}:\n`;
      data.forEach((item, i) => {
        textToCopy += `${titleOf(item, i)}:\n${item.text || "(bez textu)"}\n\n`;
      });
    }
  });

  navigator.clipboard
    .writeText(textToCopy)
    .then(() => alert("Souhrn zkopírován do schránky!"))
    .catch((err) => alert("Nepodařilo se zkopírovat text: " + err));
}
</script>

<style scoped>
details > summary::-webkit-details-marker { display: none; }

/* Fade pro modal */
.fade-enter-active,
.fade-leave-active { transition: opacity 0.2s ease; }
.fade-enter-from,
.fade-leave-to { opacity: 0; }
</style>
