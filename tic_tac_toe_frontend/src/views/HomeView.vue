<script setup lang="ts">
import { computed, ref } from 'vue'

type Cell = 'X' | 'O' | ''
type Winner = 'X' | 'O' | 'Draw' | null

const board = ref<Cell[]>(Array(9).fill(''))
const xIsNext = ref(true)
const winner = ref<Winner>(null)
const winningLine = ref<number[] | null>(null)

// PUBLIC_INTERFACE
function resetGame(): void {
  /** Resets the Tic Tac Toe board, status, and winner state. */
  board.value = Array(9).fill('')
  xIsNext.value = true
  winner.value = null
  winningLine.value = null
}

const currentPlayer = computed(() => (xIsNext.value ? 'X' : 'O'))

const statusText = computed(() => {
  if (winner.value === 'Draw') return 'It’s a draw!'
  if (winner.value === 'X' || winner.value === 'O') return `Player ${winner.value} wins!`
  return `Player ${currentPlayer.value}'s turn`
})

const lineCombos: number[][] = [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8],
  [0, 4, 8],
  [2, 4, 6],
]

function evaluateWinner(b: Cell[]): { win: Winner; line: number[] | null } {
  for (const combo of lineCombos) {
    const [a, bIdx, c] = combo
    if (b[a] && b[a] === b[bIdx] && b[a] === b[c]) {
      return { win: b[a] as Winner, line: combo }
    }
  }
  if (b.every((cell) => cell !== '')) return { win: 'Draw', line: null }
  return { win: null, line: null }
}

// PUBLIC_INTERFACE
function handleCellClick(index: number): void {
  /** Handles a click on a board cell to place the current player's mark and advance the game. */
  if (winner.value || board.value[index] !== '') return
  board.value[index] = currentPlayer.value as Cell
  const result = evaluateWinner(board.value)
  winner.value = result.win
  winningLine.value = result.line

  if (!winner.value) {
    xIsNext.value = !xIsNext.value
  }
}

const appTitle = 'Tic Tac Toe'
</script>

<template>
  <main class="page">
    <div class="card">
      <h1 class="title">{{ appTitle }}</h1>
      <p
        class="status"
        :class="{
          success: winner === 'X' || winner === 'O',
          draw: winner === 'Draw'
        }"
        role="status"
        aria-live="polite"
      >
        {{ statusText }}
      </p>

      <div class="board" role="grid" aria-label="Tic Tac Toe board">
        <button
          v-for="(cell, idx) in board"
          :key="idx"
          class="cell"
          :class="{
            x: cell === 'X',
            o: cell === 'O',
            win: winningLine && winningLine.includes(idx)
          }"
          role="gridcell"
          :aria-label="cell ? 'Cell ' + (idx+1) + ' ' + cell : 'Cell ' + (idx+1) + ' empty'"
          :disabled="!!winner || !!cell"
          @click="handleCellClick(idx)"
        >
          <span class="mark">{{ cell }}</span>
        </button>
      </div>

      <div class="actions">
        <button class="btn" @click="resetGame" :aria-label="'Restart game'">
          Restart
        </button>
      </div>
    </div>
  </main>
</template>

<style scoped>
/* Ocean Professional Theme Variables */
:root {
  --primary: #2563EB;
  --secondary: #F59E0B;
  --success: #F59E0B;
  --error: #EF4444;
  --background: #f9fafb;
  --surface: #ffffff;
  --text: #111827;
  --shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
  --ring: 0 0 0 6px rgba(37, 99, 235, 0.12);
}

.page {
  min-height: 100vh;
  display: grid;
  place-items: center;
  background: radial-gradient(1000px 600px at 50% -20%, rgba(37, 99, 235, 0.08), rgba(0,0,0,0) 60%),
              var(--background);
  padding: 1.5rem;
}

.card {
  width: 100%;
  max-width: 520px;
  background: var(--surface);
  border-radius: 16px;
  box-shadow: var(--shadow);
  padding: 1.25rem 1.25rem 1.5rem;
  border: 1px solid rgba(17, 24, 39, 0.06);
}

.title {
  text-align: center;
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: 0.2px;
  margin-bottom: 0.25rem;
}

.status {
  text-align: center;
  color: #374151;
  margin-bottom: 1rem;
}
.status.success {
  color: var(--secondary);
  font-weight: 600;
}
.status.draw {
  color: #6b7280;
  font-weight: 600;
}

.board {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
  padding: 10px;
  background: linear-gradient(180deg, rgba(37,99,235,0.06), rgba(249,250,251,1));
  border-radius: 14px;
  border: 1px solid rgba(37, 99, 235, 0.15);
  box-shadow: inset 0 1px 0 rgba(255,255,255,0.6);
}

.cell {
  position: relative;
  width: 100%;
  aspect-ratio: 1 / 1;
  border: none;
  border-radius: 12px;
  background: #ffffff;
  color: var(--text);
  font-size: clamp(2.2rem, 8vw, 3.2rem);
  font-weight: 800;
  letter-spacing: 1px;
  display: grid;
  place-items: center;
  cursor: pointer;
  box-shadow: 0 1px 0 rgba(17,24,39,0.08), 0 6px 18px rgba(17,24,39,0.06);
  transition: transform 180ms ease, box-shadow 180ms ease, background 180ms ease, border-color 180ms ease;
  border: 1px solid rgba(17, 24, 39, 0.06);
}

.cell:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 24px rgba(17,24,39,0.12);
}
.cell:active:not(:disabled) {
  transform: translateY(0);
  box-shadow: 0 6px 16px rgba(17,24,39,0.12);
}

.cell:disabled {
  cursor: default;
}

.cell.x .mark {
  color: var(--primary);
  text-shadow: 0 2px 10px rgba(37,99,235,0.15);
}
.cell.o .mark {
  color: var(--secondary);
  text-shadow: 0 2px 10px rgba(245,158,11,0.2);
}

.cell.win {
  outline: none;
  box-shadow: 0 0 0 2px var(--secondary), 0 12px 26px rgba(245,158,11,0.25);
  background: linear-gradient(180deg, #fff, rgba(245,158,11,0.06));
}

.mark {
  transform: translateZ(0);
}

.actions {
  display: flex;
  justify-content: center;
  margin-top: 14px;
}

.btn {
  appearance: none;
  border: none;
  background: var(--primary);
  color: white;
  font-weight: 600;
  padding: 0.7rem 1.1rem;
  border-radius: 10px;
  box-shadow: 0 10px 20px rgba(37, 99, 235, 0.25);
  cursor: pointer;
  transition: transform 160ms ease, box-shadow 160ms ease, filter 160ms ease;
}
.btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 12px 24px rgba(37, 99, 235, 0.35);
  filter: brightness(1.02);
}
.btn:active {
  transform: translateY(0);
}

@media (max-width: 420px) {
  .card {
    padding: 1rem;
    border-radius: 14px;
  }
  .title {
    font-size: 1.5rem;
  }
  .board {
    gap: 8px;
    padding: 8px;
  }
  .btn {
    width: 100%;
  }
}
</style>
