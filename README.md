<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Chess Game</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>Chess Game</h1>
  <div class="controls">
    <label for="difficulty">Bot Difficulty:</label>
    <select id="difficulty">
      <option value="easy">Easy</option>
      <option value="normal">Normal</option>
      <option value="hard">Hard</option>
    </select>
    <button id="reset">Reset Game</button>
  </div>
  <div id="chessboard"></div>
  <script src="script.js"></script>
</body>
</html>
body {
  font-family: Arial, sans-serif;
  text-align: center;
  background-color: #f0f0f0;
}

h1 {
  margin-bottom: 20px;
}

.controls {
  margin-bottom: 20px;
}

#chessboard {
  display: grid;
  grid-template-columns: repeat(8, 60px);
  grid-template-rows: repeat(8, 60px);
  margin: 0 auto;
  width: 480px;
  height: 480px;
  border: 2px solid #333;
}

.square {
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
}

.square.light {
  background-color: #f0d9b5;
}

.square.dark {
  background-color: #b58863;
}

.square.highlight {
  background-color: #a4c2f4;
}
const chessboard = document.getElementById('chessboard');
const difficultySelect = document.getElementById('difficulty');
const resetButton = document.getElementById('reset');

let board = [];
let selectedPiece = null;
let currentPlayer = 'white';
let botDifficulty = 'easy';
let botMemory = JSON.parse(localStorage.getItem('botMemory')) || {};

// Initialize the board
function initializeBoard() {
  board = [
    ['r', 'n', 'b', 'q', 'k', 'b', 'n', 'r'],
    ['p', 'p', 'p', 'p', 'p', 'p', 'p', 'p'],
    ['', '', '', '', '', '', '', ''],
    ['', '', '', '', '', '', '', ''],
    ['', '', '', '', '', '', '', ''],
    ['', '', '', '', '', '', '', ''],
    ['P', 'P', 'P', 'P', 'P', 'P', 'P', 'P'],
    ['R', 'N', 'B', 'Q', 'K', 'B', 'N', 'R']
  ];
  renderBoard();
}

// Render the board
function renderBoard() {
  chessboard.innerHTML = '';
  for (let row = 0; row < 8; row++) {
    for (let col = 0; col < 8; col++) {
      const square = document.createElement('div');
      square.classList.add('square');
      square.classList.add((row + col) % 2 === 0 ? 'light' : 'dark');
      square.dataset.row = row;
      square.dataset.col = col;
      square.textContent = board[row][col];
      square.addEventListener('click', () => handleSquareClick(row, col));
      chessboard.appendChild(square);
    }
  }
}

// Handle square clicks
function handleSquareClick(row, col) {
  if (currentPlayer === 'black') return; // Bot's turn

  const piece = board[row][col];
  if (selectedPiece) {
    movePiece(selectedPiece.row, selectedPiece.col, row, col);
    selectedPiece = null;
    renderBoard();
    if (currentPlayer === 'black') setTimeout(botMove, 500);
  } else if (piece && piece.toUpperCase() === piece && currentPlayer === 'white') {
    selectedPiece = { row, col };
    highlightMoves(row, col);
  }
}

// Move a piece
function movePiece(fromRow, fromCol, toRow, toCol) {
  const piece = board[fromRow][fromCol];
  board[toRow][toCol] = piece;
  board[fromRow][fromCol] = '';
  currentPlayer = currentPlayer === 'white' ? 'black' : 'white';
}

// Highlight possible moves
function highlightMoves(row, col) {
  // Simplified: Highlight all empty squares for demonstration
  const squares = document.querySelectorAll('.square');
  squares.forEach(square => {
    const r = parseInt(square.dataset.row);
    const c = parseInt(square.dataset.col);
    if (board[r][c] === '') square.classList.add('highlight');
  });
}

// Bot's move
function botMove() {
  const moves = [];
  for (let row = 0; row < 8; row++) {
    for (let col = 0; col < 8; col++) {
      if (board[row][col] && board[row][col].toLowerCase() === board[row][col]) {
        // Bot's pieces (lowercase)
        moves.push({ row, col });
      }
    }
  }

  let move;
  if (botDifficulty === 'easy') {
    move = moves[Math.floor(Math.random() * moves.length)];
  } else if (botDifficulty === 'normal') {
    // Basic evaluation: Capture if possible
    move = moves.find(m => board[m.row][m.col] !== '') || moves[Math.floor(Math.random() * moves.length)];
  } else if (botDifficulty === 'hard') {
    // Use botMemory to make better moves
    move = botMemory[JSON.stringify(board)] || moves[Math.floor(Math.random() * moves.length)];
  }

  if (move) {
    const toRow = Math.floor(Math.random() * 8);
    const toCol = Math.floor(Math.random() * 8);
    movePiece(move.row, move.col, toRow, toCol);
    botMemory[JSON.stringify(board)] = { row: toRow, col: toCol };
    localStorage.setItem('botMemory', JSON.stringify(botMemory));
    renderBoard();
  }
}

// Reset the game
resetButton.addEventListener('click', () => {
  initializeBoard();
  currentPlayer = 'white';
});

// Change bot difficulty
difficultySelect.addEventListener('change', (e) => {
  botDifficulty = e.target.value;
});

// Initialize the game
initializeBoard();
simple chess game with bot 
