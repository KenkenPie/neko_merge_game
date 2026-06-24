<script setup>
let canDropBall = true;
const DROP_COOLDOWN = 300;
const mergeEffects = ref([]);

import { onMounted, ref } from "vue";
import Matter from "matter-js";
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
import lv11Img from "../img/gameIcon/lv11.png";
import lv12Img from "../img/gameIcon/lv12.png";

/* =========================
   Vue 狀態資料
   ========================= */

// 目前準備掉下來的球等級
const currentLevel = ref(1);

// 右邊預告：下一顆球的等級
const nextLevel = ref(1);

// 分數
const props = defineProps({
  score: Number,
});

const emit = defineEmits(["update-score", "update-next-level"]);

/* =========================
   遊戲基本設定
   ========================= */

//*死亡線
const isGameOver = ref(false);
const GAME_OVER_LINE = 140;

// 最大等級，超過 LV10 就不再合成
const MAX_LEVEL = 10;

// 遊戲區尺寸
const BOARD_WIDTH = 420;
const BOARD_HEIGHT = 640;

// 各等級球的直徑尺寸，對應 Figma 圖片尺寸
const BALL_SIZES = {
  1: 40,
  2: 52,
  3: 64,
  4: 84,
  5: 104,
  6: 124,
  7: 140,
  8: 156,
  9: 170,
  10: 184,
  11: 200,
  12: 220,
};
// 依照等級計算球半徑

function getBallRadius(level) {
  return BALL_SIZES[level] / 2;
}

// 測試lv1球改圖片
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

// 不同等級的球顏色
const BALL_COLORS = {
  1: "orange",
  2: "red",
  3: "blue",
  4: "green",
  5: "purple",
  6: "pink",
  7: "yellow",
  8: "brown",
  9: "gray",
  10: "black",
};

/* =========================
   預覽球設定
   ========================= */

// 預覽球的 X 座標，預設在中間
const previewX = ref(BOARD_WIDTH / 2);

// 滑鼠移動時，讓預覽球跟著滑鼠左右移動
function movePreview(event) {
  const rect = gameBoard.value.getBoundingClientRect();
  previewX.value = event.clientX - rect.left;
}

// 合成特效

function addMergeEffect(x, y) {
  const id = Date.now() + Math.random();

  mergeEffects.value.push({ id, x, y });

  setTimeout(() => {
    mergeEffects.value = mergeEffects.value.filter((item) => item.id !== id);
  }, 400);
}

/* =========================
   工具函式
   ========================= */

// 隨機產生 LV1～LV3 的球
function getRandomLevel() {
  return Math.floor(Math.random() * 4) + 1;
}

// 根據球的等級決定要使用圖片還是顏色
// 如果該等級有對應圖片，就使用圖片(texture)
// 如果沒有圖片，就使用原本的顏色圓球

function getBallRender(level) {
  if (BALL_IMAGES[level]) {
    return {
      sprite: {
        texture: BALL_IMAGES[level],
        xScale: 1,
        yScale: 1,
      },
    };
  }

  return {
    fillStyle: BALL_COLORS[level],
  };
}

function getBallImage(level) {
  return BALL_IMAGES[level] || null;
}

/* =========================
   Matter.js 物理引擎變數
   ========================= */

// 遊戲區 DOM
const gameBoard = ref(null);

// Matter.js 主要物件
let engine;
let render;
let runner;

/* =========================
   初始化遊戲
   ========================= */

onMounted(() => {
  // 一開始先抽目前球和下一顆球
  currentLevel.value = getRandomLevel();
  nextLevel.value = getRandomLevel();

  const { Engine, Render, Runner, Bodies, Composite, Events } = Matter;

  // 建立物理引擎
  engine = Engine.create();

  // 建立 Matter.js 畫布
  render = Render.create({
    element: gameBoard.value,
    engine: engine,
    options: {
      width: BOARD_WIDTH,
      height: BOARD_HEIGHT,
      wireframes: false,
      background: "#ffffff",
    },
  });

  /* =========================
     建立牆壁和地板
     ========================= */

  const ground = Bodies.rectangle(
    BOARD_WIDTH / 2,
    BOARD_HEIGHT + 10,
    BOARD_WIDTH,
    20,
    {
      isStatic: true,
      render: {
        visible: false,
      },
    },
  );

  const leftWall = Bodies.rectangle(-25, BOARD_HEIGHT / 2, 50, BOARD_HEIGHT, {
    isStatic: true,
    render: {
      fillStyle: "#333",
    },
  });

  const rightWall = Bodies.rectangle(
    BOARD_WIDTH + 25,
    BOARD_HEIGHT / 2,
    50,
    BOARD_HEIGHT,
    {
      isStatic: true,
      render: {
        fillStyle: "#333",
      },
    },
  );

  // 把牆壁和地板加入物理世界
  Composite.add(engine.world, [ground, leftWall, rightWall]);

  // 開始渲染畫面
  Render.run(render);

  // 開始跑物理引擎
  runner = Runner.create();
  Runner.run(runner, engine);

  /* =========================
     GAMEOVER設定
     ========================= */

  //      之後建議改成：

  // if (
  //   ball.position.y - ball.circleRadius <
  //   GAME_OVER_LINE
  // )

  // 意思： 球的最上緣碰到紅線 看之後的圖片大小
  Events.on(engine, "afterUpdate", () => {
    if (isGameOver.value) return;

    const balls = engine.world.bodies.filter((body) => body.level);

    const now = Date.now();

    for (const ball of balls) {
      if (
        now - ball.createdAt > 1500 &&
        ball.speed < 0.5 &&
        ball.position.y - ball.circleRadius < GAME_OVER_LINE
      ) {
        isGameOver.value = true;
        console.log("GAME OVER");
        break;
      }
    }
  });
  /* =========================
     碰撞偵測與合成邏輯
     ========================= */

  Events.on(engine, "collisionStart", (event) => {
    event.pairs.forEach((pair) => {
      const ballA = pair.bodyA;
      const ballB = pair.bodyB;
      // 這段用來關閉合成
      // const ENABLE_MERGE = false;
      // 只有兩顆都有 level，且 level 相同時才合成　 標記正在合成中
      if (
        ballA.level &&
        ballB.level &&
        ballA.level === ballB.level &&
        !ballA.isMerging &&
        !ballB.isMerging
      ) {
        // 這段用來關閉合成
        // if (!ENABLE_MERGE) return;
        ballA.isMerging = true;
        ballB.isMerging = true;

        // 新球出現在兩顆球的中間
        const newX = (ballA.position.x + ballB.position.x) / 2;
        const newY = (ballA.position.y + ballB.position.y) / 2;
        addMergeEffect(newX, newY);

        // 新球等級 +1
        const newLevel = ballA.level + 1;

        // 超過最大等級就不再合成
        if (newLevel > MAX_LEVEL) return;

        // 合成成功加分
        emit("update-score", props.score + newLevel * 10);

        // 移除原本兩顆球
        Composite.remove(engine.world, ballA);
        Composite.remove(engine.world, ballB);

        // 建立合成後的新球
        const newBall = Bodies.circle(newX, newY, getBallRadius(newLevel), {
          level: newLevel,
          restitution: 0.25,
          friction: 0.5,
          frictionStatic: 0.5,
          render: getBallRender(newLevel),
        });

        // 加入新球
        Composite.add(engine.world, newBall);
      }
    });
  });
});

/* =========================
   點擊重整頁面
   
   ========================= */

function restartGame() {
  console.log("restart clicked");
  window.location.reload();
}

defineExpose({
  restartGame,
});

// 判斷螢幕大小

function isMobile() {
  return window.innerWidth <= 768;
}
const isAiming = ref(false);

function getBoardX(event) {
  const rect = gameBoard.value.getBoundingClientRect();
  const scaleX = BOARD_WIDTH / rect.width;

  return (event.clientX - rect.left) * scaleX;
}

function startAim(event) {
  if (!isMobile()) return;
  if (isGameOver.value) return;

  isAiming.value = true;
  previewX.value = getBoardX(event);
}

function moveAim(event) {
  if (!isMobile()) return;
  if (!isAiming.value) return;

  previewX.value = getBoardX(event);
}

function dropBall(event) {
  if (!isMobile()) return;
  if (!isAiming.value) return;

  isAiming.value = false;
  previewX.value = getBoardX(event);

  addBallAtX(previewX.value);
}

/* =========================
   點擊新增球
   ========================= */

function addBall(event) {
  // 手機版不要用 click 掉球，避免點擊和放手重複觸發
  if (isMobile()) return;

  const x = getBoardX(event);
  addBallAtX(x);
}

function addBallAtX(x) {
  if (isGameOver.value) return;
  if (!canDropBall) return;

  canDropBall = false;
  setTimeout(() => {
    canDropBall = true;
  }, DROP_COOLDOWN);

  const { Bodies, Composite } = Matter;

  const level = currentLevel.value;

  const ball = Bodies.circle(x, 40, getBallRadius(level), {
    restitution: 0.25,
    friction: 0.5,
    frictionStatic: 0.5,
    level,
    render: getBallRender(level),
  });

  ball.createdAt = Date.now();

  Composite.add(engine.world, ball);

  currentLevel.value = nextLevel.value;
  nextLevel.value = getRandomLevel();
  emit("update-next-level", nextLevel.value);
}
</script>
<template>
  <!-- <div class="score-box">分數：{{ score }}</div> -->
  <div class="game-layout">
    <div class="game-wrapper">
      <div
        ref="gameBoard"
        class="game-board"
        @mousemove="movePreview"
        @click="addBall"
        @pointerdown="startAim"
        @pointermove="moveAim"
        @pointerup="dropBall"
      >
        <div class="game-over-line"></div>
        <div
          class="aim-line"
          :style="{
            left: `${previewX}px`,
          }"
        ></div>
        <img
          v-if="getBallImage(currentLevel)"
          class="preview-ball"
          :src="getBallImage(currentLevel)"
          :style="{
            left: `${previewX}px`,
            width: `${BALL_SIZES[currentLevel]}px`,
            height: `${BALL_SIZES[currentLevel]}px`,
          }"
        />

        <div
          v-else
          class="preview-ball"
          :style="{
            left: `${previewX}px`,
            backgroundColor: BALL_COLORS[currentLevel],
            width: `${BALL_SIZES[currentLevel]}px`,
            height: `${BALL_SIZES[currentLevel]}px`,
          }"
        ></div>
        <div
          v-for="effect in mergeEffects"
          :key="effect.id"
          class="merge-effect"
          :style="{
            left: `${effect.x}px`,
            top: `${effect.y}px`,
          }"
        >
          POP!
        </div>
        <div v-if="isGameOver" class="game-over-mask">
          <div class="game-over-panel">
            <h2>GAME OVER</h2>
            <p>分數：{{ score }}</p>

            <button type="button" class="restart-btn" @click.stop="restartGame">
              再玩一次
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-board {
  position: relative;
  width: 420px;
  height: 640px;
  box-sizing: content-box;
  border: 4px solid #333;
  border-radius: 16px;
  overflow: hidden;
  background: #ffffff;
  touch-action: none;
}

.game-layout {
  position: relative;
  width: fit-content;
  margin: 0 auto;
}

.game-wrapper {
  position: relative;
}

.restart {
  cursor: pointer;
}

.game-over-line {
  position: absolute;
  width: 100%;
  top: 140px;
  left: 0;
  height: 0;
  border-top: 3px dashed red;
  z-index: 100;
  pointer-events: none;
}

.preview-ball {
  position: absolute;
  transform: translateX(-50%);
  pointer-events: none;
  opacity: 0.8;
  object-fit: contain;
  top:30px;
}

.aim-line {
  position: absolute;
  top: 84px;
  bottom: 0;
  width: 2px;
  background: aliceblue;
  transform: translateX(-50%);
  pointer-events: none;
  z-index: 10;
}

.side-panel {
  width: 120px;
}

.game-over-mask {
  position: absolute;
  inset: 0;

  background: rgba(0, 0, 0, 0.5);

  display: flex;
  justify-content: center;
  align-items: center;

  z-index: 999;
}

.game-over-panel {
  background: white;
  padding: 24px;
  border-radius: 20px;
  text-align: center;
  min-width: 220px;
}

.game-over-panel h2 {
  margin-bottom: 12px;
}

.game-over-panel p {
  margin-bottom: 16px;
}

.restart-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 999px;
  cursor: pointer;
  background: #ffb347;
  font-size: 16px;
}

/* 
/* 🎯 關鍵修正：強制讓 Matter.js 產生的 canvas 填滿 600x800 的格子 */
/* .game-board :deep(canvas) {
  width: 100% !important;
  height: 100% !important;
  display: block;
}  */

/* 合成特效 */
.merge-effect {
  position: absolute;
  transform: translate(-50%, -50%);
  pointer-events: none;
  z-index: 200;

  width: 64px;
  height: 64px;
  border-radius: 50%;

  display: flex;
  align-items: center;
  justify-content: center;

  background: rgba(255, 211, 120, 0.85);
  border: 3px solid #fff;
  color: #7a4b22;
  font-size: 14px;
  font-weight: 900;

  animation: merge-pop 0.4s ease-out forwards;
}

@keyframes merge-pop {
  0% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0.4);
  }

  40% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1.15);
  }

  100% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(1.6);
  }
}
/* rwd below */

@media (max-width: 576px) {
  .game-layout {
    width: 360px;
    height: 480px; /* 640 * 0.75 */
    margin: 0 auto;
  }

  .game-board {
    width: 480px;
    height: 640px;
    transform: scale(0.75);
    transform-origin: top left;
  }

  .game-board :deep(canvas) {
    width: 480px !important;
    height: 640px !important;
  }
}

@media (max-width: 390px) {
  .game-layout {
    width: 340px;
    height: 453px; /* 640 * 0.708 */
  }

  .game-board {
    width: 480px;
    height: 640px;
    transform: scale(0.708);
    transform-origin: top left;
  }

  .game-board :deep(canvas) {
    width: 480px !important;
    height: 640px !important;
  }
}
</style>
