# Forest Succession — Scoping Doc

_SBI3U / grade-adjacent · Ecological succession + relationship-to-nature · Drafted 2026-04-16_

---

## Two Learning Goals

The sim currently teaches one. We are scaffolding a second.

1. **Ecological succession** — how bare ground becomes a climax forest over centuries. Players see pioneers give way to early, mid, late, and climax species; learn what each era needs; recognise indicator organisms (lichen → oak; chanterelle → mycorrhiza).

2. **Consciousness-raising: the player's relationship to nature.** Through mechanics, not lectures, the sim invites the player to sit with the question: *what is my stance toward this living place?* The goal is not to deliver an answer but to make the question felt.

Both goals have to coexist. Succession is the "what is happening"; relationship is the "where am I in this." The first goal is well served by the current build. The second is barely begun.

---

## Why This Second Goal, and What Grounds It

This section is context for the implementer. You don't need to memorise the authors; you need to know the design is principled, so that when a step asks for an unusual mechanic ("Listen" instead of "Harvest", a foxglove growing from a grave) you don't flatten it into something more conventional.

### Four framings of the human–nature relationship

The sim should make all four *playable* without declaring a winner:

| Framing | Stance | In-sim signal |
|---|---|---|
| **Dominion over nature** | Nature is resource; I decide. | Cutting, harvesting, removing without replacement. Hunger always restored at cost to the forest. |
| **Stewardship of nature** | Nature is garden; I tend. | Sowing, tending saplings, removing "pests" to favour chosen species. |
| **Part of nature** | No inside/outside. My body is continuous with the forest. | Decomposition, mycelial view, foxglove-from-grave, eating and being eaten. |
| **Nature's dominion over me** | The forest is indifferent. I die, it continues. | Starvation. The forest reaching climax without the player ever acting. Poisoning. Being overgrown. |

Current build only really supports the first (harvest/cut) and gestures at the third (the foxglove-grave). The relationship layer is the missing three-quarters of this matrix.

### Queer ecology, briefly

Sandilands, Mortimer-Sandilands, Gaard, Morton and others critique two moves that mainstream "nature education" tends to make:

1. **The reproductive imperative.** Nature is often framed as organisms striving to reproduce — flowers → fruit → seeds → offspring as the narrative arc of "success." This sidelines clonal organisms, lifelong non-reproducers, fungi with thousands of mating types, sterile hybrids, ancient unchanging ferns, symbiotic beings like lichens that aren't individuals at all. It also exports a heteronormative template onto ecology.
2. **The bounded self.** The human is framed as outside nature, observing it. But your gut is a microbial ecosystem. Oak canopies trade sugar for minerals through fungal networks. Decomposition is not the end of a self; it is a self becoming other selves.

We don't state any of this in-game. We express it through **what the sim notices** — what it shows on the info card, what it lets you do, what it reflects back.

### The hunger bar is load-bearing

The hunger bar already does important philosophical work. It forces interaction: you cannot simply observe, because the body demands. This is the starting kernel. Everything in the relationship layer builds outward from it by asking: *and then what?*

---

## Current State (as of 2026-04-16)

`forest.html` — single-file canvas 2D app, no external deps. ~2060 lines.

**What works:**
- 22 species across 5 succession eras (pioneer → climax), cover-gated colonisation
- Seasonal rendering, year counter, 1×/16×/64× speed, pause
- Hunger bar drains with time; eating edibles restores it; poisonous plants harm
- Info card on click: name, latin, ecological info, edibility, harvest/remove buttons
- Animals (butterfly, rabbit, fox, deer, owl) that appear at appropriate eras
- Death (hunger reaches zero) → grave sinks → decomposed → rebirth, with a foxglove growing from the grave
- Atmospheric particles (pollen, seeds, autumn leaves, snow)

**Testing harness — succession layer (passes):**

| Test case | Expected | Result |
|---|---|---|
| Year 0, bare ground | Lichen + moss only | works |
| Year ~10 | Fireweed, grass, yarrow establish | works |
| Year ~25 | Blackberry and birch visible | works |
| Year ~60 | Hazel, hawthorn, ash begin | works |
| Year ~100 | Oak seedlings; bluebell carpets in spring | works |
| Year ~150 | Chanterelle/fly agaric under trees; holly, ivy | works |
| Player eats dandelion | +8% hunger, plant removed | works |
| Player eats foxglove | Hunger drops, "Poisonous!" | works |
| Player starves | Collapse → grave → foxglove → rebirth | works |

**What is missing (the scope of this doc):**
- No way to give back (sow, tend)
- No way to simply observe (Listen)
- No perspective that is not the upright walking human
- No reflection — the game never notices what kind of player you're being
- Species info describes ecology but not persistence-strategy (reproductive, clonal, symbiotic, immortal)
- Death → rebirth resets; nothing accumulates across lives
- No way to experience the forest as a continuous self (mycelium, canopy)

---

## Testing Harness — Relationship Layer (to be built)

These are acceptance tests for the scoped work. An implementer can verify step-by-step that each plan is landing.

| Test case | Expected result |
|---|---|
| Click any plant | Info card shows a new **"Persistence"** line (reproductive / clonal / symbiotic / ancient / sterile) |
| Info card on edible plant | New **"Listen"** button alongside "Harvest & eat" |
| Click Listen on oak | 4–6 second ambient pause; a passage of first-person text fades in; no hunger restored, no plant removed |
| Player eats berry | New **"seeds"** counter increments (seeds carried = 1) |
| With seeds > 0, click bare ground | **"Sow"** prompt; seed is consumed; a seedling of that species appears |
| Sow blackberry under mature oak canopy | Refused with soft message: "the light here is wrong for this one" |
| Player walks 5 min without harvesting | A passing line of text: "the forest does not need you to continue" |
| Player harvests 10+ plants in one life | A passing line: "the forest feels your hunger" |
| Player dies by starvation | Grave + foxglove (current). Plus: the player's nutrient boost extends ~50px and raises colonisation rate nearby for ~10 years |
| Press `m` while alive | Camera sinks underground; fungal network lines visible between trees; movement continues but rendering inverts |
| Press `m` while dead/decomposing | Seamless transition — decomposition *is* the mycelial view |
| After 3+ death-rebirth cycles | A reflection screen appears once, offers a quiet question, and dismisses without scoring |
| Reach climax forest without any harvest | A single line of text: "the forest arrived without you" |
| Reach climax forest via heavy intervention | A single line: "this is the forest you shaped" |
| All species have non-flower "persistence" entries written | Ferns: spores / ancient. Bracken: clonal rhizome. Lichen: symbiosis. Aspen-equivalent: clonal. Fly agaric: mycorrhizal partner. Ivy: opportunist-evergreen. Holly: dioecious. |

No scoring. No "good ending." The sim reflects; it does not judge.

---

## Plan A — Reciprocity: Sow, Tend, Listen

**Goal:** give the player actions that are not consumption. Today the info-card buttons are "Harvest & eat," "Cut down," "Remove." All subtract. The sim offers no verb that gives or that simply attends.

### Step A1 — Seed inventory (~20 lines)

Add to the `player` object:
```js
seeds: {},   // { speciesId: count }
```

In `eatPlant()`, when a plant with `edible.part` containing "berries", "nuts", "seeds", "keys", or "acorns" is consumed, credit one seed:
```js
if (/berries|nuts|seeds|keys|acorns/i.test(sp.edible.part)) {
  player.seeds[sp.id] = (player.seeds[sp.id] || 0) + 1;
}
```

Cap total seeds carried at 12 across all species (gut limit — keeps it embodied, not a warehouse).

### Step A2 — Seed pouch UI (~30 lines)

Small row of species icons at bottom-left, next to the hunger bar. Each icon shows a count badge. Clicking an icon selects that seed for sowing. Selected icon gets a highlight border. If `total == 0`, hide the pouch entirely.

Icon for each species = a miniature `sp.draw()` at `t = 0.3` on a 24×24 canvas, cached.

### Step A3 — Sow action (~40 lines)

When a seed is selected:
- Cursor changes to a small seed glyph
- Clicking empty ground calls `sow(selectedSp, worldX)`
- `sow()` checks that `cover` is inside `[sp.coverMin, sp.coverMax]` *with a soft buffer of 0.05*. If outside, emit float-text near the click: **"the light here is wrong for this one"** (if cover too high) or **"the soil is not yet ready"** (if too low). Seed is NOT consumed.
- If allowed: `addPlant(sp.id, worldX)`; decrement seed; play a short "sown" particle (two tiny green sparks).
- Clicking anywhere else (UI, plant) with a seed selected does NOT sow — it deselects the seed.

### Step A4 — Tend action (~30 lines)

On the info card for any plant of age `< matureAge * 0.3` (a seedling), add a **"Tend"** button (neutral style). Tending:
- Advances that plant's age by a small jump: `p.age += 0.5`
- Creates a small ring-of-light particle around it
- Adds a "tended" count to `player.gave` (see Plan E)

Tend is not a cheat code — it just lets you nudge. Limit: one tend per plant per year.

### Step A5 — Listen action (~50 lines)

The heart of the plan. On every plant's info card, a **"Listen"** button, placed above "Harvest & eat":

Visuals while listening:
- Overlay dims the edges of the screen to vignette.
- Time speed drops to 0.25× for 5 seconds regardless of setting.
- A slow passage of text — 1 to 3 short lines — fades in over the plant, letter by letter.
- No hunger change, no plant change.

Writing guidelines for the listen-texts (put them on the species object as `listen: [...strings]`, picked at random):
- First person, present tense, sometimes from the plant's point of view, sometimes from neither / both.
- Short. A sentence, maybe two.
- Avoid the "lesson voice." Avoid the word *symbolises*. Avoid the word *majestic*.
- Contradictory lines across a species are welcome — the forest is not a single voice.

Examples to ship with:
```js
SPECIES.oak.listen = [
  "I am slower than you. You will not see what I become.",
  "Two thousand of us live inside me. I do not know their names.",
  "The acorn you stepped on was one of me. I don't mind."
];
SPECIES.lichen.listen = [
  "I am two things. Neither of us alone could live here.",
  "The rock under me is my food. It will be soil in a thousand years."
];
SPECIES.fly_agaric.listen = [
  "What you see is the fruit. I am under your feet, everywhere.",
  "I grow because the birch let me. The birch grows because I let it."
];
```

Write 2–3 lines per species. Keep total prose under ~2 KB.

---

## Plan B — Queer Ecology in the Info Card

**Goal:** the info card currently answers *what is this?* and *can I eat it?* Add a third answer: *how does this being persist through time?* This gently decentres flower-and-seed reproduction as the default story of life.

### Step B1 — Add `persistence` field to every species (~200 lines of content, spread across all species)

Schema:
```js
persistence: {
  strategy: 'sexual' | 'clonal' | 'symbiotic' | 'spore' | 'dioecious' | 'opportunist' | 'ancient',
  note: string  // one short sentence
}
```

Examples (write one for each of the 22 species):
```js
SPECIES.lichen.persistence = {
  strategy: 'symbiotic',
  note: "Not one organism. A fungus and an alga living as one body — a partnership older than plants."
};
SPECIES.fern.persistence = {
  strategy: 'spore',
  note: "No flower, no seed. Reproduces by dust-like spores, a pattern unchanged for 300 million years."
};
SPECIES.bracken.persistence = {
  strategy: 'clonal',
  note: "A whole hillside of bracken can be a single organism, connected underground — one of the oldest and largest living things."
};
SPECIES.holly.persistence = {
  strategy: 'dioecious',
  note: "Holly plants are male or female. A lone holly bears no berries."
};
SPECIES.bluebell.persistence = {
  strategy: 'clonal',
  note: "Spreads mostly by bulb — seed-grown bluebells take five years to flower, and most of a wood is one extended family."
};
SPECIES.fly_agaric.persistence = {
  strategy: 'symbiotic',
  note: "The mushroom is brief. Its true body is a kilometres-wide network entangled with tree roots, trading sugar for minerals."
};
SPECIES.foxglove.persistence = {
  strategy: 'sexual',
  note: "Biennial. The first year it waits low to the ground; the second it sends up a single spike and dies."
};
// ... etc
```

Aim for tonal variety. Not every species gets a Sandilands paragraph. Some are just "sexual — standard flowering plant," written dry. The point is that *the card notices* that persistence is not one thing.

### Step B2 — Render the persistence line on the info card (~15 lines)

Below the edible/poisonous block, insert a muted-text line:
```
Persistence · clonal
A whole hillside of bracken can be a single organism…
```

Colour: `#94a3b8` (slate-400). Smaller font (0.78rem). The key word ("clonal", "symbiotic", etc.) is in a pill-style badge with a per-strategy colour:

| Strategy | Colour |
|---|---|
| sexual | neutral slate |
| clonal | soft teal `#5eead4` |
| symbiotic | soft amber `#fbbf24` |
| spore | soft violet `#c4b5fd` |
| dioecious | soft rose `#fb7185` |
| opportunist | slate |
| ancient | warm grey |

The badge is understated. This is not a Pokédex.

---

## Plan C — The Mycelial View

**Goal:** let the player experience the forest as a continuous underground body. This is the most direct mechanic for "part of nature" and for dissolving self/other.

### Step C1 — Toggle key and camera state (~20 lines)

Press `m` (and a small moon-icon button next to the speed controls) to toggle `mycelialView`.

```js
let mycelialView = false;
let mycelialT = 0; // 0..1 blend factor, animates on toggle
```

Transition takes ~0.8 s. During transition:
- Sky and ground colours blend toward dark (sky → deep indigo `#1a1030`; ground → warm black `#0a0a12`).
- Plants above ground dim to 30% opacity.
- A new layer of fungal network lines draws beneath.

### Step C2 — Compute the mycelial network (~40 lines)

Every tick (or every time plants change):
```js
function computeMycelium() {
  // Nodes: every canopy tree + every fungal plant.
  // Edges: connect each fungus to its nearest 2 canopy trees within 400px.
  //        connect each canopy tree to its nearest 1 canopy tree within 300px.
  // Store as array of {ax, ay, bx, by, strength, type}
  //   type: 'ecto' (fungus↔tree) or 'root' (tree↔tree)
  //   strength: normalised by tree maturity (mature oaks = stronger)
}
```

Cache the network; only recompute when a plant is added/removed.

### Step C3 — Draw the network (~30 lines)

Lines with a soft glow, pulsing subtly with `Math.sin(t * 2)`:
- Ecto (tree↔fungus): warm gold `rgba(220,180,80,0.35)`, glow radius ~6px
- Root (tree↔tree): cool blue `rgba(120,160,220,0.25)`, glow radius ~4px

Width scales with `strength`. Draw in the space below `groundY(x)` — i.e., these lines appear *underground*.

### Step C4 — Player representation in the mycelial view (~15 lines)

The walking figure becomes a soft glowing node. When the player moves, trailing threads extend from them briefly, then fade. This is purely cosmetic — movement controls unchanged.

### Step C5 — Contextual text while in mycelial view (~10 lines)

Once per entry into mycelial view (not every frame), fade in a single line in the top-centre, such as:

> "What you called a forest is also this."

Rotate a small pool of these lines (and others) on successive toggles. Take inspiration from 'Entangled Life' by Merlin Sheldrake, Suzanne Simard and Peter Wohlleben.

### Step C6 — Death → mycelial view is seamless (~15 lines)

When the player dies, auto-enter mycelial view during decomposition. When `player.decomposed >= 1` and they press any key (existing rebirth trigger), fade out of mycelial view back to upright walking world. This makes the death/rebirth cycle feel like surfacing, not restarting.

---

## Plan D — The Sim Notices You

**Goal:** track what kind of player the user is being and reflect it back. Not as a score. As an observation, occasionally, offered without interpretation.

### Step D1 — The ledger (~20 lines)

Add to `player`:
```js
ledger: {
  took: 0,          // plants eaten or removed
  gave: 0,          // plants sown or tended
  listened: 0,      // Listen actions
  died: 0,          // deaths (persists across rebirths)
  lifeStart: 0,     // year this life began
}
```

Increment in the appropriate handlers. `died` accumulates across rebirths; others reset per life.

### Step D2 — Reflection lines (~40 lines)

Every ~60 in-game seconds (real time — roughly every 15–60 game-years depending on speed), check the ledger and maybe emit a single passing line of text, bottom-centre, fading in and out over 6 seconds. At most one per real-time minute.

Rules for which line (first match wins; roll a low chance 30% on match to keep it rare):

```
if (life duration > 20 years AND took == 0 AND gave == 0):
  "you have walked without taking. the forest notices nothing."

if (took >= 10 AND gave == 0):
  "the forest feels your hunger."

if (gave >= 5 AND took <= gave):
  "the forest is becoming what you hoped for."

if (listened >= 5):
  "you have been quiet. the trees are louder than they were."

if (died >= 3):
  "you have returned before. the soil remembers you."

if (current canopy cover is stable and has been for 30+ years):
  "the forest does not need you to continue."
```

None of these are compliments or accusations. Tone: an old person noticing weather.

### Step D3 — End-of-age reflection (~60 lines, one-time per session)

Once, and only once, after the forest has reached climax (cover ≥ 0.8 for 20 consecutive in-game years) OR after the player has died 5 times, fade the screen to near-black and show a modal:

- No buttons except "close" (and click-anywhere-to-close).
- One sentence summarising the ledger in prose: *"You walked here for 312 years. You ate 47 times. You planted 3 times. You listened 12 times. You died four times and came back four times."*
- One question in italics: *"What is your relationship to this place?"*
- Below, smaller: *"(no answer required. the forest does not keep a record.)"*

Only once per browser session. Store `localStorage['forest_reflected'] = '1'` after it fires. A secret reset by clearing storage — but don't surface that.

---

## Plan E — Let the Grave Matter

**Goal:** the current death → foxglove mechanic is beautiful but ephemeral. Extend it so death contributes something to the forest, and so that the *place* of death accumulates meaning.

### Step E1 — Nutrient plumes from graves (~25 lines)

Each grave already has `{ x, y, sinkT }`. Add `nutrientT` (years since death).

While `nutrientT < 20`:
- Plants within 150px of grave.x get a small colonisation-rate bonus (×1.5)
- Existing plants within that radius age slightly faster (+10%)
- Render a faint warm haze in the soil layer around the grave

Store the species that grew immediately after death (the foxglove already does this). Extend: after the foxglove dies, its successor gets recorded as a "memorial species." Over many lives, a grave may host a small succession of its own.

### Step E2 — Named graves / multiple lives visible (~20 lines)

Graves persist across deaths (they already do). Draw each grave as a small mossed lump on the ground surface, visible only when the player is within 200px. Over the grave, a faint column of foxgloves / ferns / moss, depending on what era the death happened in.

Clicking near a grave shows a tiny info card, and a small statement about how the dead player lived, based on the ledger at time of death: "you died here in year 87. you listened to the trees more than you took from them."

This gives the map memory.

### Step E3 — Choose what you become (optional, low-priority) (~30 lines)

When the death-message is showing, a subtle choice appears near the bottom:
- "become foxglove" (default)
- "become fern"
- "become lichen"

If the player clicks one, that becomes the memorial species instead of foxglove. Doesn't affect gameplay — just affects the grave's character.

---

## Plan F — The Four Framings, Emergent Not Declared

**Goal:** never tell the player they are a "steward" or a "dominator." Let the game's responses shift subtly with behaviour, so that each play-style finds a slightly different tone of forest.

### Step F1 — Behaviour signature (~20 lines)

Compute each frame (cheap — just reads the ledger):
```js
function behaviourSig() {
  const { took, gave, listened } = player.ledger;
  const total = took + gave + listened;
  if (total < 3) return 'novice';
  return {
    dominion:    took / total,
    stewardship: gave / total,
    participation: listened / total,
  };
}
```

Plus: `indifference = (time alive without any action) / (time alive)`.

### Step F2 — Tint the reflection lines by signature (~15 lines)

The Step-D2 reflection-lines pool is already keyed by ledger facts. Add a second pool for each signature that emits more rarely:

- High dominion: *"the forest does not fight you. it does not fight anything."*
- High stewardship: *"you are shaping. the forest is also shaping itself."*
- High participation: *"your edges are quieter than they were."*
- High indifference: *"something is here without you. you have not asked what."*

Do not label them. Do not stack. One at a time. Let the player notice the tone and not know why.

### Step F3 — Do NOT add a "philosophy score" (~0 lines, as a design constraint)

It would be very tempting to show a little radar chart of the four framings. **Don't.** It flattens the whole thing into a personality quiz. The framings are not slots. Refuse to quantify.

---

## Build Order & Checkpoints

| Step | What | ~Lines | Depends on | Checkpoint |
|---|---|---|---|---|
| A1 | Seed inventory data | 20 | — | Seeds increment in console when eating berries |
| A2 | Seed pouch UI | 30 | A1 | Visible pouch with counts |
| A3 | Sow action | 40 | A2 | Can sow blackberry on bare ground |
| A4 | Tend action | 30 | — | "Tend" visible on seedlings; works |
| A5 | Listen action + texts | 50 | — | **"Listen" on oak plays text, no harvest.** |
| | **Reciprocity complete** | **~170** | | |
| B1 | `persistence` fields on 22 species | 200 (content) | — | Every species has one line of data |
| B2 | Render persistence on card | 15 | B1 | **Info card shows persistence badge.** |
| C1 | Mycelial toggle state | 20 | — | `m` key flips a boolean |
| C2 | Compute network | 40 | — | Console-log a sensible edge list |
| C3 | Draw network | 30 | C2 | **Pressing `m` shows glowing underground lines.** |
| C4 | Player as node | 15 | C3 | Soft trail when moving |
| C5 | Contextual text | 10 | C3 | Line fades in on toggle |
| C6 | Death→mycelial seamless | 15 | C3 | Death transitions into mycelial view |
| D1 | Ledger struct | 20 | A3, A5 | Counts accumulate |
| D2 | Passing reflection lines | 40 | D1 | **Line appears after 10 harvests.** |
| D3 | End-of-age reflection | 60 | D1 | Modal fires once at climax |
| E1 | Grave nutrient plumes | 25 | — | Plants near grave grow faster |
| E2 | Grave memory UI | 20 | E1 | Can click old graves |
| E3 | Choose memorial species | 30 | E1 | Optional |
| F1 | Behaviour signature | 20 | D1 | Returns sensible ratios |
| F2 | Signature-tinted lines | 15 | F1, D2 | Rare signature lines appear |
| | **Full feature** | **~770** | | |

Minimum viable relationship layer: **A1–A5 + B1–B2 + D1–D2** (~425 lines + content).

---

## Writing Guidelines (Important)

An implementing LLM will want to write new prose: listen-lines, persistence notes, reflection lines. Before writing any:

- **No therapist voice.** Avoid "you feel…", "this reminds you…", "take a moment to…".
- **No magic-of-nature voice.** Avoid "ancient wisdom," "majestic," "pristine," "unspoiled," "mother nature."
- **No lesson voice.** Avoid "did you know?", "this species teaches us that…", "the forest has much to show us."
- **Short.** Usually one sentence. Two is a splurge. Three is too many.
- **Concrete over abstract.** "The acorn you stepped on was one of me" beats "we are connected to all living things."
- **Allow contradictions.** A species can have one line saying it welcomes you and another saying it does not know you exist. The forest is not a single thing.
- **Do not name Sandilands or queer ecology in-game.** The sim expresses the idea. It does not cite.
- **Do not punish dominion.** If a player clear-cuts the forest, no mechanic should call them evil. The forest may regrow, or not; the line may read *"it was here before you"*; no score drops.

When in doubt, write less. Silence is part of the design.

---

## What We Are NOT Building

- A morality meter, alignment system, or "ecological footprint" score
- Quests, objectives, achievements, unlockables
- Dialogue trees, NPCs that explain things to the player
- Realistic survival mechanics (thirst, temperature, fatigue, shelter)
- Multiplayer or persistence beyond `localStorage['forest_reflected']`
- Procedural listen-text generation via an LLM at runtime — all text ships in the file
- A tutorial. The sim opens, the body gets hungry, the player figures it out.

---

## Curriculum Fit Note

The core succession content maps to Ontario SBI3U/4U ecology expectations. The relationship layer is not curriculum-bound; it is the sim's point of view. A teacher could use the sim for succession content alone and ignore the listen/mycelial features. But the features don't obstruct the succession content — they sit beside it. Reciprocity teaches seed dispersal. Mycelial view visualises mycorrhizal mutualism (a curriculum concept). Persistence badges broaden "reproductive strategies" beyond the flower-and-seed default. The philosophical frame is the bonus, not the trojan horse.
