<template>
  <div class="p-4 max-w-4xl mx-auto space-y-4">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold">Add Data</h2>
      <div class="text-sm text-gray-500">
        Vlož řádky a klikni na <span class="font-semibold">Parse &amp; Save</span>.
      </div>
    </div>

    <textarea
      v-model="raw"
      class="w-full min-h-[260px] border rounded p-3 focus:outline-none focus:ring"
      placeholder="Např.:
Sun in Capricorn 8°23’, in 1st House
Moon in Virgo 19°27’, in 8th House
ASC in Capricorn 0°36’
MC in Scorpio 4°03’"
    ></textarea>

    <div class="flex items-center gap-3">
      <button
        @click="handleParse"
        class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700 disabled:opacity-50"
        :disabled="!raw.trim() || !activeUser"
      >
        Parse &amp; Save
      </button>
      <button
        @click="clearText"
        class="px-4 py-2 rounded bg-gray-100 hover:bg-gray-200"
      >
        Clear
      </button>
      <span class="text-sm text-gray-500" v-if="activeUser">
        Ukládám do uživatele: <strong>{{ activeUser.name }}</strong>
      </span>
      <span class="text-sm text-red-600" v-else>
        Není vybraný Active User.
      </span>
    </div>

    <div v-if="messages.length" class="space-y-1">
      <div v-for="(m, i) in messages" :key="i" class="text-sm" :class="m.type === 'error' ? 'text-red-600' : 'text-green-700'">
        {{ m.text }}
      </div>
    </div>

    <div v-if="preview.length" class="mt-4">
      <h3 class="text-lg font-semibold mb-2">Preview výsledků (než se uloží):</h3>
      <ul class="list-disc pl-5 space-y-1 text-sm">
        <li v-for="(r, i) in preview" :key="i">
          <strong>{{ r.type }}</strong> — {{ r.title }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from "vue";

/**
 * === IMPORT DAT ===
 * Uprav cesty podle projektu. Každý modul by měl exportovat objekt { [key: string]: string }.
 */
import * as planetInSignMod from "@/planetinsign";
import * as planetInHouseMod from "@/planetinhouse";
import * as houseInSignMod from "@/houseinsign";

// Podpora default i pojmenovaných exportů
const planetInSignObj =
  (planetInSignMod.default && typeof planetInSignMod.default === "object")
    ? planetInSignMod.default
    : (planetInSignMod.planetInSign || planetInSignMod);

const planetInHouseObj =
  (planetInHouseMod.default && typeof planetInHouseMod.default === "object")
    ? planetInHouseMod.default
    : (planetInHouseMod.planetInHouse || planetInHouseMod);

const houseInSignObj =
  (houseInSignMod.default && typeof houseInSignMod.default === "object")
    ? houseInSignMod.default
    : (houseInSignMod.houseInSign || houseInSignMod);

/**
 * PROPS z App.vue
 */
const props = defineProps({
  users: { type: Array, required: true },
  activeUserId: { type: [String, Number], required: false, default: "" },
});

const raw = ref("");
const messages = ref([]);
const preview = ref([]);

const activeUser = computed(() => props.users.find(u => u.id === props.activeUserId));

function normalize(s) {
  return s
    .replace(/[’']/g, "'")
    .replace(/[°]/g, "°")
    .replace(/\s+/g, " ")
    .trim();
}

const SIGN_MAP = {
  aries: "aries",
  taurus: "taurus",
  gemini: "gemini",
  cancer: "cancer",
  leo: "leo",
  virgo: "virgo",
  libra: "libra",
  scorpio: "scorpio",
  sagittarius: "sagittarius",
  capricorn: "capricorn",
  aquarius: "aquarius",
  pisces: "pisces",
};

const PLANET_KEY_EN = {
  sun: "sun",
  moon: "moon",
  mercury: "mercury",
  venus: "venus",
  mars: "mars",
  jupiter: "jupiter",
  saturn: "saturn",
  uranus: "uranus",
  neptune: "neptune",
  pluto: "pluto",
  "north node": "northnode",
  lilith: "lilith",
  chiron: "chiron",
  fortune: "fortune",
  vertex: "vertex",
  asc: "asc",
  mc: "mc",
};

// Pro planetinhouse (CZ názvy – uprav dle svých klíčů)
const PLANET_KEY_CZ = {
  sun: "sun",      // změň na "slunce" pokud tak máš klíče
  moon: "luna",
  mercury: "merkur",
  venus: "venuse",
  mars: "mars",
  jupiter: "jupiter",
  saturn: "saturn",
  uranus: "uran",
  neptune: "neptun",
  pluto: "pluto",
};

function keyPlanetInSign(planetEn, signEn) {
  return `${planetEn}in${signEn}`; // např. mooninvirgo
}
function keyPlanetInHouse(planetEn, houseNum) {
  const cz = PLANET_KEY_CZ[planetEn] || planetEn;
  return `${cz}inhouse${houseNum}`; // např. lunainhouse8
}
function keyHouseInSign(houseNum, signEn) {
  return `house${houseNum}in${signEn}`; // např. house8invirgo
}

const LINE_RE = new RegExp(
  String.raw`^\s*` +
  String.raw`(?<planet>[A-Za-z ]+?)` +
  String.raw`\s+in\s+` +
  String.raw`(?<sign>[A-Za-z]+)` +
  String.raw`(?:\s+[0-9]{1,2}°[0-9]{1,2}['’]?,?)?` +
  String.raw`(?:\s*,\s*in\s+(?<house>\d{1,2})(?:st|nd|rd|th)\s+House)?` +
  String.raw`\s*$`,
  "i"
);

// ASC/MC zatím ignorujeme (můžeš snadno doplnit)
const SPECIAL_RE = /^\s*(ASC|MC)\s+in\s+[A-Za-z]+\s+[0-9]{1,2}°[0-9]{1,2}['’]?\s*$/i;

function parseAll(text) {
  const lines = normalize(text).split(/\n+/);
  const items = [];
  const errors = [];

  for (let rawLine of lines) {
    const line = rawLine.trim();
    if (!line) continue;

    if (SPECIAL_RE.test(line)) continue;

    const m = line.match(LINE_RE);
    if (!m) {
      errors.push(`Nepodařilo se rozpoznat řádek: "${line}"`);
      continue;
    }

    let planet = (m.groups.planet || "").trim().toLowerCase();
    let sign = (m.groups.sign || "").trim().toLowerCase();
    let house = m.groups.house ? parseInt(m.groups.house, 10) : null;

    planet = planet.replace(/\s+/g, " ");
    if (!(planet in PLANET_KEY_EN)) {
      if (["north node", "lilith", "chiron", "fortune", "vertex"].includes(planet)) {
        continue; // tyto body zatím přeskočíme
      }
      planet = planet.split(" ")[0];
    }

    const planetEn = PLANET_KEY_EN[planet] || planet;
    const signEn = SIGN_MAP[sign];
    if (!signEn) {
      errors.push(`Neznámé znamení: "${sign}" v řádku "${line}"`);
      continue;
    }

    const k1 = keyPlanetInSign(planetEn, signEn);
    const v1 = planetInSignObj[k1];

    let v2 = null, v3 = null;
    let k2 = null, k3 = null;

    if (house && house >= 1 && house <= 12) {
      k2 = keyPlanetInHouse(planetEn, house);
      v2 = planetInHouseObj[k2];

      k3 = keyHouseInSign(house, signEn);
      v3 = houseInSignObj[k3];
    }

    if (v1) {
      items.push({
        type: "PlanetInSign",
        title: `${capitalize(planetEn)} in ${capitalize(signEn)}`,
        text: v1,
      });
    } else {
      errors.push(`Nenašel jsem klíč v planetinsign: "${k1}"`);
    }

    if (house) {
      if (v2) {
        items.push({
          type: "PlanetInHouse",
          title: `${capitalize(planetEn)} in ${house}. House`,
          text: v2,
        });
      } else {
        errors.push(`Nenašel jsem klíč v planetinhouse: "${k2}"`);
      }

      if (v3) {
        items.push({
          type: "HouseInSign",
          title: `${house}. House in ${capitalize(signEn)}`,
          text: v3,
        });
      } else {
        errors.push(`Nenašel jsem klíč v houseinsign: "${k3}"`);
      }
    }
  }

  return { items, errors };
}

function capitalize(s) { return s ? s.charAt(0).toUpperCase() + s.slice(1) : s; }

function clearText() {
  raw.value = "";
  preview.value = [];
  messages.value = [];
}

function handleParse() {
  messages.value = [];
  preview.value = [];

  if (!activeUser.value) {
    messages.value.push({ type: "error", text: "Vyber prosím Active User." });
    return;
  }

  const { items, errors } = parseAll(raw.value);
  preview.value = items;

  if (items.length) {
    if (!Array.isArray(activeUser.value.results)) {
      activeUser.value.results = [];
    }
    const now = new Date().toISOString();
    for (const it of items) {
      activeUser.value.results.push({
        ...it,
        id: `${now}-${Math.random().toString(36).slice(2, 8)}`,
        timestamp: now,
      });
    }
    messages.value.push({ type: "ok", text: `Uloženo ${items.length} položek do výsledků uživatele ${activeUser.value.name}.` });
  }

  if (errors.length) {
    for (const e of errors) messages.value.push({ type: "error", text: e });
  } else {
    messages.value.push({ type: "ok", text: "Hotovo bez chyb." });
  }
}
</script>
