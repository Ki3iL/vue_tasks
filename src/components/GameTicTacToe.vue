<template>
  <div class="container main-font">

    <header class="wrapper">
      <div v-if="winner" class="flex-center">
      <span>🎉 Победа! 🎉</span>
      </div>
      <div v-else-if="fieldFull" class="flex-center">
        <span> Ничья! Победила Дружба!</span>
        <span> 🤚😃🤝😃✋ </span>
      </div>
    </header>

    <main class="field-wrapper" ref="field" >

      <svg class="win-line">
        <line v-for="(line, i) in winner_lines" :key="i"
        :x1="line.x1" 
        :x2="line.x2"
        :y1="line.y1"  
        :y2="line.y2"/>
      </svg>

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
    </main>

    <footer class="wrapper">
      <button v-if="endGame" @click="resetGame" type="button" class="reset flex-center">
        <span>Начать заново</span>
      </button>
    </footer>

  </div>
</template>

<script setup>
import { computed, ref} from 'vue'
import IconX from './icons/IconX.vue'
import IconO from './icons/IconO.vue'

const GRID_SIZE = 4
const CELL_SIZE = 100
const CELL_SIZE_PX = CELL_SIZE + 'px'
const BORDER_SIZE = 5
const BORDER_SIZE_PX = BORDER_SIZE + 'px'
const FIELD_WIDTH = CELL_SIZE * GRID_SIZE + BORDER_SIZE * (GRID_SIZE - 1)

function createMatrix(size, value = null) {
  return Array.from({ length: size }, () =>
    Array.from({ length: size }, () => value)
  )
}

const game_field = ref(createMatrix(GRID_SIZE))
const win_field = ref(createMatrix(GRID_SIZE))
const winner_lines = ref([])
const winner = ref(null)

const marker = ref('x')

function getWinner(type, win, i) {
  winner.value = win

  const size = game_field.value.length

  for (let j = 0; j < size; j++) {
    if (type === 'row') win_field.value[i][j] = win
    if (type === 'col') win_field.value[j][i] = win
    if (type === 'diag') win_field.value[j][Math.abs(i - j)] = win
  }
}

function getWinnerLine(type, i) {
  const step = CELL_SIZE + BORDER_SIZE
  const line_position = i * step + CELL_SIZE / 2

  const line = {x1: 0, x2: 0, y1: 0, y2: 0}
  if (type === 'row') {
    line.x2 = FIELD_WIDTH
    line.y1 = line.y2 = line_position
  }

  if (type === 'col') {
    line.x1 = line.x2 = line_position
    line.y2 = FIELD_WIDTH
  }

  if (type === 'diag') {
    line.x1 = (i === 0) ? 0 : FIELD_WIDTH
    line.x2 = (i !== 0) ? 0 : FIELD_WIDTH
    line.y2 = FIELD_WIDTH
  }
  winner_lines.value.push(line)
}

function handleWinner(type, line, i) {
  if (line[0] && line.every(el => el === line[0])) {
    getWinner(type, line[0], i)
    getWinnerLine(type, i)
  } 
}

function checkWinner() {
  const matrix = game_field.value
  const size = matrix.length

  for (let i = 0; i < size; i++) {
    const row = matrix[i]
    handleWinner('row', row, i)
    
    const col = matrix.map(raw => raw[i])
    handleWinner('col', col, i)
  }

  const diag_1 = game_field.value.map((raw, i) => raw[i])
  handleWinner('diag', diag_1, 0)

  const diag_2 = game_field.value.map((raw, i) => raw[size - 1 - i])
  handleWinner('diag', diag_2, (size - 1))
}

function setMarker(i, j) {
  if (game_field.value[i][j] || winner.value) return
  
  game_field.value[i][j] = marker.value
  
  checkWinner()
  
  if (!winner.value) {
    marker.value = marker.value === 'x' ? 'o' : 'x'
  }
}

//NOTE - End Game
const fieldFull = computed(() =>
  game_field.value.every(row => row.every(cell => cell !== null))
)

const endGame = computed(() => {
  return winner.value || fieldFull.value
})

function resetGame() {
  game_field.value = createMatrix(GRID_SIZE)
  win_field.value = createMatrix(GRID_SIZE)
  winner.value = null
  winner_lines.value = []
  marker.value = 'x'
}

//NOTE - Style logic

function getBorder(i, j) {
  const top = (i !== 0) ? 'border-top' : '';
  const left = (j !== 0) ? 'border-left' : '';
  return `${top} ${left}`
}

function getWinnerCell(i, j) {
  const win = win_field.value
  if (!win[i][j]) return
  return 'winner-cell'
}

const hoverCell = ref(null)
function isHover(i, j) {
  return !winner.value 
        && hoverCell.value?.row === i 
        && hoverCell.value?.col === j
}

</script>

<style scoped>
.main-font {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 1.4rem;
}

.flex-center {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}


.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100svh;
}

.line {
  display: flex;
}

.field-wrapper { 
  position: relative;
}

.cell {
  box-sizing: content-box;
  display: flex;
  justify-content: center;
  align-items: center;
  width: v-bind(CELL_SIZE_PX);
  height: v-bind(CELL_SIZE_PX);
  position: relative;

  --br-px: v-bind(BORDER_SIZE_PX);
  --br-color: grey;
}

.border-left {
  border-left: var(--br-px) solid var(--br-color);
}

.border-top {
  border-top: var(--br-px) solid var(--br-color);
}

.win-line {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  stroke: #22ca3e;
  stroke-width: 5;
  stroke-linecap: round;
  z-index: 100;
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

.winner-cell {
  background-color: rgb(165, 241, 165);
}

.wrapper {
  height: 100px;
  display: flex;
  align-items: center;
}

.reset {
  height: 50px;
  padding: 10px;
  background-color: rgba(10, 60, 160, 0.8);
  border-width: 0;
  border-radius: 12px;
  color: rgb(255, 255, 255);
}

.reset:hover {
  background-color: rgba(10, 60, 160, 1);
}
</style>
