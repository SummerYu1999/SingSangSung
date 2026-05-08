<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung Ultra - 極致沉浸版</title>
    <style>
        :root { --primary: #4a90e2; --accent: #f39c12; --bg: #1e1e1e; --card: #2d2d2d; --text: #e0e0e0; --gray: #555; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background: var(--bg); color: var(--text); display: flex; flex-direction: column; align-items: center; padding: 20px; overflow-x: hidden; }
        
        /* 進度條 */
        .progress-container { width: 100%; max-width: 850px; height: 6px; background: #333; border-radius: 3px; margin-bottom: 20px; }
        #progress-bar { width: 0%; height: 100%; background: var(--primary); box-shadow: 0 0 10px var(--primary); transition: width 0.2s; }

        .control-panel { background: var(--card); padding: 15px 25px; border-radius: 12px; margin-bottom: 20px; width: 95%; max-width: 850px; display: flex; justify-content: space-between; align-items: center; border: 1px solid #444; }
        
        .main-layout { display: grid; grid-template-columns: 1fr 280px; gap: 20px; width: 95%; max-width: 850px; }

        /* 核心練習區 - 增加高度限制並支援自動捲動 */
        .practice-window { 
            position: relative; background: var(--card); border-radius: 15px; height: 450px; overflow: hidden; /* 關鍵：固定高度 */
            box-shadow: inset 0 0 20px rgba(0,0,0,0.5); border: 1px solid #444; 
        }
        
        #scrolling-content {
            position: absolute; width: 100%; padding: 150px 40px; /* 上下留白讓文字能置中 */
            transition: transform 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
            font-size: 28px; line-height: 2.2; letter-spacing: 2px;
        }

        #text-display { white-space: pre-wrap; word-wrap: break-word; color: var(--gray); position: relative; }
        
        /* 游標樣式 */
        .cursor { border-left: 3px solid var(--primary); margin-left: 2px; animation: blink 0.8s infinite; }
        @keyframes blink { 50% { opacity: 0; } }

        .char-correct { color: var(--primary); text-shadow: 0 0 8px rgba(74,144,226,0.5); }
        .char-wrong { color: #ff4d4d; background: rgba(255,0,0,0.2); border-radius: 4px; }

        /* 震動動畫 */
        .shake { animation: shake 0.2s; }
        @keyframes shake { 0%, 100% { transform: translateX(0); } 25% { transform: translateX(-5px); } 75% { transform: translateX(5px); } }

        /* 側邊資訊 */
        .side-panel { display: flex; flex-direction: column; gap: 15px; }
        .info-card { background: var(--card); padding: 15px; border-radius: 12px; border-left: 4px solid var(--primary); opacity: 0.6; transition: 0.3s; }
        .info-card.active { opacity: 1; transform: scale(1.02); background: #383838; }

        #invisible-input { position: fixed; opacity: 0; }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <div class="progress-container"><div id="progress-bar"></div></div>

    <div class="control-panel">
        <select id="song-selector" onchange="initSong()" style="background:#444; color:white; border:none; padding:5px 10px; border-radius:4px;">
            <option value="">-- 選取曲目 --</option>
            <option value="song2">This Is Me</option>
        </select>
        <div style="color:var(--accent)">Combo: <span id="combo-count">0</span></div>
        <div id="accuracy-display">0%</div>
    </div>

    <div class="main-layout" id="layout" style="display:none;">
        <div class="practice-window" id="window">
            <div id="scrolling-content">
                <div id="text-display"></div>
            </div>
            <textarea id="invisible-input" spellcheck="false" autocomplete="off"></textarea>
        </div>

        <div class="side-panel">
            <div id="vocab-hint" class="info-card">
                <div style="color:var(--primary); font-size:14px;">Vocabulary Hint</div>
                <div id="hint-content" style="margin-top:5px; font-size:18px;">---</div>
            </div>
            <div class="info-card" style="border-left-color: #9b59b6; opacity:1">
                <div style="color:#9b59b6; font-size:14px;">Translation</div>
                <div id="trans-content" style="font-size:14px; margin-top:5px; line-height:1.5;"></div>
            </div>
        </div>
    </div>

    <audio id="sound-hit" src="https://assets.mixkit.co/active_storage/sfx/2571/2571-preview.mp3"></audio>
    <audio id="sound-error" src="https://assets.mixkit.co/active_storage/sfx/2573/2573-preview.mp3"></audio>

    <script>
        const songLibrary = {
            "song2": {
                "lyrics": "I am not a stranger to the dark\nHide away they say\nCause we dont want your broken parts\nIve learned to be ashamed of all my scars\nRun away they say\nNo onell love you as you are\nBut I wont let them break me down to dust\nI know that theres a place for us\nFor we are glorious",
                "translation": "我不畏懼黑暗。他們說：躲起來吧，我們不需要你那殘缺的部分。我曾學會為我身上的疤痕感到羞恥...",
                "vocab": [
                    { "trigger": "stranger", "kk": "/ˈstreɪndʒər/", "note": "陌生人" },
                    { "trigger": "ashamed", "kk": "/əˈʃeɪmd/", "note": "感到羞愧的" },
                    { "trigger": "glorious", "kk": "/ˈɡlɔːriəs/", "note": "輝煌的" }
                ]
            }
        };

        let currentData = null;
        let combo = 0;
        const display = document.getElementById('text-display');
        const input = document.getElementById('invisible-input');
        const scrollContent = document.getElementById('scrolling-content');
        const hitSound = document.getElementById('sound-hit');
        const errorSound = document.getElementById('sound-error');

        function initSong() {
            const val = document.getElementById('song-selector').value;
            currentData = songLibrary[val];
            if (!currentData) return;

            document.getElementById('layout').style.display = 'grid';
            document.getElementById('trans-content').innerText = currentData.translation;
            input.value = "";
            render(0);
            input.focus();
        }

        input.addEventListener('input', () => {
            const val = input.value;
            const target = currentData.lyrics;
            const lastCharIndex = val.length - 1;

            if (val[lastCharIndex] === target[lastCharIndex]) {
                combo++;
                hitSound.currentTime = 0;
                hitSound.play();
            } else if (val.length > 0) {
                combo = 0;
                errorSound.currentTime = 0;
                errorSound.play();
                document.getElementById('window').classList.add('shake');
                setTimeout(() => document.getElementById('window').classList.remove('shake'), 200);
            }

            render(val.length);
            updateUI(val);
            checkVocab(val);
        });

        function render(inputLen) {
            const target = currentData.lyrics;
            const userIn = input.value;
            let html = "";

            for (let i = 0; i < target.length; i++) {
                if (i < inputLen) {
                    const isCorrect = userIn[i] === target[i];
                    html += `<span class="${isCorrect ? 'char-correct' : 'char-wrong'}" id="char-${i}">${target[i]}</span>`;
                } else if (i === inputLen) {
                    // 當前輸入位置：顯示文字 + 游標
                    html += `<span id="char-${i}">${target[i]}</span><span class="cursor"></span>`;
                } else {
                    html += `<span id="char-${i}">${target[i]}</span>`;
                }
            }
            display.innerHTML = html;

            // 自動捲動邏輯：讓當前字元保持在容器中央
            const currentChar = document.getElementById(`char-${inputLen}`);
            if (currentChar) {
                const offset = currentChar.offsetTop;
                scrollContent.style.transform = `translateY(-${offset - 50}px)`;
            }
        }

        function updateUI(userInput) {
            const progress = (userInput.length / currentData.lyrics.length) * 100;
            document.getElementById('progress-bar').style.width = progress + "%";
            document.getElementById('accuracy-display').innerText = `${Math.floor(progress)}%`;
            document.getElementById('combo-count').innerText = combo;
        }

        function checkVocab(userInput) {
            const lastWord = userInput.split(/[\s\n]+/).pop().toLowerCase();
            const v = currentData.vocab.find(x => x.trigger === lastWord);
            if (v) {
                document.getElementById('hint-content').innerHTML = `<b>${v.trigger}</b> <small>${v.kk}</small><br><span style="font-size:14px">${v.note}</span>`;
                document.getElementById('vocab-hint').classList.add('active');
            }
        }
    </script>
</body>
</html>
