<template>
  <div class="page-view">
    <div class="page-container">
      <canvas ref="gameCanvas" width="1000" height="600" style="border: 2px solid green;"></canvas>
    </div>
  </div>
</template>

<style scoped>
.page-view 
{
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.page-container 
{
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
let currentNumber = 2
let skipStep = 2

function drawLilypads(ctx) 
{
  ctx.font = '20px Arial'
  lilypads.forEach(pad => 
  {
    ctx.save()

    ctx.fillStyle = '#339966'
    ctx.beginPath()
    ctx.ellipse(pad.x, pad.y, 50, 40, 0, 0, Math.PI * 2)
    ctx.fill()

     ctx.fillStyle = 'white'
    ctx.fillText(pad.number, pad.x - 10, pad.y + 5)

    ctx.restore()
  })
}

function drawFrog(ctx) 
{
  ctx.beginPath()
  ctx.arc(frogX, frogY, 30, 0, Math.PI * 2)
  ctx.fillStyle = 'green'
  ctx.fill()

  ctx.fillStyle = 'white'
  ctx.font = '16px Arial'
  ctx.fillText(currentNumber, frogX - 15, frogY + 5)

}

function generateLilypads() 
{
  const correctNumber = currentNumber + skipStep
  const wrongNumbers = new Set()
  while (wrongNumbers.size < 2) 
  {
    const rand = correctNumber + Math.floor(Math.random() * 5) + 1
    if (rand !== correctNumber) 
      wrongNumbers.add(rand)
  }

  const allNumbers = [correctNumber, ...wrongNumbers]
  allNumbers.sort(() => Math.random() - 0.5)
  const y = [130, 300, 480];

  return y.map((height,i) => (
  {
    x: 300,
    y: height,
    number: allNumbers[i],
    isCorrect: allNumbers[i] === correctNumber
  }))
}

function updateGame(canvas, ctx) 
{
 // Drawing
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  ctx.fillStyle = '#ccffcc'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  drawLilypads(ctx)
  drawFrog(ctx)
}

onMounted(() => 
{
  const canvas = gameCanvas.value
  const ctx = canvas.getContext('2d')

  lilypads = generateLilypads()

  canvas.addEventListener('mousedown', e => 
  {
    const rect = canvas.getBoundingClientRect()
    const clickX = e.clientX - rect.left
    const clickY = e.clientY - rect.top

    for (let pad of lilypads) 
    {
      const dx = clickX - pad.x
      const dy = clickY - pad.y
      const dist = Math.sqrt(dx * dx + dy * dy)

      if (dist < 50) 
      {
        if (pad.isCorrect) 
        {
          currentNumber += skipStep
          lilypads= generateLilypads()
          console.log('Correct! Current number:', pad.number)
        }
        break;
      }
    }
    updateGame(canvas, ctx)
  })

  updateGame(canvas, ctx)
})
</script>
