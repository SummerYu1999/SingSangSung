<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - Smart Focus</title>
    <style>
        :root { --primary: #74b9ff; --accent: #ff7675; --bg: #2d3436; --card: #353b48; --text: #dfe6e9; --highlight: #ffeaa7; }
        * { box-sizing: border-box; }<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - Smart Focus</title>
    <style>
        :root { --primary: #74b9ff; --accent: #ff7675; --bg: #2d3436; --card: #353b48; --text: #dfe6e9; --highlight: #ffeaa7; }
        * { box-sizing: border-box; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background-color: var(--bg); color: var(--text); margin: 0; overflow: hidden; display: flex; flex-direction: column; height: 100vh; }
        
        header { background: rgba(0,0,0,0.8); padding: 12px 25px; display: flex; justify-content: space-between; align-items: center; z-index: 100; }
        .controls { display: flex; gap: 15px; align-items: center; }
        
        .main-game { 
            display: grid; 
            grid-template-columns: 1fr 300px; 
            gap: 20px; 
            padding: 20px; 
            flex: 1; 
            height: calc(100vh - 70px); 
            max-width: 1400px; 
            margin: 0 auto; 
            width: 100%; 
        }

        .practice-window { position: relative; overflow: hidden; background: rgba(0,0,0,0.5); border-radius: 20px; padding: 60px 40px; height: 100%; border: 1px solid #444; }
        #scroll-engine { transition: transform 0.4s cubic-bezier(0.23, 1, 0.32, 1); will-change: transform; }
        #text-target { white-space: pre-wrap; color: #444; position: relative; font-size: 32px; line-height: 2; font-weight: 600; letter-spacing: 1px; }
        
        .char-ok { color: #fff; text-shadow: 0 0 15px var(--primary); }
        .char-no { color: var(--accent); background: rgba(255,118,117,0.2); border-radius: 4px; }
        .hard-word { border-bottom: 2px dashed var(--highlight); color: #666; }
        .hard-word.active { color: var(--highlight); text-shadow: 0 0 10px var(--highlight); }

        .cursor { border-left: 3px solid var(--primary); animation: blink 0.8s infinite; display: inline-block; height: 1.2em; vertical-align: middle; margin-left: 1px; box-shadow: 0 0 10px var(--primary); }
        @keyframes blink { 50% { opacity: 0; } }

        #invisible-input { position: absolute; opacity: 0; pointer-events: none; }

        .sidebar { display: flex; flex-direction: column; gap: 15px; }
        .card { background: var(--card); padding: 20px; border-radius: 14px; border-left: 5px solid var(--primary); box-shadow: 0 8px 20px rgba(0,0,0,0.4); }
        .card.hidden { opacity: 0; transform: translateX(30px); }

        /* 特效粒子 */
        .emoji-particle {
            position: absolute;
            pointer-events: none;
            z-index: 2000;
            white-space: nowrap;
            font-family: "Segoe UI Emoji", sans-serif;
            font-size: 30px;
            animation: emoji-fly 1s forwards ease-out;
        }
        @keyframes emoji-fly {
            0% { transform: translate(0, 0) scale(0.5); opacity: 0; }
            20% { opacity: 1; transform: translate(0, -20px) scale(1.2); }
            100% { transform: translate(var(--dx), var(--dy)) scale(1.5) rotate(var(--dr)); opacity: 0; }
        }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <header>
        <div class="controls">
            <span style="font-size: 22px; font-weight: 900; color: var(--primary); letter-spacing: 1px;">SingSangSung</span>
            <select id="song-select" onchange="loadSong()" style="background:#444; color:#fff; border:none; border-radius:8px; padding:8px 15px; cursor:pointer;">
                <option value="">-- 選擇曲目 --</option>
                <option value="song2">This Is Me (Greatest Showman)</option>
            </select>
        </div>
        <div id="stat-display" style="font-family: 'Courier New', monospace; font-size: 18px; color: var(--primary); font-weight: bold;">Combo: 0 | 0%</div>
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
            <div class="card" id="card-trans" style="border-left-color:#a29bfe;">
                <div id="trans-hint">請開始打字...</div>
            </div>
        </div>
    </div>

    <script>
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

    function loadSong() {
        const key = document.getElementById('song-select').value;
        current = songData[key];
        if (!current) return;
        
        fullLyricsString = "";
        lineThresholds = [];
        let currentCount = 0;
        current.lyrics.forEach(line => {
            fullLyricsString += line + "\n";
            currentCount += line.length + 1;
            lineThresholds.push(currentCount);
        });

        document.getElementById('invisible-input').value = ""; // 重置輸入
        document.getElementById('game-view').style.display = 'grid';
        render();
        document.getElementById('invisible-input').focus();
    }

    const emojiMap = {
        "dark": ["🌙", "🌑", "✨"], "fire": ["🔥", "💥"], "flood": ["🌊", "💧"],
        "sun": ["☀️", "🌻"], "warriors": ["⚔️", "🛡️"], "me": ["✨", "🌈", "🦋"],
        "brave": ["🦁", "💪"], "love": ["💖", "🥰"]
    };

    function getEmoji(word) {
        const list = emojiMap[word.toLowerCase()];
        if (list) return list[Math.floor(Math.random() * list.length)];
        const general = ["✨", "🔥", "💫", "⭐", "💎", "🌟", "⚡"];
        return general[Math.floor(Math.random() * general.length)];
    }

    document.getElementById('invisible-input').addEventListener('input', (e) => {
        const val = e.target.value;
        render();
        updateStats();
        
        const words = val.trim().split(/\s+/);
        const lastWord = words[words.length - 1] || "";
        let lineIdx = lineThresholds.findIndex(t => val.length < t);
        if (lineIdx === -1) lineIdx = current.lyrics.length - 1;

        // 特效邏輯
        const bv = current.backingVocals.find(b => b.trigger === lastWord.toLowerCase() && b.index === lineIdx);
        if (bv) {
            spawnEffect(bv.text, "var(--primary)");
        } else if (e.data) { // 只要有輸入任何字元就噴
            spawnEffect(getEmoji(lastWord));
        }
    });

    function spawnEffect(text, color = null) {
        const cursor = document.querySelector('.cursor');
        if (!cursor) return;
        const rect = cursor.getBoundingClientRect();
        const winRoot = document.getElementById('window-root');
        const winRect = winRoot.getBoundingClientRect();
        
        const el = document.createElement('div');
        el.className = 'emoji-particle';
        el.innerText = text;
        if (color) {
            el.style.color = color;
            el.style.fontWeight = "bold";
            el.style.textShadow = "0 0 10px white";
        }
        
        // 相對於 window-root 的絕對位置
        el.style.left = (rect.left - winRect.left) + "px";
        el.style.top = (rect.top - winRect.top) + "px";
        el.style.setProperty('--dx', (Math.random() - 0.5) * 250 + "px");
        el.style.setProperty('--dy', (Math.random() * -200 - 50) + "px");
        el.style.setProperty('--dr', (Math.random() * 90 - 45) + "deg");
        
        winRoot.appendChild(el);
        setTimeout(() => el.remove(), 1000);
    }

    function render() {
        const val = document.getElementById('invisible-input').value;
        const target = fullLyricsString;
        let html = "";
        
        const activeVocab = current.vocab.map(v => ({...v, start: getPos(v.index, v.word)}));

        for (let i = 0; i < target.length; i++) {
            const v = activeVocab.find(av => i >= av.start && i < av.start + av.word.length);
            let cls = "";
            if (i < val.length) cls = (val[i] === target[i] ? 'char-ok' : 'char-no');
            
            const isCursor = (i === val.length);
            if (isCursor) html += `<span class="cursor"></span>`;

            if (v) {
                const isHitting = val.length >= v.start && val.length <= v.start + v.word.length;
                html += `<span class="${cls} hard-word ${isHitting ? 'active' : ''}">${target[i]}</span>`;
            } else {
                html += `<span class="${cls}">${target[i]}</span>`;
            }
        }
        
        document.getElementById('text-target').innerHTML = html;
        
        // 捲動邏輯核心修正
        const scrollEngine = document.getElementById('scroll-engine');
        const cursor = document.querySelector('.cursor');
        
        if (val.length === 0) {
            scrollEngine.style.transform = `translateY(0px)`;
        } else if (cursor) {
            // 讓光標始終保持在視窗上方 1/3 的位置，避免滾到底部
            const scrollOffset = cursor.offsetTop - 80;
            scrollEngine.style.transform = `translateY(-${scrollOffset}px)`;
        }
    }

    function getPos(lineIdx, word) {
        let pos = 0;
        for(let i=0; i<lineIdx; i++) pos += (current.lyrics[i] ? current.lyrics[i].length + 1 : 0);
        const lineText = current.lyrics[lineIdx];
        const wordIdx = lineText ? lineText.indexOf(word) : -1;
        return wordIdx !== -1 ? pos + wordIdx : -1;
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
            return start !== -1 && val.length >= (start - 3) && val.length <= (start + v.word.length + 3);
        });

        if (targetVocab) {
            vocabCard.classList.remove('hidden');
            document.getElementById('vocab-hint').innerHTML = `
                <div style="color:var(--highlight); font-size:12px; font-weight:bold;">KEY WORD</div>
                <b style="font-size:24px; color:var(--primary)">${targetVocab.word}</b> 
                <div style="color:#aaa; font-size:14px; margin: 4px 0;">${targetVocab.k}</div>
                <div style="font-size:18px;">${targetVocab.n}</div>`;
        } else {
            vocabCard.classList.add('hidden');
        }
    }
    </script>
</body>
</html>
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background-color: var(--bg); color: var(--text); margin: 0; overflow: hidden; display: flex; flex-direction: column; height: 100vh; }
        
        header { background: rgba(0,0,0,0.6); padding: 12px 25px; display: flex; justify-content: space-between; align-items: center; z-index: 100; }
        .controls { display: flex; gap: 15px; align-items: center; }
        
        .main-game { 
            display: grid; 
            grid-template-columns: 1fr 280px; 
            gap: 20px; 
            padding: 20px; 
            flex: 1; 
            height: calc(100vh - 70px); 
            max-width: 1400px; 
            margin: 0 auto; 
            width: 100%; 
        }

        .sidebar { display: flex; flex-direction: column; gap: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 14px; border-left: 5px solid var(--primary); box-shadow: 0 4px 15px rgba(0,0,0,0.3); transition: all 0.4s ease; }
        .card.hidden { opacity: 0; transform: translateX(20px); }

        .practice-window { position: relative; overflow: hidden; background: rgba(0,0,0,0.4); border-radius: 20px; padding: 40px; height: 100%; }
        #scroll-engine { transition: transform 0.3s ease; }
        #text-target { white-space: pre-wrap; color: #555; position: relative; font-size: 32px; line-height: 1.8; font-weight: 600; }
        
        .char-ok { color: #fff; text-shadow: 0 0 12px var(--primary); }
        .char-no { color: var(--accent); background: rgba(255,118,117,0.2); }
        .hard-word { border-bottom: 2px dashed var(--highlight); color: #888; }
        .hard-word.active { color: var(--highlight); text-shadow: 0 0 8px var(--highlight); }

        .cursor { border-left: 3px solid var(--primary); animation: blink 0.8s infinite; display: inline-block; height: 1.1em; vertical-align: middle; margin-left: 2px; }
        @keyframes blink { 50% { opacity: 0; } }

        /* 隱藏輸入框 */
        #invisible-input { position: absolute; opacity: 0; width: 0; height: 0; pointer-events: none; }

        /* 特效樣式 */
        .emoji-particle {
            position: absolute;
            pointer-events: none;
            z-index: 1000;
            white-space: nowrap;
            font-family: "Segoe UI Emoji", sans-serif;
            font-size: 28px;
            font-weight: bold;
            animation: emoji-fly 1s forwards cubic-bezier(0.12, 0, 0.39, 0);
        }

        @keyframes emoji-fly {
            0% { transform: translate(0, 0) scale(0.5) rotate(0deg); opacity: 0; }
            20% { opacity: 1; transform: translate(0, -20px) scale(1.2); }
            100% { transform: translate(var(--dx), var(--dy)) scale(1.5) rotate(var(--dr)); opacity: 0; }
        }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <header>
        <div class="controls">
            <span style="font-size: 20px; font-weight: 900; color: var(--primary);">SingSangSung</span>
            <select id="song-select" onchange="loadSong()" style="background:#333; color:#fff; border-radius:8px; padding:5px;">
                <option value="">-- 選擇曲目 --</option>
                <option value="song2">This Is Me (Greatest Showman)</option>
            </select>
        </div>
        <div id="stat-display" style="font-family: monospace; color: var(--primary);">Combo: 0 | 0%</div>
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
            <div class="card" id="card-trans" style="border-left-color:#a29bfe;">
                <div id="trans-hint"></div>
            </div>
        </div>
    </div>

    <script>
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

    function loadSong() {
        const key = document.getElementById('song-select').value;
        current = songData[key];
        if (!current) return;
        
        fullLyricsString = "";
        lineThresholds = [];
        let currentCount = 0;
        current.lyrics.forEach(line => {
            fullLyricsString += line + "\n";
            currentCount += line.length + 1;
            lineThresholds.push(currentCount);
        });

        document.getElementById('game-view').style.display = 'grid';
        render();
        document.getElementById('invisible-input').focus();
    }

    // 關鍵字 Emoji 對照表
    const emojiMap = {
        "dark": ["🌙", "🌑", "✨"],
        "scars": ["🩹", "❤️‍🩹"],
        "fire": ["🔥", "💥"],
        "flood": ["🌊", "🌊","🌊"],
        "sun": ["☀️", "🌻"],
        "warriors": ["⚔️", "🛡️"],
        "bullets": ["🔫", "🧨"],
        "drum": ["🥁", "🎵"],
        "me": ["✨", "🌈", "🦋"],
        "brave": ["🦁", "💪"]
    };

    function getEmojiForWord(word) {
        const list = emojiMap[word.toLowerCase()];
        if (list) return list[Math.floor(Math.random() * list.length)];
        const general = ["✨", "🔥", "💫", "⭐", "🚀", "💎"];
        return general[Math.floor(Math.random() * general.length)];
    }

    document.getElementById('invisible-input').addEventListener('input', (e) => {
        const val = e.target.value;
        render();
        updateStats();
        
        const words = val.trim().split(/\s+/);
        const lastWord = words[words.length - 1].toLowerCase();
        let lineIdx = lineThresholds.findIndex(t => val.length < t);
        
        // 1. 檢查是否有 backingVocals (Oh-oh)
        const bv = current.backingVocals.find(b => b.trigger === lastWord && b.index === lineIdx);
        if (bv) {
            spawnEffect(bv.text, "var(--primary)");
        } 
        // 2. 隨機噴發符合歌詞的 Emoji
        else if (e.data === " " || e.inputType === "insertLineBreak") {
            spawnEffect(getEmojiForWord(lastWord));
        }
    });

    function spawnEffect(text, color = null) {
        const cursor = document.querySelector('.cursor');
        if (!cursor) return;
        const rect = cursor.getBoundingClientRect();
        const winRect = document.getElementById('window-root').getBoundingClientRect();
        const scrollY = document.getElementById('scroll-engine').style.transform.replace(/[^0-9-.]/g, '') || 0;
        
        const el = document.createElement('div');
        el.className = 'emoji-particle';
        el.innerText = text;
        if (color) el.style.color = color;
        
        el.style.left = (rect.left - winRect.left) + "px";
        el.style.top = (rect.top - winRect.top - parseFloat(scrollY)) + "px";
        el.style.setProperty('--dx', (Math.random() - 0.5) * 200 + "px");
        el.style.setProperty('--dy', (Math.random() * -150 - 50) + "px");
        el.style.setProperty('--dr', (Math.random() * 60 - 30) + "deg");
        
        document.getElementById('window-root').appendChild(el);
        setTimeout(() => el.remove(), 1000);
    }

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
        document.getElementById('text-target').innerHTML = html || (target[0] ? '<span class="cursor"></span>' + target : '');
        
        // 修正捲動邏輯：確保只有在有輸入內容時才開始捲動
        const cursor = document.querySelector('.cursor');
        const scrollEngine = document.getElementById('scroll-engine');
        
        if (val.length === 0) {
            // 如果還沒開始打字，強制回到最上方
            scrollEngine.style.transform = `translateY(0px)`;
        } else if (cursor) {
            // 計算光標相對於練習視窗頂部的距離，並保持在視窗上方約 40px 的位置
            const offset = cursor.offsetTop;
            scrollEngine.style.transform = `translateY(-${offset - 40}px)`;
        }
    }

    function getPos(lineIdx, word) {
        let pos = 0;
        for(let i=0; i<lineIdx; i++) pos += (current.lyrics[i] ? current.lyrics[i].length + 1 : 0);
        const lineText = current.lyrics[lineIdx];
        const wordIdx = lineText ? lineText.indexOf(word) : -1;
        return wordIdx !== -1 ? pos + wordIdx : -1;
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
            return start !== -1 && val.length >= (start - 5) && val.length <= (start + v.word.length);
        });

        if (targetVocab) {
            vocabCard.classList.remove('hidden');
            document.getElementById('vocab-hint').innerHTML = `
                <div style="color:var(--highlight); font-size:12px;">KEY WORD</div>
                <b style="font-size:22px; color:var(--primary)">${targetVocab.word}</b> 
                <span style="color:#aaa;">${targetVocab.k}</span>
                <div style="margin-top:5px; font-size:18px;">${targetVocab.n}</div>`;
        } else {
            vocabCard.classList.add('hidden');
        }
    }
    </script>
</body>
</html>
