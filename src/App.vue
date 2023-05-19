<script setup lang="ts">
import { ref, computed, h, watch, onMounted } from "vue";
import json from "./vokabeln.json";
import {
  NSelect,
  NCheckbox,
  NDataTable,
  NInputGroup,
  darkTheme,
  NConfigProvider,
  NForm,
  NFormItem,
  deDE,
  dateDeDE,
} from "naive-ui";
import type { GlobalTheme } from "naive-ui";
import type { Ref } from "vue";
import { useLocalStorage } from "@vueuse/core";

// Wort objetc
interface Wort {
  "*": string;
  Wort: string;
  Wortart: string;
  Chinesisch: string;
  Beispielsatz: string;
}

// define mask
let mask = useLocalStorage("mask", {
  Stern: false,
  Wort: false,
  Wortart: false,
  Chinesisch: false,
  Beispielsatz: false,
});

async function setStyle() {
  let style = `
  <style>
    tr td.table-stern {
      color: ${
        mask.value.Stern ? "transparent" : "var(--n-td-text-color)"
      } !important;
    }
    tr td.table-wort {
      color: ${
        mask.value.Wort ? "transparent" : "var(--n-td-text-color)"
      } !important;
    }
    tr td.table-wortart {
      color: ${
        mask.value.Wortart ? "transparent" : "var(--n-td-text-color)"
      } !important;
    }
    tr td.table-chinesisch {
      color: ${
        mask.value.Chinesisch ? "transparent" : "var(--n-td-text-color)"
      } !important;
    }
    tr td.table-beispielsatz {
      color: ${
        mask.value.Beispielsatz ? "transparent" : "var(--n-td-text-color)"
      } !important;
    }
    tr:hover td {
      color: var(--n-td-text-color) !important;
    }
  </style>
  `;
  let styleElement = document.getElementById("style-storage");
  if (styleElement) styleElement.innerHTML = style;
}

onMounted(() => {
  setStyle();
});

// watch mask
watch(mask, () => {
  setStyle();
});

// define Buecher
const Buecher = ref([
  {
    label: "Buch 1",
    value: "Buch_1",
  },
  {
    label: "Buch 2",
    value: "Buch_2",
  },
  {
    label: "Buch 3",
    value: "Buch_3",
  },
]);
const Buch = useLocalStorage("Buch", "Buch_1");

// define Einheiten
const Einheiten = ref([
  {
    label: "Einheit 1",
    value: "Einheit_1",
  },
  {
    label: "Einheit 2",
    value: "Einheit_2",
  },
  {
    label: "Einheit 3",
    value: "Einheit_3",
  },
  {
    label: "Einheit 4",
    value: "Einheit_4",
  },
  {
    label: "Einheit 5",
    value: "Einheit_5",
  },
  {
    label: "Einheit 6",
    value: "Einheit_6",
  },
  {
    label: "Einheit 7",
    value: "Einheit_7",
  },
  {
    label: "Einheit 8",
    value: "Einheit_8",
  },
  {
    label: "Einheit 9",
    value: "Einheit_9",
  },
  {
    label: "Einheit 10",
    value: "Einheit_10",
  },
]);
const Einheit = useLocalStorage("Einheit", "Einheit_1");

// define Teilen
const Teil = useLocalStorage("Teil", {
  Einführung: true,
  Text: true,
  Übungen: true,
  Intentionen: true,
  Hörverstehen: true,
  Leseverstehen: true,
});

// define vokabeln
const vokabeln = computed(() => (json as any)[Buch.value][Einheit.value]);

// define columns
const columns = ref([
  {
    key: "*",
    title: "*",
    className: "table-stern",
  },
  {
    key: "Wort",
    title: "Wort",
    className: "table-wort",
  },
  {
    key: "Wortart",
    title: "Wortart",
    className: "table-wortart",
  },
  {
    key: "Chinesisch",
    title: "Chinesisch",
    className: "table-chinesisch",
    render: (rowData: Wort) => parseText(rowData.Chinesisch),
  },
  {
    key: "Beispielsatz",
    title: "Beispielsatz",
    className: "table-beispielsatz",
    render: (rowData: Wort) => parseText(rowData.Beispielsatz),
  },
]);

const rowKey = (rowData: Wort) => rowData.Wort;

function parseText(text: string) {
  let newText = text.replace(/\n/g, "<br/>");
  return h("div", { innerHTML: newText });
}

// 监视当前主题是否为暗黑模式
const targetNode = document.documentElement;
let theme: Ref<GlobalTheme | undefined> = ref(() =>
  targetNode.classList.contains("dark") ? darkTheme : undefined
);

const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    if (mutation.attributeName === "class") {
      theme.value = targetNode.classList.contains("dark")
        ? darkTheme
        : undefined;
    }
  });
});

observer.observe(targetNode, { attributes: true });
</script>

<template>
  <!-- style storage -->
  <div id="style-storage"></div>

  <n-config-provider :theme="theme" :locale="deDE" :date-locale="dateDeDE">
    <!-- title -->
    <h1>Vokabeln</h1>

    <n-form>
      <n-form-item label="选择课本和单元">
        <n-input-group>
          <n-select v-model:value="Buch" :options="Buecher" />
          <n-select v-model:value="Einheit" :options="Einheiten" />
        </n-input-group>
      </n-form-item>

      <n-form-item label="选择单词表">
        <n-checkbox v-model:checked="Teil.Einführung"> Einführung </n-checkbox>
        <n-checkbox v-model:checked="Teil.Text"> Text </n-checkbox>
        <n-checkbox v-model:checked="Teil.Übungen"> Übungen </n-checkbox>
        <n-checkbox v-model:checked="Teil.Intentionen">
          Intentionen
        </n-checkbox>
        <n-checkbox v-model:checked="Teil.Hörverstehen">
          Hörverstehen
        </n-checkbox>
        <n-checkbox v-model:checked="Teil.Leseverstehen">
          Leseverstehen
        </n-checkbox>
      </n-form-item>

      <n-form-item label="遮住……">
        <n-checkbox v-model:checked="mask.Stern">Stern</n-checkbox>
        <n-checkbox v-model:checked="mask.Wort">Wort</n-checkbox>
        <n-checkbox v-model:checked="mask.Wortart">Wortart</n-checkbox>
        <n-checkbox v-model:checked="mask.Chinesisch">Chinesisch</n-checkbox>
        <n-checkbox v-model:checked="mask.Beispielsatz"
          >Beispielsatz</n-checkbox
        >
      </n-form-item>
    </n-form>

    <!-- show vokabeln tables -->
    <h2>{{ Buch.replace("_", " ") }} {{ Einheit.replace("_", " ") }}</h2>

    <!-- Einführung -->
    <div v-if="vokabeln?.Einführung && Teil.Einführung">
      <h3>Einführung</h3>
      <n-data-table
        :columns="columns"
        :data="vokabeln?.Einführung"
        :row-key="rowKey"
      />
    </div>

    <!-- Text -->
    <div v-if="vokabeln?.Text && Teil.Text">
      <h3>Text</h3>
      <n-data-table
        :columns="columns"
        :data="vokabeln?.Text"
        :row-key="rowKey"
      />
    </div>

    <!-- Übungen -->
    <div v-if="vokabeln?.Übungen && Teil.Übungen">
      <h3>Übungen</h3>
      <n-data-table
        :columns="columns"
        :data="vokabeln?.Übungen"
        :row-key="rowKey"
      />
    </div>

    <!-- Intentionen -->
    <div v-if="vokabeln?.Intentionen && Teil.Intentionen">
      <h3>Intentionen</h3>
      <n-data-table
        :columns="columns"
        :data="vokabeln?.Intentionen"
        :row-key="rowKey"
      />
    </div>

    <!-- Hörverstehen -->
    <div v-if="vokabeln?.Hörverstehen && Teil.Hörverstehen">
      <h3>Hörverstehen</h3>
      <n-data-table
        :columns="columns"
        :data="vokabeln?.Hörverstehen"
        :row-key="rowKey"
      />
    </div>

    <!-- Leseverstehen -->
    <div v-if="vokabeln?.Leseverstehen && Teil.Leseverstehen">
      <h3>Leseverstehen</h3>
      <n-data-table
        :columns="columns"
        :data="vokabeln?.Leseverstehen"
        :row-key="rowKey"
      />
    </div>
  </n-config-provider>
</template>
