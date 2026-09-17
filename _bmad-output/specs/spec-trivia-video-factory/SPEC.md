---
id: SPEC-trivia-video-factory
companions:
  - phase-catalog.md
  - glossary.md
  - question-calibration.md
  - learning-outcomes.md
sources:
  - ../../../BRAINDUMP.md
---

> **Canonical contract.** This SPEC and the files in `companions:` are the complete, preservation-validated contract for what to build, test, and validate. Source documents listed in frontmatter are for traceability — consult them only if you need narrative rationale or prose color this contract intentionally omits.

# Trivia Video Factory

## Why

A vision to realize, with a person at the center of it. Carrie wants to make TikTok trivia videos — a format that works because the viewer pauses to guess before the answer lands — and she wants AI to make them. Carrie starts out skeptical of AI and willing to learn anyway; that skepticism is the pre-state every success check measures against. Jarad is teaching her, and his own stated failure mode is dumping everything at once: MCP servers, skills, CLI choice, model choice, delivered as one batch that scares people off. So the pipeline and the curriculum are deliberately the same artifact: each phase of making the video is one lesson, and a concept is named only after the capability it bought has already done something she wanted. The video is the vehicle; the cargo is a working mental model of what AI is for. Carrie is the boss in this arrangement, which carries a boss's credit and a boss's blame — that accountability is precisely what makes a weak first video her call to redirect rather than evidence against the tool.

## Capabilities

- **CAP-1**
  - **intent:** A research run returns a calibrated question-and-answer set sized for one video, without Carrie sourcing any of it.
  - **success:** One invocation yields a Q&A set that reads aloud in roughly two minutes with answer gaps, persists outside the chat, and clears the bar in `question-calibration.md`.
- **CAP-2**
  - **intent:** A creative-direction step decides what the video looks like, grounded in which real trivia videos performed and why, and produces no media.
  - **success:** The step outputs a descriptive screenplay plus an art-and-style description naming one chosen visual form, cites the specific videos it learned from, and completes with zero image or video assets generated.
- **CAP-3**
  - **intent:** A stills step turns the question set plus the art direction into one frame per question.
  - **success:** Ten questions in yields ten frames out, and swapping the image generator changes nothing about what the next step receives.
- **CAP-4**
  - **intent:** A full-length watchable video exists early and deliberately unpolished, so Carrie sees the whole shape before any phase is deepened.
  - **success:** A complete cut of stills plus narration reading each question and answer, with the pause-to-guess gap in place, is timestamped before any animation, caption, or sound work begins.
- **CAP-5**
  - **intent:** Polish adds animation, captions, and sound to a video that already plays end to end.
  - **success:** Every polish change lands on an existing full-length cut; no polish work starts against an incomplete video.
- **CAP-6**
  - **intent:** A finished video waits for Carrie's decision rather than posting itself.
  - **success:** Each rendered video reaches a staging state where Carrie approves or rejects it, and nothing reaches TikTok without that approval.
- **CAP-7**
  - **intent:** Each successive video takes less instruction than the one before, trending toward running itself.
  - **success:** Instruction turns per video, counted across videos 1..N, decline rather than hold flat.
- **CAP-8**
  - **intent:** Carrie's entire experience is a conversation with one agent that remembers her and the project across sessions.
  - **success:** She completes a full phase without opening a file, running a command, or being handed a second interface.
- **CAP-9**
  - **intent:** The agent explains where a new capability came from and why it is being added, before adding it.
  - **success:** Every tool or MCP server installed for her is preceded by a stated source and reason; nothing appears silently.
- **CAP-10**
  - **intent:** Every session closes by telling Carrie what they did, what she learned, and what comes next.
  - **success:** A session report exists for each session, and at the next session's start she can name the next phase before being told.
- **CAP-11**
  - **intent:** The agent reaches out between sessions to invite her into the next phase by name.
  - **success:** The invitation names the specific next phase, and across the run she says yes more often than no.
- **CAP-12**
  - **intent:** Each pipeline phase doubles as one lesson delivering exactly one new concept, attached to something she actually wanted done.
  - **success:** Every lesson ends with a working capability that produced an artifact she asked for, and introduces at most one new concept, named only after it did something.
- **CAP-13**
  - **intent:** Carrie's standing instructions become reusable skills the agent builds by interviewing her.
  - **success:** At least one skill exists whose content came from her answers rather than from Jarad's authoring.
- **CAP-14**
  - **intent:** Carrie accumulates a notebook of the journey she can read back and add to on her own.
  - **success:** After several sessions the notebook holds a retrievable record of each, and she has added to it unprompted at least once.
- **CAP-15**
  - **intent:** Carrie can split a job she wants done into assignable subtasks, each with a named handback.
  - **success:** Given a fresh task unrelated to trivia, she produces distinct subtasks that each name the artifact coming back.
- **CAP-16**
  - **intent:** The curriculum exists as a written onboarding plan rather than in Jarad's head.
  - **success:** A phase-by-phase plan exists on disk that someone other than Jarad could run Carrie through.

## Constraints

- One new concept per lesson. MCP servers, skills, CLI choice, and model choice are named as the exact batch that must never arrive together.
- A lesson's objective is a task outcome, never comprehension of a technology. "The AI can search Google for you" is a lesson; "understand MCP" is not.
- Every session ends mid-appetite with a named next hook, not at a clean point of completion.
- Every pipeline stage is defined by the artifact it hands back. A stage described by its activity is not yet specified.
- Decomposition granularity is set by the school-group test: split the work the way a project leader would split it among three to six people. Neither one monolithic agent nor an atomized micro-step graph.
- Every stage carries an explicit model assignment justified by task difficulty. Routing everything to the frontier model is named as wasteful, not as harmless.
- End-to-end before deep. No phase gets polished until a full-length watchable cut exists.
- Question calibration is the product's quality bar, not render quality — see `question-calibration.md`.
- The cut leaves a gap for the viewer to pause and guess. Existing videos in this trend are too fast to play along with, which is the defect these videos can fix.
- BMAD structure runs inside the agent and is never surfaced to Carrie as commands to operate.
- The mindset argument is never delivered as a lecture; it is reached by doing the work — see `learning-outcomes.md`.
- The surface Carrie works through is fixed before any sub-lesson is authored, and lessons are written against that one surface.
- Early output quality is pre-framed as expected and uninformative, never apologized for.
- No raw jargon, and no plain-language substitute that condescends.

## Non-goals

- Shipping one video. The objective is a repeatable crew, not a deliverable.
- A tool tour, or breadth coverage of the AI landscape.
- Procedural recall. Carrie is not expected to reproduce the steps afterward, so the curriculum is not built to be memorized.
- Teaching the mindset argument as stated content.
- Carrie operating BMAD, a CLI, or any second interface.
- The street-interview variant of the format. A stills-and-voice pipeline cannot produce it, and staging it would depict interviews that never happened.
- Elaborating the trivia format itself. The format is deliberately minimal; the difficulty lives entirely in question selection.
- Carrie's adoption of AI. She may finish and decline to embrace it; the target is that she can see how it would help, not that she converts.

## Success signal

Asked at the end what she got out of this, Carrie answers about leverage rather than reciting tools, and when handed a fresh unrelated task she splits it into subtasks that each name what comes back. The defining failure is the exact inverse and is not redeemed by shipping: she finishes with a real video in hand, judges it bad, and generalizes that to AI being bad. The full check battery is in `learning-outcomes.md`, and the first read on it is due within a couple of lessons rather than at the end.

## Assumptions

- Carrie is Jarad's wife and this is a shared domestic project, not a training engagement. That sets the agent's register and the tolerance for a between-session nudge.
- Trivia videos are something Carrie genuinely wants to make. The transcript names a diagnosis where the work not being fun means the subject is wrong — which would make the right response a different project, not more lessons.
- Ten questions in a two-minute video is the working output contract. The transcript offers it as an example figure.

## Open Questions

- Where does the research step source questions — quiz databases, TikTok itself, web search, or model generation? Blocks the first lesson and which tool is introduced first.
- Does a research run return a topic per video, or only a bare question list? Changes its output contract and what creative direction consumes.
- Is ten questions in two minutes the contract or an example? Sets frame count and narration budget.
- Does research run daily or weekly, and does one run produce one video or a backlog? Sets scheduling and how many videos sit awaiting approval.
- Is Google Sheets via MCP the committed home for question sets, or one candidate among several?
- Which image generator — Ideogram, fal.ai, ChatGPT, or Canva? Blocks the media client, the key, per-image cost, and whether style consistency across ten frames is achievable.
- What reads the questions in the skeleton cut? TTS is a hard dependency of the pipeline and nothing names one.
- Is the visual form pre-constrained to one, or does the creative step pick among captioned stills, animated text, stock video, and generated video? Leaving it open means building for four renderers.
- Does the creative step need real TikTok access to study performance, and through which tool — an existing MCP server or one built for it?
- For the creative-direction lesson, is the concept taught MCP or skills? Both are candidates for the same slot, and two in one slot breaks the one-concept rule.
- Does Orca stay behind Hermes as the backing model, or is it dropped?
- Does Carrie ever open the notebook directly, or does the agent read and write it for her? Decides whether a second interface has to be made safe for a non-technical user.
- How is the walking-skeleton idea said to Carrie without landing as an excuse for shipping something bad? This is the exact moment the defining failure is most likely.
- What is the stop-line for "too much" in one session — how do you know you overshot before she does?
- How often does the agent prod her, on what channel, and what happens when she ignores it?
- Is the amplifier argument ever said to her out loud, or only ever demonstrated?
- Nothing verifies that answers are correct, and question-finding is assigned to the cheapest model. Is a fact-check step in scope?
- Repeated research runs will resurface the same questions. Where does dedupe history live?
- Does TikTok's AI-generated-content disclosure apply to these videos, and does that change the render?
- What is the per-video ceiling across image generation, TTS, and tokens at a daily cadence?
- Music and sound effects arrive in polish. Licensed source, or does TikTok's own library cover it?
