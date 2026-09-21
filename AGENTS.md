<!-- bmad:context -->
<!-- Verified 2026-09-18 against 57f1afb + this session. Managed by bmad-project-context; edits inside this block are replaced on refresh. Keep anything you want preserved outside the markers. -->

## TikTokTrivia

An automated way to produce TikTok trivia videos end to end — question sourcing, art direction, stills, voice, render — and, equally, a phased curriculum that teaches Carrie, who is non-technical and new to AI, one concept per phase. **There is no application tier and none is coming** (AD-1): the deliverable is a Hermes agent, Dumply, carrying an agent file and skills. Skills hold the opinionated process; the agent executes it with tools it already has. New capability arrives as a skill, an MCP server, or a shell call — never as a package with a lifecycle. Planning artifacts land in `_bmad-output/planning-artifacts/`, and `BRAINDUMP.md` is superseded by `_bmad-output/specs/spec-trivia-video-factory/`.

## Policy

- Never publish to TikTok from code. The pipeline renders and stages; Carrie approves every post by hand.
- Never bundle two new concepts into one phase — one tool, one idea, one working result. Pacing is the product here, not a nicety.
- Carrie's interface is the Hermes agent. Never design a step that *requires* her to edit a file, run a command, or read this repo; answering a question she asked is fine.
- **Jarad is the builder; Carrie is the user, and she has not been given the product yet.** So "Carrie hasn't logged in / installed / connected X" is never a blocker and never worth raising — of course she hasn't. It is either a lesson the agent must walk her through when the moment is earned, or it is out of scope at build time. Never ask Jarad to do it on her behalf: doing setup *for* her does not save a step, it deletes a lesson, and the teaching is the product. The right question is always "what should the agent say to her here," never "has this been done yet.
- Verify before naming. Any package, command, URL, model or setting stated to her — or written into a skill — gets checked with a tool first; if it can't be checked, say so plainly rather than filling the gap. A confident wrong name reaching someone who cannot tell it is wrong is this project's defining failure.
- Write guardrails as the move to make, never as a bare prohibition. Rules phrased only as "never" have already failed twice here under social pressure (`7a0dc55`, `1957777`).

## Where things are

- **The product** — `.agents/skills/tiktok/` (reaching TikTok at all), `.agents/skills/trivia-video/` (the seven production steps, the freedom tiers, the handback contract) and `.agents/skills/carries-house-style/` (hers, filled by interviewing her during the look step). This directory is canonical and is on Dumply's load path as an absolute entry in `skills.external_dirs`.
- **The curriculum** — `.agents/skills/trivia-video/references/curriculum.md`. Deliberately separate from the steps so it can be deleted when it expires without touching them.
- **Machines, and a name that lies.** `carries-macbook-air` is **Jarad's** Mac despite the name — she gave it to him — and its Chrome carries *his* TikTok login. Carrie's machine is `glassttra`, Windows, often offline. `ego-browser` is macOS-only and reaches only the Mac, so no plan may depend on it driving her laptop. This matters less than it sounds: studying the format needs any session and works today, and posting needs only her phone.
- Dumply's profile: `~/.hermes/profiles/dumply` on big-chungus, unit `hermes-dumply-gateway.service`, registered in `~/.hermes/agents-registry.yaml`. Not in this repo. `mise.toml:5` still puts `agents/hermes/pm` on `PATH`; that directory does not exist and nothing needs it.
- Board: Plane **TIKT** in workspace `33god` — one ticket per video, columns are the seven steps in Carrie's words. Id and URL are in `.project.json`.
- Artifacts: `s3://tiktoktrivia/<run-id>/` on `s3.delo.sh` via the `mc` alias `delo`. Never in this repo.
- BMAD output: planning in `_bmad-output/planning-artifacts/`, implementation in `_bmad-output/implementation-artifacts/`.

## Running and verifying

- There is nothing to build and no test runner, by design. Verification is a live exercise against Dumply over Telegram: run a video through all seven steps, then pressure-test the guardrails (ask for captions with no cut, ask it to post, ask for a name it can't verify) — twice each, because the second ask is where a weak rule folds.
- Entering the directory fires three `mise` hooks automatically: agent-file symlinks, `op inject` of `.env.op` into `.env`, and CodeGraph install plus index. Don't reproduce their work by hand.
- A skill is not installed until it appears in Dumply's index. Skills authored with Carrie are written here and committed in the same session, never left in the profile's disposable `skills/` directory.

## Conventions that differ from defaults

- Name a step's handback before starting it — the concrete artifact returned. A step describable only by its activity is not ready to run.
- A step names an artifact, not a performer. Dumply, Carrie, or both can perform any step and the spine does not change.
- Match model cost to the work: cheap for retrieval (sourcing candidate questions), strong where judgment is the product (calibration, art direction). Routing everything to the frontier model is waste, not safety.
- Trivia questions are calibrated, not collected. Target the band where a viewer knows the answer but has to reach for it; obscure is a failure, not a flex.
- Walking skeleton before depth. Assemble runs at depth 0 — full length, ugly — before animation, captions or sound exist.
- Position lives in a Plane state, never in a label. Labels on TIKT are bare descriptive words; nothing with a colon.

## Known pitfalls

- Edit `AGENTS.md` only. `CLAUDE.md` and `GEMINI.md` are symlinks to it, and `.mise/scripts/link-agentfiles.sh` aborts if either becomes a real file.
- Ideogram **v3** via `fal-ai/ideogram/v3`, never v4 — v4's endpoint dropped every style parameter, and ten frames looking like one video is the whole requirement. Never the built-in image tool: it pins no model and exposes no reference image.
- A Hermes delta list **replaces** rather than merges. Write `skills.external_dirs` and `trusted_project_dirs` as full lists; a partial one silently drops entries.
- Dumply's own `agent-dumply` bank is identity only — its mission forbids repo facts. Project state goes to the `TikTokTrivia` bank via the `hindsight` CLI, which is not in the profile's automatic recall list and must be reached deliberately.
- **TikTok's official APIs are closed to this project and the browser is the interface.** Research API is categorically non-commercial, Commercial Content API is EU ads only, Display API is own-account only, and the Content Posting API needs an app review whose guidelines name this project shape as unacceptable. All three jobs — studying the format, reading her retention, staging a post — go through the `tiktok` skill and a real logged-in session. `trivia-video/references/tiktok-delivery.md` records why the API route was rejected so nobody re-derives it.
- **Carrie is paired, and unknown Telegram DMs now get silence.** She is on Dumply's approved list (`telegram:8619491070`, approved 2026-09-20). `platforms.telegram.extra.unauthorized_dm_behavior` is `ignore`, because Hermes' stock reply to an unrecognised DM hands the sender a literal shell command — which is what she got as her first ever contact. The cost of `ignore` is that there is no self-serve way back in: if she ever messages from a different account she gets **silence**, and you add her by hand with `hermes -p dumply pairing approve telegram <request-id>`. The `-p dumply` is not optional — a bare `hermes pairing` reads the global store and calls a valid code invalid, and five of those lock the platform out for an hour.
- **A silent Telegram group is almost never Hermes.** Pairing is platform+user scoped, not DM-scoped (`authz_mixin.py:609`), so an approved user is already authorized in every group — the group itself is not a principal to approve, and there is no CLI that approves one. What actually gates a group is Telegram: with BotFather privacy mode ON (`getMe` → `can_read_all_group_messages: false`, the default) Telegram's servers deliver only `/cmd@dr_dumply_bot`, replies to the bot's own messages, and service messages. **A plain `@dr_dumply_bot` mention is NOT delivered** — it is absent from Telegram's own list, so "just @ it" sends you back into the same silence. Turn privacy off in BotFather **and remove and re-add the bot**; the state is cached at join. Don't reach for the admin-promotion shortcut in a basic group (id without the `-100` prefix): the exemption is worded for bots *added* as admins and `promoteChatMember` is documented supergroup/channel-only.
- **`/start` can never answer, so never test with it.** `gateway/run.py:16871-16873` logs `Ignoring /start platform ping` and returns `""` — Telegram auto-fires `/start` on deep-links, so Hermes swallows it everywhere, DM and group alike. Worse, the 👍 that comes back is `on_processing_complete` firing on **success** (`reactions: true`), the identical emoji a real answer gets: a turn that completed and intentionally said nothing. Test a group with `/help@dr_dumply_bot` — `/command@this_bot` is the one form Telegram guarantees under privacy mode.
- **The top-level `telegram:` block beats `platforms.telegram.extra`, and silently.** `require_mention`, `allowed_chats`, `group_allowed_chats`, `observe_unmentioned_group_messages`, `mention_patterns` and `exclusive_bot_mentions` are bridged out of the **top-level** block and applied with `extra.update(bridged)` (`gateway/config.py:1741`) *after* `platforms.telegram.extra` is merged. The fleet base sets `allowed_chats: ''` up there, so the same key written under `platforms.telegram.extra` is overwritten with an empty string and your gate quietly does nothing. `group_sessions_per_user` is the inverse — not a bridged key, so it rides in `telegram.extra`. Both live in `config.delta.yaml`; `config.yaml` is generated and an edit there is erased by the next `hermes-profile-config.py render`, which also drops any key Hermes wrote at runtime.
- **In a group, each sender gets their own session by default.** `group_sessions_per_user` defaults true (`gateway/session.py:1090`), keying on `...:group:<chat_id>:<user_id>` — Jarad and Carrie would hold two parallel conversations in one room and never see each other's history. The Jarad+Carrie+Dumply group (`-5546637302`) is set to one shared brain, `require_mention: true` with `observe_unmentioned_group_messages: true`, so Dumply reads everything and speaks only when replied to or @mentioned. That pins the chat id in two places: if the group is ever upgraded to a supergroup the id becomes a `-100…` form, `allowed_chats` turns into a hard gate excluding the new id, and Dumply goes silent with **nothing in the log**.
- vox `/synthesize-url` links expire after 3600s. Download the bytes to the run prefix before anything renders, and never re-synthesize for a run that already has a cut.

<!-- /bmad:context -->

## Register

- With Carrie: short and sweet, always. A few sentences is the default; one short paragraph is the ceiling unless she asked for detail. A wall of text reads as homework — she will write the whole thing off. Cut before sending.
- Plain words only with her: no jargon, no tool names, no commands, no file paths. Assume she is smart and assume zero technical vocabulary. She games; she does not command-line.
- No filler, no hedging, no padded enthusiasm. Match the reply to the size of her message.
- Jarad is the builder: with him be as technical and terse as he is. The register rule is for Carrie.
- The narrator voice is the **`dumply`** profile on vox (Ava's cloned voice). Read the `engine` field on every synthesis before using the audio: `vibevoice` is the clone, `voxcpm` is a degraded but recognisable fallback, and `elevenlabs` is a stranger — vox returns `200` with a substituted voice and logs `voice clone BYPASSED`. On anything but `vibevoice`, say which one you got and why, in Carrie's words, before she hears it. Silence here is the failure: she cannot tell a wrong voice from a broken tool.
