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

### Skills & upgrades — two governing principles
These two rules keep every skill *assist-not-replace* by construction and keep the world cozy at rest:
- **Reveal counts and states, never positions.** Skills may tell you *whether*, *how many*, and *what state* — never *which book, where*. The seek (identifying and locating the specific book) always stays the player's. The single deliberate exception is the **Hot/Cold** rescue skill, which is heavily cooldown-gated precisely because it crosses this line.
- **Permanent fixtures always show state; transient surfaces are skill-gated.** Shelves and Aisle/Floor/Wing **plaques are always-on** — they are the progress/reward layer (the gold zone visibly growing, the watch-it-clean payoff). Skills surface the action-planning layer on the **messy transient stuff** — tables, foreign books, match counts, empty surfaces — and those overlays are time-boxed so the map never becomes a HUD.

### Progression & quest economy
All progression is minted **only by playing the loop** — both currencies come from the two state transitions, so there is no way to advance except by sorting. This is what keeps the player *in* the loop.

**Two-axis reward split** (maps onto the two satisfaction axes; keeps a reward stream always flowing):

| Axis | Fires on | Grants | Feel |
|---|---|---|---|
| **Present** (containment, *fast/coarse*) | coarse thresholds — books merely *into* a region | **reveals & unlocks**: next area opens, a new skill appears (free at base tier), **charges** drip | the world keeps *expanding*; novelty arrives early |
| **Complete** (order, *slow*) | a container fully in order | **premium currency** (tier-weighted) + a **one-shot scroll** | the deep, climactic *payoff* for finishing |

- **Why Present drives reveals/unlocks (not Complete):** Complete is the slow, back-loaded tail; gating new territory behind it would violate **"never gate progress on 100%."** Present is fast, so the **next area reveals while you're still ordering the current one** — there's always a freshly-lit frontier edge beckoning. Area/library unlocks trigger on Present thresholds.
- **Two currencies:**
  - **Charges** (drip, from Present milestones) → consumable skills (Assemble, Hot/Cold, Sharpen). Rise only as fast as you contain, so you can't out-spell the work.
  - **Premium** (chunky, from Complete) → permanent upgrades & **skill-tier** purchases. **Tier-weighted geometrically** (illustrative: Cabinet 1 · Aisle 5 · Floor 25 · Wing 100) so bigger completions dominate and cascades feel huge.
- **Skill acquisition:** a Present threshold **reveals a skill free at base tier** (immediate new toy = the pull); **premium currency buys its tier upgrades** and cart-size upgrades. Spending is a **quick inline radial, never a shop menu** (menus pull players *out* of the loop); the buyable list *grows* as you complete bigger containers — completion depth paces *what's available*, premium spend chooses *order*.

**The forward-pull engine ("one more table"):**
1. **Completion cascades** — because of monotonicity + Present-before-Complete, completions chain: an aisle's last cabinet tips the aisle, maybe the floor, maybe the wing. **The last book of a region is the single most rewarding action in the game** — one small act detonates every parent bar it completes, plus currency + a scroll. Chasing that detonation *is* the addiction.
2. **Zeigarnik magnetism** — always-on plaques make a near-full bar hard to walk away from; the economy makes near-full the most lucrative to close.
3. **Reveal-the-next** — Present unlocks the next area (and often a new skill) *before* the current region is fully ordered, so finishing one loop literally hands you new territory **and** a new tool to go try.

**One-shot completion scrolls** (Complete loot, alongside premium currency):
- Each completion drops a **one-shot scroll**, drawn **at random from that tier's pool** (the "what did I get?" adds to the detonation). Held as **non-decaying inventory** — no timer, no fail-state pressure.
- Effects are **tier-scaled** and would be OP if repeatable — but they're not. Example pools:

  | Complete a… | Example scroll pool (one-shot) |
  |---|---|
  | **Cabinet** | *Recall (table)* — pull all matching books on the current table to cart · refill charges |
  | **Aisle** | *Grand Assemble* — close out one near-Present set free · *Reveal* — flash all match-counts on this floor briefly |
  | **Floor** | *Recall (floor)* — pull all matching non-shelved books on this floor to cart, up to cart size · *Overfill* — next cart load carries double |
  | **Wing** | *Recall (library)* — pull all matching non-shelved books library-wide to cart, up to cart size |

- **Why this stays assist-not-replace even at wing scale:** a scroll's supply is gated behind exactly the work it would replace — you only mint a wing-scroll by completing a wing (already sorted by hand), so scrolls can't be hoarded into a stealth-autopilot; the shortcut only mints *after* the long way. And every Recall only does the **collect** step for **one filter, up to cart size** — you still haul, dump, and shelve.

**Quests = emergent frontier spotlights** (not an authored quest log — that would be content to write and would fight the emergent frontier):
- The **container hierarchy *is* the quest system.** The game spotlights the **nearest almost-done container** as the current soft-goal, previews its cascade reward, and auto-promotes the next one on completion. Fully procedural/asset-free; it *is* the depth-first expanding frontier turned into guidance.
- Optional light nudges (e.g. "finish any Cabinet in the Alchemy wing") can steer breadth-vs-depth — kept minimal.

**Macro progression & the difficulty curve:**
- **Within a library:** enough **Present** on a floor/wing unlocks the next — spatial gating that matches depth-first expansion.
- **Across libraries (levels):** completing a library unlocks the next — a fresh seed/theme where **the seek gets subtler** (finer distinguishing features = the fair-band difficulty ramp).
- **Assist-not-replace *as* the difficulty curve:** each new library raises seek subtlety; the skills unlocked in the previous one are exactly what restore fairness. New library raises the challenge, prior skill answers it.

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

### Skill / upgrade roster (direction settled; tiers, gating & numbers TBD)
Every entry obeys the two governing principles above. Grouped by the loop friction it relieves.

**Always available from the start (base kit, not skills):**
- **Cart + click-collect** (single-filter) and **walk** — the core verbs.
- **Binary table-clear indicator** — while zoomed into a table, a "clear for this filter" mark appears once zero matches remain. Boolean only: no count, no location. Kills early-game "did I get them all?" uncertainty without handing out counts. The Match-counter skill *escalates* this same signal.
- **Shelves & container plaques always show state** (Invalid → Present → Complete + fill bars). Per the fixtures principle.

**Movement / haul**
- **Sprint** — at-will, short cooldown. Speed + cooldown are Phase-4 knobs.
- **Cart-size upgrades** — capacity N → 1.5N → 2N (illustrative). Fewer trips per table cleared = feels more powerful; every seek/haul/dump still happens, just amortized. This is the main power-growth spine. Sizes & increments are Phase-4 knobs.

**Seek — readability aids only, never auto-tag**
- **Insight / "Sharpen"** — brief, costed pulse that amplifies the active tier's cover encoding (matching code glows harder, rest dims harder) when the distinguishing feature is subtle. Temporary + amplify-only — you still read and click each cover.
- **Hot/Cold vignette** *(the one deliberate "positions" exception; long cooldown)* — on activation, locks to the **available book nearest the cursor** and gives a warm/cool vignette as the cursor moves toward/away from it; **clears on pickup** (a second book needs a second cast). **Grayed out when the table has no matches** so a long cooldown is never wasted (this reveals only ≥1-match presence — same info the binary indicator already gives). Framed as a **rescue valve** for a fair-band seek a player's eyes bounce off of, never the primary seek. Duration + cooldown are Phase-4 knobs.

**"Am I done / what's left?" — counts & states, never positions**
- **Match-counter** — escalates the base binary indicator:
  - **T1:** exact count of remaining matches while zoomed.
  - **T2:** count on **overworld hover** over a table (route-planning triage).
  - **T3:** counts **persist in the current room** without hovering. *(Past T3 = diminishing returns / clutter.)*

**Haul destination — "find an open table to dump on"**
- **Arrow-to-open-table** — player-activated; the arrow **clears on arrival** to keep the screen calm.
  - **T1:** nearest empty table, period.
  - **T2:** nearest empty table **in the cart filter's home region** (dump carries books *nearer home*); alt-color fallback when none is ideally located.
  - **T3:** weighs emptiness *and* home-proximity, and prefers routing you to **consolidate onto a partially-sorted table already trending toward the same target** (the table-pulse mechanic paying off).

**Cleanup frontier — which tables still hold wrong books**
- **Foreign-book table highlight** — activated skill that tints **tables** by foreign-book presence (transient layer); container cleanliness is shown permanently by shelves/plaques. Tiered **visibility timeout + cooldown**, both Phase-4 knobs.

**Closeout / long tail**
- **Assemble** — on a fully-Present-but-not-Complete set, spend a charge to close it out hunt-free. Agency without obligation.
- **Auto-Shelving** — background mop that files trickle-in stragglers into **already-Complete** containers only. Monotonic; never touches unsorted piles.

**Cost-lane model** (self-balancing: power rises only as fast as you actually sort)
| Lane | Earned by | Powers |
|---|---|---|
| **At-will** (cooldown only) | free | Sprint, Sharpen, Hot/Cold (long cd), Arrow, Foreign-highlight |
| **Charges** | **Present** milestones | Assemble, other skip-tedium bursts |
| **Premium upgrades** | **Complete** rewards | Cart size, Auto-Shelving speed, Match-counter/Arrow tiers, +charge capacity |

**Provisional start → gated progression** *(final gating belongs to the still-open progression/quest economy):*
| Tier | Skills |
|---|---|
| **From start** | cart + click-collect · binary table-clear indicator (zoomed) · shelves & plaques always show state · walk |
| **Early** | Sprint · Cart size +1 · Match-counter T1 |
| **Mid** | Arrow-to-open-table · Foreign-book highlight · Match-counter T2 |
| **Later** | Hot/Cold vignette · higher tiers of the above |

### Phase-4 balance parameters (running list — tune in Testing & Tuning)
Captured as we design so nothing is lost before Phase 4:
- **Cart:** base capacity N; upgrade sizes & increments.
- **Sprint:** speed multiplier; cooldown.
- **Hot/Cold:** effect duration; cooldown (long enough to stay a rescue valve, not so long it frustrates).
- **Foreign-highlight:** visibility timeout; cooldown (per tier).
- **Arrow-to-open-table:** any per-tier timing/range.
- **Match-counter:** none timing-wise (passive); interacts with table capacity N.
- **Progression currencies:** charges granted per Present milestone; premium granted per Complete; the **geometric tier weights** (Cabinet/Aisle/Floor/Wing).
- **Reveal thresholds:** which Present % (and at which tier) reveals the next area / next skill; which unlocks the next library.
- **Skill costs:** premium price of each skill-tier upgrade and cart-size upgrade.
- **Scroll pools & rates:** contents of each tier's scroll pool and the random-draw weights; scroll magnitudes (e.g. Recall scope/caps).

---

## 3. Open Questions (not yet answered)

- **Spells / upgrades economy.** *(Roster + cost-lane model now drafted — see §2 "Skill / upgrade roster.")* Remaining: lock **tiers, numbers, and start→gated ordering**, which are entangled with the progression/quest economy (below) and Phase-4 balancing.
- **Progression / quest economy.** *(Now designed — see §1 "Progression & quest economy.")* Remaining is **numbers only** (currency rates, tier weights, reveal thresholds, skill costs, scroll pools) — all captured in the Phase-4 balance list.
- **Seek-difficulty tuning & placement** ("keep exploring"). *Direction now set:* each new **library** raises seek subtlety and the previously-unlocked skill restores fairness (§1 difficulty curve). Remaining is the explicit per-library subtlety curve — a Phase-4 tuning task.
- **MVP / first vertical slice.** What the first playable is (likely the 1-wing/1-floor starter library running the full table→cart→shelf loop with one or two spells).
- **Win condition / session shape.** Escape-the-library goal vs endless/zen vs level progression through libraries; whether any optional scoring or timers exist (baseline: no fail state).
- **Persistence.** Save/resume (localStorage?), and whether a library is shareable by seed.
- **Cart filter UI.** How the player sets/changes the cart's tier filter (Wing/Floor/Aisle/Cabinet/Set) in-flow.
- **Data model & architecture.** The concrete book/container data structures, the library tree schema, and how rendering/state derive from them (to be designed once the above systems settle).
- **Juice & audio.** Feedback for click-collect, table-clear, shelf-fill, and tier state-ups (badges, sounds, particles).

---

*Status: living design doc. Core experience (loop, spatial model, state machine, interaction, art pipeline, generation, initial distribution), the **skill/upgrade roster + cost-lane model** (§2), and the **progression/quest economy** (§1) are now all settled in design. What remains for Phase 1 is closing the smaller open questions (MVP slice, win condition/session shape, cart-filter UI) and the **🚪 Gate Review**; everything numeric is parked in the Phase-4 balance list. Next system-level work is the **technical plan (Phase 2)**.*
