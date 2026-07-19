# Design Doc — Librarian 2D (working title)

## Context

A 2D web clone of *Librarian: Tidy Up the Arcane Library*, **cozy and faithful in feel** but with the progression deliberately redesigned. The original's core flaw is a broken pacing curve: ~2 hours of tedious, needle-in-a-haystack seek-and-find to assemble your first sets, then a spell-powered "auto-clean" phase where the game plays itself and the challenge evaporates. **The reward for playing becomes a tool that deletes the reason to play.**

Our goal: a clean, addictive, *continuously satisfying* cozy sorting loop, where power progression is **assist-not-replace** (spells and upgrades relieve tedium and grant agency, but never do the work for you). Constraints: dead-simple to share, **no build step**, hostable on GitHub Pages, fully playable with **keyboard + mouse**.

> **Cut entirely:** the original's late-game "spam spells to auto-clean" phase is **not in this game**. There is no autopilot ending — the active sorting loop is the whole experience.

---

## 1. Settled Designs

### Platform & tech
- **Plain HTML5 Canvas + vanilla JavaScript**, static files, **no build step**, GitHub-Pages hostable.
- Input: **WASD to move, mouse to interact** with books.

### Tone
- **Cozy, faithful, no brutal fail states.** The redesign is in *progression and pacing*, not in adding punishment.

### Core gameplay loop — "Loop 4+": progressive partition-sort
- One core verb: **"extract every book matching *this pass's* filter from the pile in front of you."**
- You sort **coarse → fine** through nested stages until final filing is trivial. We explicitly **cut** the original's other playstyles (single-set hunting, grab-and-scatter, one-pass-color-then-hunt) — every feature must serve progressive partition-sorting.
- The **seek** is bounded and always fair: difficulty comes from **how subtle the distinguishing feature is at the current tier**, never from haystack size.

### The magnet cart (interaction)
- **Single-filter cart.** Player sets the cart to a container filter (a Wing, or Wing+Floor, or Wing+Floor+Aisle, etc.).
- In a zoomed table view, **click a matching cover → it flies to the cart.** Non-matches **do not fly to a random spot** (that idea is killed — it broke monotonicity).
- Rationale for single-filter: the match decision happens **only on pickup**. A multi-slot cart would force a decision on both pickup *and* dump.
- **Cart = a mobile table**: same uniform capacity N as a table, so "dump cart onto an empty table" is a clean, decision-free operation.

### Spatial model: tables, not floor-mess
- The working surface is **bounded tables**, not a floor sea — cleaner boundaries mid-sort, and a **tunable, fair seek surface**.
- **Shelves start empty.** Initial mess lives **on tables**, spread across the library; tables start **~1/4 full** so the player is never table-constrained. Some tables start clear as workspace.
- Because the mess is distributed across many bounded tables, cleanup is a **recurring, game-long** activity — not a one-time floor sweep.
- Tables **pulse**: a table's contents get more homogeneous as they move closer to home (`raw table → cart → nearer table → shelf`). Far tables = raw mix; near-shelf tables = nearly sorted.
- **Initial distribution:** books are scattered across tables with only a **slight** locality preference — very slightly biased toward home, but close to fully random. Enough to make early frontier work land a touch easier without engineering away the mess.

### Nested containment hierarchy
- **Library ⊃ Wing ⊃ Floor ⊃ Aisle ⊃ Cabinet ⊃ Set ⊃ Volume.**
- **Tiered reading sections** encode sort-progress *physically*:
  - Aisle tables count toward the **aisle**.
  - A per-floor reading section counts toward the **floor** (not a specific aisle).
  - A per-wing reading section counts toward the **wing** (not a specific floor).
  - A **library-level** reading section counts toward nothing specific (the raw-intake tier).
- **Present is cumulative down the hierarchy**: a book on its correct aisle table counts for aisle **+ floor + wing**; on a floor section counts floor + wing; on a wing section counts wing only; on a raw table counts for nothing specific.
- Consequence (a feature): **coarse Present fills fast** (get a wing's books merely *into* the wing = early win), **fine Present fills slow** — the exact pacing gradient we wanted, purely from how far in each book has been carried.

### State machine (progress model)
- Three per-container criteria:
  - **(a) own books present**: % → complete *(containment / "Present")*
  - **(b) foreign books present**: count → none *(cleanliness — the recurring cleanup driver)*
  - **(c) sub-containers complete**: % → complete *(recursive order)*
- States: **Invalid → Present → Complete**, where `Complete(C) = Present(C) AND all children Complete`.
- Backed by **continuous fill-bars** (badges = milestones, bars = the drip). Show the **phase-relevant** meter; collapse the rest to an inspectable badge to avoid clutter.
- **Two axes of satisfaction**: containment (raised early/continuously by coarse work) and order (raised later by fine work + spells). One stream is always flowing.

### Monotonicity & the outlier solution
- **Monotonic guarantee**: every action moves books toward home (or is neutral); nothing ever increases disorder.
- **Pull, not chase**: outliers cart themselves home as you clear wherever they currently sit — you never hunt for them.
- **Present before Complete**: never *order* a container until it's fully *contained*, so completion is always **hunt-free**.
- **Never gate progress on 100%.** The long tail auto-resolves; **Auto-Shelving** mops trickle-in stragglers; spells (**Assemble/Insight**) let the player *choose* to close out a near-complete set early. Agency without obligation.

### Macro-structure
- **Depth-first expanding clean frontier** (not breadth-first tier sweeps): fully finish a small region, then spread outward. Both satisfaction axes interleave continuously; the growing "gold zone" *is* the watch-it-clean payoff.

### Presentation & art
- **Two representations from one shared attribute→visual mapping**: **covers** (face-up on tables — rich detail, where the seek happens) and **spines** (on shelves — compact; a completed set = a run of matched spines in order). The cover→spine flip is the shelving moment.
- **Interaction is two-mode**: WASD **navigate/haul** in the overworld; **click a table → zoomed cover view** to **seek/sort**. The table is larger than the screen, so you **pan** it; target ~**100 covers on screen** (≈150px each — enough for legible layered detail).
- **Table size correlates with tier** → panning and hard seeks never coincide: big library/wing tables only ever get **coarse** filters (obvious, plentiful matches); small aisle tables host the **subtle fine** seek (no pan).
- **Cover encoding**: Wing = hue · Floor = shade/texture · Aisle = glyph/emblem · Cabinet = accent/border · Set = title + motif · Volume = number. **The active filter emphasizes its own tier's code** (matching elements glow, the rest dims) so you never read all five at once.
- **Procedural cover rendering**: covers are drawn at runtime in Canvas by composing layers from a book's attributes — **zero per-book art files** (a few glyph/texture primitives at most). Scales to any library size; fits static hosting.

### Content generation
- **Data-driven library tree**: configurable branching per tier supports varied shapes (e.g. 2–3 wings × 8–9 floors, or 10 wings × 2 floors), including a **1-wing / 1-floor starter** with several aisles & cabinets.
- **Seeded, grammar-based generator** (word lists + templates, no AI, pure JS, deterministic) produces **name + hue + glyph together per wing** (mnemonic and aligned, e.g. Alchemy → teal → alembic), plus set titles and volume numbers. Reproducible from a seed; offline; asset-free.

---

## 2. Designs That Need a Little More Polish

- **Capacity math reconciliation.** *(Parked — we'll circle back.)* "Tables start ~1/4 full" and "a container's tables = 1/2 its shelf capacity" conflict — the latter yields only ~2× total table capacity (→ ~50% full at start, not 25%). Pick one:
  - (a) accept ~50%-full start;
  - (b) container tables = **1×** shelf capacity → exactly 1/4-full, uniform rule;
  - (c) keep 1/2 for wing/floor/aisle but make the **library-level section large** to soak up the rest *(current lean — matches the "large library reading section" instinct)*.
- **Table capacity N** (books per table / per screen) tuned **together with cover distinctiveness**; ~100/screen is the target, per-tier total pan-size still to set.
- **Fine-tier Present forces some shelving.** Because a container's tables can't hold all its books, reaching full aisle-Present forces you to shelve (Complete) part of it — Present and Complete interleave at the bottom of the funnel. *Leaning: keep this as a feature, not something to engineer away.*
- **Non-match cart behavior.** Fling is dead and single-filter is locked; default is **non-matches stay put** (re-pass with a new filter). Possible enhancement: **auto-route non-matches coarsely toward home** (zero wasted clicks, more machinery). Not finally chosen.
- **Opening "consolidation" move.** A no-filter cart frees table space but is *lateral* (doesn't sort). Consider a **coarse-by-default** first cart so the opening move both frees space *and* makes progress.
- **Visual-encoding specifics.** Exact treatment per tier (Floor = shade vs texture; Cabinet = accent vs mark), how many distinct hues/glyphs a library can use before legibility suffers, and readability tuning at table density. Direction is set; specifics need craft + playtesting.
- **Table-space slack.** Keep tables generous (start 1/4 full, some clear) — settled in direction; exact slack TBD. Open sub-question: is table scarcity ever a *feature* (light logistics puzzle) or always kept generous?

---

## 3. Open Questions (not yet answered)

- **Spells / upgrades economy.** Full roster (beyond Insight, Assemble, Auto-Shelving, and movement: jump/sprint/carry), their **cost model** (mana / charges / cooldown), and — critically — **which new-area difficulty each spell answers**, verifying every one stays *assist-not-replace*.
- **Progression / quest economy.** Confirmed that **state transitions drive progression** (Present → charges/unlocks; Complete → premium rewards). Still open: the actual quest structure, unlock gating, any currency, how new **areas and whole new libraries (levels)** unlock, and how quests softly guide the frontier.
- **Seek-difficulty tuning & placement** ("keep exploring"). Partly answered by table-size↔tier correlation, but the explicit difficulty curve — where **unaided** seek lives vs where **spell-aided** seek takes over — is unresolved.
- **MVP / first vertical slice.** What the first playable is (likely the 1-wing/1-floor starter library running the full table→cart→shelf loop with one or two spells).
- **Win condition / session shape.** Escape-the-library goal vs endless/zen vs level progression through libraries; whether any optional scoring or timers exist (baseline: no fail state).
- **Persistence.** Save/resume (localStorage?), and whether a library is shareable by seed.
- **Cart filter UI.** How the player sets/changes the cart's tier filter (Wing/Floor/Aisle/Cabinet/Set) in-flow.
- **Data model & architecture.** The concrete book/container data structures, the library tree schema, and how rendering/state derive from them (to be designed once the above systems settle).
- **Juice & audio.** Feedback for click-collect, table-clear, shelf-fill, and tier state-ups (badges, sounds, particles).

---

*Status: living design doc. Core experience (loop, spatial model, state machine, interaction, art pipeline, generation, initial distribution) is settled; **spells and the progression/quest economy** are the next systems to lock, then the technical plan. Capacity numbers are parked.*
