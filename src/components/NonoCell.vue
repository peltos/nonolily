<template>
  <div :class="'cell-wrapper relative ' + cellClass" oncontextmenu="return false;">

    <!-- cell square -->
    <div ref="cellElement"
      :class="'nono-cell block border-solid border border-gray-200 ' + cellClass"></div>

    <!-- add xmark fontawesome svg -->
    <svg v-if="cellClass === 'cross' || cellClass === 'wrong-cross'" class="cell-cross" xmlns="http://www.w3.org/2000/svg"
      height="16" width="12" viewBox="0 0 384 512">
      <path opacity="1"
        d="M342.6 150.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L192 210.7 86.6 105.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L146.7 256 41.4 361.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0L192 301.3 297.4 406.6c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L237.3 256 342.6 150.6z" />
    </svg>

    <!-- add custom corner svg -->
    <svg v-if="cellClass === 'wrong-fill'" class="cell-corner" xmlns="http://www.w3.org/2000/svg" height="4" width="4"
      viewBox="0 0 4 4">
      <polygon points="0 4 4 0 0 0" />
    </svg>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps({
  cellValue: {
    type: Number,
    default: 0
  },
  cellRow: {
    type: Number,
    default: 0
  },
  cellColumn: {
    type: Number,
    default: 0
  },
  full: {
    type: Array,
    default() {
      return []
    }
  }
});
watch(
  () => props.full,
  () => {
    if(clicked.value) return;
    console.log('updated');
    const fullRow = props.full[0];
    const fullColumn = props.full[1];

    if((!isNaN(fullRow) && fullRow === props.cellRow) || (!isNaN(fullColumn) && fullColumn === props.cellColumn)) {
      if(props.cellValue === 0) cellClass.value = 'cross';
      clicked.value = true;
      emit('greyOut');
    }
  }
)

const emit = defineEmits(['clickedFilledCell', 'mistakeMade', 'greyOut'])

const cellElement = ref()
let cellClass = ref('');
let clicked = ref(false);

onMounted(() => {
  // Left click
  cellElement.value.addEventListener('mousedown', (e) => {
    if(clicked.value || e.button === 2) return;
    if (props.cellValue) {
      cellClass.value = 'fill';
      emit('clickedFilledCell');
    }
    if (!props.cellValue) {
      cellClass.value = 'wrong-cross';
      emit('mistakeMade');
    }
    clicked.value = true;
  });

  // Right click
  cellElement.value.addEventListener('contextmenu', (el) => {
    el.preventDefault();
    if(clicked.value) return;
    if (!props.cellValue) cellClass.value = 'cross';
    if (props.cellValue) {
      cellClass.value = 'wrong-fill';
      emit('clickedFilledCell');
      emit('mistakeMade');
    }
    clicked.value = true;
  }, false);
})

</script>

<style lang="scss">
.cell-wrapper {
  .nono-cell {
    background-color: white;
    transition: 200ms;
    width: calc(var(--size-cell) * 1px);
    height: calc(var(--size-cell) * 1px);
  }

  &:nth-child(5n + 6) {
    .nono-cell {
      @apply border-l-gray-500;
    }
  }

  .cell-corner {
    position: absolute;
    top: 0px;
    left: 0px;
    width: 12px;
    height: 12px;
    fill: white;
    z-index: -1;
    transition: 200ms;
  }

  .cell-cross {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    fill: white;
    z-index: -1;
    opacity: 0;
    transition: 200ms;
  }

  &.fill {
    .nono-cell {
      background-color: black;
      border-color: #222;
    }
  }

  &.wrong-fill {
    .nono-cell {
      background-color: black;
      border-color: #222;
    }

    .cell-corner {
      fill: red;
      z-index: 1;
      opacity: 1;
    }
  }

  &.cross {
    .cell-cross {
      @apply fill-gray-400;
      z-index: 1;
      opacity: 1;
    }
  }

  &.wrong-cross {
    .cell-cross {
      fill: red;
      z-index: 1;
      opacity: 1;
    }
  }
}
</style>