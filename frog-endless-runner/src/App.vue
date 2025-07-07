<template>
  <div class="page-view">
    <div class="page-container">
      <canvas ref="gameCanvas" width="1000" height="600" style="border: 2px solid green;"></canvas>
    </div>
  </div>
</template>

<style scoped>
.page-view {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.page-container {
  position: relative;
  width: 1000px;
  height: 630px;
}
</style>


<script setup>
import { onMounted, ref } from 'vue'

const gameCanvas = ref(null)

let frogX = 100 // frog starts at the left
let frogY = 300

let lilypads = []

function drawLilypads(ctx) {
  lilypads.forEach(pad => {
    ctx.save()

    ctx.fillStyle = '#339966'
    ctx.beginPath()
    ctx.ellipse(pad.x, pad.y, 50, 40, 0, 0, Math.PI * 2)
    ctx.fill()

    ctx.restore()
  })
}

function drawFrog(ctx) {
  ctx.beginPath()
  ctx.arc(frogX, frogY, 30, 0, Math.PI * 2)
  ctx.fillStyle = 'green'
  ctx.fill()
}

function generateLilypads() {
  const y = [130, 300, 480];

  return y.map((i) => ({
    x: 300,
    y: i
  }))
}

function updateGame(canvas, ctx) {
 // Drawing
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  ctx.fillStyle = '#ccffcc'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  drawLilypads(ctx)
  drawFrog(ctx)
}

onMounted(() => {
  const canvas = gameCanvas.value
  const ctx = canvas.getContext('2d')

  lilypads = generateLilypads()

  updateGame(canvas, ctx)
})
</script>
