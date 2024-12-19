<template>
  <div class="game-container">
    <h1>井字棋</h1>
    <div class="timer" v-if="isCountingDown">{{ timeLeft }} 秒</div>
    <div class="board-row" v-for="row in 3" :key="row">
      <button class="cell" :class="{ [getCellValue(row, col)]: true }" v-for="col in 3" :key="col"
        @click="makeMove(row, col)" :disabled="!isGameStarted">{{ getCellValue(row, col) }}</button>
    </div>
    <div class="button-group">
      <button class="start-button" @click="startGame">开始游戏</button>
      <button class="reset-button" @click="resetGame">重置游戏</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, ref } from 'vue';
import { ElMessage } from 'element-plus';
export default defineComponent({
  name: 'TicTacToe',
  setup() {
    const board = ref<number[][]>(Array.from({ length: 3 }, () => Array(3).fill(0)));
    const currentPlayer = ref(1);
    const winner = ref<number | null>(null);
    const isCountingDown = ref(false);
    const timeLeft = ref(15);
    const isGameStarted = ref(false);
    const timer = reactive({
      intervalId: 0,
    });

    const startCountdown = () => {
      isCountingDown.value = true;
      timeLeft.value = 15;
      clearInterval(timer.intervalId);
      timer.intervalId = setInterval(() => {
        timeLeft.value--;
        if (timeLeft.value <= 0) {
          clearInterval(timer.intervalId);
          isCountingDown.value = false;
          if (currentPlayer.value === 1) {
            winner.value = 2; 
          } else {
            winner.value = 1; 
          }
          ElMessage.success(`时间到，玩家 ${winner.value} 获胜！`);
        }
      }, 1000);
    };
    const startGame = () => {
      isGameStarted.value = true;
      board.value = Array.from({ length: 3 }, () => Array(3).fill(0));
      currentPlayer.value = 1;
      winner.value = null;
      isCountingDown.value = false;
      clearInterval(timer.intervalId);
      startCountdown();
    };

    // 在 checkWinner 函数中添加平局判断
    const checkWinner = (board: number[][]) => {
      for (let i = 0; i < 3; i++) {
        if (board[i][0] === board[i][1] && board[i][1] === board[i][2] && board[i][0] !== 0) {
          return board[i][0];
        }
        if (board[0][i] === board[1][i] && board[1][i] === board[2][i] && board[0][i] !== 0) {
          return board[0][i];
        }
      }
      if (board[0][0] === board[1][1] && board[1][1] === board[2][2] && board[0][0] !== 0) {
        return board[0][0];
      }
      if (board[0][2] === board[1][1] && board[1][1] === board[2][0] && board[0][2] !== 0) {
        return board[0][2];
      }

      // 检查是否平局
      let isDraw = true;
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          if (board[i][j] === 0) {
            isDraw = false;
            break;
          }
        }
        if (!isDraw) {
          break;
        }
      }
      if (isDraw) {
        return 0; // 0 表示平局
      }

      return null;
    };

    // 在 makeMove 函数中添加游戏结果显示
    const makeMove = (row: number, col: number) => {
      if (isGameStarted.value && !winner.value && board.value[row - 1][col - 1] === 0) {
        board.value[row - 1][col - 1] = currentPlayer.value;
        currentPlayer.value = currentPlayer.value === 1 ? 2 : 1;
        winner.value = checkWinner(board.value);
        startCountdown();
        
        // 显示游戏结果
        if (winner.value === 1 || winner.value === 2) {
          clearInterval(timer.intervalId);
          isCountingDown.value = false;
          ElMessage.success(`玩家 ${winner.value} 获胜！`);
        } else if (winner.value === 0) {
          clearInterval(timer.intervalId);
          isCountingDown.value = false;
          ElMessage.info('平局！');
        }
      }
    };

    // 添加重置游戏函数
    const resetGame = () => {
      board.value = Array.from({ length: 3 }, () => Array(3).fill(0));
      currentPlayer.value = 1;
      winner.value = null;
      isGameStarted.value = true;
      isCountingDown.value = true;
      clearInterval(timer.intervalId);
    };

    const getCellValue = (row: number, col: number) => {
      const cellValue = board.value[row - 1][col - 1];
      return cellValue === 1 ? 'X' : cellValue === 2 ? 'O' : '';
    };

    return {
      board,
      currentPlayer,
      winner,
      isCountingDown,
      timeLeft,
      isGameStarted,
      makeMove,
      getCellValue,
      resetGame,
      startGame,
    };
  },
});
</script>

<style scoped>
.game-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

.board-row {
  display: flex;
}

.cell {
  width: 100px;
  height: 100px;
  border: 1px solid #ddd;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 50px;
  cursor: pointer;
  margin: 5px;
}

.cell.X {
  color: #f00;
}

.cell.O {
  color: #00f;
}

.start-button {
  padding: 10px 20px;
  background-color: #2196F3;
  color: white;
  border: none;
  cursor: pointer;
}

.start-button:hover {
  background-color: #1976D2;
}

.reset-button {
    padding: 10px 20px;
    background-color: #f44336;
    color: white;
    border: none;
    cursor: pointer;
}

.reset-button:hover {
    background-color: #d32f2f;
}

.timer {
  position: absolute;
  top: 210px;
  right: 10px;
  margin-top: 10px;
  font-size: 20px;
}

.button-group {
  width: 400px;
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-top: 20px;
}
</style>