<!-- bmad:context -->
<!-- Verified 2026-09-16 against a324bfb. Managed by bmad-project-context; edits inside this block are replaced on refresh. Keep anything you want preserved outside the markers. -->

## TikTokTrivia

An automated pipeline that produces TikTok trivia videos end to end — question sourcing, art direction, stills, voice, render — and, equally, a phased curriculum that teaches Carrie, who is non-technical and new to AI, one concept per phase. Python for media generation, TypeScript for web surfaces; no application code exists yet, so the stack below is a decision, not a build. Planning artifacts land in `_bmad-output/planning-artifacts/`, project knowledge in `docs/`, and intent lives in `BRAINDUMP.md` until a spec replaces it.

## Policy

- Never publish to TikTok from code. The pipeline renders and stages; Carrie approves every post by hand.
- Never bundle two new concepts into one phase — one tool, one idea, one working result. Pacing is the product here, not a nicety.
- Carrie's only interface is the Hermes agent. Never design a step that requires her to edit a file, run a command, or read this repo.

## Where things are

- Hermes agent profile: `agents/hermes/pm/` — already on `PATH` via `mise.toml:5`, not created yet.
- Curriculum and lesson material: `docs/` (BMAD `project_knowledge`), not created yet.
- BMAD output: planning in `_bmad-output/planning-artifacts/`, implementation in `_bmad-output/implementation-artifacts/`.

## Running and verifying

- TODO: no build, test, or render command exists yet. Stack is decided — Python for media generation, TypeScript for web surfaces — but tooling is not chosen. Replace this line on the first refresh after code lands.
- Entering the directory fires three `mise` hooks automatically: agent-file symlinks, `op inject` of `.env.op` into `.env`, and CodeGraph install plus index. Don't reproduce their work by hand.

## Conventions that differ from defaults

- Name a phase's deliverable before starting it — the concrete artifact handed back, such as a sheet of questions, a screenplay, ten stills. A phase with no named artifact is not ready to run.
- Match model cost to the phase: cheap models where the work is retrieval, like sourcing candidate questions; strong models only where judgment is the product, like difficulty calibration and art direction.
- Trivia questions are calibrated, not collected. Target the band where a viewer knows the answer but has to reach for it; obscure is a failure, not a flex.
- Prefer a walking skeleton — an end-to-end ugly video, stills plus TTS with no animation or captions — before polishing any single phase.

## Known pitfalls

- Edit `AGENTS.md` only. `CLAUDE.md` and `GEMINI.md` are symlinks to it, and `.mise/scripts/link-agentfiles.sh` aborts if either becomes a real file.
- The Plane board does not exist — `.project.json` declares identifier `TIKT` with an empty `board_id`. Create it before any ticket-driven work.

<!-- /bmad:context -->
