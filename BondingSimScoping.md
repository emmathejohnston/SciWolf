# Bonding Simulator — Scoping Doc

_SNC2D/SNC2P · Ionic & Covalent Bonding · Draft 2026-04-02_

---

## What this is

A browser-based bonding simulator where students select two elements from a graphical periodic table and watch the bonding process animated in real time — electron transfer for ionic compounds, electron sharing for covalent. The sim determines bond type, stoichiometry, and bond order from real electronegativity and valence electron data, so the chemistry is accurate without requiring a lookup table of hardcoded compounds.

**Phase 1 scope:** first 18 elements, binary compounds only.

**Filename:** `bonding.html` (single-file, same architecture as `bohr.html` and `ph.html`)

---

## Element data — first 18

The sim's data layer is a single array. All bond-type logic derives from this at runtime.

| Z  | Symbol | Name       | Period | Group | Valence e⁻ | EN (Pauling) | Oxidation states (common) |
|----|--------|------------|--------|-------|------------|--------------|---------------------------|
| 1  | H      | Hydrogen   | 1      | 1     | 1          | 2.20         | +1, −1                    |
| 2  | He     | Helium     | 1      | 18    | 2          | —            | (none)                    |
| 3  | Li     | Lithium    | 2      | 1     | 1          | 0.98         | +1                        |
| 4  | Be     | Beryllium  | 2      | 2     | 2          | 1.57         | +2                        |
| 5  | B      | Boron      | 2      | 13    | 3          | 2.04         | +3                        |
| 6  | C      | Carbon     | 2      | 14    | 4          | 2.55         | +4, −4                    |
| 7  | N      | Nitrogen   | 2      | 15    | 5          | 3.04         | −3, +3, +5                |
| 8  | O      | Oxygen     | 2      | 16    | 6          | 3.44         | −2                        |
| 9  | F      | Fluorine   | 2      | 17    | 7          | 3.98         | −1                        |
| 10 | Ne     | Neon       | 2      | 18    | 8          | —            | (none)                    |
| 11 | Na     | Sodium     | 3      | 1     | 1          | 0.93         | +1                        |
| 12 | Mg     | Magnesium  | 3      | 2     | 2          | 1.31         | +2                        |
| 13 | Al     | Aluminium  | 3      | 13    | 3          | 1.61         | +3                        |
| 14 | Si     | Silicon    | 3      | 14    | 4          | 1.90         | +4, −4                    |
| 15 | P      | Phosphorus | 3      | 15    | 5          | 2.19         | −3, +3, +5                |
| 16 | S      | Sulfur     | 3      | 16    | 6          | 2.58         | −2, +4, +6                |
| 17 | Cl     | Chlorine   | 3      | 17    | 7          | 3.16         | −1, +1, +3, +5, +7        |
| 18 | Ar     | Argon      | 3      | 18    | 8          | —            | (none)                    |

---

## Bond-type classification logic

Runs at the moment two elements are selected. All thresholds are the standard Pauling cutoffs.

```
ΔEN = |EN(A) − EN(B)|

if either element is noble gas (He, Ne, Ar):
    → NO BOND (show inert message)

else if ΔEN > 1.7:
    → IONIC
    → animate electron transfer

else if 0.4 < ΔEN ≤ 1.7:
    → POLAR COVALENT
    → animate electron sharing + dipole arrow

else (ΔEN ≤ 0.4):
    → NONPOLAR COVALENT
    → animate electron sharing, no dipole
```

The 1.7 threshold is a simplification (the actual ionic/covalent boundary is a gradient, not a hard line). This is pedagogically standard at the Grade 10 level and worth stating explicitly in the sim's info panel.

---

## Stoichiometry determination

For **ionic** compounds: balance by charge. Derive from the common oxidation state of each element.
- Na (+1) + Cl (−1) → NaCl (1:1)
- Mg (+2) + O (−2) → MgO (1:1)
- Al (+3) + O (−2) → Al₂O₃ (2:3, cross-multiply method)
- Na (+1) + O (−2) → Na₂O (2:1)

For **covalent** compounds: apply the octet rule with the "8 − valence electrons = bonds needed" heuristic, then confirm the Lewis structure satisfies octets for both atoms (H uses duet).

```
bonds_needed(A) = 8 − valence(A)   [H: 2 − valence(H) = 1]
total shared pairs = bonds_needed(A) + bonds_needed(B) − total valence electrons
bond_order = shared pairs / 2
```

Example: N₂
- bonds_needed(N) = 8 − 5 = 3; total for N+N = 6
- total valence e⁻ = 5+5 = 10
- shared = 6 (3 pairs) → triple bond ✓

Example: H₂O
- bonds_needed(H) = 1, bonds_needed(O) = 2
- Lewis structure: O makes 2 bonds (one to each H), 2 lone pairs on O → this is a ternary compound, out of Phase 1 scope

For binary covalent, Phase 1 always produces one bond between one atom of each element (e.g. HF, CO, N₂, O₂, Cl₂). Stoichiometry 1:1. Bond order varies.

**Known edge cases within first 18 (flagged, not hidden):**

| Compound | Issue |
|----------|-------|
| BF₃, BCl₃, BH₃ | Incomplete octet on B (only 6 electrons) — still forms, just flag it |
| CO | Triple bond despite low ΔEN — classified as polar covalent, bond order 3 |
| NO | Odd-electron molecule (11 valence e⁻ total) — can't satisfy octets cleanly |
| H with metals (LiH, NaH, MgH₂) | H is the anion (H⁻) here, not the cation — flag this in the info panel |
| AlCl₃, AlBr₃ | Al forms 3 bonds; can be treated as ionic (ΔEN 1.55) or polar covalent — right at the threshold. Classify as ionic, note the borderline |
| PCl₅, SF₆ | Hypervalent (expanded octet) — these are binary compounds but require d-orbital involvement; out of Phase 1, show "compound too complex for this sim" message if selected |

---

## UI layout

```
┌──────────────────────────────────────────────────────────────┐
│  PERIODIC TABLE (top half of canvas)                        │
│  Periods 1–3 displayed as a styled grid (18 columns wide)   │
│  Noble gases greyed out. Hover shows: name, EN, valence e⁻  │
│  Selected elements highlighted (up to 2 at a time)          │
│  [Clear] button to deselect                                  │
├──────────────────────────────────────────────────────────────┤
│  BONDING STAGE (bottom half of canvas)                      │
│                                                              │
│  LEFT                 CENTRE              RIGHT              │
│  Atom A               Bond region         Atom B             │
│  (Lewis dot)         (animation)         (Lewis dot)         │
│                                                              │
│  Below stage: formula · bond type · ΔEN · bond order        │
│               dipole arrow (if polar)                        │
└──────────────────────────────────────────────────────────────┘
```

The periodic table and bonding stage share a single canvas. The table sits in the top 40% of the canvas; the bonding stage occupies the bottom 60%.

---

## Periodic table display

- **Layout:** periods 1–3 across 3 rows, columns aligned to standard group positions (H in group 1, He in group 18, gap in periods 2–3 between groups 2 and 13 shown as empty space)
- **Colour coding** (matches chemistry convention):
  - Alkali metals (Li, Na): warm orange
  - Alkaline earth metals (Be, Mg): pale orange
  - Metalloids (B, Si): yellow-green
  - Nonmetals (H, C, N, O, F, P, S, Cl): cyan/blue
  - Noble gases (He, Ne, Ar): grey, dimmed, non-interactive
  - Al: light orange (post-transition metal)
- **Hover tooltip:** element name, electronegativity, valence electrons, typical oxidation states
- **Selection state:** first click = primary element (accent border), second click on a different element = secondary element → trigger bond calculation immediately. Clicking the same element twice = homonuclear bond (H₂, O₂, N₂, Cl₂, etc.)
- **Noble gas click:** show message "Noble gases have a full valence shell and do not form bonds under normal conditions."

---

## Bonding stage — animation phases

All animations run on the canvas using `requestAnimationFrame`. Each phase has a defined duration; students can also click "skip" to jump to the final state.

### Ionic bonding animation

**Phase 1 — Approach (600 ms)**
Both atoms drawn as circles with Lewis dot electrons (coloured dots at N/E/S/W/NE/NW/SE/SW positions). Atoms slide toward each other from opposite sides.

**Phase 2 — Transfer (800 ms)**
The electron(s) being transferred visibly lift off the donor atom and arc across to the acceptor atom. Each transferred electron drawn as a moving particle with a trailing glow. Donor's dot count decreases; acceptor's increases in real time as each electron arrives.

**Phase 3 — Ion formation (400 ms)**
Donor transforms: circle shrinks slightly (outer shell gone), charge label (+n) appears. Acceptor transforms: circle grows slightly, charge label (−n) appears. Electrons shown in full octets.

**Phase 4 — Attraction (600 ms)**
Ions move back together, settling at an appropriate distance. Dashed bond lines appear between them. Formula and name displayed below.

### Covalent bonding animation

**Phase 1 — Approach (600 ms)**
Both atoms drawn with Lewis dots. Atoms approach.

**Phase 2 — Orbital overlap (700 ms)**
The electron clouds (drawn as soft translucent circles) begin to overlap in the centre region. The shared electrons move to the overlap zone.

**Phase 3 — Sharing (500 ms)**
Shared electron pair(s) shown as dots (or line segments) in the centre. Lone pairs settle on outer atoms. Bond order indicated by number of pairs/lines. For triple bond (N₂): three pairs shown. For double bond (O₂): two pairs.

**Phase 4 — Final state**
For polar covalent: δ+ appears on the less electronegative atom, δ− on the more electronegative. A dipole arrow (→ pointing toward δ−) drawn across the bond. The arrow length scales with ΔEN. ΔEN value shown numerically.

### Homonuclear diatomics (H₂, O₂, N₂, F₂, Cl₂)
Same covalent animation. No dipole arrow. "Nonpolar covalent — electrons shared equally" label.

---

## Lewis dot rendering

Each atom is drawn as:
- Circle (radius scales slightly with atomic radius)
- Element symbol centred
- Valence electrons as small filled circles arranged around the perimeter (paired where applicable, following Hund's rule pairing order: first one at each of 4 positions, then pair up)

Electrons use a consistent colour scheme:
- Atom A's electrons: accent colour A (e.g. blue)
- Atom B's electrons: accent colour B (e.g. red)
- When electrons are shared or transferred, their colour blends or changes to show the new ownership

---

## Info panel

Below the bonding stage, a fixed panel shows (updates after animation completes):

```
Formula:        NaCl
IUPAC name:     sodium chloride
Bond type:      Ionic
ΔEN:            2.23  (Na 0.93 · Cl 3.16)
Transfer:       1 electron from Na → Cl
Ions formed:    Na⁺  ·  Cl⁻
---
Formula:        HF
IUPAC name:     hydrogen fluoride
Bond type:      Polar covalent
ΔEN:            1.78  (H 2.20 · F 3.98)
Bond order:     Single (1 shared pair)
Dipole:         δ+ on H · δ− on F
```

For edge cases, a ⚠️ callout appears:
> "BF₃ has only 6 electrons around boron — an incomplete octet. This is a known exception to the octet rule."

---

## Homonuclear / same-element selection

Allow selecting the same element twice. Produces:
- H₂ (single, nonpolar) — "the most abundant molecule in the universe"
- O₂ (double, nonpolar)
- N₂ (triple, nonpolar) — "the triple bond in N₂ is one of the strongest bonds in chemistry — this is why nitrogen gas is so unreactive"
- F₂ (single, nonpolar)
- Cl₂ (single, nonpolar)
- Li₂, Na₂ (single, metallic character — flag as unstable in gas phase)
- Noble gases: show monoatomic, inert message

---

## Bond order summary for Phase 1 compounds

This is the complete expected output for all interesting pairs. Used to validate the runtime logic.

**Ionic (ΔEN > 1.7):**
LiF, LiCl, LiH (H is anion), NaF, NaCl, NaH, NaO (→Na₂O), MgF₂, MgCl₂, MgO, AlF₃, AlCl₃ (borderline), Al₂O₃, AlN (borderline)

**Polar covalent:**
HF (1.78), HCl (0.96), HO (→H₂O — out of Phase 1 scope for H₂O but H+O binary is 1:1 as OH radical — flag), CO (0.89 — triple bond), NO (0.84 — odd electron, flag), SiC (0.65 — single), SF (1.58 — single), SF₂ concept (out of scope for 1:1)

**Nonpolar covalent:**
H₂ (0.00, single), N₂ (0.00, triple), O₂ (0.00, double), F₂ (0.00, single), Cl₂ (0.00, single), NN, CC, CS (0.03 — single/double), PH (0.01 — single), SiH (0.30 — single), CH (0.35 — single)

---

## What "binary compound" means in this sim

The sim models one bond between **one atom of element A and one atom of element B** (or two of the same). This is an accurate representation for:
- All diatomics (H₂, HF, HCl, N₂, O₂, Cl₂, CO, NO)
- All ionic pairs shown in their simplest formula unit (NaCl, MgO, AlN)

For ionic compounds like Na₂O or MgCl₂, the sim shows the simplest repeating unit (one Na donating to O with the "×2" or charge-balance logic clearly labelled) rather than animating two separate Na atoms simultaneously. This is a deliberate simplification — noted in the UI.

---

## Architecture

Single file: `bonding.html`

Internal sections:
```
<style>          — All CSS, same token system as other sims (--accent, --surface, etc.)
<canvas>         — Periodic table + bonding stage
<div #info>      — Info panel below canvas
<script>
  ELEMENTS[]     — Data table (Z, symbol, EN, valence, group, period, color, oxidation states)
  classifyBond() — Returns { type, deltaEN, bondOrder, formula, ions, exceptions }
  drawTable()    — Periodic table renderer
  drawAtom()     — Lewis dot atom renderer (given element + electron state)
  Animator class — Phase-sequenced animation runner
  InfoPanel      — Updates DOM after animation
</script>
```

No external dependencies. Canvas 2D only (no WebGL needed — this is 2D Lewis structures).

Estimated file size: ~700–900 lines. Comparable to `bohr.html`.

---

## Phase breakdown

### Phase 1 — Core bonding engine
_Goal: two elements selected → correct bond type determined → static final state displayed (no animation yet)_

- Periodic table rendered (first 18 elements, colour-coded, hover tooltips)
- Element selection (click one, click another → classify)
- `classifyBond()` logic running correctly for all first-18 pairs
- Lewis dot final state drawn for both atoms in their bonded configuration
- Info panel populated
- Noble gas and edge-case messages
- Light/dark mode

**Deliverable:** scientifically accurate bonding classifier with static display.

---

### Phase 2 — Ionic animation
_Goal: electron transfer animated for ionic pairs_

- Phase-sequenced ionic animation (approach → transfer → ion formation → attraction)
- Electron particle arc (the signature visual of this sim)
- Charge labels animate in
- Works correctly for 1:1, 1:2, 2:3 stoichiometries (Na⁺Cl⁻, Mg²⁺O²⁻, Al³⁺ with appropriate anion)

---

### Phase 3 — Covalent animation + polarity
_Goal: electron sharing animated; dipole shown for polar_

- Phase-sequenced covalent animation (approach → orbital overlap → sharing → final)
- Single, double, triple bond rendering
- Lone pair placement on final atoms
- Dipole arrow + δ+/δ− labels for polar
- ΔEN bar or scale indicator (visual sense of how ionic vs covalent a bond is)

---

### Phase 4 — Polish & edge cases
_Goal: complete, lesson-ready_

- Edge case callouts (incomplete octet, odd-electron, borderline ionic/covalent)
- "Replay" button
- Mobile touch support (tap to select, pinch zoom on periodic table)
- Print/screenshot-friendly static view
- Link from `bohr.html` (natural companion: Bohr diagrams → bonding → next step)
- Added to `index.html` sim card grid
- Added to `standalone.html` as the C1 sim (replaces/supplements the lesson link)

---

## Future expansion (post-Phase 1 scope, for reference)

**More elements (Z = 19–36):**
- Period 4 introduces transition metals and d-orbital electrons — oxidation states become variable (Fe²⁺/Fe³⁺, Cu⁺/Cu²⁺). Manageable with a lookup table of common states. Would unlock FeCl₂/FeCl₃, CuO/Cu₂O, etc.
- Requires expanding the periodic table render to 4 rows + the d-block columns.

**Polyatomic ions (medium complexity):**
- OH⁻, NH₄⁺, SO₄²⁻, CO₃²⁻, NO₃⁻ — treated as pre-built "super-atoms" with known charge and structure
- Unlocks NaOH, Ca(OH)₂, H₂SO₄ etc. without needing to compute multi-centre Lewis structures from scratch
- Reasonable extension: ~10–15 common polyatomic ions as data entries

**VSEPR molecular geometry (high complexity):**
- Requires computing electron pair geometry from lone pair + bonding pair count
- 3D rendering of molecular shape (tetrahedral, trigonal planar, bent, linear, etc.)
- Would integrate naturally with the Bohr/orbital 3D viewer
- Significant work; natural Phase 5 or a separate `vsepr.html`

**Resonance structures:**
- O₃, SO₂, CO₃²⁻, benzene (way out of Phase 1)
- Requires showing multiple equivalent Lewis structures and the resonance hybrid
- Interesting visually (electron delocalisation animation), but Grade 10 doesn't need it

**Organic molecules (long-horizon):**
- Carbon is the only element that forms enough bonds with itself to make this interesting
- Alkanes, alkenes, alkynes would follow naturally from single/double/triple bond logic already in the sim
- True molecular complexity (protein folding, etc.) is where the academic cluster computing begins — the sim can go up to maybe 8–12 atoms before the Lewis structure layout algorithm becomes non-trivial
- Realistic ceiling: straight-chain alkanes up to ~hexane; simple functional groups

---

## SciWolf alignment

| Lesson | This sim | Fit |
|--------|----------|-----|
| C1 — Why Do Atoms Bond? | ⭐ Direct replacement for Bohr-Rutherford in Section 1 | Strong |
| C1 — compound classification task | Sim generates the examples students name and categorise | Strong |
| C2 — reaction types | Bonding context (recognising bond breaking/forming) | Weak / background |

The C1 lesson sketch currently uses `bohr.html` as a proxy — "build sodium, count valence electrons." This sim is the proper tool for C1 and should be linked from there once built. `bohr.html` remains the right tool for the Bohr diagram drawing task; this sim is the next step after students understand valence electrons.

---

## Open questions before build

1. **Stoichiometry display for ionic:** show the full formula unit (Na₂O with "2 Na atoms" in the animation) or show 1:1 with a "×2" annotation? The latter is simpler and avoids having to animate two separate atoms simultaneously.

2. **ΔEN boundary footnote:** the sim classifies bonds using the 1.7/0.4 thresholds. Should it show a continuous "bond character" slider (0% ionic ↔ 100% ionic) in addition to the categorical label? Could be a nice visual for borderline compounds like AlCl₃ (ΔEN 1.55) or HF (ΔEN 1.78).

3. **H as anion in metal hydrides (LiH, NaH):** Grade 10 students find it surprising that H takes a −1 charge here. Worth a flagged callout, or should these just be shown as ionic without calling out H⁻ specifically?

4. **Mobile periodic table:** 18 columns in a small grid will be tight on phone screens. Options: (a) scroll/zoom the table, (b) list view on mobile with search, (c) don't support portrait phone (Chromebook/tablet is the target device anyway).
