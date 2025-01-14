<script setup>
import {onMounted, onUnmounted, reactive, ref} from 'vue'

const score = ref(0);
const playing = ref(false);
const gameOver = ref(false);

const paddleWidth = ref(10);
const paddleX = ref(50);
const paddleSpeed = ref(2)

const keysPressed = reactive({});

function incrementScore() {
  score.value += 100;
}

function start() {
  gameOver.value = false;
  playing.value = true;
  score.value = 0;
}

function setGameOver(){
  gameOver.value = true;
  playing.value = false;
}

function quit(){
  playing.value = false;
  gameOver.value = false;
}

function handleKeyDown(event) {
  keysPressed[event.key] = true;

  if (keysPressed['ArrowLeft']) {
    paddleX.value = Math.max(0, paddleX.value - paddleSpeed.value);
  }
  if (keysPressed['ArrowRight']) {
    paddleX.value = Math.min(100, paddleX.value + paddleSpeed.value);
  }
}

function handleKeyUp(event) {
  keysPressed[event.key] = false;
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
  window.addEventListener('keyup', handleKeyUp);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
  window.addEventListener('keyup', handleKeyUp);
});

</script>

<template>
  <div v-if="playing" class="main">
    <p id="score">Score: {{ score }}</p>
    <button @click="incrementScore">Increment score</button>
    <button @click="setGameOver">Game Over</button>
    <div class="paddle" :style="{width: paddleWidth + '%', left: paddleX-(paddleWidth/2) + '%'}"></div>
  </div>
  <div v-else-if="gameOver" class="main">
    <h1>GAME OVER</h1>
    <button @click="start" class="mainButton">Replay</button>
    <button @click="quit" class="mainButton">Quit</button>
  </div>
  <div v-else class="main">
    <h1>BREAKOUT</h1>
    <button @click="start" class="mainButton">Play</button>
  </div>
</template>

<style>

  *{
    font-family: 'Press Start 2P', cursive;
  }

  body {
    color: white;
    background-color: black;
  }

  h1{
    text-align: center;
    font-size: 4rem;
  }

  .mainButton{
    text-align: center;
    display: block;
    margin: 0 auto;
    font-size: 2rem;
    padding: 1rem;
    width: 20rem;
  }

  #score{
    font-size: 2rem;
  }

  .paddle{
    height: 1rem;
    background-color: white;
    top: 80%;
    position: absolute;
  }
</style>
