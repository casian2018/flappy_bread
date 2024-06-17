<template>
    <div class="flex items-center justify-center h-screen bg-blue-100" @click="jump">
      <div ref="game" class="relative w-full h-full overflow-hidden bg-blue-300">
        <div
          v-for="(obstacle, index) in obstacles"
          :key="index"
          :style="obstacle.style"
          class="absolute w-12 bg-green-600 rounded"
        ></div>
        <div
          v-if="gameRunning"
          ref="bread"
          :style="breadStyle"
          class="absolute w-12 h-12 bg-yellow-400 rounded-full"
        ></div>
        <div v-if="!gameRunning" class="absolute inset-0 flex items-center justify-center bg-blue-300 bg-opacity-75">
          <div class="p-8 text-center bg-white rounded shadow-lg">
            <h1 class="mb-4 text-2xl font-bold text-blue-600">Flappy Bread</h1>
            <p v-if="!gameStarted" class="mb-4 text-lg">Click the button to start the game.</p>
            <p v-if="gameStarted" class="mb-4 text-lg">Game Over! Click the button to restart.</p>
            <button
              @click="startGame"
              class="px-6 py-2 text-white bg-blue-600 rounded hover:bg-blue-700"
            >
              {{ gameStarted ? 'Restart Game' : 'Start Game' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, computed, nextTick } from 'vue'
  
  const bread = ref(null)
  const game = ref(null)
  const obstacles = ref([])
  const breadPosition = ref({ top: 200, left: 50 })
  const breadVelocity = ref(0)
  const gravity = 1
  const jumpStrength = -20
  const obstacleSpeed = 3
  const obstacleGap = 500
  const minGap = 100 // Minimum gap between top and bottom obstacles
  const gameRunning = ref(false)
  const gameStarted = ref(false)
  
  const breadStyle = computed(() => ({
    top: `${breadPosition.value.top}px`,
    left: `${breadPosition.value.left}px`,
  }))
  
  const createObstacle = () => {
    const maxHeight = game.value.clientHeight - obstacleGap - minGap
    const height = Math.random() * maxHeight
    const position = game.value.clientWidth
    obstacles.value.push(
      {
        style: {
          top: '0px',
          height: `${height}px`,
          right: `-${obstacleSpeed}px`,
        },
        passed: false,
      },
      {
        style: {
          bottom: '0px',
          height: `${game.value.clientHeight - height - obstacleGap}px`,
          right: `-${obstacleSpeed}px`,
        },
        passed: false,
      }
    )
  }
  
  const updateObstacles = () => {
    obstacles.value.forEach(obstacle => {
      const right = parseFloat(obstacle.style.right)
      obstacle.style.right = `${right + obstacleSpeed}px`
    })
    if (obstacles.value.length && parseFloat(obstacles.value[0].style.right) > game.value.clientWidth) {
      obstacles.value.splice(0, 2)
    }
    if (obstacles.value.length < 4) {
      createObstacle()
    }
  }
  
  const checkCollision = () => {
    if (!bread.value) return false
    const breadRect = bread.value.getBoundingClientRect()
    if (breadRect.top <= 0 || breadRect.bottom >= game.value.clientHeight) {
      return true
    }
    for (const obstacle of obstacles.value) {
      const obstacleElement = document.querySelector(`[style*="right: ${obstacle.style.right}"]`)
      if (obstacleElement) {
        const obstacleRect = obstacleElement.getBoundingClientRect()
        if (
          breadRect.right > obstacleRect.left &&
          breadRect.left < obstacleRect.right &&
          breadRect.bottom > obstacleRect.top &&
          breadRect.top < obstacleRect.bottom
        ) {
          return true
        }
      }
    }
    return false
  }
  
  const updateBreadPosition = () => {
    breadVelocity.value += gravity
    breadPosition.value.top += breadVelocity.value
    if (checkCollision()) {
      stopGame()
    }
  }
  
  const gameLoop = () => {
    if (!gameRunning.value) return
    updateBreadPosition()
    updateObstacles()
    requestAnimationFrame(gameLoop)
  }
  
  const jump = () => {
    if (gameRunning.value) {
      breadVelocity.value = jumpStrength
    }
  }
  
  const startGame = async () => {
    breadPosition.value = { top: 200, left: 50 }
    breadVelocity.value = 0
    obstacles.value = []
    gameRunning.value = true
    gameStarted.value = true
    createObstacle()
    await nextTick()
    gameLoop()
  }
  
  const stopGame = () => {
    gameRunning.value = false
  }
  
  onMounted(() => {
    window.addEventListener('click', jump)
    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space') {
        jump()
      }
    })
  })
  </script>
  
  <style scoped>
  .bread {
    width: 50px;
    height: 50px;
    background-color: yellow;
    position: absolute;
    border-radius: 50%;
  }
  .obstacle {
    width: 50px;
    position: absolute;
    background-color: green;
    border-radius: 5px;
  }
  </style>
  