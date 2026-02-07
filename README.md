# runing 🎮 🎯 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Business Pro Catch</title>
    <style>
        body { margin: 0; background: #1a1a1a; color: white; font-family: 'Segoe UI', sans-serif; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; overflow: hidden; }
        canvas { background: #2c3e50; border: 4px solid #34495e; border-radius: 8px; cursor: none; box-shadow: 0 0 30px rgba(0,0,0,0.5); }
        .ui-container { position: absolute; text-align: center; width: 320px; background: rgba(0,0,0,0.85); padding: 30px; border-radius: 15px; border: 2px solid #2ecc71; }
        .hidden { display: none; }
        button { background: #27ae60; border: none; padding: 12px 25px; color: white; font-size: 1.2rem; border-radius: 5px; cursor: pointer; margin-top: 15px; transition: 0.3s; }
        button:hover { background: #2ecc71; transform: scale(1.05); }
        .stats { position: absolute; top: 15px; width: 380px; display: flex; justify-content: space-between; font-size: 1.3rem; font-weight: bold; pointer-events: none; text-shadow: 2px 2px 4px rgba(0,0,0,0.8); }
        #highScoreLabel { color: #f1c40f; font-size: 0.9rem; margin-top: 5px; }
    </style>
</head>
<body>

    <div class="stats">
        <div>Score: <span id="scoreText">0</span></div>
        <div>Lives: <span id="livesText">3</span></div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <div id="startScreen" class="ui-container">
        <h1>Catch The Items!</h1>
        <p>Move the bar to catch falling products.<br>Gold items give bonus points + life!</p>
        <button onclick="startGame()">Start Game</button>
    </div>

    <div id="gameOverScreen" class="ui-container hidden">
        <h1>Game Over!</h1>
        <p>Your Score: <span id="finalScore">0</span></p>
        <p id="highScoreLabel">Best: <span id="highScoreText">0</span></p>
        <button onclick="startGame()">Try Again</button>
    </div>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreText = document.getElementById("scoreText");
    const livesText = document.getElementById("livesText");
    const startScreen = document.getElementById("startScreen");
    const gameOverScreen = document.getElementById("gameOverScreen");
    
    canvas.width = 400;
    canvas.height = 600;

    // --- GAME VARIABLES ---
    let score, lives, items, frameCount, gameState = 'START';
    let highScore = localStorage.getItem("bizGameBest") || 0;

    const player = { x: 160, y: 530, w: 80, h: 15, color: "#2ecc71" };

    // --- ASSETS ---
    const bgImage = new Image();
    bgImage.src = 'https://images.unsplash.com/photo-1534452286302-29691f955288?auto=format&fit=crop&w=400&q=80';

    // --- CONTROLS ---
    canvas.addEventListener('mousemove', (e) => {
        const rect = canvas.getBoundingClientRect();
        player.x = (e.clientX - rect.left) - player.w / 2;
        // Keep player in bounds
        if (player.x < 0) player.x = 0;
        if (player.x > canvas.width - player.w) player.x = canvas.width - player.w;
    });

    // --- CORE LOGIC ---
    function startGame() {
        score = 0;
        lives = 3;
        items = [];
        frameCount = 0;
        gameState = 'PLAYING';
        startScreen.classList.add('hidden');
        gameOverScreen.classList.add('hidden');
        updateUI();
        loop();
    }

    function spawnItem() {
        const isSpecial = Math.random() < 0.12; // 12% chance for gold
        items.push({
            x: Math.random() * (canvas.width - 25),
            y: -30,
            size: 20,
            speed: 3 + (score / 15),
            type: isSpecial ? 'GOLD' : 'NORMAL',
            color: isSpecial ? "#f1c40f" : "#e74c3c"
        });
    }

    function update() {
        if (gameState !== 'PLAYING') return;

        frameCount++;
        // Difficulty: items spawn faster as score increases
        if (frameCount % Math.max(20, 50 - Math.floor(score/3)) === 0) {
            spawnItem();
        }

        for (let i = items.length - 1; i >= 0; i--) {
            items[i].y += items[i].speed;

            // Catch Collision
            if (items[i].y + items[i].size > player.y && 
                items[i].x < player.x + player.w && 
                items[i].x + items[i].size > player.x) {
                
                if(items[i].type === 'GOLD') {
                    score += 5;
                    if(lives < 3) lives++; 
                } else {
                    score++;
                }
                items.splice(i, 1);
                updateUI();
            } 
            // Missed Item
            else if (items[i].y > canvas.height) {
                if(items[i].type === 'NORMAL') {
                    lives--;
                    updateUI();
                    if (lives <= 0) endGame();
                }
                items.splice(i, 1);
            }
        }
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 1. Draw Background
        if (bgImage.complete) {
            ctx.drawImage(bgImage, 0, 0, canvas.width, canvas.height);
            ctx.fillStyle = "rgba(0, 0, 0, 0.5)"; 
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        // 2. Draw Player
        ctx.fillStyle = player.color;
        ctx.shadowBlur = 15;
        ctx.shadowColor = player.color;
        ctx.fillRect(player.x, player.y, player.w, player.h);

        // 3. Draw Items
        items.forEach(item => {
            ctx.fillStyle = item.color;
            ctx.shadowColor = item.color;
            ctx.beginPath();
            if(item.type === 'GOLD') {
                // Diamond shape for special items
                ctx.moveTo(item.x + 10, item.y);
                ctx.lineTo(item.x + 20, item.y + 10);
                ctx.lineTo(item.x + 10, item.y + 20);
                ctx.lineTo(item.x, item.y + 10);
            } else {
                ctx.arc(item.x + 10, item.y + 10, 10, 0, Math.PI * 2);
            }
            ctx.fill();
        });
        ctx.shadowBlur = 0; // Reset shadow
    }

    function updateUI() {
        scoreText.innerText = score;
        livesText.innerText = lives;
    }

    function endGame() {
        gameState = 'GAMEOVER';
        if (score > highScore) {
            highScore = score;
            localStorage.setItem("bizGameBest", highScore);
        }
        document.getElementById("finalScore").innerText = score;
        document.getElementById("highScoreText").innerText = highScore;
        gameOverScreen.classList.remove('hidden');
    }

    function loop() {
        if (gameState === 'PLAYING') {
            update();
            draw();
            requestAnimationFrame(loop);
        }
    }
</script>
</body>
</html>


Short game 🎯🎮
