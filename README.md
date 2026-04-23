<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>KANAŁ 121 - PRO SYSTEM</title>
    <style>
        body, html { margin: 0; padding: 0; width: 100%; height: 100%; background: #000; color: #fff; font-family: 'Segoe UI', sans-serif; overflow: hidden; }
        
        /* WIDEO TŁO */
        #tv-player { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; object-fit: cover; z-index: 1; }

        /* ZEGAREK I KANAŁ - GÓRA */
        #status-bar {
            position: absolute; top: 0; left: 0; width: 100%;
            background: linear-gradient(to bottom, rgba(0,0,0,0.8), transparent);
            padding: 20px; display: flex; justify-content: center; align-items: center; z-index: 10;
        }
        #ch-info { font-size: 40px; font-weight: bold; color: #00ff00; text-shadow: 2px 2px 10px #000; margin: 0 50px; }
        #live-clock { font-size: 20px; background: rgba(0,255,0,0.2); padding: 5px 15px; border-radius: 5px; border: 1px solid #00ff00; }

        /* PRZYCISKI ZMIANY KANAŁÓW - PO BOKACH */
        .side-btn {
            position: absolute; top: 0; width: 80px; height: 100%;
            background: rgba(0,0,0,0.1); border: none; color: white;
            font-size: 40px; z-index: 50; cursor: pointer; transition: 0.3s;
        }
        .side-btn:hover { background: rgba(0,255,0,0.2); }
        #btn-prev { left: 0; }
        #btn-next { right: 0; }

        /* PANEL RAMÓWKI - LEWY DÓŁ */
        #epg-container {
            position: absolute; bottom: 30px; left: 30px; width: 300px;
            background: rgba(0, 0, 0, 0.85); border-radius: 10px; border-left: 5px solid #00ff00;
            padding: 15px; z-index: 60; box-shadow: 0 0 20px rgba(0,0,0,0.5);
        }
        .prog-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #333; font-size: 14px; }
        .is-live { color: #00ff00; font-weight: bold; background: rgba(0,255,0,0.1); border-radius: 4px; }

        /* PANEL DODAWANIA - PRAWY DÓŁ (UKRYTY DO NAJECHANIA) */
        #upload-panel {
            position: absolute; bottom: 30px; right: 30px; width: 250px;
            background: rgba(20,20,20,0.9); padding: 15px; z-index: 100;
            border-radius: 10px; opacity: 0.3; transition: 0.5s;
        }
        #upload-panel:hover { opacity: 1; }
        input, button { width: 100%; margin: 5px 0; padding: 10px; border-radius: 5px; border: 1px solid #444; background: #111; color: white; box-sizing: border-box; }
        button { background: #00ff00; color: #000; font-weight: bold; border: none; cursor: pointer; }

        /* START SCREEN */
        #overlay { position: fixed; top:0; left:0; width:100%; height:100%; background: #000; z-index: 1000; display: flex; align-items: center; justify-content: center; cursor: pointer; }
    </style>
</head>
<body>

    <div id="overlay" onclick="startTV()"><h1>WŁĄCZ KANAŁ 121</h1></div>

    <div id="status-bar">
        <div id="live-clock">00:00:00</div>
        <div id="ch-info">CH 121</div>
    </div>

    <button id="btn-prev" class="side-btn" onclick="changeChannel(currentCH - 1)">‹</button>
    <button id="btn-next" class="side-btn" onclick="changeChannel(currentCH + 1)">›</button>

    <video id="tv-player" playsinline loop></video>

    <div id="epg-container">
        <h3 style="margin: 0 0 10px 0; font-size: 16px;">PROGRAM TV</h3>
        <div id="schedule-list">Brak zaplanowanych filmów</div>
    </div>

    <div id="upload-panel">
        <strong style="font-size: 12px; color: #00ff00;">REŻYSERKA (Najedź by użyć)</strong>
        <input type="file" id="f-file" accept="video/*">
        <input type="time" id="f-time">
        <input type="text" id="f-title" placeholder="Tytuł filmu">
        <button onclick="addVideo()">DODAJ FILM</button>
    </div>

    <script>
        let currentCH = 121;
        let channels = {};
        let times = {}; // Pamięć czasu wideo

        for(let i = 100; i <= 130; i++) channels[i] = [];

        function startTV() {
            document.getElementById('overlay').style.display = 'none';
            document.getElementById('tv-player').muted = false;
            updateLoop();
        }

        function addVideo() {
            const file = document.getElementById('f-file').files[0];
            const time = document.getElementById('f-time').value;
            const title = document.getElementById('f-title').value;

            if (file && time && title) {
                const url = URL.createObjectURL(file);
                channels[currentCH].push({ time, title, url });
                channels[currentCH].sort((a, b) => a.time.localeCompare(b.time));
                renderEPG();
                alert("Dodano do ramówki kanału " + currentCH);
            }
        }

        function updateLoop() {
            const now = new Date();
            const nowStr = now.getHours().toString().padStart(2, '0') + ":" + now.getMinutes().toString().padStart(2, '0');
            document.getElementById('live-clock').innerText = now.toLocaleTimeString();

            let active = null;
            channels[currentCH].forEach(p => { if (nowStr >= p.time) active = p; });

            const v = document.getElementById('tv-player');
            if (active) {
                if (v.getAttribute('data-id') !== active.time) {
                    v.src = active.url;
                    v.setAttribute('data-id', active.time);
                    if (times[currentCH]) v.currentTime = times[currentCH];
                    v.play();
                    renderEPG();
                }
            } else {
                v.src = "";
                v.removeAttribute('data-id');
                document.getElementById('schedule-list').innerHTML = "Brak emisji o tej godzinie";
            }

            if (!v.paused) times[currentCH] = v.currentTime;
            setTimeout(updateLoop, 1000);
        }

        function renderEPG() {
            const list = document.getElementById('schedule-list');
            list.innerHTML = "";
            const nowT = new Date().getHours().toString().padStart(2, '0') + ":" + new Date().getMinutes().toString().padStart(2, '0');

            if(channels[currentCH].length === 0) {
                list.innerHTML = "Pusto na tym kanale";
                return;
            }

            channels[currentCH].forEach(p => {
                const isLive = (p === [...channels[currentCH]].reverse().find(x => nowT >= x.time));
                list.innerHTML += `<div class="prog-row ${isLive ? 'is-live' : ''}">
                    <span>${p.time}</span> <span>${p.title}</span>
                </div>`;
            });
        }

        function changeChannel(n) {
            if(n < 100) n = 130;
            if(n > 130) n = 100;
            const v = document.getElementById('tv-player');
            times[currentCH] = v.currentTime;
            currentCH = n;
            document.getElementById('ch-info').innerText = "CH " + currentCH;
            v.setAttribute('data-id', 'reset');
            renderEPG();
        }

        window.addEventListener('keydown', (e) => {
            if (e.key === "ArrowUp") changeChannel(currentCH + 1);
            if (e.key === "ArrowDown") changeChannel(currentCH - 1);
        });
    </script>
</body>
</html>
