<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider - Premium Cityscape</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            padding: 0;
            background: radial-gradient(circle at center, #0e101f 0%, #05060b 100%);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            color: #ffffff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        h1 {
            margin: 10px 0;
            font-size: 1.5rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 5px;
            color: #ffffff;
            opacity: 0.95;
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 24px;
            overflow: hidden;
            width: 92%;
            max-width: 480px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
            background: #090a14;
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
            box-shadow: 0 10px 25px rgba(255,255,255,0.3);
            z-index: 10;
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
            width: 60px;
            height: 60px;
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 50%;
            color: #ffffff;
            font-size: 1.3rem;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
            transition: background 0.1s;
        }
        .btn:active {
            background: rgba(255, 255, 255, 0.2);
        }
        .btn-action {
            color: #ff453a;
            border-color: rgba(255, 69, 58, 0.2);
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
    const carW = 36;
    const carH = 68;

    let obsW = 36;
    let obsH = 68;
    let obsX = 140 + Math.random() * (220 - obsW);
    let obsY = -100;

    let coinX = 140 + Math.random() * (220 - 20);
    let coinY = -300;
    const coinSize = 18;

    // Repositioned architecture lists brought perfectly inward for frame rendering visibility
    let buildings = [
        { leftSide: true, xOffset: 5, y: 0, w: 90, h: 220, accentColor: "#00fff2" },
        { leftSide: true, xOffset: 15, y: 260, w: 75, h: 160, accentColor: "#ff00bb" },
        { leftSide: false, xOffset: 5, y: 60, w: 90, h: 240, accentColor: "#bc00ff" },
        { leftSide: false, xOffset: 20, y: 320, w: 70, h: 180, accentColor: "#00ff66" }
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
        obsX = 140 + Math.random() * (220 - obsW);
        coinY = -300;
        coinX = 140 + Math.random() * (220 - coinSize);
        baseSpeed = 4;
        restartBtn.style.display = "none";
        gameLoop();
    };

    function gameLoop() {
        if (gameOver) {
            ctx.fillStyle = "rgba(10, 11, 21, 0.9)";
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

        if (touchLeft && carX > 140) carX -= 5.5;
        if (touchRight && carX < canvas.width - 140 - carW) carX += 5.5;

        roadOffset += currentSpeed;
        if (roadOffset > 60) roadOffset = 0;

        obsY += currentSpeed;
        if (obsY > canvas.height) {
            obsY = -100;
            obsX = 140 + Math.random() * (220 - obsW);
            baseSpeed += 0.15;
        }

        coinY += currentSpeed;
        if (coinY > canvas.height) {
            coinY = -150 - Math.random() * 250;
            coinX = 145 + Math.random() * (210 - coinSize);
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
            coinX = 145 + Math.random() * (210 - coinSize);
        }

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // --- DRAW VISIBLE LAYERED BUILDINGS ---
        buildings.forEach(b => {
            let drawX = b.leftSide ? b.xOffset : canvas.width - b.w - b.xOffset;
            
            // Solid structure base
            ctx.fillStyle = "#141625";
            ctx.fillRect(drawX, b.y, b.w, b.h);

            // Architectural structural window slits
            ctx.fillStyle = "rgba(255, 255, 255, 0.12)";
            for (let wx = drawX + 10; wx < drawX + b.w - 10; wx += 16) {
                for (let wy = b.y + 15; wy < b.y + b.h - 15; wy += 24) {
                    if ((Math.floor(wx + wy)) % 5 !== 0) {
                        ctx.fillStyle = "rgba(255, 225, 100, 0.25)";
                    } else {
                        ctx.fillStyle = "rgba(255, 255, 255, 0.06)";
                    }
                    ctx.fillRect(wx, wy, 6, 12);
                }
            }

            // Top Roof Trim Accent
            ctx.fillStyle = b.accentColor;
            ctx.fillRect(drawX, b.y, b.w, 4);

            // Roof Spire/Antenna
            ctx.strokeStyle = "rgba(255, 255, 255, 0.3)";
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.moveTo(drawX + b.w/2, b.y);
            ctx.lineTo(drawX + b.w/2, b.y - 14);
            ctx.stroke();

            // Flashing Warning Dot atop Spire
            ctx.fillStyle = (Math.floor(Date.now() / 300) % 2 === 0) ? "#ff3b30" : "rgba(0,0,0,0)";
            ctx.beginPath();
            ctx.arc(drawX + b.w/2, b.y - 14, 3, 0, Math.PI * 2);
            ctx.fill();
        });

        // --- HIGHWAY SCENERY LINE CORRIDORS ---
        let roadGrad = ctx.createLinearGradient(135, 0, canvas.width - 135, 0);
        roadGrad.addColorStop(0, '#0d0e17');
        roadGrad.addColorStop(0.5, '#151726');
        roadGrad.addColorStop(1, '#0d0e17');
        ctx.fillStyle = roadGrad;
        ctx.fillRect(135, 0, canvas.width - 270, canvas.height);

        // Solid Road Borders
        ctx.fillStyle = "rgba(255, 255, 255, 0.08)";
        ctx.fillRect(135, 0, 3, canvas.height);
        ctx.fillRect(canvas.width - 138, 0, 3, canvas.height);

        // Road dashed striping lanes
        ctx.fillStyle = "rgba(255, 255, 255, 0.2)";
        for (let i = -60; i < canvas.height; i += 60) {
            ctx.fillRect(canvas.width / 2 - 1.5, i + roadOffset, 3, 30);
        }

        // --- GLOWING ENERGY COIN ---
        ctx.save();
        ctx.shadowColor = "#ffcc00";
        ctx.shadowBlur = 10;
        ctx.fillStyle = "#ffcc00";
        ctx.beginPath();
        ctx.arc(coinX + coinSize/2, coinY + coinSize/2, coinSize/2, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();

        // --- ACCURATE PATH-SHAPED SPORTS CAR ---
        ctx.save();
        // Thruster exhaust trail
        if (touchAccel) {
            ctx.fillStyle = "rgba(0, 150, 255, 0.4)";
            ctx.fillRect(carX + 5, carY + carH, 4, 15);
            ctx.fillRect(carX + carW - 9, carY + carH, 4, 15);
        }

        // Aerodynamic Body Shell
        let carGrad = ctx.createLinearGradient(carX, carY, carX + carW, carY);
        carGrad.addColorStop(0, '#2f80ed');
        carGrad.addColorStop(1, '#00c6ff');
        ctx.fillStyle = carGrad;
        
        ctx.beginPath();
        ctx.moveTo(carX + 8, carY); 
        ctx.lineTo(carX + carW - 8, carY);
        ctx.lineTo(carX + carW, carY + 14); 
        ctx.lineTo(carX + carW - 1, carY + carH - 6); 
        ctx.lineTo(carX + carW - 4, carY + carH);
        ctx.lineTo(carX + 4, carY + carH);
        ctx.lineTo(carX + 1, carY + carH - 6);
        ctx.lineTo(carX, carY + 14);
        ctx.closePath();
        ctx.fill();

        // Windshield Glass Window Overlay
        ctx.fillStyle = "#090a12";
        ctx.beginPath();
        ctx.moveTo(carX + 8, carY + 18);
        ctx.lineTo(carX + carW - 8, carY + 18);
        ctx.lineTo(carX + carW - 5, carY + 40);
        ctx.lineTo(carX + 5, carY + 40);
        ctx.closePath();
        ctx.fill();

        // Bright Front Xenon Headlight LEDs
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(carX + 5, carY + 1, 4, 2);
        ctx.fillRect(carX + carW - 9, carY + 1, 4, 2);

        // Continuous Tail Light Strip
        ctx.fillStyle = touchBrake ? "#ff3b30" : "#b0121a";
        ctx.fillRect(carX + 4, carY + carH - 3, carW - 8, 2);
        ctx.restore();

        // --- RIVAL LUXURY SEDAN ---
        ctx.save();
        ctx.fillStyle = "#26293c";
        ctx.beginPath();
        ctx.roundRect(obsX, obsY, obsW, obsH, [6, 6, 4, 4]);
        ctx.fill();

        // Glass cabin tint
        ctx.fillStyle = "#0b0c14";
        ctx.beginPath();
        ctx.roundRect(obsX + 4, obsY + 22, obsW - 8, 22, 3);
        ctx.fill();

        // Red Tail Markers
        ctx.fillStyle = "#ff2d55";
        ctx.fillRect(obsX + 3, obsY + obsH - 3, 5, 2);
        ctx.fillRect(obsX + obsW - 8, obsY + obsH - 3, 5, 2);
        ctx.restore();

        // --- COHESIVE GLASS HUDBAR OVERLAY ---
        ctx.save();
        ctx.fillStyle = "rgba(255, 255, 255, 0.07)";
        ctx.beginPath();
        ctx.roundRect(20, 20, 110, 36, 12);
        ctx.fill();
        ctx.strokeStyle = "rgba(255, 255, 255, 0.12)";
        ctx.lineWidth = 1;
        ctx.stroke();
        
        ctx.fillStyle = "#ffffff";
        ctx.font = "600 13px -apple-system, sans-serif";
        ctx.textAlign = "center";
        ctx.fillText("SCORE  " + score, 75, 42);
        ctx.restore();

        requestAnimationFrame(gameLoop);
    }

    gameLoop();
});
</script>

</body>
</html>
