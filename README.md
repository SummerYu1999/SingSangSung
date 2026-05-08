<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - 沉浸式練習</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.1/dist/confetti.browser.min.js"></script>
    <style>
        :root { --primary: #74b9ff; --accent: #ff7675; --bg: #2d3436; --card: #353b48; --text: #dfe6e9; --gray: #636e72; }
        * { box-sizing: border-box; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background: var(--bg); color: var(--text); margin: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh; overflow-x: hidden; }
        
        /* 頂部進度條 */
        .progress-container { width: 100%; height: 6px; background: #1e272e; position: fixed; top: 0; z-index: 100; }
        #progress-bar { width: 0%; height: 100%; background: var(--primary); box-shadow: 0 0 10px var(--primary); transition: width 0.2s; }

        header { margin-top: 40px; text-align: center; width: 90%; max-width: 1000px; }
        .stats-bar { display: flex; justify-content: space-between; background: var(--card); padding: 15px 25px; border-radius: 12px; margin: 20px 0; border: 1px solid #4b5563; }

        /* 響應式佈局：電腦雙欄，手機單欄 */
        .main-layout { display: flex; flex-wrap: wrap; width: 95%; max-width: 1100px; gap: 20px; padding-bottom: 50px; }
        .practice-section { flex: 2; min-width: 350px; }
        .info-section { flex: 1; min-width: 300px; display: flex; flex-direction: column; gap: 15px; }

        /* 練習區 */
        .practice-window { 
            position: relative; background: #1e2124; border-radius: 20px; height: 500px; overflow: hidden; 
            border: 2px solid #4b5563; box-shadow: 0 20px 50px rgba(0,0,0,0.3);
        }
        
        #scrolling-content {
            position: absolute; width: 100%; padding: 200px 40px;
            transition: transform 0.4s cubic-bezier(0.18, 0.89, 0.32, 1.28);
            font-size: 28px; line-height: 2.2; letter-spacing: 2px;
        }

        #text-display { white-space: pre-wrap; word-wrap: break-word; color: var(--gray); }
        .cursor { border-left: 3px solid var(--primary); margin-left: 2px; animation: blink 0.8s infinite; }
        @keyframes blink { 50% { opacity: 0; } }

        .char-correct { color: var(--text); text-shadow: 0 0 5px var(--primary); }
        .char-wrong { color: var(--accent); background: rgba(255,118,117,0.2); border-radius: 4px; }

        /* 資訊卡 */
        .info-card { background: var(--card); padding: 20px; border-radius: 15px; border-left: 5px solid var(--primary); box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .card-label { font-size: 12px; text-transform: uppercase; color: var(--primary); font-weight: bold; margin-bottom: 8px; }

        #invisible-input { position: fixed; top: -100px; opacity: 0; }

        /* 飄浮星星特效 */
        .star { position: absolute; pointer-events: none; animation: floatUp 1.5s ease-out forwards; font-size: 20px; }
        @keyframes floatUp { 0% { transform: translateY(0) scale(1); opacity: 1; } 100% { transform: translateY(-100px) scale(1.5); opacity: 0; } }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <div class="progress-container"><div id="progress-bar"></div></div>

    <header>
        <h1 style="color: var(--primary); margin-bottom: 10px;">SingSangSung</h1>
        <div class="stats-bar">
            <select id="song-selector" onchange="initSong()" style="background:#2d3436; color:white; border:1px solid #636e72; padding:5px 10px; border-radius:6px;">
                <option value="">-- 選取練習曲目 --</option>
                <option value="song1">中文：小幸運</option>
                <option value="song2">英文：This Is Me</option>
            </select>
            <div style="color:var(--accent); font-weight:bold;">Combo: <span id="combo-count">0</span></div>
            <div id="accuracy-display">進度: 0%</div>
        </div>
    </header>

    <div class="main-layout" id="layout" style="display:none;">
        <div class="practice-section">
            <div class="practice-window" id="window">
                <div id="scrolling-content">
                    <div id="text-display"></div>
                </div>
                <textarea id="invisible-input" spellcheck="false" autocomplete="off" autofocus></textarea>
            </div>
        </div>

        <div class="info-section">
            <div id="vocab-hint" class="info-card">
                <div class="card-label">Vocabulary Hint</div>
                <div id="hint-content" style="font-size:18px;">選擇歌曲開始練習</div>
            </div>
            <div class="info-card" style="border-left-color: #a29bfe;">
                <div class="card-label">Translation</div>
                <div id="trans-content" style="font-size:15px; line-height:1.6; color: #b2bec3;"></div>
            </div>
        </div>
    </div>

    <script>
        const songLibrary = {
            "song1": {
                "lyrics": "我聽見雨落在青草地\n我聽見遠方下課鐘聲響起\n原來你是我最想留住的幸運",
                "translation": "I hear the rain falling on the grass, I hear the school bell ringing in the distance...",
                "vocab": [{ "trigger": "幸運", "kk": "ㄒㄧㄥˋ ㄩㄣˋ", "note": "Lucky" }]
            },
            "song2": {
                "lyrics": "I am not a stranger to the dark\nHide away they say\nCause we dont want your broken parts\nIve learned to be ashamed of all my scars\nRun away they say\nNo onell love you as you are\nBut I wont let them break me down to dust\nI know that theres a place for us\nFor we are glorious",
                "translation": "我不畏懼黑暗。他們說：躲起來吧，我們不需要你那破碎的部分。我曾學會為我身上的疤痕感到羞恥...",
                "vocab": [
                    { "trigger": "stranger", "kk": "/ˈstreɪndʒər/", "note": "陌生人" },
                    { "trigger": "ashamed", "kk": "/əˈʃeɪmd/", "note": "感到羞愧的" },
                    { "trigger": "glorious", "kk": "/ˈɡlɔːriəs/", "note": "輝煌燦爛的" }
                ]
            }
        };

        let currentData = null;
        let combo = 0;
        const display = document.getElementById('text-display');
        const input = document.getElementById('invisible-input');
        const scrollContent = document.getElementById('scrolling-content');

        function initSong() {
            const val = document.getElementById('song-selector').value;
            currentData = songLibrary[val];
            if (!currentData) return;

            document.getElementById('layout').style.display = 'flex';
            document.getElementById('trans-content').innerText = currentData.translation;
            input.value = "";
            render(0);
            input.focus();
        }

        input.addEventListener('input', () => {
            const val = input.value;
            const target = currentData.lyrics;
            const lastIdx = val.length - 1;

            if (val[lastIdx] === target[lastIdx]) {
                combo++;
                spawnVisualEffect(); // 觸發可愛效果
                if (combo % 10 === 0) triggerConfetti(); // 每10次噴紙屑
            } else {
                combo = 0;
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
                    html += `<span id="char-${i}">${target[i]}</span><span class="cursor"></span>`;
                } else {
                    html += `<span id="char-${i}">${target[i]}</span>`;
                }
            }
            display.innerHTML = html;

            const currentChar = document.getElementById(`char-${inputLen}`);
            if (currentChar) {
                const offset = currentChar.offsetTop;
                scrollContent.style.transform = `translateY(-${offset - 50}px)`;
            }
        }

        function updateUI(userInput) {
            const progress = (userInput.length / currentData.lyrics.length) * 100;
            document.getElementById('progress-bar').style.width = progress + "%";
            document.getElementById('accuracy-display').innerText = `進度: ${Math.floor(progress)}%`;
            document.getElementById('combo-count').innerText = combo;
        }

        function spawnVisualEffect() {
            const star = document.createElement('div');
            star.className = 'star';
            star.innerText = ['⭐', '✨', '💖', '🎈'][Math.floor(Math.random()*4)];
            star.style.left = (Math.random() * 80 + 10) + "%";
            star.style.bottom = "20%";
            document.getElementById('window').appendChild(star);
            setTimeout(() => star.remove(), 1500);
        }

        function triggerConfetti() {
            confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
        }

        function checkVocab(userInput) {
            const lastWord = userInput.split(/[\s\n]+/).pop().toLowerCase();
            const v = currentData.vocab.find(x => x.trigger === lastWord);
            if (v) {
                document.getElementById('hint-content').innerHTML = `<b style="color:var(--primary)">${v.trigger}</b> <small style="color:var(--accent)">${v.kk}</small><br><span style="font-size:14px; color:#b2bec3">${v.note}</span>`;
            }
        }

        function focusInput() {
            if(document.getElementById('layout').style.display === 'flex') input.focus();
        }
    </script>
</body>
</html>
