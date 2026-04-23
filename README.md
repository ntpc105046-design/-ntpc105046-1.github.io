<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <!-- viewport-fit=cover 確保填滿瀏海屏區域 -->
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
            font-family: -apple-system, sans-serif;
            color: white;
            position: fixed; /* 防止手機瀏覽器橡皮筋回彈效果 */
        }

        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        canvas {
            display: block;
            touch-action: none; /* 重要：防止觸碰時觸發網頁縮放或滾動 */
            background: radial-gradient(circle at center, #111 0%, #000 100%);
        }

        /* UI 配置：使用絕對定位確保不會擠壓畫布 */
        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none; /* 讓點擊事件穿透 UI 到達遊戲 */
            padding: env(safe-area-inset-top) 15px 0 15px;
            z-index: 10;
        }

        .stats-top {
            display: flex;
            justify-content: space-between;
            padding: 10px 0;
            font-size: 14px;
            font-weight: bold;
            text-shadow: 0 2px 4px rgba(0,0,0,0.8);
        }

        .exp-bar {
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

        /* 覆蓋層佈局 */
        #overlay {
            position: absolute;
            inset: 0;
            background: rgba(0,0,0,0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
            text-align: center;
        }

        .btn {
            background: transparent;
            color: var(--primary-color);
            border: 2px solid var(--primary-color);
            padding: 15px 40px;
            font-size: 18px;
            border-radius: 50px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 20px;
        }

        .btn:active {
            background: var(--primary-color);
            color: #000;
        }

        h1 {
            font-size: 40px;
            background: linear-gradient(to bottom, #fff, var(--primary-color));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin: 0;
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>
    
    <div id="ui-layer">
        <div class="stats-top">
            <span>LV: <span id="player-lv">1</span></span>
            <span>SCORE: <span id="score">0</span></span>
            <span>HP: <span id="health">100</span>%</span>
        </div>
        <div class="exp-bar"><div id="exp-fill"></div></div>
    </div>

    <div id="overlay">
        <h1 id="title-text">星際守護者</h1>
        <button id="startBtn" class="btn">開始戰鬥</button>
        <p style="color: #666; font-size: 12px; margin-top: 20px;">
            滑鼠或手指拖曳以移動<br>
            收集金球升級，達到 10,000 分有驚喜！
        </p>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    
    let gameActive = false;
    let score = 0;
    let health = 100;
    let playerLv = 1;
    let currentExp = 0;
    let nextExp = 100;

    const player = {
        x: 0, y: 0,
        radius: 15,
        targetX: 0, targetY: 0
    };

    let stars = [], projectiles = [], enemies = [], gems = [];

    // 核心修正：自動適應螢幕尺寸
    function resize() {
        const w = window.innerWidth;
        const h = window.innerHeight;
        
        // 設置畫布解析度與螢幕一致
        canvas.width = w;
        canvas.height = h;

        // 初始化玩家位置
        if (!gameActive) {
            player.x = w / 2;
            player.y = h * 0.8;
            player.targetX = player.x;
            player.targetY = player.y;
        }

        // 生成背景星空AS
        stars = [];
        for(let i=0; i<50; i++) {
            stars.push({
                x: Math.random() * w,
                y: Math.random() * h,
                speed: Math.random() * 2 + 0.5,
                size: Math.random() * 2
            });
        }
    }

    // 監聽視窗變化
    window.addEventListener('resize', resize);
    window.addEventListener('orientationchange', () => setTimeout(resize, 200));

    // 控制邏輯
    function handleMove(e) {
        if (!gameActive) return;
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        
        player.targetX = clientX;
        player.targetY = clientY;
    }

    canvas.addEventListener('mousemove', handleMove);
    canvas.addEventListener('touchmove', (e) => {
        e.preventDefault();
        handleMove(e);
    }, { passive: false });

    // 遊戲循環
    function update() {
        if (!gameActive) return;

        // 玩家平滑移動
        player.x += (player.targetX - player.x) * 0.2;
        player.y += (player.targetY - player.y) * 0.2;

        // 背景滾動
        stars.forEach(s => {
            s.y += s.speed;
            if (s.y > canvas.height) s.y = 0;
        });

        // 簡單射擊邏輯 (每10幀)
        if (Math.floor(Date.now() / 100) % 2 === 0) {
            projectiles.push({ x: player.x, y: player.y - 20 });
        }

        projectiles.forEach((p, i) => {
            p.y -= 10;
            if (p.y < 0) projectiles.splice(i, 1);
        });

        // 隨機生成敵人
        if (Math.random() < 0.03) {
            enemies.push({
                x: Math.random() * (canvas.width - 30),
                y: -30,
                speed: 2 + Math.random() * 2
            });
        }

        enemies.forEach((en, i) => {
            en.y += en.speed;
            if (en.y > canvas.height) enemies.splice(i, 1);

            // 碰撞子彈
            projectiles.forEach((p, pi) => {
                if (Math.hypot(p.x - (en.x+15), p.y - (en.y+15)) < 20) {
                    enemies.splice(i, 1);
                    projectiles.splice(pi, 1);
                    score += 100;
                    document.getElementById('score').innerText = score;
                    // 掉落經驗球
                    gems.push({ x: en.x + 15, y: en.y + 15 });
                }
            });
        });

        gems.forEach((g, i) => {
            g.y += 2;
            if (Math.hypot(g.x - player.x, g.y - player.y) < 30) {
                currentExp += 20;
                if (currentExp >= nextExp) {
                    currentExp = 0;
                    playerLv++;
                    document.getElementById('player-lv').innerText = playerLv;
                }
                document.getElementById('exp-fill').style.width = (currentExp/nextExp*100) + '%';
                gems.splice(i, 1);
            }
        });
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 畫星星
        ctx.fillStyle = "white";
        stars.forEach(s => ctx.fillRect(s.x, s.y, s.size, s.size));

        // 畫玩家
        ctx.fillStyle = "#00f2ff";
        ctx.beginPath();
        ctx.moveTo(player.x, player.y - 15);
        ctx.lineTo(player.x + 15, player.y + 15);
        ctx.lineTo(player.x - 15, player.y + 15);
        ctx.fill();

        // 畫子彈
        ctx.fillStyle = "white";
        projectiles.forEach(p => ctx.fillRect(p.x - 2, p.y, 4, 10));

        // 畫敵人
        ctx.fillStyle = "#ff00ea";
        enemies.forEach(en => ctx.fillRect(en.x, en.y, 30, 30));

        // 畫經驗球
        ctx.fillStyle = "#f1c40f";
        gems.forEach(g => {
            ctx.beginPath();
            ctx.arc(g.x, g.y, 5, 0, Math.PI*2);
            ctx.fill();
        });

        requestAnimationFrame(() => {
            update();
            draw();
        });
    }

    document.getElementById('startBtn').addEventListener('click', () => {
        document.getElementById('overlay').style.display = 'none';
        gameActive = true;
        resize();
        draw();
    });

    resize();
</script>
</body>
</html>
