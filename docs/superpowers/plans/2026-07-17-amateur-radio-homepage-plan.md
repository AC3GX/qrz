# AC3GX Amateur Radio Homepage Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a highly polished, responsive "Radio Shack Console" style dashboard homepage for amateur radio operator AC3GX in a single self-contained HTML file.

**Architecture:** A single-page application (SPA) layout powered by CSS Grid and Flexbox for responsiveness, using inline SVGs for layout icons, LocalStorage for state persistence, and native Web Audio API for Morse code tone generation.

**Tech Stack:** HTML5, CSS3, JavaScript (ES6+).

## Global Constraints
- Single self-contained HTML file `index.html` located at the project root.
- No external CSS or JS frameworks allowed (no Tailwind, no React, no jQuery, etc.).
- Dark mode primary console aesthetic with glowing neon/amber/cyan indicator highlights.
- Zero network dependencies beyond Google Fonts (`Outfit` and `JetBrains Mono`).

---

### Task 1: Scaffolding, Basic Layout, Clocks & Header

**Files:**
- Create: `index.html`
- Test: Verify page renders in browser and clocks update every second.

**Interfaces:**
- Produces: CSS custom variables, base skeleton structure, live UTC/Local clock ticker function.

- [ ] **Step 1: Create index.html with HTML5 skeleton and Google Fonts**
  Write the core HTML layout, embedding CSS styles and establishing the responsive grid framework.
  
  Code for `index.html`:
  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AC3GX // Radio Shack Console</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Outfit:wght@300;400;600;700&display=swap" rel="stylesheet">
    <style>
      :root {
        --bg-primary: #0b0c10;
        --bg-panel: #1a1a1f;
        --bg-card: #202026;
        --border-color: #2c3531;
        --text-primary: #f5f5f7;
        --text-secondary: #8e8e93;
        --accent-green: #00ff87;
        --accent-cyan: #00e5ff;
        --accent-amber: #ffb300;
        --accent-glow-green: rgba(0, 255, 135, 0.2);
        --accent-glow-cyan: rgba(0, 229, 255, 0.2);
        --accent-glow-amber: rgba(255, 179, 0, 0.2);
      }
      * { box-sizing: border-box; margin: 0; padding: 0; }
      body {
        background-color: var(--bg-primary);
        color: var(--text-primary);
        font-family: 'Outfit', sans-serif;
        min-height: 100vh;
        display: flex;
        flex-direction: column;
        overflow-x: hidden;
      }
      header {
        background: var(--bg-panel);
        border-bottom: 2px solid var(--border-color);
        padding: 1rem 1.5rem;
        display: flex;
        flex-wrap: wrap;
        justify-content: space-between;
        align-items: center;
        gap: 1rem;
      }
      .logo-section {
        display: flex;
        align-items: center;
        gap: 0.75rem;
      }
      .logo-call {
        font-size: 1.8rem;
        font-weight: 700;
        letter-spacing: 0.05em;
        font-family: 'JetBrains Mono', monospace;
        color: var(--text-primary);
      }
      .logo-led {
        width: 10px;
        height: 10px;
        background-color: var(--accent-green);
        border-radius: 50%;
        box-shadow: 0 0 10px var(--accent-green);
        animation: pulse 2s infinite alternate;
      }
      @keyframes pulse {
        0% { opacity: 0.4; box-shadow: 0 0 2px var(--accent-green); }
        100% { opacity: 1; box-shadow: 0 0 12px var(--accent-green); }
      }
      .clocks-section {
        display: flex;
        gap: 1.5rem;
        font-family: 'JetBrains Mono', monospace;
      }
      .clock-item {
        background: var(--bg-card);
        padding: 0.5rem 0.75rem;
        border-radius: 6px;
        border: 1px solid var(--border-color);
        min-width: 130px;
        text-align: center;
      }
      .clock-label {
        font-size: 0.7rem;
        text-transform: uppercase;
        color: var(--text-secondary);
        display: block;
      }
      .clock-time {
        font-size: 1.1rem;
        font-weight: bold;
        color: var(--accent-cyan);
      }
      .status-section {
        display: flex;
        align-items: center;
        gap: 1rem;
        font-family: 'JetBrains Mono', monospace;
      }
      .freq-display {
        color: var(--accent-amber);
        font-weight: bold;
      }
      .power-btn {
        background: transparent;
        border: 1px solid var(--accent-green);
        color: var(--accent-green);
        padding: 0.4rem 0.8rem;
        border-radius: 4px;
        cursor: pointer;
        font-family: 'Outfit', sans-serif;
        font-size: 0.8rem;
        font-weight: 600;
        transition: all 0.2s ease;
      }
      .power-btn:hover {
        background: var(--accent-green);
        color: var(--bg-primary);
        box-shadow: 0 0 8px var(--accent-green);
      }
      .power-btn.off {
        border-color: var(--text-secondary);
        color: var(--text-secondary);
      }
      .power-btn.off:hover {
        background: var(--text-secondary);
        color: var(--bg-primary);
        box-shadow: none;
      }
      .console-grid {
        flex: 1;
        display: grid;
        grid-template-columns: 280px 1fr 300px;
        gap: 1.25rem;
        padding: 1.25rem;
        max-width: 1600px;
        margin: 0 auto;
        width: 100%;
      }
      @media (max-width: 1024px) {
        .console-grid {
          grid-template-columns: 1fr 1fr;
        }
      }
      @media (max-width: 768px) {
        .console-grid {
          grid-template-columns: 1fr;
        }
      }
      .console-card {
        background: var(--bg-panel);
        border: 2px solid var(--border-color);
        border-radius: 12px;
        padding: 1.25rem;
        display: flex;
        flex-direction: column;
        gap: 1rem;
        box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
      }
      .card-header {
        border-bottom: 1px solid var(--border-color);
        padding-bottom: 0.5rem;
        font-weight: bold;
        text-transform: uppercase;
        letter-spacing: 0.05em;
        font-size: 0.9rem;
        display: flex;
        justify-content: space-between;
        align-items: center;
      }
      footer {
        text-align: center;
        padding: 1.5rem;
        color: var(--text-secondary);
        font-size: 0.8rem;
        border-top: 1px solid var(--border-color);
        background: var(--bg-panel);
        margin-top: auto;
      }
    </style>
  </head>
  <body>
    <header>
      <div class="logo-section">
        <div class="logo-call">AC3GX</div>
        <div class="logo-led" id="power-led"></div>
      </div>
      <div class="clocks-section">
        <div class="clock-item">
          <span class="clock-label">Local Time</span>
          <span class="clock-time" id="local-clock">00:00:00</span>
        </div>
        <div class="clock-item">
          <span class="clock-label">UTC Time</span>
          <span class="clock-time" id="utc-clock">00:00:00</span>
        </div>
      </div>
      <div class="status-section">
        <div>Freq: <span class="freq-display" id="freq-text">14.074 MHz</span></div>
        <button class="power-btn" id="power-btn" onclick="togglePower()">PWR ON</button>
      </div>
    </header>

    <main class="console-grid" id="console-grid">
      <!-- Grid contents added in subsequent tasks -->
    </main>

    <footer>
      AC3GX Radio Shack Console &copy; 2026. Made with Google Antigravity IDE.
    </footer>

    <script>
      let powerOn = true;

      function updateClocks() {
        const now = new Date();
        const pad = (n) => String(n).padStart(2, '0');
        
        // Local clock
        const lH = pad(now.getHours());
        const lM = pad(now.getMinutes());
        const lS = pad(now.getSeconds());
        document.getElementById('local-clock').textContent = `${lH}:${lM}:${lS}`;

        // UTC clock
        const uH = pad(now.getUTCHours());
        const uM = pad(now.getUTCMinutes());
        const uS = pad(now.getUTCSeconds());
        document.getElementById('utc-clock').textContent = `${uH}:${uM}:${uS}`;
      }

      function togglePower() {
        powerOn = !powerOn;
        const btn = document.getElementById('power-btn');
        const led = document.getElementById('power-led');
        const grid = document.getElementById('console-grid');
        
        if (powerOn) {
          btn.textContent = "PWR ON";
          btn.classList.remove('off');
          led.style.backgroundColor = "var(--accent-green)";
          led.style.boxShadow = "0 0 10px var(--accent-green)";
          grid.style.opacity = "1";
          grid.style.pointerEvents = "auto";
        } else {
          btn.textContent = "PWR OFF";
          btn.classList.add('off');
          led.style.backgroundColor = "#ff3b30";
          led.style.boxShadow = "0 0 10px #ff3b30";
          grid.style.opacity = "0.2";
          grid.style.pointerEvents = "none";
        }
      }

      setInterval(updateClocks, 1000);
      updateClocks();
    </script>
  </body>
  </html>
  ```

- [ ] **Step 2: Commit scaffolding**
  Commit the files using:
  ```bash
  git add index.html
  git commit -m "feat: add HTML skeleton, CSS theme styling, and header clocks"
  ```

---

### Task 2: Left Column Components (Station Info & DMR Ticker)

**Files:**
- Modify: `index.html`
- Test: Verify Operator Bio card and scrolling DMR feed work properly.

**Interfaces:**
- Consumes: Grid containers in `index.html`.
- Produces: Visual station details, simulated live scrolling DMR talkgroup feed.

- [ ] **Step 1: Add HTML markup for Left Column in index.html**
  Inject the Left column wrapper and card elements inside `<main class="console-grid">`.
  Insert this HTML inside `<main class="console-grid">`:
  ```html
  <!-- LEFT COLUMN -->
  <div class="left-col" style="display:flex; flex-direction:column; gap:1.25rem;">
    <!-- BIO / STATION CARD -->
    <div class="console-card">
      <div class="card-header">
        <span>Station Profile</span>
        <span style="font-size:0.7rem; color:var(--accent-green);">● ACTIVE</span>
      </div>
      <div style="display:flex; flex-direction:column; gap:0.5rem; font-size:0.9rem;">
        <div style="background:var(--bg-card); padding:0.75rem; border-radius:6px; border:1px solid var(--border-color); text-align:center;">
          <strong style="font-size:1.1rem; color:var(--accent-green);">AC3GX</strong>
          <div style="color:var(--text-secondary); font-size:0.75rem; margin-top:0.2rem;">Amateur Radio Station</div>
        </div>
        <div><strong>Operator:</strong> Alex</div>
        <div><strong>Location:</strong> Maryland, USA</div>
        <div><strong>Grid Square:</strong> FM19ff</div>
        <div><strong>License Class:</strong> Extra</div>
        <div style="border-top:1px solid var(--border-color); margin-top:0.5rem; padding-top:0.5rem;">
          <div style="font-size:0.75rem; text-transform:uppercase; color:var(--text-secondary); margin-bottom:0.25rem;">Rig / Hardware</div>
          <div style="color:var(--accent-cyan); font-family:'JetBrains Mono', monospace; font-size:0.85rem;">ICOM IC-7300 (HF)</div>
          <div style="color:var(--accent-cyan); font-family:'JetBrains Mono', monospace; font-size:0.85rem;">LDG IT-100 Tuner</div>
          <div style="color:var(--accent-cyan); font-family:'JetBrains Mono', monospace; font-size:0.85rem;">80-10m End Fed Wire</div>
        </div>
      </div>
    </div>

    <!-- DMR MONITOR CARD -->
    <div class="console-card">
      <div class="card-header">
        <span>DMR Live Monitor</span>
        <span style="font-size:0.7rem; color:var(--accent-cyan);">BM LINK</span>
      </div>
      <div style="display:flex; flex-direction:column; gap:0.5rem; flex:1;">
        <div style="font-size:0.75rem; color:var(--text-secondary); display:flex; justify-content:space-between;">
          <span>CALLSIGN/TG</span>
          <span>RSSI/DUR</span>
        </div>
        <div id="dmr-ticker" style="display:flex; flex-direction:column; gap:0.4rem; height:180px; overflow-y:hidden; font-family:'JetBrains Mono', monospace; font-size:0.8rem;">
          <!-- DMR Ticker items injected here -->
        </div>
      </div>
    </div>
  </div>
  ```

- [ ] **Step 2: Add DMR simulation scripting in index.html**
  Insert the DMR transmission simulation logic into `<script>`.
  Insert this JS inside `<script>` tag:
  ```javascript
  const dmrTicker = document.getElementById('dmr-ticker');
  const mockCallsigns = ["K3LR", "W1AW", "N3LL", "KB3Y", "N4HH", "W3AA", "K1US", "KD8G", "W6PX", "K2XX"];
  const mockTalkgroups = ["TG 3100 (USA)", "TG 91 (WW)", "TG 3124 (MD)", "TG 93 (NA)"];
  
  function addDmrFeed() {
    if (!powerOn) return;
    const call = mockCallsigns[Math.floor(Math.random() * mockCallsigns.length)];
    const tg = mockTalkgroups[Math.floor(Math.random() * mockTalkgroups.length)];
    const rssi = `-${(70 + Math.floor(Math.random() * 40))}dBm`;
    const dur = `${(2 + Math.floor(Math.random() * 8))}s`;
    
    const row = document.createElement('div');
    row.style.background = 'var(--bg-card)';
    row.style.border = '1px solid var(--border-color)';
    row.style.padding = '0.4rem';
    row.style.borderRadius = '4px';
    row.style.display = 'flex';
    row.style.justifyContent = 'space-between';
    row.style.opacity = '0';
    row.style.transition = 'opacity 0.5s ease';
    row.style.animation = 'glowGreen 1.5s ease-out';

    row.innerHTML = `
      <div style="display:flex; flex-direction:column;">
        <span style="color:var(--accent-green); font-weight:bold;">${call}</span>
        <span style="font-size:0.7rem; color:var(--text-secondary);">${tg}</span>
      </div>
      <div style="text-align:right; display:flex; flex-direction:column;">
        <span style="color:var(--accent-cyan);">${rssi}</span>
        <span style="font-size:0.7rem; color:var(--text-secondary);">${dur}</span>
      </div>
    `;

    // Glow Animation style
    if (!document.getElementById('ticker-animations')) {
      const style = document.createElement('style');
      style.id = 'ticker-animations';
      style.textContent = `
        @keyframes glowGreen {
          0% { box-shadow: 0 0 8px var(--accent-green); border-color: var(--accent-green); }
          100% { box-shadow: none; border-color: var(--border-color); }
        }
      `;
      document.head.appendChild(style);
    }

    dmrTicker.insertBefore(row, dmrTicker.firstChild);
    setTimeout(() => row.style.opacity = '1', 50);

    // Limit maximum visible rows
    while (dmrTicker.children.length > 4) {
      dmrTicker.removeChild(dmrTicker.lastChild);
    }
  }

  // Inject initial rows
  for (let i = 0; i < 4; i++) {
    addDmrFeed();
  }

  // Ticker updater loop
  setInterval(addDmrFeed, 6000);
  ```

- [ ] **Step 3: Commit Left Column components**
  ```bash
  git add index.html
  git commit -m "feat: implement operator profile and simulated live DMR activity monitor"
  ```

---

### Task 3: Right Column Components (Solar Indices & Band Conditions)

**Files:**
- Modify: `index.html`
- Test: Verify solar grid displays proper propagation stats and updates.

**Interfaces:**
- Consumes: Main Grid in `index.html`.
- Produces: Solar propagation index grid and active band band-quality table.

- [ ] **Step 1: Add HTML markup for Right Column in index.html**
  Inject Right Column wrapper containing the Solar panel card into `<main class="console-grid">`.
  Insert this HTML inside `<main class="console-grid">` (we will place it after Left column and leave space for center column):
  ```html
  <!-- RIGHT COLUMN -->
  <div class="right-col" style="display:flex; flex-direction:column; gap:1.25rem;">
    <!-- SOLAR PROPAGATION CARD -->
    <div class="console-card">
      <div class="card-header">
        <span>Solar Conditions</span>
        <span id="solar-update" style="font-size:0.65rem; color:var(--text-secondary);">LIVE</span>
      </div>
      <div style="display:grid; grid-template-columns: 1fr 1fr; gap:0.5rem; font-family:'JetBrains Mono', monospace; font-size:0.85rem;">
        <div style="background:var(--bg-card); padding:0.4rem 0.6rem; border-radius:4px; border:1px solid var(--border-color); text-align:center;">
          <span style="font-size:0.65rem; color:var(--text-secondary); display:block; text-transform:uppercase;">Solar Flux</span>
          <span style="font-size:1.1rem; color:var(--accent-amber); font-weight:bold;" id="sfi-val">142</span>
        </div>
        <div style="background:var(--bg-card); padding:0.4rem 0.6rem; border-radius:4px; border:1px solid var(--border-color); text-align:center;">
          <span style="font-size:0.65rem; color:var(--text-secondary); display:block; text-transform:uppercase;">Sunspots</span>
          <span style="font-size:1.1rem; color:var(--accent-amber); font-weight:bold;" id="ssn-val">95</span>
        </div>
        <div style="background:var(--bg-card); padding:0.4rem 0.6rem; border-radius:4px; border:1px solid var(--border-color); text-align:center;">
          <span style="font-size:0.65rem; color:var(--text-secondary); display:block; text-transform:uppercase;">A-Index</span>
          <span style="font-size:1.1rem; color:var(--accent-green); font-weight:bold;" id="a-val">6</span>
        </div>
        <div style="background:var(--bg-card); padding:0.4rem 0.6rem; border-radius:4px; border:1px solid var(--border-color); text-align:center;">
          <span style="font-size:0.65rem; color:var(--text-secondary); display:block; text-transform:uppercase;">K-Index</span>
          <span style="font-size:1.1rem; color:var(--accent-green); font-weight:bold;" id="k-val">2</span>
        </div>
      </div>
      
      <div style="border-top:1px solid var(--border-color); padding-top:0.75rem; margin-top:0.25rem;">
        <div style="font-size:0.75rem; text-transform:uppercase; color:var(--text-secondary); margin-bottom:0.5rem; font-weight:600;">HF Band Conditions</div>
        <div style="display:grid; grid-template-columns: repeat(2, 1fr); gap:0.5rem; font-size:0.75rem; font-family:'JetBrains Mono', monospace;">
          <div style="display:flex; justify-content:space-between; align-items:center; background:var(--bg-card); padding:0.35rem 0.5rem; border-radius:4px; border:1px solid var(--border-color);">
            <span>80m-40m</span>
            <span style="color:var(--accent-green); font-weight:bold;">GOOD (DX)</span>
          </div>
          <div style="display:flex; justify-content:space-between; align-items:center; background:var(--bg-card); padding:0.35rem 0.5rem; border-radius:4px; border:1px solid var(--border-color);">
            <span>30m-20m</span>
            <span style="color:var(--accent-green); font-weight:bold;">GOOD (DX)</span>
          </div>
          <div style="display:flex; justify-content:space-between; align-items:center; background:var(--bg-card); padding:0.35rem 0.5rem; border-radius:4px; border:1px solid var(--border-color);">
            <span>17m-15m</span>
            <span style="color:var(--accent-amber); font-weight:bold;">FAIR</span>
          </div>
          <div style="display:flex; justify-content:space-between; align-items:center; background:var(--bg-card); padding:0.35rem 0.5rem; border-radius:4px; border:1px solid var(--border-color);">
            <span>12m-10m</span>
            <span style="color:#ff3b30; font-weight:bold;">POOR</span>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Placeholder for Morse keyer widget in next task -->
    <div id="morse-keyer-container"></div>
  </div>
  ```

- [ ] **Step 2: Add script to randomize solar flux conditions slightly to look interactive**
  Insert a function in the script tag to refresh solar metrics every 30 seconds.
  Insert this JS inside `<script>` tag:
  ```javascript
  function updateSolarIndices() {
    if (!powerOn) return;
    const currentSFI = 135 + Math.floor(Math.random() * 15);
    const currentSSN = 85 + Math.floor(Math.random() * 20);
    const currentA = 4 + Math.floor(Math.random() * 5);
    const currentK = 1 + Math.floor(Math.random() * 3);
    
    document.getElementById('sfi-val').textContent = currentSFI;
    document.getElementById('ssn-val').textContent = currentSSN;
    document.getElementById('a-val').textContent = currentA;
    document.getElementById('k-val').textContent = currentK;
    
    const kColor = currentK > 3 ? '#ff3b30' : (currentK > 2 ? 'var(--accent-amber)' : 'var(--accent-green)');
    document.getElementById('k-val').style.color = kColor;
    
    const updateTime = new Date();
    document.getElementById('solar-update').textContent = `LST UPD: ${updateTime.getUTCHours()}:${updateTime.getUTCMinutes()}:${updateTime.getUTCSeconds()} UTC`;
  }
  
  setInterval(updateSolarIndices, 30000);
  updateSolarIndices();
  ```

- [ ] **Step 3: Commit Right Column components**
  ```bash
  git add index.html
  git commit -m "feat: add solar index panel and simulated HF band propagation status grid"
  ```

---

### Task 4: Center Column (FT8 Logbook & ADIF Exporter)

**Files:**
- Modify: `index.html`
- Test: Verify QSOs display, search works, and "Export ADIF" works.

**Interfaces:**
- Consumes: Main Grid in `index.html`.
- Produces: Log submission form, search box, scrollable contact history, ADIF downloader.

- [ ] **Step 1: Add HTML markup for Center Column in index.html**
  Inject the Center Column container containing the Log Form and Table. Insert this HTML inside `<main class="console-grid">` (before Right Column):
  ```html
  <!-- CENTER COLUMN -->
  <div class="center-col" style="display:flex; flex-direction:column; gap:1.25rem;">
    <!-- LOG BOOK CARD -->
    <div class="console-card">
      <div class="card-header">
        <span>QSO Logbook (Digital)</span>
        <button onclick="exportADIF()" style="background:transparent; border:1px solid var(--accent-cyan); color:var(--accent-cyan); padding:0.25rem 0.5rem; border-radius:4px; font-size:0.75rem; cursor:pointer; font-weight:600;">Export ADIF</button>
      </div>

      <!-- QSO ADD FORM -->
      <form id="qso-form" onsubmit="addQSO(event)" style="background:var(--bg-card); border:1px solid var(--border-color); border-radius:6px; padding:0.75rem; display:grid; grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); gap:0.5rem; align-items:flex-end;">
        <div>
          <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">Callsign</label>
          <input type="text" id="qso-call" required placeholder="e.g. W1AW" style="width:100%; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-family:'JetBrains Mono', monospace; text-transform:uppercase;">
        </div>
        <div>
          <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">Band</label>
          <select id="qso-band" style="width:100%; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white;">
            <option value="20m">20m</option>
            <option value="40m">40m</option>
            <option value="80m">80m</option>
            <option value="15m">15m</option>
            <option value="10m">10m</option>
          </select>
        </div>
        <div>
          <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">Mode</label>
          <select id="qso-mode" style="width:100%; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white;">
            <option value="FT8">FT8</option>
            <option value="FT4">FT4</option>
            <option value="PSK31">PSK31</option>
          </select>
        </div>
        <div>
          <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">RST (Sent)</label>
          <input type="text" id="qso-rst-sent" placeholder="-12" style="width:100%; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-family:'JetBrains Mono', monospace;">
        </div>
        <div>
          <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">RST (Rcvd)</label>
          <input type="text" id="qso-rst-rcvd" placeholder="-08" style="width:100%; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-family:'JetBrains Mono', monospace;">
        </div>
        <button type="submit" style="background:var(--accent-green); color:var(--bg-primary); border:none; padding:0.45rem; border-radius:4px; font-weight:bold; cursor:pointer; height:32px; font-size:0.8rem;">LOG QSO</button>
      </form>

      <!-- SEARCH FILTER -->
      <div style="display:flex; justify-content:space-between; align-items:center; gap:0.5rem; margin-top:0.25rem;">
        <input type="text" id="log-search" oninput="filterLogs()" placeholder="Search calls..." style="background:var(--bg-card); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; width:100%; max-width:200px; color:white; font-size:0.8rem;">
        <span style="font-size:0.75rem; color:var(--text-secondary);"><span id="qso-count">0</span> Contacts Logged</span>
      </div>

      <!-- LOG TABLE -->
      <div style="overflow-x:auto; border:1px solid var(--border-color); border-radius:6px; background:var(--bg-card);">
        <table style="width:100%; border-collapse:collapse; text-align:left; font-family:'JetBrains Mono', monospace; font-size:0.8rem;">
          <thead>
            <tr style="border-bottom:1px solid var(--border-color); background:var(--bg-panel); color:var(--text-secondary);">
              <th style="padding:0.6rem 0.8rem;">Date/Time</th>
              <th style="padding:0.6rem 0.8rem;">Callsign</th>
              <th style="padding:0.6rem 0.8rem;">Band</th>
              <th style="padding:0.6rem 0.8rem;">Mode</th>
              <th style="padding:0.6rem 0.8rem; text-align:center;">Sent</th>
              <th style="padding:0.6rem 0.8rem; text-align:center;">Rcvd</th>
            </tr>
          </thead>
          <tbody id="qso-table-body">
            <!-- Dynamic rows injected here -->
          </tbody>
        </table>
      </div>
    </div>
  </div>
  ```

- [ ] **Step 2: Add logbook scripting and ADIF generation**
  Preload initial mock data into local storage if it's empty, and define functions to render, filter, add, and export logs.
  Insert this JS inside `<script>` tag:
  ```javascript
  const DEFAULT_QSOS = [
    { timestamp: Date.now() - 36000000, callsign: "K3LR", band: "20m", mode: "FT8", rstSent: "-10", rstRcvd: "-12" },
    { timestamp: Date.now() - 72000000, callsign: "W1AW", band: "40m", mode: "FT8", rstSent: "+02", rstRcvd: "+05" },
    { timestamp: Date.now() - 108000000, callsign: "N3LL", band: "20m", mode: "FT4", rstSent: "-08", rstRcvd: "-10" },
    { timestamp: Date.now() - 144000000, callsign: "W3AA", band: "20m", mode: "FT8", rstSent: "-15", rstRcvd: "-14" }
  ];

  function getQSOs() {
    let qsos = localStorage.getItem('ac3gx_qsos');
    if (!qsos) {
      localStorage.setItem('ac3gx_qsos', JSON.stringify(DEFAULT_QSOS));
      return DEFAULT_QSOS;
    }
    return JSON.parse(qsos);
  }

  function saveQSOs(qsos) {
    localStorage.setItem('ac3gx_qsos', JSON.stringify(qsos));
  }

  function renderQSOs(filterTerm = '') {
    const qsos = getQSOs();
    const tbody = document.getElementById('qso-table-body');
    const term = filterTerm.toUpperCase();
    tbody.innerHTML = '';
    
    let filteredCount = 0;
    
    // Show newest first
    const sorted = [...qsos].sort((a, b) => b.timestamp - a.timestamp);

    sorted.forEach(q => {
      if (term && !q.callsign.toUpperCase().includes(term)) return;
      
      filteredCount++;
      const date = new Date(q.timestamp);
      const dateStr = `${date.getUTCFullYear()}-${String(date.getUTCMonth()+1).padStart(2, '0')}-${String(date.getUTCDate()).padStart(2, '0')} ${String(date.getUTCHours()).padStart(2, '0')}:${String(date.getUTCMinutes()).padStart(2, '0')}`;
      
      const tr = document.createElement('tr');
      tr.style.borderBottom = '1px solid var(--border-color)';
      tr.innerHTML = `
        <td style="padding:0.6rem 0.8rem; color:var(--text-secondary);">${dateStr}</td>
        <td style="padding:0.6rem 0.8rem; font-weight:bold; color:var(--accent-green);">${q.callsign.toUpperCase()}</td>
        <td style="padding:0.6rem 0.8rem;">${q.band}</td>
        <td style="padding:0.6rem 0.8rem;">${q.mode}</td>
        <td style="padding:0.6rem 0.8rem; text-align:center; color:var(--accent-cyan);">${q.rstSent || '59'}</td>
        <td style="padding:0.6rem 0.8rem; text-align:center; color:var(--accent-cyan);">${q.rstRcvd || '59'}</td>
      `;
      tbody.appendChild(tr);
    });

    document.getElementById('qso-count').textContent = qsos.length;
  }

  function addQSO(e) {
    e.preventDefault();
    if (!powerOn) return;
    const callInput = document.getElementById('qso-call');
    const bandInput = document.getElementById('qso-band');
    const modeInput = document.getElementById('qso-mode');
    const rstSentInput = document.getElementById('qso-rst-sent');
    const rstRcvdInput = document.getElementById('qso-rst-rcvd');
    
    const newQSO = {
      timestamp: Date.now(),
      callsign: callInput.value.trim().toUpperCase(),
      band: bandInput.value,
      mode: modeInput.value,
      rstSent: rstSentInput.value.trim() || '+00',
      rstRcvd: rstRcvdInput.value.trim() || '+00'
    };

    const qsos = getQSOs();
    qsos.push(newQSO);
    saveQSOs(qsos);
    renderQSOs(document.getElementById('log-search').value);
    
    // Reset Form
    callInput.value = '';
    rstSentInput.value = '';
    rstRcvdInput.value = '';
  }

  function filterLogs() {
    renderQSOs(document.getElementById('log-search').value);
  }

  function exportADIF() {
    const qsos = getQSOs();
    let adif = `ADIF Export from AC3GX Station Console\n<ADIF_VER:5>3.1.4\n<PROGRAMID:13>AC3GX_CONSOLE\n<EOH>\n`;
    
    qsos.forEach(q => {
      const d = new Date(q.timestamp);
      const dateStr = `${d.getUTCFullYear()}${String(d.getUTCMonth()+1).padStart(2, '0')}${String(d.getUTCDate()).padStart(2, '0')}`;
      const timeStr = `${String(d.getUTCHours()).padStart(2, '0')}${String(d.getUTCMinutes()).padStart(2, '0')}00`;
      
      const call = q.callsign.toUpperCase();
      const band = q.band.toUpperCase();
      const mode = q.mode.toUpperCase();
      const sent = q.rstSent || '59';
      const rcvd = q.rstRcvd || '59';
      
      adif += `<CALL:${call.length}>${call} <QSO_DATE:8>${dateStr} <TIME_ON:6>${timeStr} <BAND:${band.length}>${band} <MODE:${mode.length}>${mode} <RST_SENT:${sent.length}>${sent} <RST_RCVD:${rcvd.length}>${rcvd} <EOR>\n`;
    });

    const blob = new Blob([adif], { type: 'text/plain;charset=utf-8' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `AC3GX_Logbook.adi`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  // Render on initial load
  renderQSOs();
  ```

- [ ] **Step 3: Commit logbook components**
  ```bash
  git add index.html
  git commit -m "feat: add interactive FT8 logbook with LocalStorage persistence and ADIF file exporter"
  ```

---

### Task 5: Morse Code Keyer Widget

**Files:**
- Modify: `index.html`
- Test: Verify Spacebar and click generate 700Hz tone and translate inputs to visual text.

**Interfaces:**
- Consumes: Placeholder container in `index.html`.
- Produces: Audio synthesis pipeline, Dit/Dah decoder, visual translation box.

- [ ] **Step 1: Replace Morse Keyer Placeholder with Visual Panel in index.html**
  Locate `<div id="morse-keyer-container"></div>` and replace it with the visual keyer card.
  Target replacement code:
  ```html
  <!-- MORSE CODE KEYER CARD -->
  <div class="console-card">
    <div class="card-header">
      <span>CW Morse Keyer</span>
      <span style="font-size:0.7rem; color:var(--accent-amber);">700 Hz sidetone</span>
    </div>
    <div style="display:flex; flex-direction:column; gap:0.75rem;">
      <div style="background:var(--bg-card); padding:0.6rem; border-radius:6px; border:1px solid var(--border-color); min-height:60px; display:flex; flex-direction:column; gap:0.25rem;">
        <span style="font-size:0.65rem; color:var(--text-secondary); text-transform:uppercase;">Decoded Text</span>
        <div id="morse-output" style="font-family:'JetBrains Mono', monospace; font-size:1.1rem; color:var(--accent-green); word-break:break-all; min-height:24px;"></div>
      </div>
      <div style="display:flex; justify-content:space-between; align-items:center; font-size:0.8rem;">
        <div>
          <label style="color:var(--text-secondary); font-size:0.7rem;">WPM Speed</label>
          <input type="range" id="wpm-slider" min="10" max="30" value="18" style="vertical-align:middle; width:80px; margin-left:0.4rem;">
          <span id="wpm-label" style="font-family:'JetBrains Mono', monospace; margin-left:0.25rem;">18</span>
        </div>
        <button onclick="clearMorseText()" style="background:transparent; border:1px solid var(--border-color); color:var(--text-secondary); padding:0.2rem 0.5rem; border-radius:4px; font-size:0.7rem; cursor:pointer;">Clear</button>
      </div>
      <!-- KEY BUTTON -->
      <button id="morse-key-btn" style="background:var(--bg-card); border:2px solid var(--border-color); height:80px; border-radius:8px; cursor:pointer; color:var(--text-secondary); font-weight:bold; font-size:0.9rem; transition:all 0.1s ease; user-select:none;">
        PRESS SPACEBAR / HOLD CLICK
      </button>
    </div>
  </div>
  ```

- [ ] **Step 2: Add AudioContext Synthesizer and Decoder logic**
  Insert Morse processing script into `<script>`.
  Insert this JS inside `<script>` tag:
  ```javascript
  let audioCtx = null;
  let oscillator = null;
  let gainNode = null;
  let keyIsDown = false;
  let keyDownTime = 0;
  let keyUpTime = 0;
  let currentWord = '';
  let currentLetterSignal = '';
  let morseTimer = null;

  const MORSE_CODE = {
    '.-': 'A', '-...': 'B', '-.-.': 'C', '-..': 'D', '.': 'E',
    '..-.': 'F', '--.': 'G', '....': 'H', '..': 'I', '.---': 'J',
    '-.-': 'K', '.-..': 'L', '--': 'M', '-.': 'N', '---': 'O',
    '.--.': 'P', '--.-': 'Q', '.-.': 'R', '...': 'S', '-': 'T',
    '..-': 'U', '...-': 'V', '.--': 'W', '-..-': 'X', '-.--': 'Y',
    '--..': 'Z', '-----': '0', '.----': '1', '..---': '2', '...--': '3',
    '....-': '4', '.....': '5', '-....': '6', '--...': '7', '---..': '8',
    '----.': '9', '.-.-.-': '.', '--..--': ',', '..--..': '?', '.----.': "'",
    '-..-.': '/', '-...-': '='
  };

  const wpmSlider = document.getElementById('wpm-slider');
  const wpmLabel = document.getElementById('wpm-label');
  wpmSlider.oninput = () => wpmLabel.textContent = wpmSlider.value;

  function initAudio() {
    if (audioCtx) return;
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }

  function startTone() {
    if (!powerOn) return;
    initAudio();
    if (audioCtx.state === 'suspended') {
      audioCtx.resume();
    }
    
    // Create synthesizer nodes
    oscillator = audioCtx.createOscillator();
    gainNode = audioCtx.createGain();
    
    oscillator.type = 'sine';
    oscillator.frequency.value = 700; // standard CW sidetone frequency
    
    gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
    gainNode.gain.linearRampToValueAtTime(0.15, audioCtx.currentTime + 0.005); // anti-click envelope
    
    oscillator.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    
    oscillator.start();
  }

  function stopTone() {
    if (!oscillator) return;
    const releaseTime = audioCtx.currentTime + 0.005;
    gainNode.gain.linearRampToValueAtTime(0, releaseTime);
    oscillator.stop(releaseTime);
    oscillator = null;
  }

  function handleKeyDown() {
    if (keyIsDown || !powerOn) return;
    keyIsDown = true;
    startTone();
    keyDownTime = performance.now();
    
    document.getElementById('morse-key-btn').style.borderColor = 'var(--accent-amber)';
    document.getElementById('morse-key-btn').style.background = 'var(--accent-glow-amber)';
    
    if (morseTimer) clearTimeout(morseTimer);
  }

  function handleKeyUp() {
    if (!keyIsDown) return;
    keyIsDown = false;
    stopTone();
    keyUpTime = performance.now();
    
    document.getElementById('morse-key-btn').style.borderColor = 'var(--border-color)';
    document.getElementById('morse-key-btn').style.background = 'var(--bg-card)';
    
    const duration = keyUpTime - keyDownTime;
    const wpm = parseInt(wpmSlider.value);
    const dotLength = 1200 / wpm; // dot duration formula
    
    // Distinguish dots/dahs
    if (duration < dotLength * 1.5) {
      currentLetterSignal += '.';
    } else {
      currentLetterSignal += '-';
    }
    
    // Wait for pause to process character
    morseTimer = setTimeout(() => {
      decodeCharacter();
    }, dotLength * 3.5);
  }

  function decodeCharacter() {
    if (!currentLetterSignal) return;
    const char = MORSE_CODE[currentLetterSignal] || '?';
    document.getElementById('morse-output').textContent += char;
    currentLetterSignal = '';
    
    // Set a timer for space separator
    const wpm = parseInt(wpmSlider.value);
    const dotLength = 1200 / wpm;
    morseTimer = setTimeout(() => {
      document.getElementById('morse-output').textContent += ' ';
    }, dotLength * 4);
  }

  function clearMorseText() {
    document.getElementById('morse-output').textContent = '';
    currentLetterSignal = '';
  }

  // Hook up event listeners
  const keyBtn = document.getElementById('morse-key-btn');
  keyBtn.addEventListener('mousedown', (e) => { e.preventDefault(); handleKeyDown(); });
  keyBtn.addEventListener('mouseup', handleKeyUp);
  keyBtn.addEventListener('mouseleave', handleKeyUp);
  keyBtn.addEventListener('touchstart', (e) => { e.preventDefault(); handleKeyDown(); });
  keyBtn.addEventListener('touchend', handleKeyUp);

  window.addEventListener('keydown', (e) => {
    if (e.code === 'Space') {
      if (document.activeElement.tagName !== 'INPUT') {
        e.preventDefault();
        handleKeyDown();
      }
    }
  });
  window.addEventListener('keyup', (e) => {
    if (e.code === 'Space') {
      handleKeyUp();
    }
  });
  ```

- [ ] **Step 3: Commit CW Morse components**
  ```bash
  git add index.html
  git commit -m "feat: add visual Morse code sidetone synthesizer with real-time text decoding"
  ```

---

### Task 5.5: Clean up Brainstorming Directory

**Files:**
- Delete: `.superpowers/` files or wait for final stop.
- Modify: `.gitignore` to verify it's active.

- [ ] **Step 1: Verify .gitignore contains .superpowers/**
  Verify `c:\Users\admin\Documents\scratchpad\.gitignore` is correctly committed and includes `.superpowers/`.

---

### Task 6: Final Responsive CSS Polish & Verification

**Files:**
- Modify: `index.html`
- Test: Open file in a web browser using browser subagent and verify layouts wrap cleanly, features function, and no console errors.

- [ ] **Step 1: Apply final design touches to CSS in index.html**
  Confirm fonts render correctly, grids wrap well on small screens, and add interactive focus transitions.
  Add responsive layout override at the bottom of the style sheet:
  ```css
  /* Responsive custom adjustments */
  @media (max-width: 768px) {
    header {
      flex-direction: column;
      align-items: stretch;
      text-align: center;
    }
    .logo-section { justify-content: center; }
    .clocks-section { justify-content: center; }
    .status-section { justify-content: center; }
  }
  ```

- [ ] **Step 2: Start a validation subagent to open index.html and verify UI look & feel**
  Verify visual alignment, clock tick, form logs, ADIF triggers, and audio sidetone works.
  Commit changes and finalize development branch.
  
  ```bash
  git add index.html
  git commit -m "style: final CSS responsive design polish and desktop dashboard layout tuning"
  ```
