<script setup>
import { ref } from 'vue'

const BOARD_WIDTH = 600
const BOARD_HEIGHT = 800
const BALL_SIZE = 40

const balls = ref([])
const mouseX = ref((BOARD_WIDTH - BALL_SIZE) / 2)

function updateMouseX(event) {
    const board = event.currentTarget
    const rect = board.getBoundingClientRect()

    const x = event.clientX - rect.left - BALL_SIZE / 2

    mouseX.value = Math.max(0, Math.min(x, BOARD_WIDTH - BALL_SIZE))
}

function addBall() {
    const newBall = {
        id: Date.now(),
        x: mouseX.value,
        y: 0,
        size: BALL_SIZE,
        isDropping: true,
    }

    balls.value.push(newBall)
    startDrop()
}

let timer = null

function startDrop() {
    if (timer) return

    timer = setInterval(() => {
        balls.value.forEach((ball) => {
            if (!ball.isDropping) return

            const bottom = BOARD_HEIGHT - ball.size

            let canMove = true

            balls.value.forEach((otherBall) => {
                if (ball.id === otherBall.id) return

                const overlapX =
                    Math.abs(ball.x - otherBall.x) < ball.size

                const overlapY =
                    ball.y + ball.size >= otherBall.y

                if (
                    overlapX &&
                    overlapY &&
                    !otherBall.isDropping
                ) {
                    ball.y = otherBall.y - ball.size
                    canMove = false
                }
            })

            if (canMove && ball.y < bottom) {
                ball.y += 2
            } else {
                ball.isDropping = false
            }
        })
    }, 16)
}

function clearBalls() {
    balls.value = []
    clearInterval(timer)
    timer = null
}
</script>

<template>
    <button @click="clearBalls">清除</button>

    <div class="game-board" @mousemove="updateMouseX" @click="addBall">
        <div class="preview-ball" :style="{
            left: mouseX + 'px',
        }"></div>
        <div v-for="ball in balls" :key="ball.id" class="ball" :style="{
            width: ball.size + 'px',
            height: ball.size + 'px',
            left: ball.x + 'px',
            top: ball.y + 'px',
        }"></div>
    </div>
</template>

<style scoped>
.preview-ball {
    width: 40px;
    height: 40px;

    border-radius: 50%;

    background: rgba(255, 165, 0, 0.4);

    position: absolute;

    top: 0;
}

.game-board {
    width: 600px;
    height: 800px;
    border: 4px solid #333;
    border-radius: 16px;
    background: white;
    position: relative;
    overflow: hidden;
}

.ball {
    border-radius: 50%;
    background: orange;
    position: absolute;
}

button {
    width: 80px;
    height: 30px;
    margin: 5px;
}
</style>