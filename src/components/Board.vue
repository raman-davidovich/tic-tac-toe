<script setup lang="ts">
import { ref, computed, watch, onMounted, watchEffect } from 'vue'
import Field from './Field.vue'

type FieldValue = '' | 'X' | 'O'

type WinnerResult = {
  winner: FieldValue
  line: number[]
} | null

const player = ref<FieldValue>('X')
const fields = ref<FieldValue[][]>([
  ['', '', ''],
  ['', '', ''],
  ['', '', ''],
])

const winner = computed<WinnerResult>(() => calculateWinner(fields.value.flat()))
const isDraw = computed(() => {
  return !winner.value && fields.value.flat().every((cell) => cell !== '')
})
const history = ref<Array<'X' | 'O'>>([])
const cursorPosition = ref({ x: 0, y: 0 })
const isHoveringBoard = ref(false)

const calculateWinner = (fields: FieldValue[]): WinnerResult => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ]
  for (const line of lines) {
    const [a, b, c] = line
    if (fields[a] && fields[a] === fields[b] && fields[a] === fields[c]) {
      return {
        winner: fields[a],
        line: [a, b, c],
      }
    }
  }
  return null
}

const isWinningCell = (x: number, y: number): boolean => {
  if (!winner.value) return false
  const cellIndex = x * 3 + y
  return winner.value.line.includes(cellIndex)
}

const move = (x: number, y: number): void => {
  if (winner.value) return

  if (fields.value[x][y] !== '') return

  fields.value[x][y] = player.value
  player.value = player.value === 'O' ? 'X' : 'O'
}

const reset = (): void => {
  player.value = 'X'
  fields.value = [
    ['', '', ''],
    ['', '', ''],
    ['', '', ''],
  ]
}

watch(winner, (current, previous) => {
  if (current?.winner && !previous?.winner) {
    history.value.push(current.winner)
    localStorage.setItem('history', JSON.stringify(history.value))
  }
})

onMounted(() => {
  try {
    const storedHistory = localStorage.getItem('history')
    if (storedHistory) {
      const parsed = JSON.parse(storedHistory)

      const isHistoryValid = (data: unknown): data is Array<'X' | 'O'> => {
        return Array.isArray(data) && data.every((item) => item === 'X' || item === 'O')
      }

      if (isHistoryValid(parsed)) {
        history.value = parsed
      }
    }
  } catch (error) {
    console.error('Error loading history:', error)
    history.value = []
  }
})

const trackMouse = (event: MouseEvent) => {
  cursorPosition.value = { x: event.clientX, y: event.clientY }
}

watchEffect((onCleanup) => {
  window.addEventListener('mousemove', trackMouse)
  onCleanup(() => window.removeEventListener('mousemove', trackMouse))
})
</script>

<template>
  <div class="board" @mouseenter="isHoveringBoard = true" @mouseleave="isHoveringBoard = false">
    <h2 v-if="winner">Winner: {{ winner.winner }}</h2>
    <h2 v-else-if="isDraw" class="draw-title">Draw!</h2>
    <h2 v-else>Players Move: {{ player }}</h2>

    <button class="reset-button" @click="reset">Reset</button>

    <div v-for="(row, x) in fields" :key="x" class="row">
      <Field
        v-for="(cell, y) in row"
        :key="y"
        :value="cell"
        :is-winning="isWinningCell(x, y)"
        @click="move(x, y)"
      />
    </div>

    <h2 class="history-title">History</h2>
    <div class="history-container">
      <div v-for="(game, index) in history" :key="index" class="history-item">
        <span class="history-game-number">Game #{{ index + 1 }}</span>
        <span class="history-game-result" :data-winner="game">{{ game }} won</span>
      </div>

      <div v-if="history.length === 0" class="history-item history-empty">No games played yet</div>
    </div>

    <div
      v-if="!winner && !isDraw && isHoveringBoard"
      class="cursor-preview"
      :style="{ left: `${cursorPosition.x + 20}px`, top: `${cursorPosition.y - 10}px` }"
    >
      {{ player }}
    </div>
  </div>
</template>

<style scoped lang="scss">
$primary-color: #2c3e50;
$secondary-color: #3498db;
$background: #f0f0f0;

.board {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 20px;
  background-color: $background;
  min-height: 100vh;

  h2 {
    color: $primary-color;
    margin: 0;
    font-size: 2rem;

    &.draw-title {
      color: #f39c12;
      animation: shake 0.5s ease-in-out;
    }
  }

  .reset-button {
    padding: 10px 20px;
    font-size: 1.1rem;
    background-color: $secondary-color;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;

    &:hover {
      background-color: rgba($secondary-color, 0.8);
    }
  }

  .row {
    display: flex;
    gap: 10px;
  }

  .history {
    display: flex;
    width: 100%;
    flex-direction: column;
    align-items: center;
    &-title {
      margin: 30px 0 15px;
      font-size: 1.8rem;
      color: $primary-color;
    }

    &-container {
      width: 100%;
      max-width: 400px;
      background: white;
      border-radius: 10px;
      padding: 15px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    }

    &-item {
      padding: 12px 10px;
      margin: 8px 0;
      background: #f8f9fa;
      border-radius: 6px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      transition: transform 0.2s;

      &:hover {
        transform: translateX(5px);
      }
      &:nth-child(odd) {
        background: #fdfdfd;
      }
    }
    &-game {
      &-number {
        color: #7f8c8d;
        font-weight: 500;
      }

      &-result {
        font-weight: bold;
        text-transform: uppercase;

        &[data-winner='X'] {
          color: #e74c3c;
        }

        &[data-winner='O'] {
          color: #2ecc71;
        }
      }
    }
  }

  .cursor-preview {
    position: fixed;
    pointer-events: none;
    font-size: 2rem;
    font-weight: bold;
    color: rgba($secondary-color, 0.8);
    animation: cursor-float 1.5s ease-in-out infinite;
  }
}

@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-5px);
  }
  50% {
    transform: translateX(5px);
  }
  75% {
    transform: translateX(-3px);
  }
}

@keyframes cursor-float {
  0% {
    transform: translate(-50%, -50%) scale(1);
  }
  50% {
    transform: translate(-50%, -50%) scale(1.1);
  }
  100% {
    transform: translate(-50%, -50%) scale(1);
  }
}
</style>
