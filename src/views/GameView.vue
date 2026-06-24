<script setup>
import { ref } from "vue";
import GameBoard from "../components/GameBoard.vue";
import ruleImg from "../img/rule.png"; // 你的說明圖路徑自己改

import lv1Img from "../img/gameIcon/lv1.png";
import lv2Img from "../img/gameIcon/lv2.png";
import lv3Img from "../img/gameIcon/lv3.png";
import lv4Img from "../img/gameIcon/lv4.png";
import lv5Img from "../img/gameIcon/lv5.png";
import lv6Img from "../img/gameIcon/lv6.png";
import lv7Img from "../img/gameIcon/lv7.png";
import lv8Img from "../img/gameIcon/lv8.png";
import lv9Img from "../img/gameIcon/lv9.png";
import lv10Img from "../img/gameIcon/lv10.png";

const BALL_IMAGES = {
  1: lv1Img,
  2: lv2Img,
  3: lv3Img,
  4: lv4Img,
  5: lv5Img,
  6: lv6Img,
  7: lv7Img,
  8: lv8Img,
  9: lv9Img,
  10: lv10Img,
};

const score = ref(0);
const gameBoardRef = ref(null);
const showRule = ref(false);
const nextLevel = ref(1);

const updateNextLevel = (level) => {
  nextLevel.value = level;
};
const updateScore = (newScore) => {
  score.value = newScore;
};

const handleRestart = () => {
  gameBoardRef.value?.restartGame();
};
</script>

<template>
  <div class="game-page">
    <div class="game-shell">
      <div class="top-bar">
        <button class="help-btn" @click="showRule = true">?</button>
        <button class="restart" @click="handleRestart">↺</button>
        <div class="score-box">分數：{{ score }}</div>

        <div class="next-box">
          <p>Next</p>

          <img
            v-if="BALL_IMAGES[nextLevel]"
            :src="BALL_IMAGES[nextLevel]"
            class="next-mini"
          />
        </div>
      </div>
      <GameBoard
        ref="gameBoardRef"
        :score="score"
        @update-score="updateScore"
        @update-next-level="updateNextLevel"
      />
    </div>

    <div v-if="showRule" class="rule-mask" @click="showRule = false">
      <div class="rule-modal" @click.stop>
        <button class="rule-close" @click="showRule = false">×</button>
        <img :src="ruleImg" alt="遊戲規則說明" />
      </div>
    </div>
  </div>
</template>
<style scoped>
.game-page {
  min-height: 100vh;
  display: flex;
  justify-content: center;
}

.game-shell {
  width: 624px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.top-bar {
  width: 480px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 20px 0 12px;
}

.top-left {
  display: flex;
  gap: 8px;
}

.icon-btn,
.help-btn,
.restart {
  width: 36px;
  height: 36px;
  border: 2px solid #333;
  border-radius: 50%;
  background: #fff7ef;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
}

.score-box {
  font-size: 28px;
  font-weight: 800;
  line-height: 1;
}

.top-right {
  width: 60px;
  text-align: center;
  font-size: 14px;
  font-weight: bold;
}

.next-mini {
  width: 44px;
  height: 44px;
  object-fit: contain;
  margin-top: 4px;
}

.rule-mask {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.rule-modal {
  position: relative;
  max-width: 420px;
  width: 80%;
}

.rule-modal img {
  width: 100%;
  display: block;
  border-radius: 20px;
}

.rule-close {
  position: absolute;
  top: -12px;
  right: -12px;
  width: 36px;
  height: 36px;
  border: 2px solid #333;
  border-radius: 50%;
  background: #fff;
  font-size: 24px;
  cursor: pointer;
}
</style>
