<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>星際守護者 - 穩定優化版</title>
    <style>
        :root {
            --primary: #00f2ff;
            --boss: #ff3e3e;
            --bg: #050510;
        }
        body {
            margin: 0; padding: 0; background: var(--bg); color: white;
            font-family: sans-serif; overflow: hidden; touch-action: none;
        }
        #game-container { position: relative; width: 100vw; height: 100vh; overflow: hidden; }
        canvas { display: block; background: #000; }

        /* UI 佈局優化，防止重疊 */
        #ui {
            position: absolute; top: 10px; left: 0; width: 100%;
            display: flex; justify-content: space-around; pointer-events: none;
            z-index: 10;
        }
        .stat {
            background: rgba(255,255,255,0.1); padding: 5px 12px;
            border-radius: 15px; font-size: 14px; border: 1px solid rgba(0,242,255,0.3);
        }

        #exp-bar {
            position: absolute; top: 50px; left: 10%; width: 80%; height: 6px;
            background: rgba(255,255,255,0.1); border-radius: 3px; z-index: 10;
        }
        #exp-inner { width: 0%; height: 100%; background: #f1c40f; transition: width 0.2s; }

        #boss-ui {
            position: absolute; top: 70px; left: 50%; transform: translateX(-50%);
            width: 70%; display: none; text-align: center; z-index: 10;
        }
        .hp-bg { width: 100%; height: 8px; background: rgba(255,0,0,0.2); border: 1px solid var(--boss); border-radius: 4px; }
        #hp-fill { width: 100%; height: 100%; background: var(--boss); }

        .modal {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(10, 10, 30, 0.95); border: 2px solid var(--primary);
            padding: 20px; border-radius: 15px; display: none; flex-direction: column;
            gap: 10px; width: 260px; z-index: 100;
        }
        .opt { background: #222; padding: 12px; border-radius: 8px; border: 1px solid #444; }
        .opt:active { background: var(--primary); color: #000; }

        #msg {
            position: absolute; width: 100%; top: 40%; text-align: center;
            font-size: 32px; font-weight: bold; color: #ff00ea; display: none;
            z-index: 150; pointer-events: none; text-shadow: 0 0 10px #000;
        }

        #menu {
            position: absolute; inset: 0; background: rgba(0,0,0,0.85);
            display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 200;
        }
        .btn {
            padding: 12px 35px; background: none; color: var(--primary);
            border: 2px solid var(--primary); border-radius: 25px; font-size: 18px;
        }
    </style>
</head>
<body>

<div id="game-container">
    <div id="ui">
        <div class="stat">Lv: <span id="lv-v">1</span></div>
        <div class="stat">得分: <span id="sc-v">0</span></div>
        <div class="stat">生命: <span id="hp-v">100</span>%</div>
    </div>
    <div id="exp-bar"><div id="exp-inner"></div></div>
    
    <div id="boss-ui">
        <div style="color:red; font-size:11px; margin-bottom:2px;">BOSS 警告</div>
        <div class="hp-bg"><div id="hp-fill"></div></div>
    </div>

    <div id="msg">Happy Birthday Sam! 🎂</div>

    <div id="lv-modal" class="modal">
        <h3 style="margin:0 0 10px 0; text-align:center;">武器進化</h3>
        <div class="opt" onclick="upgrade('bullets')">增加彈道數量</div>
        <div class="opt" onclick="upgrade('rate')">提升射擊速度</div>
        <div class="opt" onclick="upgrade('dmg')">增加子彈傷害</div>
    </div>

    <div id="menu">
        <h2 style="color:var(--primary)">星際守護者</h2>
        <p style="color:#aaa; font-size:12px; margin-bottom:20px;">達到 10,000 分觸發驚喜</p>
        <button class="btn" onclick="start()">開始戰鬥</button>
    </div>

    <canvas id="canvas"></canvas>
</div>

<script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const ui = {
        lv: document.getElementById('lv-v'),
        sc: document.getElementById('sc-v'),
        hp: document.getElementById('hp-v'),
        exp: document.getElementById('exp-inner'),
        boss: document.getElementById('boss-ui'),
        bhp: document.getElementById('hp-fill'),
        msg: document.getElementById('msg'),
        modal: document.getElementById('lv-modal'),
        menu: document.getElementById('menu')
    };

    let state = {
        active: false, paused: false, frames: 0, score: 0,
        hp: 100, lv: 1, exp: 0, nExp: 100,
        surpriseTriggered: false
    };

    let weapon = { bullets: 1, interval: 18, dmg: 1 };
    let player = { x: 0, y: 0, r: 15, tx: 0, ty: 0, down: false };
    
    let enemies = [], bullets = [], eBullets = [], gems = [], parts = [], stars = [], boss = null;

    function init() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        player.x = canvas.width / 2;
        player.y = canvas.height * 0.8;
        stars = Array.from({length: 40}, () => ({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height,
            v: Math.random() * 2 + 0.5
        }));
    }

    function start() {
        ui.menu.style.display = 'none';
        state = { active: true, paused: false, frames: 0, score: 0, hp: 100, lv: 1, exp: 0, nExp: 100, surpriseTriggered: false };
        weapon = { bullets: 1, interval: 18, dmg: 1 };
        enemies = []; bullets = []; eBullets = []; gems = []; parts = []; boss = null;
        updateUI();
        loop();
    }

    function upgrade(type) {
        if(type==='bullets') weapon.bullets++;
        if(type==='rate') weapon.interval = Math.max(6, weapon.interval * 0.75);
        if(type==='dmg') weapon.dmg += 0.5;
        ui.modal.style.display = 'none';
        state.paused = false;
    }

    function spawnPart(x, y, color, n=6) {
        for(let i=0; i<n; i++) {
            parts.push({ x, y, vx: (Math.random()-0.5)*5, vy: (Math.random()-0.5)*5, a: 1, c: color });
        }
    }

    // 輸入處理
    const getIn = e => e.touches ? e.touches[0] : e;
    const onDown = e => { if(!state.active) return; player.down = true; const i = getIn(e); player.tx = i.clientX; player.ty = i.clientY; };
    const onMove = e => {
        if(!player.down || state.paused) return;
        const i = getIn(e);
        player.x = Math.max(15, Math.min(canvas.width-15, player.x + (i.clientX - player.tx)));
        player.y = Math.max(100, Math.min(canvas.height-15, player.y + (i.clientY - player.ty)));
        player.tx = i.clientX; player.ty = i.clientY;
    };
    window.addEventListener('touchstart', onDown); window.addEventListener('mousedown', onDown);
    window.addEventListener('touchmove', e => { e.preventDefault(); onMove(e); }, {passive: false});
    window.addEventListener('mousemove', onMove);
    window.addEventListener('touchend', () => player.down = false); window.addEventListener('mouseup', () => player.down = false);

    function updateUI() {
        ui.sc.innerText = Math.floor(state.score);
        ui.hp.innerText = Math.max(0, Math.floor(state.hp));
        ui.lv.innerText = state.lv;
        ui.exp.style.width = (state.exp / state.nExp * 100) + '%';
    }

    function loop() {
        if(!state.active) return;
        if(!state.paused) {
            state.frames++;
            
            // 背景
            stars.forEach(s => { s.y += s.v; if(s.y > canvas.height) s.y = 0; });

            // 射擊
            if(state.frames % Math.floor(weapon.interval) === 0) {
                const step = 0.25;
                const startIdx = -(weapon.bullets - 1) * step / 2;
                for(let i=0; i<weapon.bullets; i++) {
                    bullets.push({ x: player.x, y: player.y-10, vx: Math.sin(startIdx + i*step)*8, vy: -10 });
                }
            }

            // 安全刪除循環 - 子彈
            for(let i=bullets.length-1; i>=0; i--) {
                const b = bullets[i]; b.x += b.vx; b.y += b.vy;
                if(b.y < -10) bullets.splice(i, 1);
            }

            // 敵機生成
            if(!boss && state.frames % 50 === 0) {
                enemies.push({ x: Math.random()*(canvas.width-30), y: -40, w: 30, h: 25, hp: 1 + Math.floor(state.score/6000), v: 2.5 });
            }

            // 敵機邏輯
            for(let i=enemies.length-1; i>=0; i--) {
                const e = enemies[i]; e.y += e.v;
                
                // 碰撞子彈
                for(let j=bullets.length-1; j>=0; j--) {
                    const b = bullets[j];
                    if(b.x > e.x && b.x < e.x+e.w && b.y > e.y && b.y < e.y+e.h) {
                        e.hp -= weapon.dmg; bullets.splice(j, 1);
                        if(e.hp <= 0) {
                            spawnPart(e.x+e.w/2, e.y+e.h/2, '#a040ff');
                            gems.push({ x: e.x+e.w/2, y: e.y+e.h/2 });
                            state.score += 150;
                            enemies.splice(i, 1);
                            break;
                        }
                    }
                }
                if(e && e.y > canvas.height) enemies.splice(i, 1);
                else if(e && Math.hypot(e.x+15-player.x, e.y+12-player.y) < 25) {
                    state.hp -= 20; enemies.splice(i, 1);
                    if(state.hp <= 0) { state.active = false; ui.menu.style.display = 'flex'; }
                }
            }

            // 寶石
            for(let i=gems.length-1; i>=0; i--) {
                const g = gems[i]; g.y += 2;
                const d = Math.hypot(g.x-player.x, g.y-player.y);
                if(d < 100) { g.x += (player.x-g.x)*0.15; g.y += (player.y-g.y)*0.15; }
                if(d < 20) {
                    state.exp += 35; state.score += 20;
                    if(state.exp >= state.nExp) {
                        state.exp = 0; state.nExp *= 1.35; state.lv++;
                        state.paused = true; ui.modal.style.display = 'flex';
                    }
                    gems.splice(i, 1);
                } else if(g.y > canvas.height) gems.splice(i, 1);
            }

            // Boss 觸發
            if(!boss && !state.surpriseTriggered && state.score > 8500) {
                ui.boss.style.display = 'block';
                boss = { x: canvas.width/2-40, y: -80, w: 80, h: 60, hp: 400, mhp: 400, dx: 2 };
            }

            if(boss) {
                if(boss.y < 80) boss.y += 1;
                else {
                    boss.x += boss.dx;
                    if(boss.x < 10 || boss.x > canvas.width-boss.w-10) boss.dx *= -1;
                    if(state.frames % 60 === 0) eBullets.push({x: boss.x+boss.w/2, y: boss.y+boss.h, vx: 0, vy: 5});
                }
                for(let i=bullets.length-1; i>=0; i--) {
                    const b = bullets[i];
                    if(b.x > boss.x && b.x < boss.x+boss.w && b.y > boss.y && b.y < boss.y+boss.h) {
                        boss.hp -= weapon.dmg; bullets.splice(i, 1);
                        if(boss.hp <= 0) {
                            spawnPart(boss.x+40, boss.y+30, 'red', 20);
                            state.score += 2000;
                            boss = null;
                            ui.boss.style.display = 'none';
                            eBullets = [];
                        }
                    }
                }
                if(boss) ui.bhp.style.width = (boss.hp/boss.mhp*100)+'%';
            }

            // 敵方子彈
            for(let i=eBullets.length-1; i>=0; i--) {
                const p = eBullets[i]; p.y += p.vy;
                if(Math.hypot(p.x-player.x, p.y-player.y) < 18) {
                    state.hp -= 10; eBullets.splice(i, 1);
                    if(state.hp <= 0) { state.active = false; ui.menu.style.display = 'flex'; }
                } else if(p.y > canvas.height) eBullets.splice(i, 1);
            }

            // 驚喜觸發
            if(!state.surpriseTriggered && state.score >= 10000) {
                state.surpriseTriggered = true;
                ui.msg.style.display = 'block';
                // 驚喜時清空螢幕，確保不卡
                enemies = []; eBullets = []; boss = null; ui.boss.style.display = 'none';
                setTimeout(() => { ui.msg.style.display = 'none'; }, 4000);
            }

            // 粒子
            for(let i=parts.length-1; i>=0; i--) {
                const p = parts[i]; p.x += p.vx; p.y += p.vy; p.a -= 0.03;
                if(p.a <= 0) parts.splice(i, 1);
            }
        }

        draw();
        updateUI();
        requestAnimationFrame(loop);
    }

    function draw() {
        ctx.fillStyle = '#000'; ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        ctx.fillStyle = '#fff';
        stars.forEach(s => ctx.fillRect(s.x, s.y, 1.5, 1.5));

        // 玩家
        ctx.fillStyle = '#00f2ff';
        ctx.beginPath();
        ctx.moveTo(player.x, player.y-15);
        ctx.lineTo(player.x+12, player.y+10);
        ctx.lineTo(player.x-12, player.y+10);
        ctx.fill();

        // 各類物件
        ctx.fillStyle = '#fff'; bullets.forEach(b => ctx.fillRect(b.x-2, b.y-2, 4, 8));
        ctx.fillStyle = '#a040ff'; enemies.forEach(e => ctx.fillRect(e.x, e.y, e.w, e.h));
        ctx.fillStyle = '#f1c40f'; gems.forEach(g => ctx.fillRect(g.x-4, g.y-4, 8, 8));
        ctx.fillStyle = 'red'; eBullets.forEach(p => ctx.beginPath() || ctx.arc(p.x, p.y, 5, 0, 7) || ctx.fill());
        
        if(boss) {
            ctx.fillStyle = 'red'; ctx.fillRect(boss.x, boss.y, boss.w, boss.h);
            ctx.fillStyle = 'white'; ctx.fillRect(boss.x+10, boss.y+10, 10, 10); ctx.fillRect(boss.x+boss.w-20, boss.y+10, 10, 10);
        }

        parts.forEach(p => {
            ctx.globalAlpha = p.a; ctx.fillStyle = p.c;
            ctx.fillRect(p.x, p.y, 2, 2);
        });
        ctx.globalAlpha = 1;
    }

    window.addEventListener('resize', init);
    init();
</script>
</body>
</html>
