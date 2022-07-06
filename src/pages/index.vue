<script setup lang="ts">
import type { BlockState } from '~/type'
import { isDev, toggleDev } from '~/composables'

const WIDTH = 5
const HEIGHT = 5
const state = ref<BlockState[][]>([])

function reset() {
  state.value = Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH },
      (_, x): BlockState => ({
        x, y, adjacentMines: 0, revealed: false,
      })),
  )
}

function generateMines(state: BlockState[][], initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) <= 1)
        continue
      if (Math.abs(initial.y - block.y) <= 1)
        continue
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers(state)
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

function updateNumbers(state: BlockState[][]) {
  state.forEach((row) => {
    row.forEach((block) => {
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

function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
  checkGameState()
}

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(state.value, block)
    mineGenerated = true
  }
  block.revealed = true
  if (block.mine)
    alert('BOOm')
  expandZero(block)
  checkGameState()
}

watchEffect(checkGameState)
reset()

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

    <div @click="toggleDev()">
      {{ isDev }}
    </div>
    <div p5>
      <div
        v-for="row, y in state"
        :key="y"
        flex="~"
        items-center justify-center
      >
        <MineBlock
          v-for="block, x in row" :key="x"
          :block="block"
          @click="onClick(block)"
          @contextmenu.prevent="onRightClick(block)"
        />
      </div>
    </div>
  </div>
</template>
