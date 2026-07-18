# AC3GX Station Blog & Operator Log Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement a fully featured, offline-capable "Station Blog" tab inside the existing dashboard console, including markdown compiling, comments, passcode-protected writing panel, and JSON import/export databases.

**Architecture:** Modifies `index.html` to integrate tabbed panels in the center column, using pure JavaScript to render posts, compile markdown styles via regular expressions, and authenticate/save posts locally.

**Tech Stack:** HTML5, CSS3, JavaScript (ES6+), LocalStorage.

## Global Constraints
- Single self-contained HTML file `index.html` located at the project root.
- No external CSS or JS libraries.
- Match existing deep-glass theme variables and Outfits/JetBrains Mono fonts.

---

### Task 1: Tab Navigation UI & Styles

**Files:**
- Modify: `index.html`
- Test: Verify clicking tab buttons switches active styling and hides/shows respective pane areas.

**Interfaces:**
- Consumes: Existing header and center-column cards.
- Produces: Tab selection styles, CSS `.tab-btn`, function `switchTab(tabName)`.

- [ ] **Step 1: Inject tab styles in head element of index.html**
  Inject these CSS styles into the `<style>` block:
  ```css
  /* Center Column Tab Button Styling */
  .console-tabs {
    display: flex;
    gap: 0.5rem;
    border-bottom: 1px solid var(--border-color);
    padding-bottom: 0.5rem;
    width: 100%;
  }
  .tab-btn {
    background: transparent;
    border: none;
    border-bottom: 2px solid transparent;
    color: var(--text-secondary);
    padding: 0.25rem 0.75rem;
    font-family: 'Outfit', sans-serif;
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s ease;
  }
  .tab-btn:hover {
    color: var(--text-primary);
    text-shadow: 0 0 8px var(--accent-cyan);
  }
  .tab-btn.active {
    color: var(--accent-cyan);
    border-color: var(--accent-cyan);
    text-shadow: 0 0 8px var(--accent-glow-cyan);
  }
  .tab-content {
    display: none;
  }
  .tab-content.active {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  ```

- [ ] **Step 2: Wrap existing QSO logbook content inside #logbook-tab-content**
  Modify `<div class="center-col" ...>` card block. 
  Replace the card-header title with `<div class="console-tabs">` buttons, and wrap all content elements below it inside `<div id="logbook-tab-content" class="tab-content active">`.
  Add empty containers for the blog and admin tabs:
  ```html
  <div id="blog-tab-content" class="tab-content"></div>
  <div id="admin-tab-content" class="tab-content"></div>
  ```

- [ ] **Step 3: Add switchTab script function**
  Add the tab switcher script at the bottom of the script block:
  ```javascript
  function switchTab(tabId) {
    // Hide all tab contents
    document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
    // Deactivate all tab buttons
    document.querySelectorAll('.tab-btn').forEach(el => el.classList.remove('active'));
    
    // Show active tab contents
    const activeContent = document.getElementById(`${tabId}-tab-content`);
    if (activeContent) activeContent.classList.add('active');
    
    // Find matching button to activate
    const activeBtn = Array.from(document.querySelectorAll('.tab-btn'))
      .find(btn => btn.getAttribute('onclick').includes(tabId));
    if (activeBtn) activeBtn.classList.add('active');
  }
  ```

- [ ] **Step 4: Commit tab changes**
  ```bash
  git add index.html
  git commit -m "feat: add dashboard tab bar controls and tab switching layout structure"
  ```

---

### Task 2: Blog Database, Initial Seed & Post List View

**Files:**
- Modify: `index.html`
- Test: Verify the default posts display in the Blog tab, and tags filter active results.

**Interfaces:**
- Consumes: `#blog-tab-content` element, tab switching logic.
- Produces: Blog database seed, list cards layout, tag list filter buttons.

- [ ] **Step 1: Define seed data and list view markup in index.html**
  Populate `#blog-tab-content` in the HTML body:
  ```html
  <!-- BLOG TAB -->
  <div id="blog-tab-content" class="tab-content">
    <div style="display:flex; justify-content:space-between; align-items:center; gap:0.5rem; flex-wrap:wrap;">
      <input type="text" id="blog-search" oninput="filterBlog()" placeholder="Search posts..." style="background:var(--bg-card); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; width:100%; max-width:200px; color:white; font-size:0.8rem; outline:none;">
      <div id="blog-tag-filters" style="display:flex; gap:0.4rem; overflow-x:auto;">
        <!-- Clickable tags dynamically added here -->
      </div>
    </div>
    <div id="blog-posts-list" style="display:flex; flex-direction:column; gap:0.75rem; max-height:450px; overflow-y:auto; padding-right:0.25rem;">
      <!-- Blog post cards injected here -->
    </div>
  </div>
  ```

- [ ] **Step 2: Add CSS styling for blog post cards**
  Add card styling to the style block:
  ```css
  .blog-card {
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 8px;
    padding: 1rem;
    cursor: pointer;
    transition: all 0.2s ease;
  }
  .blog-card:hover {
    border-color: var(--accent-cyan);
    box-shadow: 0 0 8px var(--accent-glow-cyan);
    transform: translateY(-1px);
  }
  .blog-meta {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.75rem;
    color: var(--text-secondary);
    display: flex;
    justify-content: space-between;
    margin-bottom: 0.4rem;
  }
  .blog-title {
    font-size: 1.1rem;
    font-weight: 600;
    color: var(--accent-green);
    margin-bottom: 0.4rem;
  }
  .blog-tag {
    display: inline-block;
    background: var(--bg-primary);
    border: 1px solid var(--border-color);
    color: var(--accent-amber);
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.7rem;
    padding: 0.1rem 0.4rem;
    border-radius: 3px;
    margin-right: 0.3rem;
  }
  ```

- [ ] **Step 3: Implement seed data loading and render function in JavaScript**
  Insert database initializing scripts:
  ```javascript
  const DEFAULT_BLOG_POSTS = [
    {
      id: "post-1",
      title: "My Yaesu FT-891 Mobile/Portable Shack Setup",
      content: "### Portable Station Setup\nSetting up portable HF operations in local state parks is one of the most rewarding parts of this hobby. Today, I'm documenting my compact transceiver layout.\n\n### The Transceiver\nThe core rig is a `Yaesu FT-891`. It is ultra-compact, handles up to 100W, and has an incredibly robust DSP system to handle band noise.\n\n### The Antenna\nI'm using an **End-Fed Horizontal Wire Antenna** configured as a sloper. It takes less than 10 minutes to fling a guide line over a tall tree branch and tie off the feedpoint.",
      tags: ["Hardware", "HF", "Portable"],
      timestamp: Date.now() - 86400000 * 2,
      status: "published",
      comments: []
    },
    {
      id: "post-2",
      title: "Digital Modes Overview: FT8 on 20m",
      content: "### Why Digital Modes?\nDigital modes like `FT8` and `FT4` allow amateur radio operators to make contacts even when solar conditions are poor or antenna setups are highly constrained.\n\n### Station Config\n- **Rig:** Yaesu FT-891\n- **Software:** WSJT-X connected via USB soundcard interface\n- **Power:** 25 Watts is typically more than enough for worldwide DX propagation.\n\nCheck out my recent log entries in the `QSO Logbook` tab to see some of my successful digital mode contacts!",
      tags: ["Digital", "FT8", "Software"],
      timestamp: Date.now() - 86400000 * 5,
      status: "published",
      comments: []
    }
  ];

  let selectedTag = 'All';

  function getBlogPosts() {
    let posts = localStorage.getItem('ac3gx_blog_db');
    if (!posts) {
      localStorage.setItem('ac3gx_blog_db', JSON.stringify(DEFAULT_BLOG_POSTS));
      return DEFAULT_BLOG_POSTS;
    }
    return JSON.parse(posts);
  }

  function saveBlogPosts(posts) {
    localStorage.setItem('ac3gx_blog_db', JSON.stringify(posts));
  }

  function renderBlogList() {
    const posts = getBlogPosts();
    const list = document.getElementById('blog-posts-list');
    const searchVal = document.getElementById('blog-search').value.toLowerCase();
    list.innerHTML = '';

    // Collect all tags dynamically
    const tagsSet = new Set(['All']);
    posts.forEach(p => {
      if (p.status === 'published') {
        p.tags.forEach(t => tagsSet.add(t));
      }
    });
    renderTagFilters(Array.from(tagsSet));

    const published = posts.filter(p => p.status === 'published');
    const sorted = published.sort((a, b) => b.timestamp - a.timestamp);

    sorted.forEach(p => {
      const titleMatch = p.title.toLowerCase().includes(searchVal);
      const contentMatch = p.content.toLowerCase().includes(searchVal);
      const tagMatch = selectedTag === 'All' || p.tags.includes(selectedTag);

      if ((titleMatch || contentMatch) && tagMatch) {
        // Read time calculation (avg 200 words per minute)
        const wordCount = p.content.split(/\s+/).length;
        const readTime = Math.max(1, Math.ceil(wordCount / 200));

        const date = new Date(p.timestamp);
        const dateStr = `${date.getUTCFullYear()}-${String(date.getUTCMonth()+1).padStart(2, '0')}-${String(date.getUTCDate()).padStart(2, '0')}`;

        const card = document.createElement('div');
        card.className = 'blog-card';
        card.onclick = () => openReader(p.id);
        card.innerHTML = `
          <div class="blog-meta">
            <span>${dateStr} UTC</span>
            <span>${readTime} min read</span>
          </div>
          <div class="blog-title">${p.title}</div>
          <div style="font-size:0.85rem; color:var(--text-secondary); margin-bottom:0.5rem; line-height:1.4;">
            ${p.content.substring(0, 100)}...
          </div>
          <div>
            ${p.tags.map(t => `<span class="blog-tag">#${t}</span>`).join('')}
          </div>
        `;
        list.appendChild(card);
      }
    });
  }

  function renderTagFilters(tags) {
    const container = document.getElementById('blog-tag-filters');
    container.innerHTML = '';
    tags.forEach(t => {
      const btn = document.createElement('button');
      btn.style.cssText = `
        background: ${selectedTag === t ? 'var(--accent-cyan)' : 'var(--bg-card)'};
        color: ${selectedTag === t ? 'var(--bg-primary)' : 'var(--text-secondary)'};
        border: 1px solid var(--border-color);
        border-radius: 3px;
        padding: 0.15rem 0.4rem;
        font-size: 0.7rem;
        font-family: 'JetBrains Mono', monospace;
        cursor: pointer;
        font-weight: bold;
        transition: all 0.2s ease;
      `;
      btn.textContent = t === 'All' ? '[ALL]' : `#${t}`;
      btn.onclick = () => {
        selectedTag = t;
        renderBlogList();
      };
      container.appendChild(btn);
    });
  }

  function filterBlog() {
    renderBlogList();
  }
  ```

- [ ] **Step 4: Commit Blog list view**
  ```bash
  git add index.html
  git commit -m "feat: implement persistent blog post databases and list rendering with tag filters"
  ```

---

### Task 3: Markdown Compiler & Post Reader Overlay

**Files:**
- Modify: `index.html`
- Test: Verify clicking a post card displays overlay layout, compiles headers/lists correctly, and visitor comments add instantly.

**Interfaces:**
- Consumes: Post card click event, blog rendering modules.
- Produces: `compileMarkdown(text)` regex parser, `#blog-reader-overlay` container, script handlers for comment persistence.

- [ ] **Step 1: Inject reader overlay layout inside blog-tab-content**
  Append this HTML inside `<div id="blog-tab-content" ...>` (below the card grid wrapper):
  ```html
  <!-- READER OVERLAY MODAL -->
  <div id="blog-reader-overlay" style="display:none; position:absolute; inset:0; background:var(--bg-panel); border-radius:10px; z-index:100; flex-direction:column; padding:1.25rem; gap:0.75rem;">
    <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid var(--border-color); padding-bottom:0.5rem;">
      <h3 id="reader-title" style="color:var(--accent-green); margin:0;"></h3>
      <button onclick="closeReader()" style="background:transparent; border:1px solid var(--border-color); color:var(--text-secondary); padding:0.25rem 0.5rem; border-radius:4px; font-size:0.75rem; cursor:pointer;">[CLOSE]</button>
    </div>
    <div id="reader-meta" style="font-family:'JetBrains Mono', monospace; font-size:0.75rem; color:var(--text-secondary); display:flex; gap:1rem;"></div>
    <div id="reader-body" style="flex:1; overflow-y:auto; padding-right:0.25rem; margin:0.25rem 0; border-bottom:1px solid var(--border-color); font-size:0.9rem;">
      <!-- Compiled Markdown text goes here -->
    </div>
    
    <!-- COMMENTS -->
    <div style="display:flex; flex-direction:column; gap:0.5rem; max-height:160px;">
      <div style="font-weight:bold; font-size:0.8rem; text-transform:uppercase; color:var(--text-secondary);">Comments</div>
      <div id="reader-comments" style="overflow-y:auto; display:flex; flex-direction:column; gap:0.35rem; flex:1; font-size:0.75rem;">
        <!-- Comments injected here -->
      </div>
      <form id="comment-form" onsubmit="submitComment(event)" style="display:flex; gap:0.35rem;">
        <input type="text" id="comment-name" required placeholder="Name" style="width:80px; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.3rem; color:white; font-size:0.75rem; outline:none;">
        <input type="text" id="comment-text" required placeholder="Write a comment..." style="flex:1; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.3rem; color:white; font-size:0.75rem; outline:none;">
        <button type="submit" style="background:var(--accent-cyan); color:var(--bg-primary); border:none; padding:0.3rem 0.6rem; border-radius:4px; font-weight:bold; font-size:0.75rem; cursor:pointer;">SEND</button>
      </form>
    </div>
  </div>
  ```

- [ ] **Step 2: Implement Markdown compiler and overlay script**
  Inject these JS functions in `<script>`:
  ```javascript
  let activePostId = null;

  function compileMarkdown(mdText) {
    let html = mdText
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;');

    // Headers
    html = html.replace(/^### (.*?)$/gm, '<h4 style="color:var(--accent-cyan); margin: 0.6rem 0 0.3rem 0; font-size:1rem;">$1</h4>');
    html = html.replace(/^## (.*?)$/gm, '<h3 style="color:var(--accent-green); margin: 0.8rem 0 0.4rem 0; font-size:1.15rem;">$1</h3>');

    // Bold (**text**)
    html = html.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');

    // Italics (*text*)
    html = html.replace(/\*(.*?)\*/g, '<em>$1</em>');

    // Code Blocks (```code```)
    html = html.replace(/```([\s\S]*?)```/g, '<pre style="background:var(--bg-primary); padding:0.4rem; border-radius:4px; border:1px solid var(--border-color); font-family:\'JetBrains Mono\', monospace; margin: 0.4rem 0; overflow-x:auto; font-size:0.8rem; color:var(--text-primary);"><code>$1</code></pre>');

    // Inline Code (`code`)
    html = html.replace(/`(.*?)`/g, '<code style="background:var(--bg-primary); padding:0.1rem 0.25rem; border-radius:3px; font-family:\'JetBrains Mono\', monospace; color:var(--accent-amber);">$1</code>');

    // Bullet Lists
    html = html.replace(/^\- (.*?)$/gm, '<li style="margin-left: 1.2rem; margin-bottom: 0.2rem; list-style-type:square;">$1</li>');

    // Newlines to Paragraphs
    html = html.replace(/\n\n/g, '</p><p style="margin-bottom:0.6rem;">');

    return `<p style="margin-bottom:0.6rem; line-height:1.5;">` + html + `</p>`;
  }

  function openReader(postId) {
    const posts = getBlogPosts();
    const post = posts.find(p => p.id === postId);
    if (!post) return;

    activePostId = postId;
    document.getElementById('reader-title').textContent = post.title;
    
    const date = new Date(post.timestamp);
    const dateStr = `${date.getUTCFullYear()}-${String(date.getUTCMonth()+1).padStart(2, '0')}-${String(date.getUTCDate()).padStart(2, '0')} UTC`;
    document.getElementById('reader-meta').innerHTML = `
      <span>Published: ${dateStr}</span>
      <span>Tags: ${post.tags.map(t => `#${t}`).join(', ')}</span>
    `;

    document.getElementById('reader-body').innerHTML = compileMarkdown(post.content);
    renderComments(post.comments);

    const overlay = document.getElementById('blog-reader-overlay');
    overlay.style.display = 'flex';
  }

  function closeReader() {
    activePostId = null;
    document.getElementById('blog-reader-overlay').style.display = 'none';
  }

  function renderComments(comments) {
    const list = document.getElementById('reader-comments');
    list.innerHTML = '';
    
    if (!comments || comments.length === 0) {
      list.innerHTML = `<span style="color:var(--text-secondary); font-style:italic;">No comments yet. Be the first!</span>`;
      return;
    }

    comments.forEach(c => {
      const d = new Date(c.timestamp);
      const timeStr = `${d.getUTCHours()}:${String(d.getUTCMinutes()).padStart(2, '0')}`;
      const div = document.createElement('div');
      div.style.cssText = `
        background: var(--bg-primary);
        border-radius: 4px;
        padding: 0.3rem 0.5rem;
        border-left: 2px solid var(--accent-cyan);
      `;
      div.innerHTML = `
        <div style="display:flex; justify-content:space-between; margin-bottom:0.15rem; color:var(--text-secondary); font-weight:bold; font-size:0.7rem;">
          <span style="color:var(--accent-cyan);">${c.name}</span>
          <span>${timeStr} UTC</span>
        </div>
        <div>${c.comment}</div>
      `;
      list.appendChild(div);
    });
    // Auto scroll bottom
    list.scrollTop = list.scrollHeight;
  }

  function submitComment(e) {
    e.preventDefault();
    if (!activePostId || !powerOn) return;
    const nameInput = document.getElementById('comment-name');
    const commentInput = document.getElementById('comment-text');

    const posts = getBlogPosts();
    const index = posts.findIndex(p => p.id === activePostId);
    if (index === -1) return;

    if (!posts[index].comments) posts[index].comments = [];
    
    posts[index].comments.push({
      name: nameInput.value.trim(),
      comment: commentInput.value.trim(),
      timestamp: Date.now()
    });

    saveBlogPosts(posts);
    renderComments(posts[index].comments);
    commentInput.value = '';
  }
  ```

- [ ] **Step 3: Commit reader overlay module**
  ```bash
  git add index.html
  git commit -m "feat: implement self-contained Markdown parser and interactive post reader overlay"
  ```

---

### Task 4: Admin Panel Auth & Composition Form

**Files:**
- Modify: `index.html`
- Test: Verify typing passcode opens draft editor, writing post updates draft, and clicking "Publish" adds post to lists.

**Interfaces:**
- Consumes: `#admin-tab-content`, passcode inputs.
- Produces: Password lock layout, Admin panel post-manager lists, rich blog compose forms.

- [ ] **Step 1: Add HTML markup for Admin Console Panel**
  Inject passcode interface, composer form, and post manager dashboard layout inside `#admin-tab-content`.
  ```html
  <div id="admin-tab-content" class="tab-content">
    <!-- AUTH PANEL -->
    <div id="admin-auth-pane" style="display:flex; flex-direction:column; gap:0.75rem; align-items:center; justify-content:center; padding:2rem 0;">
      <span style="font-size:0.85rem; color:var(--text-secondary); text-transform:uppercase;">Admin Access Required</span>
      <div style="display:flex; gap:0.5rem;">
        <input type="password" id="admin-passcode" placeholder="Enter Access Code" style="background:var(--bg-card); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-size:0.85rem; outline:none; text-align:center; font-family:'JetBrains Mono', monospace;">
        <button onclick="loginAdmin()" style="background:var(--accent-green); color:var(--bg-primary); border:none; padding:0.4rem 0.8rem; border-radius:4px; font-weight:bold; cursor:pointer; font-size:0.85rem;">ACCESS</button>
      </div>
      <span id="auth-error" style="color:#ff3b30; font-size:0.75rem; display:none;">Invalid Code. Retry.</span>
    </div>

    <!-- WORKSTATION EDITOR PANEL (Hidden by default) -->
    <div id="admin-editor-pane" style="display:none; flex-direction:column; gap:0.75rem;">
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <span style="font-size:0.8rem; color:var(--accent-green); font-weight:bold;">COMPOSER WORKSTATION</span>
        <button onclick="logoutAdmin()" style="background:transparent; border:1px solid var(--border-color); color:var(--text-secondary); padding:0.2rem 0.5rem; border-radius:4px; font-size:0.75rem; cursor:pointer;">Exit Console</button>
      </div>

      <!-- COMPOSE FORM -->
      <form id="blog-form" onsubmit="savePost(event)" style="background:var(--bg-card); border:1px solid var(--border-color); border-radius:6px; padding:0.75rem; display:flex; flex-direction:column; gap:0.5rem;">
        <input type="hidden" id="edit-post-id">
        <div style="display:grid; grid-template-columns: 2fr 1fr; gap:0.5rem;">
          <div>
            <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">Title</label>
            <input type="text" id="post-title" required placeholder="Post Title" style="width:100%; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-size:0.85rem; outline:none;">
          </div>
          <div>
            <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">Tags (comma-separated)</label>
            <input type="text" id="post-tags" placeholder="e.g. Hardware, FT8" style="width:100%; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-size:0.85rem; outline:none; font-family:'JetBrains Mono', monospace;">
          </div>
        </div>
        <div>
          <label style="font-size:0.7rem; color:var(--text-secondary); display:block; margin-bottom:0.2rem;">Content (Markdown)</label>
          <textarea id="post-body" required placeholder="Write your post here... Use ## for headings, **bold**, *italic*, `code`, and - for lists." style="width:100%; height:180px; background:var(--bg-primary); border:1px solid var(--border-color); border-radius:4px; padding:0.4rem; color:white; font-size:0.85rem; outline:none; font-family:'Outfit', sans-serif; resize:vertical;"></textarea>
        </div>
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <div style="display:flex; align-items:center; gap:0.35rem; font-size:0.8rem;">
            <input type="checkbox" id="post-draft" style="cursor:pointer;">
            <label for="post-draft" style="cursor:pointer;">Save as Draft (Hide from Blog list)</label>
          </div>
          <div style="display:flex; gap:0.4rem;">
            <button type="button" onclick="resetEditor()" style="background:transparent; border:1px solid var(--border-color); color:var(--text-secondary); padding:0.4rem 0.8rem; border-radius:4px; cursor:pointer; font-size:0.8rem;">CLEAR</button>
            <button type="submit" style="background:var(--accent-green); color:var(--bg-primary); border:none; padding:0.4rem 1rem; border-radius:4px; font-weight:bold; cursor:pointer; font-size:0.8rem;">SAVE POST</button>
          </div>
        </div>
      </form>

      <!-- MANAGE POSTS SECTION -->
      <div style="border-top:1px solid var(--border-color); padding-top:0.75rem;">
        <span style="font-size:0.8rem; color:var(--text-secondary); text-transform:uppercase; display:block; margin-bottom:0.4rem;">Posts Catalog</span>
        <div id="admin-posts-catalog" style="display:flex; flex-direction:column; gap:0.4rem; max-height:140px; overflow-y:auto; font-size:0.8rem; font-family:'JetBrains Mono', monospace;">
          <!-- Catalog table rows injected here -->
        </div>
      </div>
    </div>
  </div>
  ```

- [ ] **Step 2: Add JS logic for Admin Login and Composer Save functions**
  Insert editor scripts:
  ```javascript
  let isAdmin = false;

  function loginAdmin() {
    const code = document.getElementById('admin-passcode').value;
    const errorSpan = document.getElementById('auth-error');
    if (code.toLowerCase() === 'admin') {
      isAdmin = true;
      errorSpan.style.display = 'none';
      document.getElementById('admin-auth-pane').style.display = 'none';
      document.getElementById('admin-editor-pane').style.display = 'flex';
      renderAdminCatalog();
    } else {
      errorSpan.style.display = 'inline';
    }
  }

  function logoutAdmin() {
    isAdmin = false;
    document.getElementById('admin-passcode').value = '';
    document.getElementById('admin-auth-pane').style.display = 'flex';
    document.getElementById('admin-editor-pane').style.display = 'none';
  }

  function renderAdminCatalog() {
    const posts = getBlogPosts();
    const catalog = document.getElementById('admin-posts-catalog');
    catalog.innerHTML = '';

    if (posts.length === 0) {
      catalog.innerHTML = `<span style="color:var(--text-secondary); font-style:italic;">No posts in database.</span>`;
      return;
    }

    const sorted = [...posts].sort((a, b) => b.timestamp - a.timestamp);
    sorted.forEach(p => {
      const row = document.createElement('div');
      row.style.cssText = `
        display:flex;
        justify-content:space-between;
        align-items:center;
        background:var(--bg-card);
        border:1px solid var(--border-color);
        padding:0.4rem;
        border-radius:4px;
      `;
      row.innerHTML = `
        <div style="overflow:hidden; text-overflow:ellipsis; white-space:nowrap; padding-right:1rem;">
          <span style="color:${p.status === 'draft' ? 'var(--accent-amber)' : 'var(--accent-green)'};">[${p.status.toUpperCase()}]</span>
          <span style="color:white; font-weight:bold;">${p.title}</span>
        </div>
        <div style="display:flex; gap:0.4rem;">
          <button onclick="editPost('${p.id}')" style="background:transparent; border:1px solid var(--accent-cyan); color:var(--accent-cyan); border-radius:3px; padding:0.1rem 0.3rem; font-size:0.7rem; cursor:pointer;">EDIT</button>
          <button onclick="deletePost('${p.id}')" style="background:transparent; border:1px solid #ff3b30; color:#ff3b30; border-radius:3px; padding:0.1rem 0.3rem; font-size:0.7rem; cursor:pointer;">DEL</button>
        </div>
      `;
      catalog.appendChild(row);
    });
  }

  function savePost(e) {
    e.preventDefault();
    if (!isAdmin || !powerOn) return;

    const editId = document.getElementById('edit-post-id').value;
    const titleVal = document.getElementById('post-title').value.trim();
    const tagsVal = document.getElementById('post-tags').value.trim();
    const contentVal = document.getElementById('post-body').value;
    const isDraft = document.getElementById('post-draft').checked;

    const posts = getBlogPosts();
    const processedTags = tagsVal ? tagsVal.split(',').map(t => t.trim()) : ['General'];

    if (editId) {
      // Modify existing
      const index = posts.findIndex(p => p.id === editId);
      if (index !== -1) {
        posts[index].title = titleVal;
        posts[index].tags = processedTags;
        posts[index].content = contentVal;
        posts[index].status = isDraft ? 'draft' : 'published';
      }
    } else {
      // Create new
      const newPost = {
        id: `post-${Date.now()}`,
        title: titleVal,
        content: contentVal,
        tags: processedTags,
        timestamp: Date.now(),
        status: isDraft ? 'draft' : 'published',
        comments: []
      };
      posts.push(newPost);
    }

    saveBlogPosts(posts);
    resetEditor();
    renderAdminCatalog();
    renderBlogList();
  }

  function editPost(id) {
    const posts = getBlogPosts();
    const post = posts.find(p => p.id === id);
    if (!post) return;

    document.getElementById('edit-post-id').value = post.id;
    document.getElementById('post-title').value = post.title;
    document.getElementById('post-tags').value = post.tags.join(', ');
    document.getElementById('post-body').value = post.content;
    document.getElementById('post-draft').checked = (post.status === 'draft');
  }

  function deletePost(id) {
    if (!confirm("Are you sure you want to delete this blog post?")) return;
    let posts = getBlogPosts();
    posts = posts.filter(p => p.id !== id);
    saveBlogPosts(posts);
    renderAdminCatalog();
    renderBlogList();
    if (activePostId === id) closeReader();
  }

  function resetEditor() {
    document.getElementById('edit-post-id').value = '';
    document.getElementById('post-title').value = '';
    document.getElementById('post-tags').value = '';
    document.getElementById('post-body').value = '';
    document.getElementById('post-draft').checked = false;
  }
  ```

- [ ] **Step 3: Initial loading updates**
  Update initial render calls in `<script>` so that `renderBlogList()` runs at start alongside `renderQSOs()`.
  
- [ ] **Step 4: Commit Admin Dashboard**
  ```bash
  git add index.html
  git commit -m "feat: add passcode-protected administrator blog console and compose form workstation"
  ```

---

### Task 5: JSON Import/Export Backup Utility

**Files:**
- Modify: `index.html`
- Test: Verify "Backup Database" exports a valid `.json` file, and upload restores logs.

**Interfaces:**
- Consumes: Admin workstation panel, blog databases.
- Produces: Import/Export buttons, event listener for file reading upload.

- [ ] **Step 1: Inject JSON Database buttons in admin editor panel**
  Add database tools inside `#admin-editor-pane` in the HTML body (above catalog section):
  ```html
  <div style="display:flex; justify-content:space-between; align-items:center; border:1px solid var(--border-color); padding:0.5rem; border-radius:4px; margin-bottom:0.25rem; background:var(--bg-card);">
    <span style="font-size:0.75rem; color:var(--text-secondary);">Database Actions</span>
    <div style="display:flex; gap:0.4rem; align-items:center;">
      <button onclick="exportBlogDB()" style="background:transparent; border:1px solid var(--accent-cyan); color:var(--accent-cyan); border-radius:3px; padding:0.2rem 0.5rem; font-size:0.7rem; cursor:pointer; font-weight:600;">Export JSON</button>
      <input type="file" id="import-blog-input" accept=".json" onchange="importBlogDB(event)" style="display:none;">
      <button onclick="document.getElementById('import-blog-input').click()" style="background:transparent; border:1px solid var(--accent-amber); color:var(--accent-amber); border-radius:3px; padding:0.2rem 0.5rem; font-size:0.7rem; cursor:pointer; font-weight:600;">Import JSON</button>
    </div>
  </div>
  ```

- [ ] **Step 2: Add Backup JavaScript handlers in script block**
  Inject database backup script routines:
  ```javascript
  function exportBlogDB() {
    const posts = getBlogPosts();
    const jsonStr = JSON.stringify(posts, null, 2);
    const blob = new Blob([jsonStr], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `ac3gx_blog_db.json`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  function importBlogDB(event) {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = function(e) {
      try {
        const posts = JSON.parse(e.target.result);
        if (Array.isArray(posts)) {
          // Validate basic schema structure
          const valid = posts.every(p => p.id && p.title && p.content && Array.isArray(p.tags));
          if (valid) {
            saveBlogPosts(posts);
            renderAdminCatalog();
            renderBlogList();
            alert("Database imported successfully!");
          } else {
            alert("Error: JSON database does not match the required post schema.");
          }
        } else {
          alert("Error: JSON file root must be an array of posts.");
        }
      } catch (err) {
        alert("Error parsing JSON backup file: " + err.message);
      }
    };
    reader.readAsText(file);
    // Clear input
    event.target.value = '';
  }
  ```

- [ ] **Step 3: Commit Backup Utility**
  ```bash
  git add index.html
  git commit -m "feat: implement JSON blog database import and export backup utility"
  ```

---

### Task 6: Final Responsive CSS Tuning & Verification

**Files:**
- Modify: `index.html`
- Test: Verify the layout responsive wrapping, tab switching, and overall page render cleanly on multiple devices.

- [ ] **Step 1: Check CSS overrides and flex items for overflow safety**
  Ensure cards wrap correctly on mobile, and the sidebar monitors scroll properly without layout breakage.
  Ensure `#blog-reader-overlay` sits nicely on high and narrow screen bounds.

- [ ] **Step 2: Test rendering via human check**
  Have user inspect local changes.
