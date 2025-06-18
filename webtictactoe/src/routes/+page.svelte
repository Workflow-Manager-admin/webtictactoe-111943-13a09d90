<script>
  // --- Game logic state ---
  let board = [
    ['', '', ''],
    ['', '', ''],
    ['', '', ''],
  ];
  let currentPlayer = 'X';
  let winner = '';
  let draw = false;
  let status = "Player X's turn";

  // PUBLIC_INTERFACE
  function handleCellClick(row, col) {
    // Prevent click if cell is already taken or game is finished
    if (board[row][col] || winner) return;

    board[row][col] = currentPlayer;
    if (checkWinner()) {
      winner = currentPlayer;
      status = `Winner: ${winner}!`;
    } else if (isBoardFull()) {
      draw = true;
      status = 'Draw!';
    } else {
      currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
      status = `Player ${currentPlayer}'s turn`;
    }
    // ensure reactivity for Svelte
    board = [...board.map(r => [...r])];
  }

  // PUBLIC_INTERFACE
  function checkWinner() {
    // Rows, columns, diagonals
    const lines = [
      // Rows
      [ [0,0], [0,1], [0,2] ],
      [ [1,0], [1,1], [1,2] ],
      [ [2,0], [2,1], [2,2] ],
      // Columns
      [ [0,0], [1,0], [2,0] ],
      [ [0,1], [1,1], [2,1] ],
      [ [0,2], [1,2], [2,2] ],
      // Diagonals
      [ [0,0], [1,1], [2,2] ],
      [ [0,2], [1,1], [2,0] ],
    ];

    return lines.some(line => {
      const [a, b, c] = line;
      return (
        board[a[0]][a[1]] &&
        board[a[0]][a[1]] === board[b[0]][b[1]] &&
        board[a[0]][a[1]] === board[c[0]][c[1]]
      );
    });
  }

  // PUBLIC_INTERFACE
  function isBoardFull() {
    return board.flat().every(cell => cell !== '');
  }

  // PUBLIC_INTERFACE
  function restartGame() {
    board = [
      ['', '', ''],
      ['', '', ''],
      ['', '', ''],
    ];
    currentPlayer = 'X';
    winner = '';
    draw = false;
    status = "Player X's turn";
  }

  // Helper to highlight winner cells (optional, fallback: none highlighted if not found)
  // This is a minimalist approach, highlight if any of the winning lines match
  function winnerCells(r, c) {
    if (!winner) return false;
    const lines = [
      // Rows
      [ [0,0], [0,1], [0,2] ],
      [ [1,0], [1,1], [1,2] ],
      [ [2,0], [2,1], [2,2] ],
      // Columns
      [ [0,0], [1,0], [2,0] ],
      [ [0,1], [1,1], [2,1] ],
      [ [0,2], [1,2], [2,2] ],
      // Diagonals
      [ [0,0], [1,1], [2,2] ],
      [ [0,2], [1,1], [2,0] ],
    ];
    return lines.some(line => {
      if (
        board[line[0][0]][line[0][1]] &&
        board[line[0][0]][line[0][1]] === board[line[1][0]][line[1][1]] &&
        board[line[0][0]][line[0][1]] === board[line[2][0]][line[2][1]] &&
        board[line[0][0]][line[0][1]] === winner
      ) {
        return line.some(([lr, lc]) => lr === r && lc === c);
      }
      return false;
    });
  }
</script>

<style>
  :root {
    --primary: #4CAF50;
    --secondary: #FFC107;
    --accent: #2196F3;
    --background: #f5f5f5;
    --foreground: #fff;
    --cell-size: min(100px, 18vw);
    --cell-font-size: min(3rem, 7vw);
  }

  main {
    height: 100vh;
    width: 100vw;
    background: var(--background);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .game-container {
    background: var(--foreground);
    box-shadow: 0 6px 32px rgba(0,0,0,0.09);
    padding: 2rem 2rem 1.5rem 2rem;
    border-radius: 2rem;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .status {
    font-size: 1.35rem;
    margin-bottom: 1.1rem;
    font-weight: 600;
    color: var(--primary);
    min-height: 2.2em;
    letter-spacing: 0.01em;
    text-align: center;
    transition: color 0.25s;
  }

  .board {
    display: grid;
    grid-template-columns: repeat(3, var(--cell-size));
    grid-template-rows: repeat(3, var(--cell-size));
    gap: 0.32em;
    margin-bottom: 1.3em;
  }

  .cell {
    width: var(--cell-size);
    height: var(--cell-size);
    font-size: var(--cell-font-size);
    background: var(--foreground);
    border: none;
    outline: 2px solid var(--accent);
    border-radius: 0.65em;
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: bold;
    color: var(--primary);
    cursor: pointer;
    transition: background 0.16s, color 0.16s;
    user-select: none;
  }

  .cell:disabled,
  .cell.disabled {
    background: #e0e0e0;
    color: #9e9e9e;
    cursor: not-allowed;
  }

  .cell[aria-current="true"]:not(:disabled) {
    background: var(--secondary);
    color: #212121;
    outline: 2.5px solid var(--primary);
    z-index: 2;
  }

  .winner-cell {
    background: var(--primary);
    color: #fff;
    animation: winner-flash 1s linear infinite alternate;
  }

  @keyframes winner-flash {
    0% { background: var(--primary); color: #fff; }
    100% { background: var(--secondary); color: #212121; }
  }

  .restart-btn {
    background: var(--accent);
    color: #fff;
    border: none;
    padding: 0.75em 2em;
    border-radius: 1em;
    font-size: 1.05rem;
    font-weight: 600;
    letter-spacing: 0.05em;
    cursor: pointer;
    transition: background 0.2s;
    box-shadow: 0 2px 8px #bbb6;
  }
  .restart-btn:active {
    background: var(--primary);
  }

  @media (max-width: 600px) {
    .game-container {
      padding: 1em 0.5em 1em 0.5em;
      border-radius: 1.1rem;
    }
  }
</style>

<main>
  <div class="game-container" data-testid="tictactoe-container">
    <div class="status" data-testid="status">{status}</div>
    <div class="board" data-testid="board">
      {#each board as row, rIdx (rIdx)}
        {#each row as cell, cIdx (cIdx)}
          <button
            class="cell {winner && winnerCells(rIdx, cIdx) ? 'winner-cell' : ''} {cell || winner ? 'disabled' : ''}"
            disabled={!!cell || !!winner}
            aria-current={currentPlayer === 'X' && !winner && !draw && !cell && rIdx*cIdx===0 ? "true" : undefined}
            data-testid={"cell-" + rIdx + "-" + cIdx}
            on:click={() => handleCellClick(rIdx, cIdx)}
          >
            {cell}
          </button>
        {/each}
      {/each}
    </div>
    <button class="restart-btn" on:click={restartGame} data-testid="restart">
      Restart
    </button>
  </div>
</main>
