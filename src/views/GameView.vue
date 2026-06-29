<script setup>
import { ref, onMounted } from "vue";
import GameBoard from "../components/GameBoard.vue";
import ruleImg from "../img/rule.webp"; // 你的說明圖路徑自己改

import lv1Img from "../img/gameIcon/lv1.webp";
import lv2Img from "../img/gameIcon/lv2.webp";
import lv3Img from "../img/gameIcon/lv3.webp";
import lv4Img from "../img/gameIcon/lv4.webp";
import lv5Img from "../img/gameIcon/lv5.webp";
import lv6Img from "../img/gameIcon/lv6.webp";
import lv7Img from "../img/gameIcon/lv7.webp";
import lv8Img from "../img/gameIcon/lv8.webp";
import lv9Img from "../img/gameIcon/lv9.webp";
import lv10Img from "../img/gameIcon/lv10.webp";
import lv11Img from "../img/gameIcon/lv11.webp";
import lv12Img from "../img/gameIcon/lv12.webp";


const musicOn = ref(false);

function handleToggleMusic() {
  gameBoardRef.value?.toggleMusic();
  musicOn.value = !musicOn.value;
}

const bestScore = ref(0);
onMounted(() => {
  bestScore.value = Number(localStorage.getItem("neko-best-score")) || 0;
});
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

  if (newScore > bestScore.value) {
    bestScore.value = newScore;
    localStorage.setItem("neko-best-score", newScore);
  }
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
       <button class="icon-btn" @click="handleToggleMusic">
          {{ musicOn ? "🔇" : "🔊" }}
        </button>
        <div class="score-area">
          <div class="score-box">Score：{{ score }}</div>
          <div class="best-score">Best Score：{{ bestScore }}</div>
        </div>

        <div class="next-box">
          <p>Next:</p>

          <img v-if="BALL_IMAGES[nextLevel]" :src="BALL_IMAGES[nextLevel]" class="next-mini" />
        </div>
      </div>
      <GameBoard ref="gameBoardRef" :score="score" @update-score="updateScore" @update-next-level="updateNextLevel" />
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
  font-family: 'Fredoka', sans-serif;
  min-height: 100vh;

  background:
    linear-gradient(rgba(255, 255, 255, 0.15), rgba(255, 255, 255, 0.15)),
    url("../img/background.webp");

  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;

  display: flex;
  justify-content: center;
  align-items: center;
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
  font-family: 'Fredoka', sans-serif;
  font-size: 32px;
  width: 52px;
  height: 52px;
  border-radius: 50%;
  border: 3px solid #444;
  background: #fffaf0;
  cursor: pointer;
  transition: all 0.2s ease;
}


.icon-btn:hover,
.help-btn:hover,
.restart:hover {
  transform: scale(1.08);
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

.next-box p {
  font-weight: 700;
  margin: 0;
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
  max-width: 600px;
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

.score-area {
  text-align: center;
}

.score-box {
  font-size: 28px;
  font-weight: 800;
  line-height: 1;
}

.best-score {
  margin-top: 4px;
  font-size: 18px;
  font-weight: 700;
  color: #7d5a50;
}

/* rwd below */

@media (max-width: 768px) {
  .game-shell {
    width: 100%;
  }

  .top-bar {
    width: 480px;
    max-width: calc(100vw - 32px);
  }
}

@media (max-width: 576px) {
  .game-page {
    overflow-x: hidden;
  }

  .top-bar {
    width: 360px;
    max-width: calc(100vw - 24px);
  }

  .score-box {
    font-size: 22px;
  }

  .best-score {
    font-size: 12px;
  }

  .help-btn,
  .restart {
    width: 32px;
    height: 32px;
    font-size: 16px;
  }
}
</style>
