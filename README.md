
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Catch the Falling Stars</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-height: 100vh;
  font-family: Arial, sans-serif;
  text-align: center;
  color: white;
  background: linear-gradient(135deg, #1e1b4b, #581c87);
  padding: 20px;
}

h1 {
  font-size: 32px;
  margin: 10px;
}

p {
  color: #ddd6fe;
}

.game-container {
  max-width: 500px;
  margin: auto;
}

.scoreboard {
  display: flex;
  justify-content: space-around;
  gap: 10px;
  margin: 20px 0;
}

.score-box {
  background: rgba(255,255,255,0.15);
  padding: 12px;
  border-radius: 12px;
  flex: 1;
}

.score-box span {
  display: block;
  font-size: 25px;
  font-weight: bold;
  margin-top: 5px;
}

canvas {
  width: 100%;
  max-width: 480px;
  border: 3px solid #a78bfa;
  border-radius: 15px;
  background: #172554;
  display: block;
  margin: auto;
  touch-action: pan-y;
}

button {
  padding: 14px 25px;
  margin: 8px;
  border: none;
  border-radius: 12px;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  background: #facc15;
  color: #422006;
}

button:hover {
  background: #fde047;
}

.controls button {
  font-size: 28px;
  width: 100px;
}

#message {
  font-size: 20px;
  font-weight: bold;
  color: #fef08a;
}

@media (max-width: 400px) {
  body {
    padding: 10px;
  }

  h1 {
    font-size: 25px;
  }
}
</style>
</head>

<body>

<div class="game-container">

  <h1>⭐ Catch the Falling Stars ⭐</h1>

  <p>Move the basket and catch the stars!</p>

  <div class="scoreboard">

    <div class="score-box">
      Score
      <span id="score">0</span>
    </div>

    <div class="score-box">
      Lives
      <span id="lives">3</span>
    </div>

    <div class="score-box">
      Best
      <span id="best">0</span>
    </div>

  </div>

  <canvas id="gameCanvas"
          width="480"
          height="600"></canvas>

  <p id="message">Press Start to Play!</p>

  <button onclick="startGame()">▶ Start Game</button>

  <button onclick="restartGame()">↻ Restart</button>

  <div class="controls">
    <button id="leftBtn">◀</button>
    <button id="rightBtn">▶</button>
  </div>

  <p>Keyboard: Use ← → or A / D</p>

</div>

<script>

// Get canvas
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Get scoreboard elements
const scoreText = document.getElementById("score");
const livesText = document.getElementById("lives");
const bestText = document.getElementById("best");
const message = document.getElementById("message");

// Game variables
let score = 0;
let lives = 3;
let best = 0;
let gameRunning = false;

let stars = [];

let basket = {
  x: 190,
  y: 540,
  width: 100,
  height: 30,
  speed: 7
};

let keys = {
  left: false,
  right: false
};

let lastTime = 0;
let spawnTimer = 0;

// Start game
function startGame() {

  if (gameRunning) return;

  gameRunning = true;

  message.textContent = "Catch the stars!";

  lastTime = 0;

  requestAnimationFrame(gameLoop);
}

// Restart game
function restartGame() {

  score = 0;
  lives = 3;
  stars = [];

  basket.x = 190;

  scoreText.textContent = score;
  livesText.textContent = lives;

  message.textContent = "Press Start to Play!";

  gameRunning = false;

  draw();
}

// Create a star
function createStar() {

  stars.push({

    x: Math.random() * (canvas.width - 30) + 15,

    y: -20,

    size: 15,

    speed: 2 + Math.random() * 3,

    rotation: 0

  });

}

// Draw star
function drawStar(x, y, radius, rotation) {

  ctx.save();

  ctx.translate(x, y);

  ctx.rotate(rotation);

  ctx.beginPath();

  for (let i = 0; i < 10; i++) {

    const angle = -Math.PI / 2 + i * Math.PI / 5;

    const r = i % 2 === 0
      ? radius
      : radius * 0.45;

    const px = Math.cos(angle) * r;

    const py = Math.sin(angle) * r;

    if (i === 0) {

      ctx.moveTo(px, py);

    } else {

      ctx.lineTo(px, py);

    }

  }

  ctx.closePath();

  ctx.fillStyle = "#facc15";

  ctx.shadowColor = "#fde68a";

  ctx.shadowBlur = 12;

  ctx.fill();

  ctx.restore();

}

// Draw game screen
function draw() {

  // Background
  const gradient = ctx.createLinearGradient(
    0, 0, 0, canvas.height
  );

  gradient.addColorStop(0, "#172554");

  gradient.addColorStop(1, "#581c87");

  ctx.fillStyle = gradient;

  ctx.fillRect(
    0,
    0,
    canvas.width,
    canvas.height
  );

  // Moon
  ctx.fillStyle = "#fef3c7";

  ctx.beginPath();

  ctx.arc(70, 70, 35, 0, Math.PI * 2);

  ctx.fill();

  ctx.fillStyle = "#172554";

  ctx.beginPath();

  ctx.arc(85, 60, 35, 0, Math.PI * 2);

  ctx.fill();

  // Background stars
  ctx.fillStyle = "rgba(255,255,255,0.5)";

  for (let i = 0; i < 40; i++) {

    let x = (i * 83 + 20) % canvas.width;

    let y = (i * 47 + 20) % canvas.height;

    ctx.beginPath();

    ctx.arc(x, y, 1.5, 0, Math.PI * 2);

    ctx.fill();

  }

  // Falling stars
  stars.forEach(star => {

    drawStar(
      star.x,
      star.y,
      star.size,
      star.rotation
    );

  });

  // Basket
  ctx.fillStyle = "#a16207";

  ctx.beginPath();

  ctx.roundRect(
    basket.x,
    basket.y,
    basket.width,
    basket.height,
    8
  );

  ctx.fill();

  // Basket rim
  ctx.fillStyle = "#f59e0b";

  ctx.beginPath();

  ctx.roundRect(
    basket.x - 5,
    basket.y - 6,
    basket.width + 10,
    10,
    5
  );

  ctx.fill();

  // Basket text
  ctx.fillStyle = "#422006";

  ctx.font = "bold 16px Arial";

  ctx.textAlign = "center";

  ctx.fillText(
    "CATCH!",
    basket.x + basket.width / 2,
    basket.y + 20
  );

}

// Update game
function update(deltaTime) {

  // Move basket left
  if (keys.left) {

    basket.x -= basket.speed;

  }

  // Move basket right
  if (keys.right) {

    basket.x += basket.speed;

  }

  // Keep basket inside canvas
  basket.x = Math.max(
    0,
    Math.min(
      canvas.width - basket.width,
      basket.x
    )
  );

  // Spawn stars
  spawnTimer += deltaTime;

  if (spawnTimer > 700) {

    createStar();

    spawnTimer = 0;

  }

  // Move stars
  stars.forEach(star => {

    star.y += star.speed * deltaTime / 16.67;

    star.rotation += 0.03;

  });

  // Check collision
  for (let i = stars.length - 1; i >= 0; i--) {

    const star = stars[i];

    const caught =

      star.y + star.size >= basket.y &&

      star.y - star.size <= basket.y + basket.height &&

      star.x >= basket.x &&

      star.x <= basket.x + basket.width;

    // Star caught
    if (caught) {

      score++;

      if (score > best) {

        best = score;

      }

      scoreText.textContent = score;

      bestText.textContent = best;

      stars.splice(i, 1);

    }

    // Star missed
    else if (star.y - star.size > canvas.height) {

      lives--;

      livesText.textContent = lives;

      stars.splice(i, 1);

      // Game over
      if (lives <= 0) {

        gameRunning = false;

        message.textContent =
          "Game Over! Your score: " + score;

        return;

      }

    }

  }

}

// Game loop
function gameLoop(timestamp) {

  if (!gameRunning) return;

  if (!lastTime) {

    lastTime = timestamp;

  }

  const deltaTime = Math.min(
    timestamp - lastTime,
    50
  );

  lastTime = timestamp;

  update(deltaTime);

  draw();

  requestAnimationFrame(gameLoop);

}

// Keyboard controls
document.addEventListener("keydown", function(event) {

  if (
    event.key === "ArrowLeft" ||
    event.key.toLowerCase() === "a"
  ) {

    event.preventDefault();

    keys.left = true;

  }

  if (
    event.key === "ArrowRight" ||
    event.key.toLowerCase() === "d"
  ) {

    event.preventDefault();

    keys.right = true;

  }

});

// Stop movement when key released
document.addEventListener("keyup", function(event) {

  if (
    event.key === "ArrowLeft" ||
    event.key.toLowerCase() === "a"
  ) {

    keys.left = false;

  }

  if (
    event.key === "ArrowRight" ||
    event.key.toLowerCase() === "d"
  ) {

    keys.right = false;

  }

});

// Touch and mouse controls
function holdButton(direction) {

  keys[direction] = true;

}

function releaseButton(direction) {

  keys[direction] = false;

}

const leftBtn = document.getElementById("leftBtn");

const rightBtn = document.getElementById("rightBtn");

leftBtn.addEventListener(
  "pointerdown",
  () => holdButton("left")
);

rightBtn.addEventListener(
  "pointerdown",
  () => holdButton("right")
);

leftBtn.addEventListener(
  "pointerup",
  () => releaseButton("left")
);

rightBtn.addEventListener(
  "pointerup",
  () => releaseButton("right")
);

leftBtn.addEventListener(
  "pointerleave",
  () => releaseButton("left")
);

rightBtn.addEventListener(
  "pointerleave",
  () => releaseButton("right")
);

leftBtn.addEventListener(
  "pointercancel",
  () => releaseButton("left")
);

rightBtn.addEventListener(
  "pointercancel",
  () => releaseButton("right")
);

// Draw initial screen
draw();

</script>

</body>
</html>
