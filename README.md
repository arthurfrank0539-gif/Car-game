<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        * {
            box-sizing: border-box;
        }
        body {
            margin: 0;
            padding: 0;
            background: #070810;
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
        .header-title {
            margin: 8px 0;
            font-size: 1.4rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 4px;
            color: #00fff2;
            text-shadow: 0 0 10px rgba(0, 255, 242, 0.3);
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 16px;
            overflow: hidden;
            width: 92%;
            max-width: 420px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.7);
            border: 2px solid #00fff2;
            background: #090a14;
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 55%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 32px;
            font-size: 1rem;
            font-weight: 600;
            letter-spacing: 1px;
            background: #00fff2;
            color: #000000;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 20px rgba(0, 255, 242, 0.5);
            z-index: 20;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 92%;
            max-width: 420px;
            margin-top: 15px;
            margin-bottom: 10px;
            padding: 0 5px;
        }
        .btn-group {
            display: flex;
            gap: 12px;
        }
        .btn {
            width: 60px;
            height: 60px;
            background: rgba(9, 10, 20, 0.8);
            border: 2px solid #00fff2;
            border-radius: 16px;
            color: #00fff2;
            font-size: 1.4rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: 0 4px 10px rgba(0, 255, 242, 0.1);
        }
        .btn:active {
            background: #00fff2;
            color: #090a14;
            box-shadow: 0 0 15px rgba(0, 255, 242, 0.6);
        }
        .btn-action {
            color: #ff2d55;
            border-color: #ff2d55;
            box-shadow: 0 4px 10px rgba(255, 45, 85, 0.1);
        }
        .btn-action:active {
            background: #ff2d55;
            color: #090a14;
            box-shadow: 0 0 15px rgba(255, 45, 85, 0.6);
        }
    </style>
</head>
<body>

<div class="header-title">Neon Rider</div>

<div class="container">
    <canvas id="gameCanvas" width="500" height="450"></canvas>
    <button id="restartBtn">Drive Again</button>
</div>

<div class="controls-pad">
    <div class="btn-group">
        <div class="btn" id="leftBtn">←</div>
        <div class="btn" id="rightBtn">→</div>
    </div>
    <div class="btn-group">
        <div class="btn btn-action" id="brakeBtn">↓</div>
        <div class="btn btn-action" id="accelBtn">↑</div>
    </div>
</div>

<script>
function startNeonRider() {
    const canvas = document.getElementById("gameCanvas");
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    const restartBtn = document.getElementById("restartBtn");

    let score = 0;
    let highScore = localStorage.getItem("neonRider_highScore") ? parseInt(localStorage.getItem("neonRider_highScore")) : 0;
    let gameOver = false;
    let roadOffset = 0;
    let baseSpeed = 4;
    let currentSpeed = baseSpeed;

    let carX = 232;
    let carY = 340;
    const carW = 36;
    const carH = 66;

    let obsW = 36;
    let obsH = 66;
    let obsX = 140 + Math.random() * (220 - obsW);
    let obsY = -100;
    let obsSpeedModifier = 1;
    let obsDirection = 1;

    let coinX = 140 + Math.random() * (220 - 18);
    let coinY = -300;
    const coinSize = 18;

    let buildings = [
        { leftSide: true, xOffset: 8, y: 0, w: 85, h: 200, accentColor: "#00fff2" },
        { leftSide: true, xOffset: 18, y: 250, w: 75, h: 150, accentColor: "#ff00bb" },
        { leftSide: false, xOffset: 8, y: 50, w: 85, h: 220, accentColor: "#bc00ff" },
        { leftSide: false, xOffset: 20, y: 300, w: 70, h: 160, accentColor: "#00ff66" }
    ];

    let touchLeft = false;
    let touchRight = false;
    let touchAccel = false;
    let touchBrake = false;

    let audioCtx = null;

    function initAudio() {
        if (!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }
        if (audioCtx && audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
    }

    function playSound(type) {
        if (!audioCtx || audioCtx.state === 'suspended') return;
        try {
            let osc = audioCtx.createOscillator();
            let gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);

            if (type === 'coin') {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(587.33, audioCtx.currentTime); 
                osc.frequency.setValueAtTime(880, audioCtx.currentTime + 0.08); 
                gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.25);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.25);
            } else if (type === 'crash') {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(140, audioCtx.currentTime);
                osc.frequency.linearRampToValueAtTime(30, audioCtx.currentTime + 0.45);
                gain.gain.setValueAtTime(0.4, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.5);
            }
        } catch(e) { console.log(e); }
    }

    function addEvent(id, startEvt, endEvt, setter) {
        const el = document.getElementById(id);
        if (!el) return;
        el.addEventListener(startEvt, (e) => { 
            e.preventDefault(); 
            initAudio(); 
            setter(true); 
        });
        el.addEventListener(endEvt, (e) => { 
            e.preventDefault(); 
            setter(false); 
        });
    }

    addEvent("leftBtn", "touchstart", "touchend", (v) => touchLeft = v);
    addEvent("rightBtn", "touchstart", "touchend", (v) => touchRight = v);
    addEvent("accelBtn", "touchstart", "touchend", (v) => touchAccel = v);
    addEvent("brakeBtn", "touchstart", "touchend", (v) => touchBrake = v);

    addEvent("leftBtn", "mousedown", "mouseup", (v) => touchLeft = v);
    addEvent("rightBtn", "mousedown", "mouseup", (v) => touchRight = v);
    addEvent("accelBtn", "mousedown", "mouseup", (v) => touchAccel = v);
    addEvent("brakeBtn", "mousedown", "mouseup", (v) => touchBrake = v);

    function resetGame() {
        initAudio();
        score = 0;
        gameOver = false;
        carX = 232;
        obsY = -100;
        obsX = 140 + Math.random() * (220 - obsW);
        obsSpeedModifier = 1;
        coinY = -300;
        coinX = 140 + Math.random() * (220 - coinSize);
        baseSpeed = 4;
        restartBtn.style.display = "none";
        gameLoop();
    }

    if (restartBtn) {
        restartBtn.addEventListener("click", resetGame);
        restartBtn.addEventListener("touchstart", (e) => {
            e.preventDefault();
            resetGame();
        });
    }

    function gameLoop() {
        if (gameOver) {
            ctx.fillStyle = "rgba(10, 11, 21, 0.95)";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = "#ffffff";
            ctx.font = "600 26px sans-serif";
            ctx.textAlign = "center";
            ctx.fillText("CRASH DETECTED", canvas.width / 2, canvas.height / 2 - 40);
            
            ctx.fillStyle = "rgba(255,255,255,0.7)";
            ctx.font = "16px sans-serif";
            ctx.fillText("Score: " + score, canvas.width / 2, canvas.height / 2);
            
            ctx.fillStyle = "#00fff2";
            ctx.font = "bold 16px sans-serif";
            ctx.fillText("BEST RUN: " + highScore, canvas.width / 2, canvas.height / 2 + 30);
            
            if (restartBtn) restartBtn.style.display = "block";
            return;
        }

        if (touchAccel) currentSpeed = baseSpeed * 1.8;
        else if (touchBrake) currentSpeed = baseSpeed * 0.4;
        else currentSpeed = baseSpeed;

        if (touchLeft && carX > 140) carX -= 5.5;
        if (touchRight && carX < canvas.width - 140 - carW) carX += 5.5;

        roadOffset += currentSpeed;
        if (roadOffset > 60) roadOffset = 0;

        obsY += currentSpeed * obsSpeedModifier;
        obsX += obsDirection * 0.8;
        if (obsX < 140 || obsX > canvas.width - 140 - obsW) {
            obsDirection *= -1;
        }

        if (obsY > canvas.height) {
            obsY = -100;
            obsX = 140 + Math.random() * (220 - obsW);
            obsSpeedModifier = 0.8 + Math.random() * 0.7; 
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
            if (score > highScore) {
                highScore = score;
                localStorage.setItem("neonRider_highScore", highScore);
            }
            playSound('crash');
        }

        if (carX < coinX + coinSize && carX + carW > coinX && carY < coinY + coinSize && carY + carH > coinY) {
            score++;
            playSound('coin');
            coinY = -150 - Math.random() * 250; 
            coinX = 145 + Math.random() * (210 - coinSize);
        }

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // SCENERY BUILDINGS
        buildings.forEach(b => {
            let drawX = b.leftSide ? b.xOffset : canvas.width - b.w - b.xOffset;
            ctx.fillStyle = "#16182c";
            ctx.fillRect(drawX, b.y, b.w, b.h);

            for (let wx = drawX + 8; wx < drawX + b.w - 8; wx += 16) {
                for (let wy = b.y + 12; wy < b.y + b.h - 12; wy += 22) {
                    if ((Math.floor(wx + wy)) % 6 !== 0) {
                        ctx.fillStyle = "rgba(255, 230, 120, 0.28)";
                    } else {
                        ctx.fillStyle = "rgba(255, 255, 255, 0.05)";
                    }
                    ctx.fillRect(wx, wy, 6, 10);
                }
            }
            ctx.fillStyle = b.accentColor;
            ctx.fillRect(drawX, b.y, b.w, 4);

            ctx.strokeStyle = "rgba(255, 255, 255, 0.25)";
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.moveTo(drawX + b.w / 2, b.y);
            ctx.lineTo(drawX + b.w / 2, b.y - 14);
            ctx.stroke();

            ctx.fillStyle = (Math.floor(Date.now() / 300) % 2 === 0) ? "#ff3b30" : "rgba(255,255,255,0.05)";
            ctx.beginPath();
            ctx.arc(drawX + b.w / 2, b.y - 14, 3, 0, Math.PI * 2);
            ctx.fill();
        });

        // HIGHWAY CORRIDOR
        let roadGrad = ctx.createLinearGradient(135, 0, canvas.width - 135, 0);
        roadGrad.addColorStop(0, '#0c0e18');
        roadGrad.addColorStop(0.5, '#16192e');
        roadGrad.addColorStop(1, '#0c0e18');
        ctx.fillStyle = roadGrad;
        ctx.fillRect(135, 0, canvas.width - 270, canvas.height);

        ctx.fillStyle = "rgba(255, 255, 255, 0.08)";
        ctx.fillRect(135, 0, 3, canvas.height);
        ctx.fillRect(canvas.width - 138, 0, 3, canvas.height);

        ctx.fillStyle = "rgba(255, 255, 255, 0.18)";
        for (let i = -60; i < canvas.height; i += 60) {
            ctx.fillRect(canvas.width / 2 - 1.5, i + roadOffset, 3, 30);
        }

        // ENERGY COIN
        ctx.fillStyle = "#ffcc00";
        ctx.beginPath();
        ctx.arc(coinX + coinSize/2, coinY + coinSize/2, coinSize/2, 0, Math.PI * 2);
        ctx.fill();

        // REAR EXHAUST FLAMES (IF ACCELERATING)
        if (touchAccel) {
            ctx.fillStyle = "rgba(0, 160, 255, 0.4)";
            ctx.fillRect(carX + 5, carY + carH, 4, 12);
            ctx.fillRect(carX + carW - 9, carY + carH, 4, 12);
        }

        // PLAYER CAR (PERFECTLY STRAIGHT CHASSIS)
        let carGrad = ctx.createLinearGradient(carX, carY, carX + carW, carY);
        carGrad.addColorStop(0, '#2f80ed');
        carGrad.addColorStop(1, '#00c6ff');
        ctx.fillStyle = carGrad;
        ctx.beginPath();
        ctx.moveTo(carX + 6, carY); 
        ctx.lineTo(carX + carW - 6, carY);
        ctx.lineTo(carX + carW, carY + 14); 
        ctx.lineTo(carX + carW, carY + carH - 6); 
        ctx.lineTo(carX + carW - 3, carY + carH);
        ctx.lineTo(carX + 3, carY + carH);
        ctx.lineTo(carX, carY + carH - 6);
        ctx.lineTo(carX, carY + 14);
        ctx.closePath();
        ctx.fill();

        // WINDSHIELD & COCKPIT
        ctx.fillStyle = "#090a12";
        ctx.beginPath();
        ctx.moveTo(carX + 6, carY + 16);
        ctx.lineTo(carX + carW - 6, carY + 16);
        ctx.lineTo(carX + carW - 4, carY + 36);
        ctx.lineTo(carX + 4, carY + 36);
        ctx.closePath();
        ctx.fill();

        // HEADLIGHTS
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(carX + 3, carY + 1, 5, 2);
        ctx.fillRect(carX + carW - 8, carY + 1, 5, 2);

        // BRAKE LIGHTS
        ctx.fillStyle = touchBrake ? "#ff3b30" : "#b2131b";
        ctx.fillRect(carX + 4, carY + carH - 3, carW - 8, 2);

        // RIVAL SEDAN (OBSTACLE)
        let obsGrad = ctx.createLinearGradient(obsX, obsY, obsX + obsW, obsY);
        obsGrad.addColorStop(0, '#8e2de2');
        obsGrad.addColorStop(1, '#4a00e0');
        ctx.fillStyle = obsGrad;
        ctx.fillRect(obsX, obsY, obsW, obsH);

        ctx.fillStyle = "#0c0d15";
        ctx.fillRect(obsX + 4, obsY + 18, obsW - 8, 20);

        ctx.fillStyle = "#ff2d55";
        ctx.fillRect(obsX + 2, obsY + obsH - 3, 5, 2);
        ctx.fillRect(obsX + obsW - 7, obsY + obsH - 3, 5, 2);

        // HUD OVERLAY
        ctx.fillStyle = "rgba(10, 12, 26, 0.75)";
        ctx.fillRect(15, 15, 125, 45);
        ctx.strokeStyle = "rgba(255, 255, 255, 0.1)";
        ctx.strokeRect(15, 15, 125, 45);
        
        ctx.fillStyle = "#ffffff";
        ctx.font = "bold 11px sans-serif";
        ctx.textAlign = "left";
        ctx.fillText("SCORE: " + score, 25, 32);
        
        ctx.fillStyle = "#00fff2";
        ctx.fillText("HI-SCORE: " + highScore, 25, 48);

        requestAnimationFrame(gameLoop);
    }

    gameLoop();
}

// Robust fallback setup for hosted files and early browser bindings
if (document.readyState === "complete" || document.readyState === "interactive") {
    startNeonRider();
} else {
    window.addEventListener("DOMContentLoaded", startNeonRider);
}
</script>

</body>
</html>
