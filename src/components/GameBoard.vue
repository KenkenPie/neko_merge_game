<script setup>
import { onMounted, ref } from 'vue'
import Matter from 'matter-js'

const BOARD_WIDTH = 600
const BOARD_HEIGHT = 800
const BALL_SIZE = 40

const gameBoard = ref(null)

let engine
let render
let runner

onMounted(() => {
    const { Engine, Render, Runner, Bodies, Composite } = Matter

    engine = Engine.create()

    render = Render.create({
        element: gameBoard.value,
        engine: engine,
        options: {
            width: BOARD_WIDTH,
            height: BOARD_HEIGHT,
            wireframes: false,
            background: '#ffffff',
        },
    })


    
    const ground = Bodies.rectangle(
        BOARD_WIDTH / 2,
        BOARD_HEIGHT + 25,
        BOARD_WIDTH,
        50,
        {
            isStatic: true,
            render: {
                fillStyle: '#333',
            },
        }
    )

    const leftWall = Bodies.rectangle(
  -25,
  BOARD_HEIGHT / 2,
  50,
  BOARD_HEIGHT,
  {
    isStatic: true,
    render: {
      fillStyle: '#333',
    },
  }
)

const rightWall = Bodies.rectangle(
  BOARD_WIDTH + 25,
  BOARD_HEIGHT / 2,
  50,
  BOARD_HEIGHT,
  {
    isStatic: true,
    render: {
      fillStyle: '#333',
    },
  }
)

    Composite.add(engine.world, [ground])

    Render.run(render)

    runner = Runner.create()
    Runner.run(runner, engine)
})

function addBall(event) {
    const { Bodies, Composite } = Matter

    const rect = gameBoard.value.getBoundingClientRect()
    const x = event.clientX - rect.left

    const ball = Bodies.circle(x, 40, BALL_SIZE / 2, {
        restitution: 0.2,
        friction: 0.05,
        render: {
            fillStyle: 'orange',
        },
    })

    Composite.add(engine.world, ball)
}
</script>

<template>
    <button>清除</button>

    <div ref="gameBoard" class="game-board" @click="addBall"></div>
</template>

<style scoped>
.game-board {
    width: 600px;
    height: 800px;
    border: 4px solid #333;
    border-radius: 16px;
    overflow: hidden;
}
</style>