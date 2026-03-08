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

    <main>
      <div class="line" v-for="(row, i) in game_field" :key="i">
        <div
          v-for="(cell, j) in row"
          :key="j"
          :class="['cell', getBorder(i, j), getWinnerCell(i, j)]"
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
import { computed, ref } from 'vue'
import IconX from './icons/IconX.vue'
import IconO from './icons/IconO.vue'

function initialGame() {
  return [
    [null, null, null],
    [null, null, null],
    [null, null, null],
  ]
}

const game_field = ref(initialGame())
const win_field = ref(initialGame())
const winner = ref(null)

const marker = ref('x')

function toggleMarker(i, j) {
  game_field.value[i][j] = marker.value
  marker.value = marker.value === 'x' ? 'o' : 'x'
}

function getWinner(type, win, i) {
  winner.value = win

  const size = game_field.value.length

  for (let j = 0; j < size; j++) {
    if (type === 'row') win_field.value[i][j] = win
    if (type === 'col') win_field.value[j][i] = win
    if (type === 'diag_1') win_field.value[j][j] = win
    if (type === 'diag_2') win_field.value[j][size - 1 - j] = win
  }
}

function handleWinner(type, line, i) {
  if (line[0] && line.every(el => el === line[0])) {
    getWinner(type, line[0], i)
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
  handleWinner('diag_1', diag_1)

  const diag_2 = game_field.value.map((raw, i) => raw[size - 1 - i])
  handleWinner('diag_2', diag_2)
}

function setMarker(i, j) {
  if (game_field.value[i][j] || winner.value) return
  toggleMarker(i, j)
  checkWinner()
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
  if(winner.value) return hoverCell.value = null
  return hoverCell.value?.row === i && hoverCell.value?.col === j
}

const fieldFull = computed(() => {
  const matrix = game_field.value
  for (let i = 0; i < matrix.length; i++) {
    for (let j = 0; j < matrix[i].length; j++) {
      if (!matrix[i][j]) return false
    }
  }
  return true
})

const endGame = computed(() => {
  return winner.value || fieldFull.value
})

function resetGame() {
  game_field.value = initialGame()
  win_field.value = initialGame()
  winner.value = null
  marker.value = 'x'
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

.cell {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100px;
  height: 100px;

  --br-px: 3px;
  --br-color: grey;
}

.border-left {
  border-left: var(--br-px) solid var(--br-color);
}

.border-top {
  border-top: var(--br-px) solid var(--br-color);
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
