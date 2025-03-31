// 写一个贪吃蛇页面，用vue3 ts 

<template>
  <div class="snake-game">
    <div class="game-info">
      <div class="score">得分: {{ score }}</div>
      <button @click="startGame" :disabled="isPlaying">开始游戏</button>
    </div>
    <div 
      class="game-board"
      :style="{ 
        gridTemplateRows: `repeat(${boardSize}, 1fr)`,
        gridTemplateColumns: `repeat(${boardSize}, 1fr)`
      }"
    >
      <div 
        v-for="(row, rowIndex) in board" 
        v-bind:key="rowIndex"
        class="row"
      >
        <div 
          v-for="(cell, colIndex) in row" 
          v-bind:key="colIndex"
          class="cell"
          :class="{
            'snake': isSnake(rowIndex, colIndex),
            'food': isFood(rowIndex, colIndex)
          }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

// 类型定义
type Position = {
  x: number
  y: number
}

type Direction = 'UP' | 'DOWN' | 'LEFT' | 'RIGHT'

// 游戏配置
const boardSize = 20
const initialSpeed = 200

// 游戏状态
const score = ref(0)
const isPlaying = ref(false)
const snake = ref<Position[]>([{ x: 10, y: 10 }])
const food = ref<Position>({ x: 5, y: 5 })
const direction = ref<Direction>('RIGHT')
const board = ref<number[][]>(Array(boardSize).fill(0).map(() => Array(boardSize).fill(0)))

// 游戏控制
let gameInterval: number | undefined

// 开始游戏
const startGame = () => {
  isPlaying.value = true
  score.value = 0
  snake.value = [{ x: 10, y: 10 }]
  direction.value = 'RIGHT'
  generateFood()
  gameInterval = setInterval(moveSnake, initialSpeed)
}

// 生成食物
const generateFood = () => {
  let newFood: Position
  do {
    newFood = {
      x: Math.floor(Math.random() * boardSize),
      y: Math.floor(Math.random() * boardSize)
    }
  } while (snake.value.some(segment => segment.x === newFood.x && segment.y === newFood.y))
  food.value = newFood
}

// 移动蛇
const moveSnake = () => {
  const head = { ...snake.value[0] }

  switch (direction.value) {
    case 'UP':
      head.y--
      break
    case 'DOWN':
      head.y++
      break
    case 'LEFT':
      head.x--
      break
    case 'RIGHT':
      head.x++
      break
  }

  // 检查游戏结束条件
  if (
    head.x < 0 || 
    head.x >= boardSize || 
    head.y < 0 || 
    head.y >= boardSize ||
    snake.value.some(segment => segment.x === head.x && segment.y === head.y)
  ) {
    endGame()
    return
  }

  // 移动蛇
  snake.value.unshift(head)

  // 检查是否吃到食物
  if (head.x === food.value.x && head.y === food.value.y) {
    score.value += 10
    generateFood()
  } else {
    snake.value.pop()
  }
}

// 结束游戏
const endGame = () => {
  isPlaying.value = false
  clearInterval(gameInterval)
  alert(`游戏结束！得分：${score.value}`)
}

// 键盘控制
const handleKeydown = (event: KeyboardEvent) => {
  if (!isPlaying.value) return

  switch (event.key) {
    case 'ArrowUp':
      if (direction.value !== 'DOWN') direction.value = 'UP'
      break
    case 'ArrowDown':
      if (direction.value !== 'UP') direction.value = 'DOWN'
      break
    case 'ArrowLeft':
      if (direction.value !== 'RIGHT') direction.value = 'LEFT'
      break
    case 'ArrowRight':
      if (direction.value !== 'LEFT') direction.value = 'RIGHT'
      break
  }
}

// 判断是否是蛇身
const isSnake = (row: number, col: number): boolean => {
  return snake.value.some(segment => segment.x === col && segment.y === row)
}

// 判断是否是食物
const isFood = (row: number, col: number): boolean => {
  return food.value.x === col && food.value.y === row
}

// 生命周期钩子
onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  clearInterval(gameInterval)
})
</script>

<style scoped>
.snake-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 20px;
}

.game-info {
  display: flex;
  gap: 20px;
  align-items: center;
}

.game-board {
  display: grid;
  width: 400px;
  height: 400px;
  border: 2px solid #333;
}

.cell {
  border: 1px solid #eee;
  background-color: white;
}

.snake {
  background-color: #4CAF50;
}

.food {
  background-color: #F44336;
}

button {
  padding: 8px 16px;
  background-color: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>


