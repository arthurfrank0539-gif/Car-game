<!DOCTYPE html>
<html>
<head>
    <title>Neon Rider - Hyper Edition</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            background: radial-gradient(circle at center, #1b113a 0%, #080511 100%);
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
            margin: 8px 0;
            font-size: 2.2rem;
            text-transform: uppercase;
            letter-spacing: 8px;
            text-shadow: 0 0 10px #00fff2, 0 0 20px #00fff2;
            font-weight: bold;
        }
        .container {
            position: relative;
            box-shadow: 0 0 45px rgba(0, 255, 242, 0.5);
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
    <button id="restartBtn" onclick="resetGame()">REIGNITE ENGINE</button>
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

// Player Setup
let carX = 325;
let carY = 400;
const carWidth = 46;
const carHeight = 80;
const steerSpeed = 8.5;

// Obstacle Setup
let obstacleX = Math.random() * (canvas.width - 46);
let obstacleY = -90;
const obstacleWidth = 46;
const obstacleHeight = 80;
let currentObstacleSpeed = baseSpeed;

// Environment Details: Generating Streetlights alongside the road margins
let streetlights = [
    { y: 0 }, { y: 130 }, { y: 260 }, { y: 390 }, { y: 520 }
];

// Touch State Monitoring
let touchLeft = false;
let touchRight = false;
let touchAccel = false;
let touchBrake = false;

const leftBtn = document.getElementById("leftBtn");
const rightBtn = document.getElementById("rightBtn");
const accelBtn = document.getElementById("accelBtn");
const brakeBtn = document.getElementById("brakeBtn");

leftBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchLeft = true; });
leftBtn.addEventListener("touchend", () => touchLeft = false);
rightBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchRight = true; });
rightBtn.addEventListener("touchend", () => touchRight = false);
accelBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchAccel = true; });
accelBtn.addEventListener("touchend", () => touchAccel = false);
brakeBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchBrake = true; });
brakeBtn.addEventListener("touchend", () => touchBrake = false);

function checkCollision(r1x, r1y, r1w, r1h, r2x, r2y, r2w, r2h) {
    return r1x < r2x + r2w && r1x + r1w > r2x && r1y < r2y + r2h && r1y + r1h > r2y;
}

function resetGame() {
    score = 0;
    gameOver = false;
    carX = 325;
    carY = 400;
    obstacleX = 90 + Math.random() * (canvas.width - 180 - obstacleWidth);
    obstacleY = -90;
    baseSpeed = 5;
    restartBtn.style.display = "none";
    gameLoop();
}

function drawAdvancedCar(x, y, width, height, themeColor, isObstacle) {
    ctx.save();
    
    // Draw Dark Heavy Performance Wheels
    ctx.fillStyle = "#0d0d11";
    ctx.fillRect(x - 5, y + 12, 5, 16);
    ctx.fillRect(x + width, y + 12, 5, 16);
    ctx.fillRect(x - 5, y + height - 28, 5, 16);
    ctx.fillRect(x + width, y + height - 28, 5, 16);
    
    // Wheel Rim Highlights
    ctx.fillStyle = themeColor;
    ctx.fillRect(x - 4, y + 18, 2, 4);
    ctx.fillRect(x + width + 2, y + 18, 2, 4);

    // Hyper-Chassis Base Layer
    ctx.fillStyle = themeColor;
    ctx.shadowColor = themeColor;
    ctx.shadowBlur = 16;
    ctx.fillRect(x, y, width, height);
    ctx.shadowBlur = 0;

    // Body Panel Contrast Lines
    ctx.fillStyle = "rgba(10, 6, 24, 0.4)";
    ctx.fillRect(x + 4, y + 4, width - 8, height - 8);
    ctx.fillStyle = themeColor;
    ctx.fillRect(x + 8, y + 6, width - 16, height - 12);

    if (!isObstacle) {
        // Player windshield design
        ctx.fillStyle = "#020205";
        ctx.fillRect(x + 8, y + 20, width - 16, 22); 
        ctx.fillStyle = "rgba(0, 255, 242, 0.5)";
        ctx.fillRect(x + 12, y + 20, 5, 22); // Solar reflection sheen
        
        // Aerodynamic Rear Wing
        ctx.fillStyle = "#08080f";
        ctx.fillRect(x + 4, y + height - 10, width - 8, 6);

        // Dynamic Thruster Exhaust Flame (Triggers on 'A' Acceleration)
        if (touchAccel) {
            ctx.shadowColor = "#ffaa00";
            ctx.shadowBlur = 20;
            let flameGrad = ctx.createLinearGradient(0, y + height, 0, y + height + 25);
            flameGrad.addColorStop(0, "#ffffff");
            flameGrad.addColorStop(0.3, "#00ffff");
            flameGrad.addColorStop(1, "rgba(0, 170, 255, 0)");
            ctx.fillStyle = flameGrad;
            ctx.fillRect(x + 10, y + height, 8, 25);
            ctx.fillRect(x + width - 18, y + height, 8, 25);
        }

        // Active Rear Brake Light Illumination (Triggers on 'B' Braking)
        ctx.shadowColor = "#ff0044";
        ctx.shadowBlur = touchBrake ? 25 : 4;
        ctx.fillStyle = touchBrake ? "#ff3333" : "#aa0022";
        ctx.fillRect(x + 2, y + height - 4, 8, 4);
        ctx.fillRect(x + width - 10, y + height - 4, 8, 4);

        // Headlight Projection Cones
        ctx.save();
        let beamGrad = ctx.createLinearGradient(0, y, 0, y - 110);
        beamGrad.addColorStop(0, "rgba(0, 255, 242, 0.4)");
        beamGrad.addColorStop(1, "rgba(0, 255, 242, 0)");
        ctx.fillStyle = beamGrad;
        
        ctx.beginPath(); ctx.moveTo(x + 4, y); ctx.lineTo(x - 25, y - 110); ctx.lineTo(x + 25, y - 110); ctx.fill();
        ctx.beginPath(); ctx.moveTo(x + width - 4, y); ctx.lineTo(x + width - 25, y - 110); ctx.lineTo(x + width + 25, y - 110); ctx.fill();
        ctx.restore();
    } else {
        // Enemy windshield
        ctx.fillStyle = "#020205";
        ctx.fillRect(x + 8, y + 38, width - 16, 22);
        ctx.fillStyle = "rgba(255, 0, 85, 0.3)";
        ctx.fillRect(x + width - 14, y + 38, 4, 22);

        // Rear Brake Light indicator strip
        ctx.fillStyle = "#ff0044";
        ctx.fillRect(x + 3, y + height - 5, 6, 4);
        ctx.fillRect(x + width - 9, y + height - 5, 6, 4);
    }
    ctx.restore();
}

function gameLoop() {
    if (gameOver) {
        ctx.fillStyle = "rgba(5, 3, 10, 0.92)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#ff0055";
        ctx.font = "bold 44px 'Courier New'";
        ctx.textAlign = "center";
        ctx.shadowColor = "#ff0055";
        ctx.shadowBlur = 25;
        ctx.fillText("SYSTEM CRASHED", canvas.width / 2, canvas.height / 2 - 30);

        ctx.fillStyle = "#fff";
        ctx.font = "bold 22px 'Courier New'";
        ctx.shadowBlur = 0;
        ctx.fillText("TOTAL DATA DODGED: " + score, canvas.width / 2, canvas.height / 2 + 25);
        
        restartBtn.style.display = "block";
        return;
    }

    // Controls
    if (touchLeft && carX > 85) carX -= steerSpeed;
    if (touchRight && carX < canvas.width - carWidth - 85) carX += steerSpeed;

    if (touchAccel) {
        currentObstacleSpeed = baseSpeed * 1.9;
    } else if (touchBrake) {
        currentObstacleSpeed = baseSpeed * 0.4;
    } else {
        currentObstacleSpeed = baseSpeed;
    }

    // Scroll Game Track Environment
    roadOffset += currentObstacleSpeed;
    if (roadOffset > 50) roadOffset = 0;

    obstacleY += currentObstacleSpeed;

    // Reset obstacle car on successful pass
    if (obstacleY > canvas.height) {
        obstacleY = -90;
        obstacleX = 85 + Math.random() * (canvas.width - obstacleWidth - 170);
        score += 1;
        baseSpeed += 0.3;
    }

    // Animate and wrap the roadside streetlights
    streetlights.forEach(light => {
        light.y += currentObstacleSpeed;
        if (light.y > canvas.height) light.y = -20;
    });

    if (checkCollision(carX, carY, carWidth, carHeight, obstacleX, obstacleY, obstacleWidth, obstacleHeight)) {
        gameOver = true;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 1. ENVIRONMENT: Outer Cyber-Grid Landscape Background
    ctx.strokeStyle = "#100c26";
    ctx.lineWidth = 1.5;
    for (let xPos = 0; xPos < canvas.width; xPos += 40) {
        ctx.beginPath(); ctx.moveTo(xPos, 0); ctx.lineTo(xPos, canvas.height); ctx.stroke();
    }

    // 2. ENVIRONMENT: High-Performance Dark Asphalt Track Mainway
    ctx.fillStyle = "#12121a";
    ctx.fillRect(80, 0, canvas.width - 160, canvas.height);

    // 3. ENVIRONMENT: Glowing Multi-layered Guardrails (Outer boundary lines)
    ctx.shadowColor = "#ff00ff";
    ctx.shadowBlur = 10;
    ctx.fillStyle = "#ff00ff"; // Outer neon pink guard strip
    ctx.fillRect(72, 0, 4, canvas.height);
    ctx.fillRect(canvas.width - 76, 0, 4, canvas.height);

    ctx.shadowColor = "#00ff66";
    ctx.shadowBlur = 10;
    ctx.fillStyle = "#00ff66"; // Inner electric green line
    ctx.fillRect(76, 0, 4, canvas.height);
    ctx.fillRect(canvas.width - 80, 0, 4, canvas.height);
    ctx.shadowBlur = 0;

    // 4. ENVIRONMENT: Pavement Center Line Markings
    ctx.fillStyle = "rgba(255, 255, 255, 0.65)";
    for (let i = -50; i < canvas.height; i += 60) {
        ctx.fillRect(canvas.width / 2 - 2, i + roadOffset, 4, 30);
    }

    // 5. ENVIRONMENT: Scrolling Light Beacons / Cyber Trees along margins
    streetlights.forEach(light => {
        ctx.save();
        ctx.shadowBlur = 15;
        // Left light (Pink/Blue aura)
        ctx.shadowColor = "#ff00bb";
        ctx.fillStyle = "#fff";
        ctx.beginPath(); ctx.arc(45, light.y, 6, 0, Math.PI * 2); ctx.fill();
        // Right light
        ctx.shadowColor = "#00fff2";
        ctx.beginPath(); ctx.arc(canvas.width - 45, light.y, 6, 0, Math.PI * 2); ctx.fill();
        ctx.restore();
    });

    // 6. CARS: Render Upgraded Vehicles with details
    drawAdvancedCar(carX, carY, carWidth, carHeight, "#00fff2", false);
    drawAdvancedCar(obstacleX, obstacleY, obstacleWidth, obstacleHeight, "#ff0055", true);

    // 7. INTERFACE: Premium Arcade HUD Scoring Box
    ctx.fillStyle = "rgba(4, 2, 10, 0.85)";
    ctx.fillRect(25, 20, 160, 45);
    ctx.strokeStyle = "#00fff2";
    ctx.lineWidth = 2;
    ctx.strokeRect(25, 20, 160, 45);
    
    ctx.fillStyle = "#00fff2";
    ctx.font = "bold 15px 'Courier New'";
    ctx.textAlign = "left";
    ctx.fillText("SCORE: " + score, 42, 48);

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
