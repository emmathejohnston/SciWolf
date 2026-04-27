# Forest Succession Simulator — Project Summary - Main file: forest.html

**Project Scope**

Forest Succession is an interactive educational simulation (grade-adjacent, SBI3U-aligned) that teaches ecological succession and invites players to explore their philosophical relationship with nature through mechanics rather than lectures. The sim visualizes how bare ground becomes a climax forest over centuries, featuring 22 species across 5 succession eras (pioneer → climax) with integrated mechanics for consumption, observation, reciprocal action (sowing and tending), and alternative perspectives (mycelial view as a continuous underground body). The core gameplay loop is driven by a persistent hunger bar that forces interaction while remaining optional for judgment: players can dominate, steward, participate with, or witness the forest according to their own stance.

**Work Summary**

The project spans seven major implementation plans totaling ~5,000 lines of code and substantial content writing. The minimum viable relationship layer (Plans A–B+D, ~425 lines) adds reciprocity systems (seed inventory, sowing, tending, and listening mechanics), queer-ecology persistence badges that decenter flower-and-seed reproduction as the default narrative, and a player-ledger system that allows the sim to reflect back observations about the player's behaviour without scoring or judgment. Plans C–F layer in a mycelial (underground network) perspective that dissolves the self/other boundary, behavioural signature tracking that subtly tints in-game text to match the player's philosophical stance, and grave-memory systems where the player's death sites accumulate meaning and feed the forest across multiple lives. Plan G introduces carrying-capacity heuristics driven by local soil health, shade preferences, and species-specific crowding limits, replacing artificial per-layer caps with emergent population regulation grounded in ecological principles. The entire design is informed by queer ecology theory (Sandilands, Gaard, Morton) and emphasizes concrete, non-didactic prose, silence, and contradiction as design tools.

---

## Core Implementation Roadmap

| Plan | Focus | Key Mechanics | Scale |
|---|---|---|---|
| **A** | Reciprocity | Seed inventory, sowing, tending, listen action | ~170 lines |
| **B** | Queer ecology | Persistence badges (clonal, symbiotic, spore, etc.) on info card | ~215 lines |
| **C** | Mycelial view | Underground network toggle, fungal lines, seamless death→rebirth | ~130 lines |
| **D** | Player reflection | Ledger tracking, passing lines, end-of-age modal | ~120 lines |
| **E** | Grave memory | Nutrient plumes from death sites, clickable graves, species choice | ~75 lines |
| **F** | Emergent tone | Behaviour signature, signature-tinted reflection lines | ~35 lines |
| **G** | Carrying capacity | Local soil health, shade metrics, crowding dynamics, persistent dead plants | ~285 lines |

---

## Philosophical Frame

The sim avoids a single "correct" relationship to nature. Instead, it makes four stances mechanically playable and observable:

- **Dominion:** harvest, cut, remove without replacement; the forest is resource
- **Stewardship:** sow, tend, favour chosen species; the forest is a garden
- **Participation:** listen, decompose, mycelial view; no boundary between self and forest
- **Indifference:** observe without acting, starve, be overgrown; the forest continues without you

None is framed as moral. The sim reflects back what the player is doing and leaves interpretation to them.

---

## Key Design Principles

1. **No therapist voice, no magic voice, no lesson voice.** Writing is concrete ("the acorn you stepped on was one of me"), brief, and allows contradictions.
2. **Mechanics do philosophy.** Queer ecology and relational frameworks are expressed through what the sim notices (info card fields, available actions), not through exposition.
3. **Reflection without judgment.** The ledger system and passing-text lines observe behaviour ("the forest feels your hunger") but never score or moralise.
4. **Silence is part of the design.** Many interactions should yield nothing — a still forest, a quiet moment, an absence.
5. **Emergent ecology over artificial caps.** Population balance arises from local soil, shade, and crowding, not enforced limits.

---

## Testing Harness: Acceptance Criteria

**Relationship layer (Plans A–F):**
- Info card displays persistence strategy badge and description
- "Listen" button plays ambient text without resource cost
- Seed pouch UI visible; sowing creates seedlings
- Mycelial view (press `m`) shows fungal network, dims plants, plays contextual text
- Passing reflection lines appear based on ledger (e.g., "the forest feels your hunger")
- End-of-age modal fires once per session, summarising the ledger without scoring

**Carrying capacity layer (Plan G):**
- Dead plants render as mossed lumps and contribute to local soil health
- Fungi spawn preferentially near graves (nutrient-sensing behaviour)
- Growth and propagation driven by local cover, shade, and soil conditions
- Crowding metric thins younger plants when species exceed caps
- Forest reaches stable climax without player intervention; layer populations self-regulate

---

## Minimum Viable Products

- **Relationship layer (MVP):** Plans A1–A5 + B1–B2 + D1–D2 (~425 lines) — reciprocity, persistence badges, player reflection
- **Carrying capacity (MVP):** Plans G1–G2 + G4–G5 (~195 lines) — local soil and shade metrics, crowding dynamics

---

## Writing Guidelines

- **Concrete over abstract:** "Two thousand of us live inside me" beats "we are all connected."
- **Short:** Usually one sentence; two is a splurge; three is too many.
- **Allow contradictions:** One plant can have one line welcoming you and another saying it does not know you exist.
- **Do not cite theory or morally evaluate play-styles.** The design expresses the philosophy; the player discovers it.

---

## Curriculum Fit

The succession content aligns with Ontario SBI3U/4U ecology expectations. The relationship layer is not curriculum-bound; a teacher can use the sim for succession content alone and ignore listen/mycelial features. But the features do not obstruct — they add reciprocal mechanics (seed dispersal), visualise mycorrhizal mutualism (a curriculum concept), and broaden reproductive strategies beyond the flower-and-seed default.

-- 

## The Problems. 

- Plant growth engine never was able to work. Several refactors, hours of bug fixing, nothing.
- Individual plants either had runaway unrealistic growth disallowing many others to fluorish, or all plants fluorished unrealistically and at succesionally inappropriate times. To balance growth parameters in a way that allowed succession to proceed naturally was an impossible task for this lowly vibe-coder. 

--

## Apr 26/26 - New Plan

0. You were interrupted in the middle of a large refactor. There may be half-finished lines of code in forest.html. Just a heads up.
1. Axe the entire idea of a ecological simulating plant growth engine. It's too hard a task for a simple HTML file. Realistic growth parameters are no longer the goal. The new goal is to simulate the appearance of them. 
2.  Shrink the map to the width of the window. Smaller screens will see less of the forest and that's fine. The human moves, the camera does not. We have a fixed view of a snapshot of a forest. When the human leaves one end of the window, they appear on the other side. 
3. Separate the map into 100px chunks. Each chunk tracks it's own successional progress. This should be a variable from 0 to 1, where 0 is earliest pioneer and 1 is late climax. It should take 150 years to reach climax. 
4. Species appear based on their successional role. Each layer (ground, shrub, canopy, fungi) has a hard cap: 2 canopy, 7 shrub, 10 ground, 2 fungi. Plant's lifespan should match their real-life lifespan. Death should occur randomly with increasing liklihood as it approaches its max age. Germination will happen randomly based on: species frequency (based on real-world frequency of a species within a typical Great Lakes ecosystem), and succesional era. We can soft-gate these eras, a ramping up and down of era-based spawn probability.
5. The only concession I'll make to a "shade-like" parameter is that if a canopy tree is present, the liklihood of spawning shrub and ground-level plants is halved. Place this behind a boolean variable I can turn on and off in the debug so I can test. However, this should also be accounted for by simply understanding "climax" species are mostly canopy + shade loving shrubs/ground plants, which is why I'm unsure if this is even needed.
- The results of items 3-5 is that undisturbed, a user could watch ecological succession play out more or less following a realistic population curve for each individual species over the course of 150 years. The difference between this and prior attempts is that the new plan is that succession is managed by hardcoding or soft-gating appearance. No light calculations, no soil, no crowding. 
6. The player can disturb the ecosystem. They can tend, they can harvest, they can remove, they can plant seeds. This will cause the local chunk's succesional variable to change appropriately. Tending a plant advances succession. Harvesting and removing and planting seeds reverses succession. Perhaps we need a disturbance veriable? Perhaps we just modify the succession variable. Harvesting/removing different plants impacts differently: A canopy tree drastically reverses succession, moreso than ground or shrub. It should be the case that a player who clearcuts a chunk will have reset the chunk to pioneer era. I want a player to see that when trees are cut, new growth takes its place. Life will continue, old growth will return, but it will take time. 
7. Invasives: Invasives should not germinate until after distrubance. Garlic, buckthorn should have an increased liklihood of replacing harvested/removed plants.
8. To all trees, split "Harvest" from "Eat" buttons. "Eat"'s label should reflect what is being consumed. White pine: "Eat bark, drink tea". Maple: "Drink syrup". etc. "Harvest" simply removes the tree and adds money to a new money counter in the top left counter, below the hunger bar. Money is useless, has no function. It is simply a number that can only increase. It should glow and emit particles when more money is added.