<script setup>
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
   基本狀態
   ========================= */

let canDropBall = true;

const DROP_COOLDOWN = 300;

const mergeEffects = ref([]);
const currentLevel = ref(1);
const nextLevel = ref(1);
const isGameOver = ref(false);
const isAiming = ref(false);

const props = defineProps({
  score: {
    type: Number,
    default: 0,
  },
});

const emit = defineEmits([
  "update-score",
  "update-next-level",
]);

/* =========================
   遊戲設定
   ========================= */

const GAME_OVER_LINE = 140;

const MAX_LEVEL = 12;

const BOARD_WIDTH = 420;
const BOARD_HEIGHT = 640;

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
  11: lv11Img,
  12: lv12Img,
};

/* =========================
   Matter.js 變數
   ========================= */

const gameBoard = ref(null);

let engine;
let render;
let runner;

/* =========================
   球的設定
   ========================= */

// 圖片比碰撞體稍微大一點，減少視覺縫隙
function getBallRadius(level) {
  return BALL_SIZES[level] * 0.46;
}

// 等級越高，密度越高
function getBallDensity(level) {
  return 0.001 + level * 0.001;
}

function getBallRender(level) {
  return {
    sprite: {
      texture: BALL_IMAGES[level],
      xScale: 1,
      yScale: 1,
    },
  };
}

function getBallImage(level) {
  return BALL_IMAGES[level];
}

// 隨機產生 LV1～LV4
function getRandomLevel() {
  return Math.floor(Math.random() * 4) + 1;
}

/* =========================
   Preview 座標
   ========================= */

const previewX = ref(BOARD_WIDTH / 2);

/*
  使用圖片寬度的一半限制位置。

  原因：
  碰撞圓半徑只有尺寸的 0.46，
  但圖片寬度仍接近完整 BALL_SIZES。

  目的：
  讓畫面上的圖片也不會超出左右牆壁。
*/
function clampPreviewX(x, level = currentLevel.value) {
  const visualRadius = BALL_SIZES[level] / 2;

  return Math.max(
    visualRadius,
    Math.min(BOARD_WIDTH - visualRadius, x),
  );
}

/*
  取得 Matter canvas 的實際座標。

  手機版 game-board 有 scale，
  所以不能直接使用 event.clientX - rect.left。
*/
function getBoardX(event) {
  const canvas =
    gameBoard.value?.querySelector("canvas");

  const targetElement =
    canvas || gameBoard.value;

  const rect =
    targetElement.getBoundingClientRect();

  const scaleX =
    BOARD_WIDTH / rect.width;

  return (
    (event.clientX - rect.left) * scaleX
  );
}

function movePreview(event) {
  if (isMobile()) return;

  previewX.value = clampPreviewX(
    getBoardX(event),
  );
}

/* =========================
   手機瞄準
   ========================= */

function isMobile() {
  return window.innerWidth <= 768;
}

function startAim(event) {
  if (!isMobile()) return;
  if (isGameOver.value) return;

  isAiming.value = true;

  previewX.value = clampPreviewX(
    getBoardX(event),
  );

  /*
    手指移出遊戲框時，
    仍然可以接收到 pointerup。
  */
  event.currentTarget?.setPointerCapture?.(
    event.pointerId,
  );
}

function moveAim(event) {
  if (!isMobile()) return;
  if (!isAiming.value) return;

  previewX.value = clampPreviewX(
    getBoardX(event),
  );
}

function dropBall(event) {
  if (!isMobile()) return;
  if (!isAiming.value) return;

  isAiming.value = false;

  previewX.value = clampPreviewX(
    getBoardX(event),
  );

  event.currentTarget?.releasePointerCapture?.(
    event.pointerId,
  );

  addBallAtX(previewX.value);
}

function cancelAim() {
  isAiming.value = false;
}

/* =========================
   合成特效
   ========================= */

function addMergeEffect(x, y) {
  const id = Date.now() + Math.random();

  mergeEffects.value.push({
    id,
    x,
    y,
  });

  setTimeout(() => {
    mergeEffects.value =
      mergeEffects.value.filter(
        (item) => item.id !== id,
      );
  }, 220);
}

/* =========================
   初始化遊戲
   ========================= */

onMounted(() => {
  currentLevel.value = getRandomLevel();
  nextLevel.value = getRandomLevel();

  const {
    Engine,
    Render,
    Runner,
    Bodies,
    Composite,
    Events,
  } = Matter;

  engine = Engine.create();

  /*
    增加碰撞計算精度，
    減少球堆疊時的抖動。
  */
  engine.positionIterations = 12;
  engine.velocityIterations = 10;

  render = Render.create({
    element: gameBoard.value,
    engine,
    options: {
      width: BOARD_WIDTH,
      height: BOARD_HEIGHT,
      wireframes: false,
      background: "#ffffff",
    },
  });

  /* =========================
     地板與牆壁
     ========================= */

  const ground = Bodies.rectangle(
    BOARD_WIDTH / 2,
    BOARD_HEIGHT + 10,
    BOARD_WIDTH,
    20,
    {
      isStatic: true,

      friction: 1,
      frictionStatic: 1,
      restitution: 0,

      render: {
        visible: false,
      },
    },
  );

  const leftWall = Bodies.rectangle(
    -25,
    BOARD_HEIGHT / 2,
    50,
    BOARD_HEIGHT,
    {
      isStatic: true,

      /*
        左右牆壁摩擦設為 0，
        避免新球貼牆時慢慢掉落。
      */
      friction: 0,
      frictionStatic: 0,
      restitution: 0,

      render: {
        visible: false,
      },
    },
  );

  const rightWall = Bodies.rectangle(
    BOARD_WIDTH + 25,
    BOARD_HEIGHT / 2,
    50,
    BOARD_HEIGHT,
    {
      isStatic: true,

      friction: 0,
      frictionStatic: 0,
      restitution: 0,

      render: {
        visible: false,
      },
    },
  );

  Composite.add(engine.world, [
    ground,
    leftWall,
    rightWall,
  ]);

  Render.run(render);

  runner = Runner.create();
  Runner.run(runner, engine);

  /* =========================
     減少殘留滑動
     ========================= */

  Events.on(engine, "beforeUpdate", () => {
    Composite.allBodies(
      engine.world,
    ).forEach((body) => {
      if (!body.level) return;

      /*
        如果較輕的球正在推動重球，
        清除重球的水平速度。

        保留 y 速度，
        所以重球仍然可以正常往下掉。
      */
      if (body.isPushedByLighter) {
        Matter.Body.setVelocity(body, {
          x: 0,
          y: body.velocity.y,
        });

        Matter.Body.setAngularVelocity(
          body,
          0,
        );

        body.isPushedByLighter = false;

        return;
      }

      /*
        只清除非常小的水平速度，
        避免已經停下的球一直慢慢滑。
      */
      if (
        Math.abs(body.velocity.x) < 0.015
      ) {
        Matter.Body.setVelocity(body, {
          x: 0,
          y: body.velocity.y,
        });
      }

      if (
        Math.abs(body.angularVelocity) <
        0.02
      ) {
        Matter.Body.setAngularVelocity(
          body,
          0,
        );
      }
    });
  });

  /* =========================
     Game Over 判斷
     ========================= */

  Events.on(engine, "afterUpdate", () => {
    if (isGameOver.value) return;

    const balls =
      engine.world.bodies.filter(
        (body) => body.level,
      );

    const now = Date.now();

    for (const ball of balls) {
      if (
        ball.createdAt &&
        now - ball.createdAt > 1500 &&
        ball.speed < 0.5 &&
        ball.position.y -
          ball.circleRadius <
          GAME_OVER_LINE
      ) {
        isGameOver.value = true;

        console.log("GAME OVER");

        break;
      }
    }
  });

  /* =========================
     相同等級合成
     ========================= */

  Events.on(
    engine,
    "collisionStart",
    (event) => {
      event.pairs.forEach((pair) => {
        const ballA = pair.bodyA;
        const ballB = pair.bodyB;

        // 牆壁與地板沒有 level
        if (
          !ballA.level ||
          !ballB.level
        ) {
          return;
        }

        // 只有相同等級才合成
        if (
          ballA.level !== ballB.level
        ) {
          return;
        }

        // 避免同一顆球重複合成
        if (
          ballA.isMerging ||
          ballB.isMerging
        ) {
          return;
        }

        ballA.isMerging = true;
        ballB.isMerging = true;

        const newLevel =
          ballA.level + 1;

        /*
          LV12 不再繼續往上合成。
        */
        if (newLevel > MAX_LEVEL) {
          ballA.isMerging = false;
          ballB.isMerging = false;

          return;
        }

        const newX =
          (ballA.position.x +
            ballB.position.x) /
          2;

        const newY =
          (ballA.position.y +
            ballB.position.y) /
          2;

        addMergeEffect(newX, newY);

        emit(
          "update-score",
          props.score + newLevel * 10,
        );

        Composite.remove(
          engine.world,
          ballA,
        );

        Composite.remove(
          engine.world,
          ballB,
        );

        const newBall =
          Bodies.circle(
            newX,
            newY,
            getBallRadius(newLevel),
            {
              level: newLevel,

              restitution: 0.05,
              friction: 0.12,
              frictionStatic: 0.8,
              frictionAir: 0.015,

              slop: 0.04,

              density:
                getBallDensity(
                  newLevel,
                ),

              inertia: Infinity,

              render:
                getBallRender(
                  newLevel,
                ),
            },
          );

        /*
          合成球也要有建立時間，
          Game Over 才能正常判斷。
        */
        newBall.createdAt = Date.now();

        Composite.add(
          engine.world,
          newBall,
        );
      });
    },
  );

  /* =========================
     輕球推動重球限制
     ========================= */

  /*
    注意：
    collisionActive 一定要放在
    collisionStart 外面。

    這樣整場遊戲只會註冊一次。
  */
  Events.on(
    engine,
    "collisionActive",
    (event) => {
      event.pairs.forEach((pair) => {
        const ballA = pair.bodyA;
        const ballB = pair.bodyB;

        // 排除牆壁和地板
        if (
          !ballA.level ||
          !ballB.level
        ) {
          return;
        }

        // 相同等級要正常合成
        if (
          ballA.level === ballB.level
        ) {
          return;
        }

        const heavyBall =
          ballA.level > ballB.level
            ? ballA
            : ballB;

        /*
          重球還在空中時不要鎖住，
          否則會影響正常掉落。
        */
        const heavyBallIsSettled =
          Math.abs(
            heavyBall.velocity.y,
          ) < 0.4;

        if (!heavyBallIsSettled) return;

        heavyBall.isPushedByLighter =
          true;
      });
    },
  );
});

/* =========================
   新增球
   ========================= */

function addBall(event) {
  /*
    手機使用 pointerup，
    不再使用 click 掉球。
  */
  if (isMobile()) return;

  const x = getBoardX(event);

  addBallAtX(x);
}

function addBallAtX(x) {
  if (isGameOver.value) return;
  if (!canDropBall) return;

  const level = currentLevel.value;

  const safeX = clampPreviewX(
    x,
    level,
  );

  canDropBall = false;

  setTimeout(() => {
    canDropBall = true;
  }, DROP_COOLDOWN);

  const {
    Bodies,
    Composite,
  } = Matter;

  const ball = Bodies.circle(
    safeX,
    40,
    getBallRadius(level),
    {
      level,

      restitution: 0.05,
      friction: 0.12,
      frictionStatic: 0.8,
      frictionAir: 0.015,

      slop: 0.04,

      density:
        getBallDensity(level),

      inertia: Infinity,

      render:
        getBallRender(level),
    },
  );

  ball.createdAt = Date.now();

  Composite.add(
    engine.world,
    ball,
  );

  currentLevel.value =
    nextLevel.value;

  nextLevel.value =
    getRandomLevel();

  emit(
    "update-next-level",
    nextLevel.value,
  );
}

/* =========================
   重新開始
   ========================= */

function restartGame() {
  window.location.reload();
}

defineExpose({
  restartGame,
});
</script>

<template>
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
        @pointercancel="cancelAim"
      >
        <div class="game-over-line"></div>

        <div
          class="aim-line"
          :style="{
            left: `${previewX}px`,
          }"
        ></div>

        <!--
          不設定固定 height，
          避免非正方形 PNG 被壓扁或拉長。
        -->
        <img
          class="preview-ball"
          :src="getBallImage(currentLevel)"
          :style="{
            left: `${previewX}px`,
            width: `${BALL_SIZES[currentLevel]}px`,
          }"
          alt=""
          draggable="false"
        />

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

        <div
          v-if="isGameOver"
          class="game-over-mask"
        >
          <div class="game-over-panel">
            <h2>GAME OVER</h2>

            <p>分數：{{ score }}</p>

            <button
              type="button"
              class="restart-btn"
              @click.stop="restartGame"
            >
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

  background:#f6f2EC;

  touch-action:none;
}
np
.game-layout {
  position: relative;

  width: fit-content;

  margin: 0 auto;
}

.game-wrapper {
  position: relative;
}

/* =========================
   死亡線
   ========================= */

.game-over-line {
  position: absolute;

  top: 140px;
  left: 0;

  width: 100%;
  height: 0;

  border-top: 3px dashed red;

  z-index: 100;

  pointer-events: none;
}

/* =========================
   預覽球
   ========================= */

.preview-ball {
  position: absolute;

  top: 30px;

  height: auto;

  display: block;

  transform: translateX(-50%);

  object-fit: contain;

  opacity: 0.8;

  pointer-events: none;

  user-select: none;
}

/* =========================
   瞄準線
   ========================= */

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

/* =========================
   Game Over
   ========================= */

.game-over-mask {
  position: absolute;

  inset: 0;

  display: flex;

  justify-content: center;
  align-items: center;

  background: rgba(0, 0, 0, 0.5);

  z-index: 999;
}

.game-over-panel {
  min-width: 220px;

  padding: 24px;

  border-radius: 20px;

  background: white;

  text-align: center;
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

  background: #ffb347;

  font-size: 16px;

  cursor: pointer;
}

/* =========================
   合成特效
   ========================= */

.merge-effect {
  position: absolute;

  width: 50px;
  height: 50px;

  display: flex;

  align-items: center;
  justify-content: center;

  transform: translate(-50%, -50%);

  color: #7a4b22;

  font-size: 14px;
  font-weight: 900;

  /*
    改成透明，
    避免蓋住剛生成的 newBall。
  */
  background: transparent;

  pointer-events: none;

  z-index: 200;

  animation:
    merge-pop 0.22s ease-out
    forwards;
}

@keyframes merge-pop {
  0% {
    opacity: 0;

    transform:
      translate(-50%, -50%)
      scale(0.4);
  }

  40% {
    opacity: 1;

    transform:
      translate(-50%, -50%)
      scale(1.15);
  }

  100% {
    opacity: 0;

    transform:
      translate(-50%, -50%)
      scale(1.6);
  }
}

/* =========================
   手機版
   ========================= */

@media (max-width: 576px) {
  .game-layout {
    width: 315px;
    height: 480px;

    margin: 0 auto;
  }

  .game-board {
    width: 420px;
    height: 640px;

    transform: scale(0.75);
    transform-origin: top left;
  }

  .game-board :deep(canvas) {
    width: 420px !important;
    height: 640px !important;

    display: block;
  }
}

@media (max-width: 390px) {
  .game-layout {
    width: 297px;
    height: 453px;
  }

  .game-board {
    width: 420px;
    height: 640px;

    transform: scale(0.708);
    transform-origin: top left;
  }

  .game-board :deep(canvas) {
    width: 420px !important;
    height: 640px !important;

    display: block;
  }
}
</style>