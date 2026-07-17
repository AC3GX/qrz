# Design Spec: AC3GX Amateur Radio Console Homepage

**Date:** 2026-07-17  
**Author:** Antigravity  
**Status:** Approved  

This document describes the design and specification for a high-tech "Radio Shack Console" style dashboard homepage for amateur radio callsign **AC3GX**. The homepage is built as a single, self-contained HTML file.

---

## 1. Objectives & Scope
- **Callsign Branding:** Distinct callsign **AC3GX** displayed prominently, tailored for digital modes (FT8, DMR).
- **Console Aesthetic:** Sleek, tactical dark-themed interface reminiscent of modern amateur radio transceivers and dashboard consoles.
- **Self-Contained:** A single file containing all CSS, HTML, and JS. No build steps, no framework dependencies, and runs completely client-side.
- **Interactivity:** Real-time clocks (local and UTC), simulated live feeds (DMR ticker, FT8 QSO stream), interactive Morse code straight-key/paddle with sidetone audio, and customizable station logging with ADIF file export.

---

## 2. Technical Stack
- **Structure:** Semantic HTML5 elements (`<header>`, `<main>`, `<section>`, `<article>`).
- **Styling:** Vanilla CSS3. Grid layout for modular panels, flexbox for internal card structure, custom CSS variables for color scheme, and keyframe animations for pulsers.
- **Typography:** 
  - *Outfit* (sans-serif) for descriptive labels, titles, and menus.
  - *JetBrains Mono* (monospaced) for frequency readouts, UTC time, Morse outputs, and log records.
- **Assets:** Inline SVGs for all icons to guarantee zero network latency and offline capabilities.
- **Audio:** Web Audio API (`AudioContext`, `OscillatorNode`, `GainNode`) for sidetone generation.
- **Data Persistence:** `localStorage` for saving newly logged QSOs.

---

## 3. UI/UX Layout & Aesthetics
The console is structured as a responsive dashboard grid.

### Grid Layout (3 Columns on Desktop, 1 Column on Mobile)
1. **Header:** 
   - Left: Station callsign **AC3GX** with an active glowing neon green LED.
   - Center: Live UTC clock and Local time clock side-by-side with a flashing seconds separator.
   - Right: Station status panel (Active frequency: e.g., `14.074 MHz` for 20m FT8, Mode: `FT8`, virtual Power button).
2. **Left Column (Station & DMR):**
   - **Station Bio Card:** Grid location FM19 (or FM19ff), operator class, transceiver model (Icom IC-7300), and antenna description.
   - **DMR Activity Ticker:** Scrolling list representing BrandMeister DMR network activity (TG 3100 / TG 91). Updates every 5-8 seconds with random callsigns, RSSI, and transmission times.
3. **Center Column (FT8 Logbook):**
   - **Interactive QSO Logger:** Input form to add QSO details (Callsign, Sent Report, Rcvd Report, Band, Mode).
   - **Log Book Table:** Scrollable log of past digital contacts. Includes band/mode filters, local search input, and an "Export ADIF" button.
4. **Right Column (Solar & Morse):**
   - **Solar Propagation Panel:** Displays SFI (Solar Flux Index), SSN (Sunspot Number), A-Index, K-Index, and active HF band status (Good/Fair/Poor) under green/yellow/red indicators.
   - **Morse Code Trainer Keyer:** Visual straight key button. Click or tap/hold `Spacebar` to sound a 700Hz tone. Audio duration is tracked to decode dits and dahs into characters in real time.

### Aesthetic Theme (Modern Tactical Dark)
- Background: `#0b0c10` (pure dark slate/charcoal)
- Card Panels: `#1f2833` with semi-transparent borders `rgba(255, 255, 255, 0.05)` and `backdrop-filter: blur(10px)` (glassmorphism)
- Border: `#2c3531`
- Accent - Neon Green: `#00ff87` (signal indicators, active bands, power)
- Accent - Cyan: `#4fc3f7` (time displays, digital frequencies)
- Accent - Amber: `#ffb300` (solar flux warning levels, active transmission highlights)

---

## 4. Logical Components & Algorithms

### Morse Code Decoder
- Employs JavaScript `performance.now()` to measure the duration of mouse/touch press or Spacebar keydowns.
- Timing thresholds:
  - Dit: `< 150ms`
  - Dah: `150ms - 450ms`
  - Character Gap: `> 450ms` (silence)
  - Word Gap: `> 1200ms` (silence)
- Maps decoded sequence (e.g. `.- -...` for `AB`) using a mapping lookup table to print characters to a text output field.

### ADIF Log Exporter
- Collects the array of QSO objects from `localStorage` (combining default preloaded FT8 contacts with user-logged contacts).
- Generates a standard ADIF (Amateur Data Interchange Format) string:
  ```text
  ADIF Export from AC3GX Station Console
  <ADIF_VER:5>3.1.4
  <PROGRAMID:13>AC3GX_CONSOLE
  <EOH>
  <CALL:5>K1ABC <QSO_DATE:8>20260717 <TIME_ON:6>163000 <BAND:3>20M <MODE:3>FT8 <RST_SENT:3>-10 <RST_RCVD:3>-12 <EOR>
  ```
- Creates a `Blob` containing this text and triggers a browser file download labeled `AC3GX_Logbook.adi`.

### Live Simulation Engines
- **UTC Clock:** Standard JS interval updating the UTC datetime string.
- **QSO Stream Simulator:** Every 15-20 seconds, a background function simulates receiving an FT8 decode packet, showing a mini scrolling waterfalls text line like `163015 -14  0.8 14074 ~ AC3GX K3LR -08`.
- **DMR Ticker:** Simulates BrandMeister DMR feed. Appends a new transmission line, popping old lines once it exceeds the dashboard card height to maintain performance.

---

## 5. Verification Plan
- **Verification Method:** Manual testing of UI features and file rendering in a web browser using visual subagents.
- **Audio Verification:** Confirm Web Audio API output works.
- **Local Persistence Check:** Validate QSO form adds to the logbook table, persists across browser reloads, and downloads a valid ADIF file.
- **Responsive Layout:** Confirm grid columns wrap correctly on mobile device sizes.
