<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - Smart Focus</title>
    <style>
        :root { --primary: #74b9ff; --accent: #ff7675; --bg: #2d3436; --card: #353b48; --text: #dfe6e9; --highlight: #ffeaa7; }
        * { box-sizing: border-box; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background-color: var(--bg); color: var(--text); margin: 0; overflow: hidden; display: flex; flex-direction: column; height: 100vh; }
        
        header { background: rgba(0,0,0,0.6); padding: 12px 25px; display: flex; justify-content: space-between; align-items: center; z-index: 100; }
        .controls { display: flex; gap: 15px; align-items: center; }
        
        .main-game { display: grid; grid-template-columns: 1fr 320px; gap: 25px; padding: 25px; flex: 1; height: calc(100vh - 70px); max-width: 1200px; margin: 0 auto; width: 100%; }
        
        .practice-window { position: relative; background: #1a1c1e; border-radius: 20px; border: 2px solid #444; overflow: hidden; box-shadow: inset 0 0 30px rgba(0,0,0,0.7); }
        
        #scroll-engine { 
            position: absolute; width: 100%; padding: 0 50px; 
            top: 50%; 
            transition: transform 0.3s cubic-bezier(0.23, 1, 0.32, 1); 
            font-size: 34px; line-height: 2.2; letter-spacing: 2px;
        }
        
        #text-target { white-space: pre-wrap; color: #555; position: relative; }
        .char-ok { color: #fff; text-shadow: 0 0 12px var(--primary); }
        .char-no { color: var(--accent); background: rgba(255,118,117,0.2); }
        
        /* 難字高亮樣式 */
        .hard-word { border-bottom: 2px dashed var(--highlight); color: #888; }
        .hard-word.active { color: var(--highlight); text-shadow: 0 0 8px var(--highlight); }

        .cursor { border-left: 3px solid var(--primary); animation: blink 0.8s infinite; display: inline-block; height: 1.1em; vertical-align: middle; }
        @keyframes blink { 50% { opacity: 0; } }

        .sidebar { display: flex; flex-direction: column; gap: 20px; }
        .card { background: var(--card); padding: 25px; border-radius: 18px; border-left: 5px solid var(--primary); box-shadow: 0 8px 20px rgba(0,0,0,0.3); transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); min-height: 120px; }
        .card.hidden { opacity: 0; transform: translateX(20px); }

        #invisible-input { position: fixed; top: -100px; opacity: 0; }
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
                    "I am not a stranger to the dark",
                    "Hide away they say",
                    "Cause we dont want your broken parts",
                    "Ive learned to be ashamed of all my scars",
                    "Run away they say",
                    "No onell love you as you are",
                    "But I wont let them break me down to dust",
                    "I know that theres a place for us",
                    "For we are glorious"
                ],
                "translations": [
                    "我不畏懼黑暗。", "他們說：躲起來吧。", "因為我們不需要你那破碎的部分。",
                    "我曾學會為我身上的疤痕感到羞恥。", "他們說：逃跑吧。", "沒人會愛你原本的樣子。",
                    "但我不會讓他們把我擊潰成灰燼。", "我知道有個地方是屬於我們的。", "因為我們是輝煌燦爛的。"
                ],
                "vocab": [
                    { "word": "stranger", "index": 0, "k": "/ˈstreɪndʒər/", "n": "陌生人" },
                    { "word": "ashamed", "index": 3, "k": "/əˈʃeɪmd/", "n": "感到羞愧的" },
                    { "word": "glorious", "index": 8, "k": "/ˈɡlɔːriəs/", "n": "輝煌燦爛的" }
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

        document.getElementById('invisible-input').addEventListener('input', () => {
            render();
            updateStats();
        });

        function render() {
            const val = document.getElementById('invisible-input').value;
            const target = fullLyricsString;
            let html = "";
            
            // 找出目前哪些難字該標記
            const activeVocab = current.vocab.map(v => ({...v, start: getPos(v.index, v.word)}));

            for (let i = 0; i < target.length; i++) {
                // 判斷是否屬於難字範圍
                const v = activeVocab.find(av => i >= av.start && i < av.start + av.word.length);
                let cls = "";
                if (i < val.length) cls = (val[i] === target[i] ? 'char-ok' : 'char-no');
                
                if (v) {
                    const isPassed = val.length > (v.start + v.word.length);
                    const isHitting = val.length >= v.start && val.length <= v.start + v.word.length;
                    html += `<span class="${cls} hard-word ${isHitting ? 'active' : ''}">${target[i]}</span>`;
                } else {
                    html += `<span class="${cls}">${target[i]}</span>`;
                }

                if (i === val.length - 1) html += `<span class="cursor"></span>`;
            }
            document.getElementById('text-target').innerHTML = html;
            
            const curEl = document.querySelector('.cursor') || document.getElementById('text-target');
            document.getElementById('scroll-engine').style.transform = `translateY(-${curEl.offsetTop}px)`;
        }

        function getPos(lineIdx, word) {
            let pos = 0;
            for(let i=0; i<lineIdx; i++) pos += current.lyrics[i].length + 1;
            return pos + current.lyrics[lineIdx].indexOf(word);
        }

        function updateStats() {
            const val = document.getElementById('invisible-input').value;
            
            // 1. 更新翻譯
            let lineIdx = lineThresholds.findIndex(t => val.length < t);
            if (lineIdx === -1) lineIdx = current.lyrics.length - 1;
            document.getElementById('trans-hint').innerText = current.translations[lineIdx];

            // 2. 更新單字 (精確邏輯)
            const vocabCard = document.getElementById('card-vocab');
            const targetVocab = current.vocab.find(v => {
                const start = getPos(v.index, v.word);
                // 顯示條件：正在打這行，且還沒打完這個字
                return val.length >= (start - 10) && val.length <= (start + v.word.length);
            });

            if (targetVocab) {
                vocabCard.classList.remove('hidden');
                document.getElementById('vocab-hint').innerHTML = `
                    <div style="color:var(--highlight); font-size:12px; margin-bottom:5px;">KEY WORD</div>
                    <b style="font-size:22px; color:var(--primary)">${targetVocab.word}</b> 
                    <span style="color:#888; font-size:14px;">${targetVocab.k}</span>
                    <div style="margin-top:8px; font-size:18px;">${targetVocab.n}</div>
                `;
            } else {
                // 如果沒在打難字，找找下一個是誰
                const nextVocab = current.vocab.find(v => getPos(v.index, v.word) > val.length);
                if (nextVocab) {
                    vocabCard.classList.remove('hidden');
                    document.getElementById('vocab-hint').innerHTML = `
                        <div style="color:#666; font-size:12px; margin-bottom:5px;">NEXT KEY WORD</div>
                        <b style="color:#666; font-size:18px;">${nextVocab.word}</b>
                    `;
                } else {
                    vocabCard.classList.add('hidden');
                }
            }
        }
    </script>
</body>
</html>
