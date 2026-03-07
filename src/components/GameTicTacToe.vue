<script setup>
import { ref } from 'vue'
import IconX from './icons/IconX.vue'
import IconO from './icons/IconO.vue'

const game_field = ref([
  [null, null, null],
  [null, null, null],
  [null, null, null],
])

function getBorder(i, j) {
  if (i !== 1 && j !== 1) return

  if (i == 1 && j == 1) return 'border-x border-y'
  if (i == 1) return 'border-x'
  if (j == 1) return 'border-y'
}

const marker = ref('x')

function setMarker(i, j) {
  if (game_field.value[i][j]) return

  game_field.value[i][j] = marker.value
  marker.value = marker.value === 'x' ? 'o' : 'x'
}

const hoverCell = ref(null)

function isHover(i, j) {
  return hoverCell.value?.row === i && hoverCell.value?.col === j
}

</script>

<template>
  <div class="wrapper">
    <div class="line" v-for="(row, i) in game_field" :key="i">
      <div 
        v-for="(cell, j) in row"
        :key="j"
        :class="['cell', getBorder(i, j)]"
        @click="setMarker(i, j)"
        @mouseenter="hoverCell = { row: i, col: j}"
        @mouseleave="hoverCell = null"
      >
        <IconX v-if="cell === 'x'" class="icon-x" />
        <IconO v-else-if="cell === 'o'" class="icon-o" />

        <template v-else-if="isHover(i, j)" >
          <IconX v-if="marker === 'x'" class="icon-preview"/>
          <IconO v-else-if="marker === 'o'" class="icon-preview"/>
        </template>
      </div>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100svh;
}

.line {
  display: flex;
}

.cell {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100px;
  height: 100px;

  --br-px: 3px;
  --br-color: grey;
}

.cell:hover {
  content: 'x';
}

.border-x {
  border-top: var(--br-px) solid var(--br-color);
  border-bottom: var(--br-px) solid var(--br-color);
}

.border-y {
  border-right: var(--br-px) solid var(--br-color);
  border-left: var(--br-px) solid var(--br-color);
}

svg {
  width: 60px;
  height: 60px;
  stroke-width: 10;
  stroke-linecap: round;
}

:deep(.icon-x line) {
  stroke: #e53e3e;
}

:deep(.icon-o circle) {
  stroke: #3182ce;
}

:deep(.icon-preview line),
:deep(.icon-preview circle) {
  stroke: #cecece;
}

.preview {
  opacity: 0.4;
  color: #888;
}
</style>
