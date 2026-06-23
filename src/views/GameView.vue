<script setup>
import GameBoard from '../components/GameBoard.vue'
</script>

<template>
  <div class="game-page">
    <h1>🐱 Neko Merge Game</h1>

    <GameBoard />
  </div>
</template>

<style scoped>
.game-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px;
}
</style>