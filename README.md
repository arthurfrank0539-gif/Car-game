<!DOCTYPE html>
<html>
<head>
    <title>Neon Rider - Arcade Edition</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            background: linear-gradient(135deg, #0b0b16 0%, #161630 50%, #0b0b16 100%);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Courier New', Courier, monospace;
            color: #fff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        h1 {
            margin: 10px 0;
            font-size: 2.2rem;
            text-transform: uppercase;
            letter-spacing: 6px;
            text-shadow: 0 0 10px #00fff2, 0 0 20px #00fff2;
            font-weight: bold;
        }
        .container {
            position: relative;
            box-shadow: 0 0 40px rgba(0, 255, 242, 0.4);
            border-radius: 16px;
            overflow: hidden;
            width: 92%;
            max-width: 699px;
            background: #000;
        }
        canvas {
            display: block;
            border: 4px solid #00fff2;
            width: 100%;
            height: auto;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 55%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 35px;
            font-family: 'Courier New', Courier, monospace;
            font-size: 1.3rem;
            font-weight: bold;
            text-transform: uppercase;
            background: #ff0055;
            color: white;
            border: 3px solid #fff;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 0 20px #ff0055;
            z-index: 10;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 92%;
            max-width: 699px;
            margin-top: 15px;
            margin-bottom: 15px;
            gap: 15px;
        }
        .steering-group, .speed-group {
            display: flex;
            gap: 15px;
        }
        .btn {
            width: 80px;
            height: 80px;
            background: rgba(0, 255, 242, 0.05);
            border: 3px solid #00fff2;
            border-radius: 20px;
            color: #00fff2;
            font-size: 2.2rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 15px rgba(0, 255, 242, 0.2);
            touch-action: none;
        }
        .btn-action {
            border-color: #ff0055;
            color: #ff0055;
            box-shadow: 0 0 15px rgba(255, 0, 85, 0.2);
        }
        .btn:active {
            background: rgba(0, 255, 242, 0.4);
            box-shadow: 0 0 30px #00fff2;
            color: #fff;
        }
        .btn-action:active {
            background: rgba(255, 0, 85, 0.4);
            box-shadow: 0 0 30px #ff0055;
            color: #fff;
        }
    </style>
</head>
<body>

<h1>NEON RIDER</h1>
<div class="container">
    <canvas id="gameCanvas" width="699" height="520"></canvas>
    <button id="restartBtn" onclick="resetGame()">TRY AGAIN</button>
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

let score = 0;
let gameOver = false;
let roadOffset = 0;
let baseSpeed = 5;

// Compact, detailed vehicle profiles
let carX = 325;
let carY = 400;
const carWidth = 46;
const carHeight = 80;
const steerSpeed = 8.5;

let obstacleX = Math.random() * (canvas.width - 46);
let obstacleY = -90;
const obstacleWidth = 46;
const obstacleHeight = 80;
let currentObstacleSpeed = baseSpeed;

let touchLeft = false;
let touchRight = false;
let touchAccel = false;
let touchBrake = false;

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

function checkCollision(r1x, r1y, r1w, r1h, r2x, r2y, r2w, r2h) {
    return r1x < r2x + r2w && r1x + r1w > r2x && r1y < r2y + r2h && r1y + r1h > r2y;
}

function resetGame() {
    score = 0;
    gameOver = false;
    carX = 325;
    carY = 400;
    obstacleX = 50 + Math.random() * (canvas.width - 146);
    obstacleY = -90;
    baseSpeed = 5;
    restartBtn.style.display = "none";
    gameLoop();
}

// Advanced Custom Drawing for Cars
function drawAdvancedCar(x, y, width, height, themeColor, isObstacle) {
    ctx.save();
    
    // Tire Details (Left & Right, Front & Back)
    ctx.fillStyle = "#111116";
    ctx.fillRect(x - 4, y + 10, 4, 16);
    ctx.fillRect(x + width, y + 10, 4, 16);
    ctx.fillRect(x - 4, y + height - 26, 4, 16);
    ctx.fillRect(x + width, y + height - 26, 4, 16);
    
    // Alloy Rim Accents
    ctx.fillStyle = themeColor;
    ctx.fillRect(x - 3, y + 16, 2, 4);
    ctx.fillRect(x + width + 1, y + 16, 2, 4);
    ctx.fillRect(x - 3, y + height - 20, 2, 4);
    ctx.fillRect(x + width + 1, y + height - 20, 2, 4);

    // Main Structural Aero-Chassis
    ctx.fillStyle = themeColor;
    ctx.shadowColor = themeColor;
    ctx.shadowBlur = 14;
    ctx.fillRect(x, y, width, height);
    ctx.shadowBlur = 0;

    // Body Lines and Racing Trim Panels
    ctx.fillStyle = "rgba(0,0,0,0.25)";
    ctx.fillRect(x + 4, y + 4, width - 8, height - 8);
    ctx.fillStyle = themeColor;
    ctx.fillRect(x + 7, y + 6, width - 14, height - 12);

    if (!isObstacle) {
        // Player Cockpit windshield & Engine Vent
        ctx.fillStyle = "#04040a";
        ctx.fillRect(x + 8, y + 22, width - 16, 20); // Window
        ctx.fillStyle = "rgba(255,255,255,0.4)";
        ctx.fillRect(x + 12, y + 22, 4, 20);        // Windshield Glare
        
        ctx.fillStyle = "#111";
        ctx.fillRect(x + 12, y + 52, width - 24, 14); // Spoiler Trim
        
        // Headlight Beams (Forward Projected Light)
        ctx.save();
        let beamGrad = ctx.createLinearGradient(0, y, 0, y - 80);
        beamGrad.addColorStop(0, "rgba(255, 255, 255, 0.4)");
        beamGrad.addColorStop(1, "rgba(255, 255, 255, 0)");
        ctx.fillStyle = beamGrad;
        
        ctx.beginPath();
        ctx.moveTo(x + 4, y);
        ctx.lineTo(x - 15, y - 80);
        ctx.lineTo(x + 20, y - 80);
        ctx.fill();
        
        ctx.beginPath();
        ctx.moveTo(x + width - 4, y);
        ctx.lineTo(x + width - 20, y - 80);
        ctx.lineTo(x + width + 15, y - 80);
        ctx.fill();
        ctx.restore();
    } else {
        // Obstacle Cockpit Rear Windshield Orientation
        ctx.fillStyle = "#04040a";
        ctx.fillRect(x + 8, y + 38, width - 16, 20);
        ctx.fillStyle = "rgba(255,255,255,0.25)";
        ctx.fillRect(x + width - 16, y + 38, 3, 20);
        
        // Brake Lights Glow
        ctx.fillStyle = "#ff0000";
        ctx.fillRect(x + 3, y + height - 4, 8, 4);
        ctx.fillRect(x + width - 11, y + height - 4, 8, 4);
    }
    ctx.restore();
}

function gameLoop() {
    if (gameOver) {
        ctx.fillStyle = "rgba(6, 6, 14, 0.9)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#ff0055";
        ctx.font = "bold 44px 'Courier New'";
        ctx.textAlign = "center";
        ctx.shadowColor = "#ff0055";
        ctx.shadowBlur = 20;
        ctx.fillText("CRASH DETECTED", canvas.width / 2, canvas.height / 2 - 30);

        ctx.fillStyle = "#fff";
        ctx.font = "bold 20px 'Courier New'";
        ctx.shadowBlur = 0;
        ctx.fillText("FINAL SCORE: " + score, canvas.width / 2, canvas.height / 2 + 25);
        
        restartBtn.style.display = "block";
        return;
    }

    if (touchLeft && carX > 55) carX -= steerSpeed;
    if (touchRight && carX < canvas.width - carWidth - 55) carX += steerSpeed;

    if (touchAccel) {
        currentObstacleSpeed = baseSpeed * 1.8;
    } else if (touchBrake) {
        currentObstacleSpeed = baseSpeed * 0.4;
    } else {
        currentObstacleSpeed = baseSpeed;
    }

    roadOffset += currentObstacleSpeed;
    if (roadOffset > 40) roadOffset = 0;

    obstacleY += currentObstacleSpeed;

    if (obstacleY > canvas.height) {
        obstacleY = -90;
        obstacleX = 55 + Math.random() * (canvas.width - obstacleWidth - 110);
        score += 1;
        baseSpeed += 0.25;
    }

    if (checkCollision(carX, carY, carWidth, carHeight, obstacleX, obstacleY, obstacleWidth, obstacleHeight)) {
        gameOver = true;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Dynamic Sci-Fi Track Grid lines (Background environment)
    ctx.strokeStyle = "#13132a";
    ctx.lineWidth = 2;
    for (let xPos = 0; xPos < canvas.width; xPos += 50) {
        ctx.beginPath();
        ctx.moveTo(xPos, 0);
        ctx.lineTo(xPos, canvas.height);
        ctx.stroke();
    }

    // Asphalt Mainway Base
    ctx.fillStyle = "#16161f";
    ctx.fillRect(50, 0, canvas.width - 100, canvas.height);

    // Neon Track Boundaries (Glowing outer rails)
    ctx.shadowColor = "#00ff66";
    ctx.shadowBlur = 8;
    ctx.fillStyle = "#00ff66";
    ctx.fillRect(45, 0, 5, canvas.height);
    ctx.fillRect(canvas.width - 50, 0, 5, canvas.height);
    ctx.shadowBlur = 0;

    // Moving Dashed Track Center Divider Lanes
    ctx.fillStyle = "rgba(255, 255, 255, 0.75)";
    for (let i = -40; i < canvas.height; i += 50) {
        ctx.fillRect(canvas.width / 2 - 3, i + roadOffset, 6, 25);
    }

    // Draw Player High-Tech Neon Car
    drawAdvancedCar(carX, carY, carWidth, carHeight, "#00fff2", false);

    // Draw Rival/Obstacle High-Tech Neon Car
    drawAdvancedCar(obstacleX, obstacleY, obstacleWidth, obstacleHeight, "#ff0055", true);

    // Sleek Modular HUD Interface Layout
    ctx.fillStyle = "rgba(0, 0, 0, 0.75)";
    ctx.fillRect(25, 20, 160, 45);
    ctx.strokeStyle = "#00fff2";
    ctx.lineWidth = 2;
    ctx.strokeRect(25, 20, 160, 45);
    
    ctx.fillStyle = "#00fff2";
    ctx.font = "bold 16px 'Courier New'";
    ctx.textAlign = "left";
    ctx.fillText("SCORE: " + score, 40, 48);

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
