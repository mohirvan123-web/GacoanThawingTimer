<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Sync Reminder</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline'; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com https://www.googleapis.com; font-src 'self' data:;">
    <script type="text/javascript" src="cordova.js"></script>
    
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>
    
    <style>
        /* ==================== 
           CSS: STYLING DAN RESPONSIVITAS 
           ==================== */
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
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-light-bg);
            min-height: 100vh;
            margin: 0; 
            padding: 20px 10px;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            transition: background-color 0.2s; 
            color: var(--color-text-dark);
        }

        .main-container {
            background-color: var(--color-white);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            text-align: center;
            width: 100%;
            max-width: 850px;
            box-sizing: border-box; 
        }

        /* --- STYLING KHUSUS UNTUK LOGIN --- */
        #auth-container {
            max-width: 350px;
            margin: 0 auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            background: #fff;
            text-align: left;
        }
        #auth-container h2 {
            text-align: center;
            color: var(--color-primary-blue);
            margin-bottom: 20px;
            font-weight: 700;
        }
        .auth-group {
            margin-bottom: 15px;
        }
        .auth-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
            font-size: 0.9em;
        }
        .auth-group input[type="email"],
        .auth-group input[type="password"] {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            box-sizing: border-box;
        }
        #login-btn, #logout-btn {
            width: 100%;
            padding: 8px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 200;
            background-color: var(--color-primary-blue);
            color: white;
            transition: background-color 0.2s;
        }
        #error-message {
            color: var(--color-alert);
            margin-top: 10px;
            text-align: center;
            font-weight: 600;
        }

        /* --- STYLING HEADER SAAT LOGIN --- */
        .header-controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }
        .header-controls h1 {
            margin: 0;
        }
        .header-controls p {
            margin: 0;
            font-size: 0.8em;
            color: #888;
        }
        .logout-btn-small {
            padding: 8px 15px;
            font-size: 0.8em;
            font-weight: 600;
            background-color: #f44336;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
        }

        /* --- STYLING TIMER (SAMA SEPERTI SEBELUMNYA) --- */

        h1 {
            color: var(--color-primary-blue);
            font-weight: 700;
            margin-bottom: 5px;
        }

        p {
            color: #666;
            margin-bottom: 25px;
            font-size: 0.9em;
        }

        .timer-list {
            display: grid;
            grid-template-columns: repeat(2, 1fr); 
            gap: 20px;
            margin-top: 20px;
        }

        .timer-card {
            border: 1px solid #e0e0e0;
            padding: 20px;
            border-radius: 12px;
            background-color: var(--color-white);
            transition: all 0.3s;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
        }

        .timer-card.running-mode {
            border-left: 5px solid var(--color-primary-blue); 
            background-color: #f0f7ff; 
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1); 
        }

        .timer-card.running-mode h2 {
            color: var(--color-primary-blue); 
        }
        
        .timer-card.running-mode .countdown-display {
            color: var(--color-accent-pink); 
            animation: none !important; 
        }

        .timer-card h2 {
            color: var(--color-accent-pink);
            padding-bottom: 5px;
            font-size: 1.3em;
            font-weight: 600;
        }

        .countdown-display {
            font-size: 2.5em;
            margin: 15px 0 5px 0; 
            font-weight: 700;
            color: var(--color-primary-blue);
        }
        
        .end-time-display {
            font-size: 0.85em;
            color: #999;
            margin-bottom: 10px;
        }

        .alarm-message {
            color: var(--color-alert);
            font-weight: 600;
            margin-top: 10px;
            font-size: 0.95em;
        }

        .timer-controls {
            display: flex;
            gap: 10px;
            margin-top: 15px;
            align-items: center;
            justify-content: center;
        }

        .timer-controls label {
            font-weight: 400;
            white-space: nowrap;
        }
        
        .timer-controls input {
            padding: 10px 5px;
            max-width: 60px; 
            text-align: center; 
            border: 1px solid #ddd;
            border-radius: 8px;
            font-family: 'Poppins', sans-serif;
            font-size: 1em;
            transition: border-color 0.2s;
        }
        
        .timer-controls input:focus {
            border-color: var(--color-primary-blue);
            outline: none;
        }

        .timer-controls button {
            padding: 10px 15px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            flex-grow: 1; 
            transition: background-color 0.2s, transform 0.1s;
        }
        .timer-controls button:active { transform: scale(0.98); }

        .start-btn.syncing {
            background-color: var(--color-syncing) !important;
            color: white;
            animation: pulse-sync 1s infinite alternate;
        }
        
        .start-btn { 
            background-color: var(--color-primary-blue); 
            color: white; 
        }
        .start-btn:hover:not(:disabled):not(.syncing) { background-color: #387ad1; }
        
        .reset-btn { 
            background-color: #6c757d; 
            color: white; 
        }
        .reset-btn:hover:not(:disabled) { background-color: #5a6268; }

        .stop-alarm-btn {
            background-color: var(--color-alert) !important; 
            color: white;
            font-size: 1.1em;
            padding: 10px 15px;
            animation: pulse-stop 0.5s infinite alternate; 
        }

        .timer-controls button:disabled {
            background-color: #999;
            cursor: not-allowed;
            color: #ccc;
        }

        .timer-card.warning {
            border-color: var(--color-warning);
            box-shadow: 0 0 10px rgba(255, 192, 203, 0.5);
            background-color: #fffafa;
        }
        .timer-card.warning .countdown-display {
            color: var(--color-primary-blue); 
            animation: pulse-warn 1s infinite alternate; 
        }

        .timer-card.alert {
            border-color: var(--color-alert);
            background-color: #ffeff3;
            box-shadow: 0 0 10px rgba(255, 105, 180, 0.7);
        }
        .timer-card.alert .countdown-display {
            color: var(--color-alert); 
            font-size: 2.2em;
            animation: none;
        }
        
        .flash-alarm-red {
            background-color: #ff4d6d !important;
            transition: background-color 0.2s; 
        }

        /* ANIMASI */
        @keyframes pulse-warn {
            from { transform: scale(1); opacity: 1; }
            to { transform: scale(1.01); opacity: 0.9; }
        }
        @keyframes pulse-sync {
            from { background-color: var(--color-syncing); }
            to { background-color: #ff7f00; }
        }
        @keyframes pulse-stop {
            from { opacity: 1; box-shadow: 0 0 10px rgba(220, 53, 69, 0.7); }
            to { opacity: 0.8; box-shadow: none; }
        }

        /* MEDIA QUERY */
        @media (max-width: 650px) {
            body { padding: 10px 0; }
            .main-container { padding: 15px; border-radius: 0; box-shadow: none; max-width: 100%;}
            .timer-list { grid-template-columns: 1fr; gap: 15px; } 
            .timer-card { padding: 15px; }
            .countdown-display { font-size: 2em; margin: 10px 0 5px 0; }
            .timer-controls { flex-wrap: wrap; gap: 8px; justify-content: space-between; }
            .timer-controls label { flex-basis: 100%; text-align: left; font-size: 0.9em; }
            .timer-controls input { max-width: 80px; flex-grow: 0; }
            .timer-controls button { padding: 10px; font-size: 0.9em; flex-basis: calc(50% - 5px); }
            .stop-alarm-btn { flex-basis: 100%; } 
        }
    </style>
</head>
<body>
    <div class="main-container">
        <div id="auth-container">
            <h2>Login ke Thawing Timer</h2>
            <div class="auth-group">
                <label for="email">Email</label>
                <input type="email" id="email" placeholder="Masukkan email Anda" required>
            </div>
            <div class="auth-group">
                <label for="password">Password</label>
                <input type="password" id="password" placeholder="Masukkan password Anda" required>
            </div>
            <div class="auth-group">
    <label for="branch-select">Pilih Cabang</label>
    <select id="branch-select" style="width: 100%; padding: 10px; border-radius: 6px; border: 1px solid #ccc;">
        <option value="Cabang_1">Cabang 1 (MLGJAK)</option>
        <option value="Cabang_2">Cabang 2 (MLGMON)</option>
    </select>
</div>
            <button id="login-btn">LOGIN</button>
            <p id="error-message" style="display: none;"></p>
        </div>

        <div id="timer-content" style="display: none;">
            <div class="header-controls">
                <div>
                    <h1>Gacoan Timer Thawing 🧊</h1>
                    <p>Malang Jakarta V.1.3 (Alarm Debounced)</p>
                </div>
                <button id="logout-btn" class="logout-btn-small">LOGOUT</button>
            </div>
            
            <div class="timer-list" id="timer-list">
            </div>
        </div>
    </div>

    <script>
// ===================================
// FIREBASE CONFIGURATION
// ===================================
    const firebaseConfig = {
    apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
    authDomain: "thawing-ec583.firebaseapp.com",
    databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
    projectId: "thawing-ec583",
    storageBucket: "thawing-ec583.firebasestorage.app",
    messagingSenderId: "1043079332713",
    appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8",
    measurementId: "G-WLBFJ6V7QT"
};

if (!firebase.apps.length) firebase.initializeApp(firebaseConfig);

const auth = firebase.auth();
const authContainer = document.getElementById('auth-container');
const timerContent = document.getElementById('timer-content');
const branchSelect = document.getElementById('branch-select');

let currentDbRef = null;
let activeIntervals = {};

// --- LOGIN LOGIC ---
document.getElementById('login-btn').addEventListener('click', () => {
    const email = document.getElementById('email').value;
    const password = document.getElementById('password').value;
    auth.signInWithEmailAndPassword(email, password).catch(err => alert("Login Gagal: " + err.message));
});

document.getElementById('logout-btn').addEventListener('click', () => auth.signOut());

auth.onAuthStateChanged((user) => {
    if (user) {
        const branch = branchSelect.value;
        currentDbRef = firebase.database().ref(`thawingTimers/${branch}`);
        authContainer.style.display = 'none';
        timerContent.style.display = 'block';
        document.querySelector('h1').innerText = `Gacoan Timer - ${branch.replace('_', ' ')}`;
        initializeTimerApp();
    } else {
        authContainer.style.display = 'block';
        timerContent.style.display = 'none';
        cleanupTimerApp();
    }
});

// --- TIMER LOGIC ---
function initializeTimerApp() {
    const THAWING_ITEMS = [
        { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
        { id: 2, name: "ACIN", defaultTimeMinutes: 60 },
        { id: 3, name: "MIE", defaultTimeMinutes: 120 },
        { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
        { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
        { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
        { id: 7, name: "KULIT PANGSIT", defaultTimeMinutes: 120 },
    ];

    const WARNING_TIME_SECONDS = 15 * 60;
    let speechQueue = [];
    let isSpeaking = false;

    function formatTime(s) {
        const h = String(Math.floor(s / 3600)).padStart(2, '0');
        const m = String(Math.floor((s % 3600) / 60)).padStart(2, '0');
        const sec = String(s % 60).padStart(2, '0');
        return `${h}:${m}:${sec}`;
    }

    function speak(msg) {
        if (!speechQueue.includes(msg)) speechQueue.push(msg);
        const process = () => {
            if (isSpeaking || speechQueue.length === 0) return;
            isSpeaking = true;
            const ut = new SpeechSynthesisUtterance(speechQueue.shift());
            ut.lang = 'id-ID';
            ut.onend = () => { isSpeaking = false; process(); };
            window.speechSynthesis.speak(ut);
        };
        process();
    }

    function tick(itemId, endTimeMs, inputMins) {
        const card = document.getElementById(`card-${itemId}`);
        if (!card) return;

        clearTimeout(activeIntervals[itemId]);
        const duration = Math.floor((endTimeMs - Date.now()) / 1000);

        card.classList.add('running-mode');
        document.getElementById(`time-input-${itemId}`).readOnly = true;
        document.getElementById(`start-btn-${itemId}`).style.display = 'none';
        document.getElementById(`reset-btn-${itemId}`).style.display = 'block';
        
        // Update End Time UI
        const endTimeStr = new Date(endTimeMs).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });
        document.getElementById(`end-time-${itemId}`).textContent = `Selesai: ${endTimeStr}`;

        if (duration > 0) {
            document.getElementById(`display-${itemId}`).textContent = formatTime(duration);
            if (duration <= WARNING_TIME_SECONDS) {
                card.classList.add('warning');
                document.getElementById(`msg-${itemId}`).textContent = `🔔 Sisa ${Math.ceil(duration/60)} mnt!`;
                document.getElementById(`msg-${itemId}`).style.display = 'block';
            }
            activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs, inputMins), 1000);
        } else {
            document.getElementById(`display-${itemId}`).textContent = "WAKTU HABIS!";
            card.classList.replace('warning', 'alert');
            const rb = document.getElementById(`reset-btn-${itemId}`);
            rb.classList.add('stop-alarm-btn');
            rb.textContent = "STOP ALARM & AMBIL";
            document.body.classList.add('flash-alarm-red');
            speak(`Waktu thawing ${THAWING_ITEMS.find(i=>i.id===itemId).name} habis!`);
        }
    }

    window.startTimer = (id) => {
        const val = parseInt(document.getElementById(`time-input-${id}`).value);
        currentDbRef.child(id).set({ endTime: Date.now() + (val * 60 * 1000), inputMinutes: val });
    };

    window.resetTimer = (id) => {
        document.body.classList.remove('flash-alarm-red');
        window.speechSynthesis.cancel();
        currentDbRef.child(id).remove();
    };

    // Render & Listen
    const list = document.getElementById('timer-list');
    list.innerHTML = '';
    THAWING_ITEMS.forEach(item => {
        const div = document.createElement('div');
        div.className = 'timer-card';
        div.id = `card-${item.id}`;
        div.innerHTML = `
            <h2>${item.name}</h2>
            <div id="display-${item.id}" class="countdown-display">${formatTime(item.defaultTimeMinutes * 60)}</div>
            <div id="end-time-${item.id}" class="end-time-display">Ready</div>
            <div id="msg-${item.id}" class="alarm-message" style="display: none;"></div>
            <div class="timer-controls">
                <input type="number" id="time-input-${item.id}" value="${item.defaultTimeMinutes}">
                <button id="start-btn-${item.id}" class="start-btn" onclick="startTimer(${item.id})">START</button>
                <button id="reset-btn-${item.id}" class="reset-btn" style="display: none;" onclick="resetTimer(${item.id})">RESET</button>
            </div>`;
        list.appendChild(div);
    });

    if(currentDbRef) {
        currentDbRef.off(); // Bersihkan listener lama
        currentDbRef.on('value', (snap) => {
            const data = snap.val() || {};
            THAWING_ITEMS.forEach(item => {
                if (data[item.id]) {
                    tick(item.id, data[item.id].endTime, data[item.id].inputMinutes);
                } else {
                    clearTimeout(activeIntervals[item.id]);
                    const c = document.getElementById(`card-${item.id}`);
                    if(c) {
                        c.classList.remove('running-mode', 'warning', 'alert');
                        document.getElementById(`display-${item.id}`).textContent = formatTime(item.defaultTimeMinutes * 60);
                        document.getElementById(`time-input-${item.id}`).readOnly = false;
                        document.getElementById(`start-btn-${item.id}`).style.display = 'block';
                        document.getElementById(`reset-btn-${item.id}`).style.display = 'none';
                        document.getElementById(`msg-${item.id}`).style.display = 'none';
                        document.getElementById(`end-time-${item.id}`).textContent = 'Ready';
                    }
                }
            });
        });
    }
}

function cleanupTimerApp() {
    if (currentDbRef) currentDbRef.off();
    Object.values(activeIntervals).forEach(clearTimeout);
    activeIntervals = {};
    window.speechSynthesis.cancel();
    document.body.classList.remove('flash-alarm-red');
}
    </script>
</body>
</html>
