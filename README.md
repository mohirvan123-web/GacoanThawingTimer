<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Reminder Pro - Gacoan</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>

    <style>
        /* CSS Root & Global */
        :root {
            --color-white: #ffffff;
            --color-light-bg: #f7f9ff;
            --color-primary-blue: #4A90E2;
            --color-accent-pink: #FF69B4;
            --color-text-dark: #333333;
            --color-warning: #FFC0CB;
            --color-alert: #DC3545;
            --color-syncing: #FFA500;
        }

        body { font-family: 'Poppins', sans-serif; background-color: var(--color-light-bg); margin: 0; padding: 10px; color: var(--color-text-dark); }
        
        /* Login Overlay */
        #login-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: var(--color-light-bg); z-index: 10000;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }
        .login-box {
            background: white; padding: 30px; border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0,0,0,0.1); width: 90%; max-width: 350px; text-align: center;
        }
        .login-box input { width: 100%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        .login-box button { width: 100%; padding: 12px; background: var(--color-primary-blue); color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }

        /* App Container */
        .main-container { max-width: 850px; margin: 0 auto; background: white; padding: 20px; border-radius: 16px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
        .header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #f0f0f0; margin-bottom: 20px; padding-bottom: 10px; }
        .header h1 { font-size: 1.2em; color: var(--color-primary-blue); margin: 0; }
        .logout-btn { background: #ff4d4d; color: white; border: none; padding: 5px 12px; border-radius: 6px; cursor: pointer; font-size: 0.8em; }

        /* Timer Grid */
        .timer-list { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 15px; }
        .timer-card { border: 1px solid #e0e0e0; padding: 15px; border-radius: 12px; transition: 0.3s; text-align: center; }
        .timer-card.running-mode { background-color: #f0f7ff; border-left: 5px solid var(--color-primary-blue); }
        .timer-card.warning { background-color: #fffafa; border-color: var(--color-warning); animation: pulse 1.5s infinite; }
        .timer-card.alert { background-color: #ffeff3; border-color: var(--color-alert); }
        
        .countdown-display { font-size: 2.2em; font-weight: 700; color: var(--color-primary-blue); margin: 10px 0; }
        .running-mode .countdown-display { color: var(--color-accent-pink); }
        
        .timer-controls { display: flex; gap: 8px; align-items: center; justify-content: center; margin-top: 10px; }
        .timer-controls input { width: 60px; padding: 8px; text-align: center; border: 1px solid #ddd; border-radius: 6px; }
        .btn { padding: 8px 15px; border: none; border-radius: 6px; font-weight: 600; cursor: pointer; flex-grow: 1; }
        .start-btn { background: var(--color-primary-blue); color: white; }
        .reset-btn { background: #6c757d; color: white; }
        .stop-alarm-btn { background: var(--color-alert) !important; color: white; animation: flash 0.5s infinite; }

        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.8; } 100% { opacity: 1; } }
        @keyframes flash { from { background: #ff4d4d; } to { background: #b30000; } }
    </style>
</head>
<body>

    <div id="login-overlay">
        <div class="login-box">
            <h2 style="color: var(--color-primary-blue);">Gacoan Login</h2>
            <input type="email" id="email" placeholder="Email Cabang">
            <input type="password" id="password" placeholder="Password">
            <button onclick="handleLogin()">MASUK</button>
        </div>
    </div>

    <div class="main-container" id="main-app" style="display: none;">
        <div class="header">
            <div>
                <h1 id="branch-title">Gacoan Thawing</h1>
                <span id="user-info" style="font-size: 0.7em; color: #888;"></span>
            </div>
            <button class="logout-btn" onclick="handleLogout()">Logout</button>
        </div>
        
        <div class="timer-list" id="timer-list"></div>
    </div>

    <script>
        // ===================================
        // 1. CONFIGURATION
        // ===================================
        const firebaseConfig = {
            apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
            authDomain: "thawing-ec583.firebaseapp.com",
            databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "thawing-ec583",
            storageBucket: "thawing-ec583.firebasestorage.app", 
            messagingSenderId: "1043079332713",
            appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8"
        };

        const SCRIPT_URL = "ISI_URL_APPS_SCRIPT_GOOGLE_SHEETS_ANDA"; // Masukkan URL Spreadsheet anda
        
        // Ambil ID Cabang dari URL (misal: ?cabang=Malang) atau default
        const urlParams = new URLSearchParams(window.location.search);
        const ID_CABANG = urlParams.get('cabang') || 'pusat_jakarta';
        document.getElementById('branch-title').textContent = "Gacoan Thawing - " + ID_CABANG.toUpperCase();

        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const dbRef = firebase.database().ref('thawingTimers/' + ID_CABANG);
        const auth = firebase.auth();

        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
            { id: 2, name: "ACIN", defaultTimeMinutes: 120 },
            { id: 3, name: "MIE", defaultTimeMinutes: 120 },
            { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
            { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
            { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
            { id: 7, name: "KULIT PANGSIT", defaultTimeMinutes: 120 },
        ];

        let activeIntervals = {};
        let speechQueue = [];
        let isSpeaking = false;

        // ===================================
        // 2. AUTHENTICATION
        // ===================================
        function handleLogin() {
            const email = document.getElementById('email').value;
            const pass = document.getElementById('password').value;
            auth.signInWithEmailAndPassword(email, pass).catch(err => alert("Gagal: " + err.message));
        }

        function handleLogout() { auth.signOut(); location.reload(); }

        auth.onAuthStateChanged(user => {
            if (user) {
                document.getElementById('login-overlay').style.display = 'none';
                document.getElementById('main-app').style.display = 'block';
                document.getElementById('user-info').textContent = "Logged in: " + user.email;
                initApp();
            } else {
                document.getElementById('login-overlay').style.display = 'flex';
                document.getElementById('main-app').style.display = 'none';
            }
        });

        // ===================================
        // 3. CORE LOGIC
        // ===================================
        function initApp() {
            const list = document.getElementById('timer-list');
            list.innerHTML = "";
            THAWING_ITEMS.forEach(item => {
                const card = document.createElement('div');
                card.className = 'timer-card';
                card.id = card-${item.id};
                card.innerHTML = `
                    <h2 style="margin:0; font-size:1em;">${item.name}</h2>
                    <div id="display-${item.id}" class="countdown-display">00:00:00</div>
                    <div id="end-time-${item.id}" style="font-size:0.7em; color:#999;">--:--</div>
                    <div class="timer-controls">
                        <input type="number" id="input-${item.id}" value="${item.defaultTimeMinutes}">
                        <button id="btn-start-${item.id}" class="btn start-btn" onclick="startTimer(${item.id})">START</button>
                        <button id="btn-reset-${item.id}" class="btn reset-btn" style="display:none" onclick="resetTimer(${item.id})">RESET</button>
                    </div>
                `;
                list.appendChild(card);
            });

            // Listen to Firebase
            dbRef.on('value', snapshot => {
                const data = snapshot.val() || {};
                THAWING_ITEMS.forEach(item => {
                    const state = data[item.id];
                    if (state) {
                        runTick(item.id, state.endTime, state.inputMinutes);
                    } else {
                        stopTick(item.id, item.defaultTimeMinutes);
                    }
                });
            });
        }

        function startTimer(id) {
            const mins = document.getElementById(input-${id}).value;
            const endTime = Date.now() + (mins * 60 * 1000);
            const item = THAWING_ITEMS.find(i => i.id === id);
            
            dbRef.child(id).set({ endTime, inputMinutes: mins });
            logToSheet(item.name, "MULAI", mins);
        }

        function resetTimer(id) {
            const item = THAWING_ITEMS.find(i => i.id === id);
            dbRef.child(id).remove();
            stopAggressiveAlarm();
            logToSheet(item.name, "DIAMBIL/RESET", "-");
        }

        function runTick(id, endTime, mins) {
            const card = document.getElementById(card-${id});
            const display = document.getElementById(display-${id});
            const endTimeDiv = document.getElementById(end-time-${id});
            const btnStart = document.getElementById(btn-start-${id});
            const btnReset = document.getElementById(btn-reset-${id});
            const input = document.getElementById(input-${id});

            clearInterval(activeIntervals[id]);
            card.classList.add('running-mode');
            btnStart.style.display = 'none';
            btnReset.style.display = 'block';
            input.readOnly = true;
            
            const endObj = new Date(endTime);
            endTimeDiv.textContent = "Selesai: " + endObj.getHours().toString().padStart(2,'0') + ":" + endObj.getMinutes().toString().padStart(2,'0');

            activeIntervals[id] = setInterval(() => {
                const now = Date.now();
                const diff = Math.floor((endTime - now) / 1000);

                if (diff <= 0) {
                    display.textContent = "HABIS!";
                    card.classList.add('alert');
                    btnReset.classList.add('stop-alarm-btn');
                    btnReset.textContent = "STOP & AMBIL";
                    triggerAlarm(THAWING_ITEMS.find(i => i.id === id).name);
                } else {
                    display.textContent = formatHMS(diff);
                    if (diff < 900) card.classList.add('warning'); // 15 menit
                }
            }, 1000);
        }

        function stopTick(id, defMins) {
            clearInterval(activeIntervals[id]);
            const card = document.getElementById(card-${id});
            if(!card) return;
            card.className = 'timer-card';
            document.getElementById(display-${id}).textContent = formatHMS(defMins * 60);
            document.getElementById(btn-start-${id}).style.display = 'block';
            document.getElementById(btn-reset-${id}).style.display = 'none';
            document.getElementById(input-${id}).readOnly = false;
        }

        function formatHMS(sec) {
            const h = Math.floor(sec / 3600).toString().padStart(2,'0');
            const m = Math.floor((sec % 3600) / 60).toString().padStart(2,'0');
            const s = (sec % 60).toString().padStart(2,'0');
            return ${h}:${m}:${s};
        }

        // ===================================
        // 4. SPREADSHEET LOGGING
        // ===================================
        function logToSheet(bahan, aksi, durasi) {
            if(!SCRIPT_URL.startsWith("http")) return;
            fetch(SCRIPT_URL, {
                method: 'POST',
                mode: 'no-cors',
                body: JSON.stringify({ restoran: ID_CABANG, namaBahan: bahan, aksi, durasi })
            });
        }

        // ===================================
        // 5. ALARM & SPEECH
        // ===================================
        function triggerAlarm(name) {
            if (!isSpeaking) {
                const msg = new SpeechSynthesisUtterance(Waktu thawing ${name} sudah habis. Segera ambil bahan.);
                msg.lang = 'id-ID';
                msg.onstart = () => isSpeaking = true;
                msg.onend = () => isSpeaking = false;
                window.speechSynthesis.speak(msg);
            }
            if('vibrate' in navigator) navigator.vibrate([500, 300, 500]);
        }

        function stopAggressiveAlarm() {
            window.speechSynthesis.cancel();
            isSpeaking = false;
        }
    </script>
</body>
</html>
