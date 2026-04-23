<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>極限跳躍 - Sam 生日快樂</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #020617;
            font-family: 'PingFang TC', 'Microsoft JhengHei', sans-serif;
            touch-action: none;
            user-select: none;
        }
        canvas {
            display: block;
        }
        .controls-container {
            position: absolute;
            bottom: 40px;
            left: 0;
            right: 0;
            height: 160px;
            z-index: 100;
            pointer-events: none;
        }
        .joy-container {
            position: absolute;
            left: 40px;
            bottom: 20px;
            width: 120px;
            height: 120px;
            background: rgba(255, 255, 255, 0.05);
            border: 2px solid rgba(255, 255, 255, 0.1);
            border-radius: 50%;
            pointer-events: auto;
            backdrop-filter: blur(8px);
        }
        .joy-handle {
            width: 50px;
            height: 50px;
            background: #fff;
            border-radius: 50%;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            box-shadow: 0 8px 20px rgba(0,0,0,0.6);
            pointer-events: none;
        }
        .jump-container {
            position: absolute;
            right: 40px;
            bottom: 20px;
            width: 110px;
            height: 110px;
            background: linear-gradient(135deg, #f472b6 0%, #db2777 100%);
            border: 4px solid #fff;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: 900;
            font-size: 1.4rem;
            pointer-events: auto;
            box-shadow: 0 8px 0 #831843;
            transition: transform 0.05s;
        }
        .jump-container:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #831843;
        }
        #overlay {
            position: absolute;
            inset: 0;
            background: rgba(2, 6, 23, 0.9);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            z-index: 200;
            padding: 20px;
            text-align: center;
        }
        .celebration-text {
            background: linear-gradient(to right, #fbbf24, #f472b6, #60a5fa);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            font-weight: 900;
            text-shadow: 0 0 30px rgba(244, 114, 182, 0.5);
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.1); }
        }
    </style>
</head>
<body>

    <div class="controls-container">
        <div id="joy-base" class="joy-container">
            <div id="joy-handle" class="joy-handle"></div>
        </div>
        <div id="jump-btn" class="jump-container">JUMP</div>
    </div>

    <div id="overlay">
        <h2 id="msg-top" class="text-3xl font-black mb-2 tracking-widest"></h2>
        <h1 id="birthday-msg" class="text-6xl font-black mb-8 celebration-text"></h1>
        <p id="msg-bottom" class="text-xl opacity-80 mb-10 font-bold"></p>
        <button id="reset-btn" class="bg-gradient-to-r from-pink-500 to-rose-600 px-16 py-4 rounded-full text-2xl font-bold shadow-2xl active:scale-95 border-2 border-white">
            再應戰一次
        </button>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const joyBase = document.getElementById('joy-base');
        const joyHandle = document.getElementById('joy-handle');
        const jumpBtn = document.getElementById('jump-btn');
        const overlay = document.getElementById('overlay');
        const msgTop = document.getElementById('msg-top');
        const birthdayMsg = document.getElementById('birthday-msg');
        const msgBottom = document.getElementById('msg-bottom');
        const resetBtn = document.getElementById('reset-btn');

        // 物理配置
        const GRAVITY = 0.65;
        const JUMP_FORCE = -13.5;
        const SPEED = 5.5;
        
        let isRunning = true;
        let isWin = false;
        let inputX = 0;
        let platforms = [];
        let goal = { x: 0, y: 0, w: 8, h: 80 };
        let joystickTouchId = null;

        // 煙火系統
        let fireworks = [];
        let particles = [];

        class Firework {
            constructor() {
                this.x = Math.random() * canvas.width;
                this.y = canvas.height;
                this.targetY = Math.random() * (canvas.height * 0.5);
                this.speed = 8 + Math.random() * 5;
                this.color = `hsl(${Math.random() * 360}, 100%, 60%)`;
                this.alive = true;
            }
            update() {
                this.y -= this.speed;
                if (this.y <= this.targetY) {
                    this.explode();
                    this.alive = false;
                }
            }
            draw() {
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, 3, 0, Math.PI * 2);
                ctx.fill();
            }
            explode() {
                for (let i = 0; i < 50; i++) {
                    particles.push(new Particle(this.x, this.y, this.color));
                }
            }
        }

        class Particle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.color = color;
                const angle = Math.random() * Math.PI * 2;
                const force = Math.random() * 6;
                this.vx = Math.cos(angle) * force;
                this.vy = Math.sin(angle) * force;
                this.alpha = 1;
                this.decay = 0.015 + Math.random() * 0.02;
            }
            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.vy += 0.1;
                this.alpha -= this.decay;
            }
            draw() {
                ctx.save();
                ctx.globalAlpha = this.alpha;
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, 2, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }
        }

        const player = {
            x: 0, y: 0, w: 32, h: 48, vx: 0, vy: 0, grounded: false,
            color: '#f472b6',
            update() {
                this.vx = inputX * SPEED;
                this.vy += GRAVITY;
                this.x += this.vx;
                this.resolveCollision('x');
                this.y += this.vy;
                this.resolveCollision('y');
                if (this.x < 0) this.x = 0;
                if (this.x + this.w > canvas.width) this.x = canvas.width - this.w;
                if (this.y > canvas.height) finish(false);
            },
            resolveCollision(axis) {
                this.grounded = false;
                for (let p of platforms) {
                    if (this.x < p.x + p.w && this.x + this.w > p.x &&
                        this.y < p.y + p.h && this.y + this.h > p.y) {
                        if (axis === 'x') {
                            if (this.vx > 0) this.x = p.x - this.w;
                            if (this.vx < 0) this.x = p.x + p.w;
                        } else {
                            if (this.vy > 0) {
                                this.y = p.y - this.h;
                                this.vy = 0;
                                this.grounded = true;
                            } else if (this.vy < 0) {
                                this.y = p.y + p.h;
                                this.vy = 0;
                            }
                        }
                    }
                }
            },
            draw() {
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.roundRect(this.x, this.y, this.w, this.h, 8);
                ctx.fill();
                ctx.fillStyle = '#fff';
                const eyeX = inputX >= 0 ? this.x + this.w - 10 : this.x + 2;
                ctx.fillRect(eyeX, this.y + 12, 8, 8);
            }
        };

        function init() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            const w = canvas.width;
            const h = canvas.height;
            platforms = [
                { x: 30, y: h - 120, w: 180, h: 120, type: 'main' },
                { x: 250, y: h - 210, w: 80, h: 25, type: 'box' },
                { x: 100, y: h - 305, w: 80, h: 25, type: 'box' },
                { x: 280, y: h - 400, w: 80, h: 25, type: 'box' },
                { x: 120, y: h - 495, w: 80, h: 25, type: 'box' },
                { x: w * 0.45, y: h - 550, w: 120, h: 25, type: 'box' },
                { x: w - 280, y: h * 0.42, w: 70, h: 25, type: 'box' },
                { x: w - 160, y: h * 0.25, w: 160, h: 30, type: 'main' }
            ];
            goal = { x: w - 70, y: h * 0.25 - 80, w: 8, h: 80 };
            player.x = 60;
            player.y = h - 180;
            player.vx = 0;
            player.vy = 0;
            fireworks = [];
            particles = [];
            isWin = false;
        }

        joyBase.addEventListener('touchstart', (e) => {
            e.preventDefault();
            const touch = e.changedTouches[0];
            joystickTouchId = touch.identifier;
            updateJoystick(touch);
        }, { passive: false });
        window.addEventListener('touchmove', (e) => {
            for (let i = 0; i < e.changedTouches.length; i++) {
                const touch = e.changedTouches[i];
                if (touch.identifier === joystickTouchId) updateJoystick(touch);
            }
        }, { passive: false });
        window.addEventListener('touchend', (e) => {
            for (let i = 0; i < e.changedTouches.length; i++) {
                if (e.changedTouches[i].identifier === joystickTouchId) {
                    joystickTouchId = null;
                    inputX = 0;
                    joyHandle.style.transform = 'translate(-50%, -50%)';
                }
            }
        });
        function updateJoystick(touch) {
            const rect = joyBase.getBoundingClientRect();
            const centerX = rect.left + rect.width / 2;
            const dist = touch.clientX - centerX;
            const limit = rect.width / 2;
            const clamped = Math.max(-limit, Math.min(limit, dist));
            joyHandle.style.transform = `translate(calc(-50% + ${clamped}px), -50%)`;
            inputX = clamped / limit;
        }
        jumpBtn.addEventListener('touchstart', (e) => {
            e.preventDefault();
            if (player.grounded) player.vy = JUMP_FORCE;
        }, { passive: false });
        jumpBtn.addEventListener('mousedown', () => { if (player.grounded) player.vy = JUMP_FORCE; });

        function finish(win) {
            isRunning = false;
            isWin = win;
            overlay.style.display = 'flex';
            if (win) {
                msgTop.innerText = "✨ CHALLENGE COMPLETE ✨";
                birthdayMsg.innerText = "祝 Sam 生日快樂！";
                msgBottom.innerText = "你是這座高峰的征服者！";
                msgTop.style.color = "#fbbf24";
            } else {
                msgTop.innerText = "❌ 挑戰失敗";
                birthdayMsg.innerText = "不要放棄，Sam！";
                msgBottom.innerText = "再試一次吧！";
                msgTop.style.color = "#f43f5e";
            }
        }

        resetBtn.onclick = () => {
            init();
            isRunning = true;
            overlay.style.display = 'none';
            gameLoop();
        };

        function render() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            if (isWin) {
                if (Math.random() < 0.05) fireworks.push(new Firework());
                fireworks = fireworks.filter(f => f.alive);
                fireworks.forEach(f => { f.update(); f.draw(); });
                particles = particles.filter(p => p.alpha > 0);
                particles.forEach(p => { p.update(); p.draw(); });
            }

            ctx.fillStyle = 'rgba(244, 114, 182, 0.05)';
            for(let i=0; i<canvas.width; i+=100) {
                for(let j=0; j<canvas.height; j+=100) {
                    ctx.beginPath();
                    ctx.arc(i + Math.cos(Date.now()*0.001 + j)*10, j, 1, 0, Math.PI*2);
                    ctx.fill();
                }
            }

            platforms.forEach(p => {
                ctx.fillStyle = p.type === 'main' ? '#0f172a' : '#1e293b';
                ctx.beginPath();
                ctx.roundRect(p.x, p.y, p.w, p.h, 6);
                ctx.fill();
                ctx.fillStyle = '#f472b6';
                ctx.fillRect(p.x, p.y, p.w, 4);
            });

            ctx.fillStyle = '#fbbf24';
            ctx.fillRect(goal.x, goal.y, goal.w, goal.h);
            ctx.beginPath();
            ctx.moveTo(goal.x + goal.w, goal.y);
            ctx.lineTo(goal.x + goal.w + 35, goal.y + 25);
            ctx.lineTo(goal.x + goal.w, goal.y + 50);
            ctx.fill();

            player.draw();
        }

        function gameLoop() {
            if (!isRunning && !isWin) return;
            if (isRunning) player.update();
            render();
            if (isRunning && player.x + player.w > goal.x && 
                player.x < goal.x + 40 && 
                player.y < goal.y + goal.h && 
                player.y + player.h > goal.y) {
                finish(true);
            }
            requestAnimationFrame(gameLoop);
        }

        window.addEventListener('resize', init);
        init();
        gameLoop();
    </script>
</body>
</html>
