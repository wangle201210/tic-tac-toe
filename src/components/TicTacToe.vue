<script setup lang="ts">
import { ElMessage } from 'element-plus'
import { computed, ref } from 'vue'

// 游戏状态
const board = ref<Array<string | null>>(Array.from({ length: 9 }, () => null))
const currentPlayer = ref<'X' | 'O'>('X')
const gameOver = ref(false)
const winner = ref<string | null>(null)
const moveHistory = ref<Array<{ position: number, player: 'X' | 'O', moveNumber: number }>>([])
const moveCount = ref(0)

// 获取格子状态
function getCellState(index: number) {
  // 找到该位置最新的移动记录
  const moves = moveHistory.value.filter(m => m.position === index)
  if (moves.length === 0)
    return 'empty'

  const latestMove = moves[moves.length - 1]
  const currentMoveNumber = moveCount.value
  const moveNumber = latestMove.moveNumber

  // 根据规则：落第6子时第1子变暗，落第7子时第1子消失第2子变暗
  // 当前步数 <= 5 时，所有棋子都正常显示
  if (currentMoveNumber <= 5) {
    return 'normal'
  }

  // 当前步数 = 6 时，第1子变暗
  if (currentMoveNumber === 6) {
    return moveNumber === 1 ? 'dim' : 'normal'
  }

  // 当前步数 >= 7 时，按规则计算
  const disappearCount = currentMoveNumber - 6 // 应该消失的棋子数量
  const dimMoveNumber = disappearCount + 1 // 应该变暗的棋子步数

  if (moveNumber <= disappearCount) {
    return 'disappeared'
  }
  else if (moveNumber === dimMoveNumber) {
    return 'dim'
  }
  else {
    return 'normal'
  }
}

// 获取格子显示的内容
function getCellContent(index: number) {
  const state = getCellState(index)
  if (state === 'empty' || state === 'disappeared') {
    return ''
  }
  // 找到该位置最新的移动记录
  const moves = moveHistory.value.filter(m => m.position === index)
  if (moves.length === 0) {
    return ''
  }
  // 返回最新的移动记录
  const latestMove = moves[moves.length - 1]
  return latestMove.player
}

// 获取格子的CSS类
function getCellClass(index: number) {
  const state = getCellState(index)
  const baseClass = 'cell'

  switch (state) {
    case 'dim':
      return `${baseClass} dim`
    case 'disappeared':
      return `${baseClass} disappeared`
    case 'normal':
      return baseClass
    default:
      return baseClass
  }
}

// 检查获胜条件
function checkWinner() {
  const winPatterns = [
    [
      0,
      1,
      2,
    ],
    [
      3,
      4,
      5,
    ],
    [
      6,
      7,
      8,
    ], // 行
    [
      0,
      3,
      6,
    ],
    [
      1,
      4,
      7,
    ],
    [
      2,
      5,
      8,
    ], // 列
    [
      0,
      4,
      8,
    ],
    [
      2,
      4,
      6,
    ], // 对角线
  ]

  // 获取当前可见的棋盘状态（排除消失的棋子）
  const visibleBoard: Array<string | null> = Array.from({ length: 9 }, () => null)
  for (let i = 0; i < 9; i++) {
    const state = getCellState(i)
    if (state !== 'empty' && state !== 'disappeared') {
      visibleBoard[i] = getCellContent(i)
    }
  }

  for (const pattern of winPatterns) {
    const [a, b, c] = pattern
    if (visibleBoard[a] && visibleBoard[a] === visibleBoard[b] && visibleBoard[a] === visibleBoard[c]) {
      return visibleBoard[a]
    }
  }
  return null
}

// 处理点击事件
function handleCellClick(index: number) {
  if (gameOver.value)
    return

  const state = getCellState(index)
  // 只有空位置和已消失的位置可以下棋
  if (state !== 'empty' && state !== 'disappeared')
    return

  // 下棋
  moveCount.value++
  moveHistory.value.push({
    position: index,
    player: currentPlayer.value,
    moveNumber: moveCount.value,
  })

  // 检查是否获胜
  const gameWinner = checkWinner()
  if (gameWinner) {
    winner.value = gameWinner
    gameOver.value = true
    ElMessage.success(`玩家 ${gameWinner} 获胜！`)
    return
  }

  // 注意：由于棋子会消失，消失的位置可以重新下棋，所以不存在平局的情况

  // 切换玩家
  currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'
}

// 获取历史记录项的状态
function getHistoryItemState(moveNumber: number) {
  const currentMoveNumber = moveCount.value

  // 根据规则：落第6子时第1子变暗，落第7子时第1子消失第2子变暗
  // 当前步数 <= 5 时，所有棋子都正常显示
  if (currentMoveNumber <= 5) {
    return 'normal'
  }

  // 当前步数 = 6 时，第1子变暗
  if (currentMoveNumber === 6) {
    return moveNumber === 1 ? 'dim' : 'normal'
  }

  // 当前步数 >= 7 时，按规则计算
  const disappearCount = currentMoveNumber - 6 // 应该消失的棋子数量
  const dimMoveNumber = disappearCount + 1 // 应该变暗的棋子步数

  if (moveNumber <= disappearCount) {
    return 'disappeared'
  }
  else if (moveNumber === dimMoveNumber) {
    return 'dim'
  }
  else {
    return 'normal'
  }
}

// 重置游戏
function resetGame() {
  board.value = Array.from({ length: 9 }, () => null)
  currentPlayer.value = 'X'
  gameOver.value = false
  winner.value = null
  moveHistory.value = []
  moveCount.value = 0
  ElMessage.info('游戏已重置')
}

// 游戏状态文本
const gameStatusText = computed(() => {
  if (winner.value) {
    return `玩家 ${winner.value} 获胜！`
  }
  return `当前玩家: ${currentPlayer.value}`
})
</script>

<template>
  <div class="tic-tac-toe">
    <h1>井字棋游戏</h1>

    <div class="game-info">
      <p class="status">
        {{ gameStatusText }}
      </p>
      <p class="rules">
        规则：双方轮流下棋，X先行，O后行，先完成三子连线者获胜！
      </p>
      <p class="special-rules">
        特殊规则：屏幕仅显示最新6子位置，其余自动消除
      </p>
    </div>

    <div class="game-board">
      <div
        v-for="(cell, index) in 9"
        :key="index"
        :class="getCellClass(index)"
        @click="handleCellClick(index)"
      >
        <span class="cell-content">{{ getCellContent(index) }}</span>
      </div>
    </div>

    <div class="game-controls">
      <el-button type="primary" @click="resetGame">
        重新开始
      </el-button>
    </div>

    <div class="move-history">
      <h3>走棋历史</h3>
      <div class="history-list">
        <div v-if="moveHistory.length === 0" class="no-history">
          暂无走棋记录
        </div>
        <div
          v-for="move in moveHistory.slice().reverse()"
          :key="move.moveNumber"
          class="history-item"
          :class="getHistoryItemState(move.moveNumber)"
        >
          第{{ move.moveNumber }}步: {{ move.player }} 在位置 {{ move.position + 1 }}
          <span v-if="getHistoryItemState(move.moveNumber) === 'dim'">(变暗)</span>
          <span v-if="getHistoryItemState(move.moveNumber) === 'disappeared'">(已消失)</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.tic-tac-toe {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.game-info {
  margin-bottom: 20px;
}

.status {
  font-size: 18px;
  font-weight: bold;
  color: var(--ep-color-primary);
  margin-bottom: 10px;
}

.rules,
.special-rules {
  font-size: 14px;
  color: var(--ep-text-color-regular);
  margin: 5px 0;
}

.special-rules {
  color: var(--ep-color-warning);
  font-weight: bold;
}

.game-board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2px;
  width: 400px;
  height: 400px;
  margin: 20px auto;
  border: 2px solid var(--ep-border-color);
  border-radius: 8px;
  overflow: hidden;
}

.cell {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--ep-bg-color);
  border: 1px solid var(--ep-border-color-light);
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 50px;
  font-weight: bold;
  user-select: none;
  width: 100%;
  height: 100%;
  min-height: 96px;
  box-sizing: border-box;
}

.cell:hover {
  background-color: var(--ep-fill-color-light);
}

.cell.dim {
  opacity: 0.4;
  background-color: var(--ep-fill-color-darker);
}

.cell.disappeared {
  background-color: var(--ep-fill-color-blank);
}

.cell.disappeared .cell-content {
  opacity: 0;
}

.cell-content {
  transition: all 0.3s ease;
}

.game-controls {
  margin: 20px 0;
}

.move-history {
  margin-top: 30px;
  text-align: left;
}

.move-history h3 {
  text-align: center;
  color: var(--ep-text-color-primary);
  margin-bottom: 15px;
}

.history-list {
  max-height: 200px;
  overflow-y: auto;
  border: 1px solid var(--ep-border-color);
  border-radius: 4px;
  padding: 10px;
}

.history-item {
  padding: 5px 0;
  border-bottom: 1px solid var(--ep-border-color-lighter);
  font-size: 14px;
}

.history-item:last-child {
  border-bottom: none;
}

.history-item.dim {
  color: var(--ep-text-color-disabled);
  opacity: 0.6;
}

.history-item.disappeared {
  color: var(--ep-text-color-placeholder);
  text-decoration: line-through;
  opacity: 0.4;
}

.no-history {
  padding: 5px 0;
  font-size: 14px;
  color: var(--ep-text-color-placeholder);
  text-align: center;
  font-style: italic;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .tic-tac-toe {
    padding: 10px;
  }

  .game-board {
    width: 350px;
    height: 350px;
  }

  .cell {
    font-size: 28px;
  }
}
</style>
