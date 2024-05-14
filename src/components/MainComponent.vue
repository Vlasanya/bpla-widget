<template>
  <div class="canvas-container">
    <canvas ref="canvas" width="800" height="500"></canvas>
    <div class="button-container">
      <button v-if="!state.animationRunning" @click="startAnimation" class="start-button">Start</button>
      <button v-else @click="stopAnimation" class="stop-button">Stop</button>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive } from 'vue';
import droneData from '../assets/flight-data.json';

const canvas = ref(null);
const ctx = ref(null);

const state = reactive({
  drone: { x: 400, y: 250 },
  animationRunning: false,
  timerId: null
});

const backgroundImage = new Image();
backgroundImage.src = '/assets/map_image.png';

const droneImage = new Image();
droneImage.src = '/assets/drone.svg';

let timePerStep = 160; 

const startAnimation = () => {
  state.drone = { x: 400, y: 250 };
  let idx = 0;
  state.animationRunning = true;

  const animate = () => {
    if (idx < droneData.length) {
      const { speed, direction } = droneData[idx];
      moveDrone(parseFloat(speed), parseFloat(direction));
      idx++;
      if (idx < droneData.length) {
        state.timerId = setTimeout(animate, timePerStep);
      } else {
        state.animationRunning = false; 
        resetCanvas();
      }
    } else {
      state.animationRunning = false; 
    }
  };

  animate();
};

const stopAnimation = () => {
  clearTimeout(state.timerId);
  state.animationRunning = false;
  resetCanvas();
};

function moveDrone(speed, direction) {
  const pixelsPerSecond = speed * (canvas.value.height / 1000);
  const distance = pixelsPerSecond * (timePerStep / 1000); 
  const angleRad = (Math.PI / 180) * direction; 

  
  const newX = state.drone.x + distance * Math.cos(angleRad);
  const newY = state.drone.y + distance * Math.sin(angleRad);

  state.drone.x = Math.min(Math.max(newX, 0), canvas.value.width);
  state.drone.y = Math.min(Math.max(newY, 0), canvas.value.height);

  drawDrone();
}

function drawDrone() {
  if (ctx.value) {
    ctx.value.clearRect(0, 0, canvas.value.width, canvas.value.height);
    ctx.value.drawImage(backgroundImage, 0, 0, canvas.value.width, canvas.value.height);
    ctx.value.drawImage(droneImage, state.drone.x - droneImage.width / 2, state.drone.y - droneImage.height / 2, droneImage.width, droneImage.height);
  }
}

function resetCanvas() {
  state.drone = { x: 400, y: 250 };
  drawDrone();
}

onMounted(() => {
  const canvasElement = canvas.value;
  if (canvasElement) {
    ctx.value = canvasElement.getContext('2d');
    backgroundImage.onload = () => {
      ctx.value.drawImage(backgroundImage, 0, 0, canvasElement.width, canvasElement.height);
      drawDrone();
    };
  }
});
</script>

<style scoped>
canvas {
  width: 100%;
  height: 100%;
  max-height: 100vh;
}

.canvas-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
}

.button-container {
  display: flex;
  justify-content: center;
  margin-top: 10px;
  position: absolute;
  bottom: 13%;
}

.start-button {
  width: 12vw;
  background-color: yellow;
  color: black;
  padding: 4px;
  cursor: pointer;
  border-radius: 10%;
  transition: background-color 0.3s, transform 0.2s;
}

.start-button:hover {
  background-color: #ffff99;
  transform: scale(1.05);
}

.stop-button {
  width: 12vw;
  background-color: black;
  color: white;
  padding: 4px;
  cursor: pointer;
  border-radius: 10%;
  transition: background-color 0.3s, transform 0.2s;
}

.stop-button:hover {
  background-color: #333;
  transform: scale(1.05);
}
</style>
