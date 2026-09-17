---
review: good-spine checklist
target: ../ARCHITECTURE-SPINE.md
reviewer: architecture review (rubric lens)
date: '2026-09-17'
verdict: pass-with-fixes
---

# Rubric review — ARCHITECTURE-SPINE.md

**Verdict: pass with fixes.** The paradigm is right and the media-tier decisions (AD-4, AD-5,
AD-6, AD-8) are the best-formed ADs I have read on this project — each names the excluded
alternative *and* why, and each was probed rather than recalled. What the spine misses is not
structure (correctly rejected) but **substrate**: under an agent-plus-skills paradigm, the two
things that carry all the load are *where a skill lives* and *what the agent remembers*, and
both are currently wrong against the machine. Plus the operational envelope is almost entirely
silent.

Everything below was verified read-only on big-chungus today, not recalled.

---

## What passes, plainly

- **AD-1** is a real invariant, not a slogan. "New capability is added as a skill, an MCP server,
  or a shell invocation" is checkable by inspection, and the escape hatch ("a script a skill
  calls — not a package with a lifecycle") closes the loophole a future builder would use.
- **AD-4 / AD-5 / AD-6** are the model. Each states the positive path, the excluded path, and the
  mechanism of the failure. AD-4's "never v4" survives the pressure test because it explains
  *why the newer model is the wrong model* — an agent that only knew "use v3" would upgrade.
- **AD-2** is enforceable and I confirmed it holds:
  `hermes-dumply-gateway.service` sets `WorkingDirectory=/home/delorenj/code/TikTokTrivia`, so the
  agent file's instruction to read `_bmad-output/specs/spec-trivia-video-factory/phase-catalog.md`
  "in your working directory" actually resolves. Service is `active` and `enabled`.
- **AD-8** is enforceable: there is no publish path, and the rule says so in a way that a builder
  adding one would have to consciously violate.
- The **Stack** table checks out: `ffmpeg 7.1.1-1ubuntu4.2` built with `--enable-libass
  --enable-libx264 --enable-libfreetype --enable-libfontconfig`; `FAL_KEY` present in the
  gateway's `EnvironmentFile`; `mc` aliases `delo`/`deloroot` resolve `https://s3.delo.sh`.
- **CAP-15** is handled honestly — "emergent; measured, not built" is the correct answer, and
  saying so is better than inventing a mechanism.

---

## Findings

### 1. CRITICAL — Skills have no location-of-truth, and the one the spine implies does not load

Under AD-1 the skills *are* the product. The Structural Seed puts them at `.agents/skills/`
("canonical skills"), and the CAP-13 row says new skills go "under `.agents/skills/`". Three
verified facts make that wrong in three different directions:

1. **Project skills are trust-gated and this repo is not trusted.** Hermes discovers project
   skills at `<root>/.hermes/skills/` and `<root>/.agents/skills/`
   (`agent/skill_utils.py:663` — `PROJECT_SKILLS_SUBDIRS`), but only loads them when the root is
   listed in `skills.trusted_project_dirs`. Dumply's config has:

   ```yaml
   skills:
     external_dirs:
     - /home/delorenj/.agents/skills
     - ./agents/skills
     trusted_project_dirs:
     - /home/delorenj/code/PoopToTheMoon
   ```

   TikTokTrivia is absent. A video skill written to `/home/delorenj/code/TikTokTrivia/.agents/skills/`
   loads for nobody. The 49 `bmad-*` skills already sitting there are invisible to Dumply for the
   same reason — see finding 6.

2. **The `./agents/skills` entry is dead, and silently so.** Relative `external_dirs` entries
   resolve against `HERMES_HOME`, not cwd (`skill_utils.py`, `get_external_skills_dirs`:
   *"Resolve relative paths against HERMES_HOME, not cwd"*). So that entry means
   `/home/delorenj/.hermes/profiles/dumply/agents/skills`, which does not exist; the loader logs a
   debug line and moves on. (It is also missing the leading dot the repo actually uses.) The only
   external dir that resolves is the global `/home/delorenj/.agents/skills` — 62 fleet skills, none
   of them this project's.

3. **Agent-authored skills land somewhere else again.** `get_skills_dir()` is
   `get_hermes_home() / "skills"` (`hermes_constants.py:1544`), and the Hermes docs are explicit:
   *"New agent-created skills are written to `~/.hermes/skills/`."* For this profile that is
   `~/.hermes/profiles/dumply/skills/` — profile-local runtime state, untracked, wiped by a profile
   rebuild. That is precisely where CAP-13's skills — *Carrie's* skills, the ones she answered her
   way into — will go, and AD-7 protects the stills and the mp4 from exactly this fate while leaving
   the product itself exposed.

**Fix:** add an AD — call it AD-10, *Skills live in one place and that place is loaded* — that (a)
names `TikTokTrivia/.agents/skills/` as canonical, (b) names the two mechanisms that make it true
(`hermes skills trust /home/delorenj/code/TikTokTrivia`, and correcting the `external_dirs` entry to
the absolute repo path or dropping it), and (c) states the check: a skill is not installed until it
appears in Dumply's skill index tagged `[project]`. Then fix the CAP-13 row so it names where
agent-created skills actually land and the rule that moves them into the repo before the session
ends — otherwise "her skills are hers" is true for one session only.

### 2. CRITICAL — AD-9 names no permitted alternative, and the agent cannot execute it anyway

Two separate defects stacked on the spine's own anti-fabrication invariant.

**(a) It is a pure prohibition.** "No package, command, URL, model, or setting is stated to Carrie
… without being checked first" tells an agent under pressure what not to do and nothing about what
to do instead. This project has already paid for that lesson. Notably, the shipped agent file gets
it *right* — *"Go look it up, then answer… If you are not sure, say you are not sure and go check"* —
and the spine dropped the second half in the compression. Judged against the standard the lens sets,
AD-9 as written fails and the system prompt passes.

**(b) The agent physically cannot check.** Verified in the rendered profile config:

```yaml
web:
  backend: ''
  search_backend: ''
  extract_backend: ''
```

There is no search and no extraction. So AD-9's rule currently reduces to *"never name anything"*,
which is not a behavior an agent can hold — it resolves under pressure into the exact failure AD-9
exists to prevent. The spine does file the web backend under **Deferred** and marks it "Blocks
Carrie's first session," which is true but undersells it: the lens asks whether anything deferred is
load-bearing enough to permit divergence, and this is the one. It under-writes AD-9 (the only
invariant binding *all*), CAP-1, CAP-2, and the entire taught concept of Phase 1 ("it cannot answer
this from memory, it has to go look").

**Fix:** rewrite AD-9's Rule to carry the alternative in the same breath — *verify with a tool
before naming; if it cannot be verified, say it is unverified and go check; never state it anyway* —
and promote the web backend out of Deferred into a blocking Open Question that names AD-9, CAP-1,
CAP-2 and Phase 1 as its dependents, so the next person sees it is not merely a missing convenience.

### 3. HIGH — The question set has no durable home; a whole lesson has no substrate

AD-7 routes stills, narration and video to S3, and the repo to source-only. The question set is
neither. But CAP-1's success requires it "persists outside the chat," and Phase 2's *taught concept*
is literally persistence — *"the agent writes to a real spreadsheet she can open, because this runs
weekly."* The spine decides nothing here, and it is not in Deferred either. Dumply's MCP servers are
`pjangler`, `vox`, `plane`, `codegraph` — no Sheets, no docs surface. The spec flagged this as an
open question ("Is Google Sheets via MCP the committed home…") and the spine dropped it.

This is the cleanest example of a Capability handled nominally: the CAP-1 row says "video skill +
web tooling," which addresses sourcing and skips the handback.

**Fix:** decide it (Plane, a Sheets MCP, or a markdown doc in S3 she gets a link to — note the
choice must survive AD-3: she must not be required to open it, but CAP-1 requires she *can*), or add
it to Deferred explicitly marked as blocking Phase 2 and note which candidates are live.

### 4. HIGH — `agent-dumply` is the wrong memory bank for project continuity, by its own mission

AD-2 fixes the bank as `agent-dumply`, and the map routes CAP-8 ("one remembering agent") and CAP-14
(notebook) there. The profile's own memory config contradicts that:

```yaml
bank_id_template: agent-{profile}
bank_mission: 'Private identity memory for a single Hermes agent. Scoped to WHO the agent is,
  never to a repo or working directory: … Repository facts belong in the per-repo Hindsight bank
  reached through the hindsight CLI/skill, not here.'
```

Confirmed `~/.hermes/profiles/dumply/hindsight/config.json` is `{"bank_id": "agent-dumply"}`, and
`hindsight bank list` returns 198 banks with **no** `TikTokTrivia` bank — the project has no
compliant home for project state at all. So "which phase we're on," "what art direction she picked,"
"which questions we already used," and CAP-10's "at the next session's start she can name the next
phase" are all pointed at a bank whose stated contract excludes them. This fails quietly and only
after several sessions, which is when it costs the most — CAP-8 is the capability the whole surface
rests on.

**Fix:** AD-2 should name the split, not one bank: identity and working style in `agent-dumply`;
project state, phase position and question history in a per-repo bank (`TikTokTrivia`), created
before the first session. Then the CAP-8 and CAP-14 rows point at the right one.

### 5. HIGH — The operational envelope is essentially silent

Five dimensions this altitude owns, none decided, none deferred, none raised:

- **Long-running work.** A ten-image fal run plus an ffmpeg render is minutes inside a single turn.
  The platform handles an in-flight message mechanically — the base adapter queues it in
  `_pending_messages` and raises an interrupt (`gateway-internals.md:86`) — so nothing is *lost*.
  The undecided part is architectural: is a render a foreground turn or a backgrounded job, what
  does Carrie see while it runs, and what is the state of half-written stills when an interrupt
  lands mid-chain? Phase 4 is the phase the spec names as the impatience risk and Phase 5 exists to
  arrive before patience runs out — a silent multi-minute stall is the named failure route, not an
  edge case.
- **Failure mid-run.** Nothing says what a failed fal call or a non-zero `ffmpeg` exit means: retry,
  partial handback, or a clean abort. Under AD-1 the answer belongs in the skill, but the spine has
  to say the answer exists.
- **Artifact durability.** AD-7 names the host and no bucket. The Naming convention makes the slug
  "the S3 prefix" of an unnamed bucket. `mc ls delo` shows twelve buckets including generic
  `artifacts/` and `hot/` — two builders land in two different places. No retention, no lifecycle.
- **Cost ceiling.** Absent entirely. The memlog records that a *cost model* was scoped out of this
  run, which is not the same as scoping out a *ceiling* — the spec asks for a per-video ceiling
  across images, TTS and tokens at a daily cadence, and this is the dimension where the difference
  between one video and a backlog is real money. At minimum it belongs under Deferred with the
  reason.
- **Observability for Jarad.** CAP-10 gives *Carrie* a session report. Nothing gives Jarad a read on
  what happened. That collides with a hard timing constraint in `learning-outcomes.md`: the first
  read on the mindset checks is due "within a couple of lessons… while there is still runway." And
  CAP-7's success is a **counted** metric — instruction turns per video, declining across 1..N —
  with nothing counting. CAP-15 is labelled "emergent; measured, not built"; CAP-7 gets the same
  treatment without the same honesty.

**Fix:** one short "Operational envelope" section — foreground vs. background for renders and what
Carrie sees during one; run-failure semantics; bucket name plus retention; a cost ceiling (or an
explicit deferral with the reason); and where Jarad reads session outcomes. Each can be one line.
Silence is the finding, not the absence of a mechanism.

### 6. MEDIUM — AD-3 asserts a mechanism it does not supply

Split verdict. The *prohibition* half passes the pressure test cleanly: "Answering a technical
question she asked is not a violation — deflecting one is" names the permitted alternative in the
same breath, which is exactly right and is why AD-3 reads stronger than AD-9.

The *mechanism* half — "BMAD runs inside the agent to keep the plan on rails" — is an aspiration.
Which BMAD skills, invoked at which phase boundary, reading and writing what? Today the claim is
simply false: the 49 `bmad-*` skills in `.agents/skills/` are behind the trust gate from finding 1,
so nothing BMAD-shaped is loaded for Dumply at all. Either name the mechanism (the two or three
skills that actually run, and when) or downgrade the clause to "structured planning runs internally"
so it stops implying a wiring that does not exist.

### 7. MEDIUM — CAP-14 is answered with a storage location instead of a surface

"S3 + agent memory" says where bytes go. CAP-14's success says Carrie "can read back and add to on
her own," and passes only when "she has added to it unprompted at least once" — that is a *surface*
requirement, and it is in live tension with AD-3 and CAP-8 (no second interface). The spec raised
exactly this as an open question ("Does Carrie ever open the notebook directly, or does the agent
read and write it for her?"); the spine did not answer it or carry it forward. As written, CAP-14's
check cannot be run.

**Fix:** decide that the notebook is Telegram-mediated (she asks, Dumply reads it back, she dictates
additions) and say so — that satisfies CAP-8 and makes the check measurable — or defer it explicitly.

### 8. LOW — The secrets convention contradicts the working substrate

"`op://` references by item UUID, resolved at call time. Never a literal key in a skill, a config,
or a message." The credential the stills step actually uses is a literal `FAL_KEY` in
`~/.hermes/.env`, reaching the gateway through `EnvironmentFile=`. That is the fleet standard and is
almost certainly fine — but as worded, the convention declares the working configuration a
violation, which sends the next builder off to "fix" a thing that is not broken. Narrow it: `op://`
by UUID is the rule for anything a skill or planning artifact writes down; the fleet `.env` is the
sanctioned injection point for process-level keys.

### 9. LOW — CAP-2 does not carry the flag CAP-1 does

CAP-1's row is annotated "**open: no web backend**." CAP-2's is not, although its success criterion
is stricter — it must "cite the specific videos it learned from," which needs a way to study real
TikTok performance, not just general search. The spec flagged it ("Does the creative step need real
TikTok access to study performance, and through which tool…"); the spine carries it nowhere. Either
annotate CAP-2 the same way or add it to Deferred.

---

## Capability walk — where coverage is nominal

| CAP | Coverage | Note |
|---|---|---|
| CAP-1 | **partial** | sourcing decided; the durable handback is undecided and unflagged — finding 3 |
| CAP-2 | **nominal** | "video skill" + model routing; no tool for studying real performance — finding 9 |
| CAP-3 | solid | AD-4, incl. the swap clause that satisfies "swapping the generator changes nothing downstream" |
| CAP-4 | solid | AD-5 + AD-6, both verified end to end |
| CAP-5 | solid | AD-6; "polish lengthens the filtergraph and changes nothing else" is the right invariant |
| CAP-6 | solid | AD-8 |
| CAP-7 | **nominal** | success is a counted metric; nothing counts — finding 5 |
| CAP-8 | **at risk** | bank contradicts its own mission — finding 4 |
| CAP-9 | solid | agent file + AD-9, modulo finding 2 |
| CAP-10 | partial | report exists for Carrie; no durable artifact, no read for Jarad — findings 4, 5 |
| CAP-11 | decided | Hermes cron on this profile; no job defined yet (implementation, not spine) |
| CAP-12 | solid | agent file carries it well |
| CAP-13 | **broken** | stated home does not load and is not where skills are written — finding 1 |
| CAP-14 | **nominal** | storage named, surface undecided — finding 7 |
| CAP-15 | solid | honestly emergent |
| CAP-16 | deferred | correctly, and scoped out of this run by direction |

## Deferred items re-judged

| Item | Verdict |
|---|---|
| Web search backend | **Not deferrable.** Load-bearing for AD-9, CAP-1, CAP-2, Phase 1 — finding 2 |
| Toolset scope (82 inherited skills) | Fine to defer. Real but not divergence-permitting |
| Plane board | Fine. Correctly called a lesson prop, not a dependency |
| `docs/` curriculum (CAP-16) | Fine, scoped out by direction |
| Cadence and dedupe | Fine — the reasoning ("nothing to deduplicate against yet") is sound |
| Fact-checking | Fine to defer, but the trigger is weak: "revisit when a wrong answer actually ships" means the trigger *is* the failure. Prefer "before the first post," since CAP-6 already puts Carrie in the loop |
| AI-disclosure | Fine, correctly bound to the first real post |
| *(missing)* Cost ceiling | Should be here — finding 5 |
| *(missing)* Question-set home | Should be here or decided — finding 3 |

---

## Suggested minimal edit set

1. New **AD-10 — Skills live in one place and that place is loaded**, with the trust command and the
   index check as its enforcement, plus a corrected CAP-13 row.
2. Rewrite **AD-9's Rule** to name the permitted alternative; promote the web backend from Deferred
   to a blocking Open Question bound to AD-9 / CAP-1 / CAP-2 / Phase 1.
3. Extend **AD-2** to name the two-bank split; create the `TikTokTrivia` bank.
4. Decide or explicitly defer the **question-set home**, marked as blocking Phase 2.
5. Add a short **Operational envelope** section: render foregrounding and progress, run-failure
   semantics, bucket + retention, cost ceiling, Jarad's read.
6. Downgrade or specify **AD-3's BMAD clause**; annotate **CAP-2**; narrow the **secrets convention**.
