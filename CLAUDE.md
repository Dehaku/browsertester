# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project

*Witchroam* (working title): a browser-based, top-down magic survival game built from simple geometric shapes. Controls are mouse and touch only. Magic is cast by drawing glyphs.

**The design document is the source of truth:** [`docs/GAME_DESIGN.md`](docs/GAME_DESIGN.md). Read the relevant section before implementing a feature. Requirements have stable IDs (`FR-SURV-03`, `FR-MAG-04`, and so on). Reference them in commit messages and code comments.

## Working Rules

- **Build by milestone.** Check the current milestone in §17 of the design doc and build only what it lists.
- **Tuning values live in data files**, not in logic. Spell data goes in `src/magic/spells.json`. Survival rates go in a config file under `src/survival/`.
- **If a rule must change, update the design doc in the same commit** as the code change, with the same ID.
- **Touch and mouse parity.** Any new interaction must work with one pointer, with no hover-only or keyboard-only paths.
- **No build step in v1.** Use plain ES modules and HTML5 Canvas 2D. Do not add a dependency without a reason stated in the PR or commit.
- **Keep the game playable with sound off** and respect the reduced-motion setting.
- **Determinism:** all randomness goes through the seeded RNG in `src/core/rng.js`.

## Commands

No build or test commands exist yet. When they are added (M0), document them here.

## Conventions

- Commit messages: short imperative subject, body explaining the why, referencing requirement IDs where relevant.
- Stay within the repo's current scope: do not add files outside the structure in §15.2 of the design doc without a note in the commit.
