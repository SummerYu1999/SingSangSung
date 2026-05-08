<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung Pro - 動態背誦</title>
    <style>
        :root { --primary: #4a90e2; --accent: #f39c12; --bg: #f0f2f5; --success: #2ecc71; }
        body { font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; background: var(--bg); display: flex; flex-direction: column; align-items: center; padding: 20px; transition: all 0.3s; }
        
        /* 頂部進度條 */
        .progress-container { width: 100%; max-width: 850px; height: 10px; background: #ddd; border-radius: 5px; margin-bottom: 20px; overflow: hidden; }
        #progress-bar { width: 0%; height: 100%; background: var(--primary); transition: width 0.3s ease; }

        .control-panel { background: white; padding: 20px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); margin-bottom: 20px; width: 95%; max-width: 850px; display: flex; justify-content: space-between; align-items: center; }
        
        .main-content { display: grid; grid-template-columns: 1fr 280px; gap: 20px; width: 95%; max-width: 850px; }

        /* 練習區動畫 */
        .practice-area { position: relative; background: white; padding: 30px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.05); min-height: 400px; font-size: 24px; line-height: 2; letter-spacing: 2px; }
        #text-display { white-space: pre-wrap; word-wrap: break-word; color: #d3d3d3; }
        .char-correct { color: #333; animation: pop 0.2s ease-out; }
        .char-wrong { color: #ff4d4d; background: rgba(255,77,77,0.1); }
        @keyframes pop { 0% { transform: scale(1.2); } 100% { transform: scale(1); } }

        /* 右側即時資訊卡 */
        .side-panel { display: flex; flex-direction: column; gap: 15px; }
        .info-card { background: white; padding: 15px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.05); border-left: 5px solid var(--primary); transform: translateX(20px); opacity: 0; transition: all 0.4s; }
        .info-card.active { transform: translateX(0); opacity: 1; }

        #invisible-input { position: fixed; opacity: 0; }
        .combo-meter { font-weight: bold; color: var(--accent); font-size: 20px; }
    </style>
</head>
<body onclick="document.getElementById('invisible-input').focus()">

    <div class="progress-container"><div id="progress-bar"></div></div>

    <div class="control-panel">
        <select id="song-selector" onchange="initSong()" style="padding: 8px; border-radius: 5px;">
            <option value="">-- 選擇曲目 --</option>
            <option value="song2">This Is Me</option>
        </select>
        <div class="combo-meter">Combo: <span id="combo-count">0</span></div>
        <div id="accuracy-display" style="font-weight: bold;">進度: 0%</div>
    </div>

    <div class="main-content" id="layout" style="display:none;">
        <div class="practice-area">
            <div id="text-display"></div>
            <textarea id="invisible-input" spellcheck="false" autocomplete="off"></textarea>
        </div>

        <div class="side-panel" id="side-panel">
            <div id="vocab-hint" class="info-card">
                <div style="color:var(--primary); font-weight:bold; margin-bottom:5px;">重點單字</div>
                <div id="hint-content"></div>
            </div>
            <div id="trans-hint" class="info-card" style="border-left-color: #9b59b6;">
                <div style="color:#9b59b6; font-weight:bold; margin-bottom:5px;">中文意譯</div>
                <div id="trans-content" style="font-size: 14px; color: #666;"></div>
            </div>
        </div>
    </div>

    <script>
        const songLibrary = {
            "song2": {
                "lyrics": "I am not a stranger to the dark\nHide away they say\nCause we dont want your broken parts\nIve learned to be ashamed of all my scars",
                "translation": "我不畏懼黑暗，他們說躲起來吧，我們不需要你破碎的部分。",
                "vocab": [
                    { "trigger": "stranger", "kk": "/ˈstreɪndʒər/", "note": "陌生人" },
                    { "trigger": "ashamed", "kk": "/əˈʃeɪmd/", "note": "感到羞愧的" },
                    { "trigger": "scars", "kk": "/skɑːrz/", "note": "疤痕" }
                ]
            }
        };

        let currentData = null;
        let combo = 0;
        const display = document.getElementById('text-display');
        const input = document.getElementById('invisible-input');

        function initSong() {
            const val = document.getElementById('song-selector').value;
            currentData = songLibrary[val];
            if (!currentData) return;

            document.getElementById('layout').style.display = 'grid';
            document.getElementById('trans-content').innerText = currentData.translation;
            document.getElementById('trans-hint').classList.add('active');
            renderDisplay("");
            input.value = "";
            input.focus();
        }

        input.addEventListener('input', () => {
            const val = input.value;
            renderDisplay(val);
            updateProgress(val);
            checkVocab(val);
        });

        function renderDisplay(userInput) {
            let html = "";
            const target = currentData.lyrics;
            
            for (let i = 0; i < target.length; i++) {
                if (i < userInput.length) {
                    if (userInput[i] === target[i]) {
                        html += `<span class="char-correct">${target[i]}</span>`;
                        if (i === userInput.length - 1) combo++;
                    } else {
                        html += `<span class="char-wrong">${target[i]}</span>`;
                        combo = 0;
                    }
                } else {
                    html += `<span>${target[i]}</span>`;
                }
            }
            display.innerHTML = html;
            document.getElementById('combo-count').innerText = combo;
        }

        function updateProgress(userInput) {
            const progress = Math.min((userInput.length / currentData.lyrics.length) * 100, 100);
            document.getElementById('progress-bar').style.width = progress + "%";
            document.getElementById('accuracy-display').innerText = `進度: ${Math.floor(progress)}%`;
        }

        function checkVocab(userInput) {
            const words = userInput.toLowerCase().split(/\s+/);
            const lastWord = words[words.length - 1];
            const found = currentData.vocab.find(v => v.trigger.toLowerCase() === lastWord);
            
            const hintCard = document.getElementById('vocab-hint');
            if (found) {
                document.getElementById('hint-content').innerHTML = `<b>${found.trigger}</b> <span style="color:#e67e22">${found.kk}</span><br>${found.note}`;
                hintCard.classList.add('active');
            } else {
                // 如果一段時間沒打到新單字，可以考慮隱藏或保留
            }
        }
    </script>
</body>
</html>
