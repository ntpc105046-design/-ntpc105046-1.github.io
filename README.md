<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>星際守護者</title>
    <style>
        :root {
            --primary-color: #00f2ff;
            --secondary-color: #ff00ea;
            --health-color: #2ecc71;
            --boss-color: #ff3e3e;
            --exp-color: #f1c40f;
            --bg-color: #0b0b1a;
        }

        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: white;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            touch-action: none;
        }

        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }

        canvas {
            display: block;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 20px;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .stats-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .stat-box {
            background: rgba(255, 255, 255, 0.1);
            padding: 8px 15px;
            border-radius: 20px;
            backdrop-filter: blur(5px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            font-weight: bold;
        }

        .exp-bar-container {
            width: 200px;
            height: 8px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 4px;
            overflow: hidden;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        #exp-fill {
            width: 0%;
            height: 100%;
            background: var(--exp-color);
            box-shadow: 0 0 10px var(--exp-color);
            transition: width 0.3s ease;
        }

        .modal {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(10, 10, 25, 0.95);
            border: 2px solid var(--primary-color);
            padding: 30px;
            border-radius: 20px;
            display: none;
            flex-direction: column;
            gap: 15px;
            min-width: 300px;
            box-shadow: 0 0 30px rgba(0, 242, 255, 0.3);
            z-index: 100;
        }

        .skill-option {
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 12px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .skill-option:hover {
            background: rgba(0, 242, 255, 0.1);
            border-color: var(--primary-color);
            transform: translateY(-2px);
        }

        .skill-name { color: var(--primary-color); font-weight: bold; margin-bottom: 5px; display: block; }
        .skill-desc { font-size: 0.9em; opacity: 0.8; }

        #boss-ui {
            position: absolute;
            bottom: 50px;
            left: 50%;
            transform: translateX(-50%);
            width: 60%;
            display: none;
        }

        .boss-name { text-align: center; color: var(--boss-color); font-weight: bold; margin-bottom: 5px; text-transform: uppercase; letter-spacing: 2px; }
        .boss-hp-bar { width: 100%; height: 12px; background: rgba(255, 255, 255, 0.1); border-radius: 6px; overflow: hidden; border: 1px solid var(--boss-color); }
        #boss-hp-fill { width: 100%; height: 100%; background: var(--boss-color); box-shadow: 0 0 15px var(--boss-color); }

        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 200;
        }

        h1 { font-size: 4em; margin-bottom: 20px; color: var(--primary-color); text-shadow: 0 0 20px var(--primary-color); }
        .start-btn {
            padding: 15px 50px;
            font-size: 1.5em;
            background: transparent;
            color: var(--primary-color);
            border: 2px solid var(--primary-color);
            border-radius: 30px;
            cursor: pointer;
            transition: all 0.3s;
        }

        .start-btn:hover { background: var(--primary-color); color: black; box-shadow: 0 0 30px var(--primary-color); }

        #birthday-msg {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 5em;
            font-weight: bold;
            color: var(--secondary-color);
            text-shadow: 0 0 30px var(--secondary-color);
            display: none;
            z-index: 50;
            white-space: nowrap;
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>
    
    <div id="ui-layer">
        <div class="stats-row">
            <div class="stat-box">等級: <span id="player-lv">1</span></div>
            <div class="stat-box">得分: <span id="score">0</span></div>
            <div class="stat-box">生命: <span id="health">100</span>%</div>
        </div>
        <div class="exp-bar-container">
            <div id="exp-fill"></div>
        </div>
        
        <div id="boss-ui">
            <div class="boss-name">警告: 敵方戰艦</div>
            <div class="boss-hp-bar">
                <div id="boss-hp-fill"></div>
            </div>
        </div>
    </div>

    <div id="birthday-msg">Happy Birthday Sam! 🎂</div>

    <div id="level-up-modal" class="modal">
        <h2 style="margin: 0 0 10px 0; text-align: center;">科技升級</h2>
        <div class="skill-option" onclick="chooseSkill('multishot')">
            <span class="skill-name">多重彈道 (+1)</span>
            <span class="skill-desc">增加每次射擊的子彈數量。</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('firerate')">
            <span class="skill-name">射速強化 (-15%)</span>
            <span class="skill-desc">提升武器射擊頻率。</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('damage')">
            <span class="skill-name">彈藥強化 (+50%)</span>
            <span class="skill-desc">提升單發子彈的傷害力。</span>
        </div>
    </div>

    <div id="overlay">
        <h1>星際守護者</h1>
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

    const skills = {
        bullets: 1,
        fireInterval: 14,
        damage: 1
    };

    const player = {
        x: 0,
        y: 0,
        radius: 20,
        targetX: 0,
        targetY: 0
    };

    let stars = [];
    let projectiles = [];
    let enemyProjectiles = [];
    let enemies = [];
    let particles = [];
    let gems = [];
    let boss = null;

    function init() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        
        player.x = canvas.width / 2;
        player.y = canvas.height * 0.8;
        player.targetX = player.x;
        player.targetY = player.y;

        stars = [];
        for(let i=0; i<100; i++) {
            stars.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                size: Math.random() * 2,
                speed: Math.random() * 3 + 1
            });
        }
    }

    window.addEventListener('mousemove', (e) => {
        if (!gameActive || isPaused) return;
        player.targetX = e.clientX;
        player.targetY = e.clientY;
    });

    window.addEventListener('touchmove', (e) => {
        if (!gameActive || isPaused) return;
        player.targetX = e.touches[0].clientX;
        player.targetY = e.touches[0].clientY;
    });

    function createExplosion(x, y, color, count = 15) {
        for(let i=0; i<count; i++) {
            particles.push({
                x, y,
                vx: (Math.random() - 0.5) * 10,
                vy: (Math.random() - 0.5) * 10,
                radius: Math.random() * 3,
                color,
                alpha: 1
            });
        }
    }

    function chooseSkill(type) {
        if (type === 'multishot') skills.bullets++;
        if (type === 'firerate') skills.fireInterval *= 0.85;
        if (type === 'damage') skills.damage += 0.5;
        
        lvModal.style.display = 'none';
        isPaused = false;
    }

    function startGame() {
        overlay.style.display = 'none';
        gameActive = true;
        score = 0;
        health = 100;
        playerLv = 1;
        currentExp = 0;
        nextExp = 100;
        bossLevel = 0;
        frames = 0;
        bossActive = false;
        birthdayMode = false;
        skills.bullets = 1;
        skills.fireInterval = 14;
        skills.damage = 1;
        projectiles = [];
        enemyProjectiles = [];
        enemies = [];
        gems = [];
        boss = null;
        updateUI();
        animate();
    }

    function updateUI() {
        scoreEl.innerText = Math.floor(score);
        healthEl.innerText = Math.max(0, Math.floor(health));
        playerLvEl.innerText = playerLv;
        expFill.style.width = (currentExp / nextExp * 100) + '%';
    }

    function update() {
        if (!gameActive || isPaused) return;

        frames++;

        if (score >= 10000 && !birthdayMode) {
            birthdayMode = true;
            birthdayMsg.style.display = 'block';
            enemies = [];
            boss = null;
        }

        stars.forEach(s => {
            s.y += s.speed;
            if (s.y > canvas.height) s.y = 0;
        });

        player.x += (player.targetX - player.x) * 0.1;
        player.y += (player.targetY - player.y) * 0.1;

        if (frames % Math.floor(skills.fireInterval) === 0) {
            const spread = 0.3;
            const startAngle = -((skills.bullets - 1) * spread) / 2;
            for(let i=0; i<skills.bullets; i++) {
                projectiles.push({
                    x: player.x,
                    y: player.y - 20,
                    vx: Math.sin(startAngle + i * spread) * 12,
                    vy: -15,
                    damage: skills.damage
                });
            }
        }

        projectiles.forEach((p, i) => {
            p.x += p.vx;
            p.y += p.vy;
            if (p.y < -20) projectiles.splice(i, 1);
        });

        if (!bossActive && !birthdayMode && frames % 40 === 0) {
            const size = Math.random() * 30 + 20;
            enemies.push({
                x: Math.random() * (canvas.width - size),
                y: -size,
                w: size,
                h: size,
                hp: 1 + Math.floor(score / 2000),
                speed: 3 + Math.random() * 2,
                color: `hsl(${Math.random() * 50 + 190}, 70%, 50%)`
            });
        }

        if (!bossActive && !birthdayMode && score > (bossLevel + 1) * 3000) {
            bossActive = true;
            bossLevel++;
            bossUi.style.display = 'block';
            const hp = 100 + bossLevel * 100;
            boss = {
                x: canvas.width / 2 - 50,
                y: -100,
                w: 100,
                h: 60,
                hp,
                maxHp: hp,
                targetY: 100,
                speed: 3,
                dir: 1,
                fireTimer: 0
            };
        }

        enemies.forEach((e, i) => {
            e.y += e.speed;
            if (e.y > canvas.height) enemies.splice(i, 1);

            projectiles.forEach((p, pi) => {
                if (p.x > e.x && p.x < e.x + e.w && p.y > e.y && p.y < e.y + e.h) {
                    e.hp -= p.damage;
                    projectiles.splice(pi, 1);
                    if (e.hp <= 0) {
                        createExplosion(e.x + e.w/2, e.y + e.h/2, e.color);
                        gems.push({ x: e.x + e.w/2, y: e.y + e.h/2 });
                        score += 100;
                        enemies.splice(i, 1);
                    }
                }
            });

            if (Math.hypot(e.x + e.w/2 - player.x, e.y + e.h/2 - player.y) < player.radius + e.w/2) {
                health -= 20;
                enemies.splice(i, 1);
                createExplosion(player.x, player.y, '#ff0000');
                if (health <= 0) gameOver();
            }
        });

        gems.forEach((g, i) => {
            const dist = Math.hypot(g.x - player.x, g.y - player.y);
            if (dist < 150) {
                g.x += (player.x - g.x) * 0.1;
                g.y += (player.y - g.y) * 0.1;
            } else {
                g.y += 3;
            }

            if (dist < 30) {
                currentExp += 25;
                if (currentExp >= nextExp) {
                    currentExp = 0;
                    nextExp *= 1.4;
                    playerLv++;
                    isPaused = true;
                    lvModal.style.display = 'flex';
                }
                gems.splice(i, 1);
            }
        });

        if (boss) {
            if (boss.y < boss.targetY) boss.y += 2;
            else {
                boss.x += boss.speed * boss.dir;
                if (boss.x < 50 || boss.x > canvas.width - 150) boss.dir *= -1;
                
                boss.fireTimer++;
                if (boss.fireTimer > 50) {
                    boss.fireTimer = 0;
                    for(let i=-2; i<=2; i++) {
                        enemyProjectiles.push({
                            x: boss.x + boss.w/2,
                            y: boss.y + boss.h,
                            vx: i * 2,
                            vy: 5
                        });
                    }
                }
            }

            projectiles.forEach((p, pi) => {
                if (p.x > boss.x && p.x < boss.x + boss.w && p.y > boss.y && p.y < boss.y + boss.h) {
                    boss.hp -= p.damage;
                    projectiles.splice(pi, 1);
                    if (boss.hp <= 0) {
                        createExplosion(boss.x + boss.w/2, boss.y + boss.h/2, '#ff3e3e', 40);
                        score += 2000;
                        boss = null;
                        bossActive = false;
                        bossUi.style.display = 'none';
                    }
                }
            });

            bossHpFill.style.width = (boss.hp / boss.maxHp * 100) + '%';
        }

        enemyProjectiles.forEach((p, i) => {
            p.x += p.vx;
            p.y += p.vy;
            if (Math.hypot(p.x - player.x, p.y - player.y) < player.radius + 5) {
                health -= 10;
                enemyProjectiles.splice(i, 1);
                createExplosion(player.x, player.y, '#ff0000');
                if (health <= 0) gameOver();
            }
            if (p.y > canvas.height) enemyProjectiles.splice(i, 1);
        });

        particles.forEach((p, i) => {
            p.x += p.vx;
            p.y += p.vy;
            p.alpha -= 0.02;
            if (p.alpha <= 0) particles.splice(i, 1);
        });

        updateUI();
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        stars.forEach(s => {
            ctx.fillStyle = 'rgba(255, 255, 255, 0.5)';
            ctx.fillRect(s.x, s.y, s.size, s.size);
        });

        ctx.save();
        ctx.translate(player.x, player.y);
        ctx.fillStyle = '#00f2ff';
        ctx.beginPath();
        ctx.moveTo(0, -25);
        ctx.lineTo(20, 15);
        ctx.lineTo(0, 5);
        ctx.lineTo(-20, 15);
        ctx.closePath();
        ctx.fill();
        ctx.restore();

        projectiles.forEach(p => {
            ctx.fillStyle = '#fff';
            ctx.beginPath();
            ctx.arc(p.x, p.y, 3, 0, Math.PI * 2);
            ctx.fill();
        });

        enemies.forEach(e => {
            ctx.fillStyle = e.color;
            ctx.fillRect(e.x, e.y, e.w, e.h);
        });

        gems.forEach(g => {
            ctx.fillStyle = '#f1c40f';
            ctx.beginPath();
            ctx.arc(g.x, g.y, 4, 0, Math.PI * 2);
            ctx.fill();
        });

        if (boss) {
            ctx.fillStyle = '#ff3e3e';
            ctx.fillRect(boss.x, boss.y, boss.w, boss.h);
        }

        enemyProjectiles.forEach(p => {
            ctx.fillStyle = '#ff3e3e';
            ctx.beginPath();
            ctx.arc(p.x, p.y, 5, 0, Math.PI * 2);
            ctx.fill();
        });

        particles.forEach(p => {
            ctx.globalAlpha = p.alpha;
            ctx.fillStyle = p.color;
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
            ctx.fill();
        });
        ctx.globalAlpha = 1;
    }

    function gameOver() {
        gameActive = false;
        overlay.style.display = 'flex';
        overlay.querySelector('h1').innerText = '任務失敗';
    }

    function animate() {
        if (!gameActive) return;
        update();
        draw();
        requestAnimationFrame(animate);
    }

    init();
</script>
</body>
</html>
