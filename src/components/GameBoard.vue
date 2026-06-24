<script setup>
import { onMounted, ref } from "vue";
import Matter from "matter-js";
import lv1Img from "../img/gameIcon/lv1.png";
import lv2Img from "../img/gameIcon/lv2.png";
/* =========================
   Vue 狀態資料
   ========================= */

// 目前準備掉下來的球等級
const currentLevel = ref(1);

// 右邊預告：下一顆球的等級
const nextLevel = ref(1);

// 分數
const score = ref(0);

/* =========================
   遊戲基本設定
   ========================= */

//*死亡線
const isGameOver = ref(false);
const GAME_OVER_LINE = 120;

// 最大等級，超過 LV10 就不再合成
const MAX_LEVEL = 10;

// 遊戲區尺寸
const BOARD_WIDTH = 480;
const BOARD_HEIGHT = 640;

// 各等級球的直徑尺寸，對應 Figma 圖片尺寸
const BALL_SIZES = {
  1: 40,
  2: 52,
  3: 64,
  4: 76,
  5: 88,
  6: 100,
  7: 112,
  8: 124,
  9: 136,
  10: 148,
};

// 依照等級計算球半徑

function getBallRadius(level) {
  return BALL_SIZES[level] / 2;
}

// 測試lv1球改圖片
const BALL_IMAGES = {
  1: lv1Img,
  2: lv2Img,
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

      // 只有兩顆都有 level，且 level 相同時才合成　 標記正在合成中
      if (
        ballA.level &&
        ballB.level &&
        ballA.level === ballB.level &&
        !ballA.isMerging &&
        !ballB.isMerging
      ) {
        ballA.isMerging = true;
        ballB.isMerging = true;

        // 新球出現在兩顆球的中間
        const newX = (ballA.position.x + ballB.position.x) / 2;
        const newY = (ballA.position.y + ballB.position.y) / 2;

        // 新球等級 +1
        const newLevel = ballA.level + 1;

        // 超過最大等級就不再合成
        if (newLevel > MAX_LEVEL) return;

        // 合成成功加分
        score.value += newLevel * 10;

        // 移除原本兩顆球
        Composite.remove(engine.world, ballA);
        Composite.remove(engine.world, ballB);

        // 建立合成後的新球
        const newBall = Bodies.circle(newX, newY, getBallRadius(newLevel), {
          level: newLevel,
          restitution: 0.2,
          friction: 0.05,
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
/* =========================
   點擊新增球
   ========================= */

function addBall(event) {
  if (isGameOver.value) return;
  const { Bodies, Composite } = Matter;

  const rect = gameBoard.value.getBoundingClientRect();
  const x = event.clientX - rect.left;

  // 這次掉下來的球等級
  const level = currentLevel.value;

  // 建立球
  const ball = Bodies.circle(x, 40, getBallRadius(level), {
    restitution: 0.2,
    friction: 0.05,
    level,
    render: getBallRender(level),
  });
  ball.createdAt = Date.now();

  // 把球加入世界
  Composite.add(engine.world, ball);

  // 更新目前球與下一顆球
  currentLevel.value = nextLevel.value;
  nextLevel.value = getRandomLevel();
}
</script>
<template>
  <div class="score-box">分數：{{ score }}</div>
  <button class="restart" @click="restartGame">
    {{ isGameOver ? "重新開始" : "重置遊戲" }}
  </button>
  <div class="game-layout">
    <div class="game-wrapper">
      <div ref="gameBoard" class="game-board" @mousemove="movePreview" @click="addBall">
        <div class="game-over-line"></div>
        <div class="preview-ball" :style="{
          left: `${previewX}px`,
          backgroundColor: BALL_COLORS[currentLevel],
          width: `${getBallRadius(currentLevel) * 2}px`,
          height: `${getBallRadius(currentLevel) * 2}px`,
        }"></div>
      </div>
    </div>

    <div class="next-box">
      <p>Next:</p>

      <div class="next-ball" :style="{
        backgroundColor: BALL_COLORS[nextLevel],
        width: `${getBallRadius(nextLevel) * 2}px`,
        height: `${getBallRadius(nextLevel) * 2}px`,
      }"></div>
    </div>
  </div>
</template>

<style scoped>
.game-board {
  position: relative;
  width: 480px;
  height: 640px;
  box-sizing: content-box;
  border: 4px solid #333;
  border-radius: 16px;
  overflow: hidden;
  background: #ffffff;
}

.game-layout {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  gap: 24px;
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
  top: 120px;
  left: 0;
  height: 2px;
  background: red;
  z-index: 100;
  pointer-events: none;
}

.preview-ball {
  position: absolute;
  border-radius: 50%;
  transform: translateX(-50%);
  pointer-events: none;
  opacity: 0.7;
}

.side-panel {
  width: 120px;
}

.next-box {
  width: 120px;
  min-height: 120px;
  border: 1px solid #333;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px;
}

.next-ball {
  border-radius: 50%;
  margin-top: 12px;
}

/* 
/* 🎯 關鍵修正：強制讓 Matter.js 產生的 canvas 填滿 600x800 的格子 */
/* .game-board :deep(canvas) {
  width: 100% !important;
  height: 100% !important;
  display: block;
}  */
</style>
