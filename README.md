<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - Game Edition</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.1/dist/confetti.browser.min.js"></script>
    <style>
        :root { --primary: #74b9ff; --accent: #ff7675; --bg: #2d3436; --card: rgba(53, 59, 72, 0.9); --text: #dfe6e9; }
        * { box-sizing: border-box; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background-color: var(--bg); color: var(--text); margin: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh; overflow: hidden; transition: background-color 0.3s ease; }
        
        /* 頂部工具列 */
        header { width: 100%; background: rgba(0,0,0,0.3); padding: 10px 20px; display: flex; justify-content: space-between; align-items: center; z-index: 100; }
        .tools { display: flex; gap: 15px; align-items: center; }
        
        /* 練習視窗 */
        .game-container { position: relative; width: 95%; max-width: 1000px; margin-top: 20px; display: grid; grid-template-columns: 1fr 300px; gap: 20px; height: 80vh; }
        .practice-window { position: relative; background: rgba(0,0,0,0.4); border-radius: 20px; border: 2px solid #555; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        
        #scrolling-content { position: absolute; width: 100%; padding: 250px 50px; transition: transform 0.2s cubic-bezier(0.1, 0.9, 0.2, 1); font-size: 32px; font-weight: bold; line-height: 2; letter-spacing: 3px; }
        
        #text-display { white-space: pre-wrap; color: #555; position: relative; }
        .char-correct { color: #fff; text-shadow: 0 0 10px var(--primary); }
        .char-wrong { color: var(--accent); background: rgba(255,118,117,0.3); }
        .cursor { border-left: 4px solid var(--primary); animation: blink 0.8s infinite; margin-left: 4px; }
        @keyframes blink { 50% { opacity: 0; } }

        /* Combo 數字特效 */
        #combo-pop { position: absolute; top: 10%; right: 10%; font-size: 80px; font-weight: 900; font-style: italic; color: var(--accent); pointer-events: none; opacity: 0; transform: scale(0.5); transition: all 0.1s; z-index: 50; }
        .pop-animation { opacity: 1 !important; transform: scale(1.2) rotate(-5deg) !important; }

        /* 側邊欄 */
        .sidebar { display: flex; flex-direction: column; gap: 15px; }
        .glass-card { background: var(--card); padding: 20px; border-radius: 15px; border: 1px solid rgba(255,255,255,0.1); backdrop-filter: blur(10px); }

        #invisible-input { position: fixed; top: -100px; opacity: 0; }
        canvas#sparks { position: fixed; top: 0; left: 0; pointer-events: none; z-index: 99; }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <canvas id="sparks"></canvas>

    <header>
        <div class="tools">
            <span style="font-weight: bold; color: var(--primary);">SingSangSung Game</span>
            <select id="song-selector" onchange="initSong()" style="padding: 5px; border-radius: 5px; background: #444; color: #fff;">
                <option value="">-- 選擇曲目 --</option>
                <option value="song2">This Is Me (EN)</option>
            </select>
            <label>背景色：<input type="color" id="bg-picker" value="#2d3436" oninput="changeBg(this.value)"></label>
        </div>
        <div id="progress-text" style="font-family: monospace;">READY... 0%</div>
    </header>

    <div id="combo-pop">0</div>

    <div class="game-container" id="layout" style="display:none;">
        <div class="practice-window">
            <div id="scrolling-content">
                <div id="text-display"></div>
            </div>
            <textarea id="invisible-input" spellcheck="false" autocomplete="off"></textarea>
        </div>

        <div class="sidebar">
            <div class="glass-card">
                <div style="color:var(--primary); font-size:12px; margin-bottom:5px;">VOCABULARY</div>
                <div id="hint-content">---</div>
            </div>
            <div class="glass-card">
                <div style="color:#a29bfe; font-size:12px; margin-bottom:5px;">TRANSLATION</div>
                <div id="trans-content" style="font-size:14px; opacity:0.8;"></div>
            </div>
        </div>
    </div>

    <script>
        const songLibrary = {
            "song2": {
                "lyrics": "I am not a stranger to the dark\nHide away they say\nCause we dont want your broken parts\nIve learned to be ashamed of all my scars\nRun away they say\nNo onell love you as you are\nBut I wont let them break me down to dust\nI know that theres a place for us\nFor we are glorious",
                "translation": "我不畏懼黑暗。他們說躲起來吧，我們不需要你那破碎的部分。我曾學會為疤痕感到羞恥...",
                "vocab": [{ "trigger": "stranger", "kk": "/ˈstreɪndʒər/", "note": "陌生人" }, { "trigger": "glorious", "kk": "/ˈɡlɔːriəs/", "note": "輝煌燦爛" }]
            }
        };

        let currentData = null, combo = 0, particles = [];
        const canvas = document.getElementById('sparks');
        const ctx = canvas.getContext('2d');

        // 初始化背景與畫布
        window.addEventListener('resize', () => { canvas.width = window.innerWidth; canvas.height = window.innerHeight; });
        canvas.width = window.innerWidth; canvas.height = window.innerHeight;

        function changeBg(color) { document.body.style.backgroundColor = color; }

        function initSong() {
            const val = document.getElementById('song-selector').value;
            currentData = songLibrary[val];
            if (!currentData) return;
            document.getElementById('layout').style.display = 'grid';
            document.getElementById('trans-content').innerText = currentData.translation;
            document.getElementById('invisible-input').value = "";
            render(0);
            document.getElementById('invisible-input').focus();
        }

        document.getElementById('invisible-input').addEventListener('input', (e) => {
            const val = e.target.value;
            const target = currentData.lyrics;
            const idx = val.length - 1;

            if (val[idx] === target[idx]) {
                combo++;
                createParticles(); // 產生火花
                showCombo();
                if (combo % 15 === 0) confetti({ particleCount: 50, spread: 60 });
            } else {
                combo = 0;
                hideCombo();
            }
            render(val.length);
            updateUI(val);
        });

        function render(inputLen) {
            const target = currentData.lyrics;
            const userIn = document.getElementById('invisible-input').value;
            let html = "";
            for (let i = 0; i < target.length; i++) {
                if (i < inputLen) {
                    html += `<span class="${userIn[i] === target[i] ? 'char-correct' : 'char-wrong'}" id="char-${i}">${target[i]}</span>`;
                } else if (i === inputLen) {
                    html += `<span id="char-${i}">${target[i]}</span><span class="cursor"></span>`;
                } else {
                    html += `<span id="char-${i}">${target[i]}</span>`;
                }
            }
            document.getElementById('text-display').innerHTML = html;
            const cur = document.getElementById(`char-${inputLen}`);
            if (cur) document.getElementById('scrolling-content').style.transform = `translateY(-${cur.offsetTop - 100}px)`;
        }

        function showCombo() {
            const el = document.getElementById('combo-pop');
            el.innerText = combo + " COMBO";
            el.classList.remove('pop-animation');
            void el.offsetWidth; // 觸發重繪
            el.classList.add('pop-animation');
        }

        function hideCombo() { document.getElementById('combo-pop').style.opacity = 0; }

        function updateUI(userInput) {
            const p = Math.floor((userInput.length / currentData.lyrics.length) * 100);
            document.getElementById('progress-text').innerText = `PROGRESS: ${p}%`;
        }

        // --- 火花粒子邏輯 ---
        function createParticles() {
            for (let i = 0; i < 8; i++) {
                particles.push({
                    x: window.innerWidth / 2 + (Math.random() - 0.5) * 400,
                    y: window.innerHeight / 2,
                    vx: (Math.random() - 0.5) * 10,
                    vy: (Math.random() - 0.5) * 10,
                    size: Math.random() * 5 + 2,
                    color: `hsl(${Math.random() * 360}, 100%, 70%)`,
                    life: 1.0
                });
            }
        }

        function animate() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            particles.forEach((p, i) => {
                p.x += p.vx; p.y += p.vy; p.life -= 0.02;
                ctx.fillStyle = p.color;
                ctx.globalAlpha = p.life;
                ctx.fillRect(p.x, p.y, p.size, p.size);
                if (p.life <= 0) particles.splice(i, 1);
            });
            requestAnimationFrame(animate);
        }
        animate();
    </script>
</body>
</html>
