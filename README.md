<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - Pro Edition</title>
    <style>
        :root { --primary: #74b9ff; --accent: #ff7675; --bg: #2d3436; --card: #353b48; --text: #dfe6e9; }
        * { box-sizing: border-box; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background-color: var(--bg); color: var(--text); margin: 0; overflow: hidden; display: flex; flex-direction: column; height: 100vh; transition: background-color 0.3s; }
        
        header { background: rgba(0,0,0,0.6); padding: 12px 25px; display: flex; justify-content: space-between; align-items: center; z-index: 100; box-shadow: 0 2px 10px rgba(0,0,0,0.3); }
        .controls { display: flex; gap: 15px; align-items: center; }
        
        .main-game { display: grid; grid-template-columns: 1fr 320px; gap: 25px; padding: 25px; flex: 1; height: calc(100vh - 70px); max-width: 1200px; margin: 0 auto; width: 100%; }
        
        /* 練習視窗優化 */
        .practice-window { position: relative; background: #1a1c1e; border-radius: 20px; border: 2px solid #444; overflow: hidden; box-shadow: inset 0 0 30px rgba(0,0,0,0.7); }
        
        #scroll-engine { 
            position: absolute; width: 100%; padding: 0 50px; 
            top: 50%; /* 初始位置在中央 */
            transition: transform 0.3s cubic-bezier(0.23, 1, 0.32, 1); 
            font-size: 34px; line-height: 2.2; letter-spacing: 2px;
        }
        
        #text-target { white-space: pre-wrap; color: #444; position: relative; }
        .char-ok { color: #fff; text-shadow: 0 0 12px var(--primary); }
        .char-no { color: var(--accent); background: rgba(255,118,117,0.2); border-radius: 6px; }
        .cursor { border-left: 3px solid var(--primary); animation: blink 0.8s infinite; display: inline-block; height: 1.1em; vertical-align: middle; margin-left: 2px; }
        @keyframes blink { 50% { opacity: 0; } }

        .sidebar { display: flex; flex-direction: column; gap: 20px; }
        .card { background: var(--card); padding: 25px; border-radius: 18px; border-left: 5px solid var(--primary); box-shadow: 0 8px 20px rgba(0,0,0,0.3); }

        #invisible-input { position: fixed; top: -100px; opacity: 0; }

        /* Emoji 噴發效果 */
        .emoji-particle { position: absolute; pointer-events: none; font-size: 30px; z-index: 1000; animation: emoji-fly 0.8s ease-out forwards; }
        @keyframes emoji-fly {
            0% { transform: translate(-50%, -50%) scale(0.5); opacity: 1; }
            100% { transform: translate(calc(-50% + var(--dx)), calc(-50% + var(--dy))) rotate(var(--dr)) scale(1.5); opacity: 0; }
        }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <header>
        <div class="controls">
            <span style="font-size: 20px; font-weight: 900; color: var(--primary); letter-spacing: 1px;">SingSangSung</span>
            <select id="song-select" onchange="loadSong()" style="background:#333; color:#fff; border:1px solid #555; padding:6px 12px; border-radius:8px; cursor:pointer;">
                <option value="">-- 選擇練習曲目 --</option>
                <option value="song2">This Is Me (Greatest Showman)</option>
            </select>
            <input type="color" id="color-picker" value="#2d3436" oninput="document.body.style.backgroundColor = this.value" style="width:35px; height:35px; border:none; padding:0; background:none; cursor:pointer;">
        </div>
        <div id="stat-display" style="font-family: 'Courier New', monospace; font-size: 18px; font-weight: bold; color: var(--primary);">Combo: 0 | 0%</div>
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
                <div style="font-size:13px; color:var(--primary); font-weight:bold; margin-bottom:10px; letter-spacing:1px;">VOCABULARY</div>
                <div id="vocab-hint" style="font-size:18px;">Ready to Start?</div>
            </div>
            <div class="card" style="border-left-color:#a29bfe;">
                <div style="font-size:13px; color:#a29bfe; font-weight:bold; margin-bottom:10px; letter-spacing:1px;">TRANSLATION</div>
                <div id="trans-hint" style="font-size:15px; line-height:1.6; opacity:0.9;"></div>
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

        const emojis = ['🐱', '🐱‍🚀', '🌈', '✨', '🥧', '🔥', '💎', '⭐', '🎈', '🍀', '🎯', '⚡'];
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
            
            // 捲動對焦邏輯優化：鎖定在中央
            const curEl = document.getElementById(`c-${val.length}`);
            if (curEl) {
                const scrollEngine = document.getElementById('scroll-engine');
                // 計算偏移量，讓當前字元位在容器的 0 點 (即 top:50% 的位置)
                scrollEngine.style.transform = `translateY(-${curEl.offsetTop}px)`;
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
            
            // 精準定位在游標處
            particle.style.left = (rect.left - winRect.left) + "px";
            particle.style.top = (rect.top - winRect.top) + "px";
            
            // 隨機噴發參數
            particle.style.setProperty('--dx', (Math.random() - 0.5) * 250 + "px");
            particle.style.setProperty('--dy', (Math.random() * -200 - 100) + "px");
            particle.style.setProperty('--dr', (Math.random() * 360) + "deg");
            
            document.getElementById('window-root').appendChild(particle);
            setTimeout(() => particle.remove(), 800);
        }

        function updateStats(val) {
            const p = Math.floor((val.length / current.lyrics.length) * 100);
            document.getElementById('stat-display').innerText = `Combo: ${combo} | ${p}%`;
            
            // 檢查單字提示
            const words = val.split(/[\s\n]+/);
            const lastWord = words[words.length - 1].toLowerCase();
            const v = current.vocab.find(x => x.t === lastWord);
            if (v) {
                document.getElementById('vocab-hint').innerHTML = `<b style="color:var(--primary)">${v.t}</b> <small>${v.k}</small><br><span style="font-size:14px; opacity:0.8">${v.n}</span>`;
            }
        }
    </script>
</body>
</html>
