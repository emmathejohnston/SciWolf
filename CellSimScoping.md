# KAN-38 — Inner Life of a Cell: Scoping Doc

_SNC1D/SNC1W · Point-and-click cell cycle game · Last updated 2026-03-31_

---

## What already exists

`cell.html` (927 lines, single file) has:
- All organelle definitions with positions, sizes, colours, info-card text
- Directed + scatter particle system (mRNA, proteins, vesicles, etc.)
- ATP HUD bar
- Info card pop-up per organelle
- WASD player character (a mitochondrion the student pilots around)
- Camera follow + cell boundary clamp
- Background mitochondria drifting

**What gets thrown out:** WASD movement, the player character, the "press E to donate ATP" interaction model.

**What gets kept/reused:** Organelle definitions, particle system, info cards, ATP bar concept, organelle draw functions.

---

## Multi-file architecture

Since this is a game-scale rebuild, split into four files:

```
cell.html         — HTML shell, all CSS, <script> tags
cell-data.js      — Static data: organelle defs, emission configs, phase progression rules
cell-draw.js      — All canvas drawing: organelles, particles, HUD overlays, mitosis animations
cell-game.js      — Game loop, state machine, click/drag input, particle logic, win condition
```

---

## Phase breakdown

### Phase 1 — Click Foundation
_Remove WASD; all organelles become clickable targets_

- Remove player character, WASD listeners, camera follow
- Camera is fixed (or gentle pan/zoom on mobile — TBD)
- All organelles respond to click/tap → open info card (same as now)
- **Mitochondria** click: ATP +25, burst animation, info card
- ATP regenerates slowly over time (passive); mitochondria click is the fast top-up
- ATP bar stays in HUD; add a **phase/progress label** to the HUD
- Tutorial text updated: "Click the mitochondria to generate ATP. Click organelles to learn about them."

Deliverable: interactive cell, click-to-learn mode, ATP mechanic grounded.

---

### Phase 2 — Central Dogma Chain
_Nucleus → mRNA → Ribosome → Protein_

- Game enters **Interphase — G1** automatically after load
- **Click nucleus** (costs ATP): fires mRNA particle toward rough ER ribosomes (existing `spawnDirected`)
- **Click rough ER** (costs ATP, requires mRNA present): generates a folded-protein token — a small draggable circle that floats near the RER
- **Drag protein → Golgi**: protein snaps to Golgi, packaging animation fires, produces a "packaged protein" token
- Protein tokens are canvas objects with drag state (`isDragging`, `heldBy: 'mouse'|'touch'`)
- Mobile: `touchstart`/`touchmove`/`touchend` on protein tokens; prevent default scroll

Deliverable: the nucleus → RER → Golgi pipeline works end-to-end on desktop and mobile.

---

### Phase 3 — Organelle Building & Endocytosis
_Packaged proteins duplicate organelles; endocytosis as alternate protein source_

**Organelle duplication:**
- Drag packaged protein from Golgi to a target organelle (Golgi itself, lysosomes, mitochondria, SER, vacuole)
- On drop: duplication progress fills; at 100% a second copy of that organelle fades in next to the original
- `cell-data.js` defines `DUPLICATE_TARGETS`: which organelles need to be doubled and how many proteins each takes
- Progress ring drawn around each duplicatable organelle showing fill %

**Endocytosis:**
- Click cell membrane: a protein blob appears outside, an endocytic vesicle wraps it and moves inward
- Vesicle must be dragged to a lysosome → digest animation → free protein token released
- Free protein tokens can substitute for Golgi-packaged ones for organelle building

**HUD addition:** small checklist panel (collapsible on mobile) showing duplication status of each organelle.

Deliverable: all organelle-building paths functional; students can fully double the required organelles.

---

### Phase 4 — S Phase & DNA Synthesis _(bridge phase)_
_Short automated sequence between G1 and G2_

- When all G1 organelles are duplicated: notification fires — "S Phase: DNA replication beginning"
- Automated animation: DNA strand in nucleus duplicates (two interleaved helices animate apart)
- Brief ATP drain — student clicks mitochondria to keep ATP up during this phase
- No drag interaction required; just watch + keep ATP up
- On completion: transitions automatically to G2 (Phase 3 organelle-building continues for remaining organelles if not done)

Deliverable: S Phase communicates to students that DNA duplication is a distinct step.

---

### Phase 5 — Mitosis Sequence
_Triggered when all organelles doubled and S Phase complete_

"Begin Mitosis" button appears. Timer starts on click.

Sequential animations, each auto-advancing after a brief pause (or with a "Next →" nudge button):

| Step | What happens on canvas |
|------|------------------------|
| Prophase | Chromatin in nucleus condenses into visible chromosomes (drawn as X shapes); nuclear envelope fades |
| Metaphase | Spindle fibres draw in from poles; chromosomes line up along cell equator |
| Anaphase | Sister chromatids pulled to opposite poles; spindle fibres shorten |
| Telophase | Two nuclear envelopes reform around each pole; chromosomes decondense back to chromatin |
| Cytokinesis | Cell membrane pinches inward at equator; duplicated organelles animate to their respective daughter cells; two new cells separate |

Win state: confetti burst, timer displayed ("You completed the cell cycle in X:XX!"), share/reset buttons.

Deliverable: full mitosis animation sequence with correct biology; win screen with timer.

---

## Data structures (cell-data.js)

```js
// Organelles to duplicate, proteins required per stage
const DUPLICATE_TARGETS = [
  { id: 'golgi',    name: 'Golgi Apparatus', proteinsNeeded: 3 },
  { id: 'lyso_a',   name: 'Lysosome',        proteinsNeeded: 2 },
  { id: 'mito_a',   name: 'Mitochondrion',   proteinsNeeded: 4 },
  { id: 'ser',      name: 'Smooth ER',        proteinsNeeded: 2 },
  { id: 'vac_lg',   name: 'Vacuole',          proteinsNeeded: 2 },
];

// Game phases
const PHASES = ['G1', 'S_PHASE', 'G2', 'MITOSIS', 'WIN'];
```

---

## Mobile considerations

- All drag interactions use pointer events (or parallel mouse + touch handlers)
- Protein tokens need a minimum tap-target size of ~44px
- Checklist HUD collapses to an icon on narrow viewports
- Mitosis animations use canvas (no DOM drag needed in that phase)
- No pinch-zoom needed — cell fits in viewport; organelles are large enough targets

---

## Out of scope (for now)

- Save/resume (no localStorage game state)
- Multiple cell types (animal only for now)
- Plant cell vacuole / chloroplast paths
- KAN-34 magnetic circuit stuff — unrelated
