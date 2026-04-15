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