<!-- # car-racing-game
This is my first repository, I build simple car racing game.
<br>
Author - Vilesh Mansore
-->
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no">
<title>Car Racing Game</title>
<style>
  html, body { margin: 0; height: 100%; background: #1b1b1f; color: #fff; font-family: Arial, sans-serif; overflow: hidden; touch-action: none; }
  #wrap { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; }
  canvas { background: #333; border: 4px solid #555; border-radius: 8px; max-height: 88vh; max-width: 100vw; }
  #hint { margin-top: 8px; font-size: 14px; opacity: .8; text-align: center; }
</style>
</head>
<body>
<div id="wrap">
  <canvas id="game" width="360" height="560"></canvas>
  <div id="hint">Arrow keys / A-D to steer &bull; Touch: tap left or right side &bull; Space / tap to restart</div>
</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const ROAD_X = 40, ROAD_W = W - 80;
const LANES = 4, LANE_W = ROAD_W / LANES;

let player, enemies, score, speed, best = 0, over, dashY, spawnTimer, keys = {}, touchDir = 0;

try { best = +localStorage.getItem('raceBest') || 0; } catch (e) {}

function reset() {
  player = { x: W / 2 - 18, y: H - 110, w: 36, h: 70 };
  enemies = [];
  score = 0;
  speed = 4;
  over = false;
  dashY = 0;
  spawnTimer = 0;
}

function drawCar(x, y, w, h, color) {
  ctx.fillStyle = color;
  ctx.fillRect(x, y, w, h);
  ctx.fillStyle = '#9fd3ff';              // windows
  ctx.fillRect(x + 5, y + h * 0.2, w - 10, h * 0.2);
  ctx.fillRect(x + 5, y + h * 0.62, w - 10, h * 0.14);
  ctx.fillStyle = '#111';                 // wheels
  ctx.fillRect(x - 4, y + 8, 5, 16);
  ctx.fillRect(x + w - 1, y + 8, 5, 16);
  ctx.fillRect(x - 4, y + h - 24, 5, 16);
  ctx.fillRect(x + w - 1, y + h - 24, 5, 16);
}

function spawnEnemy() {
  const lane = Math.floor(Math.random() * LANES);
  const colors = ['#e74c3c', '#f1c40f', '#2ecc71', '#9b59b6', '#e67e22'];
  enemies.push({
    x: ROAD_X + lane * LANE_W + (LANE_W - 36) / 2,
    y: -80, w: 36, h: 70,
    color: colors[Math.floor(Math.random() * colors.length)],
    vy: speed * (0.5 + Math.random() * 0.4)
  });
}

function update() {
  if (over) return;

  // steering
  const move = 5;
  if (keys.ArrowLeft || keys.a || touchDir < 0) player.x -= move;
  if (keys.ArrowRight || keys.d || touchDir > 0) player.x += move;
  player.x = Math.max(ROAD_X + 4, Math.min(ROAD_X + ROAD_W - player.w - 4, player.x));

  // road scroll + difficulty
  dashY = (dashY + speed) % 40;
  speed += 0.0015;
  score += speed * 0.02;

  // enemies
  spawnTimer--;
  if (spawnTimer <= 0) {
    spawnEnemy();
    spawnTimer = Math.max(25, 70 - speed * 4);
  }
  for (const e of enemies) {
    e.y += speed + e.vy * 0.3;
    if (player.x < e.x + e.w && player.x + player.w > e.x &&
        player.y < e.y + e.h && player.y + player.h > e.y) {
      over = true;
      best = Math.max(best, Math.floor(score));
      try { localStorage.setItem('raceBest', best); } catch (err) {}
    }
  }
  enemies = enemies.filter(e => e.y < H + 100);
}

function draw() {
  ctx.fillStyle = '#2d7a32';               // grass
  ctx.fillRect(0, 0, W, H);
  ctx.fillStyle = '#3a3a3f';               // road
  ctx.fillRect(ROAD_X, 0, ROAD_W, H);
  ctx.fillStyle = '#fff';                  // edges
  ctx.fillRect(ROAD_X - 4, 0, 4, H);
  ctx.fillRect(ROAD_X + ROAD_W, 0, 4, H);

  ctx.fillStyle = '#ddd';                  // lane dashes
  for (let l = 1; l < LANES; l++) {
    for (let y = -40 + dashY; y < H; y += 40) {
      ctx.fillRect(ROAD_X + l * LANE_W - 2, y, 4, 22);
    }
  }

  enemies.forEach(e => drawCar(e.x, e.y, e.w, e.h, e.color));
  drawCar(player.x, player.y, player.w, player.h, '#3498db');

  ctx.fillStyle = '#fff';
  ctx.font = 'bold 18px Arial';
  ctx.textAlign = 'left';
  ctx.fillText('Score: ' + Math.floor(score), 10, 26);
  ctx.textAlign = 'right';
  ctx.fillText('Best: ' + best, W - 10, 26);

  if (over) {
    ctx.fillStyle = 'rgba(0,0,0,.65)';
    ctx.fillRect(0, 0, W, H);
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    ctx.font = 'bold 36px Arial';
    ctx.fillText('GAME OVER', W / 2, H / 2 - 20);
    ctx.font = '20px Arial';
    ctx.fillText('Score: ' + Math.floor(score), W / 2, H / 2 + 15);
    ctx.fillText('Press Space or tap to restart', W / 2, H / 2 + 50);
  }
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

// keyboard
window.addEventListener('keydown', e => {
  keys[e.key] = true;
  if (e.key === ' ' && over) reset();
  if (['ArrowLeft', 'ArrowRight', ' '].includes(e.key)) e.preventDefault();
});
window.addEventListener('keyup', e => { keys[e.key] = false; });

// touch / mouse
function pointer(e) {
  const r = canvas.getBoundingClientRect();
  const px = (e.touches ? e.touches[0].clientX : e.clientX) - r.left;
  touchDir = px < r.width / 2 ? -1 : 1;
}
canvas.addEventListener('touchstart', e => { e.preventDefault(); if (over) reset(); pointer(e); }, { passive: false });
canvas.addEventListener('touchmove', e => { e.preventDefault(); pointer(e); }, { passive: false });
canvas.addEventListener('touchend', () => { touchDir = 0; });
canvas.addEventListener('mousedown', e => { if (over) reset(); pointer(e); });
window.addEventListener('mouseup', () => { touchDir = 0; });

reset();
loop();
</script>
</body>
</html>
