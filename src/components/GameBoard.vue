<script setup>
import { onMounted, ref } from "vue";
import Matter from "matter-js";

const currentLevel = ref(1);
const nextLevel = ref(1);
const BOARD_WIDTH = 480;
const BOARD_HEIGHT = 640;
const BALL_SIZE = 40;
const BALL_COLORS = {
  1: "orange",
  2: "red",
  3: "blue",
  4: "green",
};

const previewX = ref(BOARD_WIDTH / 2);

function movePreview(event) {
  const rect = gameBoard.value.getBoundingClientRect();
  previewX.value = event.clientX - rect.left;
}

function getBallRadius(level) {
  return BALL_SIZE / 2 + (level - 1) * 10;
}
const gameBoard = ref(null);

function getRandomLevel() {
  return Math.floor(Math.random() * 3) + 1;
}

let engine;
let render;
let runner;

onMounted(() => {
  currentLevel.value = getRandomLevel();
  nextLevel.value = getRandomLevel();
  const { Engine, Render, Runner, Bodies, Composite, Events } = Matter;

  engine = Engine.create();

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

  Composite.add(engine.world, [ground, leftWall, rightWall]);

  Render.run(render);

  runner = Runner.create();
  Runner.run(runner, engine);
  Events.on(engine, "collisionStart", (event) => {
    event.pairs.forEach((pair) => {
      const ballA = pair.bodyA;
      const ballB = pair.bodyB;

      if (ballA.level && ballB.level && ballA.level === ballB.level) {
        const newX = (ballA.position.x + ballB.position.x) / 2;
        const newY = (ballA.position.y + ballB.position.y) / 2;
        const newLevel = ballA.level + 1;

        Composite.remove(engine.world, ballA);
        Composite.remove(engine.world, ballB);

        const newBall = Bodies.circle(newX, newY, getBallRadius(newLevel), {
          level: newLevel,
          restitution: 0.2,
          friction: 0.05,
          render: {
            fillStyle: BALL_COLORS[newLevel],
          },
        });

        Composite.add(engine.world, newBall);
      }
    });
  });
});

function addBall(event) {
  const { Bodies, Composite } = Matter;

  const rect = gameBoard.value.getBoundingClientRect();
  const x = event.clientX - rect.left;

  const level = currentLevel.value;

  const ball = Bodies.circle(x, 40, getBallRadius(level), {
    restitution: 0.2,
    friction: 0.05,
    level,
    render: {
      fillStyle: BALL_COLORS[level],
    },
  });

  Composite.add(engine.world, ball);
  currentLevel.value = nextLevel.value;
  nextLevel.value = getRandomLevel();
}
</script>
<template>
  <button>清除</button>
  <div class="game-layout">
    <div class="game-wrapper">
      <div
        ref="gameBoard"
        class="game-board"
        @mousemove="movePreview"
        @click="addBall"
      >
        <div
          class="preview-ball"
          :style="{
            left: `${previewX}px`,
            backgroundColor: BALL_COLORS[currentLevel],
            width: `${getBallRadius(currentLevel) * 2}px`,
            height: `${getBallRadius(currentLevel) * 2}px`,
          }"
        ></div>
      </div>
    </div>

    <div class="next-box">
      <p>下一顆</p>
      <div>LV{{ nextLevel }}</div>
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
.preview-ball {
  position: absolute;
  border-radius: 50%;
  transform: translateX(-50%);
  pointer-events: none;
  opacity: 0.7;
}

.next-box {
  width: 100px;
  height: 100px;
  border: 3px solid #333;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.side-panel {
  width: 120px;
}
.next-box {
  width: 100px;
  height: 100px;
  border: 3px solid #333;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
/* 
/* 🎯 關鍵修正：強制讓 Matter.js 產生的 canvas 填滿 600x800 的格子 */
/* .game-board :deep(canvas) {
  width: 100% !important;
  height: 100% !important;
  display: block;
}  */
</style>
