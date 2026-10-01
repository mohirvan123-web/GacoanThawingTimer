<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gacoan Thawing & Tiris System</title>

<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">

<style>
:root {
  --bg:          #0f1117;
  --surface:     #181c27;
  --surface2:    #1e2335;
  --border:      #2a3050;
  --border2:     #3a4570;
  --blue:        #3b82f6;
  --blue-dim:    #1e3a8a;
  --cyan:        #06b6d4;
  --green:       #22c55e;
  --green-dim:   #14532d;
  --yellow:      #f59e0b;
  --yellow-dim:  #78350f;
  --red:         #ef4444;
  --red-dim:     #7f1d1d;
  --orange:      #f97316;
  --purple:      #a855f7;
  --purple-dim:  #3b0764;
  --text:        #e2e8f0;
  --text-muted:  #64748b;
  --text-dim:    #94a3b8;
  --font-ui:     'Space Grotesk', sans-serif;
  --font-mono:   'JetBrains Mono', monospace;
  --radius:      10px;
  --radius-sm:   6px;
}

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: var(--font-ui);
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  overflow-x: hidden;
}

::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: var(--surface); }
::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 3px; }

/* ── HALAMAN SYSTEM ── */
.page { display: none; min-height: 100vh; }
.page.active { display: flex; }

/* ══════════════════════════════════════
   HALAMAN LOGIN
══════════════════════════════════════ */
#page-login {
  align-items: center;
  justify-content: center;
  background:
    radial-gradient(ellipse at 20% 50%, rgba(59,130,246,.08) 0%, transparent 60%),
    radial-gradient(ellipse at 80% 20%, rgba(6,182,212,.06) 0%, transparent 50%),
    var(--bg);
  padding: 20px;
}

.login-box {
  width: 100%;
  max-width: 400px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 40px;
}

.login-logo { text-align: center; margin-bottom: 32px; }
.login-logo .ice-icon { font-size: 3rem; display: block; margin-bottom: 8px; }
.login-logo h1 { font-size: 1.5rem; font-weight: 700; color: var(--cyan); letter-spacing: -.5px; }
.login-logo p { font-size: .78rem; color: var(--text-muted); margin-top: 4px; letter-spacing: .1em; text-transform: uppercase; }

.form-group { margin-bottom: 16px; }
.form-group label { display: block; font-size: .78rem; font-weight: 600; color: var(--text-dim); margin-bottom: 6px; letter-spacing: .05em; text-transform: uppercase; }

.form-group input {
  width: 100%; padding: 12px 14px;
  background: var(--surface2); border: 1px solid var(--border);
  border-radius: var(--radius-sm); color: var(--text);
  font-family: var(--font-mono); font-size: 1rem; letter-spacing: .08em;
  text-transform: uppercase; transition: border-color .2s;
}
.form-group input::placeholder { text-transform: none; letter-spacing: normal; opacity: .4; font-size: .85rem; }
.form-group input:focus { outline: none; border-color: var(--blue); }
.form-group input[type="password"] { text-transform: none; letter-spacing: .15em; }

.btn-primary {
  width: 100%; padding: 13px; background: var(--blue); color: #fff;
  border: none; border-radius: var(--radius-sm); font-family: var(--font-ui);
  font-size: .95rem; font-weight: 700; cursor: pointer; letter-spacing: .05em;
  transition: background .2s, transform .1s; margin-top: 8px;
}
.btn-primary:hover { background: #2563eb; }
.btn-primary:active { transform: scale(.99); }
.btn-primary:disabled { background: var(--border2); cursor: not-allowed; color: var(--text-muted); }

.login-error {
  background: rgba(239,68,68,.1); border: 1px solid rgba(239,68,68,.3);
  border-radius: var(--radius-sm); padding: 10px 14px;
  font-size: .85rem; color: var(--red); margin-top: 12px; display: none;
}

.login-hint {
  text-align: center; margin-top: 20px; padding-top: 16px;
  border-top: 1px solid var(--border);
  font-size: .75rem; color: var(--text-muted);
}
.login-hint span { color: var(--text-dim); font-weight: 600; }

/* ══════════════════════════════════════
   HALAMAN APLIKASI
══════════════════════════════════════ */
#page-app { flex-direction: column; }

.topbar {
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 24px; height: 60px;
  background: var(--surface); border-bottom: 1px solid var(--border);
  position: sticky; top: 0; z-index: 100; flex-shrink: 0;
  gap: 12px;
}

.topbar-left { display: flex; align-items: center; gap: 14px; }
.topbar-logo { font-size: .95rem; font-weight: 700; color: var(--cyan); letter-spacing: -.3px; }

.outlet-badge {
  background: var(--blue-dim); border: 1px solid var(--blue);
  color: var(--blue); font-size: .72rem; font-weight: 700;
  padding: 3px 10px; border-radius: 20px; letter-spacing: .1em; text-transform: uppercase;
  font-family: var(--font-mono);
}

.topbar-right { display: flex; align-items: center; gap: 10px; }

.topbar-clock {
  font-family: var(--font-mono); font-size: .85rem; color: var(--text-muted);
  background: var(--surface2); padding: 4px 10px; border-radius: var(--radius-sm);
  border: 1px solid var(--border);
}

.tab-nav { display: flex; gap: 4px; background: var(--surface2); border-radius: var(--radius-sm); padding: 3px; }
.tab-btn {
  padding: 5px 14px; border: none; border-radius: 5px;
  font-family: var(--font-ui); font-size: .8rem; font-weight: 600;
  cursor: pointer; background: transparent; color: var(--text-muted); transition: all .2s;
}
.tab-btn.active { background: var(--blue); color: #fff; }

.btn-logout {
  padding: 6px 14px; background: transparent; border: 1px solid var(--border2);
  border-radius: var(--radius-sm); color: var(--text-muted); font-family: var(--font-ui);
  font-size: .8rem; font-weight: 600; cursor: pointer; transition: all .2s;
}
.btn-logout:hover { border-color: var(--red); color: var(--red); }

/* ── PANEL KONTEN ── */
.content-panel { display: none; flex: 1; }
.content-panel.active { display: block; }

/* ══════════════════════════════════════
   PANEL TIMER
══════════════════════════════════════ */
#panel-timer {
  padding: 24px; max-width: 1100px; margin: 0 auto; width: 100%;
}

.panel-header {
  display: flex; align-items: center; justify-content: space-between; margin-bottom: 20px;
}
.panel-title { font-size: 1.1rem; font-weight: 700; color: var(--text); }
.panel-subtitle { font-size: .8rem; color: var(--text-muted); margin-top: 2px; }

.sync-status { display: flex; align-items: center; gap: 6px; font-size: .78rem; color: var(--green); }
.sync-dot { width: 7px; height: 7px; background: var(--green); border-radius: 50%; animation: pulse-dot 2s infinite; }
@keyframes pulse-dot { 0%,100% { opacity: 1; } 50% { opacity: .3; } }

.timer-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 16px; }

/* ── KARTU TIMER ── */
.timer-card {
  background: var(--surface); border: 1px solid var(--border);
  border-radius: var(--radius); padding: 20px;
  transition: all .3s; position: relative; overflow: hidden;
}
.timer-card::before {
  content: ''; position: absolute; top: 0; left: 0; right: 0;
  height: 3px; background: var(--border2); transition: background .3s;
}
.timer-card.state-running::before { background: var(--blue); }
.timer-card.state-warning::before { background: var(--yellow); }
.timer-card.state-done::before    { background: var(--red); animation: flash-bar .5s infinite alternate; }
@keyframes flash-bar { from { opacity: 1; } to { opacity: .3; } }

.timer-card.state-running { border-color: rgba(59,130,246,.4); background: linear-gradient(135deg, var(--surface), rgba(59,130,246,.05)); }
.timer-card.state-warning { border-color: rgba(245,158,11,.4); background: linear-gradient(135deg, var(--surface), rgba(245,158,11,.05)); }
.timer-card.state-done    { border-color: rgba(239,68,68,.5);  background: linear-gradient(135deg, var(--surface), rgba(239,68,68,.08)); }

.card-header { display: flex; align-items: flex-start; justify-content: space-between; margin-bottom: 14px; }
.card-name { font-size: .9rem; font-weight: 700; color: var(--text); letter-spacing: .05em; text-transform: uppercase; }

.card-badge {
  font-size: .65rem; font-weight: 700; padding: 2px 8px;
  border-radius: 10px; text-transform: uppercase; letter-spacing: .08em;
}
.badge-idle    { background: var(--surface2); color: var(--text-muted); }
.badge-running { background: var(--blue-dim); color: var(--blue); }
.badge-warning { background: var(--yellow-dim); color: var(--yellow); }
.badge-done    { background: var(--red-dim); color: var(--red); animation: badge-flash .6s infinite alternate; }
@keyframes badge-flash { from { opacity: 1; } to { opacity: .5; } }

.countdown-display {
  font-family: var(--font-mono); font-size: 2.4rem; font-weight: 700;
  text-align: center; margin: 12px 0 6px; color: var(--text-dim);
  letter-spacing: .05em; transition: color .3s;
}
.state-running .countdown-display { color: var(--cyan); }
.state-warning .countdown-display { color: var(--yellow); }
.state-done    .countdown-display { color: var(--red); font-size: 1.5rem; }

.end-time-label { font-size: .75rem; color: var(--text-muted); text-align: center; margin-bottom: 14px; font-family: var(--font-mono); }

.alarm-msg {
  font-size: .78rem; font-weight: 600; text-align: center;
  padding: 6px 10px; border-radius: var(--radius-sm); margin-bottom: 12px; display: none;
}
.alarm-msg.warning { background: rgba(245,158,11,.15); color: var(--yellow); border: 1px solid rgba(245,158,11,.3); display: block; }
.alarm-msg.done    { background: rgba(239,68,68,.15); color: var(--red); border: 1px solid rgba(239,68,68,.3); display: block; }

.card-controls { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
.duration-wrap { display: flex; align-items: center; gap: 6px; flex: 1; }
.duration-wrap label { font-size: .72rem; color: var(--text-muted); white-space: nowrap; }
.duration-wrap input {
  width: 60px; padding: 8px 6px; text-align: center;
  background: var(--surface2); border: 1px solid var(--border); border-radius: var(--radius-sm);
  color: var(--text); font-family: var(--font-mono); font-size: .9rem; transition: border-color .2s;
}
.duration-wrap input:focus { outline: none; border-color: var(--blue); }
.duration-wrap input:read-only { color: var(--text-muted); }

.btn-start {
  padding: 8px 14px; background: var(--blue); color: #fff; border: none;
  border-radius: var(--radius-sm); font-family: var(--font-ui); font-size: .8rem;
  font-weight: 700; cursor: pointer; transition: all .2s; white-space: nowrap;
}
.btn-start:hover:not(:disabled) { background: #2563eb; }
.btn-start:disabled { background: var(--border2); color: var(--text-muted); cursor: not-allowed; }
.btn-start.syncing { background: var(--orange); animation: pulse-sync .8s infinite alternate; }
@keyframes pulse-sync { from { opacity: 1; } to { opacity: .7; } }

.btn-stop {
  padding: 8px 14px; background: var(--red); color: #fff; border: none;
  border-radius: var(--radius-sm); font-family: var(--font-ui); font-size: .8rem;
  font-weight: 700; cursor: pointer; white-space: nowrap;
  animation: pulse-stop .5s infinite alternate;
}
@keyframes pulse-stop { from { box-shadow: 0 0 8px rgba(239,68,68,.5); } to { box-shadow: none; } }

.btn-reset {
  padding: 8px 12px; background: transparent; color: var(--text-muted);
  border: 1px solid var(--border2); border-radius: var(--radius-sm);
  font-family: var(--font-ui); font-size: .8rem; font-weight: 600;
  cursor: pointer; transition: all .2s; white-space: nowrap;
}
.btn-reset:hover:not(:disabled) { border-color: var(--red); color: var(--red); }
.btn-reset:disabled { opacity: .3; cursor: not-allowed; }

/* ══════════════════════════════════════
   PANEL RIWAYAT
══════════════════════════════════════ */
#panel-history { padding: 24px; max-width: 1100px; margin: 0 auto; width: 100%; }

.history-stats {
  display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 12px; margin-bottom: 20px;
}
.stat-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px; }
.stat-label { font-size: .72rem; font-weight: 600; color: var(--text-muted); letter-spacing: .06em; text-transform: uppercase; margin-bottom: 6px; }
.stat-value { font-family: var(--font-mono); font-size: 1.6rem; font-weight: 700; color: var(--cyan); }

.history-filters {
  display: flex; gap: 12px; flex-wrap: wrap; align-items: flex-end;
  margin-bottom: 20px; padding: 16px;
  background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius);
}
.filter-group { display: flex; flex-direction: column; gap: 4px; }
.filter-group label { font-size: .72rem; font-weight: 600; color: var(--text-muted); letter-spacing: .05em; text-transform: uppercase; }
.filter-group select,
.filter-group input[type="date"] {
  padding: 8px 12px; background: var(--surface2); border: 1px solid var(--border);
  border-radius: var(--radius-sm); color: var(--text); font-family: var(--font-ui);
  font-size: .85rem; cursor: pointer; transition: border-color .2s; color-scheme: dark;
}
.filter-group select:focus,
.filter-group input[type="date"]:focus { outline: none; border-color: var(--blue); }

.btn-filter {
  padding: 8px 18px; background: var(--blue); color: #fff; border: none;
  border-radius: var(--radius-sm); font-family: var(--font-ui); font-size: .85rem;
  font-weight: 700; cursor: pointer; transition: background .2s;
}
.btn-filter:hover { background: #2563eb; }

.history-table-wrap {
  overflow-x: auto; background: var(--surface);
  border: 1px solid var(--border); border-radius: var(--radius);
}
table { width: 100%; border-collapse: collapse; font-size: .85rem; }
thead th {
  padding: 12px 16px; text-align: left; font-size: .72rem; font-weight: 700;
  color: var(--text-muted); letter-spacing: .08em; text-transform: uppercase;
  border-bottom: 1px solid var(--border); white-space: nowrap;
}
tbody tr { border-bottom: 1px solid var(--border); transition: background .15s; }
tbody tr:last-child { border-bottom: none; }
tbody tr:hover { background: var(--surface2); }
tbody td { padding: 12px 16px; color: var(--text-dim); vertical-align: middle; }
.td-item { font-weight: 700; color: var(--text); letter-spacing: .04em; }
.td-outlet {
  background: var(--blue-dim); color: var(--blue);
  font-size: .7rem; font-weight: 700; padding: 2px 8px;
  border-radius: 10px; display: inline-block; letter-spacing: .06em;
  text-transform: uppercase; white-space: nowrap; font-family: var(--font-mono);
}
.td-mono { font-family: var(--font-mono); font-size: .82rem; }
.status-badge {
  font-size: .72rem; font-weight: 700; padding: 3px 10px; border-radius: 10px;
  text-transform: uppercase; letter-spacing: .06em; display: inline-block; white-space: nowrap;
}
.status-completed { background: var(--green-dim); color: var(--green); }
.status-reset     { background: var(--surface2); color: var(--text-muted); }
.status-running   { background: var(--blue-dim); color: var(--blue); }

.history-empty { text-align: center; padding: 60px 20px; color: var(--text-muted); }
.history-empty .empty-icon { font-size: 3rem; margin-bottom: 12px; }
.history-empty p { font-size: .9rem; }
.history-loading { text-align: center; padding: 40px; color: var(--text-muted); font-size: .9rem; }

/* ══════════════════════════════════════
   PANEL ADMIN
══════════════════════════════════════ */
#panel-admin { padding: 24px; max-width: 800px; margin: 0 auto; width: 100%; }

.admin-section {
  background: var(--surface); border: 1px solid var(--border);
  border-radius: var(--radius); padding: 24px; margin-bottom: 20px;
}
.admin-section-title {
  font-size: .9rem; font-weight: 700; color: var(--purple);
  letter-spacing: .05em; text-transform: uppercase; margin-bottom: 16px;
  padding-bottom: 12px; border-bottom: 1px solid var(--border);
  display: flex; align-items: center; gap: 8px;
}

.outlet-list { display: flex; flex-direction: column; gap: 10px; margin-bottom: 16px; }
.outlet-row {
  display: flex; align-items: center; gap: 12px;
  padding: 12px 14px; background: var(--surface2);
  border: 1px solid var(--border); border-radius: var(--radius-sm);
}
.outlet-code-tag {
  font-family: var(--font-mono); font-size: .85rem; font-weight: 700;
  color: var(--cyan); background: rgba(6,182,212,.1); padding: 4px 10px;
  border-radius: var(--radius-sm); min-width: 90px; text-align: center;
}
.outlet-name-text { flex: 1; font-size: .9rem; color: var(--text); }
.outlet-city-text { font-size: .78rem; color: var(--text-muted); }

.btn-danger-sm {
  padding: 5px 12px; background: transparent; color: var(--red);
  border: 1px solid var(--red-dim); border-radius: var(--radius-sm);
  font-family: var(--font-ui); font-size: .75rem; font-weight: 700;
  cursor: pointer; transition: all .2s; white-space: nowrap;
}
.btn-danger-sm:hover { background: var(--red-dim); }

.add-outlet-form { display: grid; grid-template-columns: 1fr 1fr 1fr auto; gap: 10px; align-items: end; }
.add-outlet-form .form-group { margin-bottom: 0; }

.btn-add {
  padding: 10px 16px; background: var(--green); color: #fff; border: none;
  border-radius: var(--radius-sm); font-family: var(--font-ui); font-size: .85rem;
  font-weight: 700; cursor: pointer; transition: background .2s; white-space: nowrap;
}
.btn-add:hover { background: #16a34a; }

.admin-input {
  width: 100%; padding: 10px 12px; background: var(--surface2);
  border: 1px solid var(--border); border-radius: var(--radius-sm);
  color: var(--text); font-family: var(--font-mono); font-size: .9rem;
  letter-spacing: .05em; text-transform: uppercase;
  transition: border-color .2s;
}
.admin-input::placeholder { text-transform: none; letter-spacing: normal; opacity: .4; }
.admin-input:focus { outline: none; border-color: var(--purple); }
.admin-input.no-upper { text-transform: none; letter-spacing: normal; }

.admin-notice {
  background: rgba(168,85,247,.1); border: 1px solid rgba(168,85,247,.3);
  border-radius: var(--radius-sm); padding: 12px 16px;
  font-size: .82rem; color: var(--purple); margin-top: 16px; line-height: 1.6;
}

/* ── ALARM FLASH ── */
body.alarm-flash { animation: body-flash .4s infinite alternate; }
@keyframes body-flash { from { background-color: var(--bg); } to { background-color: #1a0a0a; } }

/* ══════════════════════════════════════
   RESPONSIF
══════════════════════════════════════ */
@media (max-width: 680px) {
  .topbar { padding: 0 14px; gap: 8px; }
  .topbar-clock { display: none; }
  .tab-btn { padding: 5px 10px; font-size: .75rem; }
  .btn-logout { padding: 5px 10px; font-size: .75rem; }
  #panel-timer, #panel-history, #panel-admin { padding: 14px; }
  .timer-grid { grid-template-columns: 1fr; gap: 12px; }
  .countdown-display { font-size: 2rem; }
  .history-filters { gap: 10px; }
  .add-outlet-form { grid-template-columns: 1fr 1fr; }
  .add-outlet-form .btn-add { grid-column: span 2; }
}
</style>
</head>
<body>

<!-- ══════════════════════════════════════
     HALAMAN: LOGIN
══════════════════════════════════════ -->
<div id="page-login" class="page active">
  <div class="login-box">
    <div class="login-logo">
      <span class="ice-icon">🧊</span>
      <h1>Gacoan Thawing</h1>
      <p>Sistem Manajemen Keamanan Pangan</p>
    </div>

    <div class="form-group">
      <label>Kode Resto</label>
      <input type="text" id="login-code" placeholder="contoh: MLGMON" maxlength="10"
        oninput="this.value=this.value.toUpperCase()" autocomplete="off" spellcheck="false">
    </div>

    <div class="form-group">
      <label>Password</label>
      <input type="password" id="login-pass" placeholder="••••••••" autocomplete="current-password">
    </div>

    <button class="btn-primary" id="btn-login" onclick="doLogin()">MASUK</button>
    <div class="login-error" id="login-error"></div>

    <div class="login-hint">
      Belum punya akun? Hubungi <span>Admin / Store Manager</span>
    </div>
  </div>
</div>

<!-- ══════════════════════════════════════
     HALAMAN: APLIKASI
══════════════════════════════════════ -->
<div id="page-app" class="page">

  <!-- TOPBAR -->
  <div class="topbar">
    <div class="topbar-left">
      <span class="topbar-logo">🧊 Gacoan Thawing</span>
      <span class="outlet-badge" id="outlet-badge">—</span>
    </div>
    <div class="topbar-right">
      <span class="topbar-clock" id="topbar-clock"></span>
      <div class="tab-nav" id="tab-nav">
        <button class="tab-btn active" onclick="switchTab('timer')">⏱ Timer</button>
        <button class="tab-btn" onclick="switchTab('history')">📋 Riwayat</button>
        <button class="tab-btn" id="tab-admin-btn" style="display:none;" onclick="switchTab('admin')">⚙️ Admin</button>
      </div>
      <button class="btn-logout" onclick="doLogout()">Keluar</button>
    </div>
  </div>

  <!-- PANEL TIMER -->
  <div id="panel-timer" class="content-panel active">
    <div style="padding:24px; max-width:1100px; margin:0 auto; width:100%;">
      <div class="panel-header">
        <div>
          <div class="panel-title">Timer Thawing & Tiris</div>
          <div class="panel-subtitle" id="timer-subtitle">Tersinkronisasi real-time khusus perangkat outlet ini</div>
        </div>
        <div class="sync-status"><div class="sync-dot"></div><span>Live Sync</span></div>
      </div>
      <div class="timer-grid" id="timer-grid"></div>
    </div>
  </div>

  <!-- PANEL RIWAYAT -->
  <div id="panel-history" class="content-panel">
    <div class="panel-header">
      <div>
        <div class="panel-title">Riwayat Aktivitas</div>
        <div class="panel-subtitle">Log semua aktivitas thawing dan penirisan</div>
      </div>
    </div>

    <div class="history-stats">
      <div class="stat-card"><div class="stat-label">Total Sesi</div><div class="stat-value" id="stat-total">—</div></div>
      <div class="stat-card"><div class="stat-label">Selesai</div><div class="stat-value" id="stat-completed">—</div></div>
      <div class="stat-card"><div class="stat-label">Di-Reset</div><div class="stat-value" id="stat-reset">—</div></div>
    </div>

    <div class="history-filters">
      <div class="filter-group">
        <label>Dari Tanggal</label>
        <input type="date" id="filter-from">
      </div>
      <div class="filter-group">
        <label>Sampai Tanggal</label>
        <input type="date" id="filter-to">
      </div>
      <div class="filter-group" id="filter-outlet-group">
        <label>Outlet</label>
        <select id="filter-outlet">
          <option value="">Semua Outlet</option>
        </select>
      </div>
      <div class="filter-group">
        <label>Bahan</label>
        <select id="filter-item">
          <option value="">Semua Bahan</option>
        </select>
      </div>
      <button class="btn-filter" onclick="loadHistory()">Tampilkan</button>
    </div>

    <div class="history-table-wrap">
      <div id="history-loading" class="history-loading">Pilih filter dan klik Tampilkan.</div>
      <table id="history-table" style="display:none;">
        <thead>
          <tr>
            <th>Waktu Mulai</th>
            <th>Bahan</th>
            <th>Outlet</th>
            <th>Durasi</th>
            <th>Target Selesai</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody id="history-tbody"></tbody>
      </table>
      <div id="history-empty" class="history-empty" style="display:none;">
        <div class="empty-icon">📋</div>
        <p>Tidak ada data untuk filter yang dipilih.</p>
      </div>
    </div>
  </div>

  <!-- PANEL ADMIN -->
  <div id="panel-admin" class="content-panel">
    <div class="panel-header">
      <div>
        <div class="panel-title">⚙️ Manajemen Outlet</div>
        <div class="panel-subtitle">Tambah, lihat, dan hapus akun outlet</div>
      </div>
    </div>

    <div class="admin-section">
      <div class="admin-section-title">🏪 Daftar Outlet Terdaftar</div>
      <div class="outlet-list" id="outlet-list">
        <div style="color:var(--text-muted); font-size:.85rem;">Memuat daftar outlet...</div>
      </div>
    </div>

    <div class="admin-section">
      <div class="admin-section-title">➕ Tambah Outlet Baru</div>
      <div class="add-outlet-form">
        <div class="form-group">
          <label style="font-size:.78rem; color:var(--text-dim); text-transform:uppercase; letter-spacing:.05em; display:block; margin-bottom:6px;">Kode Resto</label>
          <input class="admin-input" type="text" id="new-code" placeholder="MLGMON" maxlength="10"
            oninput="this.value=this.value.toUpperCase()">
        </div>
        <div class="form-group">
          <label style="font-size:.78rem; color:var(--text-dim); text-transform:uppercase; letter-spacing:.05em; display:block; margin-bottom:6px;">Nama Outlet</label>
          <input class="admin-input no-upper" type="text" id="new-name" placeholder="Mondoroko">
        </div>
        <div class="form-group">
          <label style="font-size:.78rem; color:var(--text-dim); text-transform:uppercase; letter-spacing:.05em; display:block; margin-bottom:6px;">Password</label>
          <input class="admin-input no-upper" type="text" id="new-pass" placeholder="password outlet">
        </div>
        <button class="btn-add" onclick="addOutlet()">+ Tambah</button>
      </div>
      <div class="admin-notice">
        ⚠️ <strong>Kode Resto</strong> harus unik dan singkat (maksimal 10 karakter). Contoh: <strong>MLGMON</strong> (Malang Mondoroko), <strong>MLGDIR</strong> (Malang Dirgantara). Password akan tersimpan terenkripsi sederhana.
      </div>
    </div>
  </div>

</div><!-- end page-app -->

<script>
// ════════════════════════════════════════════
// KONFIGURASI FIREBASE
// ════════════════════════════════════════════
const firebaseConfig = {
  apiKey:            "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
  authDomain:        "thawing-ec583.firebaseapp.com",
  databaseURL:       "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId:         "thawing-ec583",
  storageBucket:     "thawing-ec583.firebasestorage.app",
  messagingSenderId: "1043079332713",
  appId:             "1:1043079332713:web:6d289ad2b7c13a222bb3f8"
};

// DAFTAR ITEM (Thawing + Tiris Cabe)
const THAWING_ITEMS = [
  { id: 'adonan',       name: 'ADONAN',        defaultMinutes: 40  },
  { id: 'acin',         name: 'ACIN',          defaultMinutes: 120 },
  { id: 'mie',          name: 'MIE',           defaultMinutes: 120 },
  { id: 'pentol',       name: 'PENTOL',        defaultMinutes: 120 },
  { id: 'surai_naga',   name: 'SURAI NAGA',    defaultMinutes: 120 },
  { id: 'krupuk_mie',   name: 'KRUPUK MIE',    defaultMinutes: 120 },
  { id: 'kulit_pangsit', name: 'KULIT PANGSIT', defaultMinutes: 120 },
  { id: 'udang_keju',   name: 'UDANG KEJU',    defaultMinutes: 120 },
  { id: 'tiris_cabe',   name: 'TIRIS CABE 🌶',  defaultMinutes: 30  },
];

const ADMIN_CODE = 'ADMIN';
const WARNING_SECS = 15 * 60;
const SESSION_KEY  = 'gacoan_session_v2';

// ════════════════════════════════════════════
// INISIALISASI FIREBASE
// ════════════════════════════════════════════
firebase.initializeApp(firebaseConfig);
const db        = firebase.database();
const firestore = firebase.firestore();

// ════════════════════════════════════════════
// STATE SISTEM
// ════════════════════════════════════════════
let session         = null;
let dbTimerRef      = null;
let activeIntervals = {};
let activeSessions  = {};
let isSpeaking      = false;
let speechQueue     = [];
let flashInterval   = null;
let titleInterval   = null;
const origTitle     = document.title;

// ════════════════════════════════════════════
// BOOT — Cek Sesi Tersimpan
// ════════════════════════════════════════════
(function boot() {
  try {
    const saved = localStorage.getItem(SESSION_KEY);
    if (saved) {
      session = JSON.parse(saved);
      startApp();
      return;
    }
  } catch(e) {}
  showPage('login');
})();

// ════════════════════════════════════════════
// LOGIN
// ════════════════════════════════════════════
async function doLogin() {
  const code  = document.getElementById('login-code').value.trim().toUpperCase();
  const pass  = document.getElementById('login-pass').value;
  const errEl = document.getElementById('login-error');
  const btn   = document.getElementById('btn-login');

  errEl.style.display = 'none';

  if (!code || !pass) {
    showError('Kode Resto dan Password wajib diisi.');
    return;
  }

  btn.textContent = 'Memeriksa...';
  btn.disabled    = true;

  try {
    const snap = await db.ref(`outlets_config/${code}`).once('value');
    const data  = snap.val();

    if (!data) {
      showError('Kode Resto tidak ditemukan.');
      btn.textContent = 'MASUK'; btn.disabled = false; return;
    }

    const passHash = btoa(pass);
    if (data.passHash !== passHash) {
      showError('Password salah.');
      btn.textContent = 'MASUK'; btn.disabled = false; return;
    }

    session = { code: code, name: data.name, isAdmin: !!data.isAdmin };
    localStorage.setItem(SESSION_KEY, JSON.stringify(session));
    btn.textContent = 'MASUK'; btn.disabled = false;
    startApp();

  } catch(e) {
    console.error(e);
    showError('Gagal terhubung ke server. Periksa koneksi internet.');
    btn.textContent = 'MASUK'; btn.disabled = false;
  }
}

function showError(msg) {
  const el = document.getElementById('login-error');
  el.textContent = msg;
  el.style.display = 'block';
}

function doLogout() {
  if (!confirm('Yakin ingin keluar?')) return;
  stopAlarm();
  clearAllIntervals();
  if (dbTimerRef) dbTimerRef.off();
  session = null;
  localStorage.removeItem(SESSION_KEY);
  showPage('login');
  document.getElementById('login-code').value = '';
  document.getElementById('login-pass').value  = '';
  document.getElementById('login-error').style.display = 'none';
}

document.getElementById('login-pass').addEventListener('keydown', e => {
  if (e.key === 'Enter') doLogin();
});

// ════════════════════════════════════════════
// START APP
// ════════════════════════════════════════════
function startApp() {
  showPage('app');

  document.getElementById('outlet-badge').textContent = session.code;
  document.getElementById('timer-subtitle').textContent =
    `Outlet: ${session.name} · Tersinkronisasi real-time`;

  const adminTabBtn = document.getElementById('tab-admin-btn');
  if (session.isAdmin) {
    adminTabBtn.style.display = 'block';
  } else {
    adminTabBtn.style.display = 'none';
  }

  // --- HANYA DENGARKAN TIMER UNTUK OUTLET SAYA SENDIRI ---
  dbTimerRef = db.ref(`outlets/${session.code}/timers`);

  updateClock();
  setInterval(updateClock, 1000);
  buildTimerGrid();
  listenTimers();

  if (Notification.permission === 'default') Notification.requestPermission();
  resumeAudioCtx();
}

// ════════════════════════════════════════════
// NAVIGASI HALAMAN / TAB
// ════════════════════════════════════════════
function showPage(name) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(`page-${name}`).classList.add('active');
}

function switchTab(tab) {
  document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
  const idx = { timer: 0, history: 1, admin: 2 };
  const tabs = document.querySelectorAll('.tab-btn');
  if (tabs[idx[tab]]) tabs[idx[tab]].classList.add('active');

  document.querySelectorAll('.content-panel').forEach(p => p.classList.remove('active'));
  document.getElementById(`panel-${tab}`).classList.add('active');

  if (tab === 'history') { initHistoryFilters(); loadHistory(); }
  if (tab === 'admin')   { loadOutletList(); }
}

function updateClock() {
  const now = new Date();
  document.getElementById('topbar-clock').textContent =
    now.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
}

// ════════════════════════════════════════════
// GRID TIMER
// ════════════════════════════════════════════
function buildTimerGrid() {
  const grid = document.getElementById('timer-grid');
  grid.innerHTML = '';
  THAWING_ITEMS.forEach(item => {
    const card = document.createElement('div');
    card.className = 'timer-card state-idle';
    card.id = `card-${item.id}`;
    card.innerHTML = `
      <div class="card-header">
        <div class="card-name">${item.name}</div>
        <span class="card-badge badge-idle" id="badge-${item.id}">Idle</span>
      </div>
      <div class="countdown-display" id="disp-${item.id}">${fmtTime(item.defaultMinutes*60)}</div>
      <div class="end-time-label" id="stlabel-${item.id}">Mulai: —</div>
      <div class="end-time-label" id="etlabel-${item.id}">Selesai: —</div>
      <div class="alarm-msg" id="amsg-${item.id}"></div>
      <div class="card-controls">
        <div class="duration-wrap">
          <label>mnt</label>
          <input type="number" id="inp-${item.id}" value="${item.defaultMinutes}" min="1" max="300"
            oninput="previewTime('${item.id}')">
        </div>
        <button class="btn-start" id="startbtn-${item.id}" onclick="startTimer('${item.id}')">START</button>
        <button class="btn-reset" id="resetbtn-${item.id}" style="display:none;" onclick="resetTimer('${item.id}')">RESET</button>
      </div>`;
    grid.appendChild(card);
  });
}

function previewTime(itemId) {
  const inp = document.getElementById(`inp-${itemId}`);
  if (inp.readOnly) return;
  document.getElementById(`disp-${itemId}`).textContent = fmtTime((parseInt(inp.value)||0)*60);
}

// ════════════════════════════════════════════
// LISTENER REALTIME FIREBASE (TERISOLASI)
// ════════════════════════════════════════════
function listenTimers() {
  if (dbTimerRef) dbTimerRef.off(); // Lepas listener sebelumnya jika ada

  dbTimerRef.on('value', snap => {
    const data = snap.val() || {};
    THAWING_ITEMS.forEach(item => {
      clearTimeout(activeIntervals[item.id]);
      const state = data[item.id];
      if (state && state.endTime) {
        tick(item.id, state.endTime, state.inputMinutes || item.defaultMinutes, state);
      } else {
        resetCardUI(item.id, item.defaultMinutes);
      }
    });
  });
}

// ════════════════════════════════════════════
// TICKING TIMER
// ════════════════════════════════════════════
function tick(itemId, endTimeMs, inputMinutes, state) {
  const card  = document.getElementById(`card-${itemId}`);
  const disp  = document.getElementById(`disp-${itemId}`);
  const badge = document.getElementById(`badge-${itemId}`);
  const stlbl = document.getElementById(`stlabel-${itemId}`);
  const etlbl = document.getElementById(`etlabel-${itemId}`);
  const amsg  = document.getElementById(`amsg-${itemId}`);
  const inp   = document.getElementById(`inp-${itemId}`);
  const sbtn  = document.getElementById(`startbtn-${itemId}`);
  const rbtn  = document.getElementById(`resetbtn-${itemId}`);
  if (!card) return;

  clearTimeout(activeIntervals[itemId]);

  inp.readOnly      = true;
  inp.value         = inputMinutes;
  sbtn.style.display = 'none';
  rbtn.style.display = 'block';

  const startFmt = state.startedAt
    ? new Date(state.startedAt).toLocaleTimeString('id-ID',{hour:'2-digit',minute:'2-digit'})
    : '—';
  stlbl.textContent = `Mulai: ${startFmt}`;

  const endFmt = new Date(endTimeMs).toLocaleTimeString('id-ID',{hour:'2-digit',minute:'2-digit'});
  etlbl.textContent = `Selesai: ${endFmt}`;

  const duration = Math.floor((endTimeMs - Date.now()) / 1000);

  if (duration > 0) {
    rbtn.textContent = 'RESET';
    rbtn.className   = 'btn-reset';
    disp.textContent = fmtTime(duration);

    if (duration <= WARNING_SECS) {
      setCardState(card, badge, 'warning');
      const remMins = Math.ceil(duration / 60);
      setAlarmMsg(amsg, 'warning', `⚠️ Sisa ${remMins} menit! Segera bersiap.`);
      if (duration % 300 === 0) {
        const name = THAWING_ITEMS.find(i=>i.id===itemId)?.name || itemId;
        enqueueSpeak(`Perhatian! Waktu ${name} sisa ${remMins} menit lagi.`);
      }
    } else {
      setCardState(card, badge, 'running');
      amsg.className = 'alarm-msg';
      amsg.textContent = '';
    }
    activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs, inputMinutes, state), 1000);

  } else {
    // TIMER HABIS
    disp.textContent = '⏰ WAKTU HABIS!';
    setCardState(card, badge, 'done');
    const name = THAWING_ITEMS.find(i=>i.id===itemId)?.name || itemId;
    
    if (name.includes('CABE')) {
      setAlarmMsg(amsg, 'done', `🚨 ${name} selesai! Segera angkat dan olah cabenya.`);
    } else {
      setAlarmMsg(amsg, 'done', `🚨 ${name} selesai! Segera ambil dan proses.`);
    }

    rbtn.textContent = '✅ SELESAI & AMBIL';
    rbtn.className   = 'btn-stop';
    sbtn.style.display = 'none';

    if (state.status !== 'done') {
      dbTimerRef.child(itemId).update({ status: 'done' }).catch(()=>{});
      triggerAlarm(name);
    }
  }
}

// ════════════════════════════════════════════
// KONTROL START / RESET
// ════════════════════════════════════════════
function startTimer(itemId) {
  resumeAudioCtx();
  const inp  = document.getElementById(`inp-${itemId}`);
  const sbtn = document.getElementById(`startbtn-${itemId}`);
  const mins = parseInt(inp.value);
  if (!mins || mins <= 0) { alert('Masukkan durasi yang valid.'); return; }

  sbtn.textContent = 'SYNCING...';
  sbtn.classList.add('syncing');
  sbtn.disabled = true;

  const endTimeMs = Date.now() + mins * 60 * 1000;

  // SIMPAN KHUSUS PADA FOLDER OUTLET YANG AKTIF
  dbTimerRef.child(itemId).set({
    endTime: endTimeMs, inputMinutes: mins,
    startedAt: Date.now(), startedBy: session.name,
    status: 'running'
  })
  .then(() => logThawingStart(itemId, mins, endTimeMs))
  .catch(err => {
    console.error(err);
    alert('Gagal memulai timer. Periksa koneksi.');
    sbtn.textContent = 'START';
    sbtn.classList.remove('syncing');
    sbtn.disabled = false;
  });
}

function resetTimer(itemId) {
  const rbtn   = document.getElementById(`resetbtn-${itemId}`);
  const isDone = rbtn && rbtn.className === 'btn-stop';
  stopAlarm();
  if (rbtn) rbtn.style.display = 'none';
  logThawingEnd(itemId, isDone ? 'completed' : 'reset');
  dbTimerRef.child(itemId).remove().catch(()=>{
    if (rbtn) rbtn.style.display = 'block';
  });
}

// ════════════════════════════════════════════
// HELPER STATUS KARTU
// ════════════════════════════════════════════
function setCardState(card, badge, state) {
  ['state-idle','state-running','state-warning','state-done'].forEach(c=>card.classList.remove(c));
  card.classList.add(`state-${state}`);
  badge.className = `card-badge badge-${state}`;
  badge.textContent = {
    idle: 'Idle', 
    running: 'Berjalan', 
    warning: '⚠ Mau Habis', 
    done: '🚨 Selesai'
  }[state];
}

function setAlarmMsg(el, type, text) {
  el.className = `alarm-msg ${type}`; el.textContent = text;
}

function resetCardUI(itemId, defaultMinutes) {
  clearTimeout(activeIntervals[itemId]); delete activeIntervals[itemId];
  const card  = document.getElementById(`card-${itemId}`);
  const disp  = document.getElementById(`disp-${itemId}`);
  const badge = document.getElementById(`badge-${itemId}`);
  const stlbl = document.getElementById(`stlabel-${itemId}`);
  const etlbl = document.getElementById(`etlabel-${itemId}`);
  const amsg  = document.getElementById(`amsg-${itemId}`);
  const inp   = document.getElementById(`inp-${itemId}`);
  const sbtn  = document.getElementById(`startbtn-${itemId}`);
  const rbtn  = document.getElementById(`resetbtn-${itemId}`);
  if (!card) return;
  setCardState(card, badge, 'idle');
  disp.textContent  = fmtTime(defaultMinutes*60);
  stlbl.textContent = 'Mulai: —';
  etlbl.textContent = 'Selesai: —';
  amsg.className    = 'alarm-msg'; amsg.textContent = '';
  inp.readOnly = false; inp.value = defaultMinutes;
  sbtn.textContent = 'START'; sbtn.classList.remove('syncing');
  sbtn.disabled = false; sbtn.style.display = 'block';
  rbtn.style.display = 'none'; rbtn.className = 'btn-reset';
}

// ════════════════════════════════════════════
// SISTEM ALARM & NOTIFIKASI
// ════════════════════════════════════════════
let audioCtx = null;
function resumeAudioCtx() {
  if (!audioCtx) audioCtx = new (window.AudioContext||window.webkitAudioContext)();
  if (audioCtx.state === 'suspended') audioCtx.resume().catch(()=>{});
}

function triggerAlarm(name) {
  sendNotif(name); 
  startFlash(); 
  startTitleFlash(name);

  if (name.includes('CABE')) {
    enqueueSpeak(`Perhatian! Waktu tiris cabe telah habis! Segera angkat dan olah cabenya!`);
  } else {
    enqueueSpeak(`Perhatian! Waktu thawing ${name} telah habis! Segera ambil bahan!`);
  }

  if ('vibrate' in navigator) navigator.vibrate([1000,500,1000]);
}

function stopAlarm() {
  if (flashInterval) { clearInterval(flashInterval); flashInterval = null; }
  if (titleInterval) { clearInterval(titleInterval); titleInterval = null; }
  document.body.classList.remove('alarm-flash');
  document.title = origTitle;
  if ('speechSynthesis' in window) window.speechSynthesis.cancel();
  if ('vibrate' in navigator) navigator.vibrate(0);
  isSpeaking = false; speechQueue.length = 0;
}

function startFlash() {
  if (flashInterval) return;
  flashInterval = setInterval(()=>document.body.classList.toggle('alarm-flash'),300);
}

function startTitleFlash(name) {
  if (titleInterval) return;
  let flip = false;
  titleInterval = setInterval(()=>{
    document.title = (flip=!flip) ? `🚨 HABIS! ${name}` : origTitle;
  }, 700);
}

function sendNotif(name) {
  if (Notification.permission === 'granted') {
    new Notification('⏰ Waktu Selesai!',{
      body:`🚨 Segera proses: ${name}`, requireInteraction:true, renotify:true, tag:'thawing'
    }).onclick = ()=>{ window.focus(); stopAlarm(); };
  }
}

function enqueueSpeak(msg) {
  if (!('speechSynthesis' in window)) return;
  if (!speechQueue.includes(msg)) speechQueue.push(msg);
  processQueue();
}

function processQueue() {
  if (isSpeaking || speechQueue.length===0) return;
  isSpeaking = true;
  const msg = speechQueue.shift();
  window.speechSynthesis.cancel();
  const utt = new SpeechSynthesisUtterance(msg);
  const v = window.speechSynthesis.getVoices().find(v=>v.lang.startsWith('id'));
  if (v) utt.voice = v; else utt.lang = 'id-ID';
  utt.rate = 1.0; utt.volume = 1.0;
  utt.onend = utt.onerror = ()=>{ isSpeaking=false; processQueue(); };
  window.speechSynthesis.speak(utt);
}

// ════════════════════════════════════════════
// FORMAT WAKTU
// ════════════════════════════════════════════
function fmtTime(s) {
  s = Math.max(0,s);
  return [Math.floor(s/3600), Math.floor((s%3600)/60), s%60]
    .map(n=>String(n).padStart(2,'0')).join(':');
}

function clearAllIntervals() {
  Object.values(activeIntervals).forEach(t=>clearTimeout(t));
  activeIntervals = {};
  stopAlarm();
}

// ════════════════════════════════════════════
// FIRESTORE — PENCATATAN RIWAYAT
// ════════════════════════════════════════════
async function logThawingStart(itemId, durationMins, endTimeMs) {
  const item = THAWING_ITEMS.find(i=>i.id===itemId);
  try {
    const ref = await firestore.collection('thawing_history').add({
      outletCode:  session.code,
      outletName:  session.name,
      itemId:      itemId,
      itemName:    item?.name || itemId,
      durationMin: durationMins,
      startedAt:   firebase.firestore.FieldValue.serverTimestamp(),
      endTime:     new Date(endTimeMs),
      status:      'running',
    });
    activeSessions[itemId] = ref.id;
  } catch(e) { console.error('Log start gagal:', e); }
}

async function logThawingEnd(itemId, status) {
  const docId = activeSessions[itemId];
  if (!docId) return;
  try {
    await firestore.collection('thawing_history').doc(docId).update({
      status:      status,
      finishedAt: firebase.firestore.FieldValue.serverTimestamp(),
    });
    delete activeSessions[itemId];
  } catch(e) { console.error('Log end gagal:', e); }
}

// ════════════════════════════════════════════
// PANEL RIWAYAT
// ════════════════════════════════════════════
async function initHistoryFilters() {
  const today = new Date().toISOString().split('T')[0];
  const fromEl = document.getElementById('filter-from');
  const toEl   = document.getElementById('filter-to');
  if (!fromEl.value) fromEl.value = today;
  if (!toEl.value)   toEl.value   = today;

  const itemSel = document.getElementById('filter-item');
  if (itemSel.options.length <= 1) {
    THAWING_ITEMS.forEach(item => {
      const o = document.createElement('option');
      o.value = item.id; o.textContent = item.name;
      itemSel.appendChild(o);
    });
  }

  const outletGroup = document.getElementById('filter-outlet-group');
  const outletSel = document.getElementById('filter-outlet');
  
  if (session.isAdmin) {
    outletGroup.style.display = 'flex';
    if (outletSel.options.length <= 1) {
      try {
        const snap = await db.ref('outlets_config').once('value');
        const data = snap.val() || {};
        Object.keys(data).forEach(code => {
          const opt = document.createElement('option');
          opt.value = code;
          opt.textContent = `${code} - ${data[code].name}`;
          outletSel.appendChild(opt);
        });
      } catch (e) {
        console.error("Gagal memuat daftar outlet filter:", e);
      }
    }
  } else {
    outletGroup.style.display = 'none';
  }
}

async function loadHistory() {
  const loadEl  = document.getElementById('history-loading');
  const tableEl = document.getElementById('history-table');
  const emptyEl = document.getElementById('history-empty');
  const tbody   = document.getElementById('history-tbody');

  loadEl.style.display  = 'block';
  loadEl.textContent    = 'Memuat data...';
  tableEl.style.display = 'none';
  emptyEl.style.display = 'none';

  const dateFrom     = document.getElementById('filter-from').value;
  const dateTo       = document.getElementById('filter-to').value;
  const filterOutlet = document.getElementById('filter-outlet').value;
  const filterItem   = document.getElementById('filter-item').value;

  try {
    let q = firestore.collection('thawing_history').orderBy('startedAt','desc');

    const targetOutlet = session.isAdmin ? filterOutlet : session.code;
    if (targetOutlet) q = q.where('outletCode','==', targetOutlet);

    if (filterItem) q = q.where('itemId','==', filterItem);

    if (dateFrom) {
      const from = new Date(dateFrom); from.setHours(0,0,0,0);
      q = q.where('startedAt','>=', from);
    }
    if (dateTo) {
      const to = new Date(dateTo); to.setHours(23,59,59,999);
      q = q.where('startedAt','<=', to);
    }

    q = q.limit(300);
    const snap = await q.get();
    const docs = snap.docs.map(d=>({id:d.id,...d.data()}));

    loadEl.style.display = 'none';

    if (docs.length === 0) {
      emptyEl.style.display = 'block';
      updateStats(0,0,0); return;
    }

    tbody.innerHTML = '';
    let total=docs.length, completed=0, reset=0;

    docs.forEach(doc => {
      const startedAt = doc.startedAt?.toDate?.() || null;
      const endTime   = doc.endTime instanceof Date ? doc.endTime : doc.endTime?.toDate?.() || null;
      if (doc.status==='completed') completed++;
      else if (doc.status==='reset') reset++;

      const statusMap = {
        completed: '<span class="status-badge status-completed">✅ Selesai</span>',
        reset:     '<span class="status-badge status-reset">↩ Reset</span>',
        running:   '<span class="status-badge status-running">▶ Berjalan</span>',
      };

      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td class="td-mono">${startedAt ? fmtDatetime(startedAt) : '—'}</td>
        <td class="td-item">${doc.itemName||doc.itemId}</td>
        <td><span class="td-outlet">${doc.outletCode||'—'}</span></td>
        <td class="td-mono">${doc.durationMin ? doc.durationMin+' mnt' : '—'}</td>
        <td class="td-mono">${endTime ? fmtTime2(endTime) : '—'}</td>
        <td>${statusMap[doc.status]||'<span class="status-badge status-reset">—</span>'}</td>`;
      tbody.appendChild(tr);
    });

    updateStats(total, completed, reset);
    tableEl.style.display = 'table';

  } catch(err) {
    console.error(err);
    loadEl.textContent = '⚠ Gagal memuat data. Pastikan Firestore aktif dan index sudah dibuat.';
  }
}

function updateStats(t,c,r) {
  document.getElementById('stat-total').textContent     = t;
  document.getElementById('stat-completed').textContent = c;
  document.getElementById('stat-reset').textContent     = r;
}

function fmtDatetime(d) {
  return d.toLocaleString('id-ID',{day:'2-digit',month:'2-digit',year:'numeric',hour:'2-digit',minute:'2-digit'});
}
function fmtTime2(d) {
  return d.toLocaleTimeString('id-ID',{hour:'2-digit',minute:'2-digit'});
}

// ════════════════════════════════════════════
// KELOLA OUTLET (KHUSUS ADMIN)
// ════════════════════════════════════════════
async function loadOutletList() {
  const listEl = document.getElementById('outlet-list');
  listEl.innerHTML = '<div style="color:var(--text-muted);font-size:.85rem;">Memuat...</div>';

  try {
    const snap = await db.ref('outlets_config').once('value');
    const data = snap.val() || {};
    const codes = Object.keys(data);

    if (codes.length === 0) {
      listEl.innerHTML = '<div style="color:var(--text-muted);font-size:.85rem;">Belum ada outlet terdaftar.</div>';
      return;
    }

    listEl.innerHTML = '';
    codes.forEach(code => {
      const o = data[code];
      const row = document.createElement('div');
      row.className = 'outlet-row';
      row.innerHTML = `
        <span class="outlet-code-tag">${code}</span>
        <div style="flex:1;">
          <div class="outlet-name-text">${o.name}</div>
          ${o.isAdmin ? '<div class="outlet-city-text" style="color:var(--purple);">★ Admin</div>' : ''}
        </div>
        <button class="btn-danger-sm" onclick="deleteOutlet('${code}')">Hapus</button>`;
      listEl.appendChild(row);
    });
  } catch(e) {
    listEl.innerHTML = '<div style="color:var(--red);font-size:.85rem;">Gagal memuat daftar outlet.</div>';
  }
}

async function addOutlet() {
  const code = document.getElementById('new-code').value.trim().toUpperCase();
  const name = document.getElementById('new-name').value.trim();
  const pass = document.getElementById('new-pass').value.trim();

  if (!code || !name || !pass) { alert('Semua kolom wajib diisi.'); return; }
  if (code.length < 3) { alert('Kode minimal 3 karakter.'); return; }

  try {
    const existing = await db.ref(`outlets_config/${code}`).once('value');
    if (existing.val()) { alert(`Kode ${code} sudah digunakan.`); return; }

    await db.ref(`outlets_config/${code}`).set({
      name:      name,
      passHash:  btoa(pass),
      isAdmin:   false,
      createdAt: Date.now(),
    });

    document.getElementById('new-code').value = '';
    document.getElementById('new-name').value = '';
    document.getElementById('new-pass').value = '';
    alert(`✅ Outlet ${code} - ${name} berhasil ditambahkan!`);
    loadOutletList();
  } catch(e) {
    console.error("Detail Error:", e);
    alert(`Gagal menambahkan outlet. Alasan: ${e.message}`);
  }
}

async function deleteOutlet(code) {
  if (code === session.code) { alert('Tidak bisa menghapus outlet yang sedang aktif.'); return; }
  if (!confirm(`Apakah Anda yakin ingin menghapus outlet ${code}?`)) return;
  try {
    await db.ref(`outlets_config/${code}`).remove();
    loadOutletList();
  } catch(e) { alert('Gagal menghapus outlet.'); }
}
</script>
</body>
</html>
