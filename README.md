<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - 中文背誦版</title>
    <style>
        :root { --primary: #4a90e2; --bg: #f8f9fa; --text: #333; --gray: #d3d3d3; }
        body { font-family: "Microsoft JhengHei", sans-serif; background: var(--bg); display: flex; flex-direction: column; align-items: center; padding: 20px; }
        .control-panel { background: white; padding: 15px; border-radius: 12px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-bottom: 20px; width: 90%; max-width: 600px; text-align: center; }
        
        /* 核心容器 */
        .practice-container {
            position: relative;
            width: 90%;
            max-width: 800px;
            font-size: 28px;
            line-height: 2;
            letter-spacing: 4px;
            margin-top: 20px;
        }

        /* 淺灰色底字 (目標歌詞) */
        #lyric-target {
            color: var(--gray);
            white-space: pre-wrap;
            word-wrap: break-word;
            width: 100%;
            user-select: none;
        }

        /* 實際輸入覆蓋層 */
        #lyric-input-overlay {
            position: absolute;
            top: 0; left: 0;
            width: 100%;
            height: 100%;
            background: transparent;
            border: none;
            outline: none;
            color: #000; /* 輸入正確後的顏色 */
            font-family: inherit;
            font-size: inherit;
            line-height: inherit;
            letter-spacing: inherit;
            white-space: pre-wrap;
            word-wrap: break-word;
            resize: none;
            overflow: hidden;
            z-index: 2;
        }

        /* 正在輸入中的注音/文字標示 (底線效果) */
        #lyric-input-overlay::selection { background: rgba(74, 144, 226, 0.2); }
    </style>
</head>
<body>

    <div class="control-panel">
        <h2>🎵 SingSangSung 中文背誦</h2>
        <select id="song-selector" onchange="initSong()">
            <option value="">-- 請選擇歌曲 --</option>
            <option value="song1">小幸運</option>
            <option value="song2">倔強</option>
        </select>
        <p style="font-size: 14px; color: #888;">直接在歌詞上打字，支援注音輸入法</p>
    </div>

    <div class="practice-container" id="area" style="display:none;">
        <div id="lyric-target"></div>
        <textarea id="lyric-input-overlay" spellcheck="false" autofocus></textarea>
    </div>

    <script>
        const songLibrary = {
            "song1": "我聽見雨落在青草地\n我聽見遠方下課鐘聲響起",
            "song2": "當我和世界不一樣\n那就讓我不一樣",
        };

        const targetDiv = document.getElementById('lyric-target');
        const inputArea = document.getElementById('lyric-input-overlay');
        const area = document.getElementById('area');
        let currentFullText = "";

        function initSong() {
            const val = document.getElementById('song-selector').value;
            if (songLibrary[val]) {
                currentFullText = songLibrary[val];
                targetDiv.innerText = currentFullText;
                inputArea.value = "";
                area.style.display = 'block';
                inputArea.focus();
                syncHeight();
            }
        }

        // 監聽輸入事件
        inputArea.addEventListener('input', (e) => {
            const userInput = inputArea.value;
            
            // 檢查是否輸入錯誤（如果目前輸入的內容不是歌詞的前綴，標紅提醒）
            if (!currentFullText.startsWith(userInput)) {
                inputArea.style.color = "#ff4d4d"; // 錯誤變紅
            } else {
                inputArea.style.color = "#000"; // 正確變黑
            }

            if (userInput === currentFullText) {
                alert("恭喜！背誦完全正確！");
            }
            syncHeight();
        });

        // 讓輸入框高度跟著文字跑
        function syncHeight() {
            inputArea.style.height = targetDiv.offsetHeight + "px";
        }

        // 點擊區域自動聚焦
        document.body.addEventListener('click', () => inputArea.focus());
    </script>
</body>
</html>
