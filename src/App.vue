<script setup>
import {onMounted, onUnmounted, reactive, ref} from 'vue'

const score = ref(0);
const playing = ref(false);
const gameOver = ref(false);

const paddleWidth = ref(10);
const paddleX = ref(50);
const paddleSpeed = ref(50);
const ballX = ref(50);
const ballY = ref(50);
const ballDirection = ref(1);
const ballYVelocity = ref(25);
const ballXVelocity = ref(10);

const keysPressed = reactive({});

let lastTimestamp = 0;

const rows = 8
const bricks = ref(
    null // 42 briques
);
const acceleration = 1.01;

function incrementScore() {
  score.value += 100;
}

function start() {
  gameOver.value = false;
  playing.value = true;
  score.value = 0;
  paddleX.value = 50;
  ballX.value = 50;
  ballY.value = 50;
  ballXVelocity.value = 10 + Math.random() * 10;
  ballYVelocity.value = 25;
  bricks.value = Array(14 * rows).fill(null).map(() => (
      {visible: true}
  ));

  lastTimestamp = performance.now();
  animationFrameId = requestAnimationFrame(update);
}

function win(){

  if (paddleWidth.value === 2){
    setGameOver();
  }

  ballYVelocity.value = 25;
  paddleX.value = 50;
  ballX.value = 50;
  ballY.value = 50;
  ballXVelocity.value = 10 + Math.random() * 10;
  bricks.value = Array(14 * rows).fill(null).map(() => (
      {visible: true}
  ));
  paddleWidth.value -= 2;
}

function setGameOver() {
  gameOver.value = true;
  playing.value = false;
  cancelAnimationFrame(animationFrameId);
}

function quit() {
  playing.value = false;
  gameOver.value = false;
  cancelAnimationFrame(animationFrameId);
}

function handleKeyDown(event) {
  keysPressed[event.key] = true;
}

function handleKeyUp(event) {
  keysPressed[event.key] = false;
}

function detectPaddleCollision(x, y, width) {
  return ((x - width / 2 <= paddleX.value + paddleWidth.value / 2 && x + width / 2 >= paddleX.value - paddleWidth.value / 2) && (y <= 90 && y >= 89))
}

function detectBrickCollision() {
  bricks.value.forEach((brick, index) => {
    if (brick.visible) {
      const brickLeft = (index % 14) * (100 / 14);
      const brickTop = Math.floor(index / 14) * (20 / 8);

      if (
          ballX.value >= brickLeft &&
          ballX.value <= brickLeft + 100 / 14 &&
          ballY.value >= brickTop &&
          ballY.value <= brickTop + 20 / 8
      ) {
        brick.visible = false;
        ballDirection.value = 1;
        ballXVelocity.value *= acceleration;
        ballYVelocity.value *= acceleration;
        brickCollision();

        // Si la balle touche le côté gauche d'une brique
        if (
            (ballX.value >= brickLeft &&
                ballX.value <= brickLeft + 2) &&
            (ballY.value >= brickTop - 10 &&
                ballY.value <= brickTop + 20 / 8 + 10)
        ) {
          ballXVelocity.value = Math.abs(ballXVelocity.value) * -1;
        }

        // Si la balle touche le côté droit d'une brique
        else if (
            (ballX.value >= brickLeft + 100 / 14 &&
                ballX.value <= brickLeft + 100 / 14 - 2) &&
            (ballY.value >= brickTop - 10 &&
                ballY.value <= brickTop + 20 / 8 + 10)
        ) {
          ballXVelocity.value = Math.abs(ballXVelocity.value);
        }
      }
    }
  });
}

function brickCollision() {
  incrementScore();
  if (!bricks.value.reduce((a, brick) => a = a || brick.visible, false)) {
    win();
  }
}

function checkPressedKeys(deltaTime){
  if (keysPressed['ArrowLeft']) {
    paddleX.value = Math.max(paddleWidth.value / 2, paddleX.value - paddleSpeed.value * deltaTime);
  }
  if (keysPressed['ArrowRight']) {
    paddleX.value = Math.min(100 - paddleWidth.value / 2, paddleX.value + paddleSpeed.value * deltaTime);
  }
}

function moveBall(deltaTime){
  ballY.value += ballYVelocity.value * deltaTime * ballDirection.value;
  ballX.value += ballXVelocity.value * deltaTime;
}

function checkBallAgainstWall(){
  if (ballX.value <= 0) {
    ballXVelocity.value = Math.abs(ballXVelocity.value) * acceleration;
  }

  if (ballX.value >= 99) {
    ballXVelocity.value = Math.abs(ballXVelocity.value) * -acceleration;
  }

  if (ballY.value >= 100) {
    setGameOver();
  }
}

function update(timestamp) {

  if (!playing.value) {
    return;
  }

  const deltaTime = (timestamp - lastTimestamp) / 1000;
  lastTimestamp = timestamp;

  checkPressedKeys(deltaTime);

  moveBall(deltaTime);

  checkBallAgainstWall();

  if (detectPaddleCollision(ballX.value, ballY.value, 0)) {
    ballDirection.value = -1;
    ballYVelocity.value *= acceleration;
  }

  detectBrickCollision();

  if (ballY.value <= 0) {
    ballDirection.value = 1;
    ballYVelocity.value *= acceleration;
  }

  animationFrameId = requestAnimationFrame(update);

}

let animationFrameId;

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
  window.addEventListener('keyup', handleKeyUp);

  if (playing.value) {
    lastTimestamp = performance.now();
    animationFrameId = requestAnimationFrame(update);
  }
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
  window.removeEventListener('keyup', handleKeyUp);

  cancelAnimationFrame(animationFrameId);
});

function getBrickColor(i) {
  const row = Math.floor(i / 14); // Ligne de la brique
  switch (Math.trunc(row/2)) {
    case 0:
      return 'red';
    case 1:
      return 'orange';
    case 2:
      return 'yellow';
    case 3:
      return 'green';
    default:
      return;
  }
}

</script>
<template>
  <div v-if="playing" class="main">
    <p class="score">Score: {{ score }}</p>
    <div class="game">
      <div class="bricks">
        <div
            v-for="(brick, index) in bricks"
            :key="index"
            class="brick"
            :class="getBrickColor(index)"
            :style="{ visibility: brick.visible ? 'visible' : 'hidden' }">
        </div>
      </div>
      <div class="ball" :style="{left: ballX + '%', top: ballY + '%'}"></div>
      <div class="paddle" :style="{width: paddleWidth + '%', left: paddleX-(paddleWidth/2) + '%'}"></div>
    </div>
  </div>
  <div v-else-if="gameOver" class="main">
    <h1>GAME OVER</h1>
    <p class="final-score">FINAL SCORE : {{ score }}</p>
    <button @click="start" class="main-button">Replay</button>
    <button @click="quit" class="main-button">Quit</button>
  </div>
  <div v-else class="main">
    <h1>BREAKOUT</h1>
    <button @click="start" class="main-button">Play</button>
  </div>
</template>

<style>

* {
  font-family: 'Press Start 2P', cursive;
}

body {
  color: white;
  background-color: black;
  overflow: hidden;
}

.game {
  width: 70%;
  display: block;
  margin: auto;
  position: relative;
  height: 90vh;
  border: 5px solid white;
}

h1 {
  text-align: center;
  font-size: 4rem;
}

.main-button {
  text-align: center;
  display: block;
  margin: 0 auto;
  font-size: 2rem;
  padding: 1rem;
  width: 20rem;
}

.score {
  font-size: 1.5rem;
  position: absolute;
  left: 0;
  top: 0;
}

.paddle {
  height: .5rem;
  background-color: white;
  top: 90%;
  position: absolute;
}

.ball {
  height: .5rem;
  width: .5rem;
  position: absolute;
  background-color: red;
}

.bricks {
  top: 0;
  width: 100%;
  height: 20%;
  display: flex;
  justify-content: space-evenly;
  flex-direction: row;
  flex-wrap: wrap;
}

.brick {
  width: calc(100% / 14);
  height: calc(100% / 8);
}

.red {
  background-color: #FF4A42;
}

.orange {
  background-color: #FF7331;
}

.green {
  background-color: #109400;
}

.yellow {
  background-color: #E7C600;
}

.final-score {
  text-align: center;
  font-size: 3rem;
}


</style>