<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Zombie Survival — Scrolling Map</title>
<style>
  body {
    margin: 0;
    background: #0f172a;
    color: #e5e7eb;
    font-family: Arial, sans-serif;
    text-align: center;
    overflow: hidden;
  }
  canvas {
    background: #111827;
    border: 3px solid #334155;
    display: block;
    margin: 10px auto;
  }
  #fullscreenBtn, #restartBtn {
    position: fixed;
    bottom: 10px;
    background: #3b82f6;
    color: white;
    border: none;
    padding: 10px 14px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 14px;
  }
  #fullscreenBtn:hover, #restartBtn:hover {
    background: #2563eb;
  }
  #fullscreenBtn { right: 10px; }
  #restartBtn { left: 10px; display: none; }

  #weaponMenu {
    position: fixed;
    top: 200px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(15,23,42,0.95);
    padding: 20px;
    border: 2px solid #3b82f6;
    border-radius: 10px;
    display: none;
    color: white;
    font-size: 20px;
  }
</style>
</head>
<body>

<h2>Zombie Survival — Scrolling Map</h2>
<p>Move: WASD / Arrows • Shoot: SPACE • Walk into crate to choose weapon</p>

<canvas id="game" width="800" height="600"></canvas>
<button id="fullscreenBtn">Fullscreen</button>
<button id="restartBtn">Restart</button>

<div id="weaponMenu">
  <p><b>Choose Your Weapon</b></p>
  <p>1 — Pistol</p>
  <p>2 — Rapid Fire</p>
  <p>3 — LAZER GUN (Piercing Beam)</p>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const restartBtn = document.getElementById("restartBtn");
const weaponMenu = document.getElementById("weaponMenu");

// WORLD SIZE
const WORLD_W = 3000;
const WORLD_H = 3000;

function newGame() {

const keys = {};
document.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
document.addEventListener("keyup",   e => keys[e.key.toLowerCase()] = false);

let menuOpen = false;

document.addEventListener("keydown", e => {
  if (!menuOpen) return;

  if (e.key === "1") { player.gun = "pistol"; menuOpen = false; weaponMenu.style.display = "none"; }
  if (e.key === "2") { player.gun = "rapid"; menuOpen = false; weaponMenu.style.display = "none"; }
  if (e.key === "3") { player.gun = "lazer"; menuOpen = false; weaponMenu.style.display = "none"; }
});

const player = {
  x: WORLD_W/2,
  y: WORLD_H/2,
  r: 15,
  speed: 3,
  hp: 100,
  gun: "pistol",
  fireCooldown: 0
};

const bullets = [];
const bossBullets = [];
const zombies = [];
let boss = null;
const blood = [];

const boxes = [
  { x: 500, y: 500, size: 40 },
  { x: 2500, y: 2500, size: 40 },
  { x: 1500, y: 800, size: 40 }
];

const walls = [
  { x: 800, y: 800, w: 300, h: 40 },
  { x: 2000, y: 1200, w: 40, h: 300 },
  { x: 1200, y: 2000, w: 300, h: 40 },
  { x: 1800, y: 600, w: 300, h: 40 }
];

let spawnTimer = 0;
let bossShootTimer = 0;
let gameOver = false;
let kills = 0;
let totalSpawned = 0;
const MAX_ZOMBIES = 100;
let win = false;

// CAMERA
function camX() { return player.x - canvas.width/2; }
function camY() { return player.y - canvas.height/2; }

function circleRectCollision(cx, cy, r, rect) {
  const closestX = Math.max(rect.x, Math.min(cx, rect.x + rect.w));
  const closestY = Math.max(rect.y, Math.min(cy, rect.y + rect.h));
  const dx = cx - closestX;
  const dy = cy - closestY;
  return dx*dx + dy*dy < r*r;
}

function moveWithWalls(obj, dx, dy) {
  obj.x += dx;
  for (const w of walls) {
    if (circleRectCollision(obj.x, obj.y, obj.r, w)) {
      if (dx > 0) obj.x = w.x - obj.r;
      else        obj.x = w.x + w.w + obj.r;
    }
  }
  obj.y += dy;
  for (const w of walls) {
    if (circleRectCollision(obj.x, obj.y, obj.r, w)) {
      if (dy > 0) obj.y = w.y - obj.r;
      else        obj.y = w.y + w.h + obj.r;
    }
  }
}

function spawnZombie() {
  if (totalSpawned >= MAX_ZOMBIES) return;

  zombies.push({
    x: Math.random()*WORLD_W,
    y: Math.random()*WORLD_H,
    r: 14,
    speed: 1.2 + Math.random()*0.4
  });

  totalSpawned++;
}

function spawnBoss() {
  boss = {
    x: WORLD_W/2,
    y: 200,
    r: 40,
    hp: 100,
    speed: 1
  };
}

function update(dt) {
  if (gameOver || win || menuOpen) return;

  // Movement
  let dx=0, dy=0;
  if (keys["w"]||keys["arrowup"]) dy-=1;
  if (keys["s"]||keys["arrowdown"]) dy+=1;
  if (keys["a"]||keys["arrowleft"]) dx-=1;
  if (keys["d"]||keys["arrowright"]) dx+=1;

  const len = Math.hypot(dx,dy)||1;
  dx = dx/len * player.speed;
  dy = dy/len * player.speed;

  moveWithWalls(player, dx, dy);

  // Check crate collision → open menu
  for (const box of boxes) {
    if (Math.hypot(player.x-box.x, player.y-box.y) < player.r + box.size/2) {
      menuOpen = true;
      weaponMenu.style.display = "block";
    }
  }

  // Shooting
  if (player.fireCooldown > 0) player.fireCooldown -= dt;

  let fireDelay = 0.35;
  if (player.gun === "rapid") fireDelay = 0.08;
  if (player.gun === "lazer") fireDelay = 0.25;

  if (keys[" "] && player.fireCooldown <= 0) {

    // Find nearest target
    let target = null;

    if (boss) target = boss;
    else if (zombies.length > 0) {
      target = zombies.reduce((a,b)=>
        Math.hypot(a.x-player.x,a.y-player.y) <
        Math.hypot(b.x-player.x,b.y-player.y) ? a : b
      );
    }

    if (target) {
      const angle = Math.atan2(target.y-player.y, target.x-player.x);

      if (player.gun === "lazer") {
        // LAZER BEAM — hits instantly
        // Damage boss
        if (boss) {
          const dist = Math.hypot(boss.x-player.x, boss.y-player.y);
          const beamAngle = angle;
          const bossAngle = Math.atan2(boss.y-player.y, boss.x-player.x);

          if (Math.abs(beamAngle - bossAngle) < 0.1) {
            boss.hp -= 10;
            if (boss.hp <= 0) win = true;
          }
        }

        // Damage zombies in line
        for (let i=zombies.length-1;i>=0;i--){
          const z=zombies[i];
          const angZ = Math.atan2(z.y-player.y, z.x-player.x);
          if (Math.abs(angZ - angle) < 0.1) {
            zombies.splice(i,1);
            kills++;
            if (kills>=MAX_ZOMBIES && !boss) spawnBoss();
          }
        }

        // Draw beam
        bullets.push({ lazer:true, angle, life:0.1 });

      } else {
        // Normal bullets
        bullets.push({
          x: player.x,
          y: player.y,
          vx: Math.cos(angle)*7,
          vy: Math.sin(angle)*7,
          r: 4
        });
      }
    }

    player.fireCooldown = fireDelay;
  }

  // Lazer beam fade
  for (let i=bullets.length-1;i>=0;i--){
    if (bullets[i].lazer) {
      bullets[i].life -= dt;
      if (bullets[i].life <= 0) bullets.splice(i,1);
    }
  }

  // Boss shooting
  if (boss) {
    bossShootTimer -= dt;
    if (bossShootTimer <= 0) {
      bossShootTimer = 3;

      const angle = Math.atan2(player.y-boss.y, player.x-boss.x);
      bossBullets.push({
        x: boss.x,
        y: boss.y,
        vx: Math.cos(angle)*5,
        vy: Math.sin(angle)*5,
        r: 5
      });
    }
  }

  // Boss bullet movement
  for (let i=bossBullets.length-1;i>=0;i--){
    const b=bossBullets[i];
    b.x+=b.vx;
    b.y+=b.vy;

    if (Math.hypot(b.x-player.x,b.y-player.y) < b.r+player.r) {
      player.hp -= 10;
      bossBullets.splice(i,1);
      if (player.hp<=0) gameOver=true;
    }
  }

  // Normal bullets
  for (let i=bullets.length-1;i>=0;i--){
    const b=bullets[i];
    if (b.lazer) continue;

    b.x+=b.vx;
    b.y+=b.vy;

    if (b.x<0||b.x>WORLD_W||b.y<0||b.y>WORLD_H) {
      bullets.splice(i,1);
      continue;
    }

    // Boss hit
    if (boss && Math.hypot(b.x-boss.x,b.y-boss.y) < b.r+boss.r) {
      boss.hp--;
      bullets.splice(i,1);
      if (boss.hp<=0) win=true;
      continue;
    }

    // Zombie hit
    for (let j=zombies.length-1;j>=0;j--){
      const z=zombies[j];
      if (Math.hypot(b.x-z.x,b.y-z.y) < b.r+z.r) {

        // BLOOD
        for (let k=0;k<12;k++){
          const ang=Math.random()*Math.PI*2;
          const sp=Math.random()*2;
          blood.push({
            x:z.x, y:z.y,
            vx:Math.cos(ang)*sp,
            vy:Math.sin(ang)*sp,
            size:3+Math.random()*3,
            life:1
          });
        }

        zombies.splice(j,1);
        bullets.splice(i,1);
        kills++;

        if (kills>=MAX_ZOMBIES && !boss) spawnBoss();

        break;
      }
    }
  }

  // Blood update
  for (let i=blood.length-1;i>=0;i--){
    const p=blood[i];
    p.x+=p.vx;
    p.y+=p.vy;
    p.vx*=0.9;
    p.vy*=0.9;
    p.life-=dt*0.3;
    if (p.life<=0) blood.splice(i,1);
  }

  // Spawn zombies
  spawnTimer -= dt;
  if (spawnTimer <= 0) {
    spawnZombie();
    spawnTimer = 0.8;
  }

  // Zombie chase
  for (let i=zombies.length-1;i>=0;i--){
    const z=zombies[i];
    const ang=Math.atan2(player.y-z.y,player.x-z.x);
    moveWithWalls(z,Math.cos(ang)*z.speed,Math.sin(ang)*z.speed);

    if (Math.hypot(z.x-player.x,z.y-player.y) < z.r+player.r) {
      zombies.splice(i,1);
      player.hp -= 5;
      if (player.hp<=0) gameOver=true;
    }
  }

  // Boss chase
  if (boss) {
    const ang=Math.atan2(player.y-boss.y,player.x-boss.x);
    moveWithWalls(boss,Math.cos(ang)*boss.speed,Math.sin(ang)*boss.speed);

    if (Math.hypot(boss.x-player.x,boss.y-player.y) < boss.r+player.r) {
      player.hp=0;
      gameOver=true;
    }
  }
}

function draw() {
  const cx = camX();
  const cy = camY();

  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Background
  ctx.fillStyle="#0a0f1f";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  // Blood
  for (const p of blood) {
    ctx.fillStyle=`rgba(150,0,0,${p.life})`;
    ctx.beginPath();
    ctx.arc(p.x-cx,p.y-cy,p.size,0,Math.PI*2);
    ctx.fill();
  }

  // Walls
  ctx.fillStyle="#1f2937";
  for (const w of walls) {
    ctx.fillRect(w.x-cx,w.y-cy,w.w,w.h);
  }

  // Crates
  ctx.fillStyle="#facc15";
  for (const b of boxes) {
    ctx.fillRect(b.x-b.size/2-cx,b.y-b.size/2-cy,b.size,b.size);
  }

  // Player
  ctx.fillStyle="#22c55e";
  ctx.beginPath();
  ctx.arc(player.x-cx,player.y-cy,player.r,0,Math.PI*2);
  ctx.fill();

  // Lazer beams
  for (const b of bullets) {
    if (b.lazer) {
      ctx.strokeStyle="cyan";
      ctx.lineWidth=3;
      ctx.beginPath();
      ctx.moveTo(player.x-cx, player.y-cy);
      ctx.lineTo(player.x-cx + Math.cos(b.angle)*2000,
                 player.y-cy + Math.sin(b.angle)*2000);
      ctx.stroke();
    }
  }

  // Player bullets
  ctx.fillStyle="#f97316";
  for (const b of bullets) {
    if (b.lazer) continue;
    ctx.beginPath();
    ctx.arc(b.x-cx,b.y-cy,b.r,0,Math.PI*2);
    ctx.fill();
  }

  // Boss bullets
  ctx.fillStyle="#ffffff";
  for (const b of bossBullets) {
    ctx.beginPath();
    ctx.arc(b.x-cx,b.y-cy,b.r,0,Math.PI*2);
    ctx.fill();
  }

  // Zombies
  ctx.fillStyle="#ef4444";
  for (const z of zombies) {
    ctx.beginPath();
    ctx.arc(z.x-cx,z.y-cy,z.r,0,Math.PI*2);
    ctx.fill();
  }

  // Boss
  if (boss) {
    ctx.fillStyle="#7f1d1d";
    ctx.beginPath();
    ctx.arc(boss.x-cx,boss.y-cy,boss.r,0,Math.PI*2);
    ctx.fill();

    ctx.fillStyle="#ffffff";
    ctx.font="16px Arial";
    ctx.fillText("Boss HP: "+boss.hp,boss.x-cx-40,boss.y-cy-boss.r-10);
  }

  // UI
  ctx.fillStyle="#e5e7eb";
  ctx.font="14px Arial";
  ctx.fillText("HP: "+player.hp,10,20);
  ctx.fillText("Gun: "+player.gun,10,40);
  ctx.fillText("Kills: "+kills+" / 100",10,60);

  if (gameOver) {
    restartBtn.style.display="block";
    ctx.fillStyle="rgba(0,0,0,0.6)";
    ctx.fillRect(0,0,canvas.width,canvas.height);
    ctx.fillStyle="#f97316";
    ctx.font="32px Arial";
    ctx.fillText("GAME OVER",320,300);
  }

  if (win) {
    restartBtn.style.display="block";
    ctx.fillStyle="rgba(0,0,0,0.6)";
    ctx.fillRect(0,0,canvas.width,canvas.height);
    ctx.fillStyle="#22c55e";
    ctx.font="32px Arial";
    ctx.fillText("YOU WIN!",340,300);
  }
}

