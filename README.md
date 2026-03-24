<!DOCTYPE html>
<html lang="en">
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        :root { --neon: #00f2ff; --pink: #ff00ff; --bg: #0d0221; }
        body { font-family: 'Courier New', monospace; background: var(--bg); color: var(--neon); margin: 0; text-align: center; overflow: hidden; touch-action: manipulation; }
        #ui-layer { height: 10vh; display: flex; justify-content: space-around; align-items: center; background: #1b1464; border-bottom: 2px solid var(--pink); }
        #game-area { height: 90vh; width: 100%; position: relative; }
        
        /* Menu Style */
        .menu-btn { background: none; border: 2px solid var(--neon); color: var(--neon); padding: 15px; margin: 10px; width: 80%; border-radius: 10px; font-weight: bold; cursor: pointer; }
        .menu-btn:hover { background: var(--neon); color: var(--bg); }

        /* Game Elements */
        .item { position: absolute; cursor: pointer; font-size: 30px; user-select: none; }
        .win-screen { position: absolute; top: 0; width: 100%; height: 100%; background: var(--bg); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 100; }
        #alien { font-size: 50px; position: absolute; bottom: 20px; transition: 0.1s; }
    </style>
</head>
<body>

    <div id="ui-layer">
        <div>LVL: <span id="lvl">1</span></div>
        <div>TIME: <span id="timer">60</span>s</div>
    </div>

    <div id="game-area">
        <!-- Main Menu -->
        <div id="menu">
            <h2 style="color: var(--pink); margin-top: 50px;">CHOOSE BREAK</h2>
            <button class="menu-btn" onclick="startGame('storm')">SPACE JUNK STORM</button>
            <button class="menu-btn" onclick="startGame('gate')">NEURAL GATE (REFLEX)</button>
            <button class="menu-btn" onclick="startGame('dash')">ALIEN ENERGY DASH</button>
        </div>
    </div>

    <!-- End Screen -->
    <div id="end-screen" class="win-screen" style="display: none;">
        <h1 id="end-title">TIME UP!</h1>
        <p>FINAL LEVEL: <b id="final-lvl" style="color: var(--pink)">0</b></p>
        <p>SECRET CODE: <b id="secret-code" style="color: white">---</b></p>
        <button class="menu-btn" onclick="location.reload()">PLAY AGAIN</button>
    </div>

    <script>
        let score = 0, level = 1, timeLeft = 60, activeGame = null, timerStarted = false;
        const area = document.getElementById('game-area');

        function startGame(type) {
            document.getElementById('menu').style.display = 'none';
            activeGame = type;
            startTimer();
            if(type === 'storm') spawnStorm();
            if(type === 'gate') startGate();
            if(type === 'dash') startDash();
        }

        function startTimer() {
            if(timerStarted) return;
            timerStarted = true;
            const t = setInterval(() => {
                timeLeft--;
                document.getElementById('timer').innerText = timeLeft;
                if(timeLeft <= 0) {
                    clearInterval(t);
                    showEnd();
                }
            }, 1000);
        }

        function showEnd() {
            area.innerHTML = "";
            document.getElementById('end-screen').style.display = 'flex';
            document.getElementById('final-lvl').innerText = level;
            // Generate a code based on game and level
            document.getElementById('secret-code').innerText = `${activeGame.toUpperCase()}-${level * 7}`;
        }

        // --- GAME 1: STORM ---
        function spawnStorm() {
            if(timeLeft <= 0) return;
            const j = document.createElement('div'); j.className = 'item'; j.innerHTML = "🛰️";
            j.style.left = Math.random()*80+"%"; j.style.top = Math.random()*80+"%";
            j.onclick = () => { 
                score++; level = Math.floor(score/5)+1; 
                document.getElementById('lvl').innerText = level;
                j.remove(); spawnStorm(); if(level > 2) spawnStorm();
            };
            area.appendChild(j);
            setInterval(() => { j.style.left = Math.random()*80+"%"; j.style.top = Math.random()*80+"%"; }, 800 - (level*50));
        }

        // --- GAME 2: GATE ---
        let gateState = "idle";
        function startGate() {
            area.innerHTML = `<div id="box" style="width:90%; height:300px; margin:5%; border:3px solid var(--neon); display:flex; align-items:center; justify-content:center; font-size:24px;">WAIT...</div>`;
            const box = document.getElementById('box');
            box.onclick = () => {
                if(gateState === "go") { level++; document.getElementById('lvl').innerText = level; gateLoop(); }
                else { box.style.background = "red"; setTimeout(gateLoop, 500); }
            };
            gateLoop();
        }
        function gateLoop() {
            gateState = "waiting"; const box = document.getElementById('box');
            box.style.background = "none"; box.innerText = "WAIT...";
            setTimeout(() => {
                if(timeLeft <= 0) return;
                gateState = "go"; box.style.background = "#39ff14"; box.innerText = "TAP!";
                setTimeout(() => { if(gateState === "go") gateLoop(); }, Math.max(200, 600 - (level*40)));
            }, Math.random()*2000 + 1000);
        }

        // --- GAME 3: DASH ---
        function startDash() {
            area.innerHTML = `<div id="alien">👽</div>`;
            const alien = document.getElementById('alien');
            alien.style.left = "50%";
            window.onclick = (e) => {
                let p = parseInt(alien.style.left);
                alien.style.left = (e.clientX < window.innerWidth/2 ? p - 15 : p + 15) + "%";
            };
            function drop() {
                if(timeLeft <= 0) return;
                const good = Math.random() > 0.3;
                const d = document.createElement('div'); d.className = 'item';
                d.innerHTML = good ? "⚡" : "☄️"; d.style.left = Math.random()*90+"%";
                area.appendChild(d);
                let y = 0;
                let f = setInterval(() => {
                    y += (5 + level); d.style.top = y + "px";
                    if(y > window.innerHeight - 100 && Math.abs(alien.offsetLeft - d.offsetLeft) < 40) {
                        if(good) { score++; level = Math.floor(score/5)+1; document.getElementById('lvl').innerText = level; }
                        else { score = Math.max(0, score-1); }
                        clearInterval(f); d.remove(); drop();
                    } else if (y > window.innerHeight) { clearInterval(f); d.remove(); drop(); }
                }, 20);
            }
            drop(); drop();
        }
    </script>
</body>
</html>
