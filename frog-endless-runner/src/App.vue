<template>
  <div class="page-view">
    <div class="page-container">
      <canvas ref="gameCanvas" width="1000" height="600" style="border: 2px solid green;"></canvas>
      <button v-if="showRetry" class="retry-btn" @click="resetGame">Retry</button>
      <button v-if="!showResume" class="pause-btn" @click="pauseGame">Pause</button>
      <button v-if="showResume" class="resume-btn" @click="resumeGame">Resume</button>
      <button v-if="showRewardContinue" class="continue-btn" @click="continueGame">Continue</button>
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

.pause-btn, .resume-btn
{
  position: absolute;
  top: 20px; 
  left: 20px;
  padding: 10px 20px;
  font-size: 18px;
  background-color: #ab2a93;
  color: white;
  border-radius: 8px;
}

.pause-btn:hover , .resume-btn:hover
{
  background-color: #902085;
}

.retry-btn 
{
  position: absolute;
  top: 360px; /* Below 'Game Over' canvas text */
  transform: translateX(-50%);
  padding: 10px 20px;
  font-size: 18px;
  background-color: #2ecc71;
  color: white;
  border-radius: 8px;
}

.retry-btn:hover 
{
  background-color: #27ae60;
}

.continue-btn 
{
  position: absolute;
  top: 400px; /* Below reward screen text */
  left: 50%;
  transform: translateX(-50%);
  padding: 10px 20px;
  font-size: 18px;
  background-color: #3498db;
  color: white;
  border-radius: 8px;
}

.continue-btn:hover 
{
  background-color: #2980b9;
}
</style>

<script setup>
import { onMounted, ref } from 'vue'

const gameCanvas = ref(null)
const showRetry = ref(false)
const showResume = ref(false)
const showRewardContinue = ref(false)

let frogX = 100 // frog starts at the left
let frogY = 300

let lilypads = new Array()
let nextcolumn = 0 
let currentNumber = 2
let skipStep = 2
let speed = 0.8

let lives = 3
let gameOver = false
let pause = false
let showReward = false

let firstCorrect = false // to check if the first correct lilypad is clicked
let targetX = frogX
let targetY = frogY
let jumping = false
let velocityX = 0
let velocityY = 0
const gravity = 0.5
let stickpad = null 
let beepad = null 
let lilypadTimer = null
let AttachBeepadTimer = null
let streak = 0

// Reward animation variables
let flyX = 500
let flyY = 200
let rewardAnimationFrame = 0
let tongueExtended = false


function drawLilypads(ctx) 
{
  ctx.font = '20px Arial'
  lilypads.forEach(set => 
  {
    set.forEach(pad =>
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
  })

  if (firstCorrect)
  { //to move the lilypads from right to left
    lilypads.forEach(set => 
    {
      set.forEach(pad =>
        pad.x -= speed // speed of movement
        )
      if (set[0] < 0)
      {
        lilypads.shift()
        nextcolumn--
      }
    })
  }
}

function drawFrog(ctx) 
{ 
  if (!jumping && stickpad)
  {
    frogX = stickpad.x
    frogY = stickpad.y
  }

  if ( frogX < 0)
    forcedFrogJump()

  ctx.beginPath()
  ctx.arc(frogX, frogY, 30, 0, Math.PI * 2)
  ctx.fillStyle = 'green'
  ctx.fill()

  ctx.fillStyle = 'white'
  ctx.font = '16px Arial'
  ctx.fillText(currentNumber, frogX - 15, frogY + 5)
}

function drawRewardFrog(ctx) 
{
  // Draw frog in center
  const centerX = 500
  const centerY = 350
  
  ctx.beginPath()
  ctx.arc(centerX, centerY, 40, 0, Math.PI * 2)
  ctx.fillStyle = 'green'
  ctx.fill()

  // Draw eyes
  ctx.beginPath()
  ctx.arc(centerX - 15, centerY - 15, 5, 0, Math.PI * 2)
  ctx.arc(centerX + 15, centerY - 15, 5, 0, Math.PI * 2)
  ctx.fillStyle = 'white'
  ctx.fill()

  // Draw tongue if extended
  if (tongueExtended) 
  {
    ctx.strokeStyle = 'red'
    ctx.lineWidth = 4
    ctx.beginPath()
    ctx.moveTo(centerX, centerY)
    ctx.lineTo(flyX, flyY)
    ctx.stroke()
  }
}

function drawFly(ctx) 
{
  if (!tongueExtended)
  {
    ctx.font = '24px Arial'
    ctx.fillText('🪰', flyX - 12, flyY + 12)
  }
}

function drawDistraction(ctx) 
{
  ctx.font = '24px Arial'
  if(beepad)
    ctx.fillText("🐝", beepad.x+10,beepad.y+20)
}

function jumpTo(x,y)
{
  if (jumping) 
    return

  jumping = true
  targetX = x
  targetY = y

  const frames = 20
  velocityX = (targetX - frogX) / frames
  velocityY = (targetY - frogY - 50) / frames // slight arc
}

function forcedFrogJump()
{
 
  for (let pad of lilypads[nextcolumn])
  {
    if (pad.isCorrect) 
    {
      jumpTo(pad.x, pad.y)
      stickpad = pad
      nextcolumn++
      currentNumber += skipStep
      lives--
      streak = 0
      if(lives<=0)
      {
        lives = 0
        gameOver = true
      }
      break;
    }
  }
}

function drawHUD(ctx) 
{
  ctx.fillStyle = 'black'
  ctx.font = '20px Arial'
  ctx.fillText(`Lives: ${lives}`, 850, 30)
  ctx.fillText(`Streak: ${streak}`, 850, 60)
}

function generateLilypads() 
{
  if(firstCorrect == false)
  {const correctNumber = currentNumber + skipStep
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

    lilypads.push (y.map((height,i) => (
    {
      x: 500, 
      y: height,
      number: allNumbers[i],
      isCorrect: allNumbers[i] == correctNumber
    })))
  }

  else
  {
    const correctNumber = currentNumber + skipStep * (1 + lilypads.length - nextcolumn)
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

    lilypads.push (y.map((height,i) => (
    {
      x: 1300,
      y: height,
      number: allNumbers[i],
      isCorrect: allNumbers[i] == correctNumber
    })))
  }
}

function drawGameOver(ctx) 
{
  showRetry.value = true
  ctx.fillStyle = 'rgba(0,0,0,0.6)'
  ctx.fillRect(0, 0, 1000, 680)

  ctx.fillStyle = 'white'
  ctx.font = '40px Arial'
  ctx.fillText('Game Over', 400, 300)

  clearInterval(lilypadTimer)
  clearInterval(AttachBeepadTimer)
}

function drawPauseGame(ctx) 
{
  
  showResume.value = true
  ctx.fillStyle = 'rgba(0,0,0,0.6)'
  ctx.fillRect(0, 0, 1000, 680)

  ctx.fillStyle = 'white'
  ctx.font = '40px Arial'
  ctx.fillText('Game Paused', 400, 300)

  clearInterval(lilypadTimer)
  clearInterval(AttachBeepadTimer)
}

function drawRewardScreen(ctx) 
{
  showRewardContinue.value = true
  
  // Background
  ctx.fillStyle = 'rgba(0,0,0,0.6)'
  ctx.fillRect(0, 0, 1000, 680)

  // Title
  ctx.fillStyle = 'white'
  ctx.font = '40px Arial'
  ctx.fillText('Excellent!', 400, 150)
  
  ctx.font = '24px Arial'
  ctx.fillText('You’re counting like a pro! Keep Going', 300, 200)

  // Animate the frog catching the fly
  rewardAnimationFrame++
  
  if (rewardAnimationFrame < 60) 
  {
    // Fly buzzes around
    flyX = 500 + Math.sin(rewardAnimationFrame * 0.2) * 100
    flyY = 200 + Math.cos(rewardAnimationFrame * 0.15) * 50
    drawFly(ctx)
  } 
  else if (rewardAnimationFrame < 90) 
  {
    // Tongue extends
    tongueExtended = true
    drawFly(ctx)
  } 
  else if (rewardAnimationFrame < 120) 
  {
    // Fly caught, tongue retracts
    tongueExtended = false
  }
  
  drawRewardFrog(ctx)
  
  if (rewardAnimationFrame < 120) 
  {
    requestAnimationFrame(() => render())
  }

  clearInterval(lilypadTimer)
  clearInterval(AttachBeepadTimer)
}

function pauseGame() 
{
  pause = true
}

function resumeGame() 
{
  pause = false
  showResume.value = false

  if(firstCorrect==true)
  {
    console.log(firstCorrect)
    generateLilypads()
    lilypadTimer = setInterval(generateLilypads, 5500)
    AttachBeepadTimer = setInterval(AttachBeepad, Math.random() * 10000 + 2000)
  }
  render()
}

function continueGame() 
{
  showReward = false
  showRewardContinue.value = false
  rewardAnimationFrame = 0
  tongueExtended = false
  flyX = 500
  flyY = 200
  
  generateLilypads()
  lilypadTimer = setInterval(generateLilypads, 5500)
  AttachBeepadTimer = setInterval(AttachBeepad, Math.random() * 10000 + 2000)
  render()
}

function resetGame() 
{
  lives = 3
  gameOver = false
  showReward = false
  currentNumber = 2
  frogX= 100
  frogY = 300
  lilypads = new Array()
  nextcolumn = 0
  stickpad = null
  showRetry.value = false
  showRewardContinue.value = false
  firstCorrect = false
  streak = 0
  rewardAnimationFrame = 0
  tongueExtended = false
  flyX = 500
  flyY = 200
  updateJumpArc()
  render()
  speed = 0.8
  generateLilypads()
}

function updateJumpArc()
{
  if (jumping) 
  {
    frogX += velocityX
    frogY += velocityY
    velocityY += gravity

    // Stop when near target
    const dx = Math.abs(frogX - targetX)
    const dy = Math.abs(frogY - targetY)
    if (dx < 50 && dy < 50) 
    {
      frogX = targetX
      frogY = targetY
      velocityX = 0
      velocityY = 0
      jumping = false
    }
  }
  requestAnimationFrame(updateJumpArc)
}

function AttachBeepad()
{
  let randomIndexRow = Math.floor(Math.random() *3 -0.1)
  let randomIndexCol = Math.floor(Math.random() * lilypads.length -0.1)
  beepad = lilypads[randomIndexCol][randomIndexRow ]
}

function render()
{
  const canvas = gameCanvas.value
  const ctx = canvas.getContext('2d')

  // Drawing
  ctx.fillStyle = '#ccffcc'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  drawLilypads(ctx)
  drawFrog(ctx)
  drawHUD(ctx)
  
  if(beepad)
    drawDistraction(ctx)

  if (gameOver) 
    drawGameOver(ctx)
  else if (pause)
    drawPauseGame(ctx)
  else if (showReward)
    drawRewardScreen(ctx)
  else
    requestAnimationFrame(render)
 
}

onMounted(() => 
{
  const canvas = gameCanvas.value
  
  generateLilypads()
 
  render()

  canvas.addEventListener('mousedown', e => 
  {
    
    if (jumping ||gameOver || showReward) 
      return

    const rect = canvas.getBoundingClientRect()
    const clickX = e.clientX - rect.left
    const clickY = e.clientY - rect.top

    for (let pad of lilypads[nextcolumn]) 
    {
      const dx = clickX - pad.x
      const dy = clickY - pad.y
      const dist = Math.sqrt(dx * dx + dy * dy)

      if (dist < 50) 
      { 
        jumpTo(pad.x, pad.y)

        if (pad.isCorrect) 
        {
          streak++
          
          if (streak% 10 ==0 && streak > 0) {
            showReward = true
            speed += 0.2
            streak = 0 // Reset streak after reward
          } 
    
          if(firstCorrect == false)
          {
            firstCorrect = true
            generateLilypads()
            lilypadTimer = setInterval(generateLilypads, 5500)
            AttachBeepadTimer = setInterval(AttachBeepad, Math.random() * 10000 + 2000)
          }

          //go with the lilypad
          stickpad = pad
          nextcolumn++
          currentNumber += skipStep
          
          console.log('Correct! Current number:', pad.number)
        }
        else
        {
          streak = 0
          if(pad == beepad)
        {alert("You got distracted by a bee! Try again.")}
          lives -= 1
          streak = 0
          if (lives <= 0)
          {
            lives =0
            gameOver = true
          }
        }
        break;
      }
      
    }
    updateJumpArc()
  })

  updateJumpArc()
})
</script>