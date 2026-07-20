# Librarian 2D — Project Plan & Tracker (Waterfall)

A rough, living waterfall plan. Phases run top-to-bottom; each should be "settled enough"
before the next fully starts, with only light overlap allowed.

**Canonical design doc:** [`docs/DESIGN.md`](DESIGN.md)

### Phase-gate reviews
Every phase ends with a **🚪 Gate Review** before we move on. At each gate we:
1. **Review** what the phase produced against its goals.
2. **Course-correct** — decide if anything needs changing, and update `DESIGN.md` / this tracker.
3. **Reconfirm** that the *next* phase's steps are still correct given what we just learned.

This is our coarse-correction mechanism: it keeps a waterfall plan from drifting off-target
as reality teaches us things mid-project. A gate can send us *backward* (revise an earlier
phase) as well as forward.

## Status legend
✅ Done · 🟡 In progress · ⬜ Not started · 🅿️ Parked · ❓ Open question

## Where we are right now
**Phase 1 (Game Design), ~90%.** The core experience, the **skill/upgrade roster + cost-lane
model** (DESIGN.md §2), and the **progression / quest economy** (DESIGN.md §1) are all settled.
Remaining Phase-1 work is the smaller open questions (MVP slice, win condition/session shape,
cart-filter UI) and the **🚪 Gate Review**. All numeric tuning is parked in the Phase-4 balance
list. **No code yet.**

## Phase overview
| # | Phase | Status |
|---|-------|--------|
| 0 | Concept & Requirements | ✅ Done |
| 1 | Game Design | 🟡 ~90% |
| 2 | Technical Design / Architecture | ⬜ Not started |
| 3 | Implementation | ⬜ Not started |
| 4 | Testing & Tuning | ⬜ Not started |
| 5 | Release (GitHub Pages) | ⬜ Not started |

---

## Phase 0 — Concept & Requirements ✅
- ✅ Reference game analysed (*Librarian: Tidy Up the Arcane Library*)
- ✅ Core problem identified (grind-then-trivialize pacing curve)
- ✅ Vision (cozy, continuously satisfying sort loop; assist-not-replace power)
- ✅ Constraints (2D, vanilla JS + Canvas, no build step, GitHub Pages, WASD + mouse)
- 🚪 **Gate Review** — ✅ passed (requirements confirmed, Phase 1 scope agreed)

## Phase 1 — Game Design 🟡 (~70%)
**Core loop & experience — settled:**
- ✅ Core loop (Loop 4+ progressive partition-sort)
- ✅ Magnet cart (single-filter, click-to-collect, non-matches stay put)
- ✅ Tables / cart / shelves spatial model
- ✅ Nested hierarchy + physical-location-encodes-progress
- ✅ Three-criteria state machine (Invalid → Present → Complete)
- ✅ Monotonicity + pull-not-chase outlier handling
- ✅ Depth-first expanding frontier
- ✅ Interaction (WASD overworld + zoomed pan-table cover view)
- ✅ Art direction (covers vs spines, procedural rendering, tier encoding)
- ✅ Content-generation approach (seeded grammar generator)
- ✅ Initial distribution (slight locality grading)
- ✅ No autopilot endgame (cut)

**Remaining design:**
- ✅ Spell / upgrade roster + cost model (drafted; each verified assist-not-replace via two governing principles — DESIGN.md §2). Tiers/numbers/gating deferred to progression economy + Phase 4.
- ✅ Progression / quest economy (two-axis Present→reveals/charges, Complete→premium+scrolls; emergent frontier spotlights; area→library gating — DESIGN.md §1). Numbers deferred to Phase 4.
- ✅ Seek-difficulty curve — direction set: each new **library** raises subtlety, prior skill restores fairness (DESIGN.md §1). Per-library curve is a Phase-4 tuning task.
- ❓ Win condition / session shape (escape vs endless vs level progression)
- ❓ Cart filter-setting UI (how the player picks Wing/Floor/Aisle/Cabinet/Set)
- 🟡 Visual-encoding specifics (per-tier treatments; legibility tuning)
- 🅿️ Capacity numbers (table:shelf ratio; N per screen)
- 🚪 **Gate Review** — ⬜ review the full design; confirm the core loop hangs together with spells + progression added; reconfirm Phase 2 (architecture) scope

## Phase 2 — Technical Design / Architecture ⬜
_(depends on Phase 1 systems)_
- ⬜ Data model: book + container-tree schema
- ⬜ Procedural generator design (seed → library layout + names + cover layers)
- ⬜ Rendering architecture (Canvas; cover & spine renderers off one shared mapping)
- ⬜ Game-state & update model; overworld vs table-zoom modes
- ⬜ Persistence (localStorage save/resume; share-by-seed)
- ⬜ File / module structure (static, no bundler)
- 🚪 **Gate Review** — ⬜ sanity-check the architecture against the design; confirm nothing forces a design change; reconfirm the Phase 3 milestone breakdown

## Phase 3 — Implementation ⬜
_Build milestones (draft):_
- ⬜ **M1 — Vertical slice:** 1-wing / 1-floor starter, full table → cart → shelf loop, 1–2 spells
- ⬜ **M2 — State machine + fill-bar UI + tier reading sections**
- ⬜ **M3 — Procedural generation** for arbitrary library sizes
- ⬜ **M4 — Progression / quest economy** + area unlocks
- ⬜ **M5 — Juice & audio**, polish
- 🚪 **Gate Review** — ⬜ playable build review; confirm it matches the design intent; capture what implementation taught us; reconfirm the Phase 4 test/tuning plan

## Phase 4 — Testing & Tuning ⬜
- ⬜ Fair-seek band tuning (N, cover distinctiveness, pan sizes)
- ⬜ Capacity / table-slack balance
- ⬜ Progression pacing (Present-early / Complete-late feels right)
- ⬜ Playtest for the "one more table" addictive pull
- 🚪 **Gate Review** — ⬜ does it actually feel good? Confirm the fair-seek band and pacing land; decide if any tuning loops back into design/implementation; reconfirm the Phase 5 release plan

## Phase 5 — Release ⬜
- ⬜ GitHub Pages deploy
- ⬜ Shareable link (+ seed sharing)
- 🚪 **Gate Review** — ⬜ post-launch retrospective; what shipped vs. the vision, what we'd do differently, and what (if anything) comes next

---

## Immediate next actions
1. ✅ ~~Lock the **spell / upgrade roster** and its cost model.~~ (DESIGN.md §2)
2. ✅ ~~Design the **progression / quest economy**.~~ (DESIGN.md §1)
3. Close the remaining Phase-1 open questions: **MVP / first vertical slice**, **win condition / session shape**, **cart-filter UI**.
4. Run the **🚪 Phase-1 Gate Review**, then enter **Phase 2** (data model / architecture).
