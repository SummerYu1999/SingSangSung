<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SingSangSung - Smart Focus</title>

<style>
:root{
    --primary:#74b9ff;
    --accent:#ff7675;
    --bg:#2d3436;
    --card:#353b48;
    --text:#dfe6e9;
    --highlight:#ffeaa7;
}

*{
    box-sizing:border-box;
}

body{
    font-family:'Segoe UI','Microsoft JhengHei',sans-serif;
    background:var(--bg);
    color:var(--text);
    margin:0;
    overflow:hidden;
    display:flex;
    flex-direction:column;
    height:100vh;
}

header{
    background:rgba(0,0,0,.8);
    padding:12px 25px;
    display:flex;
    justify-content:space-between;
    align-items:center;
}

.controls{
    display:flex;
    gap:15px;
    align-items:center;
}

.main-game{
    display:grid;
    grid-template-columns:1fr 300px;
    gap:20px;
    padding:20px;
    flex:1;
    max-width:1400px;
    margin:auto;
    width:100%;
}

.practice-window{
    position:relative;
    overflow:hidden;
    background:rgba(0,0,0,.4);
    border-radius:20px;
    padding:60px 40px;
    border:1px solid #444;
}

#scroll-engine{
    transition:transform .35s ease;
    will-change:transform;
}

#text-target{
    white-space:pre-wrap;
    font-size:32px;
    line-height:2;
    font-weight:700;
    color:#555;
}

.char-ok{
    color:#fff;
    text-shadow:0 0 10px var(--primary);
}

.char-no{
    color:var(--accent);
    background:rgba(255,118,117,.15);
    border-radius:4px;
}

.hard-word{
    border-bottom:2px dashed var(--highlight);
}

.hard-word.active{
    color:var(--highlight);
    text-shadow:0 0 10px var(--highlight);
}

.cursor{
    border-left:3px solid var(--primary);
    display:inline-block;
    height:1.1em;
    margin-left:2px;
    vertical-align:middle;
    animation:blink .8s infinite;
}

@keyframes blink{
    50%{
        opacity:0;
    }
}

#invisible-input{
    position:absolute;
    opacity:0;
    left:-9999px;
}

.sidebar{
    display:flex;
    flex-direction:column;
    gap:15px;
}

.card{
    background:var(--card);
    padding:20px;
    border-radius:14px;
    border-left:5px solid var(--primary);
}

.card.hidden{
    display:none;
}

.emoji-particle{
    position:absolute;
    pointer-events:none;
    z-index:999;
    font-size:30px;
    animation:emoji-fly 1s forwards ease-out;
}

@keyframes emoji-fly{
    0%{
        transform:translate(0,0) scale(.5);
        opacity:0;
    }

    20%{
        opacity:1;
    }

    100%{
        transform:
        translate(var(--dx),var(--dy))
        rotate(var(--dr))
        scale(1.4);

        opacity:0;
    }
}
</style>
</head>

<body onclick="focusInput()">

<header>
    <div class="controls">

        <span style="font-size:22px;font-weight:900;color:var(--primary)">
            SingSangSung
        </span>

        <select id="song-select" onchange="loadSong()"
        style="background:#444;color:white;border:none;padding:8px 14px;border-radius:8px">

            <option value="">-- 選擇曲目 --</option>
            <option value="song2">This Is Me</option>

        </select>
    </div>

    <div id="stat-display"
    style="font-family:monospace;font-size:18px;color:var(--primary);font-weight:bold">
        Combo: 0 | 0%
    </div>
</header>

<div class="main-game" id="game-view" style="display:none">

    <div class="practice-window" id="window-root">

        <div id="scroll-engine">
            <div id="text-target"></div>
        </div>

        <textarea id="invisible-input"></textarea>

    </div>

    <div class="sidebar">

        <div class="card hidden" id="card-vocab">
            <div id="vocab-hint"></div>
        </div>

        <div class="card" style="border-left-color:#a29bfe">
            <div id="trans-hint">
                請開始打字...
            </div>
        </div>

    </div>
</div>

<script>

const songData = {

song2:{

lyrics:[
"i am not a stranger to the dark",
"hide away they say",
"cause we dont want your broken parts",
"ive learned to be ashamed of all my scars",
"run away they say",
"no one will love you as you are",
"this is me"
],

translations:[
"我對黑暗並不陌生",
"他們說躲起來",
"因為我們不要殘破的你",
"我曾為傷痕羞愧",
"他們說快逃",
"沒人會愛真正的你",
"這就是我"
],

vocab:[
{
word:"stranger",
index:0,
k:"[ˈstrendʒɚ]",
n:"陌生人"
},
{
word:"ashamed",
index:3,
k:"[əˈʃemd]",
n:"羞愧的"
},
{
word:"scars",
index:3,
k:"[skɑrz]",
n:"傷痕"
}
],

backingVocals:[
{
trigger:"me",
index:6,
text:"✨ THIS IS ME ✨"
}
]

}

};

let current=null;
let fullLyricsString="";
let lineThresholds=[];

function focusInput(){
    document.getElementById("invisible-input").focus();
}

window.addEventListener("load",focusInput);

function loadSong(){

    const key=document.getElementById("song-select").value;

    current=songData[key];

    if(!current) return;

    fullLyricsString="";
    lineThresholds=[];

    let count=0;

    current.lyrics.forEach(line=>{

        fullLyricsString += line + "\n";

        count += line.length + 1;

        lineThresholds.push(count);

    });

    document.getElementById("game-view").style.display="grid";

    document.getElementById("invisible-input").value="";

    render();

    focusInput();
}

function getPos(lineIdx,word){

    let pos=0;

    for(let i=0;i<lineIdx;i++){

        pos += current.lyrics[i].length + 1;

    }

    return pos + current.lyrics[lineIdx].indexOf(word);
}

function render(){

    const val=document.getElementById("invisible-input").value;

    let html="";

    for(let i=0;i<fullLyricsString.length;i++){

        let cls="";

        if(i < val.length){

            cls = val[i] === fullLyricsString[i]
            ? "char-ok"
            : "char-no";
        }

        if(i === val.length){

            html += `<span class="cursor"></span>`;
        }

        const vocab=current.vocab.find(v=>{

            const start=getPos(v.index,v.word);

            return i >= start && i < start + v.word.length;

        });

        if(vocab){

            html += `
            <span class="${cls} hard-word">
                ${fullLyricsString[i]}
            </span>
            `;

        }else{

            html += `
            <span class="${cls}">
                ${fullLyricsString[i]}
            </span>
            `;
        }
    }

    document.getElementById("text-target").innerHTML=html;

    updateScroll();
}

function updateScroll(){

    const cursor=document.querySelector(".cursor");

    if(!cursor) return;

    const offset=cursor.offsetTop;

    const y=Math.max(0,offset-120);

    document.getElementById("scroll-engine").style.transform =
    `translateY(-${y}px)`;
}

function updateStats(){

    const val=document.getElementById("invisible-input").value;

    let correct=0;

    for(let i=0;i<val.length;i++){

        if(val[i] === fullLyricsString[i]){
            correct++;
        }
    }

    const percent = val.length
    ? Math.floor((correct / val.length)*100)
    : 0;

    document.getElementById("stat-display").innerText =
    `Combo: ${val.length} | ${percent}%`;

    let lineIdx=lineThresholds.findIndex(t=>val.length < t);

    if(lineIdx === -1){
        lineIdx=current.lyrics.length-1;
    }

    document.getElementById("trans-hint").innerText =
    current.translations[lineIdx];

    const vocabCard=document.getElementById("card-vocab");

    const vocab=current.vocab.find(v=>{

        const start=getPos(v.index,v.word);

        return val.length >= start-3
        && val.length <= start + v.word.length + 3;
    });

    if(vocab){

        vocabCard.classList.remove("hidden");

        document.getElementById("vocab-hint").innerHTML = `
        <div style="font-size:12px;color:var(--highlight)">
            KEY WORD
        </div>

        <div style="font-size:26px;color:var(--primary);font-weight:bold">
            ${vocab.word}
        </div>

        <div style="color:#aaa">
            ${vocab.k}
        </div>

        <div style="font-size:18px">
            ${vocab.n}
        </div>
        `;

    }else{

        vocabCard.classList.add("hidden");
    }
}

function spawnEffect(text){

    const cursor=document.querySelector(".cursor");

    if(!cursor) return;

    const rect=cursor.getBoundingClientRect();

    const root=document.getElementById("window-root");

    const rootRect=root.getBoundingClientRect();

    const el=document.createElement("div");

    el.className="emoji-particle";

    el.innerText=text;

    el.style.left=(rect.left-rootRect.left)+"px";

    el.style.top=(rect.top-rootRect.top)+"px";

    el.style.setProperty("--dx",(Math.random()-.5)*200+"px");

    el.style.setProperty("--dy",(-100-Math.random()*120)+"px");

    el.style.setProperty("--dr",(Math.random()*90-45)+"deg");

    root.appendChild(el);

    setTimeout(()=>{

        el.remove();

    },1000);
}

document.getElementById("invisible-input")
.addEventListener("input",(e)=>{

    render();

    updateStats();

    const val=e.target.value;

    const words=val.trim().split(/\s+/);

    const lastWord=words[words.length-1];

    const effect=current.backingVocals.find(v=>
        v.trigger === lastWord
    );

    if(effect){

        spawnEffect(effect.text);

    }else if(e.data){

        spawnEffect("✨");
    }

});

</script>

</body>
</html>
