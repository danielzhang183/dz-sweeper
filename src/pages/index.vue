<script setup lang="ts">
import { isDev, toggleDev } from '~/composables'

const play = new GamePlay(10, 10, 10)

const now = $(useNow())
const timerMs = $computed(() => Math.round((+now - play.state.value.startMS) / 1000))
useStorage('dz-sweeper-state', play.state)
const state = $computed(() => play.board)
const mineRest = $computed(() => {
  if (!play.state.value.mineGenerated)
    return play.mines
  return play.blocks.reduce((a, b) => (a + (b.mine ? 1 : 0) - (b.flagged ? 1 : 0)), 0)
})

function newGame(difficulty: 'easy' | 'medium' | 'hard') {
  switch (difficulty) {
    case 'easy':
      play.reset(9, 9, 10)
      break
    case 'medium':
      play.reset(16, 16, 40)
      break
    case 'hard':
      play.reset(16, 30, 99)
      break
    default:
  }
}

watchEffect(() => {
  play.checkGameState()
})
</script>

<template>
  <div>
    MineSweeper

    <div flex="~ gap-1" justify-center p4>
      <button btn @click="toggleDev()">
        {{ isDev ? 'DEV' : 'NORMAL' }}
      </button>
      <button btn @click="play.reset()">
        NEW GAME
      </button>
      <button btn @click="newGame('easy')">
        Easy
      </button>
      <button btn @click="newGame('medium')">
        Medium
      </button>
      <button btn @click="newGame('hard')">
        Hard
      </button>
    </div>

    <div flex="~ gap-10" justify-center>
      <div font-mono text-2xl flex="~ gap-1" item-center>
        <div i-mdi-mine />
        {{ mineRest }}
      </div>
      <div font-mono text-2xl flex="~ gap-1" item-center>
        <div i-carbon-timer />
        {{ timerMs }}
      </div>
    </div>

    <div p5 w-full overflow-auto>
      <div
        v-for="row, y in state"
        :key="y"
        flex="~"
        items-center justify-center w-max ma
      >
        <MineBlock
          v-for="block, x in row" :key="x"
          :block="block"
          @click="play.onClick(block)"
          @dblclick="play.autoExpand(block)"
          @contextmenu.prevent="play.onRightClick(block)"
        />
      </div>
    </div>

    <Confetti :passed="play.state.value.gameState === 'won'" />
  </div>
</template>
