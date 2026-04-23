<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>星際守護者 - 驚喜保險版</title>
    <style>
        :root { --primary: #00f2ff; --boss: #ff3e3e; --bg: #050510; }
        body { margin: 0; background: var(--bg); color: white; font-family: sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 100vw; height: 100vh; }
        canvas { display: block; }

        #ui {
            position: absolute; top: 10px; width: 100%;
            display: flex; justify-content: space-around; pointer-events: none; z-index: 100;
        }
        .stat { background: rgba(0,0,0,0.6); padding: 5px 15px; border-radius: 20px; border: 1px solid var(--primary); font-size: 14px; }

        #msg-box {
            position: absolute; inset: 0; display: none;
            flex-direction: column; justify-content: center; align-items: center;
            background: rgba(0,0,0,0.8); z-index: 9999; text-align: center;
            animation: fadeIn 1s forwards;
        }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        
        .bday-text {
            font-size: 42px; font-weight: bold;
            background: linear-gradient(to right, #ff00ea, #00f2ff, #ffff00);
            -webkit-background-clip: text; color: transparent;
            text-shadow: 0 0 20px rgba(255,255,255,0.5);
            margin-bottom: 20px;
        }

        #boss-hp {
            position: absolute; top: 60px; left: 50%; transform: translateX(-50%);
            width: 200px; height: 10px; background: #333; border: 2px solid white;
            display: none; z-index: 50;
        }
        #hp-bar { width: 100%; height: 100%; background: red; }

        .modal {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: #111; border: 2px solid var(--primary); padding: 20px;
            display: none; flex-direction: column; gap: 10px; z-index: 500;
        }
        .btn { padding: 10px; background: #222; border: 1px solid #555; color: white; cursor: pointer; }
        .btn:active { background: var(--primary); color: black; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="ui">
        <div class="stat">等級: <span id="lv-v">1</span></div>
        <div class="stat">得分: <span id="sc-v">0</span></div>
        <div class="stat">生命: <span id="hp-v">100</span>%</div>
    </div>

    <div id="boss-hp"><div id="hp-bar"></div></div>

    <div id="msg-box">
        <div class="bday-text">Happy Birthday<br>Sam! 🎂</div>
        <div style="font-size: 18px; color: #fff;">你成功守護了星系！</div>
        <button class="btn" style="margin-top:30px; border-radius:20px; padding:10px 30px;" onclick="location.reload()">再玩一次</button>
    </div>

    <div id="lv-modal" class="modal">
        <h3 style="text-align:center">升級武器</h3>
        <button class="btn" onclick="upgrade('b')">增加子彈數量</button>
        <button class="btn" onclick="upgrade('s')">提升射速</button>
        <button class="btn" onclick="upgrade('d')">提升傷害</button>
    </div>

    <div id="menu" style="position:absolute; inset:0; background:rgba(0,0,0,0.9); display:flex; flex-direction:column; justify-content:center; align-items:center; z-index:1000;">
        <h1 style="color:var(--primary); text-shadow:0 0 10px var(--primary)">星際守護者</h1>
        <p style="color:#aaa">目標：10,000 分觸發驚喜</p>
        <button class="btn" style="padding:15px 40px; font-size:20px; border-radius:30px; color:var(--primary); border-color:var(--primary)" onclick="start()">開始戰鬥</button>
    </div>

    <canvas id="canvas"></canvas>
</div>

<script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const ui = {
        lv: document.getElementById('lv-v'), sc: document.getElementById('sc-v'),
        hp: document.getElementById('hp-v'), bhp: document.getElementById('boss-hp'),
        hbar: document.getElementById('hp-bar'), msg: document.getElementById('msg-box'),
        modal: document.getElementById('lv-modal'), menu: document.getElementById('menu')
    };

    let state = { running: false, paused: false, score: 0, hp: 100, lv: 1, exp: 0, next: 100, win: false };
    let player = { x: 0, y: 0, r: 15, tx: 0, ty: 0, down: false };
    let weapon = { count: 1, speed: 20, dmg: 2 };
    let enemies = [], bullets = [], eb = [], gems = [], boss = null, frames = 0;

    function init() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        player.x = canvas.width / 2;
        player.y = canvas.height * 0.8;
    }

    function start() {
        ui.menu.style.display = 'none';
        state.running = true;
        loop();
    }

    function upgrade(t) {
        if(t==='b') weapon.count++;
        if(t==='s') weapon.speed = Math.max(5, weapon.speed - 3);
        if(t==='d') weapon.dmg += 1;
        state.paused = false;
        ui.modal.style.display = 'none';
    }

    // 觸控邏輯
    const handleTouch = (e) => {
        const t = e.touches ? e.touches[0] : e;
        if(e.type === 'touchstart' || e.type === 'mousedown') {
            player.down = true;
            player.tx = t.clientX; player.ty = t.clientY;
        } else if(e.type === 'touchmove' || e.type === 'mousemove') {
            if(!player.down || state.paused) return;
            player.x = Math.max(15, Math.min(canvas.width-15, player.x + (t.clientX - player.tx)));
            player.y = Math.max(100, Math.min(canvas.height-15, player.y + (t.clientY - player.ty)));
            player.tx = t.clientX; player.ty = t.clientY;
        } else {
            player.down = false;
        }
    };
    window.addEventListener('touchstart', handleTouch);
    window.addEventListener('touchmove', (e) => { e.preventDefault(); handleTouch(e); }, {passive: false});
    window.addEventListener('touchend', handleTouch);
    window.addEventListener('mousedown', handleTouch);
    window.addEventListener('mousemove', handleTouch);
    window.addEventListener('mouseup', handleTouch);

    function loop() {
        if(!state.running) return;
        if(!state.paused && !state.win) {
            frames++;
            
            // 驚喜判斷 - 強力保險版
            if(state.score >= 10000) {
                state.win = true;
                enemies = []; bullets = []; eb = []; boss = null;
                ui.msg.style.display = 'flex';
                ui.bhp.style.display = 'none';
            }

            // 射擊
            if(frames % weapon.speed === 0) {
                for(let i=0; i<weapon.count; i++) {
                    bullets.push({ x: player.x + (i - (weapon.count-1)/2)*10, y: player.y - 15 });
                }
            }

            // 移動子彈
            for(let i=bullets.length-1; i>=0; i--) {
                bullets[i].y -= 10;
                if(bullets[i].y < -20) bullets.splice(i, 1);
            }

            // 敵機生成
            if(!boss && frames % 40 === 0) {
                enemies.push({ x: Math.random()*(canvas.width-40), y: -50, w: 40, h: 30, hp: 2 + state.lv });
            }

            // 敵機邏輯
            for(let i=enemies.length-1; i>=0; i--) {
                const e = enemies[i]; e.y += 3;
                if(e.y > canvas.height) { enemies.splice(i, 1); continue; }

                // 撞玩家
                if(Math.hypot(e.x+20-player.x, e.y+15-player.y) < 30) {
                    state.hp -= 10; enemies.splice(i, 1);
                    if(state.hp <= 0) location.reload();
                    continue;
                }

                // 被子彈打
                for(let j=bullets.length-1; j>=0; j--) {
                    const b = bullets[j];
                    if(b.x > e.x && b.x < e.x+e.w && b.y > e.y && b.y < e.y+e.h) {
                        e.hp -= weapon.dmg; bullets.splice(j, 1);
                        if(e.hp <= 0) {
                            state.score += 100;
                            gems.push({x: e.x+20, y: e.y+15});
                            enemies.splice(i, 1);
                            break;
                        }
                    }
                }
            }

            // 寶石邏輯
            for(let i=gems.length-1; i>=0; i--) {
                const g = gems[i]; g.y += 2;
                if(Math.hypot(g.x-player.x, g.y-player.y) < 30) {
                    state.exp += 25; gems.splice(i, 1);
                    if(state.exp >= state.next) {
                        state.lv++; state.exp = 0; state.next *= 1.5;
                        state.paused = true; ui.modal.style.display = 'flex';
                    }
                } else if(g.y > canvas.height) gems.splice(i, 1);
            }

            // Boss 邏輯
            if(!boss && state.score >= 8000 && !state.win) {
                boss = { x: canvas.width/2-50, y: -100, w: 100, h: 80, hp: 500, mhp: 500, vx: 2 };
                ui.bhp.style.display = 'block';
            }
            if(boss) {
                if(boss.y < 100) boss.y += 1;
                else {
                    boss.x += boss.vx;
                    if(boss.x < 0 || boss.x > canvas.width-boss.w) boss.vx *= -1;
                    if(frames % 50 === 0) eb.push({x: boss.x+50, y: boss.y+80});
                }
                for(let i=bullets.length-1; i>=0; i--) {
                    const b = bullets[i];
                    if(b.x > boss.x && b.x < boss.x+boss.w && b.y > boss.y && b.y < boss.y+boss.h) {
                        boss.hp -= weapon.dmg; bullets.splice(i, 1);
                        if(boss.hp <= 0) {
                            state.score += 2000; boss = null; ui.bhp.style.display = 'none';
                        }
                    }
                }
                if(boss) ui.hbar.style.width = (boss.hp/boss.mhp*100) + '%';
            }

            // 敵方子彈
            for(let i=eb.length-1; i>=0; i--) {
                eb[i].y += 5;
                if(Math.hypot(eb[i].x-player.x, eb[i].y-player.y) < 20) {
                    state.hp -= 15; eb.splice(i, 1);
                    if(state.hp <= 0) location.reload();
                } else if(eb[i].y > canvas.height) eb.splice(i, 1);
            }

            // UI 更新
            ui.sc.innerText = state.score;
            ui.lv.innerText = state.lv;
            ui.hp.innerText = Math.max(0, state.hp);
        }

        draw();
        requestAnimationFrame(loop);
    }

    function draw() {
        ctx.fillStyle = '#050510'; ctx.fillRect(0,0,canvas.width,canvas.height);
        
        // 玩家 (青色三角)
        ctx.fillStyle = '#00f2ff';
        ctx.beginPath();
        ctx.moveTo(player.x, player.y-15);
        ctx.lineTo(player.x+15, player.y+15);
        ctx.lineTo(player.x-15, player.y+15);
        ctx.fill();

        // 子彈
        ctx.fillStyle = '#fff';
        bullets.forEach(b => ctx.fillRect(b.x-2, b.y, 4, 10));

        // 敵機 (紫色方塊)
        ctx.fillStyle = '#a040ff';
        enemies.forEach(e => ctx.fillRect(e.x, e.y, e.w, e.h));

        // 寶石 (黃色點)
        ctx.fillStyle = '#ffff00';
        gems.forEach(g => ctx.beginPath() || ctx.arc(g.x, g.y, 5, 0, 7) || ctx.fill());

        // Boss (紅巨星)
        if(boss) {
            ctx.fillStyle = 'red'; ctx.fillRect(boss.x, boss.y, boss.w, boss.h);
            ctx.fillStyle = 'white'; ctx.fillRect(boss.x+15, boss.y+15, 20, 20); ctx.fillRect(boss.x+65, boss.y+15, 20, 20);
        }

        // 敵彈
        ctx.fillStyle = '#ff0000';
        eb.forEach(b => ctx.beginPath() || ctx.arc(b.x, b.y, 8, 0, 7) || ctx.fill());
    }

    window.addEventListener('resize', init);
    init();
</script>
</body>
</html>
