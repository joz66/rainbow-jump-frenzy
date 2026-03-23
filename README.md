<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Rainbow Jump Frenzy - Super Fun for Kids!</title>
<style>
  body { margin:0; background:#000; overflow:hidden; font-family:Arial; }
  canvas { display:block; margin:0 auto; background:#87CEEB; }
  #startScreen, #gameOverScreen {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
    color: white; text-align: center; font-size: 40px; text-shadow: 0 0 10px #ff0;
    display: none; pointer-events: none;
  }
  button {
    margin-top: 20px; padding: 15px 30px; font-size: 28px; background: #ff69b4;
    color: white; border: none; border-radius: 20px; cursor: pointer;
    box-shadow: 0 5px 0 #c71585;
  }
  button:active { transform: translateY(4px); box-shadow: 0 1px 0 #c71585; }
</style>
</head>
<body>

<div id="startScreen">
  🌈 RAINBOW JUMP FRENZY 🌈<br>
  <button onclick="startGame()">PLAY NOW!</button>
</div>

<canvas id="game" width="800" height="600"></canvas>

<div id="gameOverScreen">
  GAME OVER 😢<br>
  Score: <span id="finalScore">0</span><br>
  <button onclick="restartGame()">PLAY AGAIN</button>
</div>

<script>
// ============== RAINBOW JUMP FRENZY ==============
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
let gameRunning = false;

let player = { x: 200, y: 400, vy: 0, size: 50, color: '#ff0' };
let gravity = 0.8;
let jumpPower = -18;
let score = 0;
let lives = 3;
let speed = 4;

let items = [];     // collectibles (stars, hearts, candies)
let obstacles = []; // spiky clouds
let particles = []; // rainbow sparkles

let keys = {};
let lastTime = 0;
let frame = 0;

document.addEventListener('keydown', e => keys[e.key] = true);
document.addEventListener('keyup', e => keys[e.key] = false);
canvas.addEventListener('click', () => { if (gameRunning) player.vy = jumpPower; });

function createItem() {
  items.push({
    x: canvas.width + Math.random()*100,
    y: 100 + Math.random()*(canvas.height-200),
    size: 35,
    type: Math.random() > 0.5 ? 'star' : Math.random() > 0.5 ? 'heart' : 'candy',
    color: ['#ff0', '#ff69b4', '#00ff7f'][Math.floor(Math.random()*3)]
  });
}

function createObstacle() {
  obstacles.push({
    x: canvas.width + 50,
    y: 150 + Math.random()*(canvas.height-300),
    size: 60
  });
}

function createParticles(x, y, color) {
  for (let i = 0; i < 12; i++) {
    particles.push({
      x, y,
      vx: Math.random()*8 - 4,
      vy: Math.random()*8 - 6,
      life: 25,
      color
    });
  }
}

function drawBackground() {
  // sky gradient
  let grad = ctx.createLinearGradient(0, 0, 0, canvas.height);
  grad.addColorStop(0, '#87CEEB');
  grad.addColorStop(1, '#E0F7FF');
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // moving clouds
  ctx.fillStyle = 'rgba(255,255,255,0.9)';
  for (let i = 0; i < 5; i++) {
    let cx = (frame * 1.5 + i*200) % (canvas.width + 300) - 100;
    ctx.beginPath();
    ctx.ellipse(cx, 100 + i*40, 80, 35, 0, 0, Math.PI*2);
    ctx.fill();
  }

  // ground grass
  ctx.fillStyle = '#90EE90';
  ctx.fillRect(0, canvas.height-60, canvas.width, 60);
}

function drawPlayer() {
  // body
  ctx.fillStyle = player.color;
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.size/2, 0, Math.PI*2);
  ctx.fill();

  // eyes
  ctx.fillStyle = '#000';
  ctx.beginPath(); ctx.arc(player.x-12, player.y-10, 8, 0, Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.arc(player.x+12, player.y-10, 8, 0, Math.PI*2); ctx.fill();

  ctx.fillStyle = '#fff';
  ctx.beginPath(); ctx.arc(player.x-10, player.y-12, 4, 0, Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.arc(player.x+14, player.y-12, 4, 0, Math.PI*2); ctx.fill();

  // happy mouth
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 4;
  ctx.beginPath();
  ctx.arc(player.x, player.y+10, 15, 0.2, Math.PI-0.2);
  ctx.stroke();

  // rainbow trail when jumping high
  if (player.vy < -5) {
    ctx.strokeStyle = `hsla(${frame*10 % 360}, 100%, 70%, 0.6)`;
    ctx.lineWidth = 12;
    ctx.beginPath();
    ctx.moveTo(player.x-30, player.y+10);
    ctx.lineTo(player.x-60, player.y+30);
    ctx.stroke();
  }
}

function update() {
  if (!gameRunning) return;

  frame++;

  // player physics
  player.vy += gravity;
  player.y += player.vy;

  // floor & ceiling
  if (player.y > canvas.height - 110) {
    player.y = canvas.height - 110;
    player.vy = 0;
  }
  if (player.y < 50) {
    player.y = 50;
    player.vy = 2;
  }

  // spawn stuff
  if (frame % 35 === 0) createItem();
  if (frame % 80 === 0) createObstacle();

  // update items
  for (let i = items.length-1; i >= 0; i--) {
    let it = items[i];
    it.x -= speed + 1;

    // collect
    let dx = it.x - player.x;
    let dy = it.y - player.y;
    if (dx*dx + dy*dy < (it.size/2 + player.size/2)**2) {
      score += 10;
      createParticles(it.x, it.y, it.color);
      items.splice(i, 1);

      // fun sound effect simulation
      if (Math.random() > 0.7) player.vy -= 4; // bouncy reward!
    }

    if (it.x < -50) items.splice(i, 1);
  }

  // update obstacles
  for (let i = obstacles.length-1; i >= 0; i--) {
    let ob = obstacles[i];
    ob.x -= speed + 2;

    let dx = ob.x - player.x;
    let dy = ob.y - player.y;
    if (dx*dx + dy*dy < (ob.size/2 + player.size/2)**2) {
      lives--;
      createParticles(ob.x, ob.y, '#ff0000');
      obstacles.splice(i, 1);

      if (lives <= 0) endGame();
    }

    if (ob.x < -100) obstacles.splice(i, 1);
  }

  // update particles
  for (let i = particles.length-1; i >= 0; i--) {
    let p = particles[i];
    p.x += p.vx;
    p.y += p.vy;
    p.vy += 0.3;
    p.life--;
    if (p.life <= 0) particles.splice(i, 1);
  }

  // increase difficulty
  if (frame % 300 === 0) speed = Math.min(speed + 0.5, 9);

  // random rainbow color change for player
  if (frame % 12 === 0) player.color = `hsl(${frame*15 % 360}, 100%, 60%)`;
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBackground();
  drawPlayer();

  // draw collectibles
  for (let it of items) {
    ctx.font = '40px Arial';
    ctx.textAlign = 'center';
    ctx.fillStyle = it.color;
    let emoji = it.type === 'star' ? '⭐' : it.type === 'heart' ? '❤️' : '🍭';
    ctx.fillText(emoji, it.x, it.y + 15);
  }

  // draw spiky clouds
  for (let ob of obstacles) {
    ctx.fillStyle = '#ddd';
    ctx.beginPath();
    ctx.arc(ob.x, ob.y, ob.size/2, 0, Math.PI*2);
    ctx.fill();

    ctx.fillStyle = '#888';
    ctx.font = '30px Arial';
    ctx.fillText('☁️', ob.x, ob.y + 12);
    ctx.fillText('⚡', ob.x-10, ob.y-15);
  }

  // draw sparkles
  for (let p of particles) {
    ctx.globalAlpha = p.life / 25;
    ctx.fillStyle = p.color;
    ctx.fillRect(p.x, p.y, 8, 8);
  }
  ctx.globalAlpha = 1;

  // HUD
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 36px Arial';
  ctx.textAlign = 'left';
  ctx.fillText('SCORE: ' + score, 30, 60);

  ctx.fillText('❤️'.repeat(lives), canvas.width - 180, 60);

  if (score > 0 && score % 100 === 0 && frame % 30 < 10) {
    ctx.font = 'bold 60px Arial';
    ctx.fillStyle = '#ff0';
    ctx.textAlign = 'center';
    ctx.fillText('LEVEL UP! 🌟', canvas.width/2, 150);
  }
}

function gameLoop(timestamp) {
  if (!lastTime) lastTime = timestamp;
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

function startGame() {
  document.getElementById('startScreen').style.display = 'none';
  gameRunning = true;
  score = 0;
  lives = 3;
  speed = 4;
  player.y = 300;
  player.vy = 0;
  items = [];
  obstacles = [];
  particles = [];
  frame = 0;
  requestAnimationFrame(gameLoop);
}

function endGame() {
  gameRunning = false;
  document.getElementById('finalScore').textContent = score;
  document.getElementById('gameOverScreen').style.display = 'block';
}

function restartGame() {
  document.getElementById('gameOverScreen').style.display = 'none';
  startGame();
}

// Show start screen
document.getElementById('startScreen').style.display = 'block';

</script>
</body>
</html>
