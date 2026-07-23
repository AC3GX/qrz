# Live MUF & Grayline Propagation Map Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Embed a self-contained, live-updating equirectangular HTML5 canvas map display showing solar grayline day/night boundaries and a thermal Maximum Usable Frequency (MUF) propagation heatmap derived from live NOAA space telemetry.

**Architecture:** Inject a new card panel containing an HTML5 canvas inside the console's right-hand weather column. The script will render vector continent polygons offline, calculate solar zenith parameters for daylight borders, compute MUF values relative to NOAA's SFI and K-index values, and layer them onto the canvas using translucent screen overlays.

**Tech Stack:** HTML5 Canvas, Vanilla CSS3, Native ES6 JavaScript.

## Global Constraints
- **Self-contained offline-first architecture:** Rely exclusively on JavaScript coordinates for mapping continents; zero external map graphics or asset downloads.
- **NOAA Integration:** Read active SFI and K-index telemetry directly from services.swpc.noaa.gov.
- **No dependencies:** Do not introduce external libraries or node modules.

---

### Task 1: Add HTML Card Structure for Propagation Map

**Files:**
- Modify: `index.html` (after Solar Conditions card)

**Interfaces:**
- Produces: DOM node `#prop-map-canvas` accessible to JS drawing contexts.

- [ ] **Step 1: Write HTML markup for the Map console card**

  Insert the map card directly underneath the `solar-card` container (approx. lines 712-713):
  ```html
        <!-- LIVE PROPAGATION MAP CARD -->
        <div class="console-card" id="map-card" style="display:flex; flex-direction:column; gap:0.5rem;">
          <div class="card-header">
            <span>Propagation Map</span>
            <span style="font-size:0.65rem; color:var(--accent-cyan);">GRAYLINE & MUF</span>
          </div>
          <div style="position:relative; width:100%; aspect-ratio:2/1; background:#050508; border-radius:6px; border:1px solid var(--border-color); overflow:hidden;">
            <canvas id="prop-map-canvas" style="width:100%; height:100%; display:block;"></canvas>
          </div>
          <div style="display:flex; justify-content:space-between; align-items:center; font-size:0.75rem; color:var(--text-secondary); padding:0 0.1rem; flex-wrap:wrap; gap:0.25rem;">
            <div style="display:flex; gap:0.35rem; align-items:center;">
              <span style="display:inline-block; width:8px; height:8px; background:rgba(142,68,173,0.5); border-radius:50%;"></span> <span>&lt;10M</span>
              <span style="display:inline-block; width:8px; height:8px; background:rgba(46,204,113,0.5); border-radius:50%;"></span> <span>14M</span>
              <span style="display:inline-block; width:8px; height:8px; background:rgba(241,196,15,0.5); border-radius:50%;"></span> <span>21M</span>
              <span style="display:inline-block; width:8px; height:8px; background:rgba(231,76,60,0.5); border-radius:50%;"></span> <span>&gt;28M</span>
            </div>
            <span style="font-size:0.65rem; color:var(--text-secondary);">BLN: PA (FN20)</span>
          </div>
        </div>
  ```

- [ ] **Step 2: Run validation check**

  Open `index.html` in a web browser.
  Expected: A new "Propagation Map" card is displayed below the "Solar Conditions" card with a blank dark rectangle of 2:1 aspect ratio.

- [ ] **Step 3: Commit HTML markup**
  ```bash
  git add index.html
  git commit -m "feat: add HTML card structure for MUF propagation map canvas"
  ```

---

### Task 2: Vector Continents Outline Drawing

**Files:**
- Modify: `index.html` (inside `<script>` block)

**Interfaces:**
- Produces: `drawBaseMap(ctx, width, height)` and coordinate constants `MAP_CONTINENTS`.

- [ ] **Step 1: Declare Continent Vector Arrays in Script**

  Add the normalized continent coordinates at the bottom of the script block:
  ```javascript
      // Vector Longitude & Latitude Polygons for Continent Boundaries
      const MAP_CONTINENTS = [
        // North America
        [[-168,65],[-120,69],[-80,70],[-60,60],[-55,48],[-80,25],[-99,15],[-105,20],[-115,30],[-125,48],[-160,58]],
        // South America
        [[-80,10],[-50,-5],[-35,-7],[-40,-20],[-65,-45],[-75,-55],[-70,-35],[-75,-20],[-81,-5]],
        // Eurasia
        [[-5,36],[20,40],[30,60],[60,70],[100,75],[140,72],[170,66],[140,35],[120,38],[108,18],[98,10],[77,8],[60,25],[45,15],[35,12],[30,30],[10,37]],
        // Africa
        [[-17,32],[32,31],[34,27],[51,11],[46,-10],[40,-25],[33,-34],[18,-34],[10,-5],[-14,14]],
        // Australia
        [[113,-22],[135,-12],[142,-10],[153,-28],[148,-38],[115,-34],[113,-26]],
        // Greenland
        [[-50,60],[-40,60],[-30,70],[-40,80],[-60,80],[-55,70]]
      ];
  ```

- [ ] **Step 2: Add drawBaseMap drawing function**

  Write a helper to translate lat/lon points to canvas `x, y` and render polygons:
  ```javascript
      function drawBaseMap(ctx, width, height) {
        ctx.fillStyle = '#0b0c10';
        ctx.fillRect(0, 0, width, height);

        ctx.fillStyle = '#161920';
        ctx.strokeStyle = '#2c3531';
        ctx.lineWidth = 1;

        MAP_CONTINENTS.forEach(polygon => {
          ctx.beginPath();
          polygon.forEach((pt, i) => {
            const px = ((pt[0] + 180) / 360) * width;
            const py = ((90 - pt[1]) / 180) * height;
            if (i === 0) ctx.moveTo(px, py);
            else ctx.lineTo(px, py);
          });
          ctx.closePath();
          ctx.fill();
          ctx.stroke();
        });
      }
  ```

- [ ] **Step 3: Verify execution**

  Add a test render line in JavaScript, reload the browser, and check if vector continent contours render correctly on the dark canvas card.

- [ ] **Step 4: Commit**
  ```bash
  git add index.html
  git commit -m "feat: implement vector continent polygon rendering on propagation map canvas"
  ```

---

### Task 3: Solar Astronomy Calculations & Night Terminator

**Files:**
- Modify: `index.html` (inside `<script>` block)

**Interfaces:**
- Produces: `drawDayNightShadow(ctx, width, height, declination, subsolarLon)`

- [ ] **Step 1: Write Solar Declination and Zenith calculation helpers**

  Add astronomical helpers to the script block:
  ```javascript
      function getSolarDeclination(dayOfYear) {
        // Approximate solar declination angle in radians
        return 0.409 * Math.sin((2 * Math.PI / 365) * (dayOfYear - 80));
      }

      function getSolarZenithCos(lat, lon, declination, subsolarLon) {
        const phi = lat * Math.PI / 180;
        const delta = declination;
        const lambda = lon * Math.PI / 180;
        const lambda0 = subsolarLon * Math.PI / 180;
        
        // Return cosine of the solar zenith angle
        return Math.sin(phi) * Math.sin(delta) + Math.cos(phi) * Math.cos(delta) * Math.cos(lambda - lambda0);
      }
  ```

- [ ] **Step 2: Write drawDayNightShadow overlay function**

  Iterate latitude/longitude coordinates to apply a translucent dark shade over the night hemisphere ($\cos(\chi) < 0$):
  ```javascript
      function drawDayNightShadow(ctx, width, height, declination, subsolarLon) {
        const gridW = 4; // grid resolution in pixels
        const gridH = 4;
        
        ctx.fillStyle = 'rgba(0, 0, 0, 0.45)';
        for (let y = 0; y < height; y += gridH) {
          const lat = 90 - (y / height) * 180;
          for (let x = 0; x < width; x += gridW) {
            const lon = (x / width) * 360 - 180;
            const cosZ = getSolarZenithCos(lat, lon, declination, subsolarLon);
            if (cosZ < 0) {
              ctx.fillRect(x, y, gridW, gridH);
            }
          }
        }
      }
  ```

- [ ] **Step 3: Verify execution**

  Call the functions using a test time-stamp to ensure the night shadow is drawn correctly overlaying the continents.

- [ ] **Step 4: Commit**
  ```bash
  git add index.html
  git commit -m "feat: implement solar zenith astronomy math and day/night shadow overlay"
  ```

---

### Task 4: Dynamic MUF Heatmap Formulation

**Files:**
- Modify: `index.html` (inside `<script>` block)

**Interfaces:**
- Consumes: SFI and K-index telemetry from `fetchLiveSolarData`.
- Produces: `drawMUFHeatmap(ctx, width, height, sfi, k, declination, subsolarLon)`

- [ ] **Step 1: Write drawMUFHeatmap function**

  Write a grid scanner that computes local MUF and overlays semi-transparent thermal colors onto the canvas:
  ```javascript
      function drawMUFHeatmap(ctx, width, height, sfi, k, declination, subsolarLon) {
        const stepX = 5;
        const stepY = 5;

        // Base peak and floor frequencies
        const mufDay = 10 + (sfi * 0.18);
        const mufNight = 6 + (sfi * 0.03);
        
        // Geomagnetic storm penalty scaler
        let stormPenalty = 1.0;
        if (k >= 4) {
          stormPenalty = Math.max(0.6, 1.0 - 0.08 * (k - 3));
        }

        ctx.save();
        ctx.globalCompositeOperation = 'screen';

        for (let y = 0; y < height; y += stepY) {
          const lat = 90 - (y / height) * 180;
          for (let x = 0; x < width; x += stepX) {
            const lon = (x / width) * 360 - 180;
            const cosZ = getSolarZenithCos(lat, lon, declination, subsolarLon);
            
            // Approximate zenith cosine modulation
            const zenithFactor = Math.max(0, cosZ);
            let muf = mufNight + (mufDay - mufNight) * Math.sqrt(zenithFactor);
            muf *= stormPenalty;

            // Define color bounds based on calculated frequency
            let color = 'rgba(142, 68, 173, 0.06)'; // Violet (<10 MHz)
            if (muf >= 28) {
              color = 'rgba(231, 76, 60, 0.22)';  // Red (>28 MHz)
            } else if (muf >= 21) {
              color = 'rgba(241, 196, 15, 0.18)'; // Amber (21 MHz)
            } else if (muf >= 14) {
              color = 'rgba(46, 204, 113, 0.14)';  // Green (14 MHz)
            }

            ctx.fillStyle = color;
            ctx.fillRect(x, y, stepX, stepY);
          }
        }
        ctx.restore();
      }
  ```

- [ ] **Step 2: Verify execution**

  Run node tests or mock evaluations to confirm SFI changes alter color distributions.

- [ ] **Step 3: Commit**
  ```bash
  git add index.html
  git commit -m "feat: implement thermal MUF propagation heatmap calculations on canvas"
  ```

---

### Task 5: Bind Telemetry Updates & Plot Pennsylvania Blinker

**Files:**
- Modify: `index.html` (inside `fetchLiveSolarData` and canvas rendering routines)

**Interfaces:**
- Consumes: SFI, K-index, and power status.
- Produces: Live redraw loop `renderPropagationMap(sfi, k)` linked directly to NOAA telemetry events.

- [ ] **Step 1: Write renderPropagationMap coordinator**

  Write a function to orchestrate drawing the base, the MUF heatmap, the day/night shadow, and the operator dot:
  ```javascript
      function renderPropagationMap(sfi, k) {
        const canvas = document.getElementById('prop-map-canvas');
        if (!canvas) return;
        
        const ctx = canvas.getContext('2d');
        const w = canvas.width = canvas.offsetWidth || 300;
        const h = canvas.height = canvas.offsetHeight || 150;

        // 1. Draw vectors
        drawBaseMap(ctx, w, h);

        // 2. Solar calculations
        const now = new Date();
        // Day of Year
        const start = new Date(now.getFullYear(), 0, 0);
        const diff = now - start;
        const oneDay = 1000 * 60 * 60 * 24;
        const dayOfYear = Math.floor(diff / oneDay);
        
        const declination = getSolarDeclination(dayOfYear);
        // Subsolar longitude in degrees
        const subsolarLon = -15 * (now.getUTCHours() + now.getUTCMinutes() / 60 + now.getUTCSeconds() / 3600);

        // 3. Draw MUF Heatmap
        drawMUFHeatmap(ctx, w, h, sfi, k, declination, subsolarLon);

        // 4. Draw Day/Night Shadow
        drawDayNightShadow(ctx, w, h, declination, subsolarLon);

        // 5. Draw PA Blinker (FN20: -75.3 Longitude, 40.0 Latitude)
        const paX = ((-75.3 + 180) / 360) * w;
        const paY = ((90 - 40.0) / 180) * h;

        ctx.beginPath();
        ctx.arc(paX, paY, 4, 0, 2 * Math.PI);
        ctx.fillStyle = 'var(--accent-amber)';
        ctx.shadowColor = 'var(--accent-amber)';
        ctx.shadowBlur = 8;
        ctx.fill();
        ctx.shadowBlur = 0; // reset
      }
  ```

- [ ] **Step 2: Bind to fetchLiveSolarData update events**

  Modify the bottom of `fetchLiveSolarData()` to trigger a map update with the retrieved values:
  ```javascript
      // ... bottom of fetchLiveSolarData()
      updateHFBands(sfiVal, kVal);
      renderPropagationMap(sfiVal, kVal);
  ```

- [ ] **Step 3: Run full manual integration validation**

  Open `index.html` in browser. Click power switch. Check that the Propagation Map renders cleanly, displays the day/night terminator correctly relative to current UTC time, overlays the colored MUF heatmap, and shows your Pennsylvania station location.

- [ ] **Step 4: Commit and Push**
  ```bash
  git add index.html
  git commit -m "feat: bind map render coordinates to live NOAA telemetry updates and plot station blinker"
  git push
  ```
