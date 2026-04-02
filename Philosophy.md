# Sci Wolf — Philosophy & Scope

## What This Is

Sci Wolf is a collection of interactive science simulations built specifically for **Ontario high school science students**. Each tool lives in a single self-contained HTML file and runs in the browser — no installs, no accounts, no lab equipment required.

The target platform is inexpensive institutional Chromebooks (first priority), and mobile browsers(nice-to-have). 

Current simulations:
- **Chemical Equation Balancer** — coefficient practice with live atom-count feedback
- **Bohr-Rutherford Diagram** — build atoms; shell structure and element identity update live
- **Static Electricity** — rub, transfer, attract, repel; electroscope and pith ball physics
- **Ray Diagram Simulator** — drag objects and focal points; real and virtual images in real time
- **pH Indicator Lab** — 12-well spot plate with household substances, indicators, and a digital pH meter
- **Forest Succession** — walk a living landscape through centuries of ecological change

## Why We're Building This

Ontario's science curriculum asks students to visualize things that are genuinely hard to see: electron shells, charge transfer, the path of refracted light. Wet labs are often unavailable, unsafe, time-consuming, or inaccessible for differentiated learners. Textbook diagrams are static. Video is passive.

Sci Wolf fills the gap between "watch this" and "do this for real."

## Prior Art & How We Differ

We admire **PhET** (University of Colorado) and **Gizmo** (ExploreLearning). Both demonstrate that interactive simulations increase engagement and conceptual understanding. Both prioritize visual representations of abstract phenomena.

We differ in scope and audience:
- PhET and Gizmo serve a **broad, international, multi-grade audience**. We serve **Ontario high school science specifically**.
- Their catalogs are large and generalized. Ours will be **small, curated, and curriculum-mapped** — every tool should connect directly to an Ontario course expectation.
- We are built by teachers, for their own classrooms first. Fidelity to how the curriculum is actually taught matters more than exhaustive coverage.

## Design Principles

**Curriculum-first.** Each simulation earns its place by addressing a specific concept or skill in the Ontario science curriculum. We are not building for general curiosity — we are building for a student who has a test next week.

**Dry alternative, not replacement.** These tools supplement real labs and direct instruction. They are especially valuable when wet labs are impractical: safety constraints, time, cost, differentiation needs, or remote/hybrid contexts.

**Accessible and low-friction.** Open in any browser. No login. Works on a school Chromebook. Mobile-aware. Light and dark mode. A student on a phone in the hallway should be able to use this.

**Honest interactivity.** Dragging, adjusting, and observing should feel meaningfully connected to the underlying concept — not just animation for its own sake. Students should be able to manipulate variables and see consequences.

**Models are scaffolds, not truth.** The Ontario curriculum makes deliberate simplifying choices — the Bohr model, the ray diagram, the pH scale. These are useful fictions: they build intuition and are worth learning on their own terms. But where a simulation can show both the curriculum model *and* the more complex picture it approximates, it should. A student who can move between the Bohr model and orbital probability clouds — and articulate what each communicates and where each breaks down — understands more than one who only knows the curriculum version. We name the scaffold when it exists.

**Appropriate depth.** We are not building research-grade simulations. But "appropriate" is calibrated to the concept, not just the grade level. Correct intuitions are the goal; we should not actively mislead students about where a model's edges are.

## What We Are Not Building

- A full LMS or assessment platform
- A replacement for teacher instruction
- A generalized science tool for all ages and curricula
- Anything that requires school IT approval or student accounts to function

## Deployment Goals

1. **Classroom trial** — Colin and Emma's own classes
2. **Department trial** — science department at their school
3. **Public launch** — Ontario-focused, open access
