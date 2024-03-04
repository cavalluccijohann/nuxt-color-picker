<script setup lang="ts">
import { provide } from 'vue'

// 1 is the page with the color picker HSL
// 2 is the page with the color picker ALL
const page = ref([
  { id: 1, name: 'HSL', display: true},
  { id: 2, name: 'ALL', display: false}
]);
provide('page', page)
</script>

<template>
  <Html lang="en">
    <Body class="selection:bg-neutral-200 dark:selection:bg-neutral-700 bg-color-background relative">
      <NuxtLayout>
        <NuxtPage />
      </NuxtLayout>
    </Body>
  </Html>
</template>
