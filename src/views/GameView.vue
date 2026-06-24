<script setup>
import { ref } from 'vue'
import GameBoard from '../components/GameBoard.vue'

const score = ref(0)
const gameBoardRef = ref(null)

const updateScore = (newScore) => {
  score.value = newScore
}

const handleRestart = () => {
  gameBoardRef.value?.restartGame()
}
</script>
<template>
  <div class="game-page">
    <div class="game-header">
      <h1>喵喵喵喵喵</h1>

      <button class="restart" @click="handleRestart">
        重置遊戲
      </button>

      <!-- <div class="score-box">
        分數：{{ score }}
      </div> -->
    </div>

    <GameBoard ref="gameBoardRef" :score="score" @update-score="updateScore" />
  </div>
</template>

<style scoped>
.game-page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.game-header {
  width: 624px;
  text-align: center;
  margin-top: 32px;
  margin-bottom: 8px;
}

.game-header h1 {
  margin: 0 0 12px;
}

.restart {
  margin-bottom: 12px;
}

.score-box {
  margin-bottom: 8px;
}
</style>