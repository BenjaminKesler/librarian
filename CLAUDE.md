# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A 2D web clone of the game *Librarian: Tidy Up the Arcane Library*, **cozy in feel but with a deliberately redesigned progression**. The original's flaw is a grind-then-trivialize pacing curve (hours of tedious seek-and-find, then spells auto-clean the game). This project fixes that with a continuously-satisfying sorting loop and **assist-not-replace** power progression.

## Project status: design stage, no code yet

There is **no source code, build, lint, or test tooling yet**. The repository currently holds only design/planning docs. Do not fabricate commands or a build process. When implementation starts it is, by hard constraint, **build-free static files** — you run it by opening `index.html` or serving the folder statically (e.g. `python -m http.server`); there is no bundler or compile step.

## Read these first — they are the source of truth

- **`docs/DESIGN.md`** — the canonical living design doc. Organized as **Settled Designs / Needs Polish / Open Questions**. Read it in full before proposing any design or implementation; it encodes decisions reached over a long design conversation, and many "obvious" alternatives were already considered and rejected there.
- **`docs/PROJECT.md`** — the waterfall project tracker (phases, status, milestones, and phase-gate reviews).

Keep both current. When a decision is made, move it between the three DESIGN.md sections and update PROJECT.md status. These docs are the memory that grounds future sessions.

## Load-bearing design invariants (do not violate without an explicit decision)

These constraints are *why the design works*; breaking one usually reintroduces the original game's problems:

- **No build step / static / GitHub-Pages-hostable / vanilla JS + HTML5 Canvas / keyboard (WASD) + mouse.** No frameworks or bundlers.
- **One core loop — progressive partition-sort.** Single verb: "extract every book matching *this pass's* filter from the pile in front of you." Sort coarse→fine until filing is trivial. Every feature must serve this; other playstyles were deliberately cut.
- **Monotonicity.** Every action moves books *toward* home (or is neutral); nothing ever increases disorder. This guarantee powers the "pull-not-chase" outlier handling — the player never hunts scattered books.
- **Assist-not-replace.** Spells/upgrades relieve tedium and grant agency but must never do the work for the player. There is **no autopilot endgame** (explicitly cut).
- **Physical location encodes sort-progress.** Nested containers (Library ⊃ Wing ⊃ Floor ⊃ Aisle ⊃ Cabinet ⊃ Set ⊃ Volume); a book's tier of "Present" is *where it physically sits*. State machine per container is Invalid → Present → Complete, with `Complete = Present AND all children Complete`.
- **Fair-band seek.** Seek difficulty comes from *subtlety of the distinguishing feature at the current tier*, never from haystack size. Big surfaces only ever get coarse filters; subtle filters only run on small surfaces.
- **Procedural, asset-free content.** Books are rendered at runtime from attributes (no per-book image files); library layout + names + visuals come from a deterministic seeded generator.

## Working process

- **Waterfall with phase-gate reviews.** Each phase ends with a 🚪 Gate Review (see `docs/PROJECT.md`): review outputs, course-correct, and reconfirm the next phase before proceeding. A gate may loop backward to revise an earlier phase.
- Design is co-developed with the user through discussion; prefer surfacing trade-offs and a recommendation over unilaterally locking decisions. The user drives design choices.

## Git

Main branch is `main`. Commit only when the user asks; branch off `main` first when you do.
