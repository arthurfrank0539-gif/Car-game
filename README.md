<!DOCTYPE html>
<html>
<head>
    <title>My Browser Car Game</title>
    <style>
        body {
            margin: 0;
            background: #222;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
        }
        canvas {
            background: #000;
            border: 4px solid #fff;
        }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="699" height="400"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Game State
let score = 0;
let gameOver = false;

// Car properties
let carX = 325; // Centered horizontally
let carY = 300; // Positioned near the bottom
const carWidth = 40;
const carHeight = 60;
const speed = 6;

// Obstacle properties
let obstacleX = Math.random() * (canvas.width - 40);
let obstacleY = -50;
const obstacleWidth = 40;
const obstacleHeight = 40;
let obstacleSpeed = 4;

// Track keyboard inputs
const keys = {};
window.addEventListener("keydown", e => keys[e.key] = true);
window.addEventListener("keyup", e => keys[e.key] = false);

// Collision detection function
function checkCollision(rect1X, rect1Y, rect1W, rect1H, rect2X, rect2Y, rect2W, rect2H) {
    return rect1X < rect2X + rect2W &&
           rect1X + rect1W > rect2X &&
           rect1Y < rect2Y + rect2H &&
           rect1Y + rect1H > rect2Y;
}

// Main Game Loop
function gameLoop() {
    if (gameOver) {
        // Draw Game Over text
        ctx.fillStyle = "white";
        ctx.font = "40px Arial";
        ctx.fillText("GAME OVER", canvas.width / 2 - 120, canvas.height / 2);
        ctx.font = "20px Arial";
        ctx.fillText("Refresh the page to try again!", canvas.width / 2 - 120, canvas.height / 2 + 40);
        return;
    }

    // 1. Handle Movement & Screen Boundaries (Invisible Walls)
    if ((keys["ArrowLeft"] || keys["a"]) && carX > 0) {
        carX -= speed;
    }
    if ((keys["ArrowRight"] || keys["d"]) && carX < canvas.width - carWidth) {
        carX += speed;
    }
    if ((keys["ArrowUp"] || keys["w"]) && carY > 0) {
        carY -= speed;
    }
    if ((keys["ArrowDown"] || keys["s"]) && carY < canvas.height - carHeight) {
        carY += speed;
    }

    // 2. Move the Obstacle
    obstacleY += obstacleSpeed;

    // If the obstacle goes off the bottom, reset it to the top and add points
    if (obstacleY > canvas.height) {
        obstacleY = -50;
        obstacleX = Math.random() * (canvas.width - obstacleWidth);
        score += 1;
        obstacleSpeed += 0.5; // Make the game get harder over time!
    }

    // 3. Check for Crashing
    if (checkCollision(carX, carY, carWidth, carHeight, obstacleX, obstacleY, obstacleWidth, obstacleHeight)) {
        gameOver = true;
    }

    // 4. Clear Screen
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 5. Draw Player Car (Sleek red vertical rectangle)
    ctx.fillStyle = "red";
    ctx.fillRect(carX, carY, carWidth, carHeight);

    // 6. Draw Obstacle (Blue block)
    ctx.fillStyle = "dodgerblue";
    ctx.fillRect(obstacleX, obstacleY, obstacleWidth, obstacleHeight);

    // 7. Draw Score Text
    ctx.fillStyle = "white";
    ctx.font = "20px Arial";
    ctx.fillText("Score: " + score, 20, 30);

    // Keep the loop running smoothly
    requestAnimationFrame(gameLoop);
}

// Start the game
gameLoop();
</script>

</body>
</html>
