<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - Emoji Power</title>
    <style>
        :root { --primary: #74b9ff; --accent: #ff7675; --bg: #2d3436; --card: #353b48; --text: #dfe6e9; }
        * { box-sizing: border-box; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background-color: var(--bg); color: var(--text); margin: 0; overflow: hidden; display: flex; flex-direction: column; height: 100vh; transition: background-color 0.3s; }
        
        /* 頂部工具列 */
        header { background: rgba(0,0,0,0.5); padding: 10px 25px; display: flex; justify-content: space-between; align-items: center; z-index: 100; border-bottom: 1px solid #444; }
        .controls { display: flex; gap: 20px; align-items: center; }
        
        /* 遊戲主容器 */
        .main-game { display: grid; grid-template-columns: 1fr 300px; gap: 20px; padding: 20px; flex: 1; height: calc(100vh - 60px); }
        
        /* 練習視窗 */
        .practice-window { position: relative; background: #1e2124; border-radius: 15px; border: 2px solid #4b5563; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        #scroll-engine { position: absolute; width: 100%; padding: 200px 50px; transition: transform 0.25s ease-out; font-size: 32px; line-height: 2; letter-spacing: 2px; }
        #text-target { white-space: pre-wrap; color: #555; position: relative; }
        
        /* 字體顏色 */
        .char-ok { color: #fff; text-shadow: 0 0 8px var(--primary); }
        .char-no { color: var(--accent); background: rgba(255,118,117,0.2); border-radius: 4px; }
        .cursor { border-left: 3px solid var(--primary); animation: blink 0.8s infinite; display: inline-block; height: 1em; vertical-align: middle; }
        @keyframes blink { 50% { opacity: 0; } }

        /* 側邊資訊 */
        .sidebar { display: flex; flex-direction: column; gap: 15px; }
        .card { background: var(--card); padding: 20px; border-radius: 12px; border-left: 4px solid var(--primary); box-shadow: 0 4px 10px rgba(0,0,0,0.2); }

        #invisible-input { position: fixed; top: -100px; opacity: 0; }

        /* Emoji 噴發動畫 */
        .emoji-particle { position: absolute; pointer-events: none; font-size: 24px; z-index: 999; animation: emoji-fly 1s ease-out forwards; }
        @keyframes emoji-fly {
            0% { transform: translate(0, 0) scale(1); opacity: 1; }
            100% { transform: translate(var(--x), var(--y)) rotate(var(--r)) scale(2); opacity: 0; }
        }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <header>
        <div class="controls">
            <b style="color:var(--primary)">SingSangSung</b>
            <select id="song-select" onchange="loadSong()" style="background:#444; color:#fff; border:none; padding:5px 10px; border-radius:4px;">
                <option value="">-- 選擇曲目 --</option>
                <option value="song2">This Is Me (The Greatest Showman)</option>
            </select>
            <label>背景：<input type="color" id="color-picker" value="#2d3436" oninput="document.body.style.backgroundColor = this.value"></label>
        </div>
        <div id="stat-display" style="font-family: monospace; font-weight: bold; color: var(--primary);">Combo: 0 | 0%</div>
    </header>

    <div class="main-game" id="game-view" style="display:none;">
        <div class="practice-window" id="window-root">
            <div id="scroll-engine">
                <div id="text-target"></div>
            </div>
            <textarea id="invisible-input" spellcheck="false" autocomplete="off"></textarea>
        </div>

        <div class="sidebar">
            <div class="card">
                <div style="font-size:12px; color:var(--primary); margin-bottom:8px;">VOCABULARY</div>
                <div id="vocab-hint">點擊選擇曲目開始</div>
            </div>
            <div class="card" style="border-left-color:#a29bfe;">
                <div style="font-size:12px; color:#a29bfe; margin-bottom:8px;">TRANSLATION</div>
                <div id="trans-hint" style="font-size:14px; opacity:0.8;"></div>
            </div>
        </div>
    </div>

    <script>
        const songData = {
            "song2": {
                "lyrics": "I am not a stranger to the dark\nHide away they say\nCause we dont want your broken parts\nIve learned to be ashamed of all my scars\nRun away they say\nNo onell love you as you are\nBut I wont let them break me down to dust\nI know that theres a place for us\nFor we are glorious",
                "translation": "我不畏懼黑暗。他們說躲起來吧，我們不需要你那破碎的部分。我曾學會為疤痕感到羞恥...",
                "vocab": [{ "t": "stranger", "k": "/ˈstreɪndʒər/", "n": "陌生人" }, { "t": "glorious", "k": "/ˈɡlɔːriəs/", "n": "輝煌燦爛" }]
            }
        };

        const emojis = ['🐱', '🐱‍🚀', '🌈', '✨', '🥧', '🔥', '💎', '⭐', '🎈', '🍀'];
        let current = null, combo = 0;

        function loadSong() {
            const key = document.getElementById('song-select').value;
            current = songData[key];
            if (!current) return;
            document.getElementById('game-view').style.display = 'grid';
            document.getElementById('trans-hint').innerText = current.translation;
            document.getElementById('invisible-input').value = "";
            render();
            document.getElementById('invisible-input').focus();
        }

        document.getElementById('invisible-input').addEventListener('input', (e) => {
            const val = e.target.value;
            const target = current.lyrics;
            const i = val.length - 1;

            if (val[i] === target[i]) {
                combo++;
                spawnEmoji();
            } else {
                combo = 0;
            }
            render();
            updateStats(val);
        });

        function render() {
            const val = document.getElementById('invisible-input').value;
            const target = current.lyrics;
            let html = "";
            for (let i = 0; i < target.length; i++) {
                if (i < val.length) {
                    html += `<span class="${val[i] === target[i] ? 'char-ok' : 'char-no'}" id="c-${i}">${target[i]}</span>`;
                } else if (i === val.length) {
                    html += `<span id="c-${i}">${target[i]}</span><span class="cursor" id="current-cursor"></span>`;
                } else {
                    html += `<span id="c-${i}">${target[i]}</span>`;
                }
            }
            document.getElementById('text-target').innerHTML = html;
            
            const curEl = document.getElementById(`c-${val.length}`);
            if (curEl) {
                document.getElementById('scroll-engine').style.transform = `translateY(-${curEl.offsetTop - 100}px)`;
            }
        }

        function spawnEmoji() {
            const cursor = document.getElementById('current-cursor');
            if (!cursor) return;
            
            const rect = cursor.getBoundingClientRect();
            const winRect = document.getElementById('window-root').getBoundingClientRect();
            
            const particle = document.createElement('div');
            particle.className = 'emoji-particle';
            particle.innerText = emojis[Math.floor(Math.random() * emojis.length)];
            
            // 定位在游標位置 (相對於練習視窗)
            particle.style.left = (rect.left - winRect.left) + "px";
            particle.style.top = (rect.top - winRect.top) + "px";
            
            // 隨機噴發方向
            particle.style.setProperty('--x', (Math.random() - 0.5) * 300 + "px");
            particle.style.setProperty('--y', (Math.random() * -300 - 50) + "px");
            particle.style.setProperty('--r', (Math.random() * 360) + "deg");
            
            document.getElementById('window-root').appendChild(particle);
            setTimeout(() => particle.remove(), 1000);
        }

        function updateStats(val) {
            const p = Math.floor((val.length / current.lyrics.length) * 100);
            document.getElementById('stat-display').innerText = `Combo: ${combo} | ${p}%`;
        }
    </script>
</body>
</html>
