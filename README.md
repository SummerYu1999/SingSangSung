<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - The Full Stage</title>
    <style>
        :root { 
            --primary: #74b9ff; 
            --accent: #ff7675; 
            --bg: #2d3436; 
            --card: #353b48; 
            --text: #dfe6e9; 
            --highlight: #ffeaa7; 
        }
        * { box-sizing: border-box; }
        body { 
            font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; 
            background-color: var(--bg); 
            color: var(--text); 
            margin: 0; 
            overflow: hidden; 
            display: flex; 
            flex-direction: column; 
            height: 100vh;
            transition: background 0.8s ease;
        }

        /* 專屬背景樣式：透過 Class 控制 */
        body.this-is-me-bg {
            background: linear-gradient(rgba(0,0,0,0.7), rgba(0,0,0,0.7)), 
                        url('https://i.guim.co.uk/img/media/69c18167301f85e8c94ca627f9d0cec61c4edef9/339_141_1331_799/master/1331.jpg?width=1200&dpr=1&s=none&crop=none');
            background-size: cover;
            background-position: center;
        }
        
        header { background: rgba(0,0,0,0.8); padding: 12px 25px; display: flex; justify-content: space-between; align-items: center; z-index: 100; border-bottom: 1px solid rgba(255,255,255,0.1); }
        .controls { display: flex; gap: 15px; align-items: center; }
        
        .main-game { 
            display: grid; 
            grid-template-columns: 1fr 280px; 
            gap: 20px; 
            padding: 20px; 
            flex: 1; 
            max-width: 1400px; 
            margin: 0 auto; 
            width: 100%; 
            position: relative;
        }

        .practice-window { 
            position: relative; 
            overflow: hidden; 
            background: rgba(0,0,0,0.4); 
            backdrop-filter: blur(8px);
            border-radius: 20px; 
            padding: 60px 40px; 
            height: 100%; 
            box-shadow: inset 0 0 30px rgba(0,0,0,0.5);
            border: 1px solid rgba(255,255,255,0.1);
        }

        .sidebar { display: flex; flex-direction: column; gap: 15px; }
        .card { 
            background: rgba(53, 59, 72, 0.9); 
            padding: 18px; 
            border-radius: 14px; 
            border-left: 5px solid var(--primary); 
            box-shadow: 0 4px 15px rgba(0,0,0,0.3); 
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); 
        }
        .card.hidden { opacity: 0; transform: translateX(20px); }

        #text-target { white-space: pre-wrap; color: rgba(255,255,255,0.2); position: relative; font-size: 32px; line-height: 1.8; font-weight: 500; }
        .char-ok { color: #fff; text-shadow: 0 0 12px var(--primary); }
        .char-no { color: var(--accent); background: rgba(255,118,117,0.2); border-radius: 4px; }
        .hard-word { border-bottom: 2px dashed var(--highlight); color: #777; }
        .hard-word.active { color: var(--highlight); text-shadow: 0 0 10px var(--highlight); }

        .cursor { 
            border-left: 3px solid var(--primary); 
            animation: blink 0.8s infinite; 
            display: inline-block; 
            height: 1.1em; 
            vertical-align: middle; 
            margin-left: 2px;
            box-shadow: 0 0 10px var(--primary);
        }
        @keyframes blink { 50% { opacity: 0; } }

        /* 特效粒子：升級後的拋物線動態 */
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

        #invisible-input { position: fixed; top: -100px; opacity: 0; }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <header>
        <div class="controls">
            <input type="color" id="theme-color" value="#74b9ff" oninput="changeTheme(this.value)" style="border:none; width:30px; height:30px; background:none; cursor:pointer; padding:0;">
            <span style="font-size: 22px; font-weight: 900; color: var(--primary); letter-spacing: 1px;">SingSangSung</span>
            <select id="song-select" onchange="loadSong()" style="background:#444; color:#fff; border:1px solid #666; border-radius:8px; padding:6px 12px; cursor:pointer;">
                <option value="">-- 選擇舞台 --</option>
                <option value="song2">This Is Me (Greatest Showman)</option>
            </select>
        </div>
        <div id="stat-display" style="font-family: monospace; font-size: 18px; color: var(--primary); font-weight: bold;">Combo: 0 | 0%</div>
    </header>

    <div class="main-game" id="game-view" style="display:none;">
        <div class="practice-window" id="window-root">
            <div id="scroll-engine">
                <div id="text-target"></div>
            </div>
            <textarea id="invisible-input" spellcheck="false" autocomplete="off"></textarea>
        </div>

        <div class="sidebar">
            <div class="card" id="card-vocab">
                <div id="vocab-hint"></div>
            </div>
            <div class="card" id="card-trans" style="border-left-color:#a29bfe;">
                <div id="trans-hint" style="line-height: 1.5; font-size: 18px; color:#eee;"></div>
            </div>
        </div>
    </div>

    <script>
    // 這裡保證數據完整不省略
    const songData = {
        "song2": {
            "lyrics": [
                "i am not a stranger to the dark", "hide away they say", "cause we dont want your broken parts",
                "ive learned to be ashamed of all my scars", "run away they say", "no onell love you as you are",
                "but i wont let them break me down to dust", "i know that theres a place for us", "for we are glorious",
                "when the sharpest words wanna cut me down", "im gonna send a flood gonna drown em out",
                "i am brave i am bruised", "i am who im meant to be this is me", "look out cause here i come",
                "and im marching on to the beat i drum", "im not scared to be seen", "i make no apologies this is me",
                "another round of bullets hits my skin", "well fire away cause today i wont let the shame sink in",
                "we are bursting through the barricades and", "reaching for the sun we are warriors",
                "yeah thats what weve become", "i wont let them break me down to dust",
                "i know that theres a place for us", "for we are glorious",
                "when the sharpest words wanna cut me down", "im gonna send a flood gonna drown em out",
                "i am brave i am bruised", "i am who im meant to be this is me",
                "look out cause here i come", "and im marching on to the beat i drum",
                "im not scared to be seen", "i make no apologies this is me",
                "this is me", "and i know that i deserve your love", "there is nothing im not worthy of",
                "when the sharpest words wanna cut me down", "im gonna send a flood gonna drown em out",
                "this is brave this is bruised", "this is who im meant to be this is me",
                "look out cause here i come", "and im marching on to the beat i drum",
                "im not scared to be seen", "i make no apologies this is me",
                "whenever the words wanna cut me down", "ill send a flood to drown em out",
                "im gonna send a flood", "gonna drown them em out", "this is me"
            ],
            "translations": [
                "我對於黑暗來說並非陌生人", "他們說：躲起來吧", "因為我們不要殘破的你",
                "我曾學會為這滿身瘡痍感到羞恥", "他們說：跑走吧", "沒有人會愛真實的你",
                "但我不會讓他們將我碾作塵埃", "我知道一定有個地方屬於我們", "因為我們是星光璀璨的",
                "當那些銳利的言語要砍向我時", "我將掀起狂潮把全部悉數吞噬",
                "我雖滿身傷痕，卻依然無畏", "我就是我注定該成為的樣貌，這就是我", "當心我要來了",
                "我隨著我的鼓點勇往直前", "我不害怕被看見", "無需任何歉意，這就是我",
                "另一輪的槍林彈雨朝我襲來", "儘管開火吧，今天我不會讓這些羞辱侵蝕我", "我們正衝破層層阻礙",
                "奔向暖陽，我們是戰士", "是的，這就是我們蛻變後的模樣",
                "但我不會讓他們將我碾作塵埃", "我知道一定有個地方屬於我們", "因為我們是星光璀璨的",
                "當那些銳利的言語要砍向我時", "我將掀起狂潮把全部悉數吞噬",
                "我雖滿身傷痕，卻依然無畏", "我就是我注定該成為的樣貌，這就是我",
                "當心我要來了", "我隨著我的鼓點勇往直前", "我不害怕被看見", "無需任何歉意，這就是我",
                "這就是我", "我深知自己值得被愛", "沒有任何東西是我不配得的",
                "當那些銳利的言語要砍向我時", "我將掀起狂潮把全部悉數吞噬",
                "這份勇敢，這份傷痕", "這就是我注定該成為的樣貌，這就是我",
                "當心我要來了", "我隨著我的鼓點勇往直前", "我不害怕被看見", "無需任何歉意，這就是我",
                "每當那些言語要砍向我時", "我將掀起狂潮將其吞噬", "我將掀起狂潮", "悉數吞噬", "這就是我"
            ],
            "vocab": [
                { "word": "stranger", "index": 0, "k": "[ˈstrendʒɚ]", "n": "陌生人" },
                { "word": "ashamed", "index": 3, "k": "[əˈʃemd]", "n": "感到羞愧的" },
                { "word": "scars", "index": 3, "k": "[skɑrz]", "n": "疤痕；創傷" },
                { "word": "glorious", "index": 8, "k": "[ˈɡloriəs]", "n": "星光璀璨的" },
                { "word": "sharpest", "index": 9, "k": "[ˈʃɑrpɪst]", "n": "最尖銳的" },
                { "word": "bruised", "index": 11, "k": "[bruzd]", "n": "受傷的；瘀青的" },
                { "word": "marching", "index": 14, "k": "[ˈmɑrtʃɪŋ]", "n": "勇往直前" },
                { "word": "apologies", "index": 16, "k": "[əˈpɑlədʒiz]", "n": "歉意" },
                { "word": "barricades", "index": 19, "k": "[ˌbærɪˈkedz]", "n": "阻礙；障礙物" },
                { "word": "warriors", "index": 20, "k": "[ˈwɔriɚz]", "n": "戰士；勇士" },
                { "word": "deserve", "index": 34, "k": "[dɪˈzɝv]", "n": "應得；值得" },
                { "word": "worthy", "index": 35, "k": "[ˈwɝði]", "n": "配得上...的" }
            ],
            "backingVocals": [
                { "trigger": "out", "index": 47, "text": "oh-oh-oh!" },
                { "trigger": "down", "index": 44, "text": "oh-oh-oh-oh" },
                { "trigger": "me", "index": 16, "text": "Oh-oh-oh-oh!" }
            ]
        }
    };

    let current = null, fullLyricsString = "", lineThresholds = [];

    const emojiMap = {
        "dark": ["🌙", "🌑", "🌃", "🕯️"], "stranger": ["👤", "🎭"], "hide": ["🙈", "📦"],
        "broken": ["💔", "🧩", "⛓️"], "scars": ["❤️‍🩹", "🩹", "🛡️"], "dust": ["💨", "✨", "🌫️"],
        "glorious": ["👑", "💎", "✨", "🎆"], "sharpest": ["🔪", "⚔️", "🗡️"], "words": ["🗣️", "📜", "💬"],
        "flood": ["🌊", "⛈️", "💧"], "brave": ["🦁", "🔥", "✊"], "bruised": ["🥊", "💜", "🩹"],
        "me": ["🌈", "✨", "💃", "🕺", "🎩"], "march": ["🥁", "💂", "🚶"], "beat": ["🥁", "💓", "🎵"],
        "drum": ["🥁", "🎶", "💥"], "bullets": ["💥", "☄️", "⚡"], "fire": ["🔥", "🧨", "🏹"],
        "sun": ["☀️", "🌻", "🌅"], "warriors": ["⚔️", "🔱", "🛡️"], "look": ["👁️", "🔦", "🎪"],
        "apologies": ["🙏", "🕊️"], "barricades": ["🚧", "🧱", "🛡️"]
    };
    const genericEmojis = ["✨", "💫", "⭐", "🎪", "🎭", "🎩"];

    function loadSong() {
        const key = document.getElementById('song-select').value;
        current = songData[key];
        
        // 專屬背景切換邏輯
        if (key === 'song2') {
            document.body.classList.add('this-is-me-bg');
        } else {
            document.body.classList.remove('this-is-me-bg');
        }

        if (!current) {
            document.getElementById('game-view').style.display = 'none';
            return;
        }
        
        fullLyricsString = "";
        lineThresholds = [];
        let currentCount = 0;
        current.lyrics.forEach(line => {
            fullLyricsString += line + "\n";
            currentCount += line.length + 1;
            lineThresholds.push(currentCount);
        });

        document.getElementById('invisible-input').value = "";
        document.getElementById('game-view').style.display = 'grid';
        render();
        setTimeout(() => document.getElementById('invisible-input').focus(), 100);
    }

    function spawnEffect(text, isOh = false) {
        const cursor = document.querySelector('.cursor');
        if (!cursor) return;
        const rect = cursor.getBoundingClientRect();
        const winRoot = document.getElementById('window-root');
        const winRect = winRoot.getBoundingClientRect();
        
        const el = document.createElement('div');
        el.className = 'emoji-particle';
        el.innerText = text;
        
        const duration = isOh ? 1.8 : (0.8 + Math.random() * 0.7);
        const mx = (Math.random() - 0.5) * 80; 
        const dx = (Math.random() - 0.5) * 400; 
        const dy = -150 - (Math.random() * 250); 
        const r1 = (Math.random() * 90) + "deg"; 
        const r2 = (Math.random() * 360) + "deg";
        
        el.style.setProperty('--duration', `${duration}s`);
        el.style.setProperty('--mx', `${mx}px`);
        el.style.setProperty('--dx', `${dx}px`);
        el.style.setProperty('--dy', `${dy}px`);
        el.style.setProperty('--r1', r1);
        el.style.setProperty('--r2', r2);
        
        el.style.left = (rect.left - winRect.left) + "px";
        el.style.top = (rect.top - winRect.top) + "px";
        
        if (isOh) {
            el.style.fontSize = "48px";
            el.style.filter = "drop-shadow(0 0 20px gold)";
        }
        
        winRoot.appendChild(el);
        setTimeout(() => el.remove(), duration * 1000);
    }

    document.getElementById('invisible-input').addEventListener('input', (e) => {
        const val = e.target.value;
        render();
        updateStats();
        
        const lastWord = val.split(/[\s\n]+/).pop().toLowerCase().replace(/[.,!]/g, '');
        let lineIdx = lineThresholds.findIndex(t => val.length < t);
        if (lineIdx === -1) lineIdx = current.lyrics.length - 1;
        
        const bv = current.backingVocals.find(b => b.trigger === lastWord && b.index === lineIdx);
        
        if (bv) {
            for(let i=0; i<5; i++) setTimeout(() => spawnEffect(bv.text, true), i * 150);
        } else if (e.data) {
            const list = emojiMap[lastWord] || genericEmojis;
            spawnEffect(list[Math.floor(Math.random() * list.length)]);
        }
    });

    function render() {
        const val = document.getElementById('invisible-input').value;
        const target = fullLyricsString;
        let html = "";
        const activeVocab = current.vocab.map(v => ({...v, start: getPos(v.index, v.word)}));

        for (let i = 0; i < target.length; i++) {
            const v = activeVocab.find(av => i >= av.start && i < av.start + av.word.length);
            let cls = "";
            if (i < val.length) cls = (val[i] === target[i] ? 'char-ok' : 'char-no');
            
            if (v) {
                const isHitting = val.length >= v.start && val.length <= v.start + v.word.length;
                html += `<span class="${cls} hard-word ${isHitting ? 'active' : ''}">${target[i]}</span>`;
            } else {
                html += `<span class="${cls}">${target[i]}</span>`;
            }
            if (i === val.length - 1) html += `<span class="cursor"></span>`;
        }
        
        if (val.length === 0) html = `<span class="cursor"></span>` + target;
        document.getElementById('text-target').innerHTML = html;
        const curEl = document.querySelector('.cursor') || document.getElementById('text-target');
        const offset = val.length === 0 ? 0 : curEl.offsetTop;
        document.getElementById('scroll-engine').style.transform = `translateY(-${offset}px)`;
    }

    function getPos(lineIdx, word) {
        let pos = 0;
        for(let i=0; i<lineIdx; i++) pos += (current.lyrics[i] ? current.lyrics[i].length + 1 : 0);
        const lineText = current.lyrics[lineIdx];
        const wordIdx = lineText ? lineText.indexOf(word) : -1;
        return wordIdx !== -1 ? pos + wordIdx : pos;
    }

    function updateStats() {
        const val = document.getElementById('invisible-input').value;
        const target = fullLyricsString;
        let lineIdx = lineThresholds.findIndex(t => val.length < t);
        if (lineIdx === -1) lineIdx = current.lyrics.length - 1;
        document.getElementById('trans-hint').innerText = current.translations[lineIdx] || "";
        let correct = 0;
        for(let i=0; i<val.length; i++) if(val[i] === target[i]) correct++;
        const percent = val.length > 0 ? Math.floor((correct / val.length) * 100) : 0;
        document.getElementById('stat-display').innerText = `Combo: ${val.length} | ${percent}%`;
        const vocabCard = document.getElementById('card-vocab');
        const targetVocab = current.vocab.find(v => {
            const start = getPos(v.index, v.word);
            return val.length >= (start - 5) && val.length <= (start + v.word.length);
        });
        if (targetVocab) {
            vocabCard.classList.remove('hidden');
            document.getElementById('vocab-hint').innerHTML = `
                <div style="color:var(--highlight); font-size:12px; font-weight:bold; margin-bottom:4px;">KEY VOCAB</div>
                <b style="font-size:24px; color:var(--primary)">${targetVocab.word}</b> 
                <div style="color:#aaa; font-size:14px; margin:4px 0;">${targetVocab.k}</div>
                <div style="font-size:18px; color:#fff;">${targetVocab.n}</div>`;
        } else {
            const nextVocab = current.vocab.find(v => getPos(v.index, v.word) > val.length);
            if (nextVocab) {
                vocabCard.classList.remove('hidden');
                document.getElementById('vocab-hint').innerHTML = `<div style="color:#666; font-size:12px;">UPCOMING</div><b style="color:#888; font-size:18px;">${nextVocab.word}</b>`;
            } else {
                vocabCard.classList.add('hidden');
            }
        }
    }
    function changeTheme(color) {
        // 同步更新 CSS 變數中的 primary 顏色
        document.documentElement.style.setProperty('--primary', color);
    }
    </script>
</body>
</html>
