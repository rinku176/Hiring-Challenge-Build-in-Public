<template>

      <canvas ref="gameCanvas" v-bind:width="screenWidth * 0.8" v-bind:height="screenHeight * 0.8" class="canvas-class" style="border: 2px solid green;"></canvas>
      <button v-if="showRetry" class="retry-btn" @click="resetGame">Retry</button>
      <button v-if="!showResume" class="pause-btn" @click="pauseGame">Pause</button>
      <button v-if="showResume" class="resume-btn" @click="resumeGame">Resume</button>
      <button v-if="showRewardContinue" class="continue-btn" @click="resumeGame">Continue</button>

 
</template>

<style scoped>
.canvas-class 
{
  position: absolute;
  top: 2rem;
  left: 2rem;
}

.pause-btn, .resume-btn
{
  position: absolute;
  top: 3rem; 
  left: 3rem;
  height: 3rem;
  width: 7rem;
  font-size: 18px;
  background-color: #ab2a93;
  color: white;
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
  font-size: 18px;
  height: 3rem;
  width: 7rem;
  top: calc(70% - 4rem);
left: calc(50% - 4rem);
transform: translate(-50%, -50%);
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

class Frog
{
  constructor() 
  {
    this.x = 0.1 * document.documentElement.clientWidth
    this.y = 0.4 * document.documentElement.clientHeight
    this.stickpad = null
    this.jumping = false
    this.velocityX = 0
    this.velocityY = 0
    this.lives = 3
  }
}

class Fly
{
  constructor() 
  {
    this.x = 500
    this.y = 200
    this.tongueExtended = false
  }
}

class Lilypad
{
  constructor(x, y, number, isCorrect) 
  {
    this.x = x
    this.y = y
    this.number = number
    this.isCorrect = isCorrect
  }
}

const GameState = {
  GameNotStarted: 'GameNotStarted',
  Playing: 'Playing',
  Pause: 'Pause',
  Reward: 'Reward',
  GameOver: 'GameOver'
}

const frog = new Frog()
const fly = new Fly()

const gameCanvas = ref(null)
const showRetry = ref(false)
const showResume = ref(false)
const showRewardContinue = ref(false)

const gravity = 0.5
let screenWidth = document.documentElement.clientWidth
let screenHeight = document.documentElement.clientHeight

let currentGameState = GameState.GameNotStarted

let lilypads = new Array()
let nextcolumn = 0 
let currentNumber = 2
let skipStep = 2
let LilypadMovementSpeed = 0.8

let beepad = null 
let lilypadTimer = null
let AttachBeepadTimer = null
let streak = 0

// Reward animation variables

let rewardAnimationFrame = 0

function drawPretty(canvas,ctx)
{

  // Draw water background with gradient
  const waterGradient = ctx.createLinearGradient(0, 0, 0, canvas.height)
  waterGradient.addColorStop(0, '#87CEEB')
  waterGradient.addColorStop(0.3, '#4682B4')
  waterGradient.addColorStop(1, '#2F4F4F')
  ctx.fillStyle = waterGradient
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  // Draw water ripples
  ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)'
  ctx.lineWidth = 2
  for (let i = 0; i < 5; i++) {
    const y = 100 + i * 120
    ctx.beginPath()
    ctx.moveTo(0, y)
    for (let x = 0; x < canvas.width; x += 20) {
      ctx.lineTo(x, y + Math.sin(x * 0.01 + Date.now() * 0.001) * 5)
    }
    ctx.stroke()
  }
}

function drawAnimatedPlants(ctx) 
{
  const canvas = gameCanvas.value
  const bottomY = canvas.clientTop + canvas.clientHeight 
  const time = Date.now() * 0.001

  // Draw kelp-like plants 
  for (let i = 0; i < 15; i++) 
  {
    const x = (i * 67) + 35
    const segments = 8
    const segmentHeight = 0.014 * canvas.clientWidth
    
    ctx.strokeStyle = '#006400'
    ctx.lineWidth = 3
    ctx.beginPath()
    
    // Draw segmented kelp
    for (let segment = 0; segment < segments; segment++) 
    {
      const segmentY = bottomY - (segment * segmentHeight)
      const sway = Math.sin(time * 0.8 + segment * 0.3 + i * 0.2) * (segment * 2)
      const segmentX = x + sway
      const leafRadiusx = 0.007 * canvas.clientWidth
      
      // Add small side leaves every few segments
      if (segment % 2 === 0 && segment > 0) 
      {
        ctx.save()
        ctx.fillStyle = '#228B22'
        
        // Left leaf
        ctx.beginPath()
        ctx.ellipse(segmentX - 7, segmentY+5, leafRadiusx, leafRadiusx*2, -0.3, 0, Math.PI * 2)
        ctx.fill()
        
        // Right leaf
        ctx.beginPath()
        ctx.ellipse(segmentX + 7, segmentY+5, leafRadiusx, leafRadiusx *2, 0.3, 0, Math.PI * 2)
        ctx.fill()
        
        ctx.restore()
      }
    }
    ctx.stroke()
  }
  
  // Draw small ground plants
  for (let i = 0; i < 25; i++) 
  {
    const x = (i * 40) + 15
    const plantHeight = 25 + Math.sin(time * 0.4 + i) * 5
    
    ctx.strokeStyle = '#32CD32'
    ctx.lineWidth = 2
    
    // Small grass-like plants
    for (let blade = 0; blade < 3; blade++) 
    {
      const bladeX = x + (blade - 1) * 4
      const bladeHeight = plantHeight - blade * 3
      
      ctx.beginPath()
      ctx.moveTo(bladeX, bottomY)
      ctx.lineTo(bladeX + Math.sin(time + blade + i) * 2, bottomY - bladeHeight)
      ctx.stroke()
    }
  }

}

function drawLilypads(ctx) 
{
  ctx.font = '20px Arial'
  let drawleafRadiusY = 0.07482 * ctx.canvas.clientHeight
  lilypads.forEach(set => 
  {
    set.forEach(pad =>
    {
      ctx.save()

      ctx.fillStyle = '#339966'
      ctx.beginPath()
      ctx.ellipse(pad.x, pad.y, drawleafRadiusY * 1.25, drawleafRadiusY, 0, 0, Math.PI * 2)
      ctx.fill()

      ctx.fillStyle = 'white'
      ctx.fillText(pad.number, pad.x - 10, pad.y + 5)

      ctx.restore()
    })
  })  
}

function updateLilypadPosition()
{
  if (currentGameState == GameState.Playing)
  { //to move the lilypads from right to left
    lilypads.forEach(set => 
    {
      set.forEach(pad =>
        pad.x -= LilypadMovementSpeed // speed of movement
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
  ctx.beginPath()
  ctx.arc(frog.x, frog.y, 0.06 * ctx.canvas.clientHeight, 0, Math.PI * 2)
  ctx.fillStyle = 'green'
  ctx.fill()

  ctx.fillStyle = 'white'
  ctx.font = '16px Arial'
  ctx.fillText(currentNumber, frog.x - 15, frog.y + 5)
}

function updateFrogPosition()
{
  //to move frog along with the lilypad
  if (!frog.jumping && frog.stickpad)
  {
    frog.x = frog.stickpad.x
    frog.y= frog.stickpad.y
  }

  //to make the frog jump before it runs out of the screen
  if ( frog.x < 0)
    forcedFrogJump()

}

function drawRewardFrog(ctx) 
{
  // Draw frog in center
  const centerX = 0.5 * ctx.canvas.clientWidth
  const centerY = 0.48 * ctx.canvas.clientHeight
  
  ctx.beginPath()
  ctx.arc(centerX, centerY, 0.05 * ctx.canvas.clientHeight, 0, Math.PI * 2)
  ctx.fillStyle = 'green'
  ctx.fill()

  // Draw eyes
  ctx.beginPath()
  ctx.arc(centerX - (0.02056 *ctx.canvas.clientHeight), centerY - (0.02056 *ctx.canvas.clientHeight), 0.0068 *ctx.canvas.clientHeight, 0, Math.PI * 2)
  ctx.arc(centerX + (0.02056 *ctx.canvas.clientHeight), centerY - (0.02056 *ctx.canvas.clientHeight), 0.0068 *ctx.canvas.clientHeight, 0, Math.PI * 2)
  ctx.fillStyle = 'white'
  ctx.fill()

  // Draw tongue if extended
  if (fly.tongueExtended) 
  {
    ctx.strokeStyle = 'red'
    ctx.lineWidth = 4
    ctx.beginPath()
    ctx.moveTo(centerX, centerY)
    ctx.lineTo(fly.x, fly.y)
    ctx.stroke()
  }
}

function drawFly(ctx) 
{
  if (!fly.tongueExtended)
  {
    ctx.font = '1.5rem Arial'
    ctx.fillText('🪰', fly.x - 12, fly.y + 12)
  }
}

function drawDistraction(ctx) 
{
  ctx.font = '24px Arial'
  if(beepad)
    ctx.fillText("🐝", beepad.x+10,beepad.y+20)
}

function jumpTo(pad)
{
  if(pad.isCorrect)
    frog.stickpad = pad

  if (frog.jumping) 
    return

  frog.jumping = true

  const frames = 20
  frog.velocityX = (frog.stickpad.x - frog.x) / frames
  frog.velocityY = (frog.stickpad.y - frog.y - 50) / frames // slight arc
}

function forcedFrogJump()
{
  for (let pad of lilypads[nextcolumn])
  {
    if (pad.isCorrect) 
    {
      
      jumpTo(pad)
      nextcolumn++
      currentNumber += skipStep
      frog.lives--
      streak = 0
      if(frog.lives<=0)
      {
        frog.lives = 0
        currentGameState = GameState.GameOver
      }
      break;
    }
  }
}
function drawHUD(ctx) 
{
  // Draw hearts for lives 
  const heartSize = 24
  const heartSpacing = 35
  const startX = 0.86105 * ctx.canvas.clientWidth
  const startY = 0.05112 * ctx.canvas.clientHeight
  
  for (let i = 0; i < 3; i++) {
    const heartX = startX + (i * heartSpacing)
    const heartY = startY
    
    if (i < frog.lives) {
      // Draw red hearts for lives
      drawHeart(ctx, heartX, heartY, heartSize, true)
    } else {
      // Draw white hearts for lost lives
      drawHeart(ctx, heartX, heartY, heartSize, false)
    }
  }
  

}

function drawHeart(ctx, x, y, size, filled) {
  ctx.save()
  ctx.translate(x, y)
  
  ctx.font = `${size}px Arial`
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  
  if (filled) {
    ctx.fillStyle = '#FF0000'
    ctx.fillText('❤️', 0, 0)
  } else {
    ctx.fillStyle = '#CCCCCC'
    ctx.fillText('🤍', 0, 0)
  }
  
  ctx.restore()
}

function generateLilypads() 
{
    const canvas = gameCanvas.value

  if(currentGameState == GameState.GameNotStarted)
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
    let x = canvas.clientWidth * 0.5
    let lilypadColumn = new Array()

    for (let i = 0; i < 3; i++)
    {
      let y = canvas.clientHeight * 0.66 * ((i +1) / 3.0)
      lilypadColumn.push(new Lilypad(x, y, allNumbers[i], allNumbers[i] == correctNumber))
    }

    lilypads.push(lilypadColumn)
  }

  else
  {
    const correctNumber = currentNumber + skipStep * (1 + lilypads.length - nextcolumn)
    const wrongNumbers = new Set()
    let x = canvas.clientWidth * 1.28662

    while (wrongNumbers.size < 2) 
    {
      const rand = correctNumber + Math.floor(Math.random() * 5) + 1
      if (rand !== correctNumber) 
        wrongNumbers.add(rand)
    }

    const allNumbers = [correctNumber, ...wrongNumbers]
    allNumbers.sort(() => Math.random() - 0.5)
    let lilypadColumn = new Array()

    for (let i = 0; i < 3; i++)
    {
      let y = canvas.clientHeight * 0.66 * ((i +1)/ 3.0)
      lilypadColumn.push(new Lilypad(x, y, allNumbers[i], allNumbers[i] == correctNumber))
    }
    lilypads.push(lilypadColumn)
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
  ctx.fillRect(0, 0, ctx.canvas.clientWidth, ctx.canvas.clientHeight)

  // Title
  ctx.fillStyle = 'white'
  ctx.font = '1.8rem Arial'
  ctx.textAlign = "center"
  ctx.fillText('Excellent!', 0.5 * ctx.canvas.clientWidth, 0.2 * ctx.canvas.clientHeight)
  
  ctx.font = '1.2rem Arial'
  ctx.fillText('You’re counting like a pro! Keep Going', 0.5 * ctx.canvas.clientWidth, 0.27 * ctx.canvas.clientHeight)

  // Animate the frog catching the fly
  rewardAnimationFrame++
  
  if (rewardAnimationFrame < 60) 
  {
    // Fly buzzes around
    fly.x = 0.5* ctx.canvas.clientWidth + Math.sin(rewardAnimationFrame * 0.2) * 100
    fly.y = 0.27 * ctx.canvas.clientHeight + Math.cos(rewardAnimationFrame * 0.15) * 50
    drawFly(ctx)
  } 
  else if (rewardAnimationFrame < 90) 
  {
    // Tongue extends
    fly.tongueExtended = true
    drawFly(ctx)
  } 
  else if (rewardAnimationFrame < 120) 
  {
    // Fly caught, tongue retracts
    fly.tongueExtended = false
  }
  
  drawRewardFrog(ctx)

  clearInterval(lilypadTimer)
  clearInterval(AttachBeepadTimer)
}

function pauseGame() 
{
  currentGameState = GameState.Pause
}

function resumeGame() 
{
  currentGameState = GameState.Playing
  showResume.value = false
  showRewardContinue.value = false

  rewardAnimationFrame = 0
  fly.tongueExtended = false
  fly.x = 500
  fly.y = 200

  generateLilypads()
  lilypadTimer = setInterval(generateLilypads, 5500)
  AttachBeepadTimer = setInterval(AttachBeepad, Math.random() * 10000 + 2000)
  
  
}

function resetGame() 
{
  frog.lives = 3
  currentGameState = GameState.GameNotStarted
  currentNumber = 2
  frog.x= 100
  frog.y = 300
  lilypads = new Array()
  nextcolumn = 0
  frog.stickpad = null
  showRetry.value = false
  showRewardContinue.value = false
  streak = 0
  rewardAnimationFrame = 0
  fly.tongueExtended = false
  fly.x = 500
  fly.y = 200
  updateJumpArc()
  LilypadMovementSpeed = 0.8
  generateLilypads()
}

function updateJumpArc()
{
  if (frog.jumping) 
  {
    frog.x += frog.velocityX
    frog.y += frog.velocityY
    frog.velocityY += gravity

    // Stop when near target
    const dx = Math.abs(frog.x - frog.stickpad.x)
    const dy = Math.abs(frog.y - frog.stickpad.y)
    if (dx < 50 && dy < 50) 
    {
      frog.x = frog.stickpad.x
      frog.y = frog.stickpad.y
      frog.velocityX = 0
      frog.velocityY = 0
      frog.jumping = false
    }
  }
}

function AttachBeepad()
{
  let randomIndexRow = Math.floor(Math.random() *3 -0.1)
  let randomIndexCol = Math.floor(Math.random() * lilypads.length -0.1)
  beepad = lilypads[randomIndexCol][randomIndexRow ]
}

function render()
{
  screenWidth = document.documentElement.clientWidth
  screenHeight = document.documentElement.clientHeight

  const canvas = gameCanvas.value
  const ctx = canvas.getContext('2d')

  let dynamicFontSize = 0.03193 * screenHeight
  document.querySelector(":root").style.fontSize = dynamicFontSize + "px"

  drawPretty(canvas,ctx)
  drawAnimatedPlants(ctx)
  drawLilypads(ctx)
  drawFrog(ctx)
  drawHUD(ctx)
  
  if(beepad)
    drawDistraction(ctx)
  if (currentGameState == GameState.GameOver) 
    drawGameOver(ctx)
  else if (  currentGameState == GameState.Pause)
    drawPauseGame(ctx)
  else if (currentGameState == GameState.Reward)
    drawRewardScreen(ctx)
  
  requestAnimationFrame(render)
 
}

function gameUpdate()
{
  //to move the lilypads from right to left
  updateLilypadPosition()
  //to make the frog jump along with the lilypad or before it runs out of the screen
  updateFrogPosition()
  updateJumpArc()

  requestAnimationFrame(gameUpdate)
  
}

onMounted(() => 
{
  const canvas = gameCanvas.value
  
  generateLilypads()
 
  render()
  gameUpdate()
  
  canvas.addEventListener('mousedown', e => 
  {
    
    if (frog.jumping ||currentGameState == GameState.GameOver || currentGameState == GameState.Reward) 
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
        jumpTo(pad)

        if (pad.isCorrect) 
        {
          streak++
          
          if (streak% 2 ==0 && streak > 0) {
            currentGameState = GameState.Reward
            LilypadMovementSpeed += 0.2
            streak = 0 // Reset streak after reward
          } 
    
          if(currentGameState == GameState.GameNotStarted)
          {
            currentGameState = GameState.Playing
            generateLilypads()
            lilypadTimer = setInterval(generateLilypads, 5500)
            AttachBeepadTimer = setInterval(AttachBeepad, Math.random() * 10000 + 2000)
          }

          //go with the lilypad
          frog.stickpad = pad
          nextcolumn++
          currentNumber += skipStep
          
          console.log('Correct! Current number:', pad.number)
        }
        else
        { 
          streak = 0
          if(pad == beepad)
        {alert("You got distracted by a bee! Try again.")}
          frog.lives -= 1
          streak = 0
          if (frog.lives <= 0)
          {
            frog.lives =0
            currentGameState = GameState.GameOver
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