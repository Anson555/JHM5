html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>飞机与高楼井字游戏</title>
    <style>
        body {
            font-family: "Arial", sans-serif;
            text-align: center;
            background-color: #e6f7ff;
            margin: 0;
            padding: 20px;
        }
        
        .game-title {
            color: #2c3e50;
            margin-bottom: 30px;
        }
        
        .controls {
            margin: 20px 0;
        }
        
        button {
            padding: 10px 20px;
            margin: 0 8px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            background-color: #3498db;
            color: white;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: #2980b9;
        }
        
        #status {
            font-size: 22px;
            margin: 20px 0;
            padding: 10px;
            border-radius: 5px;
            background-color: #f8f9fa;
            display: inline-block;
        }
        
        .board {
            display: grid;
            grid-template-columns: repeat(3, 120px);
            grid-gap: 10px;
            margin: 30px auto;
            max-width: 380px;
        }
        
        .cell {
            width: 120px;
            height: 120px;
            background-color: white;
            border: 3px solid #34495e;
            border-radius: 8px;
            font-size: 50px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: background-color 0.2s;
        }
        
        .cell:hover {
            background-color: #f1c40f;
        }
    </style>
</head>
<body>
    <h1 class="game-title">飞机 vs 高楼 井字游戏</h1>
    
    <div class="controls">
        <button id="singlePlayer">单人游戏</button>
        <button id="twoPlayers">双人游戏</button>
        <button id="restart">重新开始</button>
    </div>
    
    <div id="status">请选择游戏模式</div>
    
    <div class="board" id="board">
        <div class="cell" data-index="0"></div>
        <div class="cell" data-index="1"></div>
        <div class="cell" data-index="2"></div>
        <div class="cell" data-index="3"></div>
        <div class="cell" data-index="4"></div>
        <div class="cell" data-index="5"></div>
        <div class="cell" data-index="6"></div>
        <div class="cell" data-index="7"></div>
        <div class="cell" data-index="8"></div>
    </div>
    
    <!-- 音效资源 -->
    <audio id="placeSound" preload="auto">
        <source src="https://assets.mixkit.co/sfx/preview/mixkit-interface-click-1109.mp3" type="audio/mpeg">
    </audio>
     
    <audio id="planeWinSound" preload="auto">
        <source src="https://assets.mixkit.co/sfx/preview/mixkit-airplane-landing-1943.mp3" type="audio/mpeg">
    </audio>
     
    <audio id="buildingWinSound" preload="auto">
        <source src="https://assets.mixkit.co/sfx/preview/mixkit-explosion-impact-1699.mp3" type="audio/mpeg">
    </audio>
      
    <audio id="drawSound" preload="auto">
        <source src="https://assets.mixkit.co/sfx/preview/mixkit-game-show-wrong-answer-fail-notification-946.mp3" type="audio/mpeg">
    </audio>

    <script>
        // 获取DOM元素
        const cells = document.querySelectorAll('.cell');
        const statusText = document.getElementById('status');
        const singlePlayerBtn = document.getElementById('singlePlayer');
        const twoPlayersBtn = document.getElementById('twoPlayers');
        const restartBtn = document.getElementById('restart');
        
        // 音效元素
        const placeSound = document.getElementById('placeSound');
        const planeWinSound = document.getElementById('planeWinSound');
        const buildingWinSound = document.getElementById('buildingWinSound');
        const drawSound = document.getElementById('drawSound');
        
        // 游戏状态变量
        let currentPlayer = '✈️'; // 飞机先开始
        let gameActive = false;
        let gameMode = ''; // 'single' 或 'two'
        let gameState = ['', '', '', '', '', '', '', '', ''];
        
        // 获胜组合
        const winningCombinations = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8], // 横向
            [0, 3, 6], [1, 4, 7], [2, 5, 8], // 纵向
            [0, 4, 8], [2, 4, 6] // 对角线
        ];
        
        // 初始化游戏
        function initGame(mode) {
            gameMode = mode;
            gameActive = true;
            gameState = ['', '', '', '', '', '', '', '', ''];
            currentPlayer = '✈️';
            cells.forEach(cell => {
                cell.textContent = '';
                cell.style.pointerEvents = 'auto';
                cell.style.backgroundColor = 'white';
            });
            statusText.textContent = `当前回合: ${currentPlayer} 飞机`;
        }
        
        // 检查游戏结果
        function checkResult() {
            let roundWon = false;
            
            // 检查是否有获胜组合
            for (let i = 0; i < winningCombinations.length; i++) {
                const [a, b, c] = winningCombinations[i];
                if (gameState[a] && gameState[a] === gameState[b] && gameState[a] === gameState[c]) {
                    roundWon = true;
                    // 高亮获胜的单元格
                    cells[a].style.backgroundColor = '#2ecc71';
                    cells[b].style.backgroundColor = '#2ecc71';
                    cells[c].style.backgroundColor = '#2ecc71';
                    break;
                }
            }

            if (roundWon) {
                // 根据获胜者播放对应音效
                if (currentPlayer === '✈️') {
                    statusText.textContent = '飞机获胜！';
                    planeWinSound.play();
                } else {
                    statusText.textContent = '高楼获胜！';
                    buildingWinSound.play();
                }
                gameActive = false;
                cells.forEach(cell => cell.style.pointerEvents = 'none');
                return;
            }
            
            // 检查平局
            if (!gameState.includes('')) {
                statusText.textContent = '平局！';
                drawSound.play();
                gameActive = false;
                return;
            }
            
            // 切换玩家
            currentPlayer = currentPlayer === '✈️' ? '🏙️' : '✈️';
            statusText.textContent = `当前回合: ${currentPlayer} ${currentPlayer === '✈️' ? '飞机' : '高楼'}`;
            
            // 如果是单人模式且轮到AI
            if (gameMode === 'single' && currentPlayer === '🏙️' && gameActive) {
                setTimeout(aiMove, 600); // 延迟一下，模拟思考
            }
        }
        
        // AI移动逻辑
        function aiMove() {
            // 1. 尝试获胜
            for (let i = 0; i < gameState.length; i++) {
                if (!gameState[i]) {
                    gameState[i] = '🏙️';
                    if (checkWin('🏙️')) {
                        makeMove(i, '🏙️');
                        return;
                    }
                    gameState[i] = ''; // 重置
                }
            }
            
            // 2. 阻止玩家获胜
            for (let i = 0; i < gameState.length; i++) {
                if (!gameState[i]) {
                    gameState[i] = '✈️';
                    if (checkWin('✈️')) {
                        makeMove(i, '🏙️');
                        return;
                    }
                    gameState[i] = ''; // 重置
                }
            }
            
            // 3. 尝试占领中心
            if (!gameState[4]) {
                makeMove(4, '🏙️');
                return;
            }
            
            // 4. 随机选择空位
            let emptyCells = [];
            gameState.forEach((cell, index) => {
                if (!cell) emptyCells.push(index);
            });
            const randomIndex = emptyCells[Math.floor(Math.random() * emptyCells.length)];
            makeMove(randomIndex, '🏙️');
        }
        
        // 检查指定玩家是否获胜
        function checkWin(player) {
            return winningCombinations.some(combination => {
                return combination.every(index => gameState[index] === player);
            });
        }
        
        // 执行移动
        function makeMove(index, player) {
            if (gameState[index] || !gameActive) return;
            
            gameState[index] = player;
            cells[index].textContent = player;
            cells[index].style.pointerEvents = 'none';
            placeSound.play();
            checkResult();
        }
        
        // 处理单元格点击
        function handleCellClick(e) {
            const index = parseInt(e.target.dataset.index);
            makeMove(index, currentPlayer);
        }
        
        // 事件监听
        cells.forEach(cell => cell.addEventListener('click', handleCellClick));
        singlePlayerBtn.addEventListener('click', () => initGame('single'));
        twoPlayersBtn.addEventListener('click', () => initGame('two'));
        restartBtn.addEventListener('click', () => gameMode ? initGame(gameMode) : null);
    </script>
</body>
</html>
