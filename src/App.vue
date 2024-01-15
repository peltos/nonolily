<template>
  <NonoNav @changeBoardSize="changeBoardSize" />

  <!-- Main content -->
  <div class="flex justify-center items-center h-full flex-col transition-all">

    <!-- Lives -->
    <NonoLives :lives="lives" />

    <div v-if="lives > 0" class="nonogram transition-all drop-shadow-lg">
      <div class="nonogram-top w-full flex transition-all">
        <div v-for="(row, rowIndex) in boardlayoutColumn" v-bind:key="row + '-' + rowIndex"
          class="nono-top-cell flex flex-col justify-end items-center pt-2 pb-2 border-solid border border-gray-200 transition-all">
          <!-- Hint Top columns -->
          <NonoHint v-for="(hint, indexHint) in calculateHints(row)" v-bind:key="indexHint + '-' + hint" :hint="hint" />
        </div>
      </div>
      <div class="nonogram-left h-full transition-all">
        <div v-for="(row, rowIndex) in boardlayoutRow" v-bind:key="row + '-' + rowIndex"
          class="nono-left-cell flex justify-end items-center pr-2 pl-2 border-solid border border-gray-200 transition-all">
          <!-- Hint Left row -->
          <NonoHint v-for="(hint, indexHint) in calculateHints(row)" v-bind:key="indexHint + '-' + hint" :hint="hint" />
        </div>
      </div>
      <div class="nonogram-board transition-all">
        <div v-for="(row, indexRow) in boardlayoutRow" v-bind:key="row" class="nono-row flex">
          <template v-for="(cellValue, indexColumn) in row" v-bind:key="cellValue">
            <!-- Cell -->
            <NonoCell @clickedFilledCell="reduceCellCount(indexRow, indexColumn)" @mistakeMade="lives = lives - 1"
              @greyOut="greyOut = 'grey'" :cellValue="cellValue" :cellRow="indexRow" :cellColumn="indexColumn"
              :full="full"></NonoCell>
          </template>
        </div>
      </div>
    </div>
    <div v-if="lives > 0" class="nono-tip flex">

      <div>
        <div class="example-cell fill"></div>

        <svg fill="#000000" version="1.1" viewBox="0 0 356.57 356.57" xml:space="preserve"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="m181.56 0c-60.801 0-122.35 30.525-122.35 88.873v148.63c0 65.658 53.412 119.07 119.07 119.07 65.658 0 119.07-53.413 119.07-119.07v-148.63c0-61.064-60.02-88.873-115.79-88.873zm93.382 237.5c0 53.303-43.362 96.657-96.659 96.657-53.299 0-96.657-43.354-96.657-96.657v-69.513c20.014 6.055 57.685 15.215 102.22 15.215 28.515 0 59.831-3.809 91.095-14.567v68.865zm0-92.706c-81.683 31.233-168.35 7.716-193.32-0.364v-55.557c0-43.168 51.489-66.46 99.934-66.46 46.481 0 93.382 20.547 93.382 66.46v55.921zm-84.052-96.405v81.248c0 6.187-5.023 11.208-11.206 11.208-6.185 0-11.207-5.021-11.207-11.208v-81.248c0-6.186 5.021-11.207 11.207-11.207 6.182 0 11.206 5.021 11.206 11.207zm-35.955-8.321v103.66c-15.879 2.802-62.566-10.271-62.566-10.271-12.139-92.455 62.566-93.391 62.566-93.391z" />
        </svg>
      </div>
      <div>
        <div class="example-cell cross">
          <svg xmlns="http://www.w3.org/2000/svg" height="16" width="12" viewBox="0 0 384 512">
            <path opacity="1"
              d="M342.6 150.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L192 210.7 86.6 105.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L146.7 256 41.4 361.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0L192 301.3 297.4 406.6c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L237.3 256 342.6 150.6z" />
          </svg>
        </div>

        <svg fill="#000000" version="1.1" viewBox="0 0 356.57 356.57" xml:space="preserve"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="m181.56 0c-60.801 0-122.35 30.525-122.35 88.873v148.63c0 65.658 53.412 119.07 119.07 119.07 65.658 0 119.07-53.413 119.07-119.07v-148.63c0-61.064-60.02-88.873-115.79-88.873zm93.382 237.5c0 53.303-43.362 96.657-96.659 96.657-53.299 0-96.657-43.354-96.657-96.657v-69.513c20.014 6.055 57.685 15.215 102.22 15.215 28.515 0 59.831-3.809 91.095-14.567v68.865zm0-92.706c-81.683 31.233-168.35 7.716-193.32-0.364v-55.557c0-43.168 51.489-66.46 99.934-66.46 46.481 0 93.382 20.547 93.382 66.46v55.921zm-84.052-96.405v81.248c0 6.187-5.023 11.208-11.206 11.208-6.185 0-11.207-5.021-11.207-11.208v-81.248c0-6.186 5.021-11.207 11.207-11.207 6.182 0 11.206 5.021 11.206 11.207zm73.379 81.989s-46.687 13.072-62.566 10.271v-103.66s74.706 0.935 62.566 93.39z" />
        </svg>
      </div>

    </div>
    <h2 class="nono-error" v-if="lives <= 0">Game over...</h2>
  </div>

  <span class="absolute bottom-1 left-1 text-slate-500">Version {{ version }}</span>
</template>

<script setup>
import { ref } from 'vue'
import NonoCell from "./components/NonoCell.vue"
import NonoNav from "./components/NonoNav.vue"
import NonoHint from "./components/NonoHint.vue"
import NonoLives from "./components/NonoLives.vue"
import getRandomInt from "./helpers/randomInt"

let sizeBoardRows = ref(10);
let sizeBoardColums = ref(10);
let boardlayoutRow = ref([]);
let boardlayoutColumn = ref([]);
let totalCellsRow = ref([]);
let totalCellsColumn = ref([]);
let full = ref([]);
let mouseDown = ref(0);
let root = document.documentElement;
let lives = ref(3);
let greyOut = ref('');
const version = import.meta.env.VITE_APP_VERSION

function boardInit() {

  if (localStorage.getItem('nonoSizeSquare')) root.style.setProperty('--size-cell', localStorage.getItem('nonoSizeSquare'));

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
  console.table(boardlayoutRow.value.map(row => {
    return row.map(cell => {
      return cell ? cell = '✅' : cell = '❌';
    });
  }));
}

function changeBoardSize(settings) {
  boardlayoutColumn.value = settings.boardlayoutColumn;
  boardlayoutRow.value = settings.boardlayoutRow;
  lives.value = settings.lives;
  sizeBoardColums.value = settings.sizeBoardColums;
  sizeBoardRows.value = settings.sizeBoardRows;

  boardInit();
}

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

function saveSizeBoard(width, height) {
  localStorage.setItem('nonoSizeBoardWidth', width);
  localStorage.setItem('nonoSizeBoardHeight', height);
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

.nono-tip {
  gap: 16px;
  margin: 32px 8px 8px;

  .example-cell {
    background-color: white;
    transition: 200ms;
    width: calc(var(--size-cell) * 1px);
    height: calc(var(--size-cell) * 1px);
    margin-bottom: 8px;

    svg {
      width: calc(var(--size-cell) * 1px);
      height: calc(var(--size-cell) * 1px);
    }

    &.fill {
      background-color: black;
    }

    &.cross {
      svg {
        @apply fill-gray-400;
      }
    }
  }
}

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