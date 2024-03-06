<script setup lang="ts">
import tinycolor from 'tinycolor2';

onMounted(() => {

  const addSwatch = document.getElementById('add-swatch') as HTMLElement;
  const modeToggle = document.getElementById('mode-toggle') as HTMLElement;
  const swatches = document.getElementsByClassName('default-swatches')[0] as HTMLElement;
  const colorIndicator = document.getElementById('color-indicator') as HTMLElement;
  const userSwatches = document.getElementById('user-swatches') as HTMLElement;

  const spectrumCanvas = document.getElementById('spectrum-canvas') as HTMLCanvasElement;
  const spectrumCtx = spectrumCanvas.getContext('2d')!;
  const spectrumCursor = document.getElementById('spectrum-cursor') as HTMLElement;
  let spectrumRect = spectrumCanvas.getBoundingClientRect();

  const hueCanvas = document.getElementById('hue-canvas') as HTMLCanvasElement;
  const hueCtx = hueCanvas.getContext('2d')!;
  const hueCursor = document.getElementById('hue-cursor') as HTMLElement;
  let hueRect = hueCanvas.getBoundingClientRect();

  let currentColor = '';
  let hue = 0;
  let saturation = 1;
  let lightness = 0.5;

  const rgbFields = document.getElementById('rgb-fields') as HTMLElement;
  const hexField = document.getElementById('hex-field') as HTMLElement;

  const red = document.getElementById('red') as HTMLInputElement;
  const blue = document.getElementById('blue') as HTMLInputElement;
  const green = document.getElementById('green') as HTMLInputElement;
  const hex = document.getElementById('hex') as HTMLInputElement;

  class ColorPicker {
    defaultSwatches: string[];

    constructor() {
      this.defaultSwatches = [
        '#FFFFFF',
        '#FFFB0D',
        '#0532FF',
        '#FF9300',
        '#00F91A',
        '#FF2700',
        '#000000',
        '#686868',
        '#EE5464',
        '#D27AEE',
        '#5BA8C4',
        '#E64AA9'
      ];
      this.addDefaultSwatches();
      this.createShadeSpectrum();
      this.createHueSpectrum();
    }


    addDefaultSwatches() {
      for (let i = 0; i < this.defaultSwatches.length; ++i) {
        this.createSwatch(swatches, this.defaultSwatches[i]);
      }
    }

    createSwatch(target: HTMLElement, color: string) {
      const swatch = document.createElement('button');
      swatch.classList.add('swatch');
      swatch.setAttribute('title', color);
      swatch.style.backgroundColor = color;
      swatch.addEventListener('click', function () {
        const color = tinycolor(this.style.backgroundColor);
        colorToPos(color);
        setColorValues(color);
      });
      target.appendChild(swatch);
      this.refreshElementRects();
    }

    refreshElementRects() {
      spectrumRect = spectrumCanvas.getBoundingClientRect();
      hueRect = hueCanvas.getBoundingClientRect();
    }

    createShadeSpectrum(color?: string) {
      const canvas = spectrumCanvas;
      const ctx = spectrumCtx;
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      if (!color) color = '#f00';
      ctx.fillStyle = color;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      const whiteGradient = ctx.createLinearGradient(0, 0, canvas.width, 0);
      whiteGradient.addColorStop(0, '#fff');
      whiteGradient.addColorStop(1, 'transparent');
      ctx.fillStyle = whiteGradient;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      const blackGradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
      blackGradient.addColorStop(0, 'transparent');
      blackGradient.addColorStop(1, '#000');
      ctx.fillStyle = blackGradient;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      canvas.addEventListener('mousedown', function (e) {
        startGetSpectrumColor(e);
      });
    }

    createHueSpectrum() {
      const canvas = hueCanvas;
      const ctx = hueCtx;
      const hueGradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
      hueGradient.addColorStop(0.00, "hsl(0, 100%, 50%)");
      hueGradient.addColorStop(0.17, "hsl(298.8, 100%, 50%)");
      hueGradient.addColorStop(0.33, "hsl(241.2, 100%, 50%)");
      hueGradient.addColorStop(0.50, "hsl(180, 100%, 50%)");
      hueGradient.addColorStop(0.67, "hsl(118.8, 100%, 50%)");
      hueGradient.addColorStop(0.83, "hsl(61.2, 100%, 50%)");
      hueGradient.addColorStop(1.00, "hsl(360, 100%, 50%)");
      ctx.fillStyle = hueGradient;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      canvas.addEventListener('mousedown', function (e) {
        startGetHueColor(e);
      });
    }
  }

  const colorPicker = new ColorPicker();

  function colorToHue(color: string) {
    const colorValue = tinycolor(color);
    const hueString = tinycolor(`hsl ${colorValue.toHsl().h} 1 .5`).toHslString();
    return hueString;
  }

  function colorToPos(color: string) {
    const colorValue = tinycolor(color);
    const hsl = colorValue.toHsl();
    hue = hsl.h;
    const hsv = colorValue.toHsv();
    const x = spectrumRect.width * hsv.s;
    const y = spectrumRect.height * (1 - hsv.v);
    let hueY = hueRect.height - ((hue / 360) * (hueRect.height - 1));
    const constHueY = hueY;
    hueY = (hueY / hueRect.height) * 100;
    hueY = (hueY / 100) * 269;
    hueY = 269 - hueY;
    updateSpectrumCursor(x, y);
    hueY = hueY;
    updateHueCursor(hueY);
    colorPicker.createShadeSpectrum(colorToHue(color));
    hueCursor.style.backgroundColor = `hsl(${colorValue.toHsl().h},100%, 50%)`;
    colorIndicator.style.backgroundColor = color;
    document.body.style.backgroundColor = color;
    spectrumCursor.style.backgroundColor = color;

  }

  function setColorValues(color: string) {
    const colorValue = tinycolor(color);
    const rgbValues = colorValue.toRgb();
    const hexValue = colorValue.toHex();

    red.value = String(rgbValues.r);
    green.value = String(rgbValues.g);
    blue.value = String(rgbValues.b);
    hex.value = hexValue;
  }

  function setCurrentColor(color: string) {
    const colorValue = tinycolor(color);
    currentColor = color;
    colorIndicator.style.backgroundColor = color;
    document.body.style.backgroundColor = color;
    spectrumCursor.style.backgroundColor = color;
    hueCursor.style.backgroundColor = `hsl(${colorValue.toHsl().h},100%, 50%)`;
  }

  function updateHueCursor(y: number) {
    console.log("cursor y", y);
    hueCursor.style.top = `${y}px`;
  }

  function updateSpectrumCursor(x: number, y: number) {
    spectrumCursor.style.left = `${x}px`;
    spectrumCursor.style.top = `${y}px`;
  }

  const startGetSpectrumColor = function (e: MouseEvent) {
    getSpectrumColor(e);
    spectrumCursor.classList.add('dragging');
    window.addEventListener('mousemove', getSpectrumColor);
    window.addEventListener('mouseup', endGetSpectrumColor);
  };

  function getSpectrumColor(e: MouseEvent) {
    e.preventDefault();

    let x = e.pageX - spectrumRect.left;
    let y = e.pageY - spectrumRect.top;

    // Adjust coordinates based on window scroll and canvas position
    x -= window.pageXOffset;
    y -= window.pageYOffset;

    if (x > spectrumRect.width) {
      x = spectrumRect.width
    }
    if (x < 0) {
      x = 0
    }
    if (y > spectrumRect.height) {
      y = spectrumRect.height
    }
    if (y < 0) {
      y = .1
    }

    const xRatio = x / spectrumRect.width * 100;
    const yRatio = y / spectrumRect.height * 100;
    const hsvValue = 1 - (yRatio / 100);
    const hsvSaturation = xRatio / 100;
    lightness = (hsvValue / 2) * (2 - hsvSaturation);
    saturation = (hsvValue * hsvSaturation) / (1 - Math.abs(2 * lightness - 1));
    const color = tinycolor(`hsl ${hue} ${saturation} ${lightness}`);
    setCurrentColor(color);
    setColorValues(color);
    updateSpectrumCursor(x, y);
  }

  function endGetSpectrumColor(e: MouseEvent) {
    spectrumCursor.classList.remove('dragging');
    window.removeEventListener('mousemove', getSpectrumColor);
  }

  const startGetHueColor = function (e: MouseEvent) {
    getHueColor(e);
    hueCursor.classList.add('dragging');
    window.addEventListener('mousemove', getHueColor);
    window.addEventListener('mouseup', endGetHueColor);
  };

  function getHueColor(e) {
    e.preventDefault();
    const x = e.pageX - hueRect.left; // Utilisez la coordonnée X au lieu de la coordonnée Y
    if (x > hueRect.width) {
      x = hueRect.width
    }
    ;
    if (x < 0) {
      x = 0
    }
    ;
    const percent = x / hueRect.width;
    hue = 360 * percent;
    const hueColor = tinycolor(`hsl ${hue} 100% 50%`).toHslString(); // Ajustez le gradient de couleur en conséquence
    const color = tinycolor(`hsl ${hue} ${saturation} ${lightness}`).toHslString();
    colorPicker.createShadeSpectrum(hueColor);
    updateHueCursor(x); // Mettez à jour la position du curseur en utilisant la coordonnée X
    setCurrentColor(color);
    setColorValues(color);
  }

  function endGetHueColor(e: MouseEvent) {
    hueCursor.classList.remove('dragging');
    window.removeEventListener('mousemove', getHueColor);
  }

  red.addEventListener('change', function () {
    const color = tinycolor(`rgb ${red.value} ${green.value} ${blue.value}`);
    colorToPos(color);
  });

  green.addEventListener('change', function () {
    const color = tinycolor(`rgb ${red.value} ${green.value} ${blue.value}`);
    colorToPos(color);
  });

  blue.addEventListener('change', function () {
    const color = tinycolor(`rgb ${red.value} ${green.value} ${blue.value}`);
    colorToPos(color);
  });

  hex.addEventListener('input', function () {
    const color = tinycolor(hex.value);
    if (color.isValid()) {
      colorToPos(color);
    }
  });


  addSwatch.addEventListener('click', function () {
    colorPicker.createSwatch(userSwatches, currentColor);
  });

  modeToggle.addEventListener('click', function () {
    rgbFields.classList.toggle('active');
    hexField.classList.toggle('active');
  });

  window.addEventListener('resize', function () {
    this.refreshElementRects();
  });

  new ColorPicker();
});

</script>

<template>

  <div
      class="bg-color-text/70 max-w-md rounded-lg border-2 border-gray-700 shadow-lg absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">

    <div class="relative m-0 mb-12 flex flex-col">
      <div class="relative w-full h-[200px]">
        <button id="spectrum-cursor"
                class="color-cursor border border-white rounded-full bg-gray-300 absolute pointer-events-none transition-all duration-200 ease-in-out w-7 h-7 z-10 ml-15 -mt-15 top-0 top left-0"></button>
        <canvas id="spectrum-canvas"
                class="absolute w-full h-full top-0 left-0 right-0 bottom-0 bg-gray-300 rounded-t-lg"></canvas>
      </div>
      <div class="absolute top-5 -bottom-56 right-[3.7rem] w-7 -rotate-90">
        <button id="hue-cursor"
                class="color-cursor z-10 border border-white rounded-full bg-gray-300 absolute pointer-events-none transition-all duration-200 ease-in-out top-0 left-1/2 h-5 w-full -mt-3 -ml-3.5 cursor-pointer"></button>
        <canvas id="hue-canvas"
                class="absolute top-0 left-0 right-0 bottom-0 w-full h-4/6 bg-gray-300 rounded-lg rotate-180"></canvas>
      </div>
    </div>
    <div class="relative m-0 mb-10 ml-0 px-2 flex flex-row items-center justify-center content-center">
      <div id="rgb-fields" class="field-group value-fields rgb-fields active">
        <div class="field-group">
          <label for="" class="field-label">R:</label>
          <input type="number" max="255" min="0" id="red" class="field-input rgb-input rounded-lg">
        </div>
        <div class="field-group">
          <label for="" class="field-label">G:</label>
          <input type="number" max="255" min="0" id="green" class="field-input rgb-input rounded-lg">
        </div>
        <div class="field-group">
          <label for="" class="field-label">B:</label>
          <input type="number" max="255" min="0" id="blue" class="field-input rgb-input rounded-lg">
        </div>
      </div>
      <div id="hex-field" class="field-group value-fields hex-field">
        <label for="" class="field-label">Hex:</label>
        <input type="text" id="hex" class="field-input rounded-lg">
      </div>
      <button id="mode-toggle"
              class="button bg-gray-700 border-0 rounded-md text-gray-400 text-base font-normal shadow-md outline-none cursor-pointer py-1 px-2 -top-1 -right-10 w-68 h-[2.5rem]">
        Mode
      </button>
    </div>
    <div class="relative m-0 mr-10 mb-10 ml-10 content-normal display-table clear-both">
      <h2 class="bg-gray-900 p-2 m-0 -mx-4 my-4 text-center">User Colors</h2>
      <div id="user-swatches" class="swatches custom-swatches"></div>
      <button id="add-swatch"
              class="button bg-gray-700 border-0 rounded-md text-gray-400 text-base font-normal shadow-md outline-none cursor-pointer py-1 px-2 block px-6 py-6 w-52 mx-auto mt-10">
        <span id="color-indicator" class="inline-block align-middle mr-5 w-5 h-5 rounded-4 bg-gray-400"></span>
        <span>Add to Swatches</span>
      </button>
    </div>
    <div class="relative mr-10 mt-10 mb-6 ml-10">
      <div class="swatch default-swatches max-w-md"></div>
      <button
          class="button bg-gray-700 border-0 rounded-md text-gray-400 text-base font-normal shadow-md outline-none cursor-pointer py-1 px-2 absolute top-0 right-0 w-16 h-16 block">
        Get Color
      </button>
    </div>
  </div>

</template>

<style scoped>
body {
  background: #1f232a;
  font-family: "Proxima Nova", sans-serif;
  color: #8b949a;
  letter-spacing: 0.05em;
  transition: background 0.5s ease;
}

.swatch {
  display: inline-block;
  width: 32px;
  height: 32px;
  background: #ccc;
  border-radius: 4px;
  margin-left: 4px;
  margin-bottom: 4px;
  box-sizing: border-box;
  border: 2px solid #364347;
  cursor: pointer;
}

.default-swatches {
}

.default-swatches .swatch:nth-child(6n + 1) {
  margin-left: 0;
}

.color-cursor.dragging {
  transition: none;
}

.button {
}

button:active {
  background: #262c31;
}

.value-fields {
  display: none;
  align-items: center;
}

.value-fields.active {
  display: flex;
}

.value-fields .field-label {
  margin-right: 6px;
}

.value-fields .field-input {
  background: #15191c;
  border: 1px solid #364347;
  box-sizing: border-box;
  line-height: 38px;
  padding: 0 4px;
  text-align: center;
  color: #8b949a;
  font-size: 1rem;
  display: block;
}

.rgb-fields .field-group {
  display: flex;
  align-items: center;
}

.rgb-fields .field-input {
  width: 42px;
  margin-right: 10px;
}

.hex-field .field-input {
  width: 170px;
}

input::-webkit-outer-spin-button, input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
</style>


