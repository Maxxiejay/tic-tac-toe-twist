<template>
  <div class="min-h-screen bg-gradient-to-br from-indigo-50 to-purple-50 flex items-center justify-center p-4">
    <div class="w-full max-w-md mx-auto">
      <!-- Game Header -->
      <div class="text-center mb-8">
        <h1 class="text-3xl font-bold text-gray-800 mb-2">Tic-Tac-Toe</h1>
        <h2 class="text-lg font-semibold text-indigo-600">Final Swap Edition</h2>
      </div>

      <!-- Game Status -->
      <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <div class="text-center">
          <div v-if="gamePhase === 'placement'" class="space-y-2">
            <div class="text-sm font-medium text-gray-600">Placement Phase</div>
            <div class="text-lg font-bold" :class="currentPlayer === 'X' ? 'text-blue-600' : 'text-red-600'">
              Player {{ currentPlayer }}'s Turn
            </div>
            <div class="text-xs text-gray-500">Move {{ moveCount }}/9</div>
          </div>
          
          <div v-else-if="gamePhase === 'swap'" class="space-y-2">
            <div class="text-sm font-medium text-purple-600">Final Swap Phase</div>
            <div class="text-lg font-bold" :class="swapPlayer === 'X' ? 'text-blue-600' : 'text-red-600'">
              Player {{ swapPlayer }} Must Swap
            </div>
            <div class="text-xs text-gray-500">
              {{ swapStep === 'select-own' ? 'Select your piece to swap' : 'Select opponent\'s piece to swap with' }}
            </div>
          </div>
          
          <div v-else-if="gamePhase === 'finished'" class="space-y-2">
            <div class="text-sm font-medium text-green-600">Game Over</div>
            <div class="text-xl font-bold" :class="getWinnerColor()">
              {{ getGameResult() }}
            </div>
          </div>
        </div>
      </div>

      <!-- Game Board -->
      <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <div class="grid grid-cols-3 gap-2 aspect-square">
          <button
            v-for="(cell, index) in board"
            :key="index"
            @click="handleCellClick(index)"
            :disabled="gamePhase === 'finished'"
            class="aspect-square border-2 border-gray-300 rounded-lg flex items-center justify-center text-4xl font-bold transition-all duration-200 hover:border-gray-400 focus:outline-none focus:ring-2 focus:ring-indigo-500"
            :class="getCellClasses(index)"
          >
            <span 
              class="transition-all duration-300 transform"
              :class="getCellTextClasses(index)"
            >
              {{ cell }}
            </span>
          </button>
        </div>
      </div>

      <!-- Swap Selection Info -->
      <div v-if="gamePhase === 'swap'" class="bg-purple-50 rounded-lg p-4 mb-6 border border-purple-200">
        <div class="text-center">
          <div v-if="selectedOwnPiece !== null" class="text-sm text-purple-700 mb-2">
            Selected your piece at position {{ selectedOwnPiece + 1 }}
          </div>
          <div class="text-xs text-purple-600">
            {{ swapStep === 'select-own' ? 'Click on one of your pieces' : 'Now click on an opponent\'s piece to complete the swap' }}
          </div>
        </div>
      </div>

      <!-- Control Buttons -->
      <div class="flex gap-3">
        <button
          @click="resetGame"
          class="flex-1 bg-gray-600 hover:bg-gray-700 text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-gray-500"
        >
          New Game
        </button>
        <button
          v-if="gamePhase === 'swap' && selectedOwnPiece !== null"
          @click="cancelSwapSelection"
          class="bg-red-500 hover:bg-red-600 text-white font-semibold py-3 px-4 rounded-lg transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-red-400"
        >
          Cancel
        </button>
      </div>

      <!-- Rules Summary -->
      <div class="mt-8 bg-gray-50 rounded-lg p-4">
        <h3 class="font-semibold text-gray-800 mb-2">How to Play:</h3>
        <ul class="text-sm text-gray-600 space-y-1">
          <li>• <strong>Phase 1:</strong> Take turns placing X and O until board is full</li>
          <li>• <strong>Phase 2:</strong> Player who didn't make last move must swap one piece</li>
          <li>• <strong>Win:</strong> Have 3 in a row after the final swap</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// Game state
const board = ref(Array(9).fill(''))
const currentPlayer = ref('X')
const gamePhase = ref('placement') // 'placement', 'swap', 'finished'
const moveCount = ref(0)
const lastMovePlayer = ref('')
const swapPlayer = ref('')
const swapStep = ref('select-own') // 'select-own', 'select-opponent'
const selectedOwnPiece = ref(null)
const winner = ref('')
const winningLines = ref([])

// Handle cell clicks
const handleCellClick = (index) => {
  if (gamePhase.value === 'placement') {
    handlePlacementClick(index)
  } else if (gamePhase.value === 'swap') {
    handleSwapClick(index)
  }
}

// Handle placement phase clicks
const handlePlacementClick = (index) => {
  if (board.value[index] !== '') return
  
  board.value[index] = currentPlayer.value
  lastMovePlayer.value = currentPlayer.value
  moveCount.value++
  
  checkWinner()
  
  if (winner.value !== '' && winner.value !== 'draw') {
    gamePhase.value = 'finished'
    return
  }
  
  if (moveCount.value === 9) {
    // Board is full, enter swap phase
    swapPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'
    gamePhase.value = 'swap'
    swapStep.value = 'select-own'
  } else {
    // Continue placement phase
    currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'
  }
}

// Handle swap phase clicks
const handleSwapClick = (index) => {
  if (swapStep.value === 'select-own') {
    // Player must select one of their own pieces
    if (board.value[index] === swapPlayer.value) {
      selectedOwnPiece.value = index
      swapStep.value = 'select-opponent'
    }
  } else if (swapStep.value === 'select-opponent') {
    // Player must select an opponent's piece
    const opponentSymbol = swapPlayer.value === 'X' ? 'O' : 'X'
    if (board.value[index] === opponentSymbol) {
      // Perform the swap
      const temp = board.value[selectedOwnPiece.value]
      board.value[selectedOwnPiece.value] = board.value[index]
      board.value[index] = temp
      
      // Check for winner and end game
      checkWinner()
      gamePhase.value = 'finished'
    }
  }
}

// Cancel swap selection
const cancelSwapSelection = () => {
  selectedOwnPiece.value = null
  swapStep.value = 'select-own'
}

// Check for winner
const checkWinner = () => {
  const lines = [
    [0, 1, 2], [3, 4, 5], [6, 7, 8], // rows
    [0, 3, 6], [1, 4, 7], [2, 5, 8], // columns
    [0, 4, 8], [2, 4, 6] // diagonals
  ]
  
  const xWins = []
  const oWins = []
  
  lines.forEach(line => {
    const [a, b, c] = line
    if (board.value[a] && board.value[a] === board.value[b] && board.value[a] === board.value[c]) {
      if (board.value[a] === 'X') {
        xWins.push(line)
      } else {
        oWins.push(line)
      }
    }
  })
  
  winningLines.value = [...xWins, ...oWins]
  
  if (xWins.length > 0 && oWins.length > 0) {
    winner.value = 'draw'
  } else if (xWins.length > 0) {
    winner.value = 'X'
  } else if (oWins.length > 0) {
    winner.value = 'O'
  } else {
    winner.value = ''
  }
}

// Get cell classes for styling
const getCellClasses = (index) => {
  const classes = []
  
  if (gamePhase.value === 'swap') {
    if (swapStep.value === 'select-own' && board.value[index] === swapPlayer.value) {
      classes.push('border-purple-400 bg-purple-50 hover:bg-purple-100 cursor-pointer')
    } else if (swapStep.value === 'select-opponent' && board.value[index] === (swapPlayer.value === 'X' ? 'O' : 'X')) {
      classes.push('border-orange-400 bg-orange-50 hover:bg-orange-100 cursor-pointer')
    } else {
      classes.push('cursor-not-allowed opacity-50')
    }
    
    if (selectedOwnPiece.value === index) {
      classes.push('ring-2 ring-purple-500 bg-purple-100')
    }
  } else if (gamePhase.value === 'placement' && board.value[index] === '') {
    classes.push('hover:bg-gray-50 cursor-pointer')
  } else if (gamePhase.value === 'finished') {
    const isWinningCell = winningLines.value.some(line => line.includes(index))
    if (isWinningCell) {
      classes.push('bg-green-100 border-green-400')
    }
  }
  
  return classes.join(' ')
}

// Get cell text classes
const getCellTextClasses = (index) => {
  const symbol = board.value[index]
  if (symbol === 'X') {
    return 'text-blue-600'
  } else if (symbol === 'O') {
    return 'text-red-600'
  }
  return ''
}

// Get winner color
const getWinnerColor = () => {
  if (winner.value === 'X') return 'text-blue-600'
  if (winner.value === 'O') return 'text-red-600'
  return 'text-gray-600'
}

// Get game result text
const getGameResult = () => {
  if (winner.value === 'draw') return 'Draw!'
  return `Player ${winner.value} Wins!`
}

// Reset game
const resetGame = () => {
  board.value = Array(9).fill('')
  currentPlayer.value = 'X'
  gamePhase.value = 'placement'
  moveCount.value = 0
  lastMovePlayer.value = ''
  swapPlayer.value = ''
  swapStep.value = 'select-own'
  selectedOwnPiece.value = null
  winner.value = ''
  winningLines.value = []
}
</script>
