/* eslint-disable */
<template>
  <div class="container">
    <div class="controls">
      <div v-if="gameOver" class="game-over">
        <p>{{ winner }} wins!</p>
      </div>
      <div class="scoreboard">
        <p>Black: {{ blackCount }} --- White: {{ whiteCount }}</p>
      </div>
      <button class="restart-button" @click="restartGame">Restart Game</button>
    </div>
    <div class="chessboard">
      <div
        v-for="(row, rowIndex) in board"
        :key="rowIndex"
        class="row"
      >
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="cell"
          :class="{ black: cell === 'black', white: cell === 'white', 'flip-animation': cell === 'black' || cell === 'white' }"
          @click="placePiece(rowIndex, colIndex)"
          :style="{ cursor: currentPlayer === 'black' && isValidMove(rowIndex, colIndex) ? 'pointer' : 'not-allowed' }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      board: Array.from({ length: 8 }, () => Array(8).fill(null)),
      currentPlayer: 'black',
      directions: [
        [-1, -1], [-1, 0], [-1, 1],
        [0, -1],         [0, 1],
        [1, -1], [1, 0], [1, 1]
      ],
      gameOver: false,
      winner: '',
      blackCount: 0,
      whiteCount: 0
    };
  },
  created() {
    this.initializeBoard();
  },
  methods: {
    initializeBoard() {
      this.board = Array.from({ length: 8 }, () => Array(8).fill(null));
      this.currentPlayer = 'black';
      this.gameOver = false;
      this.winner = '';
      this.board[3][3] = 'white';
      this.board[3][4] = 'black';
      this.board[4][3] = 'black';
      this.board[4][4] = 'white';
      this.updateCounts();
    },
    placePiece(rowIndex, colIndex) {
      if (this.currentPlayer === 'black' && this.isValidMove(rowIndex, colIndex)) {
        this.board[rowIndex][colIndex] = this.currentPlayer;
        this.flipPieces(rowIndex, colIndex);
        this.currentPlayer = 'white';
        this.checkGameOver();
        if (this.currentPlayer === 'white') {
          setTimeout(() => {
            this.placeWhitePieceRandomly();
          }, 500);
        }
      }
    },
    isValidMove(rowIndex, colIndex) {
      if (this.board[rowIndex][colIndex] !== null) return false;

      for (const [dx, dy] of this.directions) {
        if (this.canFlipInDirection(rowIndex, colIndex, dx, dy)) {
          return true;
        }
      }
      return false;
    },
    canFlipInDirection(rowIndex, colIndex, dx, dy) {
      let x = rowIndex + dx;
      let y = colIndex + dy;
      if (x < 0 || x >= 8 || y < 0 || y >= 8 || this.board[x][y] !== this.getOpponent()) {
        return false;
      }

      while (x >= 0 && x < 8 && y >= 0 && y < 8) {
        if (this.board[x][y] === null) {
          return false;
        }
        if (this.board[x][y] === this.currentPlayer) {
          return true;
        }
        x += dx;
        y += dy;
      }
      return false;
    },
    flipPieces(rowIndex, colIndex) {
      for (const [dx, dy] of this.directions) {
        if (this.canFlipInDirection(rowIndex, colIndex, dx, dy)) {
          this.flipInDirection(rowIndex, colIndex, dx, dy);
        }
      }
    },
    flipInDirection(rowIndex, colIndex, dx, dy) {
      let x = rowIndex + dx;
      let y = colIndex + dy;
      while (x >= 0 && x < 8 && y >= 0 && y < 8) {
        if (this.board[x][y] === this.currentPlayer) {
          break;
        }
        this.board[x][y] = this.currentPlayer;
        x += dx;
        y += dy;
      }
    },
    getOpponent() {
      return this.currentPlayer === 'black' ? 'white' : 'black';
    },
    checkGameOver() {
      const validMoves = this.getValidMoves();
      if (validMoves.length === 0) {
        this.currentPlayer = this.getOpponent();
        const opponentMoves = this.getValidMoves();
        if (opponentMoves.length === 0) {
          this.gameOver = true;
          this.determineWinner();
        }
      } else if (this.board.flat().every(cell => cell !== null)) {
        this.gameOver = true;
        this.determineWinner();
      }
    },
    getValidMoves() {
      const validMoves = [];
      for (let i = 0; i < 8; i++) {
        for (let j = 0; j < 8; j++) {
          if (this.isValidMove(i, j)) {
            validMoves.push({ rowIndex: i, colIndex: j });
          }
        }
      }
      return validMoves;
    },
    determineWinner() {
      let blackCount = 0;
      let whiteCount = 0;
      for (let i = 0; i < 8; i++) {
        for (let j = 0; j < 8; j++) {
          if (this.board[i][j] === 'black') {
            blackCount++;
          } else if (this.board[i][j] === 'white') {
            whiteCount++;
          }
        }
      }
      if (blackCount === 0) {
        this.winner = 'White';
      } else if (whiteCount === 0) {
        this.winner = 'Black';
      } else if (blackCount > whiteCount) {
        this.winner = 'Black';
      } else if (whiteCount > blackCount) {
        this.winner = 'White';
      } else {
        this.winner = 'Draw';
      }
      this.blackCount = blackCount;
      this.whiteCount = whiteCount;
    },
    placeWhitePieceRandomly() {
      const validMoves = this.getValidMoves();
      if (validMoves.length === 0) {
        console.log('No valid moves for white');
        this.currentPlayer = 'black';
        this.checkGameOver();
        return;
      }

      const randomIndex = Math.floor(Math.random() * validMoves.length);
      const { rowIndex, colIndex } = validMoves[randomIndex];
      this.board[rowIndex][colIndex] = 'white';
      this.flipPieces(rowIndex, colIndex);
      this.currentPlayer = 'black';
      this.checkGameOver();
    },
    updateCounts() {
      let blackCount = 0;
      let whiteCount = 0;
      for (let i = 0; i < 8; i++) {
        for (let j = 0; j < 8; j++) {
          if (this.board[i][j] === 'black') {
            blackCount++;
          } else if (this.board[i][j] === 'white') {
            whiteCount++;
          }
        }
      }
      this.blackCount = blackCount;
      this.whiteCount = whiteCount;
    },
    restartGame() {
      this.initializeBoard();
    }
  },
  watch: {
    board: {
      handler() {
        this.updateCounts();
      },
      deep: true
    }
  }
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  position: relative;
}

.controls {
  position: absolute;
  top: 20px;
  left: 20px;
  text-align: left;
  color: #fff;
}

.chessboard {
  display: grid;
  grid-template-columns: repeat(8, 50px);
  grid-template-rows: repeat(8, 50px);
  gap: 0;
  background-color: #ffe100;
  border: 2px solid #fff;
  border-radius: 5px;
}

.row {
  display: contents;
}

.cell {
  width: 50px;
  height: 50px;
  background-color: transparent; 
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.5s ease-in-out, transform 0.5s ease-in-out;
  border: 1px solid #fff;
}

.cell.black {
  background-color: transparent;
}

.cell.black::before {
  content: '';
  width: 40px;
  height: 40px;
  background-color: #000;
  border-radius: 50%;
  transition: background-color 0.5s ease-in-out, transform 0.5s ease-in-out;
}

.cell.white::before {
  content: '';
  width: 40px;
  height: 40px;
  background-color: #fff;
  border-radius: 50%;
  transition: background-color 0.5s ease-in-out, transform 0.5s ease-in-out;
}

.cell.flip-animation {
  animation: flip 0.5s ease-in-out;
}

@keyframes flip {
  0% {
    transform: rotateY(0deg);
    background-color: #fff;
  }
  50% {
    transform: rotateY(180deg);
    background-color: #ccc;
  }
  100% {
    transform: rotateY(360deg);
    background-color: #000;
  }
}

.game-over {
  margin-bottom: 10px;
  font-size: 24px;
  color: #000000;
}

.scoreboard {
  margin-bottom: 10px;
  font-size: 18px;
  color: #000000;
}

.restart-button {
  padding: 10px 20px;
  font-size: 16px;
  color: #000000;
  background-color: #ff5733;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease-in-out;
}

.restart-button:hover {
  background-color: #ff3300;
}
</style>