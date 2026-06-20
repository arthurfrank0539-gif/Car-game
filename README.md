<!DOCTYPE html>
<html>
<head>
    <title>Neon Rider - Car Game</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            background: radial-gradient(circle, #1a1a2e 0%, #0f0f1b 100%);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #fff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        h1 {
            margin: 10px 0;
            font-size: 2rem;
            text-transform: uppercase;
            letter-spacing: 4px;
            text-shadow: 0 0 10px #00fff2;
        }
        .container {
            position: relative;
            box-shadow: 0 0 30px rgba(0, 255, 242, 0.2);
            border-radius: 12px;
            overflow: hidden;
            width: 90%;
            max-width: 699px;
        }
        canvas {
            background: #111;
            display: block;
            border: 3px solid #00fff2;
            width: 100%;
            height: auto;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 12px 30px;
            font-size: 1.2rem;
            font-weight: bold;
            text-transform: uppercase;
            background: #ff0055;
            color: white;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 15px #ff0055;
            z-index: 10;
        }
        /* Full Touch Controls Pad */
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 90%;
            max-width: 699px;
            margin-top: 15px;
            margin-bottom: 15px;
            gap: 10px;
        }
        .steering-group, .speed-group {
            display: flex;
            gap: 15px;
        }
        .btn {
            width: 75px;
            height: 75px;
            background: rgba(0, 255, 242, 0.1);
            border: 3px solid #00fff2;
            border-radius: 15px;
            color: #00fff2;
            font-size: 2rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 15px rgba(0, 255, 242, 0.2);
            touch-action: none;
        }
        /* Style variation for Action Buttons */
        .btn-action {
            border-color: #ff0055;
            color: #ff0055;
            box-shadow: 0 0 15px rgba(255, 0, 85, 0.2);
        }
        .btn:active {
            background: rgba(0, 255, 242, 0.4);
            box-shadow: 0 0 25px #00fff2;
            color: #fff;
        }
        .btn-action:active {
            background: rgba(255, 0, 85, 0.4);
            box-shadow: 0 0 25px #ff0055;
            color: #fff;
        }
    </style>
</head>
<body>

<h1>Neon Rider</h1>
<div class="container">
    <canvas id="gameCanvas" width="699" height="500"></canvas>
    <button id="restartBtn" onclick="resetGame()">Drive Again</button>
</div>

<div class="controls-pad">
    <div class="steering-group">
        <div class="btn" id="leftBtn">◀</div>
        <div class="btn" id="rightBtn">▶</div>
    </div>
    
    <div class="speed-group">
        <div class="btn btn-action" id="brakeBtn">B</div>
        <div class="btn btn-action" id="accelBtn">A</div>
    </div>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const restartBtn = document.getElementById("restartBtn");

// Game State
let score = 0;
let gameOver = false;
let roadOffset = 0;
let baseSpeed = 5; // Global difficulty baseline

// Car properties
let carX = 325;
let carY = 380;
const carWidth = 45;
const carHeight = 75;
const steerSpeed = 8;

// Obstacle properties
let obstacleX = Math.random() * (canvas.width - 45);
let obstacleY = -80;
const obstacleWidth = 45;
const obstacleHeight = 75;
let currentObstacleSpeed = baseSpeed;

// Touch tracking states
let touchLeft = false;
let touchRight = false;
let touchAccel = false;
let touchBrake = false;

// Setup button listeners
const leftBtn = document.getElementById("leftBtn");
const rightBtn = document.getElementById("rightBtn");
const accelBtn = document.getElementById("accelBtn");
const brakeBtn = document.getElementById("brakeBtn");

leftBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchLeft = true; });
leftBtn.addEventListener("touchend", (e) => { e.preventDefault(); touchLeft = false; });

rightBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchRight = true; });
rightBtn.addEventListener("touchend", (e) => { e.preventDefault(); touchRight = false; });

accelBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchAccel = true; });
accelBtn.addEventListener("touchend", (e) => { e.preventDefault(); touchAccel = false; });

brakeBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchBrake = true; });
brakeBtn.addEventListener("touchend", (e) => { e.preventDefault(); touchBrake = false; });

function checkCollision(rect1X, rect1Y, rect1W, rect1H, rect2X, rect2Y, rect2W, rect2H) {
    return rect1X < rect2X + rect2W &&
           rect1X + rect1W > rect2X &&
           rect1Y < rect2Y + rect2H &&
           rect1Y + rect1H > rect2Y;
}

function resetGame() {
    score = 0;
    gameOver = false;
    carX = 325;
    carY = 380;
    obstacleX = Math.random() * (canvas.width - 45);
    obstacleY = -80;
    baseSpeed = 5;
    restartBtn.style.display = "none";
    gameLoop();
}

function gameLoop() {
    if (gameOver) {
        ctx.fillStyle = "rgba(15, 15, 27, 0.85)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#ff0055";
        ctx.font = "bold 50px Arial";
        ctx.textAlign = "center";
        ctx.shadowColor = "#ff0055";
        ctx.shadowBlur = 15;
        ctx.fillText("CRASHED!", canvas.width / 2, canvas.height / 2 - 20);

        ctx.fillStyle = "#fff";
        ctx.font = "22px Arial";
        ctx.shadowBlur = 0;
        ctx.fillText("Final Score: " + score, canvas.width / 2, canvas.height / 2 + 30);
        
        restartBtn.style.display = "block";
        return;
    }

    // 1. Evaluate Active Steering Controls
    if (touchLeft && carX > 40) carX -= steerSpeed;
    if (touchRight && carX < canvas.width - carWidth - 40) carX += steerSpeed;

    // 2. Evaluate Dynamic Core Engine Velocity (A and B mechanics)
    if (touchAccel) {
        currentObstacleSpeed = baseSpeed * 1.8; // Boost speed significantly
    } else if (touchBrake) {
        currentObstacleSpeed = baseSpeed * 0.4; // Cool down to crawl speed
    } else {
        currentObstacleSpeed = baseSpeed;       // Standard speed setting
    }

    roadOffset += currentObstacleSpeed;
    if (roadOffset > 40) roadOffset = 0;

    obstacleY += currentObstacleSpeed;

    if (obstacleY > canvas.height) {
        obstacleY = -80;
        obstacleX = 40 + Math.random() * (canvas.width - obstacleWidth - 80);
        score += 1;
        baseSpeed += 0.3; // Incremental natural scaling
    }

    if (checkCollision(carX, carY, carWidth, carHeight, obstacleX, obstacleY, obstacleWidth, obstacleHeight)) {
        gameOver = true;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.shadowBlur = 0;

    // Roadway Environment rendering
    ctx.fillStyle = "#22222b";
    ctx.fillRect(40, 0, canvas.width - 80, canvas.height);

    ctx.fillStyle = "#00ff66";
    ctx.fillRect(35, 0, 5, canvas.height);
    ctx.fillRect(canvas.width - 40, 0, 5, canvas.height);

    ctx.fillStyle = "#fff";
    for (let i = -40; i < canvas.height; i += 40) {
        ctx.fillRect(canvas.width / 2 - 3, i + roadOffset, 6, 20);
    }

    // Render Player
    ctx.shadowColor = "#00fff2";
    ctx.shadowBlur = 12;
    ctx.fillStyle = "#00fff2";
    ctx.fillRect(carX, carY, carWidth, carHeight);
    ctx.fillStyle = "#111";
    ctx.fillRect(carX + 5, carY + 15, carWidth - 10, 15);
    ctx.fillStyle = "#fff";
    ctx.fillRect(carX + 4, carY, 6, 4);
    ctx.fillRect(carX + carWidth - 10, carY, 6, 4);

    // Render Obstacle
    ctx.shadowColor = "#ff0055";
    ctx.shadowBlur = 12;
    ctx.fillStyle = "#ff0055";
    ctx.fillRect(obstacleX, obstacleY, obstacleWidth, obstacleHeight);
    ctx.fillStyle = "#111";
    ctx.fillRect(obstacleX + 5, obstacleY + 45, obstacleWidth - 10, 15);

    // Dynamic HUD readout
    ctx.shadowBlur = 0;
    ctx.fillStyle = "rgba(0, 0, 0, 0.5)";
    ctx.fillRect(20, 15, 140, 40);
    ctx.strokeStyle = "#00fff2";
    ctx.lineWidth = 1;
    ctx.strokeRect(20, 15, 140, 40);
    
    ctx.fillStyle = "#00fff2";
    ctx.font = "bold 18px Arial";
    ctx.textAlign = "left";
    ctx.fillText("SCORE: " + score, 35, 41);

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
