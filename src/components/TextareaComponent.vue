<script setup lang="ts">
const { value, placeholder, labelTitle } = defineProps([
  "value",
  "placeholder",
  "labelTitle",
]);
const emit = defineEmits();
</script>

<template>
  <div class="flex flex-col gap-1">
    <label class="text-gray-300" for="description">{{ labelTitle }}</label>
    <textarea
      class="bg-gray-800 text-gray-300 placeholder-gray-400 rounded px-4 py-2 h-24"
      :value="value"
      @input="
        (event: Event) => {
          const target = event.target as HTMLTextAreaElement;
          emit('update:value', target.value);
        }
      "
      :placeholder="placeholder"
    ></textarea>
  </div>
</template>
