<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>星際守護者</title>
    <style>
        :root {
            --primary-color: #00f2ff;
            --secondary-color: #ff00ea;
            --health-color: #2ecc71;
            --boss-color: #ff3e3e;
            --exp-color: #f1c40f;
            --bg-color: #000000;
        }

        * {
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }

        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: var(--bg-color);
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            color: white;
            position: fixed;
        }

        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            background: #000;
        }

        canvas {
            display: block;
            touch-action: none;
            background: radial-gradient(circle at center, #111 0%, #000 100%);
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            padding: env(safe-area-inset-top) 15px 0 15px;
            display: flex;
            flex-direction: column;
            z-index: 5;
        }

        .stats-top {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 0;
            font-size: 14px;
            font-weight: bold;
            text-shadow: 0 0 5px rgba(0,0,0,0.8);
        }

        .exp-bar-container {
            width: 100%;
            height: 4px;
            background: rgba(255,255,255,0.1);
            border-radius: 2px;
            overflow: hidden;
        }

        #exp-fill {
            width: 0%;
            height: 100%;
            background: var(--exp-color);
            transition: width 0.3s;
        }

        .modal {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 85%;
            max-width: 350px;
            background: rgba(10, 10, 25, 0.95);
            border: 1px solid var(--primary-color);
            padding: 25px;
            display: none;
            flex-direction: column;
            border-radius: 15px;
            z-index: 100;
            pointer-events: auto;
        }

        .skill-option {
            padding: 15px;
            margin: 8px 0;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 10px;
            cursor: pointer;
        }

        .skill-option:active {
            background: var(--primary-color);
            color: #000;
        }

        .skill-title { font-weight: bold; color: var(--primary-color); display: block; margin-bottom: 4px; }
        .skill-desc { font-size: 12px; opacity: 0.8; }

        #boss-ui {
            position: absolute;
            bottom: 40px;
            left: 50%;
            transform: translateX(-50%);
            width: 70%;
            display: none;
        }

        .boss-hp-bar { width: 100%; height: 8px; background: #222; border-radius: 4px; overflow: hidden; border: 1px solid rgba(255,255,255,0.2); }
        #boss-hp-fill { width: 100%; height: 100%; background: var(--boss-color); }

        #overlay {
            position: absolute;
            inset: 0;
            background: rgba(0,0,0,0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 200;
            text-align: center;
        }

        h1 {
            font-size: 36px;
            margin-bottom: 10px;
            background: linear-gradient(to bottom, #fff, var(--primary-color));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .btn {
            background: none;
            color: var(--primary-color);
            border: 2px solid var(--primary-color);
            padding: 15px 40px;
            font-size: 18px;
            border-radius: 50px;
            margin-top: 20px;
            font-weight: bold;
            cursor: pointer;
        }

        .btn:active { background: var(--primary-color); color: #000; }

        #birthday-msg {
            position: absolute;
            top: 45%;
            left: 0;
            width: 100%;
            text-align: center;
            font-size: 32px;
            font-weight: bold;
            color: var(--secondary-color);
            display: none;
            pointer-events: none;
            animation: pulse 1.5s infinite ease-in-out;
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); opacity: 0.8; }
            50% { transform: scale(1.1); opacity: 1; }
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>
    
    <div id="ui-layer">
        <div class="stats-top">
            <span>等級: <span id="player-lv">1</span></span>
            <span>得分: <span id="score">0</span></span>
            <span>生命: <span id="health">100</span>%</span>
        </div>
        <div class="exp-bar-container">
            <div id="exp-fill"></div>
        </div>
        
        <div id="boss-ui">
            <div style="text-align: center; color: var(--boss-color); font-size: 12px; font-weight: bold;">BOSS 警告</div>
            <div class="boss-hp-bar"><div id="boss-hp-fill"></div></div>
        </div>
    </div>

    <div id="birthday-msg">Happy Birthday Sam! 🎂</div>

    <div id="level-up-modal" class="modal">
        <h2 style="color: var(--exp-color); margin: 0 0 15px 0; text-align: center;">科技升級</h2>
        <div class="skill-option" onclick="chooseSkill('multishot')">
            <span class="skill-title">彈道增強 (+1)</span>
            <span class="skill-desc">增加子彈發射數量，覆蓋更廣。</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('firerate')">
            <span class="skill-title">射速提升 (+15%)</span>
            <span class="skill-desc">減少射擊間隔，火力更密集。</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('damage')">
            <span class="skill-title">彈藥強化 (+50%)</span>
            <span class="skill-desc">大幅提升單發子彈的破壞力。</span>
        </div>
    </div>

    <div id="overlay">
        <h1 id="main-title">星際守護者</h1>
        <button id="startBtn" class="btn">開始戰鬥</button>
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
    const startBtn = document.getElementById('startBtn');

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

    const skills = {
        bullets: 1,
        fireInterval: 14,
        damage: 1
    };

    const player = {
        x: 0, y: 0,
        radius: 18,
        targetX: 0, targetY: 0,
        lerp: 0.15
    };

    let stars = [], projectiles = [], enemyProjectiles = [], enemies = [], particles = [], gems = [], boss = null;

    function resize() {
        const w = window.innerWidth;
        const h = window.innerHeight;
        canvas.width = w;
        canvas.height = h;
        
        if (!gameActive) {
            player.x = w / 2;
            player.y = h * 0.8;
            player.targetX = player.x;
            player.targetY = player.y;
        }
        
        stars = [];
        for(let i=0; i<60; i++) {
            stars.push({
                x: Math.random() * w,
                y: Math.random() * h,
                size: Math.random() * 2,
                speed: Math.random() * 1.5 + 0.5
            });
        }
    }

    window.addEventListener('resize', resize);
    window.addEventListener('orientationchange', () => setTimeout(resize, 200));

    function handleInput(e) {
        if (!gameActive || isPaused) return;
        const rect = canvas.getBoundingClientRect();
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        
        player.targetX = Math.max(20, Math.min(canvas.width - 20, clientX - rect.left));
        player.targetY = Math.max(20, Math.min(canvas.height - 20, clientY - rect.top));
    }

    canvas.addEventListener('mousemove', handleInput);
    canvas.addEventListener('touchmove', (e) => { e.preventDefault(); handleInput(e); }, {passive: false});

    startBtn.addEventListener('click', () => {
        resetGame();
        gameActive = true;
        overlay.style.display = 'none';
        animate();
    });

    function resetGame() {
        resize();
        score = 0; health = 100; playerLv = 1; currentExp = 0; nextExp = 100;
        bossLevel = 0; frames = 0; bossActive = false; birthdayMode = false;
        skills.bullets = 1; skills.fireInterval = 14; skills.damage = 1;
        projectiles = []; enemyProjectiles = []; enemies = []; particles = []; gems = []; boss = null;
        scoreEl.innerText = "0";
        healthEl.innerText = "100";
        playerLvEl.innerText = "1";
        expFill.style.width = "0%";
        birthdayMsg.style.display = 'none';
        bossUi.style.display = 'none';
    }

    function createExplosion(x, y, color, count = 12) {
        for(let i=0; i<count; i++) {
            particles.push({
                x, y,
                vx: (Math.random() - 0.5) * 8,
                vy: (Math.random() - 0.5) * 8,
                radius: Math.random() * 2 + 1,
                color,
                alpha: 1,
                decay: Math.random() * 0.02 + 0.02
            });
        }
    }

    window.chooseSkill = function(type) {
        if (type === 'multishot') skills.bullets++;
        if (type === 'firerate') skills.fireInterval = Math.max(4, skills.fireInterval * 0.85);
        if (type === 'damage') skills.damage += 0.5;
        lvModal.style.display = 'none';
        isPaused = false;
    };

    function update() {
        if (!gameActive || isPaused) return;
        frames++;

        if (score >= 10000 && !birthdayMode) {
            birthdayMode = true;
            birthdayMsg.style.display = 'block';
            enemies = []; boss = null;
        }

        stars.forEach(s => {
            s.y += s.speed;
            if (s.y > canvas.height) s.y = 0;
        });

        player.x += (player.targetX - player.x) * player.lerp;
        player.y += (player.targetY - player.y) * player.lerp;

        if (!bossActive && !birthdayMode && frames % Math.max(15, 45 - Math.floor(score/1000)*2) === 0) {
            const size = Math.random() * 20 + 20;
            enemies.push({
                x: Math.random() * (canvas.width - size),
                y: -size, w: size, h: size,
                hp: 1 + Math.floor(score/2000),
                speed: 2 + Math.random() * 2,
                color: `hsl(${Math.random() * 60 + 180}, 70%, 50%)`
            });
        }

        if (!bossActive && !birthdayMode && score > (bossLevel + 1) * 3000) {
            bossActive = true;
            bossLevel++;
            bossUi.style.display = 'block';
            const hp = 100 + bossLevel * 150;
            boss = {
                x: canvas.width / 2 - 40, y: -100, w: 80, h: 50,
                hp, maxHp: hp, targetY: 80, speed: 2, dir: 1,
                fireTimer: 0
            };
        }

        if (frames % Math.floor(skills.fireInterval) === 0) {
            const count = skills.bullets;
            const step = 0.2;
            const start = -((count - 1) * step) / 2;
            for(let i=0; i<count; i++) {
                projectiles.push({
                    x: player.x, y: player.y - 10,
                    vx: Math.sin(start + i * step) * 10,
                    vy: -12,
                    damage: skills.damage
                });
            }
        }

        projectiles.forEach((p, i) => {
            p.x += p.vx; p.y += p.vy;
            if (p.y < -20) projectiles.splice(i, 1);
        });

        enemyProjectiles.forEach((p, i) => {
            p.x += p.vx; p.y += p.vy;
            if (Math.hypot(p.x - player.x, p.y - player.y) < player.radius + 5) {
                health -= 10;
                createExplosion(player.x, player.y, '#f00');
                enemyProjectiles.splice(i, 1);
                if (health <= 0) gameOver();
            }
            if (p.y > canvas.height) enemyProjectiles.splice(i, 1);
        });

        enemies.forEach((e, i) => {
            e.y += e.speed;
            projectiles.forEach((p, pi) => {
                if (p.x > e.x && p.x < e.x + e.w && p.y > e.y && p.y < e.y + e.h) {
                    e.hp -= p.damage;
                    projectiles.splice(pi, 1);
                    if (e.hp <= 0) {
                        createExplosion(e.x + e.w/2, e.y + e.h/2, e.color);
                        gems.push({x: e.x + e.w/2, y: e.y + e.h/2, v: 20});
                        score += 100;
                        enemies.splice(i, 1);
                    }
                }
            });
            if (Math.hypot(e.x + e.w/2 - player.x, e.y + e.h/2 - player.y) < player.radius + e.w/2) {
                health -= 15;
                createExplosion(player.x, player.y, '#f00');
                enemies.splice(i, 1);
                if (health <= 0) gameOver();
            }
            if (e.y > canvas.height) enemies.splice(i, 1);
        });

        gems.forEach((g, i) => {
            g.y += 2;
            const d = Math.hypot(g.x - player.x, g.y - player.y);
            if (d < 100) { g.x += (player.x - g.x)*0.15; g.y += (player.y - g.y)*0.15; }
            if (d < 20) {
                currentExp += g.v;
                if (currentExp >= nextExp) {
                    currentExp -= nextExp;
                    nextExp = Math.floor(nextExp * 1.3);
                    playerLv++;
                    isPaused = true;
                    lvModal.style.display = 'flex';
                }
                gems.splice(i, 1);
            }
        });

        if (boss) {
            if (boss.y < boss.targetY) boss.y += 1;
            else {
                boss.x += boss.speed * boss.dir;
                if (boss.x < 10 || boss.x > canvas.width - boss.w - 10) boss.dir *= -1;
                boss.fireTimer++;
                if (boss.fireTimer > 40) {
                    boss.fireTimer = 0;
                    for(let a=-1; a<=1; a+=0.5) {
                        enemyProjectiles.push({x: boss.x + boss.w/2, y: boss.y + boss.h, vx: a*2, vy: 4});
                    }
                }
            }
            projectiles.forEach((p, pi) => {
                if (p.x > boss.x && p.x < boss.x + boss.w && p.y > boss.y && p.y < boss.y + boss.h) {
                    boss.hp -= p.damage;
                    projectiles.splice(pi, 1);
                    if (boss.hp <= 0) {
                        createExplosion(boss.x + boss.w/2, boss.y + boss.h/2, '#f00', 30);
                        score += 2000;
                        boss = null; bossActive = false;
                        bossUi.style.display = 'none';
                    }
                }
            });
        }

        particles.forEach((p, i) => {
            p.x += p.vx; p.y += p.vy; p.alpha -= p.decay;
            if (p.alpha <= 0) particles.splice(i, 1);
        });

        scoreEl.innerText = score;
        healthEl.innerText = Math.max(0, health);
        playerLvEl.innerText = playerLv;
        expFill.style.width = `${(currentExp/nextExp)*100}%`;
        if (boss) bossHpFill.style.width = `${(boss.hp/boss.maxHp)*100}%`;
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = '#fff';
        stars.forEach(s => {
            ctx.globalAlpha = 0.4;
            ctx.fillRect(s.x, s.y, s.size, s.size);
        });
        ctx.globalAlpha = 1;

        ctx.save();
        ctx.translate(player.x, player.y);
        ctx.fillStyle = "#00f2ff";
        ctx.beginPath();
        ctx.moveTo(0, -player.radius);
        ctx.lineTo(player.radius, player.radius);
        ctx.lineTo(0, player.radius*0.6);
        ctx.lineTo(-player.radius, player.radius);
        ctx.closePath();
        ctx.fill();
        ctx.restore();

        ctx.fillStyle = "#fff";
        projectiles.forEach(p => {
            ctx.beginPath();
            ctx.arc(p.x, p.y, 3, 0, Math.PI*2);
            ctx.fill();
        });

        ctx.fillStyle = "#ff0000";
        enemyProjectiles.forEach(p => {
            ctx.beginPath();
            ctx.arc(p.x, p.y, 4, 0, Math.PI*2);
            ctx.fill();
        });

        enemies.forEach(e => {
            ctx.fillStyle = e.color;
            ctx.fillRect(e.x, e.y, e.w, e.h);
        });

        ctx.fillStyle = "#f1c40f";
        gems.forEach(g => {
            ctx.beginPath();
            ctx.arc(g.x, g.y, 3.5, 0, Math.PI*2);
            ctx.fill();
        });

        if (boss) {
            ctx.fillStyle = "#ff3e3e";
            ctx.fillRect(boss.x, boss.y, boss.w, boss.h);
        }

        particles.forEach(p => {
            ctx.globalAlpha = p.alpha;
            ctx.fillStyle = p.color;
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.radius, 0, Math.PI*2);
            ctx.fill();
        });
        ctx.globalAlpha = 1;
    }

    function gameOver() {
        gameActive = false;
        overlay.style.display = 'flex';
    }

    function animate() {
        if (!gameActive) return;
        update();
        draw();
        requestAnimationFrame(animate);
    }

    resize();
</script>
</body>
</html>
