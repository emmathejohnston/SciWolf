# Circuit Simulation — Scoping & Roadmap
_KAN-29 · SNC1W Grade 9 Electricity · Last updated 2026-03-28_

---

## Decisions Summary

| Topic | Decision |
|-------|----------|
| Canvas | Free-form, scroll/zoom |
| Sidebar | Far-left, collapsible |
| Save/Share | Yes — attempt URL or localStorage based |
| Components | Wires, battery, bulb, resistor, fuse, switch, voltmeter, ammeter, capacitor, LED, motor |
| Deletion | Double-click: detaches from wire first, then deletes on second double-click. Wires: single double-click deletes. Undo button. |
| Property editing | Post-placement slider (click component to open) |
| Fuse behaviour | Blows above threshold; no fuse = cute fire animation |
| Wiring | Wire tool in sidebar. Click/drag from terminal, wire node, or blank canvas. Every crossing = junction. Wire corners and endpoints are snappable nodes. |
| Wires | Ideal conductors (no resistance). Free endpoints allowed (wire to nowhere). |
| Open circuit | Sad face visual indicator |
| Short circuit | Cute fire animation |
| Physics model | ~~Single-loop Ohm's law~~ → **Union-Find + Gauss-Jordan nodal analysis** (handles series, parallel, and arbitrary topologies) |
| Current display | Live value readout + electron animation speed |
| V=IR | Live readout panel (top-right) + separate student calculation section below canvas (shows rearranged working) |
| Circuit authority | Circuit is always source of truth; student calculator is separate |
| Electron style | Particles along wire, negative → positive (actual electron flow) |
| Parallel electrons | Split visually across branches |
| Electron speed | Updates in real time as sliders move |
| Default values | Realistic (e.g. 100 Ω, 9 V), adjustable by slider |
| Visual style | Illustrated cartoon default; toggle to schematic symbols |
| Dark/light mode | Yes, match existing system |
| Bulb glow | Scales with voltage/current |
| Blown fuse | Visually distinct |
| Grade level | SNC1W Grade 9 |
| Exploration mode | Free exploration (no guided tasks for now) |
| Starter circuit | Pre-built: battery + switch + light bulb |
| Complexity limit | Unlimited |

---

## Project Roadmap

### ✅ Phase 1 — Canvas & Wiring Foundation
_Goal: you can draw a circuit on screen_ — **Complete**

- [x] Blank free-form canvas with scroll and pinch/scroll zoom
- [x] Collapsible left sidebar (component palette)
- [x] Wire tool in sidebar: click/drag from terminal, wire node, or blank canvas; snaps to nearby nodes. Wire corners and free endpoints are snappable nodes (grey dots).
- [x] Junction creation when two wires cross **or touch** (T-junctions, endpoint joins — all detected and electrically connected)
- [x] Undo button (Ctrl/Cmd+Z), 60-step stack
- [x] Double-click to detach component from wire; second double-click deletes it. Double-click on a wire deletes it immediately.
- [x] Pre-built starter circuit loaded on open: battery + switch + light bulb

---

### ✅ Phase 2 — Core Components
_Goal: the essential SNC1W set is buildable_ — **Complete**

- [x] **Battery** — adjustable voltage slider (default 9 V); labelled +/− terminals
- [x] **Wire** — ideal conductor; green glow when live, red on short circuit
- [x] **Switch** — click to open/close; open = breaks circuit (lever angled), closed = connects (lever flat)
- [x] **Light bulb** — glows brighter as current increases; dims to off when circuit is open
- [x] **Resistor** — adjustable resistance slider (default 100 Ω)
- [x] **Fuse** — blows (dashed rectangle + X, looks charred) when current exceeds rated threshold; adjustable rating slider (default 1 A)
- [x] **Open circuit indicator** — 😢 overlaid near open switch (or canvas centre); status badge
- [x] **Short circuit animation** — animated flame at battery; status badge

---

### ✅ Phase 3 — Physics & Electron Animation
_Goal: the circuit is alive_ — **Complete**

- [x] ~~Single-loop~~ **Full nodal analysis** solver (Union-Find + Gauss-Jordan): V = IR per branch, solved on every state change, handles series and parallel circuits
- [x] Electron particles animated along wires, travelling negative → positive (actual electron flow direction)
- [x] Electron speed scales in real time with current (faster = more current); updates every animation frame
- [x] Live V / I / R readout panel — top-right overlay, updates as sliders move or switches toggle; shows selected component's individual values
- [x] **Electrons split proportionally across parallel branches** — per-wire current = min of two endpoint component currents; parallel branches now show individual branch speeds correctly
- [x] **Scrollable V=IR calculation section below the canvas:**
  - "▼ V=IR Calculator" tab at bottom of canvas scrolls down to section
  - Left card: live circuit V / I / R values + worked example showing rearranged equation from circuit
  - Rearranges equation before solving (e.g. I = V ÷ R) based on selected solve-for target
  - Right card: student input — pick solve-for (V / I / R), enter two knowns, click Calculate
  - Shows full step-by-step working; compares student answer against live circuit value

---

### Phase 4 — Visual Polish & Theming
_Goal: it looks and feels like a SciWolf sim_

> **Note:** Some Phase 4 items were built early:
> - [x] Dark / light mode — already implemented (CSS custom properties + localStorage)
> - [x] Component click → slide-out property panel with slider (voltage, resistance, fuse rating)
> - [x] Blown fuse visual — dashed rectangle + X marks

- [ ] Illustrated cartoon component art (default view)
- [x] Toggle to standard schematic symbols (IEC/ANSI) — topbar "📐 Schematic" button; hides glow/electrons, draws clean lines, shows value labels (V, Ω, A)
- [x] Voltmeter and ammeter display live readings on their faces

---

### Phase 5 — Advanced Components
_Goal: extend beyond the SNC1W minimum_

- [x] **LED** — directional; triangle+bar IEC symbol; glows green when forward-biased (anode=b), dark if reversed; light-ray arrows
- [x] **Capacitor** — DC open circuit; two-plate symbol; blue charge fill + voltage reading scales with terminal voltage
- [x] **Motor** — animated spinning arc+arrowhead, speed ∝ current; M label; resistance adjustable via slider
- [x] **Voltmeter** — 1 MΩ, circle with V, live voltage reading displayed above face
- [x] **Ammeter** — 0.001 Ω, circle with A, live current reading displayed above face

---

### Phase 6 — Save & Share
_Goal: students can save and come back, or send a circuit to a friend_

- [ ] Serialize circuit to JSON; store in `localStorage` as autosave
- [ ] "Share" button generates a URL with circuit state encoded (query param or hash)
- [ ] Loading a share URL restores the exact circuit

---

## Open Questions for Later

- Should the student calculator section give feedback if the student's answer is wrong (vs. what the circuit actually computes)?
- Should motors or capacitors affect the V=IR calculation, or are they visual-only for now?
- Is there a reset button to clear the canvas (in addition to the pre-built starter)? _(Clear button exists in topbar — resolve whether it should reload the starter circuit or leave blank)_
- Should the sidebar group components by category (sources / loads / controls / meters)? _(Already done)_
- Per-wire current for electron animation: upgrade from max-in-net heuristic to true branch current for accurate parallel-branch electron speeds?
