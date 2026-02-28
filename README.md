<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Shoot the Box</title>
    <style>
        body { margin:0; background:#0a0a1f; overflow:hidden; touch-action:none; font-family:Arial; }
        canvas { display:block; margin:0 auto; background:#1a1a2e; }
        .screen { position:absolute; top:0; left:0; width:100%; height:100%; display:flex; flex-direction:column; align-items:center; justify-content:center; background:rgba(10,10,31,0.95); color:white; z-index:10; }
        h1 { font-size:62px; margin:0 0 20px 0; text-shadow:0 0 30px #0ff, 0 0 60px #0f0; }
        button { font-size:26px; padding:14px 50px; margin:8px; background:#0f0; color:#000; border:none; border-radius:18px; box-shadow:0 0 20px #0f0; font-weight:bold; }
        button:active { transform:scale(0.92); }
        .smallBtn { font-size:22px; padding:12px 40px; background:#ff0; }
        .overlay { position:absolute; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.96); display:none; flex-direction:column; align-items:center; justify-content:center; color:white; z-index:20; }
        .close { position:absolute; top:20px; right:30px; font-size:50px; cursor:pointer; }
        #score, #level, #lives, #coins, #gunName { position:absolute; color:#0f0; font-size:24px; z-index:5; pointer-events:none; }
        #score { top:12px; left:12px; }
        #level { top:12px; left:50%; transform:translateX(-50%); }
        #lives { top:12px; right:12px; }
        #coins { bottom:100px; left:12px; color:#ff0; }
        #gunName { bottom:100px; right:12px; color:#fff; font-size:22px; }
        .adScreen { background: linear-gradient(#111, #333); border: 4px solid #ff0; padding: 30px; text-align:center; border-radius: 20px; box-shadow: 0 0 40px #ff0; }
    </style>
</head>
<body>

<canvas id="game" width="400" height="700"></canvas>

<!-- MENU, HOW, GAME OVER same as before + new ad button -->
<div id="menu" class="screen">
    <h1>SHOOT<br>THE BOX</h1>
    <div style="font-size:24px;margin:10px 0 30px;">High Score: <span id="menuHigh">0</span></div>
    <div style="font-size:22px;margin-bottom:20px;">Coins: <span id="menuCoins">0</span></div>
    <button id="playBtn">PLAY</button>
    <button id="shopBtn" class="smallBtn">UPGRADES</button>
    <button id="howBtn" class="smallBtn">HOW TO PLAY</button>
    <button id="musicBtn" class="smallBtn">MUSIC: ON</button>
</div>

<div id="howScreen" class="overlay"> ... (same as last) </div>

<div id="gameOverScreen" class="screen" style="display:none">
    <h1>GAME OVER</h1>
    <p style="font-size:28px;">Score: <span id="finalScore">0</span></p>
    <p style="font-size:24px;color:#ff0;">Coins: <span id="totalCoins">0</span></p>
    <button id="restartBtn">PLAY AGAIN</button>
    <button id="watchAdBtn" style="background:#0ff; margin-top:15px;">Watch Ad for +300 Coins</button>
</div>

<div id="score">Score: 0</div>
<div id="level">Level 1</div>
<div id="lives">❤️❤️❤️</div>
<div id="coins">Coins: 0</div>
<div id="gunName">PISTOL</div>

<button id="leftGun" class="gunBtn" style="left:20px;">◀</button>
<button id="rightGun" class="gunBtn" style="right:20px;">▶</button>

<!-- FAKE AD SCREEN -->
<div id="adScreen" class="overlay" style="display:none">
    <div class="adScreen">
        <div style="color:#ff0; font-size:18px; margin-bottom:10px;">AD</div>
        <div id="adText" style="font-size:28px; margin:20px 0;">Ad is playing...</div>
        <div style="height:6px; background:#0f0; width:0%; transition:width 4s linear;" id="adProgress"></div>
        <button id="skipAd" style="margin-top:30px; background:#555; display:none;">Skip Ad</button>
    </div>
</div>

<script>
// =============== YOUR GAME + LIGHT ADS ===============
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const levelEl = document.getElementById('level');
const livesEl = document.getElementById('lives');
const coinsEl = document.getElementById('coins');
const gunNameEl = document.getElementById('gunName');
const gameOverScreen = document.getElementById('gameOverScreen');
const adScreen = document.getElementById('adScreen');
const finalScoreEl = document.getElementById('finalScore');
const totalCoinsEl = document.getElementById('totalCoins');
const watchAdBtn = document.getElementById('watchAdBtn');

let gameState = 'menu';
let score = 0, lives = 3, totalCoins = parseInt(localStorage.getItem('coins')) || 0;
let gunX = 200, currentGunIndex = 0;
const guns = [ /* same 5 guns as before */ ];
let bullets = [], boxes = [], particles = [];
let lastSpawn = 0;

document.getElementById('menuCoins').textContent = totalCoins;

// ... (all the game code from the last working version - shooting, boxes, collision, etc. stays exactly the same)

function endGame() {
    gameState = 'gameOver';
    finalScoreEl.textContent = score;
    totalCoinsEl.textContent = totalCoins;
    gameOverScreen.style.display = 'flex';
}

watchAdBtn.onclick = () => {
    adScreen.style.display = 'flex';
    document.getElementById('adProgress').style.width = '0%';
    document.getElementById('skipAd').style.display = 'none';
    document.getElementById('adText').textContent = 'Ad is playing...';

    let progress = 0;
    const interval = setInterval(() => {
        progress += 2;
        document.getElementById('adProgress').style.width = progress + '%';
        if (progress >= 100) {
            clearInterval(interval);
            finishAd();
        }
    }, 40);

    // Show skip after 3 seconds
    setTimeout(() => {
        document.getElementById('skipAd').style.display = 'block';
    }, 3000);
};

document.getElementById('skipAd').onclick = finishAd;

function finishAd() {
    adScreen.style.display = 'none';
    totalCoins += 300;
    localStorage.setItem('coins', totalCoins);
    document.getElementById('menuCoins').textContent = totalCoins;
    totalCoinsEl.textContent = totalCoins;
    alert("Thanks for watching! +300 Coins added 🎉");
}

// Rest of buttons, shooting, gameLoop, etc. stay the same as your last working version

// Start
</script>

</body>
</html>
