# Sci Wolf — Jira Work Plan
_KAN-1 through KAN-38 · Last synced 2026-03-31_

> Statuses reflect the live Jira board as of today's sync.
> KAN-2 and KAN-3 do not exist on the board.
> When starting a task, move it to 'In Progress'
> Comment on completion of a task, beginning your comment with "Claude Edits:"

---

## Done ✅

| Ticket | Summary |
|--------|---------|
| KAN-1  | Strength of Attraction |
| KAN-4  | Remove vibration / shoot-apart bug |
| KAN-5  | Neutral & charged object attraction |
| KAN-6  | Colin sends clip art PNGs |
| KAN-7  | Replace emojis with clip art |
| KAN-8  | Reset button on static sim |
| KAN-12 | Mild / Medium / Spicy difficulty picker |
| KAN-13 | Hide atom count toggle |
| KAN-14 | BCE cumulative timer |
| KAN-15 | BCE equation randomizer |
| KAN-17 | Colin sent Jira API to Emma |
| KAN-21 | Show Orbitals Past Element 20 (Three.js WebGL viewer + Bohr↔Orbital toggle) |
| KAN-22 | Drag Object Along Normal (Ray Diagrams) |
| KAN-23 | Fix Ray Diagram Mobile UI |
| KAN-24 | Toggle Labels on Ray Diagram |
| KAN-25 | pH Indicator Concentration Calculator |
| KAN-27 | Switch to New Wolf Logo _(Colin)_ |
| KAN-30 | Create a Moon Phase Simulation |

---

## Prioritized Plan of Attack

### 🔴 Priority 1 — Light / Dark Mode Cross-App _(Emma)_

#### KAN-26 — Light and Dark Mode
**Status: In Progress**
Theme toggle live on `index.html`, `ph.html`, `bohr.html`, `equationbalancer.html` via CSS custom properties + `localStorage`.

**Remaining work (Colin's comment, Mar 26):**
- Theme is not persisting when navigating to a new sim tab — `localStorage` read on load is not firing correctly across pages
- `raydiagram.html` and `static.html` still use hardcoded hex colours and need conversion to CSS custom properties before light mode can apply

High priority: cross-cutting, visible on every page, and close to done.

---

### 🟠 Priority 2 — Static Electricity Overhaul _(Emma)_

KAN-32 is the authoritative spec for all charge-indicator work. KAN-9, KAN-10, KAN-11, and KAN-28 are superseded by it.

#### KAN-32 — Charges on Electroscope, Objects, and Pith Ball
**Status: To Do**

Full spec from Colin (Mar 26):
- **Protons are fixed at all times** — only electrons animate
- Charges should sit *on* the objects, not swirl around them
- **Electroscope — negative object approaches:** electrons flee to leaves → leaves open; protons stay on head
- **Electroscope — negative object touches:** electrons transfer onto electroscope; net negative charge keeps leaves open
- **Electroscope — positive object approaches:** electrons flood the head; net protons remain on leaves → leaves open
- **Electroscope — positive object touches:** electrons jump onto the charged object; net positive charge keeps leaves open via proton-proton repulsion
- All draggable objects: show electrons gaining/losing (protons never leave); neutral objects show balanced ±
- Pith ball spec (from KAN-11): hangs from retort stand. Neutral → attracted to any charge (induction). Same-sign → repels. Opposite-sign → attracts. Charge by contact on touch.

> ⚠️ KAN-9, KAN-10, KAN-11, KAN-28 can be closed once KAN-32 is complete.

---

### 🟡 Priority 3 — Orbital Highlighter _(Emma)_

#### KAN-36 — Orbital Highlighter
**Status: To Do**

Spec from Colin (Mar 29):
- When cursor hovers over an orbital notation token in the electron config string at the bottom of the 3D viewer (e.g. `1s²`, `4f¹⁴`), highlight that orbital's probability cloud and make all others much more translucent
- Should work on both desktop (hover) and mobile (finger drag/tap)

---

### 🟢 Priority 4 — Circuit Review Fixes _(Emma)_ — needed to close KAN-29

KAN-29 is currently **In Review**. Colin's review generated these two blockers:

#### KAN-35 — Bulb/LED Brightness Scales with Voltage & Resistance
**Status: To Do**

Spec from Colin (Mar 29):
- When circuit is closed, the glow of a lightbulb or LED should grow brighter/dimmer as voltage increases/resistance decreases (and vice versa)

#### KAN-34 — Wire Snapping (usability fix)
**Status: To Do**

Spec from Colin (Mar 29):
- Wires currently have trouble connecting to each other and to components, making it hard to close a circuit
- Wires should "snap" to the nearest endpoint or component terminal when dragged close

> Note: KAN-34's ticket title also mentions a "magnetic aspect to circuit components" — that is a separate, longer-horizon feature (see Priority 8).

---

### 🔵 Priority 5 — BCE Fixes & Expansion _(Emma)_

Three tickets; tackle in order.

#### KAN-33 — Fix Pre-Balanced Mild Equations
**Status: To Do**

Spec from Colin (Mar 26):
- Several mild equations are already balanced at the start — students just toggle up then down with nothing to do
- Replace all pre-balanced mild equations with ones that require changing at least one coefficient

#### KAN-16 — Bigger Equation Bank
**Status: In Progress**
Current bank too thin for repeat student use. Needs significantly more equations, especially at Medium and Spicy.

#### KAN-37 — Spicy Equation Diversity
**Status: To Do**

Spec from Colin (Mar 29):
- Spicy tier is currently dominated by combustion equations
- Add variety: redox, double displacement, synthesis, decomposition, etc.

---

### 🟣 Priority 6 — pH Mobile Overhaul _(Emma)_

#### KAN-31 — pH Simulation Mobile Improvements
**Status: To Do**
The well plate is completely invisible on mobile, and drag interactions (placing samples into wells, dragging strips) are entirely non-functional. Requires a mobile layout rework: sidebar/tray UI to keep the well plate visible, and touch-based drag support for samples and strips.

Significant rework — scope carefully before starting.

---

### ⚫ Priority 7 — Circuit Magnetic Components _(Emma)_

#### KAN-34 — Magnetic Aspect of Circuit Components _(longer-horizon)_
**Status: To Do**
The title feature: add a magnetic aspect to relevant components in the circuit simulation. No spec yet beyond the ticket title — needs scoping discussion with Colin before starting.

---

### ⬜ Priority 8 — Inner Life of a Cell _(Emma)_

#### KAN-38 — Inner Life of a Cell
**Status: To Do**

Full spec from Colin (Mar 31) — this is a ground-up revamp of an existing cell sim:

**Core concept:** Replace WASD movement with point-and-click. Students progress through the full cell cycle and "win" by completing mitosis, tracked by a timer.

**Game loop:**
1. **ATP generation** — click mitochondria to burn glucose and generate ATP; can be clicked at any time to replenish ATP
2. **Transcription** — click nucleus to create mRNA transcripts that travel to ribosomes on the rough ER (burns ATP, along with all tasks)
3. **Translation** — click ribosomes on rough ER to generate protein; protein is folded
4. **Golgi packaging** — drag folded proteins to the Golgi apparatus (must work on mobile touch)
5. **Organelle building** — packaged proteins dragged to organelles to duplicate them (Golgi, lysosomes, mitochondria, others); burning ATP per action
6. **Endocytosis** — alternate path to gain more proteins; digested in lysosome before use
7. **G1/G2 prep** — duplicate all organelles; notification fires when ready for mitosis
8. **Mitosis sequence** — individual animations for DNA synthesis → prophase → metaphase → anaphase → telophase → cytokinesis; duplicated organelles distribute to each daughter cell; cells split apart
9. **Win state** — timer shows total time to complete the cell cycle

Large, complex sim — see [CellSimScoping.md](CellSimScoping.md) for full phase plan and architecture.

**Build phases (multi-file: `cell.html` + `cell-data.js` + `cell-draw.js` + `cell-game.js`):**
1. **Phase 1** — Click foundation: remove WASD, mitochondria click for ATP, all organelles clickable
2. **Phase 2** — Central dogma chain: nucleus → mRNA → RER → drag protein → Golgi
3. **Phase 3** — Organelle building + endocytosis: drag packaged proteins to duplicate organelles
4. **Phase 4** — S Phase bridge: automated DNA duplication animation
5. **Phase 5** — Mitosis sequence: prophase → metaphase → anaphase → telophase → cytokinesis → win screen with timer

---

## Deployment Milestones (Epics)

| Ticket | Milestone | Status |
|--------|-----------|--------|
| KAN-18 | Trial in Colin & Emma's classes | To Do |
| KAN-19 | Trial department-wide | To Do |
| KAN-20 | Public launch (Ontario focus) | To Do |

KAN-18 is the natural next gate — realistic once Priority 1–3 items are stable.
