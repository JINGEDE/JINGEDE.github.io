<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>五子棋游戏</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            primary: '#8B5A2B',
            secondary: '#D2B48C',
            board: '#DEB887',
            black: '#000000',
            white: '#FFFFFF',
          },
          fontFamily: {
            sans: ['Inter', 'system-ui', 'sans-serif'],
          },
        },
      }
    }
  </script>
  <style type="text/tailwindcss">
    @layer utilities {
      .board-grid {
        background-image: linear-gradient(#000 1px, transparent 1px),
                          linear-gradient(90deg, #000 1px, transparent 1px);
      }
      .piece-shadow {
        box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3);
      }
      .btn-hover {
        @apply transition-all duration-300 hover:shadow-lg hover:-translate-y-1;
      }
      .piece-transition {
        transition: all 0.2s ease-out;
      }
      .modal-appear {
        animation: fadeIn 0.3s ease-out;
      }
      @keyframes fadeIn {
        from { opacity: 0; transform: scale(0.95); }
        to { opacity: 1; transform: scale(1); }
      }
    }
  </style>
</head>
<body class="bg-gradient-to-br from-amber-50 to-amber-100 min-h-screen flex flex-col items-center justify-center p-4 font-sans">
  <!-- 游戏标题 -->
  <h1 class="text-[clamp(2rem,5vw,3rem)] font-bold text-primary mb-6 text-center tracking-wide">五子棋</h1>
  
  <div class="max-w-6xl w-full flex flex-col md:flex-row gap-6 items-center md:items-start justify-center">
    <!-- 游戏控制面板 -->
    <div class="bg-white/80 backdrop-blur-sm rounded-xl p-6 shadow-xl w-full md:w-80 order-2 md:order-1">
      <div class="space-y-6">
        <!-- 游戏信息 -->
        <div class="bg-secondary/30 rounded-lg p-4">
          <h2 class="text-xl font-semibold text-primary mb-3 flex items-center">
            <i class="fa fa-info-circle mr-2"></i>游戏信息
          </h2>
          <div class="space-y-3">
            <div class="flex items-center justify-between">
              <span class="text-gray-700">当前玩家:</span>
              <div id="current-player" class="flex items-center">
                <div class="w-5 h-5 rounded-full bg-black mr-2"></div>
                <span>黑方</span>
              </div>
            </div>
            <div class="flex items-center justify-between">
              <span class="text-gray-700">游戏状态:</span>
              <span id="game-status" class="font-medium">进行中</span>
            </div>
          </div>
        </div>
        
        <!-- 棋盘设置 -->
        <div class="bg-secondary/30 rounded-lg p-4">
          <h2 class="text-xl font-semibold text-primary mb-3 flex items-center">
            <i class="fa fa-th mr-2"></i>棋盘设置
          </h2>
          <div class="space-y-4">
            <div>
              <label class="block text-gray-700 mb-2">棋盘大小</label>
              <div class="grid grid-cols-3 gap-2">
                <button id="size-9" class="board-size-btn bg-primary text-white py-2 px-3 rounded-md btn-hover">9×9</button>
                <button id="size-13" class="board-size-btn bg-white text-primary border border-primary py-2 px-3 rounded-md btn-hover">13×13</button>
                <button id="size-19" class="board-size-btn bg-white text-primary border border-primary py-2 px-3 rounded-md btn-hover">19×19</button>
              </div>
            </div>
            
            <div>
              <label class="block text-gray-700 mb-2">游戏模式</label>
              <div class="grid grid-cols-2 gap-2">
                <button id="mode-pvp" class="game-mode-btn bg-primary text-white py-2 px-3 rounded-md btn-hover">双人对战</button>
                <button id="mode-ai" class="game-mode-btn bg-white text-primary border border-primary py-2 px-3 rounded-md btn-hover">AI对战</button>
              </div>
            </div>
            
            <div id="ai-difficulty-container" class="hidden">
              <label class="block text-gray-700 mb-2">AI难度</label>
              <div class="grid grid-cols-2 gap-2">
                <button id="difficulty-easy" class="ai-difficulty-btn bg-primary text-white py-2 px-3 rounded-md btn-hover">初级</button>
                <button id="difficulty-medium" class="ai-difficulty-btn bg-white text-primary border border-primary py-2 px-3 rounded-md btn-hover">中级</button>
                <button id="difficulty-hard" class="ai-difficulty-btn bg-white text-primary border border-primary py-2 px-3 rounded-md btn-hover">高级</button>
                <button id="difficulty-expert" class="ai-difficulty-btn bg-white text-primary border border-primary py-2 px-3 rounded-md btn-hover">专家</button>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 游戏控制按钮 -->
        <div class="space-y-3">
          <button id="start-game" class="w-full bg-green-600 text-white py-3 px-4 rounded-md font-medium btn-hover flex items-center justify-center">
            <i class="fa fa-play-circle mr-2"></i>开始游戏
          </button>
          <button id="undo-move" class="w-full bg-blue-600 text-white py-3 px-4 rounded-md font-medium btn-hover flex items-center justify-center">
            <i class="fa fa-undo mr-2"></i>悔棋
          </button>
          <button id="restart-game" class="w-full bg-red-600 text-white py-3 px-4 rounded-md font-medium btn-hover flex items-center justify-center">
            <i class="fa fa-refresh mr-2"></i>重新开始
          </button>
        </div>
      </div>
    </div>
    
    <!-- 棋盘区域 -->
    <div class="order-1 md:order-2 relative">
      <div id="board-container" class="relative rounded-lg overflow-hidden shadow-2xl bg-board/90">
        <canvas id="board" class="cursor-pointer"></canvas>
        <div id="win-line" class="absolute top-0 left-0 w-full h-full pointer-events-none hidden"></div>
      </div>
      
      <!-- 游戏结果模态框 -->
      <div id="result-modal" class="fixed inset-0 flex items-center justify-center bg-black/50 z-50 hidden modal-appear">
        <div class="bg-white rounded-xl p-8 max-w-md w-full mx-4 shadow-2xl">
          <div class="text-center">
            <div id="winner-icon" class="mx-auto w-16 h-16 rounded-full flex items-center justify-center mb-4">
              <div class="w-12 h-12 rounded-full bg-black"></div>
            </div>
            <h2 class="text-2xl font-bold text-gray-800 mb-2">黑方获胜!</h2>
            <p class="text-gray-600 mb-6">恭喜你赢得了这场精彩的比赛!</p>
            <div class="flex space-x-4">
              <button id="play-again" class="flex-1 bg-primary text-white py-3 px-4 rounded-md font-medium btn-hover">
                再来一局
              </button>
              <button id="close-result" class="flex-1 bg-gray-200 text-gray-700 py-3 px-4 rounded-md font-medium btn-hover">
                关闭
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <script>
    document.addEventListener('DOMContentLoaded', () => {
      // 游戏配置
      const config = {
        boardSize: 13,
        cellSize: 30,
        gameMode: 'pvp', // pvp 或 ai
        aiDifficulty: 'medium', // easy, medium, hard, expert
        currentPlayer: 'black', // black 或 white
        gameStatus: 'playing', // playing, over
        winner: null,
      };
      
      // 游戏数据
      const gameData = {
        board: [],
        history: [],
        winLine: null,
      };
      
      // DOM 元素
      const canvas = document.getElementById('board');
      const ctx = canvas.getContext('2d');
      const boardContainer = document.getElementById('board-container');
      const currentPlayerEl = document.getElementById('current-player');
      const gameStatusEl = document.getElementById('game-status');
      const resultModal = document.getElementById('result-modal');
      const winnerIcon = document.getElementById('winner-icon');
      const playAgainBtn = document.getElementById('play-again');
      const closeResultBtn = document.getElementById('close-result');
      const startGameBtn = document.getElementById('start-game');
      const undoMoveBtn = document.getElementById('undo-move');
      const restartGameBtn = document.getElementById('restart-game');
      const winLineEl = document.getElementById('win-line');
      
      // 初始化棋盘数据
      function initBoard() {
        gameData.board = Array(config.boardSize).fill().map(() => Array(config.boardSize).fill(null));
        gameData.history = [];
        gameData.winLine = null;
        config.currentPlayer = 'black';
        config.gameStatus = 'playing';
        config.winner = null;
        updateGameInfo();
      }
      
      // 更新游戏信息显示
      function updateGameInfo() {
        // 更新当前玩家显示
        const playerColor = config.currentPlayer === 'black' ? 'black' : 'white';
        const playerText = config.currentPlayer === 'black' ? '黑方' : '白方';
        currentPlayerEl.innerHTML = `
          <div class="w-5 h-5 rounded-full bg-${playerColor} mr-2"></div>
          <span>${playerText}</span>
        `;
        
        // 更新游戏状态
        gameStatusEl.textContent = config.gameStatus === 'playing' ? '进行中' : '已结束';
        
        // 更新棋盘尺寸显示
        document.querySelectorAll('.board-size-btn').forEach(btn => {
          const size = btn.id.split('-')[1];
          btn.classList.toggle('bg-primary', size === config.boardSize.toString());
          btn.classList.toggle('bg-white', size !== config.boardSize.toString());
          btn.classList.toggle('text-white', size === config.boardSize.toString());
          btn.classList.toggle('text-primary', size !== config.boardSize.toString());
          btn.classList.toggle('border-primary', size !== config.boardSize.toString());
        });
        
        // 更新游戏模式显示
        document.querySelectorAll('.game-mode-btn').forEach(btn => {
          const mode = btn.id.split('-')[1];
          btn.classList.toggle('bg-primary', mode === config.gameMode);
          btn.classList.toggle('bg-white', mode !== config.gameMode);
          btn.classList.toggle('text-white', mode === config.gameMode);
          btn.classList.toggle('text-primary', mode !== config.gameMode);
          btn.classList.toggle('border-primary', mode !== config.gameMode);
        });
        
        // 更新AI难度显示
        document.querySelectorAll('.ai-difficulty-btn').forEach(btn => {
          const difficulty = btn.id.split('-')[1];
          btn.classList.toggle('bg-primary', difficulty === config.aiDifficulty);
          btn.classList.toggle('bg-white', difficulty !== config.aiDifficulty);
          btn.classList.toggle('text-white', difficulty === config.aiDifficulty);
          btn.classList.toggle('text-primary', difficulty !== config.aiDifficulty);
          btn.classList.toggle('border-primary', difficulty !== config.aiDifficulty);
        });
        
        // 显示或隐藏AI难度选择
        document.getElementById('ai-difficulty-container').classList.toggle('hidden', config.gameMode !== 'ai');
      }
      
      // 设置棋盘尺寸
      function setBoardSize(size) {
        config.boardSize = size;
        resizeCanvas();
        initBoard();
        drawBoard();
      }
      
      // 设置游戏模式
      function setGameMode(mode) {
        config.gameMode = mode;
        updateGameInfo();
      }
      
      // 设置AI难度
      function setAIDifficulty(difficulty) {
        config.aiDifficulty = difficulty;
        updateGameInfo();
      }
      
      // 调整Canvas大小
      function resizeCanvas() {
        const canvasSize = config.boardSize * config.cellSize;
        canvas.width = canvasSize;
        canvas.height = canvasSize;
        boardContainer.style.width = `${canvasSize}px`;
        boardContainer.style.height = `${canvasSize}px`;
      }
      
      // 绘制棋盘
      function drawBoard() {
        const { boardSize, cellSize } = config;
        const canvasSize = boardSize * cellSize;
        
        // 清空画布
        ctx.clearRect(0, 0, canvasSize, canvasSize);
        
        // 绘制棋盘背景
        ctx.fillStyle = '#DEB887';
        ctx.fillRect(0, 0, canvasSize, canvasSize);
        
        // 绘制网格线
        ctx.strokeStyle = '#000';
        ctx.lineWidth = 1;
        
        // 绘制横线和竖线
        for (let i = 0; i < boardSize; i++) {
          // 横线
          ctx.beginPath();
          ctx.moveTo(cellSize / 2, i * cellSize + cellSize / 2);
          ctx.lineTo(canvasSize - cellSize / 2, i * cellSize + cellSize / 2);
          ctx.stroke();
          
          // 竖线
          ctx.beginPath();
          ctx.moveTo(i * cellSize + cellSize / 2, cellSize / 2);
          ctx.lineTo(i * cellSize + cellSize / 2, canvasSize - cellSize / 2);
          ctx.stroke();
        }
        
        // 绘制天元和星位
        const starPoints = getStarPoints();
        starPoints.forEach(({ x, y }) => {
          ctx.beginPath();
          ctx.arc(x * cellSize + cellSize / 2, y * cellSize + cellSize / 2, 4, 0, Math.PI * 2);
          ctx.fillStyle = '#000';
          ctx.fill();
        });
        
        // 绘制棋子
        for (let i = 0; i < boardSize; i++) {
          for (let j = 0; j < boardSize; j++) {
            if (gameData.board[i][j]) {
              drawPiece(j, i, gameData.board[i][j]);
            }
          }
        }
        
        // 绘制胜利线
        if (gameData.winLine) {
          drawWinLine();
        }
      }
      
      // 获取星位点坐标
      function getStarPoints() {
        const points = [];
        const size = config.boardSize;
        
        if (size === 9) {
          points.push({ x: 2, y: 2 }, { x: 4, y: 4 }, { x: 6, y: 2 });
          points.push({ x: 2, y: 6 }, { x: 6, y: 6 });
        } else if (size === 13) {
          points.push({ x: 3, y: 3 }, { x: 9, y: 3 }, { x: 6, y: 6 });
          points.push({ x: 3, y: 9 }, { x: 9, y: 9 });
        } else if (size === 19) {
          points.push({ x: 3, y: 3 }, { x: 9, y: 3 }, { x: 15, y: 3 });
          points.push({ x: 3, y: 9 }, { x: 9, y: 9 }, { x: 15, y: 9 });
          points.push({ x: 3, y: 15 }, { x: 9, y: 15 }, { x: 15, y: 15 });
        }
        
        return points;
      }
      
      // 绘制棋子
      function drawPiece(x, y, color) {
        const { cellSize } = config;
        const centerX = x * cellSize + cellSize / 2;
        const centerY = y * cellSize + cellSize / 2;
        const radius = cellSize / 2 - 2;
        
        // 创建径向渐变使棋子更立体
        const gradient = ctx.createRadialGradient(
          centerX - radius * 0.3, centerY - radius * 0.3, radius * 0.1,
          centerX, centerY, radius
        );
        
        if (color === 'black') {
          gradient.addColorStop(0, '#555');
          gradient.addColorStop(1, '#000');
        } else {
          gradient.addColorStop(0, '#fff');
          gradient.addColorStop(1, '#ddd');
        }
        
        ctx.beginPath();
        ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
        ctx.fillStyle = gradient;
        ctx.fill();
        
        // 添加高光效果
        if (color === 'white') {
          const highlight = ctx.createRadialGradient(
            centerX - radius * 0.3, centerY - radius * 0.3, 0,
            centerX - radius * 0.3, centerY - radius * 0.3, radius * 0.6
          );
          highlight.addColorStop(0, 'rgba(255, 255, 255, 0.8)');
          highlight.addColorStop(1, 'rgba(255, 255, 255, 0)');
          
          ctx.beginPath();
          ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
          ctx.fillStyle = highlight;
          ctx.fill();
        }
      }
      
      // 绘制胜利线
      function drawWinLine() {
        if (!gameData.winLine) return;
        
        const { cellSize } = config;
        const { start, end } = gameData.winLine;
        
        const startX = start.x * cellSize + cellSize / 2;
        const startY = start.y * cellSize + cellSize / 2;
        const endX = end.x * cellSize + cellSize / 2;
        const endY = end.y * cellSize + cellSize / 2;
        
        // 创建SVG元素
        const svgNS = "http://www.w3.org/2000/svg";
        const svg = document.createElementNS(svgNS, "svg");
        const line = document.createElementNS(svgNS, "line");
        
        svg.setAttributeNS(null, "width", canvas.width);
        svg.setAttributeNS(null, "height", canvas.height);
        
        line.setAttributeNS(null, "x1", startX);
        line.setAttributeNS(null, "y1", startY);
        line.setAttributeNS(null, "x2", endX);
        line.setAttributeNS(null, "y2", endY);
        line.setAttributeNS(null, "stroke", "#ff0000");
        line.setAttributeNS(null, "stroke-width", "3");
        line.setAttributeNS(null, "stroke-dasharray", "none");
        line.setAttributeNS(null, "stroke-linecap", "round");
        
        svg.appendChild(line);
        
        // 清空并添加新的胜利线
        winLineEl.innerHTML = '';
        winLineEl.appendChild(svg);
        winLineEl.classList.remove('hidden');
      }
      
      // 落子
      function placePiece(x, y) {
        // 检查是否在游戏中且该位置为空
        if (config.gameStatus !== 'playing' || gameData.board[y][x] !== null) {
          return false;
        }
        
        // 记录历史
        gameData.history.push({ x, y, player: config.currentPlayer });
        
        // 落子
        gameData.board[y][x] = config.currentPlayer;
        
        // 重绘棋盘
        drawBoard();
        
        // 检查是否获胜
        if (checkWin(x, y, config.currentPlayer)) {
          config.gameStatus = 'over';
          config.winner = config.currentPlayer;
          showResultModal();
          return true;
        }
        
        // 检查是否平局
        if (checkDraw()) {
          config.gameStatus = 'over';
          config.winner = 'tie';
          showResultModal();
          return true;
        }
        
        // 切换玩家
        config.currentPlayer = config.currentPlayer === 'black' ? 'white' : 'black';
        updateGameInfo();
        
        // 如果是AI对战且轮到AI
        if (config.gameMode === 'ai' && config.currentPlayer === 'white') {
          setTimeout(() => {
            makeAIMove();
          }, 500);
        }
        
        return true;
      }
      
      // AI落子
      function makeAIMove() {
        if (config.gameStatus !== 'playing') return;
        
        let move;
        
        // 根据难度选择AI策略
        switch (config.aiDifficulty) {
          case 'easy':
            move = makeEasyAIMove();
            break;
          case 'medium':
            move = makeMediumAIMove();
            break;
          case 'hard':
            move = makeHardAIMove();
            break;
          case 'expert':
            move = makeExpertAIMove();
            break;
          default:
            move = makeMediumAIMove();
        }
        
        if (move) {
          placePiece(move.x, move.y);
        }
      }
      
      // 初级AI：随机落子
      function makeEasyAIMove() {
        const availableMoves = [];
        
        for (let i = 0; i < config.boardSize; i++) {
          for (let j = 0; j < config.boardSize; j++) {
            if (gameData.board[i][j] === null) {
              availableMoves.push({ x: j, y: i });
            }
          }
        }
        
        if (availableMoves.length > 0) {
          const randomIndex = Math.floor(Math.random() * availableMoves.length);
          return availableMoves[randomIndex];
        }
        
        return null;
      }
      
      // 中级AI：有一定策略的落子
      function makeMediumAIMove() {
        // 优先选择能形成自己连子的位置
        const aiMove = findBestMove('white');
        if (aiMove && aiMove.score >= 3) {
          return aiMove;
        }
        
        // 其次选择能阻止对手连子的位置
        const playerMove = findBestMove('black');
        if (playerMove && playerMove.score >= 2) {
          return playerMove;
        }
        
        // 否则随机落子
        return makeEasyAIMove();
      }
      
      // 高级AI：考虑更多可能性和防守
      function makeHardAIMove() {
        // 优先选择能形成自己连子的位置
        const aiMove = findBestMove('white');
        
        // 其次选择能阻止对手连子的位置
        const playerMove = findBestMove('black');
        
        // 如果AI的最佳位置比玩家的最佳位置好，或者两者一样好但AI能形成更多连子，则选择AI的位置
        if ((aiMove && aiMove.score > playerMove.score) || 
            (aiMove && aiMove.score === playerMove.score && aiMove.score >= 3)) {
          return aiMove;
        }
        
        // 否则选择阻止玩家的位置
        if (playerMove) {
          return playerMove;
        }
        
        // 否则随机落子
        return makeEasyAIMove();
      }
      
      // 专家AI：使用简单的极小极大算法
      function makeExpertAIMove() {
        const depth = 3; // 搜索深度
        let bestScore = -Infinity;
        let bestMove = null;
        
        // 遍历所有可能的移动
        for (let i = 0; i < config.boardSize; i++) {
          for (let j = 0; j < config.boardSize; j++) {
            if (gameData.board[i][j] === null) {
              // 模拟AI落子
              gameData.board[i][j] = 'white';
              
              // 评估局面
              const score = minimax(depth, false, -Infinity, Infinity);
              
              // 撤销落子
              gameData.board[i][j] = null;
              
              // 更新最佳移动
              if (score > bestScore) {
                bestScore = score;
                bestMove = { x: j, y: i };
              }
            }
          }
        }
        
        return bestMove || makeHardAIMove();
      }
      
      // 极小极大算法，带Alpha-Beta剪枝
      function minimax(depth, isMaximizing, alpha, beta) {
        // 检查游戏是否结束或达到最大深度
        const aiWin = checkWinWithoutSaving(0, 0, 'white');
        const playerWin = checkWinWithoutSaving(0, 0, 'black');
        
        if (aiWin) return Infinity;
        if (playerWin) return -Infinity;
        if (depth === 0 || checkDraw()) return evaluateBoard();
        
        if (isMaximizing) {
          let maxScore = -Infinity;
          
          // 遍历所有可能的移动
          for (let i = 0; i < config.boardSize; i++) {
            for (let j = 0; j < config.boardSize; j++) {
              if (gameData.board[i][j] === null) {
                // 模拟AI落子
                gameData.board[i][j] = 'white';
                
                // 递归评估
                const score = minimax(depth - 1, false, alpha, beta);
                
                // 撤销落子
                gameData.board[i][j] = null;
                
                maxScore = Math.max(score, maxScore);
                alpha = Math.max(alpha, score);
                
                // Alpha-Beta剪枝
                if (beta <= alpha) break;
              }
            }
            
            if (beta <= alpha) break;
          }
          
          return maxScore;
        } else {
          let minScore = Infinity;
          
          // 遍历所有可能的移动
          for (let i = 0; i < config.boardSize; i++) {
            for (let j = 0; j < config.boardSize; j++) {
              if (gameData.board[i][j] === null) {
                // 模拟玩家落子
                gameData.board[i][j] = 'black';
                
                // 递归评估
                const score = minimax(depth - 1, true, alpha, beta);
                
                // 撤销落子
                gameData.board[i][j] = null;
                
                minScore = Math.min(score, minScore);
                beta = Math.min(beta, score);
                
                // Alpha-Beta剪枝
                if (beta <= alpha) break;
              }
            }
            
            if (beta <= alpha) break;
          }
          
          return minScore;
        }
      }
      
      // 评估棋盘状态
      function evaluateBoard() {
        let score = 0;
        
        // 检查AI的连子情况
        score += evaluatePlayer('white') * 1.2; // AI的分数乘以1.2，使其更具攻击性
        
        // 检查玩家的连子情况
        score -= evaluatePlayer('black');
        
        return score;
      }
      
      // 评估玩家的连子情况
      function evaluatePlayer(player) {
        let score = 0;
        const directions = [
          { dx: 1, dy: 0 },  // 水平
          { dx: 0, dy: 1 },  // 垂直
          { dx: 1, dy: 1 },  // 对角线
          { dx: 1, dy: -1 }  // 反对角线
        ];
        
        for (let i = 0; i < config.boardSize; i++) {
          for (let j = 0; j < config.boardSize; j++) {
            if (gameData.board[i][j] === player) {
              for (const dir of directions) {
                const line = getLine(j, i, dir.dx, dir.dy, player);
                score += evaluateLine(line);
              }
            }
          }
        }
        
        return score;
      }
      
      // 获取一条线上的棋子情况
      function getLine(x, y, dx, dy, player) {
        let line = 1;  // 当前位置已经有一个棋子
        let emptyEnds = 0;
        
        // 正方向
        for (let i = 1; i < 5; i++) {
          const nx = x + dx * i;
          const ny = y + dy * i;
          
          if (nx < 0 || nx >= config.boardSize || ny < 0 || ny >= config.boardSize) {
            break;
          }
          
          if (gameData.board[ny][nx] === player) {
            line++;
          } else if (gameData.board[ny][nx] === null) {
            emptyEnds++;
            break;
          } else {
            break;
          }
        }
        
        // 反方向
        for (let i = 1; i < 5; i++) {
          const nx = x - dx * i;
          const ny = y - dy * i;
          
          if (nx < 0 || nx >= config.boardSize || ny < 0 || ny >= config.boardSize) {
            break;
          }
          
          if (gameData.board[ny][nx] === player) {
            line++;
          } else if (gameData.board[ny][nx] === null) {
            emptyEnds++;
            break;
          } else {
            break;
          }
        }
        
        return { length: line, emptyEnds };
      }
      
      // 评估一条线的得分
      function evaluateLine(line) {
        const { length, emptyEnds } = line;
        
        if (length >= 5) {
          return 10000;  // 五连
        } else if (length === 4) {
          if (emptyEnds === 2) {
            return 1000;  // 活四
          } else if (emptyEnds === 1) {
            return 100;   // 冲四
          }
        } else if (length === 3) {
          if (emptyEnds === 2) {
            return 50;    // 活三
          } else if (emptyEnds === 1) {
            return 10;    // 冲三
          }
        } else if (length === 2) {
          if (emptyEnds === 2) {
            return 5;     // 活二
          } else if (emptyEnds === 1) {
            return 2;     // 冲二
          }
        }
        
        return 0;
      }
      
      // 寻找最佳落子位置
      function findBestMove(player) {
        let bestScore = 0;
        let bestMove = null;
        
        for (let i = 0; i < config.boardSize; i++) {
          for (let j = 0; j < config.boardSize; j++) {
            if (gameData.board[i][j] === null) {
              // 模拟落子
              gameData.board[i][j] = player;
              
              // 评估这个位置的分数
              const score = evaluatePosition(j, i, player);
              
              // 撤销落子
              gameData.board[i][j] = null;
              
              // 更新最佳位置
              if (score > bestScore) {
                bestScore = score;
                bestMove = { x: j, y: i, score };
              }
            }
          }
        }
        
        return bestMove;
      }
      
      // 评估一个位置的分数
      function evaluatePosition(x, y, player) {
        let score = 0;
        const directions = [
          { dx: 1, dy: 0 },  // 水平
          { dx: 0, dy: 1 },  // 垂直
          { dx: 1, dy: 1 },  // 对角线
          { dx: 1, dy: -1 }  // 反对角线
        ];
        
        for (const dir of directions) {
          const line = getLine(x, y, dir.dx, dir.dy, player);
          score += evaluateLine(line);
        }
        
        return score;
      }
      
      // 检查胜利
      function checkWin(x, y, player) {
        const directions = [
          { dx: 1, dy: 0 },  // 水平
          { dx: 0, dy: 1 },  // 垂直
          { dx: 1, dy: 1 },  // 对角线
          { dx: 1, dy: -1 }  // 反对角线
        ];
        
        for (const dir of directions) {
          let count = 1;  // 当前位置已经有一个棋子
          
          // 正方向
          for (let i = 1; i < 5; i++) {
            const nx = x + dir.dx * i;
            const ny = y + dir.dy * i;
            
            if (nx < 0 || nx >= config.boardSize || ny < 0 || ny >= config.boardSize) {
              break;
            }
            
            if (gameData.board[ny][nx] === player) {
              count++;
            } else {
              break;
            }
          }
          
          // 反方向
          for (let i = 1; i < 5; i++) {
            const nx = x - dir.dx * i;
            const ny = y - dir.dy * i;
            
            if (nx < 0 || nx >= config.boardSize || ny < 0 || ny >= config.boardSize) {
              break;
            }
            
            if (gameData.board[ny][nx] === player) {
              count++;
            } else {
              break;
            }
          }
          
          // 如果有五子连珠，记录胜利线并返回true
          if (count >= 5) {
            gameData.winLine = {
              start: { x: x - dir.dx * (count - 1), y: y - dir.dy * (count - 1) },
              end: { x: x + dir.dx * (count - 1), y: y + dir.dy * (count - 1) }
            };
            return true;
          }
        }
        
        return false;
      }
      
      // 检查胜利（不保存胜利线）
      function checkWinWithoutSaving(x, y, player) {
        const directions = [
          { dx: 1, dy: 0 },  // 水平
          { dx: 0, dy: 1 },  // 垂直
          { dx: 1, dy: 1 },  // 对角线
          { dx: 1, dy: -1 }  // 反对角线
        ];
        
        for (const dir of directions) {
          let count = 0;
          
          // 检查五个连续位置
          for (let i = -4; i <= 0; i++) {
            count = 0;
            for (let j = 0; j < 5; j++) {
              const nx = x + dir.dx * (i + j);
              const ny = y + dir.dy * (i + j);
              
              if (nx < 0 || nx >= config.boardSize || ny < 0 || ny >= config.boardSize) {
                break;
              }
              
              if (gameData.board[ny][nx] === player) {
                count++;
              } else {
                break;
              }
            }
            
            if (count >= 5) {
              return true;
            }
          }
        }
        
        return false;
      }
      
      // 检查平局
      function checkDraw() {
        for (let i = 0; i < config.boardSize; i++) {
          for (let j = 0; j < config.boardSize; j++) {
            if (gameData.board[i][j] === null) {
              return false;
            }
          }
        }
        return true;
      }
      
      // 显示结果模态框
      function showResultModal() {
        if (config.winner === 'tie') {
          winnerIcon.innerHTML = `
            <i class="fa fa-handshake-o text-4xl text-gray-500"></i>
          `;
          document.querySelector('#result-modal h2').textContent = '平局！';
          document.querySelector('#result-modal p').textContent = '双方势均力敌，难分胜负！';
        } else {
          const winnerColor = config.winner === 'black' ? 'black' : 'white';
          winnerIcon.innerHTML = `
            <div class="w-12 h-12 rounded-full bg-${winnerColor}"></div>
          `;
          const winnerText = config.winner === 'black' ? '黑方' : '白方';
          document.querySelector('#result-modal h2').textContent = `${winnerText}获胜!`;
          document.querySelector('#result-modal p').textContent = `恭喜你赢得了这场精彩的比赛!`;
        }
        
        resultModal.classList.remove('hidden');
      }
      
      // 隐藏结果模态框
      function hideResultModal() {
        resultModal.classList.add('hidden');
      }
      
      // 悔棋
      function undoMove() {
        if (gameData.history.length === 0 || config.gameStatus === 'over') {
          return;
        }
        
        // 撤销最后一步
        const lastMove = gameData.history.pop();
        gameData.board[lastMove.y][lastMove.x] = null;
        
        // 如果是AI对战，需要撤销AI的上一步
        if (config.gameMode === 'ai' && gameData.history.length > 0) {
          const aiMove = gameData.history.pop();
          gameData.board[aiMove.y][aiMove.x] = null;
        }
        
        // 重绘棋盘
        gameData.winLine = null;
        winLineEl.classList.add('hidden');
        drawBoard();
        
        // 更新游戏状态
        config.gameStatus = 'playing';
        config.winner = null;
        updateGameInfo();
      }
      
      // 重新开始游戏
      function restartGame() {
        initBoard();
        drawBoard();
        hideResultModal();
      }
      
      // 初始化事件监听
      function initEventListeners() {
        // 棋盘点击事件
        canvas.addEventListener('click', (e) => {
          if (config.gameStatus !== 'playing') return;
          
          const rect = canvas.getBoundingClientRect();
          const scaleX = canvas.width / rect.width;
          const scaleY = canvas.height / rect.height;
          
          const clickX = (e.clientX - rect.left) * scaleX;
          const clickY = (e.clientY - rect.top) * scaleY;
          
          const x = Math.round((clickX - config.cellSize / 2) / config.cellSize);
          const y = Math.round((clickY - config.cellSize / 2) / config.cellSize);
          
          if (x >= 0 && x < config.boardSize && y >= 0 && y < config.boardSize) {
            placePiece(x, y);
          }
        });
        
        // 棋盘大小选择
        document.getElementById('size-9').addEventListener('click', () => setBoardSize(9));
        document.getElementById('size-13').addEventListener('click', () => setBoardSize(13));
        document.getElementById('size-19').addEventListener('click', () => setBoardSize(19));
        
        // 游戏模式选择
        document.getElementById('mode-pvp').addEventListener('click', () => setGameMode('pvp'));
        document.getElementById('mode-ai').addEventListener('click', () => setGameMode('ai'));
        
        // AI难度选择
        document.getElementById('difficulty-easy').addEventListener('click', () => setAIDifficulty('easy'));
        document.getElementById('difficulty-medium').addEventListener('click', () => setAIDifficulty('medium'));
        document.getElementById('difficulty-hard').addEventListener('click', () => setAIDifficulty('hard'));
        document.getElementById('difficulty-expert').addEventListener('click', () => setAIDifficulty('expert'));
        
        // 按钮事件
        startGameBtn.addEventListener('click', restartGame);
        undoMoveBtn.addEventListener('click', undoMove);
        restartGameBtn.addEventListener('click', restartGame);
        
        // 结果模态框按钮
        playAgainBtn.addEventListener('click', restartGame);
        closeResultBtn.addEventListener('click', hideResultModal);
        
        // 窗口大小变化时重绘棋盘
        window.addEventListener('resize', drawBoard);
      }
      
      // 初始化游戏
      function initGame() {
        resizeCanvas();
        initBoard();
        initEventListeners();
        drawBoard();
      }
      
      // 启动游戏
      initGame();
    });
  </script>
</body>
</html>
    
