<script lang="ts" setup>
import { inject } from 'vue'
const page = inject('page')
</script>

<template>
  <div class="flex flex-col h-screen justify-between">
    <div class="flex flex-center justify-center h-screen w-screen items-center">
      <ColorPickerHSL v-if="page[0].display === true" />
      <ColorPickerAll v-if="page[1].display === true" />
    </div>
  </div>
</template>

<style scoped>

</style>