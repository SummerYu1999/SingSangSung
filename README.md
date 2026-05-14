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
            "I am not a stranger to the dark", "Hide away they say", "Cause we dont want your broken parts",
            "Ive learned to be ashamed of all my scars", "Run away they say", "No onell love you as you are",
            "But I wont let them break me down to dust", "I know that theres a place for us", "For we are glorious",
            "When the sharpest words wanna cut me down", "Im gonna send a flood gonna drown em out",
            "I am brave I am bruised", "I am who Im meant to be this is me", "Look out cause here I come",
            "And Im marching on to the beat I drum", "Im not scared to be seen", "I make no apologies this is me",
            "Another round of bullets hits my skin", "Well fire away cause today I wont let the shame sink in",
            "We are bursting through the barricades and", "Reaching for the sun we are warriors",
            "Yeah thats what weve become", "I wont let them break me down to dust",
            "I know that theres a place for us", "For we are glorious",
            "When the sharpest words wanna cut me down", "Im gonna send a flood gonna drown em out",
            "I am brave I am bruised", "I am who Im meant to be this is me",
            "Look out cause here I come", "And Im marching on to the beat I drum",
            "Im not scared to be seen", "I make no apologies this is me",
            "This is me", "And I know that I deserve your love", "There is nothing Im not worthy of",
            "When the sharpest words wanna cut me down", "Im gonna send a flood gonna drown em out",
            "This is brave this is bruised", "This is who Im meant to be this is me",
            "Look out cause here I come", "And Im marching on to the beat I drum",
            "Im not scared to be seen", "I make no apologies this is me",
            "Whenever the words wanna cut me down", "Ill send a flood to drown em out",
            "Im gonna send a flood", "Gonna drown them em out", "This is me"
        ],
        "translations": [
            "我不畏懼黑暗。", "他們說：躲起來吧。", "因為我們不需要你那破碎的部分。",
            "我曾學會為我身上的疤痕感到羞恥。", "他們說：逃跑吧。", "沒人會愛你原本的樣子。",
            "但我不會讓他們把我擊潰成灰燼。", "我知道有個地方是屬於我們的。", "因為我們是輝煌燦爛的。",
            "當最尖銳的言語想擊倒我時，", "我將發起洪流將它們淹沒。", "我很勇敢，我滿身傷痕，",
            "但我就是我該有的樣子，這就是我。", "小心點，因為我來了，", "我隨著自己的鼓聲闊步向前，",
            "我不害怕被看見，", "我不需要道歉，這就是我。", "另一輪子彈擊中我的皮膚，",
            "儘管開火吧，因為今天我不會讓羞恥感吞噬我。", "我們正衝破重重障礙，",
            "向太陽伸手，我們是英勇的戰士。", "是的，這就是我們蛻變後的樣子。",
            "我不會讓他們把我擊潰成灰燼。", "我知道有個地方是屬於我們的。", "因為我們是輝煌燦爛的。",
            "當最尖銳的言語想擊倒我時，", "我將發起洪流將它們淹沒。", "我很勇敢，我滿身傷痕，",
            "但我就是我該有的樣子，這就是我。", "小心點，因為我來了，", "我隨著自己的鼓聲闊步向前，",
            "我不害怕被看見，", "我不需要道歉，這就是我。", "這就是我。",
            "我知道我值得擁有你的愛。", "沒有什麼是我配不上的。", "當最尖銳的言語想擊倒我時，",
            "我將發起洪流將它們淹沒。", "這就是勇敢，這就是傷痕，", "這就是我該有的樣子，這就是我。",
            "小心點，因為我來了。", "我隨著自己的鼓聲闊步向前。", "我不害怕被看見。",
            "我不需要道歉，這就是我。", "每當那些言語想擊倒我時，", "我將發起洪流將它們淹沒。",
            "我將發起洪流，", "將它們通通淹沒。", "這就是我。"
        ],
        "vocab": [
            { "word": "stranger", "index": 0, "k": "[ˈstrendʒɚ]", "n": "陌生人" },
            { "word": "ashamed", "index": 3, "k": "[əˈʃemd]", "n": "感到羞愧的" },
            { "word": "scars", "index": 3, "k": "[skɑrz]", "n": "疤痕；創傷" },
            { "word": "glorious", "index": 8, "k": "[ˈɡloriəs]", "n": "輝煌燦爛的" },
            { "word": "sharpest", "index": 9, "k": "[ˈʃɑrpɪst]", "n": "最尖銳的" },
            { "word": "bruised", "index": 11, "k": "[bruzd]", "n": "受傷的；瘀青的" },
            { "word": "marching", "index": 14, "k": "[ˈmɑrtʃɪŋ]", "n": "闊步前進" },
            { "word": "apologies", "index": 16, "k": "[əˈpɑlədʒiz]", "n": "道歉" },
            { "word": "barricades", "index": 19, "k": "[ˌbærɪˈkedz]", "n": "路障；障礙物" },
            { "word": "warriors", "index": 20, "k": "[ˈwɔriɚz]", "n": "戰士；勇士" },
            { "word": "deserve", "index": 34, "k": "[dɪˈzɝv]", "n": "應得；值得" },
            { "word": "worthy", "index": 35, "k": "[ˈwɝði]", "n": "配得上...的" }
        ],
        // 背景合聲：當輸入到這些位置時觸發動畫
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

document.getElementById('invisible-input').addEventListener('input', (e) => {
    const val = e.target.value;
    render();
    updateStats();
    
    // 偵測是否觸發 Oh 動畫
    const lastSpaceIdx = val.lastIndexOf(" ");
    const lastWord = val.substring(lastSpaceIdx + 1).toLowerCase();
    let lineIdx = lineThresholds.findIndex(t => val.length < t);
    
    const bv = current.backingVocals.find(b => b.trigger === lastWord && b.index === lineIdx);
    if (bv) spawnOh(bv.text);
});

function spawnOh(text) {
    const cursor = document.querySelector('.cursor');
    if (!cursor) return;
    const rect = cursor.getBoundingClientRect();
    const winRect = document.getElementById('window-root').getBoundingClientRect();
    
    const oh = document.createElement('div');
    oh.className = 'emoji-particle';
    oh.innerText = text;
    oh.style.color = "var(--primary)";
    oh.style.fontSize = "24px";
    oh.style.fontWeight = "bold";
    oh.style.left = (rect.left - winRect.left) + "px";
    oh.style.top = (rect.top - winRect.top) + "px";
    oh.style.setProperty('--dx', (Math.random() - 0.5) * 100 + "px");
    oh.style.setProperty('--dy', "-150px");
    oh.style.setProperty('--dr', "0deg");
    document.getElementById('window-root').appendChild(oh);
    setTimeout(() => oh.remove(), 1000);
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
    document.getElementById('text-target').innerHTML = html;
    
    const curEl = document.querySelector('.cursor') || document.getElementById('text-target');
    document.getElementById('scroll-engine').style.transform = `translateY(-${curEl.offsetTop}px)`;
}

function getPos(lineIdx, word) {
    let pos = 0;
    for(let i=0; i<lineIdx; i++) pos += (current.lyrics[i] ? current.lyrics[i].length + 1 : 0);
    const lineText = current.lyrics[lineIdx];
    return pos + (lineText ? lineText.indexOf(word) : 0);
}

function updateStats() {
    const val = document.getElementById('invisible-input').value;
    let lineIdx = lineThresholds.findIndex(t => val.length < t);
    if (lineIdx === -1) lineIdx = current.lyrics.length - 1;
    document.getElementById('trans-hint').innerText = current.translations[lineIdx];

    const vocabCard = document.getElementById('card-vocab');
    const targetVocab = current.vocab.find(v => {
        const start = getPos(v.index, v.word);
        return val.length >= (start - 5) && val.length <= (start + v.word.length);
    });

    if (targetVocab) {
        vocabCard.classList.remove('hidden');
        document.getElementById('vocab-hint').innerHTML = `
            <div style="color:var(--highlight); font-size:12px;">KEY WORD</div>
            <b style="font-size:22px; color:var(--primary)">${targetVocab.word}</b> 
            <span style="color:#aaa;">${targetVocab.k}</span>
            <div style="margin-top:5px; font-size:18px;">${targetVocab.n}</div>`;
    } else {
        const nextVocab = current.vocab.find(v => getPos(v.index, v.word) > val.length);
        if (nextVocab) {
            vocabCard.classList.remove('hidden');
            document.getElementById('vocab-hint').innerHTML = `<div style="color:#666; font-size:12px;">NEXT</div><b style="color:#666; font-size:18px;">${nextVocab.word}</b>`;
        } else {
            vocabCard.classList.add('hidden');
        }
    }
}
    </script>
</body>
</html>
