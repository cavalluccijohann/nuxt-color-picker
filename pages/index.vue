<script lang="ts" setup>
import Footer from "~/components/layout/Footer.vue";
import { inject } from 'vue'
const page = inject('page')
definePageMeta({
  title: "Home",
  name: "Home",
  path: "/",
  description: "Home page",
  keywords: "Home",
});
</script>

<template>
  <div class="flex flex-col h-screen justify-between">
    <div class="flex flex-center justify-center h-screen w-screen items-center">
      <ColorPicker v-if="page[0].display === true" />
      <ColorPickerAll v-if="page[1].display === true" />
    </div>
    {{ page }}
  </div>
</template>

<style scoped>

</style>