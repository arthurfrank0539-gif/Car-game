<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider - Cyberpunk Arcade</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #05030a;
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
            margin: 5px 0;
            font-size: 2rem;
            text-transform: uppercase;
            letter-spacing: 6px;
            text-shadow: 0 0 10px #00fff2, 0 0 20px #00fff2;
            color: #00fff2;
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 16px;
            overflow: hidden;
            width: 95%;
            max-width: 500px;
            box-shadow: 0 0 30px rgba(0, 255, 242, 0.2);
        }
        canvas {
            display: block;
            border: 3px solid #00fff2;
            width: 100%;
            height: auto;
            background: #090614;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 30px;
            font-size: 1.2rem;
            font-weight: bold;
            letter-spacing: 2px;
            background: #ff0055;
            color: white;
            border: 2px solid #fff;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 0 20px #ff0055;
            z-index: 10;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 95%;
            max-width: 500px;
            margin-top: 15px;
            gap: 15px;
        }
        .steering-group, .speed-group {
            display: flex;
            gap: 12px;
        }
        .btn {
            width: 65px;
            height: 65px;
            background: rgba(0, 255, 242, 0.05);
            border: 2px solid #00fff2;
            border-radius: 18px;
            color: #00fff2;
            font-size: 1.8rem;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: inset 0 0 10px rgba(0, 255, 242, 0.1);
            transition: all 0.1s ease;
        }
        .btn-action {
            border-color: #ff0055;
            color: #ff0055;
            background: rgba(255, 0, 85, 0.05);
            box-shadow: inset 0 0 10px rgba(255, 0, 85, 0.1);
        }
        .btn:active {
            background: rgba(0, 255, 242, 0.3);
            box-shadow: 0 0 15px #00fff2;
            color: #fff;
        }
        .btn-action:active {
            background: rgba(255, 0, 85, 0.3);
            box-shadow: 0 0 15px #ff0055;
            color: #fff;
        }
    </style>
</head>
<body>

<h1>NEON RIDER</h1>
<div class="container">
    <canvas id="gameCanvas" width="500" height="450"></canvas>
    <button id="restartBtn" onclick="resetGame()">RESPAWN</button>
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
    const carW = 40;
    const carH = 70;

    let obsW = 42;
    let obsH = 72;
    let obsX = 130 + Math.random() * (240 - obsW);
    let obsY = -100;

    let coinX = 130 + Math.random() * (240 - 22);
    let coinY = -300;
    const coinSize = 22;

    let buildings = [
        { leftSide: true, y: 0, w: 80, h: 140, color: "#ff00bb" },
        { leftSide: true, y: 180, w: 65, h: 100, color: "#bc00ff" },
        { leftSide: true, y: 320, w: 85, h: 160, color: "#00ff66" },
        { leftSide: false, y: 30, w: 75, h: 120, color: "#00fff2" },
        { leftSide: false, y: 190, w: 85, h: 150, color: "#ff0055" },
        { leftSide: false, y: 370, w: 70, h: 110, color: "#00fff2" }
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
            ctx.fillStyle = "rgba(5, 3, 10, 0.9)";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.save();
            ctx.shadowColor = "#ff0055";
            ctx.shadowBlur = 15;
            ctx.fillStyle = "#ff0055";
            ctx.font = "bold 32px monospace";
            ctx.textAlign = "center";
            ctx.fillText("SYSTEM CRASH", canvas.width / 2, canvas.height / 2 - 15);
            ctx.restore();
            
            ctx.save();
            ctx.shadowColor = "#ffea00";
            ctx.shadowBlur = 10;
            ctx.fillStyle = "#ffea00";
            ctx.font = "bold 22px monospace";
            ctx.textAlign = "center";
            ctx.fillText("COINS RECOVERED: " + score, canvas.width / 2, canvas.height / 2 + 35);
            ctx.restore();
            
            restartBtn.style.display = "block";
            return;
        }

        if (touchAccel) currentSpeed = baseSpeed * 1.9;
        else if (touchBrake) currentSpeed = baseSpeed * 0.4;
        else currentSpeed = baseSpeed;

        if (touchLeft && carX > 125) carX -= 6.5;
        if (touchRight && carX < canvas.width - 125 - carW) carX += 6.5;

        roadOffset += currentSpeed;
        if (roadOffset > 40) roadOffset = 0;

        obsY += currentSpeed;
        if (obsY > canvas.height) {
            obsY = -100;
            obsX = 125 + Math.random() * (250 - obsW);
            baseSpeed += 0.15;
        }

        coinY += currentSpeed;
        if (coinY > canvas.height) {
            coinY = -150 - Math.random() * 200;
            coinX = 130 + Math.random() * (240 - coinSize);
        }

        buildings.forEach(b => {
            b.y += currentSpeed * 0.5;
            if (b.y > canvas.height) b.y = -b.h;
        });

        if (carX < obsX + obsW && carX + carW > obsX && carY < obsY + obsH && carY + carH > obsY) {
            gameOver = true;
        }

        if (carX < coinX + coinSize && carX + carW > coinX && carY < coinY + coinSize && carY + carH > coinY) {
            score++;
            coinY = -150 - Math.random() * 200; 
            coinX = 130 + Math.random() * (240 - coinSize);
        }

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // --- DRAW SKYLINE BACKGROUND ---
        buildings.forEach(b => {
            let drawX = b.leftSide ? 5 : canvas.width - b.w - 5;
            
            // Neon building silhouette block
            ctx.fillStyle = "#0c081d";
            ctx.fillRect(drawX, b.y, b.w, b.h);
            
            // Glowing neon building edges
            ctx.save();
            ctx.shadowColor = b.color;
            ctx.shadowBlur = 8;
            ctx.fillStyle = b.color;
            ctx.fillRect(drawX, b.y, b.w, 3); // Roof line
            ctx.restore();

            // Animated cyber window arrays
            ctx.fillStyle = "rgba(255, 255, 255, 0.15)";
            for (let wx = drawX + 10; wx < drawX + b.w - 8; wx += 16) {
                for (let wy = b.y + 15; wy < b.y + b.h - 10; wy += 22) {
                    if ((Math.floor(wy + b.y)) % 3 !== 0) { // Gives variation to lights
                        ctx.fillStyle = (Math.random() > 0.3) ? "rgba(255, 234, 0, 0.2)" : "rgba(0, 255, 242, 0.2)";
                        ctx.fillRect(wx, wy, 5, 7);
                    }
                }
            }
        });

        // --- DRAW CYBERGRID HIGHWAY CORRIDOR ---
        ctx.fillStyle = "#090615";
        ctx.fillRect(120, 0, canvas.width - 240, canvas.height);

        // Cyber Grid Horizontal perspective lines simulation
        ctx.strokeStyle = "rgba(0, 255, 242, 0.07)";
        ctx.lineWidth = 1;
        for (let gy = roadOffset % 20; gy < canvas.height; gy += 20) {
            ctx.beginPath();
            ctx.moveTo(120, gy);
            ctx.lineTo(canvas.width - 120, gy);
            ctx.stroke();
        }

        // Deep Glowing Neon Curbs (Double line layer style)
        ctx.save();
        ctx.shadowColor = "#ff00ff";
        ctx.shadowBlur = 10;
        ctx.fillStyle = "#ff00ff";
        ctx.fillRect(114, 0, 3, canvas.height);
        ctx.fillRect(canvas.width - 117, 0, 3, canvas.height);
        
        ctx.shadowColor = "#00fff2";
        ctx.fillStyle = "#00fff2";
        ctx.fillRect(117, 0, 3, canvas.height);
        ctx.fillRect(canvas.width - 120, 0, 3, canvas.height);
        ctx.restore();

        // Laser Lane Dividers
        ctx.fillStyle = "rgba(0, 255, 242, 0.4)";
        for (let i = -40; i < canvas.height; i += 50) {
            ctx.fillRect(canvas.width / 2 - 2, i + roadOffset, 4, 25);
        }

        // --- DRAW GLOWING COIN ---
        ctx.save();
        ctx.shadowColor = "#ffea00";
        ctx.shadowBlur = 14;
        ctx.fillStyle = "#ffea00";
        ctx.beginPath();
        ctx.arc(coinX + coinSize/2, coinY + coinSize/2, coinSize/2, 0, Math.PI * 2);
        ctx.fill();
        
        // Inner Hologram Ring
        ctx.strokeStyle = "#ffffff";
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(coinX + coinSize/2, coinY + coinSize/2, coinSize/4, 0, Math.PI * 2);
        ctx.stroke();
        ctx.restore();

        // --- DRAW PLAYER SPORTS COUPE (CYAN) ---
        // Thrust trail fires when speed boost is triggered
        ctx.save();
        ctx.shadowColor = "#00fff2";
        ctx.shadowBlur = 12;
        ctx.fillStyle = touchAccel ? "#ffea00" : "#00fff2";
        let engineFlameH = touchAccel ? 25 : 10;
        ctx.fillRect(carX + 8, carY + carH, 6, engineFlameH);
        ctx.fillRect(carX + carW - 14, carY + carH, 6, engineFlameH);

        // Main Car Canopy Shape
        ctx.fillStyle = "#021726";
        ctx.strokeStyle = "#00fff2";
        ctx.lineWidth = 3;
        ctx.fillRect(carX, carY, carW, carH);
        ctx.strokeRect(carX, carY, carW, carH);

        // Aero Windshield Glass cockpit
        ctx.fillStyle = "#00fff2";
        ctx.fillRect(carX + 5, carY + 16, carW - 10, 20);
        ctx.fillStyle = "#05050a";
        ctx.fillRect(carX + 7, carY + 18, carW - 14, 16);

        // Headlight Beams (Front)
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(carX + 3, carY, 6, 4);
        ctx.fillRect(carX + carW - 9, carY, 6, 4);

        // Vector Brake Tail Lights
        ctx.fillStyle = touchBrake ? "#ff0055" : "#770022";
        ctx.fillRect(carX + 2, carY + carH - 5, 8, 4);
        ctx.fillRect(carX + carW - 10, carY + carH - 5, 8, 4);
        ctx.restore();

        // --- DRAW ENEMY MUSCLE CAR (MAGENTA) ---
        ctx.save();
        ctx.shadowColor = "#ff0055";
        ctx.shadowBlur = 12;
        ctx.fillStyle = "#260112";
        ctx.strokeStyle = "#ff0055";
        ctx.lineWidth = 3;
        ctx.fillRect(obsX, obsY, obsW, obsH);
        ctx.strokeRect(obsX, obsY, obsW, obsH);

        // Rear Windshield
        ctx.fillStyle = "#ff0055";
        ctx.fillRect(obsX + 5, obsY + obsH - 28, obsW - 10, 14);
        ctx.fillStyle = "#05050a";
        ctx.fillRect(obsX + 7, obsY + obsH - 26, obsW - 14, 10);

        // Menacing Threat Headlight bar indicators
        ctx.fillStyle = "#ffea00";
        ctx.fillRect(obsX + 4, obsY, 8, 4);
        ctx.fillRect(obsX + obsW - 12, obsY, 8, 4);
        ctx.restore();

        // --- CYBERPUNK HUD OVERLAY ---
        ctx.save();
        ctx.fillStyle = "rgba(5, 3, 10, 0.85)";
        ctx.fillRect(15, 15, 130, 40);
        ctx.strokeStyle = "#00fff2";
        ctx.lineWidth = 2;
        ctx.strokeRect(15, 15, 130, 40);
        
        // Data scan lines crosshairs corner details
        ctx.fillStyle = "#00fff2";
        ctx.fillRect(11, 11, 8, 2);
        ctx.fillRect(11, 11, 2, 8);

        ctx.fillStyle = "#ffea00";
        ctx.font = "bold 14px 'Courier New', monospace";
        ctx.textAlign = "left";
        ctx.fillText("CORE DATA: " + score, 26, 40);
        ctx.restore();

        requestAnimationFrame(gameLoop);
    }

    gameLoop();
});
</script>

</body>
</html>
