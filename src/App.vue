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
const ballYDirection = ref(1);
const ballXDirection = ref(1);
const ballYVelocity = ref(25);
const ballXVelocity = ref(10);
const ballSize = ref(6);

const bonusVisible = ref(false);
const bonusX = ref(0);
const bonusY = ref(0);
const bonusSpeed = ref(50);
const bonusActive = ref(false);
const bonusTime = ref(0);
const bonusName = ref("");
const initBonusTime = 10;

const largePaddleBonusSize = 10;
const fastPaddleBonusSpeed = 30;
const slowBallBonusSpeed = -10;
const largeBallBonusSizeMultiplier = 4;

const keysPressed = reactive({});

let lastTimestamp = 0;

const rows = 8
const bricks = ref(
    null // 42 briques
);
const ballAcceleration = 1.01;

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

function win() {

  if (paddleWidth.value === 2) {
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
  return ((x <= paddleX.value + paddleWidth.value && x + width >= paddleX.value - paddleWidth.value / 2) && (y + width <= 90 && y + width >= 89))
}

function detectBrickCollision() {
  bricks.value.forEach((brick, index) => {
    if (brick.visible) {
      const brickLeft = Math.floor(index % 14) * (100 / 14);
      const brickRight = brickLeft + 100 / 14;
      const brickTop = Math.floor(index / 14) * (20 / 8);
      const brickBottom = brickTop + 20 / 8;

      const ballLeft = ballX.value;
      const ballRight = ballX.value + ballSize.value;
      const ballTop = ballY.value;
      const ballBottom = ballY.value + ballSize.value;

      if (
          ballRight >= brickLeft &&
          ballLeft <= brickRight &&
          ballBottom >= brickTop &&
          ballTop <= brickBottom
      ) {
        brick.visible = false;
        ballXVelocity.value *= ballAcceleration;
        ballYVelocity.value *= ballAcceleration;
        brickCollision(index);

        // Si la balle touche le dessous d'une brique
        if (
            ballTop <= brickBottom + 2 &&
            ballTop >= brickBottom - 2
        ) {
          ballYDirection.value = 1;
        }

        // Si la balle touche le dessus d'une brique
        else if (
            ballBottom >= brickTop - 2 &&
            ballBottom <= brickTop + 2
        ) {
          ballYDirection.value = -1;
        }

        // Si la balle touche le côté gauche d'une brique
        if (
            ballRight >= brickLeft - 2 &&
            ballRight <= brickLeft + 2
        ) {
          ballXDirection.value = -1;
        }

        // Si la balle touche le côté droit d'une brique
        else if (
            ballLeft >= brickRight + 2 &&
            ballLeft <= brickRight - 2
        ) {
          ballXDirection.value = 1;
        }
      }
    }
  });
}

function getBrickCoords(index) {
  const x = Math.floor((index % 14) * (100 / 14) + (100 / 14) / 2);
  const y = Math.floor((index / 14) * (20 / 8) + (20 / 8) / 2);
  return {x, y};
}

function spawnBonus(coords) {
  bonusVisible.value = true;
  bonusX.value = coords["x"];
  bonusY.value = coords["y"];
  console.log("x", coords["x"], "y", coords["y"]);
}

function moveBonus(deltaTime) {

  if (detectPaddleCollision(bonusX.value, bonusY.value, 0)) {
    getBonus();
    bonusVisible.value = false;
    bonusX.value = 0;
    bonusY.value = 0;
    return;
  }

  if (bonusY.value <= 0) {
    bonusVisible.value = false;
  }

  bonusY.value += bonusSpeed.value * deltaTime;
}

function getBonus() {
  const bonusIndex = Math.floor(Math.random() * 5);
  switch (bonusIndex) {
    case 0:
      largePaddleBonus();
      break;

    case 1:
      fastPaddleBonus();
      break;

    case 2:
      slowBallBonus();
      break;

    case 3:
      missileBonus();
      break;

    case 4:
      largeBallBonus();
      break;
  }
}

function largePaddleBonus() {
  activateBonus("Large Paddle", largePaddleBonusDisable);
  paddleWidth.value += largePaddleBonusSize;
}

function largePaddleBonusDisable() {
  paddleWidth.value -= largePaddleBonusSize;
}

function fastPaddleBonus() {
  activateBonus("Fast Paddle", fastPaddleBonusDisable);
  paddleSpeed.value += fastPaddleBonusSpeed;
}

function fastPaddleBonusDisable() {
  paddleSpeed.value -= fastPaddleBonusSpeed;
}

function slowBallBonus() {
  activateBonus("Slow Ball", slowBallBonusDisable);
  ballYVelocity.value += slowBallBonusSpeed;
  ballXVelocity.value += slowBallBonusSpeed;
}

function slowBallBonusDisable() {
  ballYVelocity.value -= slowBallBonusSpeed;
  ballXVelocity.value -= slowBallBonusSpeed;
}

function missileBonus() {
  activateBonus("Missiles", missileBonusDisable);
}

function missileBonusDisable() {

}

function largeBallBonus() {
  activateBonus("Large Ball", largeBallBonusDisable);
  ballSize.value *= largeBallBonusSizeMultiplier;
}

function largeBallBonusDisable() {
  ballSize.value /= largeBallBonusSizeMultiplier;
}

function activateBonusTime(disableFunction) {
  bonusTime.value = initBonusTime;
  const interval = setInterval(() => {
    bonusTime.value--;
    if (bonusTime.value <= 0) {
      bonusActive.value = false;
      disableFunction();
      clearInterval(interval);
    }
  }, 1000);
}

function activateBonus(name, disableFunction) {
  bonusName.value = name;
  activateBonusTime(disableFunction);
  bonusActive.value = true;
}

function brickCollision(index) {
  incrementScore();

  if (!bonusActive.value && Math.random() < .3) spawnBonus(getBrickCoords(index));

  if (!bricks.value.reduce((a, brick) => a = a || brick.visible, false)) {
    win();
  }
}

function checkPressedKeys(deltaTime) {
  if (keysPressed['ArrowLeft']) {
    paddleX.value = Math.max(paddleWidth.value / 2, paddleX.value - paddleSpeed.value * deltaTime);
  }
  if (keysPressed['ArrowRight']) {
    paddleX.value = Math.min(100 - paddleWidth.value / 2, paddleX.value + paddleSpeed.value * deltaTime);
  }
}

function moveBall(deltaTime) {
  ballY.value += ballYVelocity.value * deltaTime * ballYDirection.value;
  ballX.value += ballXVelocity.value * deltaTime * ballXDirection.value;
}

function accelerateBall(){
  ballXVelocity.value *= ballAcceleration;
  bonusSpeed.value *= ballAcceleration;
}

function checkBallAgainstWall() {
  if (ballX.value <= 0) {
    accelerateBall()
    ballXDirection.value = 1;
  }

  if (ballX.value >= 99 - ballSize.value) {
    accelerateBall()
    ballXDirection.value = -1;
  }

  if (ballY.value + ballSize.value >= 100) {
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

  if (detectPaddleCollision(ballX.value, ballY.value, ballSize.value)) {
    ballYDirection.value = -1;
    ballYVelocity.value *= ballAcceleration;
  }

  detectBrickCollision();

  if (ballY.value <= 0) {
    ballYDirection.value = 1;
    ballYVelocity.value *= ballAcceleration;
  }

  if (bonusVisible.value) {
    moveBonus(deltaTime);
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
  switch (Math.trunc(row / 2)) {
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
      <div class="bonus" :style="{left: bonusX + '%', top: bonusY + '%'}" v-if="bonusVisible"></div>
      <div class="ball"
           :style="{left: ballX + '%', top: ballY + '%', height: ballSize + '%', width: ballSize + '%'}"></div>
      <div class="paddle" :style="{width: paddleWidth + '%', left: paddleX-(paddleWidth/2) + '%'}"></div>
    </div>
    <div class="bonus-panel" v-if="bonusActive">
      <h2>{{ bonusName }}</h2>
      <div class="bonus-time" :style="{width: bonusTime + '%'}"></div>
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
  width: 70vh;
  display: block;
  margin: 5% auto auto;
  position: relative;
  height: 70vh;
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
  height: 1.5%;
  background-color: white;
  top: 90%;
  position: absolute;
}

.ball {
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

.bonus {
  height: .5rem;
  width: .5rem;
  position: absolute;
  background-color: limegreen;
}


</style>