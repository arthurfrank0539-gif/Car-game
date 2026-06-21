<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider - Modern City Edition</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            padding: 0;
            background: radial-gradient(circle at center, #111222 0%, #060711 100%);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            color: #ffffff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        h1 {
            margin: 10px 0;
            font-size: 1.6rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 4px;
            color: #ffffff;
            opacity: 0.9;
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 24px;
            overflow: hidden;
            width: 92%;
            max-width: 480px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.4);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
            background: #0d0f1d;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 32px;
            font-size: 1rem;
            font-weight: 600;
            letter-spacing: 1px;
            background: #ffffff;
            color: #000000;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 10px 25px rgba(255,255,255,0.25);
            z-index: 10;
            transition: transform 0.2s;
        }
        #restartBtn:active {
            transform: translate(-50%, -50%) scale(0.95);
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 92%;
            max-width: 480px;
            margin-top: 20px;
            gap: 20px;
        }
        .steering-group, .speed-group {
            display: flex;
            gap: 12px;
        }
        .btn {
            width: 64px;
            height: 64px;
            background: rgba(255, 255, 255, 0.04);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.12);
            border-radius: 50%;
            color: #ffffff;
            font-size: 1.4rem;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
            transition: background 0.1s, transform 0.1s;
        }
        .btn:active {
            background: rgba(255, 255, 255, 0.18);
            transform: scale(0.95);
        }
        .btn-action {
            color: #ff3b30;
            border-color: rgba(255, 59, 48, 0.2);
        }
        .btn-action:active {
            background: rgba(255, 59, 48, 0.15);
        }
    </style>
</head>
<body>

<h1>NEON RIDER</h1>
<div class="container">
    <canvas id="gameCanvas" width="500" height="450"></canvas>
    <button id="restartBtn" onclick="resetGame()">Try Again</button>
</div>

<div class="controls-pad">
    <div class="steering-group">
        <div class="btn" id="leftBtn">←</div>
        <div class="btn" id="rightBtn">→</div>
    </div>
    <div class="speed-group">
        <div class="btn btn-action" id="brakeBtn">↓</div>
        <div class="btn btn-action" id="accelBtn">↑</div>
    </div>
</div>

<script>
window.addEventListener('load', function() {
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const restartBtn = document.getElementById("restartBtn");

    let score = 0;
    let gameOver = false;
    let roadOffset = 0;
    let baseSpeed = 4;
    let currentSpeed = baseSpeed;

    let carX = 230;
    let carY = 340;
    const carW = 38;
    const carH = 72;

    let obsW = 38;
    let obsH = 72;
    let obsX = 130 + Math.random() * (240 - obsW);
    let obsY = -100;

    let coinX = 130 + Math.random() * (240 - 24);
    let coinY = -300;
    const coinSize = 20;

    // Upgraded Modern Buildings Array Configurations
    let buildings = [
        { leftSide: true, y: 0, w: 75, h: 180, color: "rgba(0, 255, 242, 0.15)", glowColor: "#00fff2" },
        { leftSide: true, y: 240, w: 85, h: 160, color: "rgba(255, 0, 187, 0.12)", glowColor: "#ff00bb" },
        { leftSide: false, y: 40, w: 80, h: 200, color: "rgba(188, 0, 255, 0.12)", glowColor: "#bc00ff" },
        { leftSide: false, y: 280, w: 75, h: 150, color: "rgba(0, 255, 102, 0.12)", glowColor: "#00ff66" }
    ];

    let touchLeft = false;
    let touchRight = false;
    let touchAccel = false;
    let touchBrake = false;

    function addEvent(id, startEvt, endEvt, setter) {
        const el = document.getElementById(id);
        if (!el) return;
        el.addEventListener(startEvt, (e) => { e.preventDefault(); setter(true); });
        el.addEventListener(endEvt, (e) => { e.preventDefault(); setter(false); });
    }

    addEvent("leftBtn", "touchstart", "touchend", (v) => touchLeft = v);
    addEvent("rightBtn", "touchstart", "touchend", (v) => touchRight = v);
    addEvent("accelBtn", "touchstart", "touchend", (v) => touchAccel = v);
    addEvent("brakeBtn", "touchstart", "touchend", (v) => touchBrake = v);

    addEvent("leftBtn", "mousedown", "mouseup", (v) => touchLeft = v);
    addEvent("rightBtn", "mousedown", "mouseup", (v) => touchRight = v);
    addEvent("accelBtn", "mousedown", "mouseup", (v) => touchAccel = v);
    addEvent("brakeBtn", "mousedown", "mouseup", (v) => touchBrake = v);

    window.resetGame = function() {
        score = 0;
        gameOver = false;
        carX = 230;
        obsY = -100;
        obsX = 130 + Math.random() * (240 - obsW);
        coinY = -300;
        coinX = 130 + Math.random() * (240 - coinSize);
        baseSpeed = 4;
        restartBtn.style.display = "none";
        gameLoop();
    };

    function gameLoop() {
        if (gameOver) {
            ctx.fillStyle = "rgba(10, 11, 21, 0.85)";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = "#ffffff";
            ctx.font = "600 24px -apple-system, sans-serif";
            ctx.textAlign = "center";
            ctx.fillText("Game Over", canvas.width / 2, canvas.height / 2 - 20);
            
            ctx.fillStyle = "rgba(255,255,255,0.6)";
            ctx.font = "15px -apple-system, sans-serif";
            ctx.fillText("Score: " + score, canvas.width / 2, canvas.height / 2 + 15);
            
            restartBtn.style.display = "block";
            return;
        }

        if (touchAccel) currentSpeed = baseSpeed * 1.8;
        else if (touchBrake) currentSpeed = baseSpeed * 0.4;
        else currentSpeed = baseSpeed;

        if (touchLeft && carX > 130) carX -= 6;
        if (touchRight && carX < canvas.width - 130 - carW) carX += 6;

        roadOffset += currentSpeed;
        if (roadOffset > 60) roadOffset = 0;

        obsY += currentSpeed;
        if (obsY > canvas.height) {
            obsY = -100;
            obsX = 130 + Math.random() * (240 - obsW);
            baseSpeed += 0.15;
        }

        coinY += currentSpeed;
        if (coinY > canvas.height) {
            coinY = -150 - Math.random() * 250;
            coinX = 135 + Math.random() * (230 - coinSize);
        }

        buildings.forEach(b => {
            b.y += currentSpeed * 0.4;
            if (b.y > canvas.height) b.y = -b.h;
        });

        if (carX < obsX + obsW && carX + carW > obsX && carY < obsY + obsH && carY + carH > obsY) {
            gameOver = true;
        }

        if (carX < coinX + coinSize && carX + carW > coinX && carY < coinY + coinSize && carY + carH > coinY) {
            score++;
            coinY = -150 - Math.random() * 250; 
            coinX = 135 + Math.random() * (230 - coinSize);
        }

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // --- DRAW DETAILED MODERN BUILDINGS WITH LIGHTS ---
        buildings.forEach(b => {
            let drawX = b.leftSide ? 10 : canvas.width - b.w - 10;
            
            // Core Skyscraper Block
            ctx.fillStyle = "rgba(22, 25, 46, 0.4)";
            ctx.fillRect(drawX, b.y, b.w, b.h);
            
            // Elegant Architectural Top Roof Accent Lines
            ctx.fillStyle = b.color;
            ctx.fillRect(drawX, b.y, b.w, 3);
            
            // Antennas atop buildings
            ctx.strokeStyle = "rgba(255, 255, 255, 0.2)";
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(drawX + b.w / 2, b.y);
            ctx.lineTo(drawX + b.w / 2, b.y - 12);
            ctx.stroke();
