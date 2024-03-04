<script setup lang="ts">
import { inject } from 'vue'
const page = inject('page')

function changePage(id: number) {
  page.value.forEach((item) => {
    if (item.id === id) {
      item.display = true;
    } else {
      item.display = false;
    }
  });
  console.log(page.value);
}

</script>

<template>
  <div>
    <div class="hidden md:flex fixed z-20 top-7 left-1/2 transform -translate-x-1/2">
      <div class="flex items-center justify-center gap-6 sm:gap-10 rounded-full p-2 bg-color-text/5 bg-opacity-50 backdrop-blur-2xl">
        <div v-for="item in page" :key="item.id">
          <div
            class="flex items-center justify-center cursor-pointer bg-color-primary/10 rounded-full"
            :class="[
              item.display === true
                ? 'text-primary-color font-bold pt-2 pb-2 pl-5 pr-5 rounded-full card-nav-item bg-color-primary/50'
                : 'text-primary-color/70 font-medium pt-2 pb-2 pl-4 pr-4 hover:text-primary-color transition-all duration-300 border-none',
            ]"
            @click="changePage(item.id)"
          >
            <span>{{ item.name }}</span>
          </div>
        </div>
    </div>
  </div>
  </div>
</template>

<style scoped>

</style>