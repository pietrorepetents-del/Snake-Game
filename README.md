<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SERPENTE</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@400;700;900&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg:        #050810;
    --panel:     #0a0f1e;
    --border:    #1a2a4a;
    --snake:     #00ff88;
    --snake2:    #00cc66;
    --head:      #ffffff;
    --food:      #ff4466;
    --food2:     #ff0033;
    --wall:      #334466;
    --wall-lit:  #4466aa;
    --pw-speed:  #ffaa00;
    --pw-inv:    #aa44ff;
    --pw-score:  #00ccff;
    --text:      #c8d8f0;
    --dim:       #4a5a7a;
    --accent:    #00ff88;
    --danger:    #ff4466;
    --glow:      0 0 8px #00ff8866, 0 0 20px #00ff8833;
    --glow-red:  0 0 8px #ff446688, 0 0 20px #ff446633;
    --cell:      20px;
    --cols:      25;
    --rows:      25;
  }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Share Tech Mono', monospace;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 16px;
    padding: 16px;
    overflow: hidden;
  }

  body::before {
    content: '';
    position: fixed; inset: 0;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0,0,0,0.08) 2px,
      rgba(0,0,0,0.08) 4px
    );
    pointer-events: none;
    z-index: 9999;
  }

  body::after {
    content: '';
    position: fixed; inset: 0;
    background: radial-gradient(ellipse at center, transparent 55%, rgba(0,0,0,0.7) 100%);
    pointer-events: none;
    z-index: 9998;
  }

  h1 {
    font-family: 'Orbitron', monospace;
    font-weight: 900;
    font-size: clamp(1.4rem, 4vw, 2.2rem);
    letter-spacing: 0.3em;
    color: var(--accent);
    text-shadow: var(--glow);
    text-transform: uppercase;
  }

  .layout { display: flex; gap: 16px; align-items: flex-start; }

  .sidebar { display: flex; flex-direction: column; gap: 12px; width: 160px; }

  .panel {
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 12px;
  }

  .panel-label {
    font-size: 0.6rem;
    letter-spacing: 0.2em;
    color: var(--dim);
    text-transform: uppercase;
    margin-bottom: 4px;
  }

  .panel-value {
    font-family: 'Orbitron', monospace;
    font-size: 1.4rem;
    font-weight: 700;
    color: var(--accent);
    text-shadow: var(--glow);
  }

  .panel-value.danger { color: var(--danger); text-shadow: var(--glow-red); }

  .level-bar-wrap {
    margin-top: 6px; height: 4px;
    background: var(--border); border-radius: 2px; overflow: hidden;
  }
  .level-bar {
    height: 100%; background: var(--accent);
    box-shadow: var(--glow); transition: width 0.3s ease;
  }

  .powerup-list { display: flex; flex-direction: column; gap: 6px; }
  .powerup-item {
    display: flex; align-items: center; gap: 8px;
    font-size: 0.7rem; color: var(--dim); transition: color 0.2s;
  }
  .powerup-item.active { color: var(--text); }
  .powerup-dot {
    width: 8px; height: 8px; border-radius: 50%;
    background: var(--border); transition: background 0.2s, box-shadow 0.2s;
  }
  .powerup-item.active .powerup-dot { box-shadow: 0 0 6px currentColor; }
  .pw-speed  .powerup-dot { background: var(--pw-speed); }
  .pw-inv    .powerup-dot { background: var(--pw-inv); }
  .pw-score  .powerup-dot { background: var(--pw-score); }

  .pw-timer {
    margin-left: auto; font-size: 0.65rem;
    font-family: 'Orbitron', monospace; color: var(--dim);
  }
  .powerup-item.active .pw-timer { color: var(--text); }

  .mode-btns { display: flex; flex-direction: column; gap: 6px; }
  .mode-btn {
    background: transparent; border: 1px solid var(--border);
    border-radius: 3px; color: var(--dim);
    font-family: 'Share Tech Mono', monospace; font-size: 0.7rem;
    letter-spacing: 0.1em; padding: 6px 8px; cursor: pointer;
    text-align: left; transition: all 0.15s; text-transform: uppercase;
  }
  .mode-btn:hover { border-color: var(--accent); color: var(--text); }
  .mode-btn.active {
    border-color: var(--accent); color: var(--accent);
    text-shadow: var(--glow); background: rgba(0,255,136,0.05);
  }

  .canvas-wrap {
    position: relative; border: 1px solid var(--border);
    border-radius: 4px; overflow: hidden;
    box-shadow: 0 0 40px rgba(0,255,136,0.05), inset 0 0 40px rgba(0,0,0,0.5);
  }

  canvas { display: block; image-rendering: pixelated; }

  .overlay {
    position: absolute; inset: 0;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    background: rgba(5,8,16,0.88); gap: 16px;
    opacity: 0; pointer-events: none; transition: opacity 0.25s;
  }
  .overlay.visible { opacity: 1; pointer-events: all; }

  .overlay-title {
    font-family: 'Orbitron', monospace; font-weight: 900;
    font-size: 1.8rem; letter-spacing: 0.2em;
    color: var(--accent); text-shadow: var(--glow);
  }
  .overlay-title.red { color: var(--danger); text-shadow: var(--glow-red); }

  .overlay-sub {
    font-size: 0.75rem; color: var(--dim);
    letter-spacing: 0.15em; text-align: center; line-height: 1.8;
  }

  .btn {
    font-family: 'Orbitron', monospace; font-size: 0.75rem;
    font-weight: 700; letter-spacing: 0.2em; padding: 10px 28px;
    background: transparent; border: 1px solid var(--accent);
    color: var(--accent); border-radius: 3px; cursor: pointer;
    text-transform: uppercase; transition: all 0.15s;
    text-shadow: var(--glow); box-shadow: 0 0 12px rgba(0,255,136,0.15);
  }
  .btn:hover { background: rgba(0,255,136,0.12); box-shadow: 0 0 20px rgba(0,255,136,0.3); }

  .final-score {
    font-family: 'Orbitron', monospace; font-size: 1rem;
    color: var(--text); letter-spacing: 0.15em;
  }
  .final-score span { color: var(--accent); text-shadow: var(--glow); }

  .controls-bar {
    display: flex; gap: 24px; font-size: 0.65rem;
    color: var(--dim); letter-spacing: 0.1em;
  }
  .controls-bar kbd {
    background: var(--panel); border: 1px solid var(--border);
    border-radius: 2px; padding: 1px 5px;
    font-family: 'Share Tech Mono', monospace; color: var(--text);
  }
</style>
</head>
<body>

<h1>Serpente</h1>

<div class="layout">
  <div class="sidebar">
    <div class="panel">
      <div class="panel-label">Score</div>
      <div class="panel-value" id="score">0</div>
    </div>
    <div class="panel">
      <div class="panel-label">High Score</div>
      <div class="panel-value" id="highscore">0</div>
    </div>
    <div class="panel">
      <div class="panel-label">Level</div>
      <div class="panel-value" id="level">1</div>
      <div class="level-bar-wrap">
        <div class="level-bar" id="level-bar" style="width:0%"></div>
      </div>
    </div>
    <div class="panel">
      <div class="panel-label">Power-Ups</div>
      <div class="powerup-list">
        <div class="powerup-item pw-speed" id="pw-speed-item">
          <div class="powerup-dot"></div>
          <span>Speed+</span>
          <span class="pw-timer" id="pw-speed-timer"></span>
        </div>
        <div class="powerup-item pw-inv" id="pw-inv-item">
          <div class="powerup-dot"></div>
          <span>Shield</span>
          <span class="pw-timer" id="pw-inv-timer"></span>
        </div>
        <div class="powerup-item pw-score" id="pw-score-item">
          <div class="powerup-dot"></div>
          <span>2× Score</span>
          <span class="pw-timer" id="pw-score-timer"></span>
        </div>
      </div>
    </div>
    <div class="panel">
      <div class="panel-label">Modo</div>
      <div class="mode-btns">
        <button class="mode-btn active" data-mode="classic">Classic</button>
        <button class="mode-btn" data-mode="walls">Paredes</button>
        <button class="mode-btn" data-mode="portal">Portal</button>
      </div>
    </div>
  </div>

  <div class="canvas-wrap">
    <canvas id="game" width="500" height="500"></canvas>
    <div class="overlay visible" id="overlay-start">
      <div class="overlay-title">SERPENTE</div>
      <div class="overlay-sub">Selecione um modo e pressione<br><kbd>ESPAÇO</kbd> ou <kbd>ENTER</kbd> para iniciar</div>
      <button class="btn" id="btn-start">Iniciar</button>
    </div>
    <div class="overlay" id="overlay-pause">
      <div class="overlay-title">PAUSADO</div>
      <div class="overlay-sub">Pressione <kbd>P</kbd> para continuar</div>
      <button class="btn" id="btn-resume">Continuar</button>
    </div>
    <div class="overlay" id="overlay-gameover">
      <div class="overlay-title red">GAME OVER</div>
      <div class="final-score">Score: <span id="final-score">0</span></div>
      <div class="final-score" id="new-record" style="display:none">★ Novo Recorde! ★</div>
      <button class="btn" id="btn-restart">Reiniciar</button>
    </div>
  </div>

  <div class="sidebar">
    <div class="panel">
      <div class="panel-label">Vidas</div>
      <div class="panel-value" id="lives">3</div>
    </div>
    <div class="panel">
      <div class="panel-label">Comprimento</div>
      <div class="panel-value" id="length">3</div>
    </div>
    <div class="panel">
      <div class="panel-label">Velocidade</div>
      <div class="panel-value" id="speed-display">1</div>
    </div>
    <div class="panel">
      <div class="panel-label">Modo Atual</div>
      <div class="panel-value" id="mode-display" style="font-size:0.85rem;letter-spacing:0.1em">Classic</div>
    </div>
    <div class="panel" style="font-size:0.65rem; color: var(--dim); line-height:2;">
      <div class="panel-label">Legenda</div>
      <div>🟩 Cobra</div>
      <div>🔴 Comida</div>
      <div>🟡 Speed+</div>
      <div>🟣 Shield</div>
      <div>🔵 2× Score</div>
      <div>⬛ Parede</div>
    </div>
  </div>
</div>

<div class="controls-bar">
  <span><kbd>↑↓←→</kbd> mover</span>
  <span><kbd>P</kbd> pausar</span>
  <span><kbd>R</kbd> reiniciar</span>
  <span><kbd>Space</kbd> iniciar</span>
</div>

<script>
(() => {
  'use strict';

  const COLS = 25, ROWS = 25, CELL = 20;
  const INITIAL_INTERVAL = 150;
  const LEVEL_UP_SCORE   = 50;
  const PW_DURATION      = 8000;
  const POWERUP_SPAWN_CHANCE = 0.25;
  const MAX_LIVES        = 3;
  const MODES = { classic: 'classic', walls: 'walls', portal: 'portal' };

  let snake, dir, nextDir, food, powerups, walls;
  let score, level, lives, highscore;
  let gameState;
  let loopTimer, lastTime, accumulator;
  let mode = MODES.classic;
  let activePowerups;
  let scoreMultiplier, isInvincible;
  let foodEatenSinceLastPW;
  let wallPattern;

  const canvas     = document.getElementById('game');
  const ctx        = canvas.getContext('2d');
  const $score     = document.getElementById('score');
  const $highscore = document.getElementById('highscore');
  const $level     = document.getElementById('level');
  const $levelBar  = document.getElementById('level-bar');
  const $lives     = document.getElementById('lives');
  const $length    = document.getElementById('length');
  const $speedDisp = document.getElementById('speed-display');
  const $modeDisp  = document.getElementById('mode-display');
  const $finalScore = document.getElementById('final-score');
  const $newRecord  = document.getElementById('new-record');
  const overlayStart    = document.getElementById('overlay-start');
  const overlayPause    = document.getElementById('overlay-pause');
  const overlayGameover = document.getElementById('overlay-gameover');

  const rand = (n) => Math.floor(Math.random() * n);
  const key  = (x, y) => `${x},${y}`;

  function cellOccupied(x, y, excludeFood = false) {
    if (snake.some(s => s.x === x && s.y === y)) return true;
    if (!excludeFood && food && food.x === x && food.y === y) return true;
    if (walls.has(key(x, y))) return true;
    return false;
  }

  function randomFreeCell() {
    let x, y, tries = 0;
    do { x = rand(COLS); y = rand(ROWS); tries++; }
    while (cellOccupied(x, y) && tries < 500);
    return tries >= 500 ? null : { x, y };
  }

  function buildWalls() {
    walls = new Set();
    if (mode !== MODES.walls) return;
    wallPattern = wallPattern || rand(3);
    const p = wallPattern;
    if (p === 0) {
      for (let i = 5; i < 20; i++) { walls.add(key(12, i)); walls.add(key(i, 12)); }
    } else if (p === 1) {
      [[3,3],[3,4],[4,3],[4,4],[20,3],[20,4],[21,3],[21,4],
       [3,20],[3,21],[4,20],[4,21],[20,20],[20,21],[21,20],[21,21]]
        .forEach(([x,y]) => walls.add(key(x,y)));
      for (let i = 10; i <= 14; i++) for (let j = 10; j <= 14; j++) walls.add(key(i,j));
    } else {
      for (let i = 3; i < 22; i++) {
        if (i !== 12) walls.add(key(i, 8));
        if (i !== 12) walls.add(key(i, 16));
      }
      for (let j = 9; j < 16; j++) { walls.add(key(6,j)); walls.add(key(18,j)); }
    }
  }

  function init() {
    score = 0; level = 1; lives = MAX_LIVES;
    activePowerups = []; powerups = [];
    scoreMultiplier = 1; isInvincible = false;
    foodEatenSinceLastPW = 0; wallPattern = null;
    buildWalls();
    const cx = Math.floor(COLS/2), cy = Math.floor(ROWS/2);
    snake = [{x:cx,y:cy},{x:cx-1,y:cy},{x:cx-2,y:cy}];
    dir = {x:1,y:0}; nextDir = {x:1,y:0};
    spawnFood(); updateUI();
  }

  function spawnFood() {
    const cell = randomFreeCell();
    food = cell || {x:0,y:0};
  }

  const PW_TYPES = ['speed','inv','score'];

  function maybeSpawnPowerup() {
    foodEatenSinceLastPW++;
    if (powerups.length >= 2) return;
    if (Math.random() > POWERUP_SPAWN_CHANCE) return;
    const type = PW_TYPES[rand(PW_TYPES.length)];
    const cell = randomFreeCell();
    if (!cell) return;
    powerups.push({...cell, type, spawnedAt: performance.now()});
    foodEatenSinceLastPW = 0;
  }

  function getInterval() {
    const baseSpeed = Math.max(60, INITIAL_INTERVAL - (level-1)*10);
    return activePowerups.some(p => p.type==='speed') ? Math.max(40, baseSpeed*0.6) : baseSpeed;
  }

  function startLoop() {
    if (loopTimer) cancelAnimationFrame(loopTimer);
    lastTime = performance.now(); accumulator = 0;
    loop(lastTime);
  }

  function loop(ts) {
    if (gameState !== 'running') return;
    const dt = ts - lastTime; lastTime = ts; accumulator += dt;
    const interval = getInterval();
    while (accumulator >= interval) {
      tick(); accumulator -= interval;
      if (gameState !== 'running') return;
    }
    draw();
    loopTimer = requestAnimationFrame(loop);
  }

  function tick() {
    dir = {...nextDir};
    const head = snake[0];
    let nx = head.x + dir.x, ny = head.y + dir.y;
    if (mode === MODES.portal || mode === MODES.classic) {
      nx = (nx + COLS) % COLS; ny = (ny + ROWS) % ROWS;
    }
    const hitWall = walls.has(key(nx, ny));
    const hitSelf = snake.some((s,i) => i>0 && s.x===nx && s.y===ny);
    const oob = mode===MODES.walls && (nx<0||nx>=COLS||ny<0||ny>=ROWS);
    if (hitWall || hitSelf || oob) {
      if (isInvincible) return;
      handleDeath(); return;
    }
    snake.unshift({x:nx,y:ny});
    if (nx===food.x && ny===food.y) {
      score += 10 * scoreMultiplier;
      if (score > highscore) highscore = score;
      maybeSpawnPowerup(); spawnFood(); checkLevelUp();
    } else { snake.pop(); }
    for (let i=powerups.length-1; i>=0; i--) {
      const pw = powerups[i];
      if (nx===pw.x && ny===pw.y) { applyPowerup(pw.type); powerups.splice(i,1); }
    }
    const now = performance.now();
    activePowerups = activePowerups.filter(p => {
      if (now >= p.endsAt) { deactivatePowerup(p.type); return false; } return true;
    });
    powerups = powerups.filter(pw => now - pw.spawnedAt < 10000);
    updateUI();
  }

  function handleDeath() {
    lives--; updateUI();
    if (lives <= 0) { gameState='dead'; showOverlay('gameover'); draw(); return; }
    activePowerups=[]; scoreMultiplier=1; isInvincible=false; powerups=[];
    const cx=Math.floor(COLS/2), cy=Math.floor(ROWS/2);
    snake=[{x:cx,y:cy},{x:cx-1,y:cy},{x:cx-2,y:cy}];
    dir={x:1,y:0}; nextDir={x:1,y:0}; spawnFood();
  }

  function applyPowerup(type) {
    const endsAt = performance.now() + PW_DURATION;
    activePowerups = activePowerups.filter(p => p.type !== type);
    activePowerups.push({type, endsAt});
    if (type==='score') scoreMultiplier=2;
    if (type==='inv')   isInvincible=true;
  }

  function deactivatePowerup(type) {
    if (type==='score') scoreMultiplier=1;
    if (type==='inv')   isInvincible=false;
  }

  function checkLevelUp() {
    const newLevel = Math.floor(score/LEVEL_UP_SCORE)+1;
    if (newLevel > level) { level=newLevel; wallPattern=null; buildWalls(); }
  }

  const COLORS = {
    bg:'#050810', grid:'#0c1222', food:'#ff4466',
    head:'#ffffff', body1:'#00ff88', body2:'#00cc66',
    wall:'#1e3055', wallLit:'#2a4470',
    pwSpeed:'#ffaa00', pwInv:'#aa44ff', pwScore:'#00ccff',
  };

  function draw() {
    const W=COLS*CELL, H=ROWS*CELL;
    ctx.clearRect(0,0,W,H);
    ctx.fillStyle=COLORS.bg; ctx.fillRect(0,0,W,H);
    ctx.strokeStyle=COLORS.grid; ctx.lineWidth=0.5;
    for(let x=0;x<=COLS;x++){ctx.beginPath();ctx.moveTo(x*CELL,0);ctx.lineTo(x*CELL,H);ctx.stroke();}
    for(let y=0;y<=ROWS;y++){ctx.beginPath();ctx.moveTo(0,y*CELL);ctx.lineTo(W,y*CELL);ctx.stroke();}
    walls.forEach(k=>{
      const[wx,wy]=k.split(',').map(Number);
      ctx.fillStyle=COLORS.wall; ctx.fillRect(wx*CELL+1,wy*CELL+1,CELL-2,CELL-2);
      ctx.fillStyle=COLORS.wallLit;
      ctx.fillRect(wx*CELL+1,wy*CELL+1,CELL-2,2);
      ctx.fillRect(wx*CELL+1,wy*CELL+1,2,CELL-2);
    });
    const now=performance.now();
    powerups.forEach(pw=>{
      const age=now-pw.spawnedAt;
      const pulse=0.6+0.4*Math.sin(now/200);
      const alpha=age>7000?1-(age-7000)/3000:1;
      ctx.globalAlpha=alpha*pulse;
      const color=pw.type==='speed'?COLORS.pwSpeed:pw.type==='inv'?COLORS.pwInv:COLORS.pwScore;
      ctx.fillStyle=color;
      ctx.beginPath();
      ctx.arc(pw.x*CELL+CELL/2,pw.y*CELL+CELL/2,CELL/2-3,0,Math.PI*2);
      ctx.fill();
      ctx.globalAlpha=alpha;
      ctx.fillStyle='#000';
      ctx.font=`${CELL*0.55}px monospace`;
      ctx.textAlign='center'; ctx.textBaseline='middle';
      const icon=pw.type==='speed'?'⚡':pw.type==='inv'?'🛡':'×2';
      ctx.fillText(icon,pw.x*CELL+CELL/2,pw.y*CELL+CELL/2+1);
    });
    ctx.globalAlpha=1; ctx.textAlign='left';
    if(food){
      const fp=0.7+0.3*Math.sin(now/400);
      ctx.fillStyle=COLORS.food; ctx.shadowColor=COLORS.food; ctx.shadowBlur=12*fp;
      ctx.beginPath();
      ctx.arc(food.x*CELL+CELL/2,food.y*CELL+CELL/2,CELL/2-2,0,Math.PI*2);
      ctx.fill(); ctx.shadowBlur=0;
    }
    const invPulse=isInvincible?(0.5+0.5*Math.sin(now/100)):0;
    for(let i=snake.length-1;i>=0;i--){
      const seg=snake[i], isHead=i===0, t=i/(snake.length-1||1);
      if(isHead){
        ctx.fillStyle=isInvincible?`rgba(170,68,255,${0.8+0.2*invPulse})`:COLORS.head;
        ctx.shadowColor=isInvincible?'#aa44ff':COLORS.body1;
        ctx.shadowBlur=isInvincible?15:10;
      } else {
        const r1=parseInt(COLORS.body1.slice(1,3),16),g1=parseInt(COLORS.body1.slice(3,5),16),b1=parseInt(COLORS.body1.slice(5,7),16);
        const r2=parseInt(COLORS.body2.slice(1,3),16),g2=parseInt(COLORS.body2.slice(3,5),16),b2=parseInt(COLORS.body2.slice(5,7),16);
        const r=Math.round(r1+(r2-r1)*t),g=Math.round(g1+(g2-g1)*t),b=Math.round(b1+(b2-b1)*t);
        ctx.fillStyle=isInvincible?`rgba(170,68,255,${0.6-t*0.4})`:`rgb(${r},${g},${b})`;
        ctx.shadowColor=isInvincible?'#aa44ff':COLORS.body1;
        ctx.shadowBlur=isInvincible?6*(1-t):4*(1-t);
      }
      const pad=isHead?1:2, br=isHead?4:3;
      roundRect(ctx,seg.x*CELL+pad,seg.y*CELL+pad,CELL-pad*2,CELL-pad*2,br);
      ctx.fill();
    }
    ctx.shadowBlur=0;
    if(snake.length>0&&gameState!=='dead') drawEyes(snake[0],dir);
  }

  function drawEyes(head,d){
    const cx=head.x*CELL+CELL/2,cy=head.y*CELL+CELL/2,eo=3,er=2;
    let e1,e2;
    if(d.x===1)      {e1={x:cx+4,y:cy-eo};e2={x:cx+4,y:cy+eo};}
    else if(d.x===-1){e1={x:cx-4,y:cy-eo};e2={x:cx-4,y:cy+eo};}
    else if(d.y===-1){e1={x:cx-eo,y:cy-4};e2={x:cx+eo,y:cy-4};}
    else             {e1={x:cx-eo,y:cy+4};e2={x:cx+eo,y:cy+4};}
    ctx.fillStyle='#000';
    [e1,e2].forEach(e=>{ctx.beginPath();ctx.arc(e.x,e.y,er,0,Math.PI*2);ctx.fill();});
  }

  function roundRect(ctx,x,y,w,h,r){
    ctx.beginPath();
    ctx.moveTo(x+r,y);ctx.lineTo(x+w-r,y);ctx.quadraticCurveTo(x+w,y,x+w,y+r);
    ctx.lineTo(x+w,y+h-r);ctx.quadraticCurveTo(x+w,y+h,x+w-r,y+h);
    ctx.lineTo(x+r,y+h);ctx.quadraticCurveTo(x,y+h,x,y+h-r);
    ctx.lineTo(x,y+r);ctx.quadraticCurveTo(x,y,x+r,y);
    ctx.closePath();
  }

  function updateUI(){
    $score.textContent=score; $highscore.textContent=highscore;
    $level.textContent=level; $lives.textContent=lives;
    $length.textContent=snake.length; $speedDisp.textContent=level;
    $levelBar.style.width=(score%LEVEL_UP_SCORE)/LEVEL_UP_SCORE*100+'%';
    $lives.className='panel-value'+(lives===1?' danger':'');
    const now=performance.now();
    ['speed','inv','score'].forEach(type=>{
      const active=activePowerups.find(p=>p.type===type);
      const item=document.getElementById(`pw-${type}-item`);
      const timer=document.getElementById(`pw-${type}-timer`);
      if(active){item.classList.add('active');timer.textContent=Math.ceil((active.endsAt-now)/1000)+'s';}
      else{item.classList.remove('active');timer.textContent='';}
    });
  }

  function showOverlay(which){
    overlayStart.classList.remove('visible');
    overlayPause.classList.remove('visible');
    overlayGameover.classList.remove('visible');
    if(which==='start')   overlayStart.classList.add('visible');
    if(which==='pause')   overlayPause.classList.add('visible');
    if(which==='gameover'){
      $finalScore.textContent=score;
      $newRecord.style.display=(score>0&&score>=highscore)?'block':'none';
      overlayGameover.classList.add('visible');
    }
  }

  const DIR_MAP={
    ArrowUp:{x:0,y:-1},ArrowDown:{x:0,y:1},ArrowLeft:{x:-1,y:0},ArrowRight:{x:1,y:0},
    w:{x:0,y:-1},s:{x:0,y:1},a:{x:-1,y:0},d:{x:1,y:0},
  };

  document.addEventListener('keydown',e=>{
    const d=DIR_MAP[e.key];
    if(d){
      e.preventDefault();
      if(d.x!=-dir.x||d.y!=-dir.y) nextDir=d;
      if(gameState==='idle') startGame();
      return;
    }
    if(e.key===' '||e.key==='Enter'){
      e.preventDefault();
      if(gameState==='idle') startGame();
      else if(gameState==='dead') restartGame();
      return;
    }
    if((e.key==='p'||e.key==='P')&&gameState==='running'){pauseGame();return;}
    if((e.key==='p'||e.key==='P')&&gameState==='paused'){resumeGame();return;}
    if(e.key==='r'||e.key==='R') restartGame();
  });

  document.getElementById('btn-start').addEventListener('click',startGame);
  document.getElementById('btn-resume').addEventListener('click',resumeGame);
  document.getElementById('btn-restart').addEventListener('click',restartGame);

  document.querySelectorAll('.mode-btn').forEach(btn=>{
    btn.addEventListener('click',()=>{
      if(gameState==='running') return;
      mode=btn.dataset.mode;
      document.querySelectorAll('.mode-btn').forEach(b=>b.classList.remove('active'));
      btn.classList.add('active');
      $modeDisp.textContent=btn.textContent;
      init(); draw();
    });
  });

  let touchStart=null;
  canvas.addEventListener('touchstart',e=>{touchStart={x:e.touches[0].clientX,y:e.touches[0].clientY};},{passive:true});
  canvas.addEventListener('touchend',e=>{
    if(!touchStart) return;
    const dx=e.changedTouches[0].clientX-touchStart.x;
    const dy=e.changedTouches[0].clientY-touchStart.y;
    if(Math.abs(dx)>Math.abs(dy)){const d=dx>0?DIR_MAP.ArrowRight:DIR_MAP.ArrowLeft;if(d.x!=-dir.x)nextDir=d;}
    else{const d=dy>0?DIR_MAP.ArrowDown:DIR_MAP.ArrowUp;if(d.y!=-dir.y)nextDir=d;}
    touchStart=null;
    if(gameState==='idle') startGame();
  },{passive:true});

  function startGame(){if(gameState==='running')return;gameState='running';showOverlay(null);startLoop();}
  function pauseGame(){gameState='paused';showOverlay('pause');draw();}
  function resumeGame(){gameState='running';showOverlay(null);startLoop();}
  function restartGame(){
    if(loopTimer)cancelAnimationFrame(loopTimer);
    highscore=Math.max(highscore,score);
    init();gameState='running';showOverlay(null);draw();startLoop();
  }

  try{const s=parseInt(localStorage.getItem('snake_hs')||'0',10);highscore=isNaN(s)?0:s;}catch{highscore=0;}
  window.addEventListener('beforeunload',()=>{try{localStorage.setItem('snake_hs',highscore);}catch{}});

  gameState='idle'; init(); draw(); showOverlay('start');
  function idleLoop(ts){if(gameState!=='idle')return;draw();requestAnimationFrame(idleLoop);}
  requestAnimationFrame(idleLoop);

})();
</script>
</body>
</html>
