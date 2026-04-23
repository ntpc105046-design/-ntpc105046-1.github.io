<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>星際守護者</title>
    <style>
        :root {
            --primary-color: #00f2ff;
            --secondary-color: #ff00ea;
            --health-color: #2ecc71;
            --boss-color: #ff3e3e;
            --exp-color: #f1c40f;
            --bg-color: #050510;
        }

        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: white;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            overflow: hidden;
            touch-action: none;
            width: 100vw;
            height: 100vh;
        }

        #game-container {
            position: relative;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            background: radial-gradient(circle at center, #1a1a3a 0%, #050510 100%);
        }

        canvas {
            display: block;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 15px;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            gap: 10px;
            box-sizing: border-box;
            z-index: 10;
        }

        .stats-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 100%;
        }

        .stat-box {
            background: rgba(0, 0, 0, 0.6);
            padding: 6px 15px;
            border-radius: 20px;
            border: 1px solid rgba(0, 242, 255, 0.3);
            font-weight: bold;
            font-size: 14px;
            color: white;
            min-width: 80px;
            text-align: center;
        }

        .exp-bar-container {
            width: 100%;
            height: 8px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 4px;
            overflow: hidden;
        }

        #exp-fill {
            width: 0%;
            height: 100%;
            background: var(--exp-color);
            box-shadow: 0 0 10px var(--exp-color);
            transition: width 0.3s ease;
        }

        /* 升級視窗 */
        .modal {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(10, 10, 30, 0.95);
            border: 2px solid var(--primary-color);
            padding: 20px;
            border-radius: 20px;
            display: none;
            flex-direction: column;
            gap: 10px;
            width: 85%;
            max-width: 300px;
            z-index: 100;
        }

        .skill-option {
            background: rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 12px;
            cursor: pointer;
            pointer-events: auto;
            border: 1px solid transparent;
        }
        .skill-option:active { background: rgba(0, 242, 255, 0.2); border-color: var(--primary-color); }
        .skill-name { color: var(--primary-color); font-weight: bold; font-size: 16px; display: block; }
        .skill-desc { font-size: 12px; opacity: 0.8; }

        /* Boss UI */
        #boss-ui {
            position: absolute;
            top: 90px;
            left: 50%;
            transform: translateX(-50%);
            width: 70%;
            display: none;
            text-align: center;
        }
        .boss-hp-bar { width: 100%; height: 10px; background: rgba(0,0,0,0.5); border: 1px solid var(--boss-color); border-radius: 5px; }
        #boss-hp-fill { width: 100%; height: 100%; background: var(--boss-color); transition: width 0.1s linear; }

        #overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8);
            display: flex; flex-direction: column; justify-content: center; align-items: center;
            z-index: 200;
        }

        .start-btn {
            padding: 15px 40px; font-size: 1.2em; background: transparent;
            color: var(--primary-color); border: 2px solid var(--primary-color);
            border-radius: 30px; cursor: pointer; pointer-events: auto;
        }

        #birthday-msg {
            position: absolute; top: 30%; width: 100%; text-align: center;
            font-size: 2em; color: var(--secondary-color); display: none;
            text-shadow: 0 0 15px var(--secondary-color); font-weight: bold;
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>
    
    <div id="ui-layer">
        <div class="stats-row">
            <div class="stat-box">Lv: <span id="player-lv">1</span></div>
            <div class="stat-box">Score: <span id="score">0</span></div>
            <div class="stat-box">HP: <span id="health">100</span>%</div>
        </div>
        <div class="exp-bar-container">
            <div id="exp-fill"></div>
        </div>
        
        <div id="boss-ui">
            <div style="color:red; font-size:12px; margin-bottom:4px; font-weight:bold;">警告：大型不明物體接近</div>
            <div class="boss-hp-bar"><div id="boss-hp-fill"></div></div>
        </div>
    </div>

    <div id="birthday-msg">Happy Birthday Sam! 🎂</div>

    <div id="level-up-modal" class="modal">
        <h2 style="text-align: center; margin-top:0;">武器升級</h2>
        <div class="skill-option" onclick="chooseSkill('bullets')">
            <span class="skill-name">增加彈道</span>
            <span class="skill-desc">每次射擊發射更多子彈</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('firerate')">
            <span class="skill-name">射速提升</span>
            <span class="skill-desc">縮短武器冷卻時間</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('damage')">
            <span class="skill-name">傷害強化</span>
            <span class="skill-desc">大幅提升子彈破壞力</span>
        </div>
    </div>

    <div id="overlay">
        <h1 style="color:var(--primary-color); margin-bottom:10px;">星際守護者</h1>
        <p style="color:white; opacity:0.6; margin-bottom:30px; font-size:14px;">拖動戰機 | 收集寶石升級 | 擊敗魔王</p>
        <button class="start-btn" onclick="startGame()">開始戰鬥</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    
    const scoreEl = document.getElementById('score');
    const healthEl = document.getElementById('health');
    const expFill = document.getElementById('exp-fill');
    const playerLvEl = document.getElementById('player-lv');
    const lvModal = document.getElementById('level-up-modal');
    const bossUi = document.getElementById('boss-ui');
    const bossHpFill = document.getElementById('boss-hp-fill');
    const birthdayMsg = document.getElementById('birthday-msg');
    const overlay = document.getElementById('overlay');

    let gameActive = false;
    let isPaused = false;
    let score = 0;
    let health = 100;
    let playerLv = 1;
    let currentExp = 0;
    let nextExp = 100;
    let frames = 0;
    let bossLevel = 0;
    let bossActive = false;
    let birthdayMode = false;

    const skills = { bullets: 1, fireInterval: 16, damage: 1 };
    const player = { x: 0, y: 0, radius: 15, lastTouchX: 0, lastTouchY: 0, isTouching: false };

    let stars = [], projectiles = [], enemyProjectiles = [], enemies = [], particles = [], gems = [], boss = null;

    function init() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        player.x = canvas.width / 2;
        player.y = canvas.height * 0.8;
        stars = Array.from({length: 50}, () => ({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height,
            size: Math.random() * 2,
            speed: Math.random() * 1.5 + 0.5
        }));
    }

    // 觸控優化
    const handleStart = (x, y) => { if(!gameActive || isPaused) return; player.isTouching = true; player.lastTouchX = x; player.lastTouchY = y; };
    const handleMove = (x, y) => {
        if(!player.isTouching || isPaused) return;
        const dx = x - player.lastTouchX;
        const dy = y - player.lastTouchY;
        player.x = Math.max(player.radius, Math.min(canvas.width - player.radius, player.x + dx));
        player.y = Math.max(100, Math.min(canvas.height - player.radius, player.y + dy));
        player.lastTouchX = x; player.lastTouchY = y;
    };
    window.addEventListener('touchstart', e => handleStart(e.touches[0].clientX, e.touches[0].clientY));
    window.addEventListener('touchmove', e => { e.preventDefault(); handleMove(e.touches[0].clientX, e.touches[0].clientY); }, {passive: false});
    window.addEventListener('touchend', () => player.isTouching = false);
    window.addEventListener('mousedown', e => handleStart(e.clientX, e.clientY));
    window.addEventListener('mousemove', e => handleMove(e.clientX, e.clientY));
    window.addEventListener('mouseup', () => player.isTouching = false);

    function createExplosion(x, y, color, count = 8) {
        for(let i=0; i<count; i++) {
            particles.push({
                x, y, vx: (Math.random()-0.5)*6, vy: (Math.random()-0.5)*6,
                radius: Math.random()*2, color, alpha: 1
            });
        }
    }

    function chooseSkill(type) {
        if(type === 'bullets') skills.bullets++;
        if(type === 'firerate') skills.fireInterval = Math.max(5, skills.fireInterval * 0.8);
        if(type === 'damage') skills.damage += 0.5;
        lvModal.style.display = 'none';
        isPaused = false;
    }

    function startGame() {
        overlay.style.display = 'none';
        gameActive = true; isPaused = false;
        score = 0; health = 100; playerLv = 1; currentExp = 0; nextExp = 100;
        bossLevel = 0; bossActive = false; boss = null;
        enemies = []; projectiles = []; enemyProjectiles = []; gems = []; particles = [];
        skills.bullets = 1; skills.fireInterval = 16; skills.damage = 1;
        updateUI();
        animate();
    }

    function updateUI() {
        scoreEl.innerText = Math.floor(score);
        healthEl.innerText = Math.max(0, Math.floor(health));
        playerLvEl.innerText = playerLv;
        expFill.style.width = (currentExp / nextExp * 100) + '%';
    }

    function gameOver() {
        gameActive = false;
        overlay.style.display = 'flex';
        overlay.querySelector('h1').innerText = '戰機被毀';
        overlay.querySelector('p').innerText = `最終得分: ${Math.floor(score)}`;
        bossUi.style.display = 'none';
    }

    function update() {
        if(!gameActive || isPaused) return;
        frames++;

        // 星空移動
        stars.forEach(s => { s.y += s.speed; if(s.y > canvas.height) s.y = 0; });

        // 玩家射擊
        if(frames % Math.floor(skills.fireInterval) === 0) {
            const spread = 0.3;
            const startAngle = -((skills.bullets - 1) * spread) / 2;
            for(let i=0; i<skills.bullets; i++) {
                projectiles.push({
                    x: player.x, y: player.y - 15,
                    vx: Math.sin(startAngle + i * spread) * 10,
                    vy: -12, damage: skills.damage
                });
            }
        }

        // 子彈邏輯
        projectiles.forEach((p, i) => {
            p.x += p.vx; p.y += p.vy;
            if(p.y < -20) projectiles.splice(i, 1);
        });

        // 敵人生成
        if(!bossActive && frames % 45 === 0) {
            const size = 30;
            enemies.push({
                x: Math.random() * (canvas.width - size), y: -size,
                w: size, h: size, hp: 1 + Math.floor(score / 5000),
                speed: 2 + Math.random() * 2, color: '#a040ff'
            });
        }

        // 敵人邏輯
        enemies.forEach((e, i) => {
            e.y += e.speed;
            if(e.y > canvas.height) enemies.splice(i, 1);
            
            projectiles.forEach((p, pi) => {
                if(p.x > e.x && p.x < e.x + e.w && p.y > e.y && p.y < e.y + e.h) {
                    e.hp -= p.damage;
                    projectiles.splice(pi, 1);
                    if(e.hp <= 0) {
                        createExplosion(e.x + e.w/2, e.y + e.h/2, e.color);
                        gems.push({ x: e.x + e.w/2, y: e.y + e.h/2 });
                        score += 150;
                        enemies.splice(i, 1);
                    }
                }
            });

            if(Math.hypot(e.x+e.w/2 - player.x, e.y+e.h/2 - player.y) < player.radius + 15) {
                health -= 20;
                enemies.splice(i, 1);
                createExplosion(player.x, player.y, 'red');
                if(health <= 0) gameOver();
            }
        });

        // 寶石與升級
        gems.forEach((g, i) => {
            const d = Math.hypot(g.x - player.x, g.y - player.y);
            if(d < 120) { g.x += (player.x - g.x)*0.1; g.y += (player.y - g.y)*0.1; }
            else g.y += 2.5;

            if(d < 25) {
                currentExp += 35; score += 20;
                if(currentExp >= nextExp) {
                    currentExp = 0; nextExp *= 1.3; playerLv++;
                    isPaused = true; lvModal.style.display = 'flex';
                }
                gems.splice(i, 1);
            } else if(g.y > canvas.height) gems.splice(i, 1);
        });

        // Boss 邏輯
        if(!bossActive && score > (bossLevel + 1) * 8000) {
            bossActive = true; bossLevel++;
            bossUi.style.display = 'block';
            const hp = 100 + bossLevel * 200;
            boss = { x: canvas.width/2 - 50, y: -100, w: 100, h: 70, hp, maxHp: hp, dir: 1, timer: 0 };
        }

        if(boss) {
            if(boss.y < 80) boss.y += 1.5;
            else {
                boss.x += 2 * boss.dir;
                if(boss.x < 10 || boss.x > canvas.width - boss.w - 10) boss.dir *= -1;
                boss.timer++;
                if(boss.timer > 50) {
                    boss.timer = 0;
                    for(let j=-1; j<=1; j++) {
                        enemyProjectiles.push({ x: boss.x + boss.w/2, y: boss.y + boss.h, vx: j*2, vy: 5 });
                    }
                }
            }

            projectiles.forEach((p, pi) => {
                if(p.x > boss.x && p.x < boss.x + boss.w && p.y > boss.y && p.y < boss.y + boss.h) {
                    boss.hp -= p.damage;
                    projectiles.splice(pi, 1);
                    if(boss.hp <= 0) {
                        createExplosion(boss.x + boss.w/2, boss.y + boss.h/2, 'red', 30);
                        score += 10000; // 擊敗 Boss 獲得大量分數
                        boss = null;
                        setTimeout(() => { 
                            bossActive = false; 
                            bossUi.style.display = 'none';
                            enemyProjectiles = []; // 清空子彈防卡住
                        }, 100);
                    }
                }
            });
            if(boss) bossHpFill.style.width = (boss.hp / boss.maxHp * 100) + '%';
        }

        enemyProjectiles.forEach((p, i) => {
            p.x += p.vx; p.y += p.vy;
            if(Math.hypot(p.x - player.x, p.y - player.y) < player.radius + 5) {
                health -= 15; enemyProjectiles.splice(i, 1);
                createExplosion(player.x, player.y, 'red');
                if(health <= 0) gameOver();
            } else if(p.y > canvas.height) enemyProjectiles.splice(i, 1);
        });

        particles.forEach((p, i) => {
            p.x += p.vx; p.y += p.vy; p.alpha -= 0.02;
            if(p.alpha <= 0) particles.splice(i, 1);
        });

        updateUI();
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        // 繪製背景星空
        ctx.fillStyle = "white";
        stars.forEach(s => ctx.fillRect(s.x, s.y, s.size, s.size));

        // 繪製玩家 (發光效果)
        ctx.save();
        ctx.translate(player.x, player.y);
        ctx.shadowBlur = 15; ctx.shadowColor = "#00f2ff";
        ctx.fillStyle = "#00f2ff";
        ctx.beginPath();
        ctx.moveTo(0, -18); ctx.lineTo(15, 12); ctx.lineTo(0, 5); ctx.lineTo(-15, 12); ctx.closePath();
        ctx.fill();
        ctx.restore();

        // 子彈
        ctx.fillStyle = "white";
        projectiles.forEach(p => { ctx.beginPath(); ctx.arc(p.x, p.y, 3, 0, Math.PI*2); ctx.fill(); });

        // 敵人
        enemies.forEach(e => {
            ctx.fillStyle = e.color;
            ctx.fillRect(e.x, e.y, e.w, e.h);
        });

        // 寶石
        gems.forEach(g => {
            ctx.fillStyle = "#f1c40f";
            ctx.beginPath(); ctx.arc(g.x, g.y, 5, 0, Math.PI*2); ctx.fill();
        });

        // Boss
        if(boss) {
            ctx.fillStyle = "#ff3e3e";
            ctx.shadowBlur = 20; ctx.shadowColor = "red";
            ctx.fillRect(boss.x, boss.y, boss.w, boss.h);
            ctx.shadowBlur = 0;
        }

        // 敵人子彈
        ctx.fillStyle = "red";
        enemyProjectiles.forEach(p => { ctx.beginPath(); ctx.arc(p.x, p.y, 5, 0, Math.PI*2); ctx.fill(); });

        // 粒子
        particles.forEach(p => {
            ctx.globalAlpha = p.alpha; ctx.fillStyle = p.color;
            ctx.beginPath(); ctx.arc(p.x, p.y, p.radius, 0, Math.PI*2); ctx.fill();
        });
        ctx.globalAlpha = 1;
    }

    function animate() {
        if(!gameActive) return;
        update();
        draw();
        requestAnimationFrame(animate);
    }

    window.addEventListener('resize', init);
    init();
</script>
</body>
</html>
