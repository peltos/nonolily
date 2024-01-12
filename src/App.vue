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

  <!-- Main content -->
  <div class="flex justify-center items-center h-full flex-col transition-all">
    <div class="nono-life-wrapper flex flex-wrap justify-center my-4 fill-red-700 drop-shadow-lg">
      <svg class="nono-life" v-for="live in lives" :key="live" xmlns="http://www.w3.org/2000/svg" height="16" width="16"
        viewBox="0 0 512 512">
        <path opacity="1"
          d="M47.6 300.4L228.3 469.1c7.5 7 17.4 10.9 27.7 10.9s20.2-3.9 27.7-10.9L464.4 300.4c30.4-28.3 47.6-68 47.6-109.5v-5.8c0-69.9-50.5-129.5-119.4-141C347 36.5 300.6 51.4 268 84L256 96 244 84c-32.6-32.6-79-47.5-124.6-39.9C50.5 55.6 0 115.2 0 185.1v5.8c0 41.5 17.2 81.2 47.6 109.5z" />
      </svg>
    </div>
    <div v-if="lives > 0" class="nonogram transition-all drop-shadow-lg">
      <div class="nonogram-top w-full flex transition-all">
        <div v-for="(row, rowIndex) in boardlayoutColumn" v-bind:key="row + '-' + rowIndex"
          class="nono-top-cell flex flex-col justify-end items-center pt-2 pb-2 border-solid border border-gray-200 transition-all">
          <span :class="'nono-hint flex justify-center items-center transition-all '+greyOut"
            v-for="(hint, index) in calculateHints(row)" v-bind:key="index + '-' + hint"> {{ hint }} </span>
        </div>
      </div>
      <div class="nonogram-left h-full transition-all">
        <div v-for="(row, rowIndex) in boardlayoutRow" v-bind:key="row + '-' + rowIndex"
          class="nono-left-cell flex justify-end items-center pr-2 pl-2 border-solid border border-gray-200 transition-all">
          <span :class="'nono-hint flex justify-center items-center transition-all '+greyOut"
            v-for="(hint, index) in calculateHints(row)" v-bind:key="index + '-' + hint"> {{ hint }} </span>
        </div>
      </div>
      <div class="nonogram-board transition-all">
        <div v-for="(row, indexRow) in boardlayoutRow" v-bind:key="row" class="nono-row flex">
          <template v-for="(cellValue, indexColumn) in row" v-bind:key="cellValue">
            <NonoCell @clickedFilledCell="reduceCellCount(indexRow, indexColumn)" @mistakeMade="lives = lives - 1" @greyOut="greyOut = 'grey'"
              :cellValue="cellValue" :cellRow="indexRow" :cellColumn="indexColumn" :full="full"></NonoCell>
          </template>
        </div>
      </div>
    </div>
    <strong v-if="lives > 0" class="nono-tip">Left click = fill<br> Right click = cross </strong>
    <h2 class="nono-error" v-if="lives <= 0">Game over...</h2>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import NonoCell from "./components/NonoCell.vue"
import Sidebar from 'primevue/sidebar';

let sizeBoardRows = ref(10);
let sizeBoardColums = ref(10);
let inputBoardWidth = ref(10);
let inputBoardHeight = ref(10);
let boardlayoutRow = ref([]);
let boardlayoutColumn = ref([]);
let totalCellsRow = ref([]);
let totalCellsColumn = ref([]);
let full = ref([]);
let mouseDown = ref(0);
let visible = ref(false);
let root = document.documentElement;
let lives = ref(3);
let greyOut = ref('');

function boardInit() {

  if (localStorage.getItem('sizeSquare')) root.style.setProperty('--size-cell', localStorage.getItem('sizeSquare'));

  for (let i = 0; i < sizeBoardRows.value; i++) {
    for (let j = 0; j < sizeBoardColums.value; j++) {
      if (j === 0) {
        boardlayoutRow.value[i] = [];
        totalCellsRow.value[i] = 0;
      }
      boardlayoutRow.value[i][j] = getRandomInt(2);
      totalCellsRow.value[i] = totalCellsRow.value[i] + boardlayoutRow.value[i][j];
    }
  }
  for (let i = 0; i < sizeBoardColums.value; i++) {
    for (let j = 0; j < sizeBoardRows.value; j++) {
      if (j === 0) {
        boardlayoutColumn.value[i] = [];
        totalCellsColumn.value[i] = 0;
      }
      boardlayoutColumn.value[i][j] = boardlayoutRow.value[j][i];
      totalCellsColumn.value[i] = totalCellsColumn.value[i] + boardlayoutColumn.value[i][j];
    }
  }

  saveSizeBoard(sizeBoardColums.value.toString(), sizeBoardRows.value.toString());

  totalCellsRow.value.forEach((row, indexRow) => {
    requestAnimationFrame(() => {
      if (row === 0) full.value = [indexRow, NaN]
    })
  });
  totalCellsColumn.value.forEach((column, indexColumn) => {
    requestAnimationFrame(() => {
      if (column === 0) full.value = [NaN, indexColumn]
    })
  });

  // TODO: remove, is for debugging
  console.table(boardlayoutRow.value);
  console.log(totalCellsRow.value);
  console.log(totalCellsColumn.value);
}

if (localStorage.getItem('sizeBoardWidth')) {
  sizeBoardColums.value = localStorage.getItem('sizeBoardWidth');
  inputBoardWidth.value = localStorage.getItem('sizeBoardWidth');
}
if (localStorage.getItem('sizeBoardHeight')) {
  sizeBoardRows.value = localStorage.getItem('sizeBoardHeight');
  inputBoardHeight.value = localStorage.getItem('sizeBoardHeight');
}
boardInit()

function calculateHints(row) {
  let counter = 0
  let newRowArray = []
  row.forEach(element => {
    if (element === 0) {
      if (counter != 0) newRowArray.push(counter); counter = 0;
      return;
    }
    if (element === 1) counter += element;
  });
  if (counter != 0) newRowArray.push(counter); counter = 0;

  // return 0 if there are no cells in that row or column
  return newRowArray.length !== 0 ? newRowArray : [0];
}

function reduceCellCount(cellRow, cellColumn) {
  totalCellsRow.value[cellRow] = totalCellsRow.value[cellRow] - 1;
  totalCellsColumn.value[cellColumn] = totalCellsColumn.value[cellColumn] - 1;

  if (totalCellsRow.value[cellRow] === 0 && totalCellsColumn.value[cellColumn] === 0) {
    return full.value = [cellRow, cellColumn]
  }
  if (totalCellsRow.value[cellRow] === 0) return full.value = [cellRow, NaN];
  if (totalCellsColumn.value[cellColumn] === 0) return full.value = [NaN, cellColumn];
}

function resetBoard(width, height) {
  if (width) inputBoardWidth.value = width;
  if (height) inputBoardHeight.value = height;

  boardlayoutRow.value = []
  boardlayoutColumn.value = []
  lives.value = 3

  if (inputBoardWidth.value <= 20) sizeBoardColums.value = inputBoardWidth.value;
  else sizeBoardColums.value = 20;
  if (inputBoardWidth.value >= 3) sizeBoardColums.value = inputBoardWidth.value;
  else sizeBoardColums.value = 3;

  if (inputBoardHeight.value <= 20) sizeBoardRows.value = inputBoardHeight.value;
  else sizeBoardRows.value = 20;
  if (inputBoardHeight.value >= 3) sizeBoardRows.value = inputBoardHeight.value;
  else sizeBoardRows.value = 3;

  if (inputBoardWidth.value > 20) inputBoardWidth.value = 20
  if (inputBoardWidth.value < 3) inputBoardWidth.value = 3

  if (inputBoardHeight.value > 20) inputBoardHeight.value = 20
  if (inputBoardHeight.value < 3) inputBoardHeight.value = 3
  boardInit();
}

function saveSizeBoard(width, height) {
  localStorage.setItem('sizeBoardWidth', width);
  localStorage.setItem('sizeBoardHeight', height);
}

function saveSizeSquares(size) {
  root.style.setProperty('--size-cell', size);
  localStorage.setItem('sizeSquare', size);
}

function getRandomInt(max) {
  return Math.floor(Math.random() * max);
}

document.body.onmousedown = () => mouseDown.value = 1;
document.body.onmouseup = () => mouseDown.value = 0;
</script>

<style lang="scss">
*,
*:before,
*:after {
  box-sizing: border-box;
}

:root {
  --size-cell: 40;
}

h2 {
  font-weight: 900;
  font-size: 16px;
}

.nonogram {
  display: grid;
  grid-template-columns: auto auto;
  grid-template-rows: auto auto;

  .nonogram-top {
    grid-row: 1 / 2;
    grid-column: 2 / 3;
    background: white;
    @apply border;
    @apply border-gray-500;

    .nono-top-cell {
      width: calc(var(--size-cell) * 1px);
    }
  }

  .nonogram-left {
    grid-row: 2 / 3;
    grid-column: 1 / 2;
    background: white;
    @apply border;
    @apply border-gray-500;

    .nono-left-cell {
      height: calc(var(--size-cell) * 1px);
    }
  }

  .nonogram-board {
    grid-row: 2 / 3;
    grid-column: 2 / 3;
    @apply border;
    @apply border-gray-500;

    .nono-row {
      &:nth-child(5n + 6) {
        .nono-cell {
          @apply border-t-gray-500;
        }
      }
    }
  }

  .nono-hint {
    font-size: calc((var(--size-cell) * 1px) / 2);
    line-height: calc((var(--size-cell) * 1px) / 1.5);
    width: calc((var(--size-cell) * 1px) / 1.5);
    height: calc((var(--size-cell) * 1px) / 1.5);

    &.grey {
      // @apply text-slate-300
    }
  }
}

.nono-tip,
.nono-error {
  font-size: calc((var(--size-cell) * 1px) / 2);
  line-height: calc((var(--size-cell) * 1px) / 1.5);
  margin: calc((var(--size-cell) * 1px) / 2) 0;
}

.nono-life-wrapper {
  gap: calc((var(--size-cell) * 1px) / 2);
  margin: calc((var(--size-cell) * 1px) / 2) 0;
  max-width: calc((var(--size-cell) * 1px) * 8);
}

.nono-life {
  width: calc((var(--size-cell) * 1px) / 1.5);
  height: calc((var(--size-cell) * 1px) / 1.5);
}
</style>