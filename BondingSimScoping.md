# Bonding Simulator — Scoping Doc

_SNC2D/SNC2P · Ionic, Covalent & Polyatomic Bonding · Revised 2026-04-15_

---

## Testing Harness

| Test case             | Expected result                                     | Current Result  |
|-----------------------|-------------------------------------------------------|-----------------|
| H + H                 | H₂, nonpolar covalent, BO=1                           | works
| H + O + H             | H₂O, polar covalent, 2 single bonds                   | works
| H + N + H + H         | NH₃ (ammonia), polar, 3 single bonds                  | refers to as H3N, otherwise works
| C + 4H                | CH₄ (methane), nonpolar, 4 single bonds               | works
| N + N                 | N₂, nonpolar, BO=3                                    | works
| C + 2O                | CO₂, polar, 2 double bonds                            | works
| C + C + 6H            | C₂H₆ (ethane), 7 single bonds                         | works
| Na + Cl               | NaCl, ionic, stoich 1:1                               | works
| 2Na + O               | Na₂O, ionic, stoich 2:1                               | works
| 2Al + 3O              | Al₂O₃, ionic, stoich 2:3                              | works
| NaCl + KBr (separate) | Both complete independently, info per-component       | works. can click back and forth
| H + H + H             | Third H can't bond (valence full), warning shown      | forms H2, rejects third H, no warning shown
| O + O + O             | O₃ (ozone), correct bond orders                       | Referred to incorrectly as trioxidane
| H+Cl-C-C-Cl+H         | 1,2-dichloroethene					| works!
| [benzene]		  | C6H6 (benzene)					| works!
| [glucose]             | C6H12O6 (glucose)					| works!
| Na + [OH palette]     | NaOH — full structural model, ionic bond shown        | works!
| Ca + [NO3 palette]    | Ca(NO₃)₂ —full model with visible N=O / N–O bonds     | works!
| [NH₄ palette] + Cl    | NH₄Cl — structural model with visible N–H bonds       | works!
| Build N+3O on canvas  | NO₃⁻ recognized, promotable to polyatomic group       | works!
| Sodium phenoxide      | Phenol created, bonds to cation                       | Results in nonsense "sodium hydrogen carbon oxide"

**Known bugs:**
- Canvas size weirdness when expanding/contracting periodic table
- Multivalent picker appears/disappears frustratingly

---

## Current state (as of 2026-04-15)

bonding.html is a single-file canvas 2D app, no external deps. Full periodic table (all 118 elements, Z=1–118), drag-and-drop atom placement, ionic transfer animation, covalent sharing animation, molecular graph engine with bond-order solver, and structural polyatomic ions.

**What works well:**
- Binary ionic compounds (NaCl, MgO, CaCl₂, Al₂O₃) — stoichiometry, naming, animation
- d-block simplified cation model with Stock nomenclature (Fe(II) chloride, etc.)
- Simple/complex covalent molecules (H₂ through glucose) with correct bond orders
- Alkane/alkene/alkyne pattern recognition (methane → decane)
- Common-name lookup for ~17 well-known formulas; PubChem lookup for the rest
- ΔEN bar, dipole arrows, exception messages, theme toggle
- **Polyatomic ions as real atom clusters** — palette ions (NO₃⁻, OH⁻, NH₄⁺, SO₄²⁻, etc.) spawn as pre-bonded atom groups with visible covalent structure, charge badge, and correct ionic bonding to metals
- **Build-from-scratch polyatomics** — drawing N+3O detects NO₃⁻ match and offers promotion to charged group
- **All 118 elements** — periods 5–7 (d-block simplified, f-block lanthanide/actinide rows); collapsible table (periods 1–4 default, "Show all elements ▾" toggle expands to full table)

**What's broken / not yet implemented:**
- **NH₃ naming:** Molecular graph reports H₃N (Hill order) instead of NH₃
- **Ozone naming:** O₃ identified as "trioxidane" rather than "ozone"
- **Valence-full warning missing:** Placing a third H near H₂ silently rejects it; no user-facing message
- **Organic naming:** Anything beyond alkane/alkene/alkyne patterns and the ~17 hard-coded names produces nonsense (e.g. sodium phenoxide → "sodium hydrogen carbon oxide")
- **Canvas resize jank:** Toggling the periodic table expand/collapse has visual weirdness
- **Multivalent picker UX:** Picker for d-block elements appears/disappears unexpectedly on hover

---

## Plan A — Polyatomic Ions as Real Atom Clusters ✓ DONE

Replaced phantom `Z:-1` atoms with real atom clusters. Key pieces:
- `POLY_PALETTE` entries gained `topology: { atoms, central, bonds }` describing internal structure
- `spawnPolyGroup()` now places N real atoms in trigonal/tetrahedral geometry and instantly bonds them
- Bond-order solver extended with `netCharge` on components; charged electron count drives correct bond orders (NO₃⁻ gets one double bond + two singles; resonance noted in info panel)
- Bracket/badge rendering replaced with translucent convex-hull enclosure + superscript charge
- Ionic bonding proximity test updated to nearest atom in group rather than phantom centroid
- Build-from-scratch: `findGroupableCandidates()` prompts "This looks like nitrate — add as ion?"

---

## Plan B — All 118 Elements ✓ DONE (Steps 1–3)

- **Data:** All 118 elements in `ELEMENTS[]` with Z, sym, name, period, group, valence, EN, ox[], cat. Lanthanides/actinides have `fblock` + `fblockCol` fields; d/f-block use `simplified:true`.
- **Layout:** `elemRect()` routes f-block elements to separate rows below the main 7-period grid; dashed `*` placeholder at group 3 for periods 6–7. `recomputeLayout()` recalculates all layout constants on toggle.
- **Collapsible UI (Option A):** Default shows periods 1–4. Centered "Show all elements ▾" button expands to all 118 elements + f-block rows; canvas height grows from 710→860px.
- **New categories:** `lanthanide` (fuchsia tiles) and `actinide` (teal tiles) added to `CAT_STYLE`.

---

## Plan C — Organic Naming

Removed this from scope, replaced with PubChem lookup.

---

## Plan D — "Toggle 3D Shape" (VSEPR Geometry + Wedge/Dash Rendering)

### Goal

Add a **"Toggle 3D Shape"** button that:
1. Identifies the VSEPR geometry of each central atom in the selected molecule
2. Repositions all atoms to match the canonical textbook geometry (correct bond angles, uniform bond lengths)
3. Renders bonds using **wedge** (solid triangle) and **dash** (hashed lines) conventions to convey 3D depth on the 2D canvas
4. Acts as a "make this messy sketch look like a textbook 3D diagram" button

Toggling off restores the original freehand positions.

---

### Design Decision: Procedural VSEPR (not PubChem 3D)

PubChem does offer 3D conformer data via `record_type=3d`, but procedural VSEPR is the right choice here:

- **Works instantly and offline** — no network call, no 404 for exotic/partial molecules
- **Produces canonical textbook shapes** — hardcoded per geometry, not dependent on viewing angle
- **Wedge/dash assignment is deterministic** — each shape has a fixed, well-known representation
- **No atom-order reconciliation needed** — PubChem canonicalizes SMILES atom order, which would require a graph isomorphism solver to map back to canvas atoms
- **The app already computes the inputs** — lone pairs (`loneA`/`loneB` from `classifyBond`) and bond orders are already calculated

PubChem 3D could be added later as a "verify your prediction" feature for advanced students, but it shouldn't drive the core geometry engine.

---

### VSEPR Geometry Reference

Geometry depends on **steric number** (bonding pairs + lone pairs around a central atom):

| Steric # | Lone Pairs | Shape | Bond Angle(s) | Example |
|----------|------------|-------|---------------|---------|
| 2 | 0 | Linear | 180° | CO₂, BeCl₂ |
| 3 | 0 | Trigonal planar | 120° | BF₃, SO₃ |
| 3 | 1 | Bent | ~117° | SO₂, O₃ |
| 4 | 0 | Tetrahedral | 109.5° | CH₄, CCl₄ |
| 4 | 1 | Trigonal pyramidal | ~107° | NH₃, PCl₃ |
| 4 | 2 | Bent | ~104.5° | H₂O, H₂S |
| 5 | 0 | Trigonal bipyramidal | 90°/120° | PCl₅ |
| 5 | 1 | Seesaw | ~90°/120° | SF₄ |
| 5 | 2 | T-shaped | ~90° | ClF₃ |
| 5 | 3 | Linear | 180° | XeF₂ |
| 6 | 0 | Octahedral | 90° | SF₆ |
| 6 | 1 | Square pyramidal | ~90° | BrF₅ |
| 6 | 2 | Square planar | 90° | XeF₄ |

### Wedge/Dash Convention per Shape

Each shape has a fixed textbook depiction:

| Shape | Plain (in-page) | Wedge (toward viewer) | Dash (away from viewer) |
|-------|------------------|-----------------------|------------------------|
| Linear | 2 | — | — |
| Trigonal planar | 3 | — | — |
| Bent (from trig.) | 2 | — | — |
| Tetrahedral | 2 (left, right) | 1 (bottom-left) | 1 (bottom-right) |
| Trig. pyramidal | 2 (left, right) | 1 (bottom-centre) | — |
| Bent (from tet.) | 2 | — | — |
| Trig. bipyramidal | 3 equatorial | 1 axial | 1 axial |
| Octahedral | 4 equatorial | 1 | 1 |

---

### Step 0 — VSEPR Lookup Table & Classifier

**0A. `VSEPR_SHAPES` constant** (~50 lines)

A map from `"stericNumber_lonePairs"` → shape descriptor:

```js
const VSEPR_SHAPES = {
  '2_0': {
    name: 'linear',
    electronGeometry: 'linear',
    bondAngle: '180°',
    // Unit-vector (x, y) for each bonding position in the canonical textbook view.
    // These ARE the textbook — they define what the snap looks like.
    positions: [{ x: -1, y: 0 }, { x: 1, y: 0 }],
    lonePairPositions: [],
    bondStyles: ['plain', 'plain'],
  },
  '3_0': {
    name: 'trigonal planar',
    electronGeometry: 'trigonal planar',
    bondAngle: '120°',
    positions: [{ x: 0, y: -1 }, { x: -0.866, y: 0.5 }, { x: 0.866, y: 0.5 }],
    lonePairPositions: [],
    bondStyles: ['plain', 'plain', 'plain'],
  },
  '4_0': {
    name: 'tetrahedral',
    electronGeometry: 'tetrahedral',
    bondAngle: '109.5°',
    positions: [
      { x: -0.7, y: -0.2 },   // left  — plain
      { x:  0.7, y: -0.2 },   // right — plain
      { x: -0.35, y: 0.55 },  // front-left — wedge
      { x:  0.35, y: 0.55 },  // back-right — dash
    ],
    lonePairPositions: [],
    bondStyles: ['plain', 'plain', 'wedge', 'dash'],
  },
  '4_1': {
    name: 'trigonal pyramidal',
    electronGeometry: 'tetrahedral',
    bondAngle: '~107°',
    positions: [
      { x: -0.7, y: 0.15 },   // left  — plain
      { x:  0.7, y: 0.15 },   // right — plain
      { x:  0,   y: 0.65 },   // front — wedge
    ],
    lonePairPositions: [{ x: 0, y: -0.7 }],
    bondStyles: ['plain', 'plain', 'wedge'],
  },
  // ... etc for all rows in the table above
};
```

**0B. `classifyVSEPR(atom, bonds)` function** (~25 lines)

```
1. Filter atom.bonds to covalent bonds only
2. Count bonding partners (number of unique partner atoms)
3. Compute lone pairs from valence electrons:
     lonePairs = (valenceElectrons − Σ bondOrders) / 2
   NOTE: The per-bond loneA/loneB values from classifyBond() describe the
   lone pairs relative to ONE bond. For VSEPR we need the global count
   considering ALL bonds. Derive from first principles using elem.valence.
4. stericNumber = bondingPartners + lonePairs
5. Return VSEPR_SHAPES[stericNumber + '_' + lonePairs] + ordered partner list
```

**Gotcha — lone pair calculation:** The existing `loneA`/`loneB` in each bond's `result` are computed per-bond (lines 795–802: `calcLone(a, bondOrder)`). These assume the atom is in a single bond. For VSEPR we need:
```js
const totalBondingE = atom.bonds
  .filter(id => bondById(id).result.type !== 'ionic')
  .reduce((sum, id) => sum + (bondById(id).result.bondOrder || 1), 0);
const lonePairs = Math.max(0, (atom.elem.valence - totalBondingE) / 2);
```

---

### Step 1 — Toggle Button & State

**1A. Toggle button** (~15 lines HTML + ~10 lines CSS)

Add inside the info panel area:
```html
<button id="toggle-3d" class="toggle-3d-btn" style="display:none">
  Toggle 3D Shape
</button>
```

Style: small pill button, muted colour, sits below the bond-info text. Active state gets a highlight border.

**1B. State variables** (~5 lines)

```js
let show3DShape = false;               // global toggle state
const savedPositions = new Map();       // atomId → { x, y }
const bondStyleOverrides = new Map();   // bondId → 'plain' | 'wedge' | 'dash'
let shape3DLabel = null;                // e.g. 'tetrahedral' for info panel
```

**1C. Save/restore logic** (~20 lines)

Toggle ON:
```
1. Identify selected component (reuse existing connected-component logic)
2. For each atom: savedPositions.set(atom.id, { x: atom.x, y: atom.y })
3. Compute VSEPR layout (Step 2)
4. Start snap animation (Step 4)
5. show3DShape = true
```

Toggle OFF:
```
1. For each atom: read savedPositions, set as animation target
2. Animate back to original positions
3. Clear bondStyleOverrides
4. show3DShape = false
```

---

### Step 2 — Geometry Layout Engine

**2A. Find the central atom** (~15 lines)

For the selected connected component:
```
1. Filter to covalent bonds only
2. For each atom, count its covalent bond partners (degree)
3. Central = atom with highest degree
4. Tie-break: lower electronegativity wins (metals more central)
5. For chains (all degree ≤ 2): pick the middle atom
```

**2B. Compute target positions for a single centre** (~40 lines)

```
1. Get VSEPR shape from classifyVSEPR(centralAtom)
2. Place centralAtom at centroid of current component positions
   (preserves approximate screen location)
3. BOND_LENGTH = 90px (matches existing BOND_R ≈ 95)
4. Sort bonded partners to assign them to shape positions:
   - Heavier / more-bonded atoms get the in-plane positions (plain bonds)
   - Lighter terminal atoms (H) get wedge/dash positions
   This matches textbook convention (e.g., in CH₂Cl₂ the Cls are in-plane)
5. For each partner i:
     target.x = central.x + shape.positions[i].x × BOND_LENGTH
     target.y = central.y + shape.positions[i].y × BOND_LENGTH
6. Store bondStyleOverrides for each bond
```

**2C. Multi-centre molecules** (~30 lines)

For molecules like ethanol (CH₃CH₂OH) with multiple non-terminal atoms:
```
1. Walk the molecular graph; identify every atom with ≥2 covalent bonds
   as a local VSEPR centre
2. Process centres in BFS order from the "most central" atom outward
3. For each centre: apply VSEPR geometry locally
   - The bond connecting to the already-placed parent atom is fixed
     (direction toward parent determines rotation of the local shape)
   - Remaining positions filled by unplaced partners
4. After all centres placed, run 5–10 iterations of overlap nudging:
   - If two non-bonded atoms are closer than 2 × R_ATOM, push apart
```

---

### Step 3 — Wedge & Dash Bond Rendering

**3A. Extend `drawBond()`** (~15 lines of dispatch logic)

In the existing `drawBond()` function (currently lines 2429–2442), add at the top of the covalent branch:
```js
if (show3DShape && bondStyleOverrides.has(bond.id)) {
  const style = bondStyleOverrides.get(bond.id);
  if (style === 'wedge') return drawWedgeBond(ctx, aX, aY, bX, bY, bo);
  if (style === 'dash')  return drawDashBond(ctx, aX, aY, bX, bY, bo);
  // 'plain' falls through to existing line-drawing code
}
```

**3B. `drawWedgeBond(ctx, x1, y1, x2, y2, bondOrder)`** (~30 lines)

Draws a filled triangle (narrow at central atom, wide at peripheral atom):
```
1. Direction vector: dx = x2−x1, dy = y2−y1, len = hypot
2. Perpendicular unit: nx = −dy/len, ny = dx/len
3. Narrow half-width at (x1,y1): 1.5px
4. Wide half-width at (x2,y2): 7px
5. Four corners of the trapezoid:
     topLeft  = (x1 + nx×1.5, y1 + ny×1.5)
     topRight = (x1 − nx×1.5, y1 − ny×1.5)
     botLeft  = (x2 + nx×7,   y2 + ny×7)
     botRight = (x2 − nx×7,   y2 − ny×7)
6. ctx.beginPath(); moveTo → lineTo × 3 → closePath; ctx.fill()
7. For BO=2: draw two wedges offset ±3px perpendicular
8. For BO=3: three wedges at offsets −5, 0, +5
```

**3C. `drawDashBond(ctx, x1, y1, x2, y2, bondOrder)`** (~25 lines)

Draws a series of perpendicular lines inside a triangular envelope:
```
1. Same direction/perpendicular vectors as wedge
2. N = 7 cross-lines (adjustable)
3. For i = 0..N−1:
     t = (i + 0.5) / N                    // position along bond (0→1)
     cx = lerp(x1, x2, t)                 // centre of cross-line
     cy = lerp(y1, y2, t)
     halfW = lerp(1.5, 7, t)              // envelope widens
     draw line from (cx + nx×halfW, cy + ny×halfW)
                  to (cx − nx×halfW, cy − ny×halfW)
4. Line width: 1.5px, colour matches bond colour
5. For BO=2/3: offset as with wedge
```

---

### Step 4 — Animated Transition

**4A. Smooth snap** (~25 lines)

Don't teleport atoms — lerp them over ~400ms:
```js
// On each atom in the component:
atom._snapFromX = atom.x;  atom._snapFromY = atom.y;
atom._snapToX   = targetX; atom._snapToY   = targetY;
atom._snapT     = 0;

// In gameLoop, before drawAll():
for (const atom of stageAtoms) {
  if (atom._snapT !== undefined && atom._snapT < 1) {
    atom._snapT = Math.min(1, atom._snapT + dt / 0.4);  // 400ms
    const ease = atom._snapT < 0.5
      ? 2 * atom._snapT * atom._snapT
      : 1 - Math.pow(-2 * atom._snapT + 2, 2) / 2;  // easeInOutQuad
    atom.x = atom._snapFromX + (atom._snapToX - atom._snapFromX) * ease;
    atom.y = atom._snapFromY + (atom._snapToY - atom._snapFromY) * ease;
  }
}
```

Bond styles switch at t=0.5 (halfway through the animation) for a clean visual transition.

**4B. Lock interaction** (~10 lines)

While `show3DShape` is active for a component:
- Prevent dragging individual atoms within the 3D-viewed component
- Allow dragging the whole molecule as a rigid body (translate all atoms equally, like polyatomic group dragging)
- Disable bonding snap for atoms in the 3D-viewed component
- Right-click removal of any atom in the component should also toggle off 3D mode

---

### Step 5 — Lone Pair Lobes (Visual Enhancement)

**5A. Draw translucent lone pair lobes** (~30 lines)

In textbook 3D diagrams, lone pairs are shown as diffuse lobes occupying their tetrahedral/trigonal positions. When `show3DShape` is active:

```
1. Read shape.lonePairPositions (directions where lone pairs sit)
2. For each lone pair direction (dx, dy):
     lobeX = central.x + dx × BOND_LENGTH × 0.6  (shorter than bonds)
     lobeY = central.y + dy × BOND_LENGTH × 0.6
3. Draw as translucent teardrop/ellipse:
   - Radial gradient: dense near atom centre, fading at tip
   - Colour: match electron dot colour at ~25% opacity
   - Shape: ellipse rotated along the lobe direction, ~15px wide × 35px long
```

This visually explains *why* the shape is what it is (e.g., NH₃ is pyramidal because the fourth tetrahedral position is a lone pair).

---

### Step 6 — Info Panel Integration

**6A. Display geometry info** (~15 lines)

When `show3DShape` is active, append to the info panel:

```
Molecular geometry: trigonal pyramidal
Electron geometry: tetrahedral
Bond angle: ~107°
```

Use the `electronGeometry` and `name` fields from `VSEPR_SHAPES`. The distinction between electron geometry and molecular geometry is a key teaching point.

**6B. Toggle button visibility** (~10 lines)

Show the toggle button only when:
- A covalent molecule is selected (not ionic-only)
- At least one atom in the component has ≥2 covalent bonds (otherwise geometry is trivially linear or just a single bond)
- All bonds in the component are in `phase === 'formed'` (not mid-animation)

---

### Build Order & Checkpoints

| Step | What | ~Lines | Depends on | Checkpoint |
|------|------|--------|------------|------------|
| 0A | `VSEPR_SHAPES` table | 50 | — | |
| 0B | `classifyVSEPR()` | 25 | 0A | Can log shape names to console |
| 1A | Toggle button HTML/CSS | 25 | — | Button visible in UI |
| 1B–C | State + save/restore | 25 | 1A | Toggle saves/restores positions (no geometry yet) |
| 2A | Central atom finder | 15 | 0B | |
| 2B | Single-centre layout | 40 | 0A, 2A | **CH₄, NH₃, H₂O snap to correct shapes** |
| 3A–C | Wedge/dash rendering | 70 | 2B | **Bonds render with 3D notation** |
| 4A–B | Animation + locking | 35 | 2B, 3A | Smooth transition, no drag in 3D mode |
| | **Core feature complete** | **~285** | | **Test with all molecules in harness** |
| 2C | Multi-centre layout | 30 | 2B | Ethane, ethanol, etc. lay out correctly |
| 5A | Lone pair lobes | 30 | 0A, 2B | Lone pairs visible as diffuse clouds |
| 6A–B | Info panel labels | 25 | 0B, 1A | Shape name + angles in panel |
| | **Full feature** | **~370** | | |

---

### Edge Cases

- **Ionic bonds:** Toggle hidden — VSEPR doesn't apply to ionic compounds
- **Diatomics** (H₂, O₂, N₂): Linear snap; just straighten the bond horizontally. Technically works but not very interesting — could hide toggle for these
- **Polyatomic groups:** Already placed in rough geometry by `spawnPolyGroup`. The 3D toggle should override with proper VSEPR angles (e.g., SO₄²⁻ → tetrahedral with wedge/dash, not the current even-circle layout)
- **Expanded octets** (PCl₅, SF₆): Steric numbers 5–6 need full bipyramidal/octahedral templates. Less common in intro chem but should be supported
- **Resonance** (SO₃, O₃, benzene): Geometry is still determined by steric number. All equivalent bonds stay as plain lines (no wedge/dash distinction among resonance bonds)
- **Hydrogen terminal atoms:** Always terminal. Position determined entirely by the centre they're bonded to. No VSEPR needed for H itself
- **Multiple components on stage:** Toggle applies only to the selected component
- **Very large molecules:** For >10 atoms, multi-centre layout (Step 2C) is needed. For truly large molecules, overlap resolution may get crowded — acceptable limitation for educational scope