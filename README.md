# AC3GX Amateur Radio Console Homepage

A high-tech, responsive "Radio Shack Console" style dashboard homepage for amateur radio callsign **AC3GX**. This website is built as a single, self-contained HTML file designed for fast loading, clean visuals, and complete offline capability.

## 🚀 Features

- **Branding & Console Aesthetic:** Sleek tactical dark theme with glowing neon LED accents and transceiver-like dashboards.
- **UTC & Local Clocks:** Real-time clock displays with a pulsing second indicator.
- **Solar Propagation Panel:** Displays simulated solar flux index (SFI), Sunspot Number (SSN), A/K indices, and HF band conditions (80m-10m) color-coded by quality.
- **Simulated DMR Feed:** Live scrolling activity ticker representing contacts on the BrandMeister DMR network (TG 3100/91).
- **FT8 Logbook:** Log digital QSO contacts (Callsign, Band, Mode, Reports) directly on the page. Data is saved in the browser's `localStorage` and stays intact across reloads.
- **ADIF Log Exporter:** Download your logged contacts as a standard `.adi` (ADIF) file to import into other amateur radio logging software.
- **Morse Code Keyer:** An interactive CW keyer (straight key). Sounds a 700Hz sidetone using the native browser Web Audio API and decodes your spacebar or click patterns into text (WPM speed is adjustable).
- **Station Blog & Reader Overlay:** Center-column tab panel hosting blog post cards with tag filters, post search, a slide-up terminal markdown parser (headers, bold/italic, code, bullet lists), and persistent guest comments.
- **Admin Composer Workstation:** Passcode-secured console (`admin`) to compose, draft, edit, and delete blog posts, complete with JSON backup import/export utilities.
- **UDOP Dynamic Layout Control:**
  - **Dynamic Card Resizing:** Custom grab-resize handles (`resize: both; overflow: hidden;`) on all console cards, with adjustments persistently stored. Hides outer boundaries to prevent double scrollbars.
  - **Drag & Drop Layout Reordering:** Reorder panels within or across layout columns via drag-and-drop actions, persistently saved to local browser storage.
  - **Resizable Inner Viewports:** Vertical resizability on scroll boxes (QSO logs, blog post cards feed, text overlay editor textareas, post viewer) with full flexbox-stretching support when parent cards are scaled.

## 🛠️ Tech Stack

- **Structure:** Semantic HTML5
- **Styling:** Vanilla CSS3 (CSS Grid & Flexbox, responsive design, custom properties, animations)
- **Logic:** Native JavaScript (ES6+)
- **Audio:** Web Audio API (Synthesized tone generator)
- **Data:** Web Storage API (LocalStorage)

## 📖 How to Use

1. Clone or download this repository.
2. Double-click the `index.html` file to open it in any web browser (Chrome, Firefox, Safari, Edge).
3. Connect your headphones or turn up your volume to practice Morse code with the keyer!
