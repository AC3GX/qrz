# AC3GX UDOP Dynamic Layout Implementation Plan

**Goal:** Implement a User Defined Operational Picture (UDOP) dynamic dashboard layout, allowing the user to resize cards and drag-and-drop cards between layout columns, persistently saving configurations.

---

### Task 1: CSS Resizability & Custom Scrollbars

- [ ] Add `.console-card` properties to enable resize grab handles and container scroll management:
  ```css
  .console-card {
    resize: both;
    overflow: auto;
    min-width: 180px;
    min-height: 100px;
  }
  .console-card::-webkit-scrollbar {
    width: 4px;
    height: 4px;
  }
  .console-card::-webkit-scrollbar-track {
    background: transparent;
  }
  .console-card::-webkit-scrollbar-thumb {
    background: var(--border-color);
    border-radius: 2px;
  }
  .console-card::-webkit-scrollbar-thumb:hover {
    background: var(--accent-cyan);
  }
  ```

---

### Task 2: Dimensions & Order Persistence Script

- [ ] Add JavaScript module at the bottom of the script block to track and apply dimensions and ordering:
  ```javascript
  // Dimensions Observer
  const resizeObserver = new ResizeObserver(entries => {
    if (!powerOn) return;
    entries.forEach(entry => {
      const id = entry.target.id;
      if (!id) return;
      const dims = { width: entry.target.style.width, height: entry.target.style.height };
      localStorage.setItem(`ac3gx_panel_dim_${id}`, JSON.stringify(dims));
    });
  });

  function initPanelResizing() {
    document.querySelectorAll('.console-card').forEach((card, index) => {
      if (!card.id) card.id = `panel-${index}`;
      const saved = localStorage.getItem(`ac3gx_panel_dim_${card.id}`);
      if (saved) {
        try {
          const dims = JSON.parse(saved);
          if (dims.width) card.style.width = dims.width;
          if (dims.height) card.style.height = dims.height;
        } catch (e) {}
      }
      resizeObserver.observe(card);
    });
  }

  // Drag and Drop Layout Swapper
  let draggedElement = null;

  function initDragAndDrop() {
    const cards = document.querySelectorAll('.console-card');
    cards.forEach((card, index) => {
      if (!card.id) card.id = `panel-${index}`;
      card.setAttribute('draggable', 'true');
      
      card.addEventListener('dragstart', (e) => {
        if (!powerOn) {
          e.preventDefault();
          return;
        }
        draggedElement = card;
        e.dataTransfer.effectAllowed = 'move';
        card.style.opacity = '0.5';
      });

      card.addEventListener('dragend', () => {
        card.style.opacity = '1';
        saveCardOrder();
      });

      card.addEventListener('dragover', (e) => {
        e.preventDefault();
        e.dataTransfer.dropEffect = 'move';
      });

      card.addEventListener('drop', (e) => {
        e.preventDefault();
        if (draggedElement && draggedElement !== card) {
          swapNodes(draggedElement, card);
        }
      });
    });
    loadCardOrder();
  }

  function swapNodes(node1, node2) {
    const parent1 = node1.parentNode;
    const parent2 = node2.parentNode;
    const placeholder = document.createElement('div');
    parent1.insertBefore(placeholder, node1);
    parent2.insertBefore(node1, node2);
    parent1.insertBefore(node2, placeholder);
    parent1.removeChild(placeholder);
  }

  function saveCardOrder() {
    const order = [];
    document.querySelectorAll('.console-grid > div').forEach(col => {
      const colCards = [];
      col.querySelectorAll('.console-card').forEach(card => {
        colCards.push(card.id);
      });
      order.push(colCards);
    });
    localStorage.setItem('ac3gx_panel_order', JSON.stringify(order));
  }

  function loadCardOrder() {
    const saved = localStorage.getItem('ac3gx_panel_order');
    if (!saved) return;
    try {
      const order = JSON.parse(saved);
      const cols = document.querySelectorAll('.console-grid > div');
      order.forEach((colCards, colIdx) => {
        const col = cols[colIdx];
        if (!col) return;
        colCards.forEach(cardId => {
          const card = document.getElementById(cardId);
          if (card) col.appendChild(card);
        });
      });
    } catch (e) {}
  }
  ```

- [ ] Execute `initPanelResizing()` and `initDragAndDrop()` at initial load.
