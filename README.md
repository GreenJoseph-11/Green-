<html lang="zh-Hans">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GreenJoseph网页</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            line-height: 1.6;
            background-color: #f8f8f8;
        }

        h1 {
            color: #2e8b57;
        }

        h2 {
            color: #3cb371;
        }

        blockquote {
            background-color: #f5f5f5;
            padding: 15px;
            border-left: 4px solid #2e8b57;
            margin: 20px 0;
        }

        #game-container {
            margin-top: 40px;
            padding: 20px;
            border: 2px solid #2e8b57;
            border-radius: 8px;
            background-color: #fff;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }

        .game-header {
            display: flex;
            justify-content: space-between;
            margin-bottom: 15px;
            flex-wrap: wrap;
        }

        #game-board {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        #tetris {
            border: 2px solid #2e8b57;
            background-color: #000;
            margin-bottom: 15px;
        }

        .game-info {
            display: flex;
            justify-content: space-between;
            width: 100%;
            margin-bottom: 15px;
        }

        .info-box {
            padding: 10px;
            background-color: #f0f8f4;
            border-radius: 5px;
            text-align: center;
            min-width: 80px;
        }

        .controls {
            margin-top: 20px;
            padding: 15px;
            background-color: #f0f8f4;
            border-radius: 5px;
            width: 100%;
        }

        .controls h3 {
            color: #2e8b57;
            margin-top: 0;
            text-align: center;
        }

        .key {
            display: inline-block;
            padding: 3px 8px;
            background-color: #2e8b57;
            color: white;
            border-radius: 4px;
            margin: 0 5px;
            font-weight: bold;
            min-width: 30px;
            text-align: center;
        }

        .key-row {
            display: flex;
            justify-content: space-between;
            margin: 10px 0;
        }

        button {
            background-color: #2e8b57;
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 4px;
            cursor: pointer;
            margin: 5px;
            font-weight: bold;
        }

        button:hover {
            background-color: #3cb371;
        }

        .buttons {
            display: flex;
            justify-content: center;
            margin-top: 15px;
        }

        .score-display {
            font-size: 18px;
            font-weight: bold;
            color: #2e8b57;
        }
    </style>
</head>

<body>
    <h1>Welcome to my homepage!</h1>
    <h2>Hello!Green!</h2>
    <p><strong>在这里你会看到许多的，好看的，有趣的东西！</strong></p >
    <p><strong>保护自己最重要！</strong></p >
    <h1>编写者：b站@GreenJoseph
        <ul>
        </ul>
        <blockquote cite="#">
            "进步从人民中来，<br>真理回人民中去"
        </blockquote>

    <!-- 精简版俄罗斯方块游戏 -->
    <div id="game-container">
        <h2>俄罗斯方块小游戏</h2>
        
        <div class="game-header">
            <div class="info-box">
                <div>分数</div>
                <div id="score">0</div>
            </div>
            <div class="info-box">
                <div>行数</div>
                <div id="lines">0</div>
            </div>
        </div>
        
        <div id="game-board">
            <canvas id="tetris" width="240" height="480"></canvas>
            
            <div class="controls">
                <h3>游戏控制 (WASD)</h3>
                <div class="key-row">
                    <div><span class="key">A</span> 左移</div>
                    <div><span class="key">D</span> 右移</div>
                </div>
                <div class="key-row">
                    <div><span class="key">S</span> 加速</div>
                    <div><span class="key">W</span> 旋转</div>
                </div>
                <div class="key-row">
                    <div><span class="key">空格</span> 硬降</div>
                    <div><span class="key">P</span> 暂停</div>
                </div>
                
                <div class="buttons">
                    <button id="start-btn">开始</button>
                    <button id="pause-btn">暂停</button>
                    <button id="reset-btn">重置</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 游戏常量
        const COLS = 10;
        const ROWS = 20;
        const BLOCK_SIZE = 24;
        const COLORS = ['#000', '#FF0D72', '#0DC2FF', '#0DFF72', '#F538FF', '#FF8E0D', '#FFE138', '#3877FF'];
        
        // 方块形状
        const SHAPES = [
            [],
            [[0,0,0,0], [1,1,1,1], [0,0,0,0], [0,0,0,0]], // I
            [[2,0,0], [2,2,2], [0,0,0]], // J
            [[0,0,3], [3,3,3], [0,0,0]], // L
            [[4,4], [4,4]], // O
            [[0,5,5], [5,5,0], [0,0,0]], // S
            [[0,6,0], [6,6,6], [0,0,0]], // T
            [[7,7,0], [0,7,7], [0,0,0]]  // Z
        ];
        
        // 游戏变量
        let canvas, ctx, board, piece, score, lines, gameOver, paused, dropCounter, dropInterval, lastTime, requestId;
        
        // 初始化游戏
        function initGame() {
            canvas = document.getElementById('tetris');
            ctx = canvas.getContext('2d');
            
            resetGame();
            drawBoard();
            addEventListeners();
        }
        
        // 重置游戏
        function resetGame() {
            board = Array.from({length: ROWS}, () => Array(COLS).fill(0));
            piece = createPiece();
            score = 0;
            lines = 0;
            gameOver = false;
            paused = false;
            dropInterval = 1000;
            
            updateScoreDisplay();
        }
        
        // 创建方块
        function createPiece() {
            const type = Math.floor(Math.random() * 7) + 1;
            return {
                shape: SHAPES[type],
                color: COLORS[type],
                x: Math.floor(COLS / 2) - 1,
                y: 0
            };
        }
        
        // 绘制游戏
        function drawBoard() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 绘制已放置的方块
            for (let y = 0; y < ROWS; y++) {
                for (let x = 0; x < COLS; x++) {
                    if (board[y][x]) {
                        drawBlock(x, y, COLORS[board[y][x]]);
                    }
                }
            }
            
            // 绘制当前方块
            if (piece) {
                piece.shape.forEach((row, y) => {
                    row.forEach((value, x) => {
                        if (value) {
                            drawBlock(piece.x + x, piece.y + y, piece.color);
                        }
                    });
                });
            }
        }
        
        // 绘制单个方块
        function drawBlock(x, y, color) {
            ctx.fillStyle = color;
            ctx.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
            
            ctx.strokeStyle = '#000';
            ctx.strokeRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
        }
        
        // 碰撞检测
        function collide() {
            for (let y = 0; y < piece.shape.length; y++) {
                for (let x = 0; x < piece.shape[y].length; x++) {
                    if (piece.shape[y][x] && 
                        (piece.y + y >= ROWS || 
                         piece.x + x < 0 || 
                         piece.x + x >= COLS || 
                         board[piece.y + y][piece.x + x])) {
                        return true;
                    }
                }
            }
            return false;
        }
        
        // 旋转方块
        function rotate() {
            if (piece.shape === SHAPES[4]) return; // O形方块不旋转
            
            const oldShape = piece.shape;
            const newShape = [];
            
            // 矩阵转置
            for (let y = 0; y < oldShape[0].length; y++) {
                newShape[y] = [];
                for (let x = 0; x < oldShape.length; x++) {
                    newShape[y][x] = oldShape[oldShape.length - 1 - x][y];
                }
            }
            
            piece.shape = newShape;
            
            // 如果旋转后碰撞，恢复原状
            if (collide()) {
                piece.shape = oldShape;
            }
        }
        
        // 移动方块
        function move(dx) {
            piece.x += dx;
            if (collide()) {
                piece.x -= dx;
            }
        }
        
        // 下落方块
        function drop() {
            piece.y++;
            
            if (collide()) {
                piece.y--;
                mergePiece();
                removeLines();
                piece = createPiece();
                
                if (collide()) {
                    gameOver = true;
                    alert('游戏结束！得分: ' + score);
                }
            }
            
            dropCounter = 0;
        }
        
        // 硬降（直接落到底部）
        function hardDrop() {
            while (!collide()) {
                piece.y++;
            }
            piece.y--;
            drop();
        }
        
        // 合并方块到游戏板
        function mergePiece() {
            piece.shape.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        board[piece.y + y][piece.x + x] = SHAPES.findIndex(shape => shape === piece.shape);
                    }
                });
            });
        }
        
        // 移除已填满的行
        function removeLines() {
            let linesRemoved = 0;
            
            for (let y = ROWS - 1; y >= 0; y--) {
                if (board[y].every(value => value !== 0)) {
                    board.splice(y, 1);
                    board.unshift(Array(COLS).fill(0));
                    linesRemoved++;
                    y++;
                }
            }
            
            if (linesRemoved > 0) {
                lines += linesRemoved;
                // 每行5分
                score += linesRemoved * 5;
                
                updateScoreDisplay();
            }
        }
        
        // 更新分数显示
        function updateScoreDisplay() {
            document.getElementById('score').textContent = score;
            document.getElementById('lines').textContent = lines;
        }
        
        // 游戏主循环
        function gameLoop(time = 0) {
            if (gameOver) {
                cancelAnimationFrame(requestId);
                return;
            }
            
            const deltaTime = time - lastTime;
            lastTime = time;
            
            if (!paused) {
                dropCounter += deltaTime;
                
                if (dropCounter > dropInterval) {
                    drop();
                }
            }
            
            drawBoard();
            requestId = requestAnimationFrame(gameLoop);
        }
        
        // 开始游戏
        function startGame() {
            if (gameOver) {
                resetGame();
            }
            
            if (paused) {
                paused = false;
                document.getElementById('pause-btn').textContent = '暂停';
            }
            
            lastTime = performance.now();
            gameLoop();
        }
        
        // 暂停游戏
        function togglePause() {
            paused = !paused;
            document.getElementById('pause-btn').textContent = paused ? '继续' : '暂停';
            
            if (!paused) {
                lastTime = performance.now();
                gameLoop();
            }
        }
        
        // 添加事件监听器
        function addEventListeners() {
            document.addEventListener('keydown', e => {
                if (gameOver || paused) return;
                
                switch (e.keyCode) {
                    case 65: // A键 (左移)
                    case 37: // 左箭头 (兼容)
                        move(-1); 
                        break;
                    case 68: // D键 (右移)
                    case 39: // 右箭头 (兼容)
                        move(1); 
                        break;
                    case 83: // S键 (加速下落)
                    case 40: // 下箭头 (兼容)
                        drop(); 
                        break;
                    case 87: // W键 (旋转)
                    case 38: // 上箭头 (兼容)
                        rotate(); 
                        break;
                    case 32: // 空格键 (硬降)
                        hardDrop(); 
                        break;
                    case 80: // P键 (暂停)
                        togglePause(); 
                        break;
                }
            });
            
            document.getElementById('start-btn').addEventListener('click', startGame);
            document.getElementById('pause-btn').addEventListener('click', togglePause);
            document.getElementById('reset-btn').addEventListener('click', () => {
                resetGame();
                startGame();
            });
        }
        
        // 初始化游戏
        window.onload = initGame;
    </script>
