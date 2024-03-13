<script setup lang="ts">
import { inject } from 'vue'

const eventSidePannel = ref(true);

const colorType = ref('hsl');

// Fonction pour la conversion HSB en HSL
function hsb2hsl(h: number, s: number, b: number) {
  const hsl = {
    h: h,
    l: (2 - s) * b,
    s: s * b
  };

  if (hsl.l <= 1 && hsl.l > 0) {
    hsl.s /= hsl.l;
  } else {
    hsl.s /= 2 - hsl.l;
  }

  hsl.l /= 2;

  if (hsl.s > 1) {
    hsl.s = 1;
  }

  if (!hsl.s > 0) hsl.s = 0;

  hsl.h *= 360;
  hsl.s *= 100;
  hsl.l *= 100;

  return hsl;
}

// Variables réactives
const isVisible = ref(true);
const h = ref(265);
const s = ref(80);
const l = ref(99);

const hex =  ref('#ff0000');

const r = ref(255);
const g = ref(0);
const b = ref(0);

// Calculs calculés
const color = ref(computed(() => {
  const hsl = hsb2hsl(h.value / 360, s.value / 100, l.value / 100);
  return `hsl(${hsl.h}, ${hsl.s}%, ${hsl.l}%)`;
}));

const colorString = ref(computed(() => {
  return `${h.value}, ${s.value}%, ${l.value}%`;
}));

const gradientH = ref(computed(() => {
  const stops = [];
  for (let i = 0; i < 7; i++) {
    const hue = i * 60;
    const hsl = hsb2hsl(hue / 360, s.value / 100, l.value / 100);
    stops.push(`hsl(${hsl.h}, ${hsl.s}%, ${hsl.l}%)`);
  }
  return {
    backgroundImage: `linear-gradient(to right, ${stops.join(', ')})`
  };
}));

// Méthode pour générer le dégradé en fonction de la valeur hexadécimale
function generateHexGradient() {
  const gradientColors = [];

  // Générer les couleurs intermédiaires pour chaque valeur hexadécimale possible
  for (let i = 0; i <= 0xFFFFFF; i++) {
    gradientColors.push(`#${i.toString(16).padStart(6, '0')}`);
  }

  return gradientColors;
}

const gradientS = ref(computed(() => {
  const stops = [];
  const hsl1 = hsb2hsl(h.value / 360, 0, l.value / 100);
  const hsl2 = hsb2hsl(h.value / 360, 1, l.value / 100);
  stops.push(`hsl(${hsl1.h}, ${hsl1.s}%, ${hsl1.l}%)`);
  stops.push(`hsl(${hsl2.h}, ${hsl2.s}%, ${hsl2.l}%)`);
  return {
    backgroundImage: `linear-gradient(to right, ${stops.join(', ')})`
  };
}));

const gradientL = ref(computed(() => {
  const stops = [];
  const hsl1 = hsb2hsl(h.value / 360, s.value / 100, 0);
  const hsl2 = hsb2hsl(h.value / 360, s.value / 100, 1);
  stops.push(`hsl(${hsl1.h}, ${hsl1.s}%, ${hsl1.l}%)`);
  stops.push(`hsl(${hsl2.h}, ${hsl2.s}%, ${hsl2.l}%)`);
  return {
    backgroundImage: `linear-gradient(to right, ${stops.join(', ')})`
  };
}));

// Méthodes
const show = () => {
  isVisible.value = true;
};

const hide = () => {
  isVisible.value = false;
};

const toggle = () => {
  isVisible.value = !isVisible.value;
};

// Initialisation
const initColor = () => {
  h.value = Math.floor(Math.random() * 360);
};

const updateColor = () => {
  color.value = `hsl(${h.value}, ${s.value}%, ${l.value}%)`;
  colorString.value = `${h.value}, ${s.value}%, ${l.value}%`;
  document.body.style.backgroundColor = color.value;
};

initColor();

</script>

<template>
  <div class="flex w-full h-full justify-center items-center">
    <div class="w-100 h-100 fixed z-0" v-if="isVisible">
      <transition name="pop">
        <div class="w-60 border-[1px] border-color-background bg-color-background z-2 rounded-lg shadow-md" v-if="isVisible">
          <div class="h-64 flex justify-center items-center text-white rounded-t-lg" :style="{ 'background': color }">
            <div class="p-1 py-6 text-center">
              <h1 class="text-white m-0 text-4xl tracking-wide font-helvetica-neue text-shadow">{{ colorType }}</h1>
              <h3 class="mt-2 opacity-70 font-normal text-base text-shadow">
                <span v-if="colorType === 'hsl'">{{ h }}, {{ s }}%, {{ l }}%</span>
                <span v-else-if="colorType === 'hex'">{{ hex }}</span>
                <span v-else-if="colorType === 'rgb'">{{ r }}, {{ g }}, {{ b }}</span>
              </h3>
            </div>
          </div>
          <div class="p-1 py-6">
            <!--     Input HSL      -->
            <div class="w-full h-4 rounded-full border border-text-color mb-2" :style="gradientH">
              <input class="appearance-none w-full bg-transparent focus:outline-none w-100 m-0 input-range" type="range" min="0" max="360" v-model="h" @click="updateColor" style="-webkit-appearance: none;"/>
            </div>
            <div class="w-full h-4 rounded-full border border-text-color mb-2" :style="gradientS">
              <input class="appearance-none w-full bg-transparent focus:outline-none w-100 m-0 input-range" type="range" min="0" max="100" v-model="s" @click="updateColor" style="-webkit-appearance: none;"/>
            </div>
            <div class="w-full h-4 rounded-full border border-text-color mb-2" :style="generateHexGradient">
              <input class="appearance-none w-full bg-transparent focus:outline-none w-100 m-0 input-range" type="range" min="0" max="100" v-model="l" @click="updateColor" style="-webkit-appearance: none;"/>
            </div>
            <div class="w-full flex flex-col">
              <!--     Input HSL      -->
              <div class="w-full flex flex-row">
                <input class="flex-1 rounded-md p-1" min="0" max="360" type="number" v-model="h">
                <input class="flex-1 rounded-md p-1 mx-1" min="0" max="100" type="number" v-model="s">
                <input class="flex-1 rounded-md p-1" min="0" max="100" type="number" v-model="l">
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<style scoped>
.input-range[type=range]::-ms-track {
  width: 100%;
  cursor: pointer;
  background: transparent;
  border-color: transparent;
  color: transparent;
}

.input-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  border: 1px solid #ddd;
  height: 20px;
  width: 20px;
  border-radius: 50px;
  background: #ffffff;
  cursor: pointer;
  box-shadow: 0px 1px 2px rgba(0, 0, 0, 0.12);
  margin-top: -4px;
}
</style>