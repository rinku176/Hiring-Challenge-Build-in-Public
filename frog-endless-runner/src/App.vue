<template>
  <canvas ref="gameCanvas" v-bind:width="screenWidth * 0.8" v-bind:height="screenHeight * 0.8" class="canvas-class" style="border: 2px solid green;"></canvas>
  <button v-if="showRetry" class="retry-btn" @click="resetGame">Retry</button>
  <button v-if="!showResume && !showLevelSelect" class="pause-btn" @click="pauseGame">Pause</button>
  <button v-if="showResume" class="resume-btn" @click="resumeGame">Resume</button>
  <button v-if="showRewardContinue" class="continue-btn" @click="resumeGame">Continue</button>
  <button v-if="showLevelSelect" class="level-select-btn" id= "level1button" @click="startLevel1">Level1</button>
  <button v-if="showLevelSelect" class="level-select-btn" id= "level2button"@click="startLevel2">Level2</button>
  <button v-if="showLevelSelect" class="level-select-btn" id= "level3button" @click="startLevel3">Level3</button>
</template>

<style scoped>
.canvas-class 
{
  position: absolute;
  top: 2rem;
  left: 2rem;
}

.level-select-btn
{
  position: absolute;
  font-size: 1rem;
  height: 3rem;
  width: 7rem;
  transform: translate(-50%, -50%);
  background-color: #5d12f3;
  color: white;
  border-radius: 8px;
}

#level1button{
  top: 50%;
  left: 30%;
}

#level2button{
  top: 50%;
  left: 45%;
}

#level3button{
  top: 50%;
  left: 60%;
}

.pause-btn, .resume-btn
{
  position: absolute;
  top: 3rem; 
  left: 3rem;
  font-size: 1rem;
  background-color: #732aab;
  color: white;
}

.pause-btn:hover , .resume-btn:hover
{
  background-color: rgb(105, 32, 144);
}

.retry-btn 
{
  position: absolute;
  font-size: 1rem;
  height: 3rem;
  width: 7rem;
  top: calc(70% - 4rem);
  left: calc(50% - 4rem);
  transform: translate(-50%, -50%);
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
  font-size: 1rem;
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
import { h, onMounted, ref } from 'vue'

const RuleSet = {
  None: 'None',
  Level1: 'Level1',  // Level 1: The Basic Level with Hints
  Level2: 'Level2',  // Level 2: The Level with Arbitrary Numbers
  Level3: 'Level3'   // Level 3: The Reverse-skip number Level
}

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
    this.jumpTarget = null
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
  LevelSelect: 'LevelSelect',
  GameNotStarted: 'GameNotStarted',
  Playing: 'Playing',
  Pause: 'Pause',
  Reward: 'Reward',
  GameOver: 'GameOver',
  Distracted: 'Distracted'
}

const frog = new Frog()
const fly = new Fly()

const gameCanvas = ref(null)
const showRetry = ref(false)
const showResume = ref(false)
const showRewardContinue = ref(false)
const showLevelSelect = ref(true)

const gravity = 0.5
let screenWidth = document.documentElement.clientWidth
let screenHeight = document.documentElement.clientHeight

let currentGameState = GameState.LevelSelect
let nowRuleSet = RuleSet.Level1
let lilypads = new Array()
let nextcolumn = 0 
let currentNumber = 2
let skipStep = 2
let LilypadMovementSpeed = 0.8

let beepad = null 
let lilypadTimer = null
let AttachBeepadTimer = null
let streak = 0
let hintLilypad = 0

let crocodile = {
  x: 0,
  y: 0,
  visible: false,
  timer: 0,
  animationFrame: 0
}

// Reward animation variables

let rewardAnimationFrame = 0

function startGame()
{
  showLevelSelect.value = false
  currentGameState = GameState.GameNotStarted
  generateLilypads()
}

function startLevel1()
{
  nowRuleSet = RuleSet.Level1
  skipStep = 2
  
  startGame()
}

function startLevel2()
{  
  nowRuleSet = RuleSet.Level2
  currentNumber = Math.floor(Math.random() *70 +20) // Random start number between 20-70 for Level 2
  skipStep = 4//Math.floor(Math.random() *5 +1)
  startGame()
}

function startLevel3()
{ 
  nowRuleSet = RuleSet.Level3
  skipStep =3
  currentNumber = 3
  startGame()
}

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
  for (let i = 0; i < 30; i++) 
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
  for (let i = 0; i < 32; i++) 
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


function showCrocodile() 
{
  const canvas = gameCanvas.value
  crocodile.x = canvas.clientWidth  
  crocodile.y = canvas.clientHeight -20 
  crocodile.visible = true
  crocodile.timer = 480 
  crocodile.animationFrame = 0
}

function updateCrocodile() 
{
  if (crocodile.visible) {
    crocodile.timer--
    crocodile.animationFrame++
    
    if (crocodile.timer <= 0) {
      crocodile.visible = false
    }
  }
}

function drawCrocodile(ctx) 
{
  if (!crocodile.visible) return
  
  const canvas = ctx.canvas
  const canvasWidth = canvas.clientWidth
  const canvasHeight = canvas.clientHeight
  
  const crocodileWidth = canvasWidth * 0.8 
  const crocodileHeight = canvasHeight * 0.15 
  
 
  let yOffset = 0
  if (crocodile.animationFrame < 20) {
    yOffset = (20 - crocodile.animationFrame) * 4 
  } else if (crocodile.timer < 20) {
    yOffset = (20 - crocodile.timer) * 4 
  }
  
  const drawY = crocodile.y + yOffset
  const centerX = canvasWidth * 0.5
  
  ctx.save()
  
  ctx.fillStyle = '#2F4F2F'
  ctx.beginPath()
  ctx.ellipse(centerX, drawY, crocodileWidth * 0.4, crocodileHeight * 0.3, 0, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw head (left side)
  ctx.fillStyle = '#228B22'
  ctx.beginPath()
  ctx.ellipse(centerX - crocodileWidth * 0.35, drawY - crocodileHeight * 0.05, crocodileWidth * 0.15, crocodileHeight * 0.25, 0, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw tip
  ctx.fillStyle = '#32CD32'
  ctx.beginPath()
  ctx.ellipse(centerX - crocodileWidth * 0.45, drawY - crocodileHeight * 0.05, crocodileWidth * 0.05, crocodileHeight * 0.12, 0, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw tail (right side)
  ctx.fillStyle = '#2F4F2F'
  ctx.beginPath()
  ctx.ellipse(centerX + crocodileWidth * 0.35, drawY, crocodileWidth * 0.12, crocodileHeight * 0.2, 0, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw back ridges/spikes across the length
  ctx.fillStyle = '#1C3A1C'
  const numSpikes = 12
  for (let i = 0; i < numSpikes; i++) {
    const spikeX = centerX - crocodileWidth * 0.3 + (i * crocodileWidth * 0.05)
    const spikeHeight = crocodileHeight * 0.15
    ctx.beginPath()
    ctx.moveTo(spikeX, drawY - crocodileHeight * 0.3)
    ctx.lineTo(spikeX + crocodileWidth * 0.02, drawY - crocodileHeight * 0.3 - spikeHeight)
    ctx.lineTo(spikeX + crocodileWidth * 0.04, drawY - crocodileHeight * 0.3)
    ctx.closePath()
    ctx.fill()
  }
  
  // Draw nostrils
  ctx.fillStyle = '#000000'
  ctx.beginPath()
  ctx.ellipse(centerX - crocodileWidth * 0.42, drawY - crocodileHeight * 0.08, crocodileWidth * 0.008, crocodileHeight * 0.02, 0, 0, Math.PI * 2)
  ctx.ellipse(centerX - crocodileWidth * 0.4, drawY - crocodileHeight * 0.08, crocodileWidth * 0.008, crocodileHeight * 0.02, 0, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw eyes 
  ctx.fillStyle = '#228B22'
  ctx.beginPath()
  ctx.ellipse(centerX - crocodileWidth * 0.25, drawY - crocodileHeight * 0.2, crocodileWidth * 0.03, crocodileHeight * 0.08, 0, 0, Math.PI * 2)
  ctx.ellipse(centerX - crocodileWidth * 0.15, drawY - crocodileHeight * 0.2, crocodileWidth * 0.03, crocodileHeight * 0.08, 0, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw eye pupils
  ctx.fillStyle = '#FFFF00'
  ctx.beginPath()
  ctx.ellipse(centerX - crocodileWidth * 0.25, drawY - crocodileHeight * 0.2, crocodileWidth * 0.015, crocodileHeight * 0.06, 0, 0, Math.PI * 2)
  ctx.ellipse(centerX - crocodileWidth * 0.15, drawY - crocodileHeight * 0.2, crocodileWidth * 0.015, crocodileHeight * 0.06, 0, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw vertical pupils 
  ctx.fillStyle = '#000000'
  ctx.fillRect(centerX - crocodileWidth * 0.255, drawY - crocodileHeight * 0.25, crocodileWidth * 0.01, crocodileHeight * 0.1)
  ctx.fillRect(centerX - crocodileWidth * 0.155, drawY - crocodileHeight * 0.25, crocodileWidth * 0.01, crocodileHeight * 0.1)
  
  // Draw mouth line
  ctx.strokeStyle = '#000000'
  ctx.lineWidth = 3
  ctx.beginPath()
  ctx.moveTo(centerX - crocodileWidth * 0.45, drawY + crocodileHeight * 0.05)
  ctx.lineTo(centerX - crocodileWidth * 0.1, drawY + crocodileHeight * 0.1)
  ctx.stroke()
  
  // Draw sharp teeth along the mouth
  ctx.fillStyle = '#FFFFFF'
  const numTeeth = 16
  for (let i = 0; i < numTeeth; i++) {
    const toothX = centerX - crocodileWidth * 0.42 + (i * crocodileWidth * 0.02)
    const toothHeight = crocodileHeight * 0.08
    ctx.beginPath()
    ctx.moveTo(toothX, drawY + crocodileHeight * 0.05)
    ctx.lineTo(toothX + crocodileWidth * 0.005, drawY + crocodileHeight * 0.05 - toothHeight)
    ctx.lineTo(toothX + crocodileWidth * 0.01, drawY + crocodileHeight * 0.05)
    ctx.closePath()
    ctx.fill()
  }
   
  ctx.restore()
}

function drawLilypads(ctx) 
{
  let drawleafRadiusY = 0.07482 * ctx.canvas.clientHeight
  let drawleafRadiusX = drawleafRadiusY * 1.25
  
  lilypads.forEach(set => 
  {
    set.forEach(pad =>
    {
      ctx.save()

      // Create gradient for lilypad
      const gradient = ctx.createRadialGradient(pad.x - 15, pad.y - 10, 0, pad.x, pad.y, drawleafRadiusX)
      gradient.addColorStop(0, '#90EE90')
      //gradient.addColorStop(0.4, '#32CD32')
      //gradient.addColorStop(0.8, '#228B22')
      gradient.addColorStop(1, '#006400')

      // Draw main lilypad
      ctx.fillStyle = gradient
      ctx.beginPath()
      ctx.ellipse(pad.x, pad.y, drawleafRadiusX, drawleafRadiusY, 0, 0, Math.PI * 2)
      ctx.fill()

      // Draw lilypad outline
      ctx.strokeStyle = '#006400'
      ctx.lineWidth = 2
      ctx.stroke()


      // Draw veins on the lilypad
      ctx.strokeStyle = '#228B22'
      ctx.lineWidth = 1
      ctx.setLineDash([3, 3])
      
      // Central vein
      ctx.beginPath()
      ctx.moveTo(pad.x - drawleafRadiusX * 0.6, pad.y)
      ctx.lineTo(pad.x + drawleafRadiusX * 0.6, pad.y)
      ctx.stroke()
    
      
      ctx.setLineDash([]) // Reset line dash

      // Highlight effect for hint lilypads
      if(hintLilypad > 0 && pad.isCorrect && nowRuleSet == RuleSet.Level1)
      {
        console.log("hint lilypad", hintLilypad)
        
        // Animated glow effect
        const glowIntensity = 0.5 + 0.5 * Math.sin(Date.now() * 0.005)
        ctx.strokeStyle = `rgba(255, 255, 0, ${glowIntensity})`
        ctx.lineWidth = 4
        ctx.beginPath()
        ctx.ellipse(pad.x, pad.y, drawleafRadiusX * 1.1, drawleafRadiusY * 1.1, 0, 0, Math.PI * 2)
        ctx.stroke()

        // Hint text
        ctx.fillStyle = '#FFFF00'
        ctx.font = ' 1rem Arial'
        ctx.textAlign = 'center'
        ctx.fillText(`Count by 2s are: 2, 4, 6, 8...`, 0.5 * ctx.canvas.clientWidth, 0.16 * ctx.canvas.clientHeight)
      }

      
      // Number background circle
      ctx.fillStyle = 'rgba(255, 255, 255, 0.95)'
      ctx.beginPath()
      ctx.arc(pad.x, pad.y, drawleafRadiusY * 0.45, 0, Math.PI * 2)
      ctx.fill()
      
      ctx.strokeStyle = '#006400'
      ctx.lineWidth = 2
      ctx.stroke()

      // Draw the number
      ctx.fillStyle = '#006400'
      ctx.font = `bold 1rem Arial`
      ctx.textAlign = 'center'
      ctx.textBaseline = 'middle'
      ctx.fillText(pad.number, pad.x, pad.y)


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
  const frogRadius = 0.06 * ctx.canvas.clientHeight
  
  ctx.save()

  // Draw shadow for depth
  ctx.fillStyle = 'rgba(0, 0, 0, 0.3)'
  ctx.beginPath()
  ctx.ellipse(frog.x + 3, frog.y + 8, frogRadius * 0.8, frogRadius * 0.4, 0, 0, Math.PI * 2)
  ctx.fill()
  ctx.restore()
  
  // Draw main frog body with gradient
  const gradient = ctx.createRadialGradient(frog.x - 10, frog.y - 10, 0, frog.x, frog.y, frogRadius)
  gradient.addColorStop(0, '#90EE90')
  //gradient.addColorStop(1, '#228B22')
  
  ctx.fillStyle = gradient
  ctx.beginPath()
  ctx.arc(frog.x, frog.y, frogRadius, 0, Math.PI * 2)
  ctx.fill()
  
  // Draw frog outline
  ctx.strokeStyle = '#006400'
  ctx.lineWidth = 2
  ctx.stroke()
  
  // Draw eyes
  const eyeRadius = frogRadius * 0.25
  const eyeOffset = frogRadius * 0.4
  
  // Left eye
  ctx.fillStyle = '#FFFFFF'
  ctx.beginPath()
  ctx.arc(frog.x - eyeOffset, frog.y - eyeOffset, eyeRadius, 0, Math.PI * 2)
  ctx.fill()
  ctx.strokeStyle = '#000000'
  ctx.lineWidth = 1
  ctx.stroke()
  
  // Right eye
  ctx.beginPath()
  ctx.arc(frog.x + eyeOffset, frog.y - eyeOffset, eyeRadius, 0, Math.PI * 2)
  ctx.fill()
  ctx.stroke()
  
  // Draw pupils
  ctx.fillStyle = '#000000'
  ctx.beginPath()
  ctx.arc(frog.x - eyeOffset, frog.y - eyeOffset, eyeRadius * 0.5, 0, Math.PI * 2)
  ctx.fill()
  ctx.beginPath()
  ctx.arc(frog.x + eyeOffset, frog.y - eyeOffset, eyeRadius * 0.5, 0, Math.PI * 2)
  ctx.fill()
  
  
  // Draw number on frog's belly with background
  ctx.save()
   if(currentGameState != GameState.Reward)
  {
  // Number background circle
  ctx.fillStyle = 'rgba(255, 255, 255, 0.9)'
  ctx.beginPath()
  ctx.arc(frog.x, frog.y + 5, frogRadius * 0.6, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.strokeStyle = '#228B22'
  ctx.lineWidth = 2
  ctx.stroke()

 
  // Draw the number
  ctx.fillStyle = '#006400'
  ctx.font = `bold 1rem Arial`
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText(currentNumber, frog.x, frog.y + 5)
  }
  
  ctx.restore()
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
  frog.x = 0.5 * ctx.canvas.clientWidth
  frog.y = 0.48 * ctx.canvas.clientHeight
  
  drawFrog(ctx)

  // Draw tongue if extended
  if (fly.tongueExtended) 
  {
    ctx.strokeStyle = 'red'
    ctx.lineWidth = 4
    ctx.beginPath()
    ctx.moveTo(frog.x, frog.y)
    ctx.lineTo(fly.x, fly.y)
    ctx.stroke()
  }
}

function bestowReward()
{
  streak++
  if(hintLilypad > 0)
    hintLilypad--
  
  
  if (streak% 2 ==0 && streak > 0) 
  {
    currentGameState = GameState.Reward
    LilypadMovementSpeed += 0.2
    streak = 0 // Reset streak after reward
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

  if (frog.jumping) 
    return

    frog.jumpTarget = pad
  frog.jumping = true

  const frames = 20
  frog.velocityX = (pad.x - frog.x) / frames
  frog.velocityY = (pad.y - frog.y - 50) / frames // slight arc
}

function forcedFrogJump()
{
  hintLilypad = 3
  for (let pad of lilypads[nextcolumn])
  {
    if (pad.isCorrect) 
    {
      frog.stickpad = pad
      jumpTo(pad)
      nextcolumn++
      currentNumber += skipStep
      frog.lives--
      showCrocodile() 
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
  const startX = ctx.canvas.clientWidth - heartSize *3 - heartSpacing
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

function drawHeart(ctx, x, y, size, filled) 
{
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

function drawTask(ctx)
{
  ctx.fillStyle = 'white'
  ctx.font = '1rem Arial'
  ctx.textAlign = 'center'
  
  if(nowRuleSet == RuleSet.Level3)
    ctx.fillText(`Task: Skip by ${skipStep}s`, 0.5* ctx.canvas.clientWidth,  0.08* ctx.canvas.clientHeight)
  else 
    ctx.fillText(`Task: Count by ${skipStep}s`, 0.5* ctx.canvas.clientWidth,  0.08* ctx.canvas.clientHeight)
  
}

// titleX, titleY,bodyX and bodyYare relative positions to the screen
function drawScreenOverlay(ctx,title,body = "",titleX = 0.5,titleY = 0.45, bodyX =0.5,bodyY= 0.27)
{
  showLevelSelect.value = (currentGameState == GameState.LevelSelect)
  showResume.value = (currentGameState == GameState.Pause)
  showRetry.value = (currentGameState == GameState.GameOver)
  showRewardContinue.value = (currentGameState == GameState.Reward || currentGameState == GameState.Distracted)

  ctx.textAlign = 'center'

  ctx.fillStyle = 'rgba(0,0,0,0.6)'
  ctx.fillRect(0, 0, ctx.canvas.clientWidth, ctx.canvas.clientHeight)

  ctx.fillStyle = 'white'
  ctx.font = '1.8rem Arial'
  ctx.fillText(title, titleX* ctx.canvas.clientWidth, titleY * ctx.canvas.clientHeight)

  ctx.font = '1.2rem Arial'
  ctx.fillText(body, bodyX * ctx.canvas.clientWidth, bodyY * ctx.canvas.clientHeight)

  clearInterval(lilypadTimer)
  clearInterval(AttachBeepadTimer)
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
      if(nowRuleSet == RuleSet.Level3)
      {
        lilypadColumn[i].isCorrect = (allNumbers[i] !=correctNumber)
      }
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
      if(nowRuleSet == RuleSet.Level3)
      {
        lilypadColumn[i].isCorrect = (allNumbers[i] !=correctNumber)
      }
    }
    lilypads.push(lilypadColumn)
  }
}

function drawRewardScreen(ctx) 
{
  drawScreenOverlay(ctx, 'Excellent!', 'You’re counting like a pro! Keep Going',0.5,0.2,0.5,0.27)

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
  currentGameState = GameState.LevelSelect
  currentNumber = 2
  frog.x = 0.1 * document.documentElement.clientWidth
  frog.y = 0.4 * document.documentElement.clientHeight
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
  hintLilypad = 0
}

function updateJumpArc()
{
  if (frog.jumping) 
  {

    frog.x += frog.velocityX
    frog.y += frog.velocityY

    frog.velocityY += gravity

    // Stop when near target
    const dx = Math.abs(frog.x - frog.jumpTarget.x)
    const dy = Math.abs(frog.y - frog.jumpTarget.y)
    
    if (dx < 50 && dy < 50 && frog.jumpTarget.isCorrect) 
    {
      bestowReward()
      frog.x = frog.stickpad.x
      frog.y = frog.stickpad.y
      frog.velocityX = 0
      frog.velocityY = 0
      frog.jumping = false
    }
    else if (dx< 50 && dy < 50 && !frog.jumpTarget.isCorrect && frog.stickpad != null) 
    {
      const frames = 20
      frog.jumpTarget = frog.stickpad
      frog.velocityX = (frog.stickpad.x - frog.x) / frames
      frog.velocityY = (frog.stickpad.y - frog.y - 50) / frames // slight arc
    }
    else if(dx< 50 && dy < 50 && !frog.jumpTarget.isCorrect && frog.jumpTarget.number!=-1)
    {
      const frames = 20
      let dummyLilypad = new Lilypad(0.1 * document.documentElement.clientWidth, 0.4 * document.documentElement.clientHeight, -1, false)
      frog.jumpTarget = dummyLilypad
      frog.velocityX = (dummyLilypad.x- frog.x) / frames
      frog.velocityY = (dummyLilypad.y - frog.y - 50) /frames// slight arc
    }
    else if(dx <50&& dy < 50 && frog.jumpTarget.number == -1 )
    {
      frog.x = frog.jumpTarget.x
      frog.y = frog.jumpTarget.y
      frog.jumping = false
      frog.jumpTarget = null
      frog.velocityX = 0
      frog.velocityY = 0
      
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
  if(currentGameState != GameState.Reward)
    drawFrog(ctx)
  drawCrocodile(ctx)
  drawHUD(ctx)
  drawTask(ctx )

  
  if(beepad)
    drawDistraction(ctx)
  if( currentGameState == GameState.LevelSelect)
    drawScreenOverlay(ctx, 'Select Level', 'Click to start the game', 0.5, 0.2, 0.5, 0.27)
  else if (currentGameState == GameState.GameOver) 
    drawScreenOverlay(ctx, 'Game Over')
  else if (  currentGameState == GameState.Pause)
    drawScreenOverlay(ctx,'Game Paused')
  else if (currentGameState == GameState.Reward)
    drawRewardScreen(ctx)
  else if( currentGameState == GameState.Distracted)
    drawScreenOverlay(ctx, 'Distracted!', 'You got distracted by a bee! Try again',0.5,0.2,0.5,0.27)
 
  requestAnimationFrame(render)
 
}

function gameUpdate()
{
  //to move the lilypads from right to left
  updateLilypadPosition()
  //to make the frog jump along with the lilypad or before it runs out of the screen
  updateFrogPosition()
  updateJumpArc()
  updateCrocodile()

  requestAnimationFrame(gameUpdate)
  
}

onMounted(() => 
{
  const canvas = gameCanvas.value
  
 
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
          {
            currentGameState=GameState.Distracted

          }
          else{
          frog.lives -= 1
showCrocodile()}
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