<!DOCTYPE html>
<html lang="pl">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KANAŁ 121 - PREMIUM TV</title>
    <style>
        :root {
            --main-color: #00ff88;
            --bg-dark: rgba(10, 10, 10, 0.9);
        }

        body,
        html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background: #000;
            color: white;
            font-family: 'Segoe UI', Roboto, sans-serif;
            overflow: hidden;
        }

        /* WIDEO - TŁO */
        #tv-player {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            object-fit: cover;
            z-index: 1;
            transition: opacity 0.5s;
        }

        /* PASEK GÓRNY */
        #top-bar {
            position: absolute;
            top: 0;
            width: 100%;
            z-index: 10;
            background: linear-gradient(to bottom, rgba(0, 0, 0, 0.9), transparent);
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 20px 40px;
            box-sizing: border-box;
        }

        #ch-display {
            font-size: 32px;
            font-weight: 800;
            color: var(--main-color);
            letter-spacing: 2px;
        }

        #clock {
            font-size: 20px;
            font-family: monospace;
            background: rgba(255, 255, 255, 0.1);
            padding: 5px 15px;
            border-radius: 8px;
            backdrop-filter: blur(5px);
        }

        /* NAWIGACJA BOCZNA (PRZEZROCZYSTE STREFY) */
        .nav-zone {
            position: absolute;
            top: 0;
            height: 100%;
            width: 15%;
            z-index: 5;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: 0.3s;
            color: rgba(255, 255, 255, 0);
            font-size: 50px;
        }

        .nav-zone:hover {
            background: rgba(255, 255, 255, 0.05);
            color: rgba(255, 255, 255, 0.5);
        }

        #zone-prev {
            left: 0;
        }

        #zone-next {
            right: 0;
        }

        /* PANEL REŻYSERSKI (MODAL) */
        #admin-panel {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 90%;
            max-width: 400px;
            background: var(--bg-dark);
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 20px;
            padding: 30px;
            z-index: 1000;
            display: none;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
        }

        #admin-panel h2 {
            margin: 0 0 20px 0;
            font-size: 20px;
            color: var(--main-color);
            text-align: center;
            text-transform: uppercase;
        }

        /* FORMULARZ */
        label {
            display: block;
            margin: 10px 0 5px;
            font-size: 12px;
            color: #888;
            text-transform: uppercase;
        }

        input {
            width: 100%;
            padding: 12px;
            margin-bottom: 15px;
            border-radius: 10px;
            border: 1px solid #333;
            background: rgba(255, 255, 255, 0.05);
            color: white;
            box-sizing: border-box;
            outline: none;
        }

        input:focus {
            border-color: var(--main-color);
        }

        .btn-add {
            width: 100%;
            padding: 15px;
            border: none;
            border-radius: 10px;
            background: var(--main-color);
            color: black;
            font-weight: bold;
            cursor: pointer;
            text-transform: uppercase;
        }

        .btn-close {
            width: 100%;
            padding: 10px;
            margin-top: 10px;
            background: transparent;
            border: 1px solid #444;
            color: white;
            border-radius: 10px;
            cursor: pointer;
        }

        /* PRZYCISK MENU */
        #menu-trigger {
            position: absolute;
            bottom: 30px;
            left: 30px;
            z-index: 100;
            background: rgba(255, 255, 255, 0.15);
            border: 1px solid rgba(255, 255, 255, 0.2);
            padding: 15px 25px;
            border-radius: 50px;
            color: white;
            cursor: pointer;
            backdrop-filter: blur(10px);
            font-weight: bold;
            transition: 0.3s;
        }

        #menu-trigger:hover {
            background: var(--main-color);
            color: black;
        }

        /* RAMÓWKA (EPG) */
        #epg {
            position: absolute;
            bottom: 30px;
            right: 30px;
            width: 280px;
            background: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(10px);
            border-radius: 15px;
            padding: 15px;
            z-index: 10;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .prog-item {
            display: flex;
            justify-content: space-between;
            padding: 10px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.05);
            font-size: 14px;
        }

        .is-active {
            color: var(--main-color);
            font-weight: bold;
            background: rgba(0, 255, 136, 0.1);
            border-radius: 8px;
        }

        /* START OVERLAY */
        #start-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: #000;
            z-index: 9999;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
        }

        .start-btn {
            padding: 20px 40px;
            border: 2px solid var(--main-color);
            color: var(--main-color);
            border-radius: 50px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
        }
    </style>
</head>

<body>

    <div id="start-overlay" onclick="bootTV()">
        <div class="start-btn">Uruchom Kanał 121</div>
    </div>

    <div id="top-bar">
        <div id="ch-display">CH 121</div>
        <div id="clock">00:00:00</div>
    </div>

    <div id="zone-prev" class="nav-zone" onclick="changeCH(currentCH - 1)">‹</div>
    <div id="zone-next" class="nav-zone" onclick="changeCH(currentCH + 1)">›</div>

    <video id="tv-player" playsinline loop></video>

    <button id="menu-trigger" onclick="toggleAdmin(true)">REŻYSERKA</button>

    <div id="admin-panel">
        <h2>Ustawienia Emisji</h2>
        <label>Wybierz wideo</label>
        <input type="file" id="inp-file" accept="video/*">
        <label>Godzina rozpoczęcia</label>
        <input type="time" id="inp-time">
        <label>Tytuł programu</label>
        <input type="text" id="inp-title" placeholder="np. Wiadomości Wieczorne">
        <button class="btn-add" onclick="addProgram()">Zapisz w ramówce</button>
        <button class="btn-close" onclick="toggleAdmin(false)">Anuluj</button>
    </div>

    <div id="epg">
        <div style="font-size: 12px; color: #888; margin-bottom: 10px; text-transform: uppercase;">Teraz w TV:</div>
        <div id="list-container"></div>
    </div>

    <script>
        let currentCH = 121;
        let tvData = {};
        let savedPositions = {};

        for (let i = 100; i <= 130; i++) tvData[i] = [];

        function bootTV() {
            document.getElementById('start-overlay').style.display = 'none';
            document.getElementById('tv-player').muted = false;
            heartbeat();
        }

        function toggleAdmin(show) {
            document.getElementById('admin-panel').style.display = show ? 'block' : 'none';
        }

        function addProgram() {
            const f = document.getElementById('inp-file').files[0];
            const t = document.getElementById('inp-time').value;
            const tit = document.getElementById('inp-title').value;

            if (f && t && tit) {
                const url = URL.createObjectURL(f);
                tvData[currentCH].push({ time: t, title: tit, url: url });
                tvData[currentCH].sort((a, b) => a.time.localeCompare(b.time));
                renderEPG();
                toggleAdmin(false);
            }
        }

        function heartbeat() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + now.getMinutes().toString().padStart(2, '0');
            document.getElementById('clock').innerText = now.toLocaleTimeString();

            let activeProg = null;
            tvData[currentCH].forEach(p => { if (timeStr >= p.time) activeProg = p; });

            const player = document.getElementById('tv-player');
            if (activeProg) {
                if (player.getAttribute('data-id') !== activeProg.time) {
                    player.src = activeProg.url;
                    player.setAttribute('data-id', activeProg.time);
                    if (savedPositions[currentCH]) player.currentTime = savedPositions[currentCH];
                    player.play();
                    renderEPG();
                }
            } else {
                player.src = "";
                player.removeAttribute('data-id');
            }

            if (!player.paused) savedPositions[currentCH] = player.currentTime;
            setTimeout(heartbeat, 1000);
        }

        function renderEPG() {
            const container = document.getElementById('list-container');
            container.innerHTML = "";
            const nowTime = new Date().getHours().toString().padStart(2, '0') + ":" + new Date().getMinutes().toString().padStart(2, '0');

            if (tvData[currentCH].length === 0) {
                container.innerHTML = "<div style='font-size:12px; color:#555;'>Brak zaplanowanej emisji na tym kanale.</div>";
                return;
            }

            tvData[currentCH].forEach(p => {
                const isLive = (p === [...tvData[currentCH]].reverse().find(x => nowTime >= x.time));
                container.innerHTML += `<div class="prog-item ${isLive ? 'is-active' : ''}">
                    <span>${p.time}</span> <span>${p.title}</span>
                </div>`;
            });
        }

        function changeCH(n) {
            if (n < 100) n = 130;
            if (n > 130) n = 100;

            savedPositions[currentCH] = document.getElementById('tv-player').currentTime;
            currentCH = n;
            document.getElementById('ch-display').innerText = "CH " + currentCH;
            document.getElementById('tv-player').setAttribute('data-id', 'switching');
            renderEPG();
        }

        window.addEventListener('keydown', (e) => {
            if (e.key === "ArrowUp") changeCH(currentCH + 1);
            if (e.key === "ArrowDown") changeCH(currentCH - 1);
        });
    </script>
</body>

</html>
