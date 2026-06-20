<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider - City Coupe</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #080511;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: monospace;
            color: #fff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        h1 {
            margin: 5px 0;
            font-size: 1.8rem;
            text-transform: uppercase;
            letter-spacing: 5px;
            text-shadow: 0 0 10px #00fff2;
            color: #00fff2;
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 12px;
            overflow: hidden;
            width: 95%;
            max-width: 500px;
            background: #000;
        }
        canvas {
            display: block;
            border: 3px solid #00fff2;
            width: 100%;
            height: auto;
            background: #0d081b;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 12px 25px;
            font-size: 1.1rem;
            font-weight: bold;
            background: #ff0055;
            color: white;
            border: 2px solid #fff;
            border-radius: 6px;
            cursor: pointer;
            box-shadow: 0 0 15px #ff0055;
            z-index: 10;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 95%;
            max-width: 500px;
            margin-top: 10px;
            gap: 10px;
        }
        .steering-group, .speed-group {
            display: flex;
            gap: 10px;
        }
        .btn {
            width: 65px;
            height: 65px;
            background: rgba(0, 255, 242, 0.1);
            border: 2px solid #00fff2;
            border-radius: 15px;
            color: #00fff2;
            font-size: 1.8rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
        }
        .btn-action {
            border-color: #ff0055;
            color: #ff0055;
            background: rgba(255, 0, 85, 0.1);
        }
        .btn:active {
            background: rgba(0, 255, 242, 0.4);
            color: #fff;
        }
        .btn-action:active {
            background: rgba(255, 0, 85, 0.4);
            color: #fff;
        }
    </style>
</head>
<body>

<h1>NEON RIDER</h1>
<div class="container">
    <canvas id="gameCanvas" width="500" height="450"></canvas>
    <button id="restartBtn" onclick="resetGame()">RESTART</button>
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
let baseSpeed = 4;
let currentSpeed = baseSpeed;

// Player Position (Centered in lane)
let carX = 225;
let carY = 340;
const carW = 40;
const carH = 70;

// Obstacle Rival Car
let obsW = 40;
let obsH = 70;
let obsX = 130 + Math.random() * (240 - obsW);
let obsY = -100;

// Simple Procedural Buildings Dataset (Y coordinates, width, height, neon color)
let buildings = [
    { leftSide: true, y: 0, w: 80, h: 120, color: "#ff00bb" },
    { leftSide: true, y: 160, w: 70, h: 100, color: "#bc00ff" },
    { leftSide: true, y: 300, w: 90, h: 140, color: "#00ff66" },
    { leftSide: false, y: 40, w: 75, h: 110, color: "#00fff2" },
    { leftSide: false, y: 200, w: 85, h: 130, color: "#ff0055" },
    { leftSide: false, y: 360, w: 70, h: 90, color: "#00fff2" }
];

let touchLeft = false;
let touchRight = false;
let touchAccel = false;
let touchBrake = false;

// Setup Event Listeners safely
function addEvent(id, startEvt, endEvt, setter) {
    const el = document.getElementById(id);
    el.addEventListener(startEvt, (e) => { e.preventDefault(); setter(true); });
    el.addEventListener(endEvt, (e) => { e.preventDefault(); setter(false); });
}

addEvent("leftBtn", "touchstart", "touchend", (v) => touchLeft = v);
addEvent("rightBtn", "touchstart", "touchend", (v) => touchRight = v);
addEvent("accelBtn", "touchstart", "touchend", (v) => touchAccel = v);
addEvent("brakeBtn", "touchstart", "touchend", (v) => touchBrake = v);

// Desktop mouse fallbacks
addEvent("leftBtn", "mousedown", "mouseup", (v) => touchLeft = v);
addEvent("rightBtn", "mousedown", "mouseup", (v) => touchRight = v);
addEvent("accelBtn", "mousedown", "mouseup", (v) => touchAccel = v);
addEvent("brakeBtn", "mousedown", "mouseup", (v) => touchBrake = v);

function checkCollision() {
    return carX < obsX + obsW && carX + carW > obsX && carY < obsY + obsH && carY + carH > obsY;
}

function resetGame() {
    score = 0;
    gameOver = false;
    carX = 225;
    obsY = -100;
    obsX = 130 + Math.random() * (240 - obsW);
    baseSpeed = 4;
    restartBtn.style.display = "none";
    gameLoop();
}

// Fixed Render Engine Block
function gameLoop() {
    if (gameOver) {
        ctx.fillStyle = "rgba(0,0,0,0.85)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        ctx.fillStyle = "#ff0055";
        ctx.font = "bold 30px monospace";
        ctx.textAlign = "center";
        ctx.fillText("GAME OVER", canvas.width / 2, canvas.height / 2 - 10);
        
        ctx.fillStyle = "#fff";
        ctx.font = "18px monospace";
        ctx.fillText("SCORE: " + score, canvas.width / 2, canvas.height / 2 + 25);
        
        restartBtn.style.display = "block";
        return;
    }

    // Speed modifiers
    if (touchAccel) currentSpeed = baseSpeed * 1.8;
    else if (touchBrake) currentSpeed = baseSpeed * 0.4;
    else currentSpeed = baseSpeed;

    // Side Movement Bounds (Keeping car on the road track width)
    if (touchLeft && carX > 125) carX -= 6;
    if (touchRight && carX < canvas.width - 125 - carW) carX += 6;

    // Movement updates
    roadOffset += currentSpeed;
    if (roadOffset > 40) roadOffset = 0;

    obsY += currentSpeed;
    if (obsY > canvas.height) {
        obsY = -100;
        obsX = 125 + Math.random() * (250 - obsW);
        score++;
        baseSpeed += 0.2;
    }

    // Scroll city buildings loop
    buildings.forEach(b => {
        b.y += currentSpeed * 0.6; // Parallax delay
        if (b.y > canvas.height) b.y = -b.h;
    });

    if (checkCollision()) {
        gameOver = true;
    }

    // --- DRAWING STAGE ---
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Draw Skyscrapers on Sides
    buildings.forEach(b => {
        let drawX = b.leftSide ? 5 : canvas.width - b.w - 5;
        // Dark Building Body
        ctx.fillStyle = "#110e26";
        ctx.fillRect(drawX, b.y, b.w, b.h);
        // Neon Roof Trim
        ctx.fillStyle = b.color;
        ctx.fillRect(drawX, b.y, b.w, 4);
        // Window dots matrix lines
        ctx.fillStyle = "rgba(255, 255, 255, 0.2)";
        for (let wx = drawX + 8; wx < drawX + b.w - 5; wx += 15) {
            for (let wy = b.y + 15; wy < b.y + b.h - 10; wy += 20) {
                ctx.fillRect(wx, wy, 4, 6);
            }
        }
    });

    // Draw Central Main Road Track
    ctx.fillStyle = "#14141f";
    ctx.fillRect(120, 0, canvas.width - 240, canvas.height);

    // Pink / Aqua Dual Neon Guardrails
    ctx.fillStyle = "#ff00ff";
    ctx.fillRect(115, 0, 3, canvas.height);
    ctx.fillRect(canvas.width - 118, 0, 3, canvas.height);
    ctx.fillStyle = "#00fff2";
    ctx.fillRect(118, 0, 2, canvas.height);
    ctx.fillRect(canvas.width - 120, 0, 2, canvas.height);

    // Moving Center Road Lines
    ctx.fillStyle = "rgba(255, 255, 255, 0.5)";
    for (let i = -40; i < canvas.height; i += 50) {
        ctx.fillRect(canvas.width / 2 - 2, i + roadOffset, 4, 25);
    }

    // Draw Player Sports Coupe (Sleek Shape)
    let pColor = "#00fff2";
    ctx.fillStyle = pColor;
    ctx.fillRect(carX, carY, carW, carH); // Base Chassis
    ctx.fillStyle = "#05050a";
    ctx.fillRect(carX + 4, carY + 18, carW - 8, 22); // Windshield/Cabin canopy
    // Rear Tail/Brake light beams
    ctx.fillStyle = touchBrake ? "#ff3366" : "#990022";
    ctx.fillRect(carX + 2, carY + carH - 5, 8, 5);
    ctx.fillRect(carX + carW - 10, carY + carH - 5, 8, 5);

    // Draw Rival Car Coupe
    let oColor = "#ff0055";
    ctx.fillStyle = oColor;
    ctx.fillRect(obsX, obsY, obsW, obsH);
    ctx.fillStyle = "#05050a";
    ctx.fillRect(obsX + 4, obsY + 18, obsW - 8, 22);
    ctx.fillStyle = "#990022";
    ctx.fillRect(obsX + 2, obsY + obsH - 5, 8, 5);
    ctx.fillRect(obsX + obsW - 10, obsY + obsH - 5, 8, 5);

    // Score Overlay Layout Box
    ctx.fillStyle = "rgba(0,0,0,0.6)";
    ctx.fillRect(15, 15, 110, 30);
    ctx.strokeStyle = "#00fff2";
    ctx.strokeRect(15, 15, 110, 30);
    ctx.fillStyle = "#00fff2";
    ctx.font = "bold 13px monospace";
    ctx.textAlign = "left";
    ctx.fillText("SCORE: " + score, 25, 34);

    requestAnimationFrame(gameLoop);
}

// Start game loop safely
gameLoop();
</script>

</body>
</html>
