<template>
  <!-- navigation -->
  <div class="card flex justify-content-center">
    <Sidebar v-model:visible="visible" header="Settings">
      <div class="relative mt-2 rounded-md shadow-sm">
        <div>
          <h2 class="text-center"> Size Board</h2>
          <label for="inputBoardWidth">Width</label>
          <input id="inputBoardWidth" max="20" min="3" type="number" v-model="inputBoardWidth"
            class="inline-block w-full rounded-md border-0 py-1 pl-2 pr-2 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-slate-600 sm:text-sm sm:leading-6"
            placeholder="0" value="10">
          <br>
          <label for="inputBoardHeight">Height</label>
          <input id="inputBoardHeight" max="20" min="3" type="number" v-model="inputBoardHeight"
            class="inline-block w-full rounded-md border-0 py-1 pl-2 pr-2 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-slate-600 sm:text-sm sm:leading-6"
            placeholder="0" value="10">
          <br>
          <div class="flex justify-between">
            <button @click="resetBoard()"
              class="bg-blue-500 hover:bg-blue-950 transition mt-2 text-white font-bold py-2 px-4 rounded">
              Update
            </button>
            <button @click="resetBoard(getRandomInt(18) + 3, getRandomInt(18) + 3)"
              class="bg-white-500 hover:bg-blue-500 transition mt-2 text-black hover:text-white font-bold py-2 px-4 rounded">
              Random
            </button>
          </div>
        </div>
        <span class="block my-6 border border-gray-200"></span>
        <div>
          <h2 class="text-center"> Size Squares </h2>
          <div class="flex justify-between">
            <button @click="saveSizeSquares('20')"
              class="bg-blue-500 hover:bg-blue-950 transition mt-2 text-white font-bold py-2 px-4 rounded">
              Small
            </button>
            <button @click="saveSizeSquares('40')"
              class="bg-blue-500 hover:bg-blue-950 transition mt-2 text-white font-bold py-2 px-4 rounded">
              Medium
            </button>
            <button @click="saveSizeSquares('60')"
              class="bg-blue-500 hover:bg-blue-950 transition mt-2 text-white font-bold py-2 px-4 rounded">
              Large
            </button>
          </div>
        </div>
      </div>
    </Sidebar>
    <button @click="resetBoard()"
      class="bg-red-700 hover:bg-red-950 transition ml-2 text-white font-bold py-2 px-4 rounded absolute top-4 left-4">
      Reset
    </button>
    <button @click="visible = true"
      class="bg-blue-500 hover:bg-blue-950 transition ml-2 text-white font-bold py-2 px-4 rounded absolute top-16 left-4">
      Settings
    </button>
  </div>
</template>

<script setup>

import { ref } from 'vue'
import Sidebar from 'primevue/sidebar';
import getRandomInt from "../helpers/randomInt"

const emit = defineEmits(['changeBoardSize'])

let visible = ref(false);
let inputBoardWidth = ref(10);
let inputBoardHeight = ref(10);
let sizeBoardRows = 0;
let sizeBoardColums = 0;
let root = document.documentElement;

if (localStorage.getItem('sizeBoardWidth')) {
  sizeBoardColums = localStorage.getItem('sizeBoardWidth');
  inputBoardWidth.value  = localStorage.getItem('sizeBoardWidth');
}
if (localStorage.getItem('sizeBoardHeight')) {
  sizeBoardRows = localStorage.getItem('sizeBoardHeight');
  inputBoardHeight.value  = localStorage.getItem('sizeBoardHeight');
}

resetBoard(inputBoardWidth.value, inputBoardHeight.value)

function resetBoard(width, height) {
  if (width) inputBoardWidth.value = width;
  if (height) inputBoardHeight.value = height;

  let boardlayoutRow = []
  let boardlayoutColumn = []
  let lives = 3

  // board width cant be below 3 and above 20
  if (inputBoardWidth.value <= 20) sizeBoardColums = inputBoardWidth.value;
  else sizeBoardColums = 20;
  if (inputBoardWidth.value >= 3) sizeBoardColums = inputBoardWidth.value;
  else sizeBoardColums = 3;

  // board height cant be below 3 and above 20
  if (inputBoardHeight.value <= 20) sizeBoardRows = inputBoardHeight.value;
  else sizeBoardRows = 20;
  if (inputBoardHeight.value >= 3) sizeBoardRows = inputBoardHeight.value;
  else sizeBoardRows = 3;

  // reset width input if below 3 or above 20
  if (inputBoardWidth.value > 20) inputBoardWidth.value = 20
  if (inputBoardWidth.value < 3) inputBoardWidth.value = 3

  // reset height input if below 3 or above 20
  if (inputBoardHeight.value > 20) inputBoardHeight.value = 20
  if (inputBoardHeight.value < 3) inputBoardHeight.value = 3

  emit('changeBoardSize', {
    sizeBoardRows,
    sizeBoardColums,
    boardlayoutRow,
    boardlayoutColumn,
    lives
  })
}

function saveSizeSquares(size) {
  root.style.setProperty('--size-cell', size);
  localStorage.setItem('sizeSquare', size);
}
</script>

<style lang="scss">

</style>