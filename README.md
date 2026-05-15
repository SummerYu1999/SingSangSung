<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - The Greatest Stage</title>
    <style>
        :root { 
            --primary: #74b9ff; 
            --accent: #ff7675; 
            --highlight: #ffeaa7; 
            --glass: rgba(45, 52, 54, 0.85); /* 磨砂玻璃色調 */
        }
        * { box-sizing: border-box; }
        
        body { 
            font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; 
            margin: 0; 
            overflow: hidden; 
            display: flex; 
            flex-direction: column; 
            height: 100vh;
            /* 設定電影劇照為背景 */
            background: linear-gradient(rgba(0,0,0,0.6), rgba(0,0,0,0.6)), 
                        url('https://i.guim.co.uk/img/media/69c18167301f85e8c94ca627f9d0cec61c4edef9/339_141_1331_799/master/1331.jpg?width=700&dpr=2&s=none&crop=none');
            background-size: cover;
            background-position: center;
            transition: transform 0.1s ease; /* 用於背景震動 */
        }
        
        header { background: rgba(0,0,0,0.8); padding: 12px 25px; display: flex; justify-content: space-between; align-items: center; z-index: 100; border-bottom: 1px solid rgba(255,255,255,0.1); }
        .controls { display: flex; gap: 15px; align-items: center; }
        
        .main-game { 
            display: grid; 
            grid-template-columns: 1fr 300px; 
            gap: 20px; 
            padding: 30px; 
            flex: 1; 
            max-width: 1400px; 
            margin: 0 auto; 
            width: 100%; 
        }

        /* 磨砂玻璃效果的卡片 */
        .practice-window { 
            position: relative; 
            overflow: hidden; 
            background: var(--glass); 
            backdrop-filter: blur(10px);
            border-radius: 24px; 
            padding: 60px 50px; 
            border: 1px solid rgba(255,255,255,0.1);
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        }

        .sidebar { display: flex; flex-direction: column; gap: 20px; }
        .card { 
            background: var(--glass); 
            backdrop-filter: blur(10px);
            padding: 20px; 
            border-radius: 18px; 
            border-left: 6px solid var(--primary); 
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
        }

        #text-target { 
            white-space: pre-wrap; 
            color: rgba(255,255,255,0.2); /* 未打字體轉淡 */
            font-size: 34px; 
            line-height: 1.8; 
            font-weight: 600; 
        }
        
        .char-ok { color: #fff; text-shadow: 0 0 15px var(--primary), 0 0 5px #fff; }
        .char-no { color: var(--accent); background: rgba(255,118,117,0.3); border-radius: 4px; }
        
        .cursor { 
            border-left: 3px solid var(--primary); 
            animation: blink 0.8s infinite; 
            display: inline-block; 
            height: 1.2em; 
            vertical-align: middle; 
            box-shadow: 0 0 15px var(--primary);
        }

        /* 更活潑的 Emoji 動畫 */
        .emoji-particle {
            position: absolute;
            pointer-events: none;
            z-index: 1000;
            font-size: 32px;
            animation: lively-fountain var(--duration) forwards cubic-bezier(0.15, 0.85, 0.35, 1.2);
        }

        @keyframes lively-fountain {
            0% { transform: translate(0, 0) scale(0) rotate(0deg); opacity: 0; }
            20% { opacity: 1; transform: translate(var(--mx), -40px) scale(1.4) rotate(var(--r1)); }
            100% { transform: translate(var(--dx), var(--dy)) scale(0.6) rotate(var(--r2)); opacity: 0; }
        }

        /* 螢幕震動動畫 */
        @keyframes shake {
            0% { transform: translate(1px, 1px); }
            20% { transform: translate(-2px, 0px); }
            40% { transform: translate(2px, 1px); }
            60% { transform: translate(-1px, -1px); }
            80% { transform: translate(1px, 2px); }
            100% { transform: translate(0px, 0px); }
        }
        .shake-effect { animation: shake 0.3s ease-in-out; }

        #invisible-input { position: fixed; top: -100px; opacity: 0; }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <header>
        <div class="controls">
            <span style="font-size: 24px; font-weight: 900; color: #fff; text-shadow: 0 0 10px var(--primary);">SingSangSung</span>
            <select id="song-select" onchange="loadSong()" style="background:rgba(255,255,255,0.1); color:#fff; border:1px solid #777; border-radius:8px; padding:6px 15px;">
                <option value="">-- 舞台準備 --</option>
                <option value="song2">This Is Me (Greatest Showman)</option>
            </select>
        </div>
        <div id="stat-display" style="font-family: monospace; color: var(--highlight); font-size: 20px;">Combo: 0 | 0%</div>
    </header>

    <div class="main-game" id="game-view" style="display:none;">
        <div class="practice-window" id="window-root">
            <div id="scroll-engine">
                <div id="text-target"></div>
            </div>
            <textarea id="invisible-input" spellcheck="false"></textarea>
        </div>

        <div class="sidebar">
            <div class="card" id="card-vocab">
                <div id="vocab-hint"></div>
            </div>
            <div class="card" id="card-trans" style="border-left-color:#a29bfe; color: #eee;">
                <div id="trans-hint"></div>
            </div>
        </div>
    </div>

    <script>
    // 此處保留你原本的 songData 與邏輯，僅修改 spawnEffect 與 input 監聽
    const songData = { "song2": { /* ...原本的歌詞數據... */ } }; 
    // (為了節省篇幅，請延用上一版本完整 songData)

    function spawnEffect(text, isOh = false) {
        const cursor = document.querySelector('.cursor');
        if (!cursor) return;
        const rect = cursor.getBoundingClientRect();
        const winRoot = document.getElementById('window-root');
        const winRect = winRoot.getBoundingClientRect();
        
        const el = document.createElement('div');
        el.className = 'emoji-particle';
        el.innerText = text;
        
        // 隨機化物理參數，讓動態更自然
        const duration = isOh ? 2.0 : (0.8 + Math.random() * 0.7);
        const midX = (Math.random() - 0.5) * 60; // 中途偏移
        const dx = (Math.random() - 0.5) * 300; // 最終偏移
        const dy = -150 - (Math.random() * 200);
        const r1 = (Math.random() - 0.5) * 90 + "deg"; // 中途旋轉
        const r2 = (Math.random() - 0.5) * 360 + "deg"; // 最終旋轉
        
        el.style.setProperty('--duration', `${duration}s`);
        el.style.setProperty('--mx', `${midX}px`);
        el.style.setProperty('--dx', `${dx}px`);
        el.style.setProperty('--dy', `${dy}px`);
        el.style.setProperty('--r1', r1);
        el.style.setProperty('--r2', r2);
        
        el.style.left = (rect.left - winRect.left) + "px";
        el.style.top = (rect.top - winRect.top) + "px";
        
        if (isOh) {
            el.style.fontSize = "45px";
            el.style.filter = "drop-shadow(0 0 20px gold)";
            // 觸發螢幕震動
            document.body.classList.add('shake-effect');
            setTimeout(() => document.body.classList.remove('shake-effect'), 300);
        }
        
        winRoot.appendChild(el);
        setTimeout(() => el.remove(), duration * 1000);
    }

    // 更新 input 觸發點
    document.getElementById('invisible-input').addEventListener('input', (e) => {
        const val = e.target.value;
        render();
        updateStats();
        
        const lastWord = val.split(" ").pop().toLowerCase().replace(/[.,!]/g, '');
        let lineIdx = lineThresholds.findIndex(t => val.length < t);
        if (lineIdx === -1) lineIdx = current.lyrics.length - 1;

        const bv = current.backingVocals.find(b => b.trigger === lastWord && b.index === lineIdx);
        
        if (bv) {
            for(let i=0; i<5; i++) setTimeout(() => spawnEffect(bv.text, true), i * 100);
        } else if (e.data) {
            const list = emojiMap[lastWord] || genericEmojis;
            spawnEffect(list[Math.floor(Math.random() * list.length)]);
        }
    });

    // ... 其餘 render, loadSong, updateStats 函數請維持上一版 ...
    </script>
</body>
</html>
