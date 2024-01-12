<template>

  <NonoNav @changeBoardSize="changeBoardSize"/>

  <!-- Main content -->
  <div class="flex justify-center items-center h-full flex-col transition-all">

    <!-- Lives -->
    <NonoLives :lives="lives"/>

    <div v-if="lives > 0" class="nonogram transition-all drop-shadow-lg">
      <div class="nonogram-top w-full flex transition-all">
        <div v-for="(row, rowIndex) in boardlayoutColumn" v-bind:key="row + '-' + rowIndex"
          class="nono-top-cell flex flex-col justify-end items-center pt-2 pb-2 border-solid border border-gray-200 transition-all">
          <!-- Hint Top columns -->
          <NonoHint v-for="(hint, indexHint) in calculateHints(row)" v-bind:key="indexHint + '-' + hint" :hint="hint"/>
        </div>
      </div>
      <div class="nonogram-left h-full transition-all">
        <div v-for="(row, rowIndex) in boardlayoutRow" v-bind:key="row + '-' + rowIndex"
          class="nono-left-cell flex justify-end items-center pr-2 pl-2 border-solid border border-gray-200 transition-all">
          <!-- Hint Left row -->
          <NonoHint v-for="(hint, indexHint) in calculateHints(row)" v-bind:key="indexHint + '-' + hint" :hint="hint"/>
        </div>
      </div>
      <div class="nonogram-board transition-all">
        <div v-for="(row, indexRow) in boardlayoutRow" v-bind:key="row" class="nono-row flex">
          <template v-for="(cellValue, indexColumn) in row" v-bind:key="cellValue">
            <!-- Cell -->
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
  console.table(boardlayoutRow.value.map(row => {
    return row.map(cell => {
      return cell ? cell = '✅': cell = '❌';
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
  localStorage.setItem('sizeBoardWidth', width);
  localStorage.setItem('sizeBoardHeight', height);
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