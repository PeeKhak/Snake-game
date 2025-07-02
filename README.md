<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Colorful Snake Game</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
            overflow: hidden;
        }
        #gameContainer {
            text-align: center;
        }
        #gameCanvas {
            border: 3px solid #333;
            background-color: #000;
            display: none;
        }
        #levelSelect {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #startButton, #restartButton, .level-button {
            padding: 35px 70px;
            font-size: 32px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 12px;
            cursor: pointer;
            margin: 20px;
            transition: transform 0.2s, box-shadow 0.2s;
            box-shadow: 0 0 12px rgba(0, 255, 0, 0.5);
        }
        #startButton:hover, #restartButton:hover, .level-button:hover {
            transform: scale(1.1);
            box-shadow: 0 0 24px rgba(0, 255, 0, 0.8);
        }
        #pauseButton, #continueButton {
            padding: 30px;
            font-size: 34px;
            background-color: #FF9800;
            color: white;
            border: none;
            border-radius: 12px;
            cursor: pointer;
            margin: 15px;
            transition: transform 0.2s, box-shadow 0.2s;
            box-shadow: 0 0 12px rgba(255, 152, 0, 0.5);
        }
        #pauseButton:hover, #continueButton:hover {
            transform: scale(1.1);
            box-shadow: 0 0 24px rgba(255, 152, 0, 0.8);
        }
        #score {
            font-size: 32px;
            margin: 20px 0;
            display: none;
            color: #333;
        }
        #levelDisplay {
            font-size: 24px;
            margin: 10px 0;
            display: none;
            color: #333;
        }
        #controls {
            display: none;
            margin-top: 25px;
        }
        .control-button {
            padding: 30px;
            font-size: 34px;
            margin: 15px;
            width: 100px;
            height: 100px;
            background-color: #008CBA;
            color: white;
            border: none;
            border-radius: 12px;
            touch-action: manipulation;
            transition: transform 0.2s, box-shadow 0.2s;
            box-shadow: 0 0 12px rgba(0, 140, 186, 0.5);
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .control-button:active, .control-button:hover {
            transform: scale(1.1);
            box-shadow: 0 0 24px rgba(0, 140, 186, 0.8);
        }
        #credits {
            font-size: 20px;
            color: #333;
            margin-top: 15px;
            text-align: center;
        }
        #credits p {
            margin: 8px 0;
        }
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.5); }
            100% { transform: scale(1); }
        }
        .control-row {
            display: flex;
            justify-content: center;
            align-items: center;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <button id="startButton">Play Game</button>
        <div id="levelSelect" style="display: none;">
            <button class="level-button" onclick="startGameWithLevel('easy')">Easy</button>
            <button class="level-button" onclick="startGameWithLevel('normal')">Normal</button>
            <button class="level-button" onclick="startGameWithLevel('hard')">Hard</button>
        </div>
        <button id="restartButton" style="display: none;">Restart</button>
        <div id="score">Score: 0</div>
        <div id="levelDisplay"></div>
        <canvas id="gameCanvas" width="600" height="600"></canvas>
        <div id="controls">
            <div class="control-row">
                <button class="control-button" ontouchstart="changeDirection('up')" onclick="changeDirection('up')">↑</button>
            </div>
            <div class="control-row">
                <button class="control-button" ontouchstart="changeDirection('left')" onclick="changeDirection('left')">←</button>
                <button id="pauseButton" style="display: none;">Pause</button>
                <button id="continueButton" style="display: none;">Continue</button>
                <button class="control-button" ontouchstart="changeDirection('right')" onclick="changeDirection('right')">→</button>
            </div>
            <div class="control-row">
                <button class="control-button" ontouchstart="changeDirection('down')" onclick="changeDirection('down')">↓</button>
            </div>
        </div>
        <div id="credits">
            <p>@Credit By PeeKhak</p>
            <p>Thank you for playing. <3</p>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startButton = document.getElementById('startButton');
        const levelSelect = document.getElementById('levelSelect');
        const pauseButton = document.getElementById('pauseButton');
        const continueButton = document.getElementById('continueButton');
        const restartButton = document.getElementById('restartButton');
        const scoreDisplay = document.getElementById('score');
        const levelDisplay = document.getElementById('levelDisplay');
        const controls = document.getElementById('controls');
        const gridSize = 30;
        const tileCount = canvas.width / gridSize;
        let snake = [{ x: 10, y: 10 }];
        let food = { x: 15, y: 15 };
        let dx = 0;
        let dy = 0;
        let score = 0;
        let gameLoop;
        let bounceScale = 1;
        let bounceDirection = 0.07;
        let isPaused = false;
        let trail = [];
        let gameSpeed = 100; // Default: Normal

        function getRandomColor() {
            const letters = '0123456789ABCDEF';
            let color = '#';
            for (let i = 0; i < 6; i++) {
                color += letters[Math.floor(Math.random() * 16)];
            }
            return color;
        }

        function createGradient(x, y, color1, color2) {
            const gradient = ctx.createRadialGradient(
                x, y, gridSize / 4,
                x, y, gridSize / 2
            );
            gradient.addColorStop(0, color1);
            gradient.addColorStop(1, color2);
            return gradient;
        }

        let snakeColor = getRandomColor();
        let foodColor = getRandomColor();

        function changeDirection(direction) {
            if (isPaused) return;
            switch(direction) {
                case 'up':
                    if (dy === 0) { dx = 0; dy = -1; }
                    break;
                case 'down':
                    if (dy === 0) { dx = 0; dy = 1; }
                    break;
                case 'left':
                    if (dx === 0) { dx = -1; dy = 0; }
                    break;
                case 'right':
                    if (dx === 0) { dx = 1; dy = 0; }
                    break;
            }
        }

        function startGame() {
            startButton.style.display = 'none';
            levelSelect.style.display = 'flex';
        }

        function startGameWithLevel(level) {
            levelSelect.style.display = 'none';
            pauseButton.style.display = 'block';
            restartButton.style.display = 'block';
            canvas.style.display = 'block';
            scoreDisplay.style.display = 'block';
            levelDisplay.style.display = 'block';
            controls.style.display = 'block';
            if (level === 'easy') {
                gameSpeed = 150;
                levelDisplay.textContent = 'Level: Easy';
            } else if (level === 'normal') {
                gameSpeed = 100;
                levelDisplay.textContent = 'Level: Normal';
            } else {
                gameSpeed = 70;
                levelDisplay.textContent = 'Level: Hard';
            }
            gameLoop = setInterval(game, gameSpeed);
        }

        function pauseGame() {
            isPaused = true;
            clearInterval(gameLoop);
            pauseButton.style.display = 'none';
            continueButton.style.display = 'block';
        }

        function continueGame() {
            if (!isPaused) return;
            isPaused = false;
            gameLoop = setInterval(game, gameSpeed);
            continueButton.style.display = 'none';
            pauseButton.style.display = 'block';
        }

        function restartGame() {
            clearInterval(gameLoop);
            snake = [{ x: 10, y: 10 }];
            food = { x: 15, y: 15 };
            dx = 0;
            dy = 0;
            score = 0;
            isPaused = false;
            trail = [];
            scoreDisplay.textContent = `Score: ${score}`;
            snakeColor = getRandomColor();
            foodColor = getRandomColor();
            pauseButton.style.display = 'block';
            continueButton.style.display = 'none';
            gameLoop = setInterval(game, gameSpeed);
        }

        function game() {
            if (isPaused) return;
            updateSnake();
            if (checkCollision()) {
                endGame();
                return;
            }
            drawGame();
        }

        function updateSnake() {
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };
            snake.unshift(head);
            trail.push({ x: head.x, y: head.y, alpha: 0.5 });
            if (trail.length > 10) trail.shift();

            if (head.x === food.x && head.y === food.y) {
                score += 10;
                scoreDisplay.textContent = `Score: ${score}`;
                food = {
                    x: Math.floor(Math.random() * tileCount),
                    y: Math.floor(Math.random() * tileCount)
                };
                foodColor = getRandomColor();
                snakeColor = getRandomColor();
            } else {
                snake.pop();
                trail.forEach(t => t.alpha -= 0.05);
                trail = trail.filter(t => t.alpha > 0);
            }

            bounceScale += bounceDirection;
            if (bounceScale > 1.5 || bounceScale < 0.5) {
                bounceDirection = -bounceDirection;
            }
        }

        function checkCollision() {
            const head = snake[0];
            if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
                return true;
            }
            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    return true;
                }
            }
            return false;
        }

        function drawGame() {
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            trail.forEach(t => {
                ctx.fillStyle = `rgba(${parseInt(snakeColor.slice(1,3),16)},${parseInt(snakeColor.slice(3,5),16)},${parseInt(snakeColor.slice(5,7),16)},${t.alpha})`;
                ctx.beginPath();
                ctx.arc((t.x + 0.5) * gridSize, (t.y + 0.5) * gridSize, gridSize / 2 * 0.8, 0, Math.PI * 2);
                ctx.fill();
            });

            ctx.fillStyle = createGradient((food.x + 0.5) * gridSize, (food.y + 0.5) * gridSize, foodColor, '#fff');
            ctx.beginPath();
            ctx.arc((food.x + 0.5) * gridSize, (food.y + 0.5) * gridSize, gridSize / 2 * bounceScale, 0, Math.PI * 2);
            ctx.fill();
            ctx.strokeStyle = '#fff';
            ctx.stroke();

            snake.forEach(segment => {
                ctx.fillStyle = createGradient((segment.x + 0.5) * gridSize, (segment.y + 0.5) * gridSize, snakeColor, '#fff');
                ctx.beginPath();
                ctx.arc((segment.x + 0.5) * gridSize, (segment.y + 0.5) * gridSize, gridSize / 2 * bounceScale, 0, Math.PI * 2);
                ctx.fill();
                ctx.strokeStyle = '#fff';
                ctx.stroke();
            });
        }

        function endGame() {
            clearInterval(gameLoop);
            isPaused = false;
            alert(`Game Over! Your score: ${score}`);
            snake = [{ x: 10, y: 10 }];
            food = { x: 15, y: 15 };
            dx = 0;
            dy = 0;
            score = 0;
            trail = [];
            scoreDisplay.textContent = `Score: ${score}`;
            levelDisplay.textContent = '';
            canvas.style.display = 'none';
            scoreDisplay.style.display = 'none';
            levelDisplay.style.display = 'none';
            controls.style.display = 'none';
            pauseButton.style.display = 'none';
            continueButton.style.display = 'none';
            restartButton.style.display = 'none';
            levelSelect.style.display = 'none';
            startButton.style.display = 'block';
            snakeColor = getRandomColor();
            foodColor = getRandomColor();
        }

        document.addEventListener('keydown', (event) => {
            if (isPaused) return;
            switch(event.key) {
                case 'ArrowUp':
                    changeDirection('up');
                    break;
                case 'ArrowDown':
                    changeDirection('down');
                    break;
                case 'ArrowLeft':
                    changeDirection('left');
                    break;
                case 'ArrowRight':
                    changeDirection('right');
                    break;
            }
        });

        startButton.addEventListener('click', startGame);
        pauseButton.addEventListener('click', pauseGame);
        continueButton.addEventListener('click', continueGame);
        restartButton.addEventListener('click', restartGame);
    </script>
</body>
</html>
