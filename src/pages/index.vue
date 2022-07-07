<script setup lang="ts">
import { toggleDev } from '~/composables'

const play = new GamePlay(10, 10)
const state = play.state
</script>

<template>
  <div>
    MineSweeper

    <div flex="~ gap-1" justify-center>
      <button btn @click="toggleDev()">
        {{ isDev ? 'DEV' : 'NORMAL' }}
      </button>
      <button btn @click="play.reset()">
        RESET
      </button>
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
          @click="play.onClick(block)"
          @contextmenu.prevent="play.onRightClick(block)"
        />
      </div>
    </div>
  </div>
</template>
