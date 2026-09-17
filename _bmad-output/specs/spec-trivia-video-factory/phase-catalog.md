# Phase catalog

Every phase is simultaneously a pipeline stage and one lesson. A phase is specified only when its
**handback** — the concrete artifact it returns — is named; that description is the instruction.
Each phase carries exactly one new concept (CAP-12) and one model assignment justified by difficulty.

Phase 0 is a prerequisite Jarad completes alone; Carrie's first session starts at Phase 1.

---

## Phase 0 — Surface lock-in

*Prerequisite. Not a lesson; Carrie never sees it.*

| | |
|---|---|
| **Handback** | Carrie's Hermes agent profile, provisioned and primed |
| **Concept taught** | None — this is the substrate every later lesson sits on |
| **Model** | n/a |

Fixing the surface before authoring any sub-lesson is a constraint, not a convenience: lessons written
against a drifting interface have to be rewritten. Open: whether Orca stays behind Hermes as the
backing model, and whether the notebook is a second interface Carrie ever opens directly.

---

## Phase 1 — Trend research

| | |
|---|---|
| **Handback** | A written account of what makes a good trivia video of this kind, drawn from real examples |
| **Concept taught** | Giving the AI a tool — it cannot answer this from memory, it has to go look |
| **Model** | Cheap. This is retrieval, not judgment. |

The concept lands because she wants the answer, not because MCP is on a syllabus. Open: whether
studying real TikTok performance needs an existing MCP server or one built for it.

---

## Phase 2 — Question research

| | |
|---|---|
| **Handback** | A calibrated question-and-answer set, sized for one video, written to a durable document |
| **Concept taught** | Persistence — the agent writes to a real spreadsheet she can open, because this runs weekly |
| **Model** | Cheap, deliberately. This is the named example of casting the cheap worker. |

The hard part of the whole product lives here — see `question-calibration.md`. The step runs
repeatedly, which is exactly what makes durable output worth teaching rather than asserting.

Two gaps carried forward: nothing verifies the answers are correct, and repeated runs will resurface
the same questions with no dedupe history.

---

## Phase 3 — Creative direction

| | |
|---|---|
| **Handback** | A descriptive screenplay plus an art-and-style description, naming one chosen visual form |
| **Concept taught** | Skills — encoding what to look for and what shape to report back in |
| **Model** | Strong. Judgment is the product here. |

Produces vision only; zero media. The brief must settle on one form rather than presenting a menu of
captioned stills, animated text, stock video, and generated video — leaving it open means building
four renderers. Studying what performed well is its own activity within this phase.

There is no house style yet; the style is the output of this research, not an input to it. Once enough
videos exist, Carrie hands over her established style and this phase collapses to a reference.

Open: whether the concept taught here is MCP or skills. Both are candidates for this one slot, and two
in one slot breaks the one-concept rule.

---

## Phase 4 — Stills

| | |
|---|---|
| **Handback** | One still frame per question — ten questions in, ten frames out |
| **Concept taught** | Generation, and that the provider behind it is swappable |
| **Model** | Image generation; choice open |

This is where impatience is anticipated: a pile of intermediate assets and still nothing to watch.
Phase 5 exists to arrive before that patience runs out.

Open: Ideogram, fal.ai, ChatGPT, or Canva — the pick decides the client, the key, per-image cost, and
whether style consistency across ten frames is achievable at all.

---

## Phase 5 — Skeleton cut

| | |
|---|---|
| **Handback** | A complete, watchable, deliberately ugly video — stills plus narration, no animation, no captions, no effects |
| **Concept taught** | End-to-end before deep; iteration as the normal shape of the work |
| **Model** | TTS; choice open |

The narration reads each question, leaves the pause-to-guess gap, then reads the answer. This is the
first artifact that is actually a video, and it must be timestamped before any polish work starts.

This phase is the highest-risk moment in the curriculum. The deliverable is by design not good, and
the named total-misfire failure is Carrie concluding the video is bad therefore AI is bad. How the
walking-skeleton idea gets said to her — without it sounding like an excuse — has to exist before this
session runs, and must not come out condescending.

---

## Phase 6 — Polish

| | |
|---|---|
| **Handback** | The same video with animation, captions, and sound effects added |
| **Concept taught** | That she is training a repeatable crew, not producing one video |
| **Model** | Mixed |

Every change here lands on a cut that already plays end to end. Each addition is framed as the part
that is hers — the accumulating evidence for CAP-15 and the mindset target.

Open: whether music and sound effects come from a licensed source or TikTok's own library.

---

## Phase 7 — Stage and approve

| | |
|---|---|
| **Handback** | A rendered video sitting in a staging state, waiting on Carrie |
| **Concept taught** | That she is the boss — with the credit and the blame |
| **Model** | n/a |

Nothing posts without her explicit approval. The approval gate is not a safety measure bolted on; it
is where the boss framing becomes literal, and where rejecting and redirecting an output is the
behavior being measured.

Open: whether TikTok's AI-generated-content disclosure applies and changes what gets rendered.

---

## Cross-cutting

- A phase with no named handback is not ready to run.
- A phase with no explicit model assignment is not ready to run.
- A phase that teaches two new concepts is over budget; split it.
- Phase count and size are set by the school-group test — the way a project leader would split this
  among three to six people.
