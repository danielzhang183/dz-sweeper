<script setup lang="ts">
interface BlockState {
  x: number
  y: number
  revealed: boolean
  mine?: boolean
  flagged?: boolean
  adjacentMines: number
}

const WIDTH = 5
const HEIGHT = 5
const state = ref(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH },
      (_, x): BlockState => ({
        x, y, adjacentMines: 0, revealed: false,
      })),
  ),
)

function generateMines(initial: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) <= 1)
        continue
      if (Math.abs(initial.y - block.y) <= 1)
        continue
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers()
}

function expandZero(block: BlockState) {
  if (block.adjacentMines)
    return
  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expandZero(s)
    }
  })
}

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]

const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
]

function updateNumbers() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined
    return state.value[y2][x2]
  })
    .filter(Boolean) as BlockState[]
}
let mineGenerated = false
const dev = true

function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
  checkGameState()
}

function onClick(e: MouseEvent, block: BlockState) {
  e.preventDefault()

  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  if (block.mine)
    alert('BOOm')
  expandZero(block)
  checkGameState()
}

function getBlockClass(block: BlockState) {
  if (!block.revealed)
    return 'bg-gray-500/10'
  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}

watchEffect(checkGameState)

function checkGameState() {
  if (!mineGenerated)
    return
  const blocks = state.value.flat()
  if (blocks.every(block => block.revealed || block.flagged)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('You lose')
    else
      alert('You win')
  }
}
</script>

<template>
  <div>
    MineSweeper

    <div p5>
      <div
        v-for="row, y in state"
        :key="y"
        flex="~"
        items-center justify-center
      >
        <button
          v-for="block, x in row"
          :key="x"
          flex="~"
          items-center justify-center
          w-10 h-10 m="0.5"
          hover="bg-gray/10"
          border="1 gray-400/10"
          :class="getBlockClass(block)"
          @click="onClick($event, block)"
          @contextmenu.prevent="onRightClick(block)"
        >
          <template v-if="block.flagged">
            <div i-mdi:flag text-red />
          </template>
          <template v-else-if="block.revealed || dev">
            <div v-if="block.mine" i-mdi:mine />
            <div v-else>
              {{ block.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
