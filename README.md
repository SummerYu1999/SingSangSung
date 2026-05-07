<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - 歌詞背誦庫</title>
    <style>
        :root { --primary: #4a90e2; --bg: #f8f9fa; --text: #333; --gray: #d3d3d3; --success: #28a745; }
        body { font-family: "Microsoft JhengHei", sans-serif; background: var(--bg); display: flex; flex-direction: column; align-items: center; padding: 20px; color: var(--text); }
        .control-panel { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 30px; width: 90%; max-width: 600px; text-align: center; }
        select, button { padding: 10px; font-size: 16px; border-radius: 6px; border: 1px solid #ccc; margin: 5px; cursor: pointer; }
        button { background: var(--primary); color: white; border: none; transition: 0.3s; }
        button:hover { opacity: 0.8; }
        
        .container { position: relative; width: 90%; max-width: 800px; font-size: 24px; line-height: 1.8; letter-spacing: 2px; min-height: 300px; }
        #lyric-placeholder { color: var(--gray); white-space: pre-wrap; word-wrap: break-word; position: absolute; top: 0; left: 0; z-index: 1; pointer-events: none; width: 100%; }
        #lyric-display { color: var(--text); white-space: pre-wrap; word-wrap: break-word; position: relative; z-index: 2; width: 100%; }
        #lyric-input { position: fixed; opacity: 0; }
        
        .cursor { border-left: 3px solid var(--primary); animation: blink 1s infinite; margin-left: 2px; }
        @keyframes blink { 50% { opacity: 0; } }
        .completed-tag { color: var(--success); font-weight: bold; margin-top: 10px; display: none; }
    </style>
</head>
<body onclick="focusInput()">

    <div class="control-panel">
        <h2>🎵 SingSangSung 練習庫</h2>
        <select id="song-selector" onchange="loadSong()">
            <option value="">-- 請選擇歌曲 --</option>
            <option value="song1">範例歌曲：小幸運</option>
            <option value="song2">範例歌曲：倔強</option>
            <option value="custom">自定義貼上...</option>
        </select>
        <div id="status-display" class="completed-tag">此首已背誦完成 ✅</div>
    </div>

    <div class="container" id="practice-area" style="display:none;">
        <div id="lyric-placeholder"></div>
        <div id="lyric-display"></div>
        <input type="text" id="lyric-input" autocomplete="off">
    </div>

    <script>
        // --- 這裡是你管理歌詞庫的地方 ---
        const songLibrary = {
            "song1": "我聽見雨落在青草地\n我聽見遠方下課鐘聲響起",
            "song2": "當我和世界不一樣\n那就讓我不一樣",
        };

        const selector = document.getElementById('song-selector');
        const placeholder = document.getElementById('lyric-placeholder');
        const display = document.getElementById('lyric-display');
        const input = document.getElementById('lyric-input');
        const practiceArea = document.getElementById('practice-area');
        const statusDisplay = document.getElementById('status-display');

        let currentLyrics = "";

        function loadSong() {
            const selected = selector.value;
            statusDisplay.style.display = 'none';
            input.value = "";
            display.innerHTML = '<span class="cursor"></span>';

            if (selected === "custom") {
                const custom = prompt("請貼上你想背誦的歌詞：");
                if (custom) {
                    currentLyrics = custom;
                    startPractice();
                }
            } else if (songLibrary[selected]) {
                currentLyrics = songLibrary[selected];
                startPractice();
                checkRecord(selected);
            } else {
                practiceArea.style.display = 'none';
            }
        }

        function startPractice() {
            practiceArea.style.display = 'block';
            placeholder.innerText = currentLyrics;
            setTimeout(() => input.focus(), 100);
        }

        function focusInput() {
            if (practiceArea.style.display === 'block') input.focus();
        }

        function checkRecord(songId) {
            if (localStorage.getItem('done_' + songId)) {
                statusDisplay.style.display = 'block';
            }
        }

        input.addEventListener('input', () => {
            const userIn = input.value;
            let resultHTML = '';
            
            for (let i = 0; i < userIn.length; i++) {
                if (userIn[i] === currentLyrics[i]) {
                    resultHTML += `<span style="color: #000;">${currentLyrics[i]}</span>`;
                } else {
                    resultHTML += `<span style="color: #ff4d4d;">${currentLyrics[i]}</span>`;
                }
            }
            
            display.innerHTML = resultHTML + '<span class="cursor"></span>';
            
            if (userIn === currentLyrics && currentLyrics !== "") {
                const songId = selector.value;
                if (songId && songId !== "custom") {
                    localStorage.setItem('done_' + songId, 'true');
                    statusDisplay.style.display = 'block';
                }
                alert('太棒了！背誦完全正確！');
            }
        });
    </script>
</body>
</html>
