<script setup lang="ts">
import { computed, onMounted, ref } from 'vue';

interface Point {
  x: number;
  y: number;
}
interface Branch {
  start: Point;
  length: number;
  angle: number
}

const el = ref<HTMLCanvasElement>();
const ctx = computed(() => el.value!.getContext('2d'));

const height = ref(document.body.clientHeight);
const width = ref(document.body.clientWidth);

const pendingTasks: Function[] = [];

function init() {
  ctx.value!.strokeStyle = 'rgba(100, 100, 100, 0.5)';
  ctx.value!.lineWidth = .1;
  step({
    start: { x: width.value / 2, y: height.value },
    length: 4,
    angle: -Math.PI / 2
  });
  step({
    start: { x: width.value / 2, y: 0 },
    length: 4,
    angle: Math.PI / 2
  });
  step({
    start: { x: 0, y: height.value / 2 },
    length: 4,
    angle: 0
  });
  step({
    start: { x: width.value, y: height.value / 2 },
    length: 4,
    angle: Math.PI
  });
}

function step(branch: Branch, depth = 0) {
  let endPoint = getEndPoint(branch);
  drawBranch(branch);

  if (depth < 5 || Math.random() > 0.5) {
    pendingTasks.push(() => step({
      start: endPoint,
      length: branch.length + (Math.random() * 4 - 2),
      angle: branch.angle + Math.random() * 0.3
    }, depth + 1));
  }
  if (depth < 5 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: endPoint,
      length: branch.length + (Math.random() * 4 - 2),
      angle: branch.angle - Math.random() * 0.3
    }, depth + 1));
  }
}

function frame() {
  const tasks = [...pendingTasks];
  pendingTasks.length = 0;
  tasks.forEach(fn => fn());
}

let framesCount = 0;
function startFrame() {
  requestAnimationFrame(() => {
    framesCount++;
    if (framesCount % 5 == 0) {
      frame();
    }
    startFrame();
  });
}

function drawBranch(branch: Branch) {
  lintTo(branch.start, getEndPoint(branch));
}

function getEndPoint(branch: Branch): Point {
  const point = {
    x: branch.start.x + branch.length * Math.cos(branch.angle),
    y: branch.start.y + branch.length * Math.sin(branch.angle)
  };
  return point;
}

function lintTo(startPoint: Point, endPoint: Point) {
  console.log(startPoint, endPoint)
  ctx.value!.beginPath();
  ctx.value!.moveTo(startPoint.x, startPoint.y);
  ctx.value!.lineTo(endPoint.x, endPoint.y);
  ctx.value!.stroke();
}

startFrame();
onMounted(() => {
  init();
});

</script>

<template>
    <canvas ref="el" :height="height" :width="width"></canvas>
    <h1>Keep Learning</h1>
</template>

<style>
* {
  margin: 0;
  padding: 0;
}
body {
  height: 100vh;
  width: 100vw;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  height: 100%;
  background-color: #202124;
}
h1 {
  position: absolute;
  top: 50%;
  left: 50%;
  color: #FFF;
  transform: translate(-50%, -50%);
  font-size: 50px;
}
</style>
