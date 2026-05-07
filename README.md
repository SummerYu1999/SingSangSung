<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SingSangSung - 多語系背誦庫</title>
    <style>
        :root { --primary: #4a90e2; --bg: #f8f9fa; --text: #333; --gray: #d3d3d3; }
        body { font-family: "Segoe UI", "Microsoft JhengHei", sans-serif; background: var(--bg); display: flex; flex-direction: column; align-items: center; padding: 20px; }
        .control-panel { background: white; padding: 15px; border-radius: 12px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-bottom: 20px; width: 95%; max-width: 800px; text-align: center; }
        
        .practice-container {
            position: relative;
            width: 95%;
            max-width: 800px;
            font-size: 22px; /* 稍微縮小字體以適應較長英文句子 */
            line-height: 1.8;
            letter-spacing: 1px;
            margin-top: 10px;
        }

        #lyric-target {
            color: var(--gray);
            white-space: pre-wrap;
            word-wrap: break-word;
            width: 100%;
            user-select: none;
            padding: 10px;
        }

        #lyric-input-overlay {
            position: absolute;
            top: 0; left: 0;
            width: 100%;
            height: 100%;
            background: transparent;
            border: none;
            outline: none;
            color: #000;
            font-family: inherit;
            font-size: inherit;
            line-height: inherit;
            letter-spacing: inherit;
            white-space: pre-wrap;
            word-wrap: break-word;
            padding: 10px;
            resize: none;
            overflow: hidden;
            z-index: 2;
        }
    </style>
</head>
<body>

    <div class="control-panel">
        <h2>🎵 SingSangSung 語言練習庫</h2>
        <select id="song-selector" onchange="initSong()">
            <option value="">-- 請選擇歌曲 --</option>
            <option value="song1">中文：小幸運</option>
            <option value="song2">英文：This Is Me</option>
            <option value="song3">日文：(預留位置)</option>
        </select>
        <p style="font-size: 13px; color: #999;">支援中/英/日輸入，打錯字會變紅</p>
    </div>

    <div class="practice-container" id="area" style="display:none;">
        <div id="lyric-target"></div>
        <textarea id="lyric-input-overlay" spellcheck="false"></textarea>
    </div>

    <script>
        const songLibrary = {
            "song1": "我聽見雨落在青草地\n我聽見遠方下課鐘聲響起",
            "song2": "I am not a stranger to the dark\nHide away they say\nCause we dont want your broken parts\nIve learned to be ashamed of all my scars\nRun away they say\nNo onell love you as you are\nBut I wont let them break me down to dust\nI know that theres a place for us\nFor we are glorious\nWhen the sharpest words wanna cut me down\nIm gonna send a flood gonna drown em out\nI am brave I am bruised\nI am who Im meant to be this is me\nLook out cause here I come\nAnd Im marching on to the beat I drum\nIm not scared to be seen\nI make no apologies this is me\nOh-oh-oh-oh\nOh-oh-oh-oh\nOh-oh-oh-oh\nOh-oh-oh-oh\nOh-oh-oh oh-oh-oh oh-oh-oh oh oh\nAnother round of bullets hits my skin\nWell fire away cause today I wont let the shame sink in\nWe are bursting through the barricades and\nReaching for the sun we are warriors\nYeah thats what weve become yeah thats what weve become\nI wont let them break me down to dust\nI know that theres a place for us\nFor we are glorious\nWhen the sharpest words wanna cut me down\nIm gonna send a flood gonna drown em out\nI am brave I am bruised\nI am who Im meant to be this is me\nLook out cause here I come\nAnd Im marching on to the beat I drum\nIm not scared to be seen\nI make no apologies this is me\nOh-oh-oh-oh\nOh-oh-oh-oh\nOh-oh-oh-oh\nOh-oh-oh-oh\nOh-oh-oh oh-oh-oh oh-oh-oh oh oh\nThis is me\nand I know that I deserve your love\nOh-oh-oh-oh theres nothing Im not worthy of\nOh-oh-oh oh-oh-oh oh-oh-oh oh oh\nWhen the sharpest words wanna cut me down\nIm gonna send a flood gonna drown em out\nThis is brave this is bruised\nThis is who Im meant to be this is me\nLook out cause here I come look out cause here I come\nAnd Im marching on to the beat I drum marching on marching marching on\nI'm not scared to be seen\nI make no apologies this is me\nWhenever the words wanna cut me down oh-oh-oh-oh\nIll send a flood to drown em out oh oh-oh oh-oh\nIm gonna send a flood oh-oh-oh-oh\nGonna drown them em out oh-oh-oh oh-oh-oh oh-oh-oh oh oh\nOh\nThis is me",
            "song3": "こんにちは、これは日本語の練習です。"
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
                setTimeout(() => {
                    inputArea.focus();
                    syncHeight();
                }, 100);
            }
        }

        inputArea.addEventListener('input', () => {
            const userInput = inputArea.value;
            
            // 檢查前綴是否正確
            if (!currentFullText.startsWith(userInput)) {
                inputArea.style.color = "#ff4d4d"; // 錯誤變紅
            } else {
                inputArea.style.color = "#000"; // 正確變黑
            }

            if (userInput === currentFullText && currentFullText !== "") {
                alert("This is you! 背誦完全正確！");
            }
            syncHeight();
        });

        function syncHeight() {
            inputArea.style.height = targetDiv.offsetHeight + "px";
        }

        document.body.addEventListener('click', () => {
            if(area.style.display === 'block') inputArea.focus();
        });
    </script>
</body>
</html>
