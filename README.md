<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gacoan Thawing Sync V.1.5</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com https://script.google.com https://script.googleusercontent.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com; font-src 'self' data: https://fonts.gstatic.com;">
    
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    
    <style>
        :root {
            --color-white: #ffffff;
            --color-light-bg: #f7f9ff;
            --color-primary-blue: #4A90E2;
            --color-accent-pink: #FF69B4;
            --color-text-dark: #333333;
            --color-warning: #FFC0CB;
            --color-alert: #DC3545;
        }
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-light-bg);
            margin: 0; padding: 20px 10px;
            display: flex; justify-content: center;
        }

        .main-container {
            background-color: var(--color-white);
            padding: 25px; border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            text-align: center; width: 100%; max-width: 850px;
        }

        h1 { color: var(--color-primary-blue); margin-bottom: 5px; }
        .sub-header { color: #666; font-size: 0.9em; margin-bottom: 25px; }

        /* GRID TIMER */
        .timer-list {
            display: grid; grid-template-columns: repeat(2, 1fr); 
            gap: 15px; margin-bottom: 30px;
        }

        .timer-card {
            border: 1px solid #e0e0e0; padding: 15px;
            border-radius: 12px; transition: all 0.3s;
        }

        .timer-card.running-mode { border-left: 5px solid var(--color-primary-blue); background-color: #f0f7ff; }
        .timer-card.warning { background-color: #fffafa; border-color: var(--color-warning); }
        .timer-card.alert { background-color: #ffeff3; border-color: var(--color-alert); }

        .countdown-display { font-size: 2.2em; font-weight: 700; color: var(--color-primary-blue); margin: 10px 0; }
        .end-time-display { font-size: 0.8em; color: #999; }
        
        .timer-controls { display: flex; gap: 8px; margin-top: 10px; justify-content: center; flex-wrap: wrap; }
        .timer-controls input { width: 50px; padding: 8px; border-radius: 5px; border: 1px solid #ddd; text-align: center; }
        
        button { border: none; padding: 10px 15px; border-radius: 8px; font-weight: 600; cursor: pointer; transition: 0.2s; }
        .start-btn { background: var(--color-primary-blue); color: white; width: 100%; }
        .reset-btn { background: #6c757d; color: white; width: 100%; }
        .stop-alarm-btn { background: var(--color-alert) !important; animation: pulse 0.5s infinite alternate; }

        /* HISTORY SECTION */
        .history-section { text-align: left; margin-top: 40px; border-top: 2px dashed #eee; padding-top: 20px; }
        .history-card { 
            background: #fff; border: 1px solid #eee; padding: 12px; 
            border-radius: 8px; margin-bottom: 8px; display: flex; 
            justify-content: space-between; align-items: center; font-size: 0.85em;
        }

        @keyframes pulse { from { opacity: 1; } to { opacity: 0.7; } }
        .flash-red { background-color: #ff4d6d !important; }

        @media (max-width: 600px) { .timer-list { grid-template-columns: 1fr; } }
    </style>
</head>
<body>

<div class="main-container">
    <h1>Gacoan Timer Thawing 🧊</h1>
    <p class="sub-header">V.1.5 - Sinkron Real-time & Riwayat</p>
    
    <div class="timer-list" id="timer-list"></div>

    <div class="history-section">
        <h3 style="color: var(--color-primary-blue); margin-bottom: 15px;">📜 10 Aktivitas Terakhir</h3>
        <div id="history-list">
            <p style="text-align:center; color:#999;">Memuat riwayat...</p>
        </div>
    </div>
</div>

<script>
    // 1. CONFIGURATION
    const firebaseConfig = {
        apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
        authDomain: "thawing-ec583.firebaseapp.com",
        databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
        projectId: "thawing-ec583",
        storageBucket: "thawing-ec583.firebasestorage.app",
        messagingSenderId: "1043079332713",
        appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8"
    };

    firebase.initializeApp(firebaseConfig);
    const dbRef = firebase.database().ref('thawingTimers');
    const historyRef = firebase.database().ref('thawingHistory');
    const SPREADSHEET_URL = "https://script.google.com/macros/s/AKfycbybjUmJsd4TzeXCpNkquddhjK0b0uOdA6ghjViR2boE6vjuWnutcmh3aQE7QBijbU5W/exec";

    const THAWING_ITEMS = [
        { id: 1, name: "ADONAN", defaultTime: 40 },
        { id: 2, name: "ACIN", defaultTime: 120 },
        { id: 3, name: "MIE", defaultTime: 120 },
        { id: 4, name: "PENTOL", defaultTime: 120 },
        { id: 5, name: "SURAI NAGA", defaultTime: 120 },
        { id: 6, name: "KRUPUK MIE", defaultTime: 120 },
        { id: 7, name: "KULIT PANGSIT", defaultTime: 120 }
    ];

    let activeIntervals = {};
    let flashInterval = null;

    // 2. FUNCTIONS
    function formatTime(sec) {
        const h = Math.floor(sec / 3600).toString().padStart(2, '0');
        const m = Math.floor((sec % 3600) / 60).toString().padStart(2, '0');
        const s = (sec % 60).toString().padStart(2, '0');
        return `${h}:${m}:${s}`;
    }

    function sendToSpreadsheet(itemName, duration, endTimeMs) {
        const endStr = new Date(endTimeMs).toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit'});
        fetch(SPREADSHEET_URL, {
            method: "POST",
            mode: "no-cors",
            body: JSON.stringify({ itemName, duration, endTimeString: endStr })
        }).catch(e => console.log("Spreadsheet error"));
    }

    function saveHistory(itemName, duration, endTimeMs) {
        const startStr = new Date().toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit'});
        const endStr = new Date(endTimeMs).toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit'});
        historyRef.push({
            itemName, duration, timeStart: startStr, timeEnd: endStr,
            timestamp: firebase.database.ServerValue.TIMESTAMP
        });
    }

    function speak(msg) {
        window.speechSynthesis.cancel();
        const u = new SpeechSynthesisUtterance(msg);
        u.lang = 'id-ID';
        window.speechSynthesis.speak(u);
    }

    // 3. CORE TIMER LOGIC
    function tick(itemId, endTimeMs) {
        const card = document.getElementById(`card-${itemId}`);
        const display = document.getElementById(`display-${itemId}`);
        const msgArea = document.getElementById(`msg-${itemId}`);
        const item = THAWING_ITEMS.find(i => i.id === itemId);

        clearTimeout(activeIntervals[itemId]);
        const now = Date.now();
        const diff = Math.floor((endTimeMs - now) / 1000);

        card.classList.add('running-mode');
        document.getElementById(`start-btn-${itemId}`).style.display = 'none';
        const rb = document.getElementById(`reset-btn-${itemId}`);
        rb.style.display = 'block';

        if (diff > 0) {
            display.textContent = formatTime(diff);
            if (diff <= 900) { // 15 menit
                card.classList.add('warning');
                msgArea.textContent = `⚠️ Kurang ${Math.ceil(diff/60)} menit!`;
                if (diff % 300 === 0) speak(`Waktu ${item.name} hampir habis.`);
            }
            activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs), 1000);
        } else {
            display.textContent = "HABIS!";
            card.classList.replace('warning', 'alert');
            rb.textContent = "STOP ALARM";
            rb.classList.add('stop-alarm-btn');
            if (!flashInterval) flashInterval = setInterval(() => document.body.classList.toggle('flash-red'), 400);
            speak(`Peringatan! Thawing ${item.name} selesai. Segera ambil bahan.`);
        }
    }

    // 4. USER ACTIONS
    window.startTimer = (itemId) => {
        const dur = parseInt(document.getElementById(`input-${itemId}`).value);
        if (isNaN(dur) || dur <= 0) return;
        
        const endTimeMs = Date.now() + (dur * 60 * 1000);
        const item = THAWING_ITEMS.find(i => i.id === itemId);

        dbRef.child(itemId).set({ endTime: endTimeMs, inputMinutes: dur });
        saveHistory(item.name, dur, endTimeMs);
        sendToSpreadsheet(item.name, dur, endTimeMs);
    };

    window.resetTimer = (itemId) => {
        dbRef.child(itemId).remove();
        clearInterval(flashInterval);
        flashInterval = null;
        document.body.classList.remove('flash-red');
        window.speechSynthesis.cancel();
    };

    // 5. INITIALIZATION & SYNC
    document.addEventListener('DOMContentLoaded', () => {
        const list = document.getElementById('timer-list');
        THAWING_ITEMS.forEach(item => {
            const div = document.createElement('div');
            div.className = 'timer-card';
            div.id = `card-${item.id}`;
            div.innerHTML = `
                <h2 style="margin:0; font-size:1.1em;">${item.name}</h2>
                <div class="countdown-display" id="display-${item.id}">${formatTime(item.defaultTime*60)}</div>
                <div class="end-time-display" id="end-${item.id}">Durasi default</div>
                <div id="msg-${item.id}" style="color:red; font-size:0.8em; font-weight:bold;"></div>
                <div class="timer-controls">
                    <input type="number" id="input-${item.id}" value="${item.defaultTime}">
                    <button class="start-btn" id="start-btn-${item.id}" onclick="startTimer(${item.id})">START</button>
                    <button class="reset-btn" id="reset-btn-${item.id}" style="display:none" onclick="resetTimer(${item.id})">RESET</button>
                </div>
            `;
            list.appendChild(div);
        });

        // Sync Timers
        dbRef.on('value', snap => {
            const data = snap.val() || {};
            THAWING_ITEMS.forEach(item => {
                if (data[item.id]) {
                    tick(item.id, data[item.id].endTime);
                } else {
                    clearTimeout(activeIntervals[item.id]);
                    const card = document.getElementById(`card-${item.id}`);
                    card.className = 'timer-card';
                    document.getElementById(`display-${item.id}`).textContent = formatTime(item.defaultTime*60);
                    document.getElementById(`start-btn-${item.id}`).style.display = 'block';
                    const rb = document.getElementById(`reset-btn-${item.id}`);
                    rb.style.display = 'none'; rb.className = 'reset-btn'; rb.textContent = 'RESET';
                    document.getElementById(`msg-${item.id}`).textContent = '';
                }
            });
        });

        // Sync History (Tampilkan 10 terakhir)
        historyRef.limitToLast(10).on('value', snap => {
            const histDiv = document.getElementById('history-list');
            histDiv.innerHTML = '';
            const items = [];
            snap.forEach(c => items.unshift(c.val()));
            
            if (items.length === 0) histDiv.innerHTML = '<p style="text-align:center; color:#999;">Belum ada data.</p>';
            
            items.forEach(h => {
                const row = document.createElement('div');
                row.className = 'history-card';
                row.innerHTML = `
                    <div><strong>${h.itemName}</strong><br><small>${h.duration} mnt</small></div>
                    <div style="text-align:right">
                        <span style="color:var(--color-primary-blue)">Start: ${h.timeStart}</span><br>
                        <span style="color:var(--color-accent-pink)">End: ${h.timeEnd}</span>
                    </div>
                `;
                histDiv.appendChild(row);
            });
        });
    });
</script>

</body>
</html>
