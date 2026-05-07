<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - 專業背誦庫</title>
    <style>
        :root { --primary: #4a90e2; --bg: #f8f9fa; --text: #333; --gray: #d3d3d3; --border: #e0e0e0; }
        body { font-family: "Segoe UI", "Microsoft JhengHei", sans-serif; background: var(--bg); display: flex; flex-direction: column; align-items: center; padding: 20px; color: var(--text); }
        .control-panel { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.05); margin-bottom: 20px; width: 95%; max-width: 850px; text-align: center; }
        
        .main-layout { display: flex; flex-direction: column; width: 95%; max-width: 850px; gap: 20px; }
        
        /* 練習區域 */
        .practice-container { position: relative; font-size: 22px; line-height: 1.8; letter-spacing: 1px; background: white; padding: 20px; border-radius: 12px; border: 1px solid var(--border); min-height: 300px; }
        #lyric-target { color: var(--gray); white-space: pre-wrap; word-wrap: break-word; width: 100%; user-select: none; }
        #lyric-input-overlay { position: absolute; top: 20px; left: 20px; width: calc(100% - 40px); height: calc(100% - 40px); background: transparent; border: none; outline: none; color: #000; font-family: inherit; font-size: inherit; line-height: inherit; letter-spacing: inherit; white-space: pre-wrap; word-wrap: break-word; resize: none; overflow: hidden; z-index: 2; }

        /* 資訊補充區域 (中文翻譯與 KK 音標) */
        .info-panel { background: #fff; padding: 20px; border-radius: 12px; border: 1px solid var(--border); }
        .translation-title { font-weight: bold; color: var(--primary); margin-bottom: 8px; border-left: 4px solid var(--primary); padding-left: 10px; }
        #translation-text { color: #666; line-height: 1.6; margin-bottom: 20px; font-size: 16px; white-space: pre-wrap; }
        
        .vocab-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 12px; }
        .vocab-item { padding: 8px 12px; background: #f0f4f8; border-radius: 6px; font-size: 14px; }
        .vocab-word { color: var(--primary); font-weight: bold; margin-right: 5px; }
        .vocab-kk { color: #e67e22; font-family: "Arial", sans-serif; font-size: 13px; }
        .vocab-note { display: block; color: #555; margin-top: 2px; }
    </style>
</head>
<body onclick="focusInput()">

    <div class="control-panel">
        <h2>🎵 SingSangSung 語言進階練習</h2>
        <select id="song-selector" onchange="initSong()">
            <option value="">-- 請選擇歌曲 --</option>
            <option value="song2">英文：This Is Me (The Greatest Showman)</option>
        </select>
    </div>

    <div class="main-layout" id="main-layout" style="display:none;">
        <div class="practice-container">
            <div id="lyric-target"></div>
            <textarea id="lyric-input-overlay" spellcheck="false" autocomplete="off"></textarea>
        </div>

        <div class="info-panel">
            <div class="translation-title">歌詞義譯</div>
            <div id="translation-text"></div>
            
            <div class="translation-title">進階單字 (TOEIC 840+)</div>
            <div class="vocab-grid" id="vocab-grid"></div>
        </div>
    </div>

    <script>
        const songLibrary = {
            "song2": {
                "lyrics": "I am not a stranger to the dark\nHide away they say\nCause we dont want your broken parts\nIve learned to be ashamed of all my scars\nRun away they say\nNo onell love you as you are\nBut I wont let them break me down to dust\nI know that there's a place for us\nFor we are glorious", 
                // 先提供部分歌詞供試驗，結構已完整
                "translation": "我對黑暗並不陌生\n他們說：躲起來吧\n因為我們不需要你那破碎的身軀\n我曾學會為我身上的疤痕感到羞恥\n他們說：逃跑吧\n沒人會愛你真實的樣子\n但我不會讓他們將我擊碎成粉塵\n我知道有處屬於我們的地方\n因為我們是如此燦爛輝煌",
                "vocabulary": [
                    { "word": "Stranger", "kk": "/ˈstreɪndʒər/", "note": "陌生人、外行人" },
                    { "word": "Ashamed", "kk": "/əˈʃeɪmd/", "note": "感到羞愧的 (TOEIC 常考情緒形容詞)" },
                    { "word": "Scars", "kk": "/skɑːrz/", "note": "疤痕、創傷" },
                    { "word": "Glorious", "kk": "/ˈɡlɔːriəs/", "note": "輝煌的、壯爛的" },
                    { "word": "Dust", "kk": "/dʌst/", "note": "塵土、粉末 (此處指毀滅)" }
                ]
            }
        };

        const targetDiv = document.getElementById('lyric-target');
        const inputArea = document.getElementById('lyric-input-overlay');
        const layout = document.getElementById('main-layout');
        const transText = document.getElementById('translation-text');
        const vocabGrid = document.getElementById('vocab-grid');
        let currentFullText = "";

        function initSong() {
            const val = document.getElementById('song-selector').value;
            const data = songLibrary[val];
            if (data) {
                currentFullText = data.lyrics;
                targetDiv.innerText = currentFullText;
                transText.innerText = data.translation;
                
                // 渲染單字表
                vocabGrid.innerHTML = data.vocabulary.map(v => `
                    <div class="vocab-item">
                        <span class="vocab-word">${v.word}</span>
                        <span class="vocab-kk">${v.kk}</span>
                        <span class="vocab-note">${v.note}</span>
                    </div>
                `).join('');

                inputArea.value = "";
                layout.style.display = 'flex';
                setTimeout(() => { syncHeight(); inputArea.focus(); }, 100);
            }
        }

        inputArea.addEventListener('input', () => {
            const userInput = inputArea.value;
            // 比對邏輯：打錯時變紅
            if (!currentFullText.startsWith(userInput)) {
                inputArea.style.color = "#ff4d4d";
            } else {
                inputArea.style.color = "#000";
            }
            if (userInput === currentFullText && currentFullText !== "") {
                alert("完全正確！You are glorious!");
            }
            syncHeight();
        });

        function syncHeight() {
            inputArea.style.height = targetDiv.offsetHeight + "px";
        }

        function focusInput() {
            if(layout.style.display === 'flex') inputArea.focus();
        }
    </script>
</body>
</html>
