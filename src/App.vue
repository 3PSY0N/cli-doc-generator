<script setup lang="ts">
import { ref, computed, onMounted, watch } from "vue";
import TextareaComponent from "@/components/TextareaComponent.vue";

interface Options {
  description: string;
  command: string;
  type: string;
}

const appTitle = ref("");
const description = ref("");
const usage = ref("");
const examples = ref("");
const options = ref<Array<Options>>([]);

const addOption = () => {
  options.value.push({
    description: "",
    command: "",
    type: "",
  });
};

const removeOption = (index: number) => {
  options.value.splice(index, 1);
};

function formatTextWithSpaces(text: string) {
  const lines = text.split("\n");
  const formattedLines = lines.map((line) => `  │   ${line}`);
  return formattedLines.join("\n");
}

const generatedHelp = computed(() => {
  const formatDescription = formatTextWithSpaces(description.value);
  let text = `
  ┌─────  ${appTitle.value}  ─────
  │
  │ Description:  \n${formatDescription} \n  │`;

  const formatUsage = formatTextWithSpaces(usage.value);
  text += `
  │ Usage: \n${formatUsage}`;

  if (options.value.length > 0) {
    text += `
  │
  │ Options:
`;

    options.value.forEach((option, index) => {
      let typeString = option.type ? `<${option.type}>` : "";
      text += `  │   ${option.command} ${typeString}: ${option.description}`;

      if (index !== options.value.length - 1) {
        text += "\n";
      }
    });
  }

  if (examples.value.length > 0) {
    const formatExamples = formatTextWithSpaces(examples.value);
    text += `\n  |\n  │ Examples: \n${formatExamples}`;
  }

  text += `\n  │\n  └─────`;
  return text;
});

const copyToClipboard = () => {
  navigator.clipboard
    .writeText(generatedHelp.value)
    .then(() => {
      copied.value = true;
      setTimeout(() => {
        copied.value = false;
      }, 3000);
    })
    .catch((err) => {
      console.error("Unable to copy text: ", err);
    });
};

const copied = ref(false);

const saveData = () => {
  localStorage.setItem("appTitle", appTitle.value);
  localStorage.setItem("description", description.value);
  localStorage.setItem("usage", usage.value);
  localStorage.setItem("options", JSON.stringify(options.value));
  localStorage.setItem("examples", examples.value);
};

const loadData = () => {
  appTitle.value = localStorage.getItem("appTitle") || "";
  description.value = localStorage.getItem("description") || "";
  usage.value = localStorage.getItem("usage") || "";
  options.value = JSON.parse(localStorage.getItem("options") || "[]");
  examples.value = localStorage.getItem("examples") || "";
};

onMounted(() => {
  loadData();
});

watch(
  [appTitle, description, usage, options, examples],
  () => {
    saveData();
  },
  { deep: true },
);

const resetForms = () => {
  appTitle.value = "";
  description.value = "";
  usage.value = "";
  options.value = [];
  examples.value = "";
  localStorage.removeItem("appTitle");
  localStorage.removeItem("description");
  localStorage.removeItem("usage");
  localStorage.removeItem("options");
  localStorage.removeItem("examples");
};
</script>

<!-- App.vue -->
<template>
  <div class="p-8 max-w-screen-lg m-auto flex flex-col gap-2">
    <div class="flex gap-4 items-center">
      <h1 class="text-2xl font-bold text-gray-300">Cli Doc Generator</h1>

      <button
        class="hover:bg-red-800 transition-colors mt-2"
        @click="resetForms"
      >
        New
      </button>
    </div>

    <TextareaComponent
      :value="appTitle"
      labelTitle="Title: "
      placeholder="Title"
      @update:value="appTitle = $event"
    />

    <TextareaComponent
      :value="description"
      labelTitle="Description: "
      placeholder="Description"
      @update:value="description = $event"
    />

    <TextareaComponent
      :value="usage"
      labelTitle="Usage: "
      placeholder="Usage"
      @update:value="usage = $event"
    />

    <div class="flex flex-col">
      <fieldset
        class="flex flex-col gap-2 mb-2 border border-gray-700 rounded p-4"
        v-for="(option, index) in options"
        :key="index"
      >
        <legend class="text-gray-300 px-4">Option {{ index + 1 }}</legend>
        <div class="flex justify-between gap-2">
          <input
            class="w-full"
            v-model="option.command"
            placeholder="Command"
          />
          <input class="w-full" v-model="option.type" placeholder="Type" />
        </div>
        <div class="flex gap-2">
          <textarea
            class="w-full"
            style="height: fit-content"
            v-model="option.description"
            placeholder="Description"
            rows="1"
          />
          <button
            class="hover:bg-red-800 transition-colors"
            @click="removeOption(index)"
          >
            <svg class="fill-gray-300 w-6 h-6" viewBox="0 0 24 24">
              <path
                d="m9.4 16.5l2.6-2.6l2.6 2.6l1.4-1.4l-2.6-2.6L16 9.9l-1.4-1.4l-2.6 2.6l-2.6-2.6L8 9.9l2.6 2.6L8 15.1l1.4 1.4ZM7 21q-.825 0-1.413-.588T5 19V6H4V4h5V3h6v1h5v2h-1v13q0 .825-.588 1.413T17 21H7ZM17 6H7v13h10V6ZM7 6v13V6Z"
              />
            </svg>
          </button>
        </div>
      </fieldset>
      <div class="flex justify-center">
        <button
          class="hover:bg-green-800 transition-colors mt-2 w-1/4"
          @click="addOption"
        >
          Add option
        </button>
      </div>
    </div>

    <TextareaComponent
      :value="examples"
      labelTitle="Examples: "
      placeholder="Examples"
      @update:value="examples = $event"
    />

    <hr class="my-12 border-gray-700" />
    <div class="flex flex-col gap-1 relative">
      <label class="text-gray-300" for="generatedHelp">
        Generated help :
        <span v-if="copied" class="text-green-500">Copied !</span>
      </label>

      <pre>{{ generatedHelp }}</pre>

      <button
        class="absolute top-8 right-1 text-white rounded border border-gray-700 hover:!bg-blue-800 transition-colors"
        @click="copyToClipboard"
      >
        <svg class="w-6 h-6 fill-gray-300" viewBox="0 0 256 256">
          <path
            d="M200 28h-34.53a51.88 51.88 0 0 0-74.94 0H56a20 20 0 0 0-20 20v168a20 20 0 0 0 20 20h144a20 20 0 0 0 20-20V48a20 20 0 0 0-20-20Zm-44.29 32h-55.42a28 28 0 0 1 55.42 0ZM196 212H60V52h17.41A52.13 52.13 0 0 0 76 64v8a12 12 0 0 0 12 12h80a12 12 0 0 0 12-12v-8a52.13 52.13 0 0 0-1.41-12H196Z"
          />
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped>
textarea,
input,
pre {
  @apply bg-gray-800 text-gray-300 placeholder-gray-400 rounded px-4 py-2;
}

button {
  @apply bg-gray-800 text-gray-300 rounded px-4 py-2;
}

textarea {
  height: 100px;
}
</style>
