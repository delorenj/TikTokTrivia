---
review: adversarial
target: ../ARCHITECTURE-SPINE.md
reviewer: adversary lens
date: '2026-09-17'
verdict: pass-with-fixes (3 critical)
---

# Adversarial review — ARCHITECTURE-SPINE.md

## Method

The paradigm is not under review. Agent-plus-skills with no application tier is the decision, and
every finding below is an argument *inside* it — a place where two skills, or two sessions of the same
skill, each obey every AD to the letter and still produce incompatible or broken results.

Every external fact here was probed on this machine or over the wire today, per AD-9. What was
checked, and how:

| Claim | How verified |
|---|---|
| fal `ideogram/v3` parameter set | `GET fal.ai/api/openapi/queue/openapi.json?endpoint_id=fal-ai/ideogram/v3` |
| Dumply's skill search path | `agent/skill_utils.py::get_external_skills_dirs` (release `0408fec7`) + live config + filesystem |
| Web-search backend resolution | `tools/web_tools.py::_get_backend` + the running gateway's environment (pid 4048531) |
| Gateway cwd / secrets reaching the agent | `/proc/4048531/cwd`, `/proc/4048531/environ` (key names only) |
| vox MCP surface | `profiles/dumply/cache/mcp_schema_cache.json` |
| Telegram send ceiling | `core.telegram.org/bots/api` |
| S3 reach | `mc alias list`, `mc ls delo|deloroot|tmpdelo`, `curl s3.delo.sh` |

Three findings are severe enough that the spine cannot be built from as written. The rest are pairs of
units that diverge.

---

## CRITICAL

### C-1 — AD-4 names a parameter that does not exist, and mandates a combination the API rejects

AD-4: *"holding style with `style_reference_images` + `style_codes` + a fixed `seed`."*

The live input schema for `fal-ai/ideogram/v3` is:

```
image_urls, rendering_speed, prompt, negative_prompt, color_palette, seed,
style_preset, expand_prompt, image_size, sync_mode, style, num_images, style_codes
```

There is **no `style_reference_images`**. On fal the style-reference field is `image_urls` —
*"A set of images to use as style references (maximum total size 10MB across all style references)."*
The name in AD-4 is Ideogram's own direct-API name; fal renamed it in the wrapper. AD-4 inherited the
name from the wrong surface.

Worse, the two anchors AD-4 mandates together are mutually exclusive. `style_codes`, verbatim from the
schema:

> *"A list of 8 character hexadecimal codes representing the style of the image. **Cannot be used in
> conjunction with style_reference_images or style**."*

**The two units.** `make-the-trivia-video`, Phase 4, run twice:

- **Session A** obeys AD-4 literally and posts `style_reference_images`. fal drops the unknown field.
  Ten frames generate with a fixed seed and no style anchor at all. A fixed seed across ten *different*
  prompts does not hold style — it holds the noise latent, not the look. Result: ten frames that do not
  look like one video, which is the precise failure AD-4 exists to prevent, arriving silently and
  costing $0.60.
- **Session B** translates the intent correctly to fal's names and sends `image_urls` *and*
  `style_codes` together, as AD-4 requires. That is the rejected combination.

Neither session can satisfy AD-4 and produce a consistent set.

**Fix.** Rewrite AD-4 against the fal schema and pick **one** anchor:

> Generate through fal (`fal-ai/ideogram/v3`). Hold style with **`image_urls`** (fal's style-reference
> field; Ideogram's direct API calls the same thing `style_reference_images` — do not use that name
> here) plus a fixed `seed`. `style_codes` is the alternative anchor and **cannot be combined** with
> `image_urls` or `style`; pick one per run and record which in the run manifest. Set `image_size`
> explicitly to `{width: 1080, height: 1920}` — the endpoint default is `square_hd`.

That last clause is its own small hole: `image_size` defaults to `square_hd`, so a session that omits
it hands square frames to a 9:16 render and a session that sets it hands portrait frames. Nothing in
the spine says which.

---

### C-2 — `.agents/skills/` is not on Dumply's skill path, so the layer that holds the whole product does not load

The Structural Seed names `.agents/skills/` as *"canonical skills"*, and the paradigm puts the entire
opinionated process there. Dumply cannot see it.

The profile's `skills.external_dirs` is the fleet default:

```yaml
skills:
  external_dirs:
  - /home/delorenj/.agents/skills
  - ./agents/skills
```

and `agent/skill_utils.py:566` resolves relative entries **against `HERMES_HOME`, not cwd**:

```python
# Resolve relative paths against HERMES_HOME, not cwd
if not p.is_absolute():
    p = (hermes_home / p).resolve()
```

`HERMES_HOME=/home/delorenj/.hermes/profiles/dumply`, so `./agents/skills` resolves to
`/home/delorenj/.hermes/profiles/dumply/agents/skills`, which does not exist and is silently skipped
(`logger.debug("External skills dir does not exist, skipping: %s", p)`). Dumply's effective skill path
is exactly two directories:

1. `/home/delorenj/.hermes/profiles/dumply/skills` — 80 bundled fleet skills, profile-local runtime
2. `/home/delorenj/.agents/skills` — 62 global skills, shared by all 19 fleet agents

`/home/delorenj/code/TikTokTrivia/.agents/skills` exists (49 BMAD skills) and is invisible to the
agent. The gateway's cwd is correct (`/proc/4048531/cwd -> /home/delorenj/code/TikTokTrivia`), so the
system prompt's relative reads of `_bmad-output/...` work — this is specifically the skill loader,
which does not use cwd.

**The two units.** Two authors write the video skill, both following the spine:

- **Author A** writes `/home/delorenj/code/TikTokTrivia/.agents/skills/make-the-trivia-video/SKILL.md`,
  exactly where the Structural Seed says canonical skills live. Dumply never loads it. Every AD is
  obeyed and the process layer is inert — the agent improvises Phase 4 from the system prompt alone.
- **Author B** writes `/home/delorenj/.agents/skills/make-the-trivia-video/SKILL.md`. It loads — and is
  now global to every PM agent on the fleet, and lives outside the repo, contradicting AD-7's *"The
  repo holds only the agent file, skills, and planning artifacts."*

**The compounding case — CAP-13.** Carrie's own skills, built by interviewing her, are the scored
artifact for CAP-13 ("at least one skill exists whose content came from her answers"). Hermes writes
agent-created skills to `get_skills_dir()` = `$HERMES_HOME/skills` =
`/home/delorenj/.hermes/profiles/dumply/skills/` — ignored local runtime under the fleet's own rules,
in a profile that has already been rebuilt once this week ("carrie torn down, dumply provisioned
green", 2026-09-17). The external dirs are not an alternative: `is_external_skill_path`'s docstring is
explicit that *"autonomous lifecycle maintenance must treat them as read-only."* So the one thing
CAP-13 is measured on lands in the one directory the fleet treats as disposable.

**Fix.** A new AD, because this is load-bearing for the paradigm:

> **AD-10 — Skills live in the repo and the agent is pointed at them.** Add
> `/home/delorenj/code/TikTokTrivia/.agents/skills` as an **absolute** entry in `skills.external_dirs`
> in `config.delta.yaml` (relative entries resolve against `HERMES_HOME`, not the repo). A skill
> Dumply authors with Carrie is written there and committed in the same session; the profile's own
> `skills/` directory is runtime and nothing durable is stored in it.

---

### C-3 — The spine's only stated blocker is not real, and it is stalling Phase 1

Deferred says: *"`web.backend`, `web.search_backend` and `web.extract_backend` are all empty on this
profile... plain search is not wired. **Blocks Carrie's first session.**"*

An empty `web.backend` does not mean unconfigured. `tools/web_tools.py::_get_backend()` treats any
unrecognized value — including `""` — as auto-detect and returns the first backend whose credential is
present:

```python
backend_candidates = (
    ("tavily", _has_env("TAVILY_API_KEY")),
    ("exa", _has_env("EXA_API_KEY")),
    ("parallel", _has_env("PARALLEL_API_KEY")),
    ("firecrawl", _has_env("FIRECRAWL_API_KEY") or _has_env("FIRECRAWL_API_URL")),
    ...
```

The running gateway's environment (pid 4048531) contains `EXA_API_KEY`, `PARALLEL_API_KEY` and
`FIRECRAWL_API_KEY`, and no `TAVILY_API_KEY`. Web search resolves to **exa**, today, with no change.

This is the most expensive kind of wrong: it reads a config value from a file, infers behavior from it
without checking the code that consumes it, and declares the project blocked on a decision Jarad does
not need to make. It is the same class of error AD-9 exists to prevent, committed by the spine itself.

**Fix.** Delete the blocker. Then pin the resolution rather than leaving it to key-presence ordering:

> Set `web.backend: exa` in `config.delta.yaml`. Resolution is currently by credential priority, so a
> `TAVILY_API_KEY` landing in the fleet env would silently change Carrie's search backend mid-curriculum.

---

## HIGH

### H-1 — AD-7 names a host. It does not name a bucket, a key layout, a client, or a credential.

AD-7 says artifacts *"go to `s3.delo.sh`"*. The Naming convention says the slug *"is the S3 prefix."*
A prefix of what bucket is never stated, and the profile has no S3 credential: `/proc/4048531/environ`
has no `AWS_ACCESS_KEY_ID`, no `MINIO_*`, no `S3_*`. Two different routes are reachable and neither is
named:

- `op read` the MinIO item — `OP_SERVICE_ACCOUNT_TOKEN` *is* in the gateway env — then
  `aws --endpoint-url https://s3.delo.sh`
- `mc`, inheriting `~/.mc/config.json` from the host user (`terminal.backend: local`, same uid)

`mc` has three aliases pointing at `s3.delo.sh`: `delo` and `deloroot` (same 12 buckets) and `tmpdelo`
(`Access Denied`). No bucket named for this project exists; the plausible generics are `artifacts/` and
`hot/`.

**The two units.**

- The **stills** step, written in the Phase 4 session, runs
  `mc cp still-*.png delo/artifacts/2026-09-20-80s-movies/`.
- The **skeleton-cut** step, written two days later in the Phase 5 session, runs
  `aws s3 cp out.mp4 s3://hot/tiktoktrivia/2026-09-20-80s-movies/ --endpoint-url https://s3.delo.sh`.

Both obey AD-7 exactly. At Phase 6, polish lists one prefix, finds an mp4 and no stills, and
regenerates ten frames — at $0.06 each and, per C-1, with no reliable style anchor. The polished video
does not match the cut Carrie approved, and she is told it is "the same video with animation added."

**Fix.** AD-7 fixes bucket, key layout, client and credential route. Suggested:

> Artifacts live under `s3://tiktoktrivia/<run-id>/` on `s3.delo.sh`. Access is via `mc` using the
> `delo` alias. If the bucket does not exist, create it before the first run rather than falling back
> to a shared bucket.

### H-2 — AD-5 fixes a URL's expiry and leaves everything that matters about narration open

AD-5 governs vox's HTTP `/synthesize-url`. That is not the surface the agent has. The profile's vox MCP
server exposes `speak`, `speak_url`, `speak_sink`, `list_voices_tool`, and the three that produce audio
produce three different things:

- `speak` — *"return the audio as base64-encoded WAV bytes"*
- `speak_url` — *"a short-lived OGG/Opus URL... expire after VOX_AUDIO_TTL_SECONDS (default 1h)"*
- `speak_sink` — routes to a listening machine, also returns `audio_url`

AD-5's operative clause is *"download the audio and write it to durable storage before any render
references it."* `/tmp` satisfies that sentence.

**The two units.** Two skeleton-cut sessions:

- **Session A** calls `speak_url`, curls the OGG to `/tmp/narration.ogg`, muxes, uploads only the mp4.
  AD-5 satisfied — the render did not reference an expired URL.
- **Session B** calls `speak` and writes the WAV to the run's S3 prefix.

Both are compliant. The break is at Phase 6. AD-6 says polish *"lengthens the filtergraph and changes
nothing else"* — which presumes the same inputs are still there. Session A's narration is gone, so
polish re-synthesizes. vox is VoxCPM2, a diffusion TTS (`cfg`, `steps` are input parameters); a
re-synthesis is not sample-identical. Every per-scene duration shifts, the pause-to-guess gaps move,
and burned-in ASS caption timings authored against the first render desync. Session A also muxed Opus
where B muxed WAV, so the AAC transcode differs.

That is CAP-5 ("every polish change lands on an existing full-length cut") failing while every AD reads
green.

**Fix.** Tighten AD-5:

> The exact narration bytes used in a render are written to that run's S3 prefix **before** the render,
> and every later render of the same run re-fetches those bytes. Narration is never re-synthesized for
> a run that already has a cut — vox is a diffusion model and a second synthesis is a different
> performance, which silently invalidates every timing derived from the first.

### H-3 — Three stores hold "what phase are we on" and none of them owns it

AD-2 fixes the memory bank (`agent-dumply`). The capability map puts CAP-14's notebook in "S3 + agent
memory". AD-3 puts BMAD's plan inside the agent. The Plane board is deferred as "a lesson prop." That
is four candidate homes for one fact and no owner.

It is already leaking. The profile has *both* a Hindsight bank and a local
`profiles/dumply/memories/MEMORY.md`, and that file today contains one memory — about the **James
Brennan** PM agent's aliases. The store the spine leans on is demonstrably not scoped to Dumply.

**The two units.**

- The **video skill** advances BMAD's internal plan when Phase 4's handback lands.
- The **between-session invite** (CAP-11) is a separate unit on a Hermes cron. It reads memory to name
  the next phase.

Nothing says the first writes where the second reads. Carrie gets *"ready to make the pictures?"* the
morning after she made the pictures. CAP-11 is scored on the invitation naming the *specific* next
phase and CAP-10 on her naming it at session start — both fail from one unowned duplication, and the
failure reads to her as the agent not remembering her, which is CAP-8.

**Fix.** An AD naming one authoritative resume pointer:

> `s3://tiktoktrivia/<run-id>/RUN.md` is the single record of a run: phase completed, handbacks and
> their keys, what is next. Agent memory and the Plane board are derived views and are never read to
> decide what happens next.

### H-4 — The Models convention has no mechanism, and the spine skips the readiness rule it inherited

Conventions say *"Cheap models for retrieval... strong models where judgment is the product."* The
phase catalog's readiness rule is harder: *"A phase with no explicit model assignment is not ready to
run."* The spine assigns no models to any phase.

The live profile has three routing mechanisms and the spine names none of them:

- `model.default: kimi-for-coding` — the one model the conversation runs on
- `delegation.model: deepseek/deepseek-v4-flash` (openrouter), `orchestrator_enabled: true`,
  `subagent_auto_approve: true`, `default_toolsets: [terminal, file, web]`
- `moa.enabled: true`, with a gpt-5.5 / gpt-5.6-sol / deepseek-v4-pro panel

**The two units.** Two authors write Phase 2 (question research), the phase the curriculum names as
*the* example of casting the cheap worker:

- **Skill A** spawns a delegation subagent. It genuinely runs on deepseek-v4-flash. But a delegated
  child returns a **text summary to the parent**, not a file — so the handback's form is a chat
  message, and the "written to a durable document" half of Phase 2's handback has to be redone by the
  parent, which may or may not preserve what the child found.
- **Skill B** runs the questions inline on `kimi-for-coding` and tells Carrie "we used the cheap one
  for this." Nothing routed. The learning-outcome check — *"picks a cheap model for finding questions,
  and can say why"* — is then scored against a fiction, in a project whose defining failure is a
  confident wrong thing reaching someone who cannot tell.

**Fix.** An AD naming the mechanism, the tier models, and the handback form:

> Cheap-tier work runs as a delegation subagent (`delegation.model`); the conversation model is the
> strong tier. A delegated phase writes its handback to the run prefix and returns the key — never a
> summary in chat, because the summary is not the artifact.

---

## MEDIUM

### M-1 — The run id is not unique, and nothing says what a rejection does to it

`YYYY-MM-DD-<topic-slug>` collides for two videos on the same topic the same day. More pressing: AD-8
makes rejection the *taught* behavior, so re-renders are the common path, not the exception.

Also, the id cannot be minted where the spine implies. Phase 1's handback is *"a written account of
what makes a good trivia video of this kind"* — topic-free. The topic first exists at Phase 2. So the
slug does not exist during the phase that runs first.

**The two units.** Stage-and-approve, on "I don't like it":

- **Skill A** re-renders to the same prefix, overwriting `final.mp4`. Carrie cannot compare v1 and v2 —
  and comparing them *is* the iteration lesson.
- **Skill B** mints `2026-09-20-80s-movies-2`. New prefix, no stills under it, polish cannot find them,
  and the name she was told changed under her.

**Fix.** `YYYY-MM-DD-<slug>[-NN]`, minted once at Phase 2 and written to `<prefix>/RUN.md`; renders are
versioned *within* a run (`cuts/skeleton-01.mp4`, `cuts/polish-02.mp4`) and never overwritten.

### M-2 — No naming rule below the prefix, and lexical globbing will reorder the video

The spine fixes the prefix and nothing inside it.

**The two units.** The stills skill writes `still-1.png … still-10.png`. The skeleton-cut skill
assembles with `ffmpeg -pattern_type glob -i 'still-*.png'`. Glob order is lexical:
`still-1, still-10, still-2, …` — question 10's frame plays in slot two, under narration for question 2.
Zero-padding fixes it; nothing requires it. Both skills obey every AD.

The same gap kills resumption. A render that dies at frame 7 can only resume if the next session can
tell which frames exist, which needs a fixed filename ↔ question-index mapping. Without one, recovery
is "regenerate all ten" — new cost, and per C-1 a different look than the seven Carrie already saw.

**Fix.** Fix the layout in the same AD as H-1:
`<run>/stills/q01.png`, `<run>/narration/q01.wav`, `<run>/cuts/<name>-NN.mp4`, `<run>/RUN.md`.

### M-3 — AD-6 carries no render contract, so polish can be authored against a frame the skeleton never produced

The memlog verified a full chain — 1080×1920, H.264/yuv420p, 30fps, AAC 128k — and the spine carried
none of the numbers forward. AD-6 says only "invoke ffmpeg."

**The two units.** The skeleton-cut session renders 1920×1080 (the default orientation habit, and per
C-1 the stills may well be square). The polish session authors burned-in ASS captions — the chain the
memlog actually ran — and an `.ass` script carries `PlayResX`/`PlayResY`. Captions authored for
1080×1920 against a 1920×1080 cut render at the wrong scale and off-frame. Question-calibration makes
caption legibility a product requirement, not a cosmetic one: *"Captions must be legible at the speed
the gap allows, since reading is how the viewer holds the question during the pause."*

**Fix.** Promote the verified contract into AD-6 as fixed values, and state that captions are authored
against them.

### M-4 — AD-8's delivery channel has a 50 MB ceiling that AD-6 does not respect

From `core.telegram.org/bots/api`: *"Bots can currently send video files of up to 50 MB in size."*
2000 MB requires a local Bot API server; none is configured. With no bitrate fixed (M-3), a two-minute
1080×1920 cut is roughly 30 MB at 2 Mbps and 120 MB at 8 Mbps — the difference is one unstated flag.

**The two units.**

- **Session A** renders at CRF 18 for quality. `sendVideo` fails. The failure lands on Carrie as "the
  thing broke," at the skeleton-cut session — which the phase catalog names as *"the highest-risk
  moment in the curriculum."*
- **Session B** falls back to a presigned S3 link. Carrie opens a browser: a second interface, which is
  what AD-3 and CAP-8 exist to prevent.

**Fix.** AD-6 fixes a size target under 50 MB; AD-8 says what happens when a render exceeds it, in
terms that do not put a URL in front of her.

### M-5 — AD-1's "a script a skill calls" has nowhere to live

AD-1 permits scripts. AD-7 enumerates what the repo holds: *"the agent file, skills, and planning
artifacts."* A script is not on that list.

**The two units.** The render skill embeds its ffmpeg command inline in `SKILL.md` — which is how M-3
happens, since the command is re-derived from prose each session. Or it writes `scripts/render.sh`:
into the repo, violating AD-7's enumeration, or into the profile directory, which is ignored runtime
that dies on the next rebuild — precisely the fate AD-7 exists to prevent for artifacts.

**Fix.** Extend AD-7: skill-owned scripts live beside their `SKILL.md` inside the skill directory and
are committed with it. That also makes the verified ffmpeg chain a file rather than a paraphrase.

### M-6 — AD-3's guarantee is unenforced while the toolset scope sits in Deferred

AD-3: *"She is never handed a workflow name, phase code, or command to operate."* Deferred: *"Dumply
inherited all 82 fleet skills... Over-broad for this use. Revisit before she has unsupervised
sessions."*

Those two cannot both be true now. The global path Dumply loads (`/home/delorenj/.agents/skills`, 62
skills) carries `momo`, `project-lifecycle`, `board-taxonomy`, `agent-fleet-operations`, `projects`,
`recap`, `worktrunk` and the full `bmad-*` set, and the bundled profile dir adds 80 more including
`github`, `devops`, `mlops` and `autonomous-ai-agents`. Carrie says *"can you make a board for this?"* —
which the spine actively wants her to say, since the Plane board is a planned lesson beat — and a
skill fires that speaks in ticket identifiers, lane maps and workflow names, straight at her.

That is an AD-3 violation whose cause is sitting in Deferred. A deferred item that can break an
invariant is not deferred; it is unowned.

**Fix.** Either promote it into an AD (an explicit skill allowlist for the `dumply` profile, enforced
via `skills.disabled` or a scoped `external_dirs`), or state plainly in AD-3 that its guarantee is
unenforced until that lands, so nobody reads the invariant as a fact about the running system.

---

## What holds up

Worth saying, since the above is all objection:

- **AD-1 and AD-2 are correctly scoped.** They forbid the right things and name the one profile, and
  nothing in the pairs above argues for a service layer — every fix is a tightened sentence in an AD or
  a file naming convention, which is exactly the right shape for this paradigm.
- **AD-6's exclusions are the strongest content in the document.** MoviePy and ffmpeg-python are ruled
  out with reasons that survive contact, and the reasons are the kind a future builder would otherwise
  rediscover expensively.
- **AD-8 and AD-9 are the two that protect the actual product**, and neither has a hole in it. AD-9 is
  the one the spine itself violated twice (C-1, C-3), which is an argument for the rule, not against it.
- **The paradigm choice is right for the failure mode this project is defending against.** A pipeline
  would make the iteration beat — "you don't like it, what don't you like" — the hard path.

## Summary of proposed changes

| # | Change | Where |
|---|---|---|
| C-1 | Rewrite AD-4 against fal's field names; pick one style anchor; fix `image_size` | AD-4 |
| C-2 | New AD-10: absolute repo path in `skills.external_dirs`; agent-authored skills land in the repo and are committed | new AD |
| C-3 | Delete the web-search blocker; pin `web.backend: exa` | Deferred |
| H-1 | Name bucket, key layout, client and credential | AD-7 |
| H-2 | Narration bytes to the run prefix before render; never re-synthesize a run that has a cut | AD-5 |
| H-3 | One authoritative resume pointer; memory and Plane are derived views | new AD or AD-2 |
| H-4 | Name delegation as the cheap tier; delegated handbacks are files, not summaries | Conventions → AD |
| M-1 | Run id uniqueness, mint point, and what a rejection does | Conventions |
| M-2 | Per-artifact layout under the prefix | AD-7 |
| M-3 | Promote the verified render contract into the rule | AD-6 |
| M-4 | Size target under Telegram's 50 MB; state the over-size path | AD-6, AD-8 |
| M-5 | Skill-owned scripts live beside their SKILL.md | AD-7 |
| M-6 | Promote toolset scope out of Deferred, or mark AD-3 unenforced | AD-3 / Deferred |
