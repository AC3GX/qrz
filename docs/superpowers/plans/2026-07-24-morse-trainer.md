# Morse Code Trainer & Text-to-CW Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend the Morse Code Keyer card to act as an interactive Trainer (quiz game) and support Text-to-CW translations (converting typed characters into scheduled audio sidetones).

**Architecture:** Redesign the Morse card's body to support three operational sub-modes toggled by a mode-switch header. Embed audio queuing loops to schedule Web Audio API oscillators at precise WPM intervals for text translation, and implement character comparisons in the keyer's character decoder to manage quiz scores.

**Tech Stack:** Web Audio API, Vanilla HTML5 / CSS3, Native JavaScript.

## Global Constraints
- **Self-contained:** No external sound samples or libraries.
- **No dependencies:** Do not add Node modules.
- **Stable UI:** Maintain card dimensions and respect existing custom scrollbar setups.

---

### Task 1: Add Mode Toggles and UI controls to Morse Card

**Files:**
- Modify: `index.html` (inside the Morse card body, lines 804-833)

**Interfaces:**
- Produces: HTML mode selectors (`#morse-mode-select`), Text inputs (`#cw-text-input`), and Trainer prompts (`#trainer-prompt-panel`).

- [ ] **Step 1: Write HTML markup for Morse training panel controls**

  Replace the content of `#morse-card` inside `index.html` (lines 809-832) with a mode switcher and panels:
  ```html
        <div style="display:flex; flex-direction:column; gap:0.5rem;">
          <!-- MODE SWITCHER -->
          <div style="display:flex; border:1px solid var(--border-color); border-radius:4px; overflow:hidden; background:var(--bg-primary);">
            <button onclick="switchMorseMode('keyer')" id="m-btn-keyer" class="tab-btn active" style="flex:1; padding:0.3rem; font-size:0.7rem; font-weight:600; border:none; cursor:pointer; background:transparent;">KEYER</button>
            <button onclick="switchMorseMode('trainer')" id="m-btn-trainer" class="tab-btn" style="flex:1; padding:0.3rem; font-size:0.7rem; font-weight:600; border:none; cursor:pointer; background:transparent; border-left:1px solid var(--border-color); border-right:1px solid var(--border-color);">TRAINER</button>
            <button onclick="switchMorseMode('playback')" id="m-btn-playback" class="tab-btn" style="flex:1; padding:0.3rem; font-size:0.7rem; font-weight:600; border:none; cursor:pointer; background:transparent;">TEXT-TO-CW</button>
          </div>

          <!-- 1. KEYER PANEL -->
          <div id="morse-panel-keyer" class="morse-sub-panel active" style="display:flex; flex-direction:column; gap:0.5rem;">
            <div style="background:var(--bg-card); padding:0.6rem; border-radius:6px; border:1px solid var(--border-color); min-height:60px; display:flex; flex-direction:column; gap:0.25rem;">
              <span style="font-size:0.65rem; color:var(--text-secondary); text-transform:uppercase;">Decoded Text</span>
              <div id="morse-output" style="font-family:'JetBrains Mono', monospace; font-size:1.1rem; color:var(--accent-green); word-break:break-all; min-height:24px;"></div>
            </div>
          </div>

          <!-- 2. TRAINER PANEL -->
          <div id="morse-panel-trainer" class="morse-sub-panel" style="display:none; flex-direction:column; gap:0.5rem;">
            <div style="background:var(--bg-card); padding:0.6rem; border-radius:6px; border:1px solid var(--border-color); min-height:60px; display:flex; flex-direction:column; justify-content:space-between; align-items:center; gap:0.25rem;">
              <div style="display:flex; justify-content:space-between; width:100%; font-size:0.65rem; color:var(--text-secondary);">
                <span>TARGET LETTER</span>
                <span id="trainer-score">SCORE: 0/0</span>
              </div>
              <div style="display:flex; align-items:baseline; gap:0.5rem;">
                <span id="trainer-target" style="font-size:1.8rem; font-weight:bold; color:var(--accent-cyan); font-family:'JetBrains Mono', monospace;">A</span>
                <span id="trainer-hint" style="font-size:0.9rem; color:var(--text-secondary); font-family:'JetBrains Mono', monospace;">(.-)</span>
              </div>
            </div>
          </div>

          <!-- 3. PLAYBACK PANEL -->
          <div id="morse-panel-playback" class="morse-sub-panel" style="display:none; flex-direction:column; gap:0.5rem;">
            <div style="display:flex; gap:0.4rem;">
              <input type="text" id="cw-text-input" placeholder="Type text to play..." style="flex:1; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-family:'Outfit', sans-serif; font-size:0.8rem; outline:none;">
              <button onclick="playCWSidetone()" style="background:transparent; border:1px solid var(--accent-cyan); color:var(--accent-cyan); font-weight:600; padding:0 0.6rem; border-radius:4px; font-size:0.75rem; cursor:pointer;">PLAY</button>
              <button onclick="stopCWSidetone()" style="background:transparent; border:1px solid var(--border-color); color:#ff3b30; font-weight:600; padding:0 0.6rem; border-radius:4px; font-size:0.75rem; cursor:pointer;">STOP</button>
            </div>
            <div style="background:var(--bg-card); padding:0.6rem; border-radius:6px; border:1px solid var(--border-color); min-height:40px; display:flex; flex-direction:column; gap:0.25rem;">
              <span style="font-size:0.65rem; color:var(--text-secondary); text-transform:uppercase;">Playing character</span>
              <div id="cw-playback-highlight" style="font-family:'JetBrains Mono', monospace; font-size:1.1rem; color:var(--accent-amber); min-height:20px;">-</div>
            </div>
          </div>

          <!-- COMMON CONTROLS -->
          <div style="display:flex; justify-content:space-between; align-items:center; font-size:0.8rem;">
            <div>
              <label style="color:var(--text-secondary); font-size:0.7rem;">WPM Speed</label>
              <input type="range" id="wpm-slider" min="10" max="30" value="18" style="vertical-align:middle; width:80px; margin-left:0.4rem;">
              <span id="wpm-label" style="font-family:'JetBrains Mono', monospace; margin-left:0.25rem;">18</span>
            </div>
            <button onclick="clearMorseText()" style="background:transparent; border:1px solid var(--border-color); color:var(--text-secondary); padding:0.2rem 0.5rem; border-radius:4px; font-size:0.7rem; cursor:pointer; font-family:'Outfit', sans-serif;">Clear</button>
          </div>

          <!-- KEY BUTTON -->
          <button id="morse-key-btn" style="background:var(--bg-card); border:2px solid var(--border-color); height:80px; border-radius:8px; cursor:pointer; color:var(--text-secondary); font-weight:bold; font-size:0.9rem; transition:all 0.1s ease; user-select:none; font-family:'Outfit', sans-serif; outline:none;">
            PRESS SPACEBAR / HOLD CLICK
          </button>
        </div>
  ```

- [ ] **Step 2: Add switchMorseMode function to script**

  Insert the mode-switching helper function at the bottom of the script:
  ```javascript
      function switchMorseMode(mode) {
        document.querySelectorAll('.morse-sub-panel').forEach(p => p.style.display = 'none');
        document.getElementById(`morse-panel-${mode}`).style.display = 'flex';
        
        const mSwitcher = document.getElementById('m-btn-keyer').parentNode;
        mSwitcher.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
        document.getElementById(`m-btn-${mode}`).classList.add('active');

        // Reset states
        stopCWSidetone();
        if (mode === 'trainer') {
          initTrainer();
        }
      }
  ```

- [ ] **Step 3: Verify markup and visual tabs**

  Reload page and click through KEYER, TRAINER, and TEXT-TO-CW tabs to confirm smooth view switching.

- [ ] **Step 4: Commit UI scaffolding**
  ```bash
  git add index.html
  git commit -m "feat: implement HTML sub-panels and mode-switcher for Morse Trainer"
  ```

---

### Task 2: Implement Text-to-CW Playback

**Files:**
- Modify: `index.html` (inside `<script>` block)

**Interfaces:**
- Produces: `playCWSidetone()`, `stopCWSidetone()`, `playMorseSequence(words)`

- [ ] **Step 1: Write playCWSidetone and playMorseSequence queue logic**

  Add timing calculations and play functions:
  ```javascript
      let playbackTimeouts = [];

      function stopCWSidetone() {
        playbackTimeouts.forEach(t => clearTimeout(t));
        playbackTimeouts = [];
        stopTone();
        document.getElementById('cw-playback-highlight').textContent = '-';
      }

      function playCWSidetone() {
        stopCWSidetone();
        const text = document.getElementById('cw-text-input').value.toUpperCase().trim();
        if (!text) return;
        
        const wpm = parseInt(wpmSlider.value);
        const dotLength = 1200 / wpm; // ms
        
        // Reverse lookup mapping for alphanumeric characters to dot-dash codes
        const REVERSE_MORSE = {};
        for (const [code, char] of Object.entries(MORSE_CODE)) {
          REVERSE_MORSE[char] = code;
        }

        let timeOffset = 0;
        
        for (let i = 0; i < text.length; i++) {
          const char = text[i];
          if (char === ' ') {
            // Word boundary: 7 dots total gap. (already has 3 dots gap from last char, add 4 more)
            timeOffset += dotLength * 4;
            continue;
          }
          
          const code = REVERSE_MORSE[char];
          if (!code) continue;

          // Schedule character highlight
          const highlightTimeout = setTimeout(() => {
            document.getElementById('cw-playback-highlight').textContent = `[${char}]   ${code}`;
          }, timeOffset);
          playbackTimeouts.push(highlightTimeout);

          for (let j = 0; j < code.length; j++) {
            const sym = code[j];
            const symDuration = sym === '.' ? dotLength : dotLength * 3;

            // Schedule tone start
            const startTimeout = setTimeout(() => {
              startTone();
            }, timeOffset);
            playbackTimeouts.push(startTimeout);

            timeOffset += symDuration;

            // Schedule tone stop
            const stopTimeout = setTimeout(() => {
              stopTone();
            }, timeOffset);
            playbackTimeouts.push(stopTimeout);

            // Inter-symbol gap: 1 dot length
            timeOffset += dotLength;
          }
          
          // Inter-character gap: 3 dots total (already has 1 dot from last symbol, add 2 more)
          timeOffset += dotLength * 2;
        }

        // Done playing reset
        const doneTimeout = setTimeout(() => {
          document.getElementById('cw-playback-highlight').textContent = 'DONE';
        }, timeOffset);
        playbackTimeouts.push(doneTimeout);
      }
  ```

- [ ] **Step 2: Test translation output**

  Input "CQ" in the Text-to-CW input field, click PLAY. Verify sidetones play correctly matching `-.--. -..-` and highlight the target characters.

- [ ] **Step 3: Commit**
  ```bash
  git add index.html
  git commit -m "feat: implement Text-to-CW translation playback queue scheduler"
  ```

---

### Task 3: Interactive Morse Code Trainer (Quiz Mode)

**Files:**
- Modify: `index.html` (inside `<script>` block)

**Interfaces:**
- Produces: `initTrainer()`, updates `decodeCharacter()` behavior to check answers.

- [ ] **Step 1: Write Trainer state variables and quiz generator**

  ```javascript
      let trainerState = {
        target: '',
        correct: 0,
        total: 0
      };

      function initTrainer() {
        const letters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
        const randChar = letters[Math.floor(Math.random() * letters.length)];
        trainerState.target = randChar;
        
        // Find corresponding code
        let targetCode = '';
        for (const [code, char] of Object.entries(MORSE_CODE)) {
          if (char === randChar) {
            targetCode = code;
            break;
          }
        }

        document.getElementById('trainer-target').textContent = randChar;
        document.getElementById('trainer-hint').textContent = `(${targetCode})`;
        document.getElementById('trainer-score').textContent = `SCORE: ${trainerState.correct}/${trainerState.total}`;
      }
  ```

- [ ] **Step 2: Update decodeCharacter check function**

  Modify `decodeCharacter` (approx. line 1170) to perform interactive scoring if training mode is active:
  ```javascript
      function decodeCharacter() {
        if (!currentLetterSignal) return;
        const char = MORSE_CODE[currentLetterSignal] || '?';
        currentLetterSignal = '';

        const activePanel = document.getElementById('morse-panel-trainer');
        const isTrainerMode = activePanel && activePanel.style.display === 'flex';

        if (isTrainerMode) {
          trainerState.total++;
          const keyBtn = document.getElementById('morse-key-btn');
          
          if (char === trainerState.target) {
            trainerState.correct++;
            // Flash Green
            keyBtn.style.borderColor = 'var(--accent-green)';
            keyBtn.style.background = 'rgba(46,204,113,0.2)';
            setTimeout(() => {
              keyBtn.style.borderColor = 'var(--border-color)';
              keyBtn.style.background = 'var(--bg-card)';
              initTrainer();
            }, 500);
          } else {
            // Flash Red
            keyBtn.style.borderColor = '#ff3b30';
            keyBtn.style.background = 'rgba(255,59,48,0.2)';
            setTimeout(() => {
              keyBtn.style.borderColor = 'var(--border-color)';
              keyBtn.style.background = 'var(--bg-card)';
              document.getElementById('trainer-score').textContent = `SCORE: ${trainerState.correct}/${trainerState.total}`;
            }, 500);
          }
        } else {
          document.getElementById('morse-output').textContent += char;
          const wpm = parseInt(wpmSlider.value);
          const dotLength = 1200 / wpm;
          morseTimer = setTimeout(() => {
            document.getElementById('morse-output').textContent += ' ';
          }, dotLength * 4);
        }
      }
  ```

- [ ] **Step 3: Verify quiz logic**

  Toggle to Trainer. Press W (dot-dash-dash) on keyer. Confirm score updates, background flashes green or red, and a new prompt loads.

- [ ] **Step 4: Commit and Push**
  ```bash
  git add index.html
  git commit -m "feat: implement Morse Code Trainer character quiz logic and color cues"
  git push
  ```
