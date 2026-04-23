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
            --bg-color: #050505;
        }

        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: var(--bg-color);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            touch-action: none;
        }

        #game-container {
            position: relative;
            box-shadow: 0 0 50px rgba(0, 242, 255, 0.2);
            border: 2px solid #333;
            background: #000;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        canvas {
            display: block;
            background: radial-gradient(circle at center, #111 0%, #000 100%);
            max-width: 100%;
            max-height: 100%;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            padding: 15px;
            box-sizing: border-box;
        }

        .stats-top {
            display: flex;
            justify-content: space-between;
            font-size: 14px;
            text-transform: uppercase;
            letter-spacing: 1px;
            text-shadow: 0 0 10px rgba(0, 242, 255, 0.5);
        }

        .exp-bar-container {
            width: 100%;
            height: 6px;
            background: #222;
            margin-top: 10px;
            border-radius: 3px;
            overflow: hidden;
        }

        #exp-fill {
            width: 0%;
            height: 100%;
            background: var(--exp-color);
            transition: width 0.3s;
            box-shadow: 0 0 10px var(--exp-color);
        }

        #level-up-modal {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 85%;
            max-width: 400px;
            background: rgba(10, 10, 20, 0.95);
            border: 2px solid var(--primary-color);
            padding: 20px;
            display: none;
            flex-direction: column;
            align-items: center;
            z-index: 100;
            pointer-events: auto;
            border-radius: 10px;
            box-shadow: 0 0 30px var(--primary-color);
        }

        .skill-option {
            width: 100%;
            padding: 12px;
            margin: 8px 0;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid #444;
            color: white;
            cursor: pointer;
            transition: all 0.2s;
            text-align: left;
            border-radius: 5px;
            box-sizing: border-box;
        }

        .skill-option:active {
            background: rgba(0, 242, 255, 0.3);
            transform: scale(0.98);
        }

        .skill-title {
            font-weight: bold;
            color: var(--primary-color);
            display: block;
            margin-bottom: 3px;
            font-size: 16px;
        }

        .skill-desc {
            font-size: 12px;
            color: #ccc;
        }

        #boss-ui {
            position: absolute;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            width: 80%;
            display: none;
        }

        .boss-hp-bar {
            width: 100%;
            height: 10px;
            background: #333;
            border: 1px solid #fff;
            margin-top: 5px;
        }

        #boss-hp-fill {
            width: 100%;
            height: 100%;
            background: var(--boss-color);
            transition: width 0.2s;
        }

        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }

        #birthday-msg {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 28px;
            font-weight: bold;
            color: #ff00ea;
            text-shadow: 0 0 20px #ff00ea, 0 0 40px #00f2ff;
            display: none;
            text-align: center;
            z-index: 50;
            white-space: nowrap;
            pointer-events: none;
            animation: pulse 1s infinite alternate;
        }

        @keyframes pulse {
            from { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
            to { transform: translate(-50%, -50%) scale(1.1); opacity: 1; }
        }

        h1 {
            font-size: 32px;
            margin-bottom: 30px;
            background: linear-gradient(to bottom, var(--primary-color), var(--secondary-color));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-align: center;
        }

        .btn {
            padding: 12px 35px;
            font-size: 18px;
            background: transparent;
            color: var(--primary-color);
            border: 2px solid var(--primary-color);
            cursor: pointer;
            transition: all 0.3s;
            text-transform: uppercase;
            letter-spacing: 2px;
            pointer-events: auto;
            border-radius: 5px;
        }

        .btn:active {
            background: var(--primary-color);
            color: #000;
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>
    
    <div id="ui-layer">
        <div class="stats-top">
            <div>等級: <span id="player-lv">1</span></div>
            <div>得分: <span id="score">0</span></div>
            <div>生命: <span id="health">100</span>%</div>
        </div>
        <div class="exp-bar-container">
            <div id="exp-fill"></div>
        </div>
        
        <div id="boss-ui">
            <div id="boss-label" style="text-align: center; color: var(--boss-color); font-size: 12px; font-weight: bold;">BOSS 警告</div>
            <div class="boss-hp-bar"><div id="boss-hp-fill"></div></div>
        </div>

        <div id="birthday-msg">Happy Birthday Sam! 🎂</div>
    </div>

    <div id="level-up-modal">
        <h2 style="color: var(--exp-color); margin-top: 0; font-size: 24px;">科技升級</h2>
        <p style="font-size: 14px; color: #aaa; margin-bottom: 15px;">選擇一項強化項目</p>
        <div class="skill-option" onclick="chooseSkill('multishot')">
            <span class="skill-title">彈道增強 (+1)</span>
            <span class="skill-desc">增加子彈發射數量，擴大打擊範圍。</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('firerate')">
            <span class="skill-title">射速提升 (+15%)</span>
            <span class="skill-desc">提高每秒射彈數，更密集的火力。</span>
        </div>
        <div class="skill-option" onclick="chooseSkill('damage')">
            <span class="skill-title">彈藥強化 (+50%)</span>
            <span class="skill-desc">提升每發子彈造成的基礎傷害。</span>
        </div>
    </div>

    <div id="overlay">
        <h1 id="title">星際守護者</h1>
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
    const titleEl = document.getElementById('title');

    // 遊戲狀態
    let gameActive = false;
    let isPaused = false;
    let score = 0;
    let health = 100;
    let bossLevel = 0;
    let frames = 0;
    let bossActive = false;
    let birthdayMode = false;

    // 玩家屬性
    let playerLv = 1;
    let currentExp = 0;
    let nextExp = 100;
    
    const skills = {
        bullets: 1,      
        fireInterval: 12, 
        bulletDamage: 1   
    };

    function resize() {
        const container = document.getElementById('game-container');
        const containerW = container.clientWidth;
        const containerH = container.clientHeight;
        
        const targetRatio = 9/16;
        let w, h;
        
        if (containerW / containerH > targetRatio) {
            h = containerH;
            w = h * targetRatio;
        } else {
            w = containerW;
            h = w / targetRatio;
        }
        
        canvas.width = w;
        canvas.height = h;
        
        if (player) {
            player.x = canvas.width / 2;
            player.y = canvas.height - 80;
            player.targetX = player.x;
            player.targetY = player.y;
        }
    }
    window.addEventListener('resize', resize);

    const player = {
        x: 0,
        y: 0,
        w: 35,
        h: 35,
        color: '#00f2ff',
        targetX: 0,
        targetY: 0,
        speed: 0.2 
    };

    let projectiles = [];
    let enemyProjectiles = []; 
    let enemies = [];
    let particles = [];
    let stars = [];
    let experienceGems = [];
    let boss = null;

    function initStars() {
        stars = [];
        for(let i=0; i<80; i++) {
            stars.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                size: Math.random() * 2,
                speed: Math.random() * 2 + 1
            });
        }
    }

    function updatePointerPos(e) {
        if (isPaused || !gameActive) return;
        const rect = canvas.getBoundingClientRect();
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        
        let newX = clientX - rect.left;
        let newY = clientY - rect.top;
        
        player.targetX = Math.max(20, Math.min(canvas.width - 20, newX));
        player.targetY = Math.max(20, Math.min(canvas.height - 20, newY));
    }

    canvas.addEventListener('mousemove', updatePointerPos);
    canvas.addEventListener('touchstart', updatePointerPos);
    canvas.addEventListener('touchmove', (e) => {
        if (gameActive) e.preventDefault();
        updatePointerPos(e);
    }, { passive: false });

    startBtn.addEventListener('click', initGame);

    function initGame() {
        resize();
        initStars();
        score = 0;
        health = 100;
        bossLevel = 0;
        playerLv = 1;
        currentExp = 0;
        nextExp = 100;
        skills.bullets = 1;
        skills.fireInterval = 12;
        skills.bulletDamage = 1;
        projectiles = [];
        enemyProjectiles = [];
        enemies = [];
        particles = [];
        experienceGems = [];
        boss = null;
        bossActive = false;
        birthdayMode = false;
        birthdayMsg.style.display = 'none';
        gameActive = true;
        isPaused = false;
        overlay.style.display = 'none';
        lvModal.style.display = 'none';
        bossUi.style.display = 'none';
        scoreEl.innerText = score;
        healthEl.innerText = health;
        playerLvEl.innerText = playerLv;
        updateExpBar();
        animate();
    }

    function updateExpBar() {
        const percent = Math.min(100, (currentExp / nextExp) * 100);
        expFill.style.width = `${percent}%`;
    }

    function gainExp(amount) {
        currentExp += amount;
        if (currentExp >= nextExp) {
            levelUp();
        }
        updateExpBar();
    }

    function levelUp() {
        currentExp -= nextExp;
        playerLv++;
        nextExp = Math.floor(nextExp * 1.3);
        playerLvEl.innerText = playerLv;
        isPaused = true;
        lvModal.style.display = 'flex';
    }

    window.chooseSkill = function(type) {
        if (type === 'multishot') skills.bullets += 1;
        else if (type === 'firerate') skills.fireInterval = Math.max(3, skills.fireInterval * 0.85);
        else if (type === 'damage') skills.bulletDamage += 0.5;
        
        lvModal.style.display = 'none';
        isPaused = false;
        frames = 0;
    };

    function createExplosion(x, y, color, count = 15, speed = 10) {
        for(let i=0; i<count; i++) {
            particles.push({
                x: x,
                y: y,
                vx: (Math.random() - 0.5) * speed,
                vy: (Math.random() - 0.5) * speed,
                radius: Math.random() * 3 + 1,
                color: color,
                alpha: 1,
                decay: Math.random() * 0.01 + 0.01
            });
        }
    }

    function spawnFirework() {
        const x = Math.random() * canvas.width;
        const y = Math.random() * (canvas.height * 0.6);
        const colors = ['#ff00ea', '#00f2ff', '#f1c40f', '#2ecc71', '#ffffff'];
        createExplosion(x, y, colors[Math.floor(Math.random() * colors.length)], 40, 15);
    }

    function spawnEnemy() {
        if (bossActive || isPaused || birthdayMode) return;
        const size = Math.random() * 25 + 20;
        enemies.push({
            x: Math.random() * (canvas.width - size),
            y: -size,
            w: size,
            h: size,
            speed: (Math.random() * 1.2 + 1.2 + (bossLevel * 0.3)),
            hp: Math.ceil(size / 12) + (bossLevel * 2), // 隨 Boss 等級增加敵人血量
            expValue: 25,
            color: `hsl(${Math.random() * 40 + 260}, 80%, 60%)`
        });
    }

    function spawnExperience(x, y, value) {
        experienceGems.push({
            x: x, y: y, size: 6, value: value || 25, vy: 1.5
        });
    }

    function spawnBoss() {
        bossActive = true;
        bossUi.style.display = 'block';
        bossLevel++;
        const maxHp = 500 + (bossLevel * 400);
        boss = {
            x: canvas.width / 2 - 50,
            y: -120,
            targetY: 80,
            w: 100, h: 60,
            hp: maxHp, maxHp: maxHp,
            color: '#ff3e3e',
            dir: 1, speed: 1.5 + (bossLevel * 0.3),
            attackTimer: 0
        };
    }

    function update() {
        if (isPaused || !gameActive) return;
        frames++;

        // 勝利條件 (10000 分)
        if (score >= 10000 && !birthdayMode) {
            birthdayMode = true;
            birthdayMsg.style.display = 'block';
            enemies = [];
            enemyProjectiles = [];
            boss = null;
            bossActive = false;
            bossUi.style.display = 'none';
        }

        if (birthdayMode && frames % 20 === 0) {
            spawnFirework();
        }

        stars.forEach(star => {
            star.y += star.speed;
            if(star.y > canvas.height) star.y = 0;
        });

        if (!birthdayMode) {
            // Boss 門檻
            const nextBossThreshold = (bossLevel + 1) * 5000; 
            if (score >= nextBossThreshold && !bossActive) {
                spawnBoss();
            }

            // 確保非 Boss 期間會不斷生成小怪
            if (!bossActive) {
                const spawnRate = Math.max(10, 45 - Math.floor(score / 800));
                if(frames % spawnRate === 0) spawnEnemy();
            }
        }

        // 玩家自動射擊
        if(frames % Math.floor(skills.fireInterval) === 0) {
            const bulletCount = skills.bullets;
            const spreadAngle = 0.15;
            const startAngle = -((bulletCount - 1) * spreadAngle) / 2;
            for(let i=0; i < bulletCount; i++) {
                const angle = startAngle + i * spreadAngle;
                projectiles.push({
                    x: player.x,
                    y: player.y - 15,
                    vx: Math.sin(angle) * 10,
                    vy: -Math.cos(angle) * 12,
                    radius: 3.5,
                    damage: skills.bulletDamage,
                    color: '#fff'
                });
            }
        }

        processGameObjects();
    }

    function processGameObjects() {
        // 子彈
        for (let i = projectiles.length - 1; i >= 0; i--) {
            const p = projectiles[i];
            p.x += p.vx || 0; p.y += p.vy;
            if(p.y < -50 || p.x < -50 || p.x > canvas.width + 50) projectiles.splice(i, 1);
        }

        // 敵人子彈
        for (let i = enemyProjectiles.length - 1; i >= 0; i--) {
            const ep = enemyProjectiles[i];
            ep.x += ep.vx; ep.y += ep.vy;
            const dist = Math.hypot(player.x - ep.x, player.y - ep.y);
            if (dist < 12 + ep.radius) {
                health -= 15;
                healthEl.innerText = Math.max(0, Math.floor(health));
                enemyProjectiles.splice(i, 1);
                createExplosion(player.x, player.y, '#f00', 5, 5);
                if (health <= 0) gameOver();
            } else if (ep.y > canvas.height + 50) {
                enemyProjectiles.splice(i, 1);
            }
        }

        // Boss 邏輯
        if (boss) {
            if (boss.y < boss.targetY) boss.y += 1.5;
            else {
                boss.x += boss.speed * boss.dir;
                if (boss.x <= 5 || boss.x + boss.w >= canvas.width - 5) boss.dir *= -1;
            }
            boss.attackTimer++;
            if (boss.attackTimer >= Math.max(30, 80 - (bossLevel * 8))) {
                boss.attackTimer = 0;
                for (let i = 0; i < 5; i++) {
                    const angle = Math.PI/2 - 0.4 + (0.2 * i);
                    enemyProjectiles.push({
                        x: boss.x + boss.w / 2, y: boss.y + boss.h,
                        vx: Math.cos(angle) * 3.5, vy: Math.sin(angle) * 3.5,
                        radius: 5, color: '#ff3e3e'
                    });
                }
            }
            for (let i = projectiles.length - 1; i >= 0; i--) {
                const p = projectiles[i];
                if (p.x > boss.x && p.x < boss.x + boss.w && p.y > boss.y && p.y < boss.y + boss.h) {
                    boss.hp -= p.damage;
                    bossHpFill.style.width = `${(boss.hp / boss.maxHp) * 100}%`;
                    createExplosion(p.x, p.y, '#fff', 2, 4);
                    projectiles.splice(i, 1);
                    if (boss.hp <= 0) {
                        score += 3000;
                        scoreEl.innerText = score;
                        createExplosion(boss.x + boss.w/2, boss.y + boss.h/2, '#ff3e3e', 40, 12);
                        boss = null; 
                        bossActive = false; 
                        bossUi.style.display = 'none';
                    }
                }
            }
        }

        // 經驗值
        for (let i = experienceGems.length - 1; i >= 0; i--) {
            const gem = experienceGems[i];
            gem.y += gem.vy;
            const dist = Math.hypot(player.x - gem.x, player.y - gem.y);
            if (dist < 120) {
                gem.x += (player.x - gem.x) * 0.18; gem.y += (player.y - gem.y) * 0.18;
            }
            if (dist < 20) { gainExp(gem.value); experienceGems.splice(i, 1); }
        }

        // 敵人
        for (let i = enemies.length - 1; i >= 0; i--) {
            const enemy = enemies[i];
            enemy.y += enemy.speed;
            let hit = false;
            for (let j = projectiles.length - 1; j >= 0; j--) {
                const p = projectiles[j];
                if (Math.hypot(p.x - (enemy.x+enemy.w/2), p.y - (enemy.y+enemy.h/2)) < enemy.w/2) {
                    enemy.hp -= p.damage; projectiles.splice(j, 1);
                    if (enemy.hp <= 0) {
                        score += 150;
                        scoreEl.innerText = score;
                        spawnExperience(enemy.x + enemy.w/2, enemy.y + enemy.h/2);
                        createExplosion(enemy.x + enemy.w/2, enemy.y + enemy.h/2, enemy.color, 8, 6);
                        enemies.splice(i, 1);
                        hit = true; break;
                    }
                }
            }
            if (!hit && Math.hypot(player.x - (enemy.x+enemy.w/2), player.y - (enemy.y+enemy.h/2)) < enemy.w/2 + 12) {
                health -= 15; healthEl.innerText = Math.max(0, Math.floor(health));
                createExplosion(enemy.x + enemy.w/2, enemy.y + enemy.h/2, enemy.color, 12, 8);
                enemies.splice(i, 1);
                if (health <= 0) gameOver();
            } else if (!hit && enemy.y > canvas.height) {
                enemies.splice(i, 1);
            }
        }

        for (let i = particles.length - 1; i >= 0; i--) {
            const p = particles[i];
            p.x += p.vx; p.y += p.vy; p.alpha -= p.decay;
            if(p.alpha <= 0) particles.splice(i, 1);
        }
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        ctx.fillStyle = '#fff';
        stars.forEach(star => {
            ctx.globalAlpha = 0.3;
            ctx.fillRect(star.x, star.y, star.size, star.size);
        });
        ctx.globalAlpha = 1;

        if (gameActive && !isPaused) {
            player.x += (player.targetX - player.x) * player.speed;
            player.y += (player.targetY - player.y) * player.speed;
        }

        projectiles.forEach(p => { ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, p.radius, 0, Math.PI*2); ctx.fill(); });
        enemyProjectiles.forEach(ep => { ctx.fillStyle = ep.color; ctx.beginPath(); ctx.arc(ep.x, ep.y, ep.radius, 0, Math.PI*2); ctx.fill(); });
        enemies.forEach(e => { ctx.fillStyle = e.color; ctx.fillRect(e.x, e.y, e.w, e.h); });
        experienceGems.forEach(g => { ctx.fillStyle = '#f1c40f'; ctx.beginPath(); ctx.arc(g.x, g.y, 4, 0, Math.PI*2); ctx.fill(); });
        particles.forEach(p => { ctx.globalAlpha = p.alpha; ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, p.radius, 0, Math.PI*2); ctx.fill(); });
        ctx.globalAlpha = 1;
        
        if (boss) { ctx.fillStyle = boss.color; ctx.fillRect(boss.x, boss.y, boss.w, boss.h); }

        ctx.save();
        ctx.translate(player.x, player.y);
        ctx.fillStyle = player.color;
        ctx.shadowBlur = 10;
        ctx.shadowColor = player.color;
        ctx.beginPath();
        ctx.moveTo(0, -18); ctx.lineTo(18, 18); ctx.lineTo(0, 8); ctx.lineTo(-18, 18);
        ctx.closePath(); ctx.fill();
        ctx.restore();
    }

    function gameOver() {
        gameActive = false;
        titleEl.innerText = "戰機損毀";
        startBtn.innerText = "重啟系統";
        overlay.style.display = 'flex';
    }

    function animate() {
        if(!gameActive) return;
        update();
        draw();
        requestAnimationFrame(animate);
    }
    
    resize();
</script>
</body>
</html>
