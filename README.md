<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>加油跑馬燈 - 張登淋加油</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes marquee {
            0% { transform: translateX(100vw); }
            100% { transform: translateX(-100%); }
        }

        .marquee-content {
            display: inline-block;
            white-space: nowrap;
            animation: marquee var(--duration, 8s) linear infinite;
            will-change: transform;
        }

        .glow-text {
            text-shadow: 0 0 10px rgba(255, 255, 255, 0.8),
                         0 0 20px currentColor,
                         0 0 30px currentColor;
        }

        /* 隱藏控制介面的動畫 */
        .controls-panel {
            transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.3s;
        }

        .controls-hidden .controls-panel {
            transform: translateY(100%);
            opacity: 0;
            pointer-events: none;
        }

        body {
            background-color: #000;
            overflow: hidden;
            touch-action: manipulation;
            font-family: sans-serif;
        }
    </style>
</head>
<body class="h-screen flex flex-col items-center justify-center text-white">

    <!-- 跑馬燈顯示區域 -->
    <div id="displayArea" class="flex-grow w-full flex items-center overflow-hidden cursor-pointer" title="點擊任何處顯示設定">
        <div id="marqueeWrapper" class="w-full">
            <span id="marqueeText" class="marquee-content font-black glow-text leading-none">
                張登淋加油
            </span>
        </div>
    </div>

    <!-- 控制面板 -->
    <div id="controls" class="controls-panel fixed bottom-0 left-0 right-0 p-6 bg-gray-900/95 backdrop-blur-xl border-t border-gray-700 z-50">
        <div class="max-w-4xl mx-auto grid grid-cols-1 md:grid-cols-2 gap-6">
            
            <!-- 文字輸入與顏色 -->
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium mb-1 text-gray-400">加油文字</label>
                    <input type="text" id="textInput" value="張登淋加油" 
                           class="w-full bg-gray-800 border border-gray-600 rounded-lg px-4 py-3 text-xl focus:ring-2 focus:ring-yellow-500 outline-none transition-all">
                </div>
                <div class="flex gap-4">
                    <div class="flex-1">
                        <label class="block text-sm font-medium mb-1 text-gray-400">文字顏色</label>
                        <input type="color" id="colorInput" value="#ffff00" class="w-full h-12 bg-gray-800 border border-gray-600 rounded-lg p-1 cursor-pointer">
                    </div>
                    <div class="flex-1">
                        <label class="block text-sm font-medium mb-1 text-gray-400">背景顏色</label>
                        <input type="color" id="bgColorInput" value="#000000" class="w-full h-12 bg-gray-800 border border-gray-600 rounded-lg p-1 cursor-pointer">
                    </div>
                </div>
            </div>

            <!-- 速度與大小 -->
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium mb-1 flex justify-between text-gray-400">
                        字體大小 <span id="sizeValue" class="text-white">55vh</span>
                    </label>
                    <input type="range" id="sizeInput" min="10" max="95" value="55" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-yellow-500">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1 flex justify-between text-gray-400">
                        滾動速度 <span id="speedValue" class="text-white">快</span>
                    </label>
                    <input type="range" id="speedInput" min="1" max="25" value="18" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-yellow-500">
                </div>
            </div>
        </div>

        <!-- 功能按鈕 -->
        <div class="mt-8 flex flex-col items-center gap-3">
            <button id="toggleBtn" class="bg-yellow-500 hover:bg-yellow-400 text-black px-10 py-3 rounded-full font-bold text-lg transition-transform active:scale-95 shadow-lg shadow-yellow-500/20">
                進入應援模式 (隱藏介面)
            </button>
            <p class="text-gray-500 text-sm italic">點擊畫面上方任何區域可喚回設定選單</p>
        </div>
    </div>

    <script>
        const textInput = document.getElementById('textInput');
        const colorInput = document.getElementById('colorInput');
        const bgColorInput = document.getElementById('bgColorInput');
        const sizeInput = document.getElementById('sizeInput');
        const speedInput = document.getElementById('speedInput');
        
        const marqueeText = document.getElementById('marqueeText');
        const sizeValueDisplay = document.getElementById('sizeValue');
        const speedValueDisplay = document.getElementById('speedValue');
        const body = document.body;
        const controls = document.getElementById('controls');
        const displayArea = document.getElementById('displayArea');
        const toggleBtn = document.getElementById('toggleBtn');

        // 更新文字內容
        textInput.addEventListener('input', () => {
            marqueeText.innerText = textInput.value || ' ';
        });

        // 更新文字顏色
        colorInput.addEventListener('input', () => {
            marqueeText.style.color = colorInput.value;
        });

        // 更新背景顏色
        bgColorInput.addEventListener('input', () => {
            body.style.backgroundColor = bgColorInput.value;
            // 讓控制面板保持半透明對比
            controls.style.backgroundColor = bgColorInput.value === '#000000' ? 'rgba(17, 24, 39, 0.95)' : bgColorInput.value + 'F2';
        });

        // 更新大小
        sizeInput.addEventListener('input', () => {
            const size = sizeInput.value + 'vh';
            marqueeText.style.fontSize = size;
            sizeValueDisplay.innerText = size;
        });

        // 更新速度
        speedInput.addEventListener('input', () => {
            const val = parseInt(speedInput.value);
            let label = "中";
            if (val < 8) label = "極慢";
            else if (val < 13) label = "慢";
            else if (val < 19) label = "快";
            else label = "極快";
            
            speedValueDisplay.innerText = label;
            
            // 速度換算：30s 到 1s
            const duration = 31 - val;
            marqueeText.style.setProperty('--duration', `${duration}s`);
        });

        // 切換控制面板
        function toggleControls() {
            document.body.classList.toggle('controls-hidden');
        }

        toggleBtn.addEventListener('click', (e) => {
            e.stopPropagation();
            toggleControls();
        });

        displayArea.addEventListener('click', () => {
            if (document.body.classList.contains('controls-hidden')) {
                toggleControls();
            }
        });

        // 初始化樣式設定
        function init() {
            marqueeText.style.color = colorInput.value;
            marqueeText.style.fontSize = sizeInput.value + 'vh';
            const duration = 31 - speedInput.value;
            marqueeText.style.setProperty('--duration', `${duration}s`);
            body.style.backgroundColor = bgColorInput.value;
        }

        window.onload = init;
    </script>
</body>
</html>
