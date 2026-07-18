# Design Spec: AC3GX Station Blog & Operator Log

**Date:** 2026-07-17  
**Author:** Antigravity  
**Status:** Approved  

This document describes the design and specification for the integrated, tabbed Station Blog & Operator Log feature on the AC3GX console homepage. The implementation remains inside the single `index.html` file.

---

## 1. Objectives & Scope
- **Integrated Tabs:** Replace the static QSO logbook header with a tab bar selector (`QSO Logbook`, `Station Blog`, `Admin Console`).
- **Station Blog Panel:** Modern responsive blog listing with search and category filters.
- **Post Reader Modal:** Smooth in-place blog reader displaying formatted text, tags, and persistent visitor comments.
- **Admin Panel:** Passcode-protected interface (`ac3gx123`) to compose, draft, edit, and delete blog posts.
- **Backup Utility:** Database import and export of blog posts/comments using JSON files.
- **Self-Contained Markdown Compiler:** Native JS parser to render headers, bold, italics, code segments, and lists.

---

## 2. UI/UX Changes & Markup

### Tab Selector Layout
Inject tab selector buttons into the center column console card:
```html
<div class="card-header" style="padding-bottom: 0;">
  <div style="display: flex; gap: 0.5rem; border-bottom: none;">
    <button class="tab-btn active" onclick="switchTab('logbook')">QSO Logbook</button>
    <button class="tab-btn" onclick="switchTab('blog')">Station Blog</button>
    <button class="tab-btn" onclick="switchTab('admin')">Admin Console</button>
  </div>
</div>
```

### Tab Containers
The card body is split into three sibling container elements:
1. `#logbook-tab-content`: Contains the existing FT8 logger form, search bar, and QSO table.
2. `#blog-tab-content`: Hidden by default. Contains the post search bar, tag filter buttons, scrollable post grid, and `#blog-reader-overlay`.
3. `#admin-tab-content`: Hidden by default. Contains the login screen, draft composer form, post edit index list, and JSON database tools.

### Theme & Animations
- Tab buttons: Hover state lights up with neon cyan shadow. Active state has a solid glowing line indicator underneath.
- Post cards: Glassmorphism (`backdrop-filter: blur(8px)`) with glowing hover states.
- Blog reader overlay: absolute layout covering the center card, sliding up from the bottom or fading in with high-fidelity transition.

---

## 3. Data Schema & Persistence

### Blog Database Schema
All posts are stored in `localStorage` under the key `ac3gx_blog_db` as an array of JSON objects:
```json
[
  {
    "id": "post-unique-id",
    "title": "First QSO with Pennsylvania Shack Setup",
    "content": "### Hello World\nRecently set up the Yaesu FT-891 and an end-fed horizontal wire...",
    "tags": ["Hardware", "HF", "FT-891"],
    "timestamp": 1784305800000,
    "status": "published",
    "comments": [
      {
        "name": "K3LR",
        "comment": "Nice setup! Signals are clean on 20m.",
        "timestamp": 1784305900000
      }
    ]
  }
]
```

### Initial Seed Data
If `ac3gx_blog_db` is missing, populate it with two pre-written posts:
1. **Title:** "My Yaesu FT-891 Mobile/Portable Shack Setup"  
   **Content:** Detailed markdown post highlighting the FT-891 transceiver features, the LDG IT-100 tuner, and an end-fed wire antenna setup.
2. **Title:** "Digital Modes Overview: FT8 on 20m"  
   **Content:** Guide detailing FT8 software (WSJT-X) configurations, signal reporting, and the benefits of weak-signal communications.

---

## 4. Logical Components & Compiler

### Simple Markdown Compiler
We will construct a client-side regex-based parser in JavaScript:
```javascript
function compileMarkdown(mdText) {
  let html = mdText
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');

  // Headers (e.g. ### Header)
  html = html.replace(/^### (.*?)$/gm, '<h3 style="color:var(--accent-cyan); margin: 0.8rem 0 0.4rem 0;">$1</h3>');
  html = html.replace(/^## (.*?)$/gm, '<h2 style="color:var(--accent-green); margin: 1rem 0 0.5rem 0;">$1</h2>');

  // Bold (**text**)
  html = html.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');

  // Italics (*text*)
  html = html.replace(/\*(.*?)\*/g, '<em>$1</em>');

  // Code Blocks (```code```)
  html = html.replace(/```([\s\S]*?)```/g, '<pre style="background:var(--bg-primary); padding:0.5rem; border-radius:4px; border:1px solid var(--border-color); font-family:\'JetBrains Mono\', monospace; margin: 0.5rem 0; overflow-x:auto;"><code>$1</code></pre>');

  // Inline Code (`code`)
  html = html.replace(/`(.*?)`/g, '<code style="background:var(--bg-primary); padding:0.1rem 0.3rem; border-radius:3px; font-family:\'JetBrains Mono\', monospace; color:var(--accent-amber);">$1</code>');

  // Bullet Lists
  html = html.replace(/^\- (.*?)$/gm, '<li style="margin-left: 1.2rem; margin-bottom: 0.25rem;">$1</li>');

  // Newlines to Paragraphs
  html = html.replace(/\n\n/g, '</p><p style="margin-bottom:0.8rem;">');

  return `<p style="margin-bottom:0.8rem; line-height:1.6; font-size:0.95rem;">` + html + `</p>`;
}
```

### Authentication Logic
- Passcode matches `ac3gx123`.
- Set session state `isAdmin = true` in JavaScript upon successful passcode validation, enabling editing and composition buttons.

---

## 5. Verification Plan
- **Verification Method:** Manual testing of blog features directly inside the browser.
- **Verify UI Controls:** Check tab buttons correctly hide/show containers.
- **Verify Blog Reader:** Confirm click on blog post displays reader overlay, renders markdown elements properly, and updates comments on submit.
- **Verify Database Utility:** Check JSON database exports a valid JSON file and loads it back successfully.
