<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>星際挑戰者 - Boss 強化版</title>
    <style>
        :root { --primary: #00f2ff; --boss: #ff3e3e; --bg: #050510; --sub: #f1c40f; }
        body { margin: 0; background: var(--bg); color: white; font-family: sans-serif; overflow: hidden; touch-action: none; }
        #game-container { position: relative; width: 100vw; height: 100vh; }
        canvas { display: block; }

        #ui {
            position: absolute; top: 10px; width: 100%;
            display: flex; justify-content: space-around; pointer-events: none; z-index: 100;
        }
        .stat { background: rgba(0,0,0,0.6); padding: 5px 15px; border-radius: 20px; border: 1px solid var(--primary); font-size: 14px; }

        #boss-hp {
            position: absolute; top: 60px; left: 50%; transform: translateX(-50%);
            width: 80%; max-width: 400px; height: 12px; background: #333; border: 2px solid white;
            display: none; z-index: 50; border-radius: 6px; overflow: hidden;
        }
        #hp-bar { width: 100%; height: 100%; background: linear-gradient(90deg, #ff0000, #ff8800); transition: width 0.3s; }

        .modal {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: #111; border: 2px solid var(--primary); padding: 20px;
            display: none; flex-direction: column; gap: 10px; z-index: 500; border-radius: 15px;
            box-shadow: 0 0 20px var(--primary); width: 280px;
        }
        .btn { padding: 12px; background: #222; border: 1px solid #555; color: white; cursor: pointer; border-radius: 8px; font-weight: bold; text-align: center; }
        .btn:active { background: var(--primary); color: black; }
        
        #sub-modal { border-color: var(--sub); box-shadow: 0 0 20px var(--sub); }
        .sub-btn { border-color: var(--sub); color: var(--sub); }
        .sub-btn:active { background: var(--sub); color: black; }
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

    <div id="lv-modal" class="modal">
        <h3 style="text-align:center; margin-top:0;">等級提升！進化選擇</h3>
        <button class="btn" onclick="upgrade('barrage')">🔥 彈幕進化 (數量+散射)</button>
        <button class="btn" onclick="upgrade('s')">⚡ 提升射速</button>
        <button class="btn" onclick="upgrade('d')">💎 強化傷害</button>
    </div>

    <div id="sub-modal" class="modal">
        <h3 style="text-align:center; color:var(--sub); margin-top:0;">BOSS 擊潰！<br>獲得強大副武器</h3>
        <button class="btn sub-btn" onclick="pickSub('missile')">🚀 追蹤導彈 (自動打擊)</button>
        <button class="btn sub-btn" onclick="pickSub('drone')">🛸 浮游雷射 (側翼火力)</button>
        <button class="btn sub-btn" onclick="pickSub('shield')">🛡️ 能量護盾 (抵擋傷害)</button>
    </div>

    <div id="menu" style="position:absolute; inset:0; background:rgba(0,0,0,0.9); display:flex; flex-direction:column; justify-content:center; align-items:center; z-index:1000;">
        <h1 style="color:var(--primary); text-shadow:0 0 10px var(--primary)">星際挑戰者</h1>
        <p id="loading-text" style="color:#aaa">Boss 已強化：具備多樣彈幕</p>
        <button id="start-btn" class="btn" style="padding:15px 40px; font-size:20px; border-radius:30px; color:var(--primary); border-color:var(--primary); display:none;" onclick="start()">進入宇宙</button>
    </div>

    <canvas id="canvas"></canvas>
</div>

<script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const ui = {
        lv: document.getElementById('lv-v'), sc: document.getElementById('sc-v'),
        hp: document.getElementById('hp-v'), bhp: document.getElementById('boss-hp'),
        hbar: document.getElementById('hp-bar'),
        modal: document.getElementById('lv-modal'), 
        subModal: document.getElementById('sub-modal'),
        menu: document.getElementById('menu'),
        startBtn: document.getElementById('start-btn'),
        loading: document.getElementById('loading-text')
    };

    const shipImg = new Image();
    shipImg.src = 'https://png.pngtree.com/png-vector/20240321/ourmid/pngtree-blue-fighter-plane-vector-illustration-png-image_12185790.png';
    shipImg.onload = () => {
        ui.loading.innerText = "戰鬥系統整備完成！";
        ui.startBtn.style.display = 'block';
    };
    shipImg.onerror = () => { ui.startBtn.style.display = 'block'; };

    let state = { running: false, paused: false, score: 0, hp: 100, lv: 1, exp: 0, next: 100 };
    let player = { x: 0, y: 0, w: 50, h: 50, tx: 0, ty: 0, down: false };
    let weapon = { count: 1, speed: 18, dmg: 2, spread: 0.1, sub: null };
    let enemies = [], bullets = [], eb = [], gems = [], subProjectiles = [], boss = null, frames = 0;

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
        if(t==='barrage') {
            weapon.count++;
            weapon.spread = Math.min(0.5, weapon.spread + 0.05);
        }
        if(t==='s') weapon.speed = Math.max(5, weapon.speed - 3);
        if(t==='d') weapon.dmg += 1;
        state.paused = false;
        ui.modal.style.display = 'none';
    }

    function pickSub(type) {
        weapon.sub = type;
        state.paused = false;
        ui.subModal.style.display = 'none';
        state.hp = Math.min(100, state.hp + 40);
    }

    const handleTouch = (e) => {
        const t = e.touches ? e.touches[0] : e;
        if(e.type === 'touchstart' || e.type === 'mousedown') {
            player.down = true; player.tx = t.clientX; player.ty = t.clientY;
        } else if(e.type === 'touchmove' || e.type === 'mousemove') {
            if(!player.down || state.paused) return;
            player.x = Math.max(25, Math.min(canvas.width-25, player.x + (t.clientX - player.tx)));
            player.y = Math.max(100, Math.min(canvas.height-25, player.y + (t.clientY - player.ty)));
            player.tx = t.clientX; player.ty = t.clientY;
        } else { player.down = false; }
    };

    window.addEventListener('touchstart', handleTouch);
    window.addEventListener('touchmove', (e) => { e.preventDefault(); handleTouch(e); }, {passive: false});
    window.addEventListener('touchend', handleTouch);
    window.addEventListener('mousedown', handleTouch);
    window.addEventListener('mousemove', handleTouch);
    window.addEventListener('mouseup', handleTouch);

    function loop() {
        if(!state.running) return;
        if(!state.paused) {
            frames++;
            
            // 玩家開火
            if(frames % weapon.speed === 0) {
                for(let i=0; i<weapon.count; i++) {
                    let angleOffset = (i - (weapon.count-1)/2) * weapon.spread;
                    bullets.push({ 
                        x: player.x, y: player.y - 25, 
                        vx: Math.sin(angleOffset) * 12, vy: -Math.cos(angleOffset) * 12,
                        dmg: weapon.dmg 
                    });
                }
            }

            // 副武器
            if(weapon.sub === 'missile' && frames % 60 === 0 && (enemies.length > 0 || boss)) {
                let target = boss || enemies[0];
                subProjectiles.push({ type: 'missile', x: player.x, y: player.y, target: target, vx: 0, vy: -2 });
            }
            if(weapon.sub === 'drone' && frames % 40 === 0) {
                subProjectiles.push({ type: 'laser', x: player.x - 40, y: player.y, vy: -15 });
                subProjectiles.push({ type: 'laser', x: player.x + 40, y: player.y, vy: -15 });
            }

            // 子彈移動
            bullets.forEach((b, i) => {
                b.x += b.vx; b.y += b.vy;
                if(b.y < -20 || b.x < -20 || b.x > canvas.width+20) bullets.splice(i, 1);
            });

            subProjectiles.forEach((p, i) => {
                if(p.type === 'missile') {
                    if(p.target && (enemies.includes(p.target) || p.target === boss)) {
                        let angle = Math.atan2(p.target.y + 30 - p.y, p.target.x + 30 - p.x);
                        p.vx += Math.cos(angle) * 0.8; p.vy += Math.sin(angle) * 0.8;
                    } else { p.vy -= 0.5; }
                    p.x += p.vx; p.y += p.vy; p.vx *= 0.96; p.vy *= 0.96;
                } else { p.y += p.vy; }
                if(p.y < -50 || p.y > canvas.height + 50) subProjectiles.splice(i, 1);
            });

            // 敵機邏輯
            if(!boss && frames % 35 === 0) {
                enemies.push({ x: Math.random()*(canvas.width-40), y: -50, w: 40, h: 30, hp: 3 + Math.floor(state.score/5000) });
            }

            enemies.forEach((e, i) => {
                e.y += (3 + state.lv * 0.1);
                if(e.y > canvas.height) { enemies.splice(i, 1); return; }

                if(Math.hypot(e.x+20-player.x, e.y+15-player.y) < 35) {
                    if(weapon.sub === 'shield') { weapon.sub = null; } else { state.hp -= 15; }
                    enemies.splice(i, 1); return;
                }

                bullets.forEach((b, bi) => {
                    if(b.x > e.x && b.x < e.x+e.w && b.y > e.y && b.y < e.y+e.h) {
                        e.hp -= b.dmg; bullets.splice(bi, 1);
                    }
                });

                if(e.hp <= 0) {
                    state.score += 100; gems.push({x: e.x+20, y: e.y+15}); enemies.splice(i, 1);
                }
            });

            // 升級寶石
            gems.forEach((g, i) => {
                g.y += 3;
                if(Math.hypot(g.x-player.x, g.y-player.y) < 45) {
                    state.exp += 35; gems.splice(i, 1);
                    if(state.exp >= state.next) {
                        state.lv++; state.exp = 0; state.next *= 1.35;
                        state.paused = true; ui.modal.style.display = 'flex';
                    }
                } else if(g.y > canvas.height) gems.splice(i, 1);
            });

            // 強大 Boss 邏輯
            if(!boss && state.score > 0 && state.score % 10000 >= 9000) {
                let mhp = 1200 + (state.lv * 400);
                boss = { x: canvas.width/2-60, y: -150, w: 120, h: 100, hp: mhp, mhp: mhp, vx: 3, phase: 0 };
                ui.bhp.style.display = 'block';
            }
            if(boss) {
                if(boss.y < 100) boss.y += 2;
                else {
                    boss.x += boss.vx;
                    if(boss.x < 10 || boss.x > canvas.width-boss.w-10) boss.vx *= -1;
                    
                    // Boss 攻擊機制
                    if(frames % 45 === 0) {
                        // 模式 1: 散彈
                        eb.push({x: boss.x+60, y: boss.y+100, vx: -2, vy: 5});
                        eb.push({x: boss.x+60, y: boss.y+100, vx: 0, vy: 6});
                        eb.push({x: boss.x+60, y: boss.y+100, vx: 2, vy: 5});
                        
                        // 模式 2: 低血量時發動狙擊彈
                        if(boss.hp < boss.mhp * 0.5 && frames % 90 === 0) {
                            let angle = Math.atan2(player.y - (boss.y+50), player.x - (boss.x+60));
                            eb.push({x: boss.x+60, y: boss.y+50, vx: Math.cos(angle)*8, vy: Math.sin(angle)*8, special: true});
                        }
                    }
                }
                
                bullets.forEach((b, bi) => {
                    if(b.x > boss.x && b.x < boss.x+boss.w && b.y > boss.y && b.y < boss.y+boss.h) {
                        boss.hp -= b.dmg; bullets.splice(bi, 1);
                    }
                });

                if(boss.hp <= 0) {
                    state.score += 5000; boss = null; ui.bhp.style.display = 'none';
                    state.paused = true; ui.subModal.style.display = 'flex';
                } else {
                    ui.hbar.style.width = (boss.hp/boss.mhp*100) + '%';
                }
            }

            // 敵方子彈
            eb.forEach((b, i) => {
                b.x += (b.vx || 0); b.y += (b.vy || 7);
                if(Math.hypot(b.x-player.x, b.y-player.y) < 25) {
                    if(weapon.sub === 'shield') { weapon.sub = null; } else { state.hp -= 20; }
                    eb.splice(i, 1);
                } else if(b.y > canvas.height || b.y < -100) eb.splice(i, 1);
            });

            if(state.hp <= 0) location.reload();
            ui.sc.innerText = state.score; ui.lv.innerText = state.lv; ui.hp.innerText = Math.max(0, state.hp);
        }
        draw();
        requestAnimationFrame(loop);
    }

    function draw() {
        ctx.fillStyle = '#050510'; ctx.fillRect(0,0,canvas.width,canvas.height);
        
        // 星星背景
        ctx.fillStyle = '#fff';
        for(let i=0; i<8; i++) {
            let sY = (frames*(1+i*0.2)) % canvas.height;
            ctx.fillRect((i*canvas.width/8 + 50)%canvas.width, sY, 2, 2);
        }

        if(weapon.sub === 'shield') {
            ctx.strokeStyle = '#00f2ff'; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.arc(player.x, player.y, 42, 0, Math.PI*2); ctx.stroke();
            ctx.fillStyle = 'rgba(0,242,255,0.15)'; ctx.fill();
        }

        if (shipImg.complete) {
            ctx.drawImage(shipImg, player.x - 25, player.y - 25, 50, 50);
        }

        bullets.forEach(b => {
            ctx.fillStyle = '#00f2ff'; ctx.shadowBlur = 8; ctx.shadowColor = '#00f2ff';
            ctx.fillRect(b.x-2, b.y-8, 4, 16); ctx.shadowBlur = 0;
        });

        subProjectiles.forEach(p => {
            ctx.fillStyle = p.type==='missile'?'#ff4e00':'#f1c40f';
            ctx.beginPath(); ctx.arc(p.x, p.y, 5, 0, Math.PI*2); ctx.fill();
        });

        enemies.forEach(e => {
            ctx.fillStyle = '#a040ff'; ctx.fillRect(e.x, e.y, e.w, e.h);
            ctx.strokeStyle = '#fff'; ctx.strokeRect(e.x, e.y, e.w, e.h);
        });
        
        gems.forEach(g => {
            ctx.fillStyle = '#ffff00'; ctx.beginPath(); ctx.arc(g.x, g.y, 6, 0, Math.PI*2); ctx.fill();
        });

        if(boss) {
            // Boss 本體渲染
            ctx.fillStyle = '#ff3e3e';
            ctx.shadowBlur = 20; ctx.shadowColor = 'red';
            ctx.fillRect(boss.x, boss.y, boss.w, boss.h);
            // 裝飾
            ctx.fillStyle = '#000'; ctx.fillRect(boss.x+20, boss.y+20, 25, 25); ctx.fillRect(boss.x+boss.w-45, boss.y+20, 25, 25);
            ctx.shadowBlur = 0;
        }

        eb.forEach(b => {
            ctx.fillStyle = b.special ? '#ff00ff' : '#ff0000';
            ctx.beginPath(); ctx.arc(b.x, b.y, b.special?10:8, 0, Math.PI*2); ctx.fill();
        });
    }

    window.addEventListener('resize', init);
    init();
</script>
</body>
</html>
